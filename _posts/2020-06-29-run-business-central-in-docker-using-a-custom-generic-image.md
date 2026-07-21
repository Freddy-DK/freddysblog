---
layout: post
title: "Run Business Central in docker using a custom generic image"
date: 2020-06-29 12:11:35
categories: ["Docker", "PowerShell"]
tags: ["Artifacts", "Docker", "Generic", "Generic Image"]
permalink: /2020/06/29/run-business-central-in-docker-using-a-custom-generic-image/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

The change to running Business Central in docker using artifacts also opens a new opportunity of running Business Central using a custom generic image.

# Why would you do that?

Most people will probably not have to create their own generic image, but if you for some reason aren’t satisfied with the internals of the generic image, maybe you want a different version of SQL inside the image, maybe you don’t want SQL inside the image etc. etc.

Remember that when you change things in the generic layer, NavContainerHelper and other tools might stop working, if they were dependent on the version number, the install location, the SQL Server Instance name etc. etc.

In this blog post, I will show you how to create a generic image, which runs with SQL Developer Edition instead of SQL Express. I will still name the instance SQLEXPRESS so that I don’t have to change anything but the installation (the DOCKERFILE).

# Clone the project

Start Visual Studio code as Administrator, open the command palette using Ctrl+Shift+P, select Git: Clone and type in this URL: **[https://github.com/microsoft/nav-docker.git](https://github.com/microsoft/nav-docker.git)**

# Build the project

Open the repository, navigate to the **generic** folder and open **build.ps1** and press **F5** (you will need the PowerShell extension installed to do this). If everything works, you should end up with something that looks like this:

![](/assets/images/2020/run-business-central-in-docker-using-a-custom-generic-image/built.png)

The image is built and you are ready to use it.

# Use your generic image

You can specify your generic image to New-BcContainer using the -useGenericImage parameter:

```
New-BcContainer -accept_eula -containerName mytest -artifactUrl (get-bcartifacturl -type onprem -country w1) -useGenericImage mygeneric:latest
```

This should start a container called mytest, running on top of your newly created generic image, but since you didn’t change anything, it will still be the same version of SQL Server inside:

```
Invoke-ScriptInBCContainer -containerName mytest -scriptblock { sqlcmd -Q "SELECT Convert(varchar(20),ServerProperty('ProductVersion')) as version,Convert(Varchar(40),Serverproperty('edition')) as edition" }

version              edition
-------------------- ----------------------------------------
14.0.1000.169        Express Edition (64-bit)

(1 rows affected)
```

# Modify the DOCKERFILE

In Visual Studio Code, open the DOCKERFILE and locate the first RUN section. You can replace that with this section:

```
RUN Add-WindowsFeature Web-Server,web-AppInit,web-Asp-Net45,web-Windows-Auth,web-Dyn-Compression,web-WebSockets; \
    Stop-Service 'W3SVC' ; \
    Set-Service 'W3SVC' -startuptype manual ; \
    Set-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name ServerPriorityTimeLimit -Value 0 -Type DWord; \
    Set-ItemProperty -Path "HKLM:\system\CurrentControlSet\control" -name ServicesPipeTimeout -Value 300000 -Type DWORD -Force; \
    Invoke-WebRequest -Uri 'https://go.microsoft.com/fwlink/?linkid=840945' -OutFile sql.exe ; \
    Invoke-WebRequest -Uri 'https://go.microsoft.com/fwlink/?linkid=840944' -OutFile sql.box ; \
    Start-Process -Wait -FilePath .\sql.exe -ArgumentList /qs, /x:setup ; \
    .\setup\setup.exe /q /ACTION=Install /INSTANCENAME=SQLEXPRESS /FEATURES=SQLEngine /UPDATEENABLED=0 /SQLSVCACCOUNT='NT AUTHORITY\System' /SQLSYSADMINACCOUNTS='BUILTIN\ADMINISTRATORS' /TCPENABLED=1 /NPENABLED=0 /IACCEPTSQLSERVERLICENSETERMS ; \
    While (!(get-service 'MSSQL$SQLEXPRESS' -ErrorAction SilentlyContinue)) { Start-Sleep -Seconds 5 } ; \
    Stop-Service 'MSSQL$SQLEXPRESS' ; \
    Set-itemproperty -path 'HKLM:\software\microsoft\microsoft sql server\mssql14.SQLEXPRESS\mssqlserver\supersocketnetlib\tcp\ipall' -name tcpdynamicports -value '' ; \
    Set-itemproperty -path 'HKLM:\software\microsoft\microsoft sql server\mssql14.SQLEXPRESS\mssqlserver\supersocketnetlib\tcp\ipall' -name tcpport -value 1433 ; \
    Set-itemproperty -path 'HKLM:\software\microsoft\microsoft sql server\mssql14.SQLEXPRESS\mssqlserver\' -name LoginMode -value 2 ; \
    Set-Service 'MSSQL$SQLEXPRESS' -startuptype manual ; \
    Set-Service 'SQLTELEMETRY$SQLEXPRESS' -startuptype manual ; \
    Set-Service 'SQLWriter' -startuptype manual ; \
    Set-Service 'SQLBrowser' -startuptype manual ; \
    Remove-Item -Recurse -Force sql.exe, sql.box, setup
```

Save and investigate the differences using the source control feature in Visual Studio Code:

![](/assets/images/2020/run-business-central-in-docker-using-a-custom-generic-image/dockerfile.png)

Not a lot, but it downloads, executes and cleans up a different version. The actual invoking of the installation is similar since we use the same instance name.

Re-run the build.ps1 script and you should again get a new generic image.

# Use your new generic image

Again, specify your generic image to New-BcContainer using the -useGenericImage parameter:

```
New-BcContainer -accept_eula -containerName mytest -artifactUrl (get-bcartifacturl -type onprem -country w1) -useGenericImage mygeneric:latest
```

The new container should now be using SQL Server Developer Edition instead. You cannot really see the difference as the instance is still called SQLEXPRESS, but if you query the ServerProperty again:

```
Invoke-ScriptInBCContainer -containerName mytest -scriptblock { sqlcmd -Q "SELECT Convert(varchar(20),ServerProperty('ProductVersion')) as version,Convert(Varchar(40),Serverproperty('edition')) as edition" }

version              edition
-------------------- ----------------------------------------
14.0.1000.169        Developer Edition (64-bit)

(1 rows affected)
```

# Other changes

You can also modify scripts or add other software to your generic image, but please note that as mentioned in the start – if you change the structure or functionality of things – you might end up breaking NavContainerHelper and I won’t be able to help you fix that. Please spend time investigating the scripts and understand how things works before trying to change it, thanks.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
