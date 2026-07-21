---
layout: post
title: "NavContainerHelper – Overriding scripts in NAV containers"
date: 2018-03-20 04:12:51
categories: ["Docker", "NavContainerHelper"]
tags: ["additionaloutput", "additionalsetup", "Docker", "MainLoop", "NAV on Docker", "NavContainerHelper", "navstart", "override", "scripts", "setupaddins", "setupcertificate", "setupclickonce", "setupclickoncedirectory", "setupconfiguration", "setupdatabase", "setupfileshare", "setuplicense", "setupnavusers", "setupsqlusers", "setuptenant", "setupvariables", "setupwebclient", "setupwebconfiguration", "setupwindowsusers"]
permalink: /2018/03/20/navcontainerhelper-overriding-scripts-in-nav-containers/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

When building, running or restarting the NAV container, the c:\\run\\start.ps1 script is being run. This script will launch navstart.ps1, which will launch a number of other scripts (listed below in the order in which they are called from navstart.ps1). Each of these scripts exists in the c:\\run folder. If a folder called c:\\run\\my exists and a script with the same name is found in that folder, then **that** script will be executed **instead** of the script in c:\\run (called overriding scripts).

Overriding scripts id done by specifying the scripts you want to override in the myscripts parameter. The myScripts parameter is an array and every item can be one of the following:

-   A filename on the host. This file will be copied to the container’s c:\\run\\my folder with the same name. If the file is a .zip file, it will be extracted to the c:\\run\\my folder.
-   A secure URL to a file. This file will be downloaded to the container’s c:\\run\\my folder with the same name. If the file is a .zip file, it will be extracted to the c:\\run\\my folder.
-   A folder name on the host. The content of this folder is copied to the c:\\run\\my folder. .zip files in the folder will not be extracted.
-   A hashtable with the filename and content of files to be written in the c:\\run\\my folder.

Example (filename):

```
New-NavContainer -accept_eula `
                 -containerName test `
                 -imageName microsoft/dynamics-nav `
                 -auth NavUserPassword `
                 -myScripts @("c:\tempAdditionalOutput.ps1")
```

Example (secure url):

```
New-NavContainer -accept_eula `
                 -containerName test `
                 -imageName microsoft/dynamics-nav `
                 -auth NavUserPassword `
                 -myScripts @("https://www.dropbox.com/s/yokximlfz2vws2i/additionalOutput.ps1?dl=1")
```

Example (.zip file):

```
New-NavContainer -accept_eula `
                 -containerName test `
                 -imageName microsoft/dynamics-nav `
                 -auth NavUserPassword `
                 -myScripts @("https://www.dropbox.com/s/y3na6vxxjlg2xig/myoverrides.zip?dl=1")
```

Example (hashtable):

```
$additionalOutputScript = @"
Write-Host "--------------------"
Write-Host "| AdditionalOutput |"
Write-Host "--------------------"
"@

New-NavContainer -accept_eula `
                 -containerName test `
                 -imageName microsoft/dynamics-nav `
                 -auth NavUserPassword `
                 -myScripts @{"AdditionalOutput.ps1" = $additionalOutputScript} 
```

The output of the container should be something like:

```
...
Container IP Address: 172.19.155.150
Container Hostname  : test
Container Dns Name  : test
Web Client          : http://test/NAV/
Dev. Server         : http://test
Dev. ServerInstance : NAV
--------------------
| AdditionalOutput |
--------------------

Files:
http://test:8080/al-0.12.17720.vsix
```

The list below is all the overridable scripts in the c:\\run folder, a link to the source code and a description of their responsibility.

1.  [navstart.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/navstart.ps1) – navstart is the very first script to run and is responsible for running the following scripts.
2.  [Helperfunctions.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/HelperFunctions.ps1) – set of helper functions.
3.  [SetupVariables.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupVariables.ps1) – read environment variables and set PowerShell variables.
4.  [SetupDatabase.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupDatabase.ps1) – setup the database used for this container.
5.  [SetupCertificate.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupCertificate.ps1) – setup certificate to use.
6.  [SetupConfiguration.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupConfiguration.ps1) – setup configuration for service tier.
7.  [SetupAddIns.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupAddIns.ps1) – setup addins in service tier and roletailored client folders.
8.  [SetupLicense.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupLicense.ps1) – setup license to use.
9.  [SetupTenant.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupTenant.ps1) – setup tenant (if multitenancy).
10.  [SetupWebClient.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/110/SetupWebClient.ps1) – setup Web Client (this script is different for different versions of NAV).
11.  [SetupWebConfiguration.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupWebConfiguration.ps1) – setup Web Client configuration (default file is empty).
12.  [SetupFileShare.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupFileShare.ps1) – setup file share with certificate, vsix file and more.
13.  [SetupWindowsUsers.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupWindowsUsers.ps1) – setup Windows users.
14.  [SetupSqlUsers.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupSqlUsers.ps1) – setup SQL users.
15.  [SetupNavUsers.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupNavUsers.ps1) – setup NAV users.
16.  [SetupClickOnce.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/110/SetupClickOnce.ps1) – setup ClickOnce deployed Windows Client (this script is different for different versions of NAV).
17.  [SetupClickOnceDirectory.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/110/SetupClickOnceDirectory.ps1) – setup ClickOnce directory with the necessary files for Windows Client deployment.
18.  [AdditionalSetup.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/AdditionalSetup.ps1) – additional setup script (default file is empty).
19.  [AdditionalOutput.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/AdditionalOutput.ps1) – additional output script (default file is empty).
20.  [MainLoop.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/MainLoop.ps1) – NAV container main loop, exiting the main loop will terminate the container.

When overriding scripts, you need to determine whether or not you are going to invoke the default behavior. Some script overrides (like SetupCertificate.ps1) will typically not invoke the default script, others (like SetupConfiguration.ps1) typically will invoke the default behavior.

Insert this line in your script to invoke the default behavior of the script:

```
. (Join-Path $runPath $MyInvocation.MyCommand.Name)
```

Furthermore, within your scripts there are a number of variables you can/should use. The $restartingInstance is important to consider as this determines whether the container is restarting or starting for the first time. A number of things are NOT needed to be done again on restart.

-   _$restartingInstance_ – this variable is true when the script is being run as a result of a restart of the docker instance.

The following variables are used to indicate locations of stuff in the image:

-   _$runPath_ – this variable points to the location of the run folder (C:\\RUN)
-   _$myPath_ – this variable points to the location of my scripts (C:\\RUN\\MY)
-   _$NavDvdPath_ – this variable points to the location of the NAV DVD (C:\\NAVDVD)

The following variables are parameters, which are defined when running the image:

-   _$Auth_ – this variable is set to the NAV authentication mechanism based on the environment variable of the same name. Supported values at this time is Windows and NavUserPassword.
-   _$serviceTierFolder_ – this variable is set to the folder in which the Service Tier is installed.
-   _$WebClientFolder_ – this variable is set to the folder in which the Web Client binaries are present.
-   _$roleTailoredClientFolder_ – this variable is set to the folder in which the RoleTailored Client files are present.

Please go through the navstart.ps1 script to understand how this works and how the overridable scripts are launched.

## [navstart.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/navstart.ps1)

navstart.ps1 is the main script runner, which will invoke all the other scripts.

### Default behavior

Invoke scripts in the order mentioned above.

### Reasons to override

-   If you want to change behavior of the NAV container totally, then this can be done by specifying another navstart.ps1.

## [Helperfunctions.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/HelperFunctions.ps1)

HelperFunctions is a library of helper functions, used by the scripts.

### Default behavior

You should always invoke the default helperfunctions script.

### Reasons to override

-   Override functions in helperfunctions.

## [SetupVariables.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupVariables.ps1)

When running the NAV container image, most parameters are specified by using -e parameter=value. This will actually set the environment variable parameter to value and in the SetupVariables script, these environment variables are transferred to PowerShell variables.

### Default behavior

The script will transfer all known parameters from environment variables to PowerShell variables, and make sure that default values are correct.

### Reasons to override

-   Hardcode variables

## [SetupDatabase.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupDatabase.ps1)

The responsibility of SetupDatabase is to make sure that a database is ready for the NAV Service Tier to open. The script will not be executed if a $databaseServer and $databaseName parameter is specified as environment variables.

### Default behavior

The script will be executed when running the generic or a specific image, and it will be executed when the container is being restarted. The default implementation of the script will perform these checks:

1.  If the container is being restarted, do nothing.
2.  If an environment variable called bakfile is specified (either path+filename or http/https) that bakfile will be restored and used as the NAV Database.
3.  If an environment variables called appBacpac and tenantBacpac are specified (either path+filename or http/https) they will be restored and used as the NAV Database.
4.  If database credentials are specified, then the script will setup connection to an external SQL Server and setup key for encryption.
5.  If multitenant switch is specified, the NAV container will switch to multi-tenancy mode.

### Reasons to override

-   Place your database file on a file share on the Docker host
-   Connect to a SQL Azure Database or another SQL Server

## [SetupCertificate.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupCertificate.ps1)

The responsibility of the SetupCertificate script is to make sure that a certificate for secure communication is in place. The certificate will be used for the communication between Client and Server (if necessary) and for securing communication to the Web Client and to Web Services (unless UseSSL has been set to N).

The script will only be executed during run (not build or restart) and the script will not be executed if you run Windows Authentication unless you set UseSSL to Y and you would typically not need to call the default SetupCertificate.ps1 script from your script.

The script will need to set 3 variables, which are used by navstart.ps1 afterwards.

-   $certificateCerFile (if self signed)
-   $certificateThumbprint
-   $dnsIdentity

### Default behavior

The default script will create a self-signed certificate, and use this for securing access to NAV.

**Note**, services like PowerBI and the Office Excel Add-in will not be able to trust your self signed certificate, meaning that

### Reasons to override

-   Using a certificate issues by a trusted authority.

## [SetupConfiguration.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupConfiguration.ps1)

The SetupConfiguration script will setup the NAV Service Tier configuration file. The script also adds port reservations if the configuration is setup for SSL.

### Default behavior

Configure the NAV Service Tier with all instance specific settings. Hostname, Authentication, Database, SSL Certificate and other things, which changes per instance of the NAV container.

### Reasons to override

-   Changes needed to the settings for the NAV Service Tier (although this can also be done by specifying the CustomNavSettings environment variable in the container)

Example:

```
# Invoke default behavior
. (Join-Path $runPath $MyInvocation.MyCommand.Name)
$CustomConfigFile = Join-Path $ServiceTierFolder "CustomSettings.config"
$CustomConfig = [xml](Get-Content $CustomConfigFile)
$customConfig.SelectSingleNode("//appSettings/add[@key='MaxConcurrentCalls']").Value = "10"
$CustomConfig.Save($CustomConfigFile)
```

## [SetupAddIns.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupAddIns.ps1)

SetupAddIns must make sure that custom add-ins are available to the Service Tier and in the RoleTailored Client folder.

### Default Behavior

Copy the content of the C:\\Run\\Add-ins folder (if it exists) to the Add-ins folder under the Service Tier and the RoleTailored Client folder.

If you override this script, you should execute the default behavior before doing what you need to do. In your script you should use the $serviceTierFolder and $roleTailoredClientFolder variables to determine the location of the folders.

**Note**, you can also share a folder with Add-Ins directly to the ServiceTier Add-Ins folder and avoid copying stuff around altogether.

### Reasons to override

-   Copy Add-Ins from a network location

## [SetupLicense.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupLicense.ps1)

The responsibility of the SetupLicense script is to ensure that a license is available for the NAV Service Tier.

### Default Behavior

The default behavior of the setupLicense script does nothing during restart of the Docker instance.

Else, the default behavior will check whether the LicenseFile parameter is set (either to a path on a share or a http download location). If the licenseFile parameter is specified, this license will be used. If no licenseFile is specified, then the CRONUS Demo license is used.

In all specific NAV container images, the license is already imported. If you are running the generic image, the license will be imported.

### Reasons to override

-   If you have moved the database or you are using a different database
-   Import the license to a different location (default is NavDatabase)

## [SetupTenant.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupTenant.ps1)

This script will create a tenant database as a copy of the tenant template database.

### Default behavior

Copy the tenant template database and mount it as a new tenant.

### Reasons to override

-   Use a different tenant template.
-   Initialize tenant after mount.

## [SetupWebClient.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/110/SetupWebClient.ps1)

This script is used to setup the WebClient. This script is different for different versions of NAV.

### Default behavior

Setup the WebClient under IIS.

### Reasons to override

-   If you want to setup the WebClient as a service and not under IIS.

## [SetupWebConfiguration.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupWebConfiguration.ps1)

The responsibility of the SetupWebConfiguration is to do final configuration changes to Web config.

### Default Behavior

The default script is left empty, base Web Configuration is done in SetupWebClient.ps1.

### Reasons to override

-   Change things in the Web configuration, which isn’t supported by parameters already.

## [SetupFileShare.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupFileShare.ps1)

The SetupFileShare script needs to copy files, which you want to be available to the user to the file share folder.

### Default Behavior

-   Copy .vsix file (NAV new Development Environment add-in) if it exists to file share folder.
-   Copy self-signed certificate (if you are using SSL) to file share folder.

You should always invoke the default behavior if you override this script (unless the intention is to not have the file share).

### Reasons to override

-   Add additional files to the file share (Copy files need to $httpPath)

## [SetupWindowsUsers.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupWindowsUsers.ps1)

This script will create the user specified as a Windows user in the container in order to allow Windows authentication to work.

### Default behavior

Create the Windows user.

### Reasons to override

-   avoid creating the Windows user.

## [SetupSqlUsers.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupSqlUsers.ps1)

The SetupSqlUsers script must make sure that the necessary users are created in the SQL Server.

### Default Behavior

-   If the databaseServer is not localhost, then the default behavior does nothing, else…
-   If a password is specified, then set the SA password and enable the SA user for classic development access.
-   If you are using NavUserPassword authentication, then add the user to the SQL Database as a sysadmin.
-   If you are using windows authentication and gMSA, then add the user to the SQL Database as a sysadmin.

**Note**, using NavContainerHelper, you will not be able to avoid specifying a username and password.

If you override this script, you might or might not need to invoke the default behavior.

### Reasons to override

-   Change configurations to SQL Server

## [SetupNavUsers.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/SetupNavUsers.ps1)

The responsibility of the SetupNavUsers script is to setup users in NAV.

### Default Behavior

If the container is running _Windows Authentication_, then this script will create the current Windows User as a SUPER user in NAV. This script will also create the LocalUser if necessary you have specified username and password (i.e. if you are NOT using gMSA). If the user already exists in the database, no action is taken.

If the container is running _NavUserPassword authentication_, then this script will create a new SUPER user in NAV. If Username and Password are specified, then they are used, else a user named **admin** with a random password is created. If the user already exists in the database, no action is taken.

**Note**, using NavContainerHelper, you will not be able to avoid specifying a username and password.

If you override this script, you might or might not need to invoke the default behavior.

### Reasons to override

-   Create multiple users in NAV for demo purposes

## [SetupClickOnce.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/110/SetupClickOnce.ps1)

The SetupClickOnce script will setup a ClickOnce manifest in the download area.

### Default Behavior

Create a ClickOnce manifest of the Windows Client

### Reasons to override

-   This script is rarely overridden, but If you want to create an additional ClickOnce manifest, this is where you would do it.

## [SetupClickOnceDirectory.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/110/SetupClickOnceDirectory.ps1)

The responsibility of the SetupClickOnceDirectory script is to copy the files needed for the ClickOnce manifest from the RoleTailored Client directory to the ClickOnce ApplicationFiles directory.

### Default behavior

Copy all files needed for a standard installation, including the Add-ins folder.

If you override this script, you would probably always call the default behavior and then perform whatever changes you need to do afterwards. The location of the Windows Client binaries is given by _$roleTailoredClientFolder_ and the location to which you need to copy the files is _$ClickOnceApplicationFilesDirectory_.

### Reasons to override

-   Changes to ClientUserSettings.config
-   Copy additional files. If you need to copy additional files, invoke the default behavior and perform copy-item cmdlets like:

Example:

```
Copy-Item "$roleTailoredClientFolderNewtonsoft.Json.dll" -Destination "$ClickOnceApplicationFilesDirectory"
```

## [AdditionalSetup.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/AdditionalSetup.ps1)

This script is added to allow you to add additional setup to your Docker container, which gets run after everything else is setup. You will see, that in the scenarios, the AdditionalSetup script is frequently overridden to achieve things.

### Default Behavior

The default script is empty and does nothing. If you override this script there is no need to call the default behavior.

This script is the last script, which gets executed before the output section and the main loop.

### Reasons to override

-   If you need to perform additional setup when running the docker container

## [AdditionalOutput.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/AdditionalOutput.ps1)

This script is added to allow you to add additional output to your Docker container.

### Default Behavior

The default script is empty and does nothing.

If you override this script there is no need to call the default behavior.

### Reasons to override

If you need to output information to the user running the Docker Container, you can write stuff to the host in this script and it will be visible to the user running the container.

## [MainLoop.ps1](https://github.com/Microsoft/nav-docker/blob/master/Run/MainLoop.ps1)

The responsibility of the MainLoop script is to make sure that the container doesn’t exit. If no “message” loop is running, the container will stop running and be marked as Exited.

### Default Behavior

Default behavior of the MainLoop is, to display Application event log entries concerning Dynamics products.

If you override the MainLoop, you would rarely invoke the default behavior.

### Reasons to override

-   Avoid printing out event log entries
-   Override the MainLoop and sleep for a 100 years😊

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
