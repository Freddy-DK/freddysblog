---
layout: post
title: "Using SQL Server on the host"
date: 2019-11-04 20:29:00
categories: ["Docker", "NavContainerHelper", "PowerShell"]
tags: ["Docker", "NavContainerHelper", "SQL Server 2017", "SSMS"]
permalink: /2019/11/04/using-sql-server-on-the-host/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I have had quite a few partners asking me how to connect a NAV / Business Central container to a SQL Server placed on the host. This is the way I started a blog post back in april here: [https://freddysblog.com/2019/04/12/placing-the-database-on-the-host/](/2019/04/12/placing-the-database-on-the-host/) and elegantly, I avoided to answer the question and instead described how to place the database on the host and use the SQL engine in the container. Of course this doesn’t solve the issue if you have a very large database, which SQL Express doesn’t support.

In this blog post I will try to address the original problem.

# The problem

As described in the original blog post, the problem is that a container cannot really see the host, but need to connect to the host like any other machine on the network. Furthermore, if you are running standard networking for containers, the container will be behind NAT on the host and might not even be able to see machines on the network.

This picture is a very simple image of a network:

![network](/assets/images/2019/using-sql-server-on-the-host/network.png)

Remember that there are many many different ways to configure networks and this picture might not at all resemble what your setup looks like.

In this example, the local network is using **NAT** (Network Address Translation). The router is configured with **DHCP** to give out **local IP numbers** for the Computers, but when **Computer1** and **Computer2** connects to the internet, they will look like they connect from **140.32.45.66** (global IP number). This is because of **NAT**, everything behind the Router is internal.

Typically **Computer1** can ping the IP number of **Computer2**, but if the network doesn’t allow **ICMP**, then ping is not allowed. Whether **Computer1** can resolve the name **Computer2** to it’s IP number depends on the name resolution configuration in the Local Network.

**Connections from the outside** to the global IP number not be able to hit any of the computers unless you have defined that port **xx** should go to **Computer1** and port **yy** should go to **Computer2**.

In this example, **Docker** is also using **NAT**. The **Docker Virtual Network** acts like a router with **DHCP** and hands out IP numbers to containers in the virtual network. The **IP ranges** and **IP number** varies and they might even get new **IP numbers assigned** after a Computer restart.

**Container1** might be able to ping **192.168.1.116** (**Computer2**) if **ICMP** is allowed, but it is unlikely that it can do name resolution of **Computer2** to it’s IP number. For **Container1** to be able to do name resolution of **Computer1** is exactly the same problem as with **Computer2** as this name resolution is done in the same place. Maybe it works when your laptop is connected to your **home network**, but not when you are connected to the **corporate network @ work**.

Like with computers in the local network, the only way anybody can connect to Container1 from the local network or from the internet is if routing tables have been defined telling the network which ports to forward to which container.

So, if **Container1** wants to use the **SQL Server** on **Computer1** (above) it needs a way to define this. The **IP number is changing all the time** and it **cannot do name resolution of Computer1** – this is the problem.

# A possible solution

In real life, computers will have more **IP numbers**. One for each network adapter (**wifi**, **ethernet**). Also virtual network adapters will have IP numbers. In the above image, the Docker Virtual Network Adapter also has an IP number, which is defined in the containers as their default gateway.

The SQL Server in **Computer1** will (**if TCP is enabled**) by default listen on all network adapters, so what if we could tell the container to use **SQL Server listening on the default gateway IP address**. This way we don’t have to resolve Computer1 and we don’t have to ever leave Computer1.

# \-UpdateHosts

Since the beginning of Docker Containers for NAV and Business Central, there has been a parameter called **\-updatehosts** on **New-NavContainer**. The functionality of this is to write the **container IP address** and the **container name** in the hosts file on the host, making it possible to open the Web Client by **specifying the hostname in a browser on the host**.

Since **NavContainerHelper 0.6.4.16**, **\-updatehosts** is doing one more thing. On every restart of a container (which also executes on reboot of the computer), it will also add an entry to the **hosts file inside the container**. An entry pointing to the the **Docker Virtual Network Gateway** with the name **host.containerhelper.internal**.

This means that you can now use **host.containerhelper.internal** as the databaseServer name inside a container in order to connect to a SQL Server on the host, **and it should survive reboots, restarts and all**.

**_Please note that I write should – I have not been able to test this on all operating systems and all network configurations._**

# Install SQL Server on the host

In order to try this, you need to install/configure a few things. Here’s what I did:

1.  Install **SQL Server 2017 Developer edition** on my laptop (my docker host)
2.  Install **SSMS (SQL Server Management Studio)** on my laptop
3.  Open **SQL Server 2017 Configuration Manager** and **enable TCP/IP protocol** for SQL Server.
4.  Open **SSMS**, connect to **localhost SQL Server**, select **Properties** on **localhost**, Security and enable **SQL Server and Windows Authentication mode**.
5.  In the **Object Explorer in SSMS**, expand **Security** and **Logins**. Select **properties** on sa and **set a password** (under **General**) and **enable sa** (under **Status**).
6.  Open port **1433** for inbound traffic in the Firewall.

# Extract the Database from a Container and attach

Much like in the first blog post, you can extract the .mdf and .ldf files from a container image and attach these to the SQL Server on the host (if you do not have a database you need to use).

```
$containerName = "test"
$DatabaseFolder = "c:\databases"
$DatabaseName = "MyCronus"

if (!(Test-Path $DatabaseFolder)) {
    New-Item $DatabaseFolder -ItemType Directory | Out-Null
}

if (Test-Path (Join-Path $DatabaseFolder "$($DatabaseName).*")) {
    throw "Database $DatabaseName already exists in $databaseFolder"
}

$imageName = Get-BestBCContainerImageName -imageName "mcr.microsoft.com/businesscentral/onprem"
docker pull $imageName

$dbPath = Join-Path $env:TEMP ([Guid]::NewGuid().ToString())
Extract-FilesFromBCContainerImage -imageName $imageName -extract database -path $dbPath -force

$files = @()
Get-ChildItem -Path (Join-Path $dbPath "databases") | % {
    $DestinationFile = "{0}\{1}{2}" -f $databaseFolder, $DatabaseName, $_.Extension
    Copy-Item -Path $_.FullName -Destination $DestinationFile -Force
    $files += @("(FILENAME = N'$DestinationFile')")
}

Remove-Item -Path $dbpath -Recurse -Force

Write-Host "Attaching files as new Database $DatabaseName"
Invoke-SqlCmd -Query "CREATE DATABASE [$DatabaseName] ON $([string]::Join(", ",$Files)) FOR ATTACH"
```

After running this, you should be able to see your newly created database in SSMS.

# Create a container connecting to the new database

Now you can run a container and connect to that database:

```
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$dbcredentials = New-Object PSCredential -ArgumentList 'sa', $credential.Password
New-BCContainer `
    -accept_eula `
    -containerName $containerName `
    -imageName $imageName `
    -updateHosts `
    -auth UserPassword `
    -Credential $credential `
    -databaseServer 'host.containerhelper.internal' `
    -databaseInstance '' `
    -databaseName $DatabaseName `
    -databaseCredential $dbcredentials
New-NavContainerNavUser -containerName $containerName -Credential $credential -ChangePasswordAtNextLogOn:$false -PermissionSetId SUPER
```

**Note:** Docker won’t create the user in the database if you specify a foreign database connection like here, which is why the script adds the user at the end.

# Remove the container and the database

When all is done, you can remove the container, the database and the database files using this script:

```
Remove-BCContainer $containerName
Write-Host "Dropping database $DatabaseName"
Invoke-SqlCmd -Query "ALTER DATABASE [$DatabaseName] SET OFFLINE WITH ROLLBACK IMMEDIATE" 
Invoke-Sqlcmd -Query "DROP DATABASE [$DatabaseName]"
Write-Host "Removing Database files $($databaseFolder)\$($DatabaseName).*"
Remove-Item -Path (Join-Path $DatabaseFolder "$($DatabaseName).*") -Force
```

# What if it doesn’t work?

Maybe you haven’t updated the NavContainerHelper?

**You need 0.6.4.16 or newer.**

This setup works on my machines and on Azure VMs, but I cannot guarantee that it works on yours. You might have other firewalls, endpoint protections, anti virus or other things installs which for one reason or the other causes this to not work and there is probably nothing I can do about that – sorry.

You can still open an issue on [http://www.github.com/microsoft/navcontainerhelper/issues](http://www.github.com/microsoft/navcontainerhelper/issues) and include the full scripts you use. Maybe somebody else can help you.

# The ARM templates

I will add this to the ARM templaes. You will be able to specify SQL Developer as database in [http://aka.ms/getbcext](http://aka.ms/getbcext) and the scripts will then install SQL Server 2017 Developer edition, configure users, firewalls and all – and of course configure the default container using the above scripts.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
