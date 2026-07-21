---
layout: post
title: "Multiple Service Tiers"
date: 2008-10-29 08:00:00
categories: ["Archive"]
tags: ["Multiple", "NAV 2009", "Service", "Service Tier", "Web Services"]
permalink: /2008/10/29/multiple-service-tiers/
---

**NOTE – there is an updated post reg. Multiple Service Tiers in NAV 2009 SP1** [here](/2009/08/05/multiple-service-tiers-sp1/)**.**

If you haven’t done so, please read the post about the Service Tier before reading this:-)

A very typical scenario with both partners and customers is to have more than one database. This can be because you have a development database and a production database – or it could be the partner having a copy of all customer databases locally for troubleshooting.

You could of course install the Service Tier locally on all computers – and then change the CustomSettings.config to point to a new database every time you need to logon, but that doesn’t really sound like something we want people to do.

The setup we would like to have is:

-   One or more SQL Server boxes with a bunch of databases on
-   One or more Service Tier boxes with at least one Service Tier pr. database
-   The Client installed locally on all machines being able to connect to these Service Tiers.

This post will go into detail about how to accomplish this.

### The Simple Story!

To make a long story short – adding a Service Tier isn’t any harder than copying the Service directory to another directory (maybe called test) and then registering a new service based on the executable in that folder using the SC command:

SC CREATE testServiceTier binpath= “C:\\Program Files\\Microsoft Dynamics NAV\\60\\test\\Microsoft.Dynamics.Nav.Server.exe” start= auto obj= “NT AuthorityNetworkService”

This would actually work if you change the CustomSettings.config to use a different port than 7046 – so why write a big post about it?

### And why it isn’t that simple after all!

You typically want to use the same port for all your Service Tiers – allowing you to distinguish them on the instance name, and since the default Service Tier doesn’t have a dependency on NetTcpPortSharing – you cannot just start adding new ones.

You want a consistent naming algorithm for your Service Tiers, and you want to make sure, that if you create a new Service Tier, it doesn’t inherit settings from one of the other Service Tiers by coincidence.

And last but not least, you often want to create a Web Service listener to sit next to your Service Tier (that is of course if you intend to use Web Services).

So – I created a bunch of .BAT files which would do the job for me. Feel free to look at the .BAT files, copy them, use them, modify them, but I do encourage you to send any improvements of the .BAT files to me, so that I can make them available to the community.

Note that I am NOT a .BAT file expert – but I did learn a LOT by creating these .BAT files.

The first 3 .BAT files I created are called:

CreateService.bat, DeleteService.bat and RecreateOriginalService.bat

I think the names speaks for themselves. These .BAT files then have dependencies on another .BAT file, a .VBS script and a new CustomSettings.template – all of these will be included in this post (I hope you are not in a hurry)

The .BAT files needs to be placed in the NAV installation directory (which typically would be **C:\\Program Files\\Microsoft Dynamics NAV\\60**) – and the very first thing you want to do, is to run the RecreateOriginalService.bat.

### RecreateOriginalService.bat

```
@ECHO OFF
IF NOT "%1" == "" GOTO usage
SET NAVPATH=%~dp0
IF EXIST "%NAVPATH%serviceMicrosoft.Dynamics.Nav.Server.exe" GOTO NavPathOK
ECHO.
ECHO Unable to locate installation service directory
ECHO.
ECHO %NAVPATH%service
ECHO.
ECHO Maybe you already ran recreateoriginalservice.bat
goto :eof
:NavPathOK
IF NOT EXIST "%NAVPATH%service.orgMicrosoft.Dynamics.Nav.Server.exe" GOTO orgok
ECHO.
ECHO Directory already exists
ECHO.
ECHO %NAVPATH%service.org
ECHO.
ECHO Maybe you already ran recreateoriginalservice.bat
GOTO :eof
:orgok
C:
CD "%NAVPATH%"
SC stop MicrosoftDynamicsNavWS
CALL SLEEP.BAT 3
SC stop MicrosoftDynamicsNavServer
CALL SLEEP.BAT 3
SC delete MicrosoftDynamicsNavWS
SC delete MicrosoftDynamicsNavServer
RENAME Service Service.org
CALL createservice DynamicsNAV dummy dummy auto
COPY /Y customsettings.template service.orgcustomsettings.config
GOTO :eof
:usage
ECHO.
ECHO Usage:
ECHO.
ECHO recreateoriginalservice.bat
ECHO.
```

A couple of comments to the  “source”:

-   %~dp0 returns the directory in which the .BAT file is placed (with trailing backslash C:\\Program Files\\Microsoft Dynamics NAV\\60)
-   SLEEP.BAT is a small .BAT file which sleeps in a number of seconds (approx.)
-   SC stop – tries to stop a Service (the Service is set in STOP\_PENDING mode)
-   SC delete – tries to delete a Service (note that if the Service isn’t stopped it will put the Service into a DELETE\_PENDING mode which then sometimes requires a server reboot).
-   CustomSettings.template is the original CustomSettings.Config with a few modifications.
-   CreateService.bat is called with two dummy parameters – look below for further explanation

The way this .BAT file works is, that it removes the default installed Service Tier, Renames the Service Directory to Service.org and adds the original Service Tier again (using the CustomSettings.config that already was in the Service Directory). After this, it copies in a new CustomSettings.config template with specific fields that later can be auto-replaced by the CreateService.bat. CreateService will create a Service in a directory called the same as the instance name – so after running RecreateOriginalService.bat you will find 2 new directories in the NAV path: DynamicsNAV and Service.org instead of the original Service directory.

RecreateOriginalService.bat checks whether it has ran already – so please do not create a Service Tier called Service – you can probably guess why by looking at the .bat file. Also you cannot create services called Classic, Database, RoleTailored Client or OutlookAddin – but who wants to do that anyway.

### Sleep.bat

As you saw, RecreateOriginalService.bat uses a .BAT file called SLEEP.BAT. The main purpose of this .BAT file is to wait for a number of seconds – and there really isn’t any command line tool, which works in all versions of Windows that can do this – so I made this one

```
@ping 127.0.0.1 -n 2 -w 1000 > nul
@ping 127.0.0.1 -n %1% -w 1000 > nul
```

Works fine – but is kind of strange to look at (that is why it got its own .BAT file).  
Vista, Windows Server 2003 and 2008 has a command called TIMEOUT, but that doesn’t work on XP.

### CustomSettings.template

The CustomSettings.template is a copy of the original CustomSettings.config with 3 changes:

replacing the original DatabaseServer, DatabaseName and ServerInstance with three “variables”.

When CreateService.bat is called it will replace these variables with values given on the command line, this way you can create a Service Tier without having to edit the config file afterwards. (very useful when doing testing on multiple Service Tiers)

### CreateService.bat

`Now this is the fun stuff…`

```
@ECHO OFF
IF "%1" == "" GOTO usage
SET SERVICE=%1
SET DBSERVER=%2
SET DATABASE=%3
SET START=%4
SET WHICH=%5
IF "%START%" == "" SET START=demand
IF "%START%" == "auto" goto startok
IF "%START%" == "demand" goto startok
IF "%START%" == "disabled" goto startok
ECHO.
ECHO Illegal value for 4th parameter
GOTO usage
:startok
IF "%WHICH%" == "" SET WHICH=both
IF "%WHICH%" == "both" goto whichok
IF "%WHICH%" == "servicetier" goto whichok
IF "%WHICH%" == "ws" goto whichok
ECHO.
ECHO Illegal value for 5th parameter
GOTO usage
:whichok
SET type=own
IF "%WHICH%" == "both" SET type=share
SET NAVPATH=%~dp0
IF EXIST "%NAVPATH%service.orgMicrosoft.Dynamics.Nav.Server.exe" GOTO NavPathOK
ECHO.
ECHO Unable to locate original Service directory
ECHO.
ECHO in %NAVPATH%service.org
ECHO.
ECHO Maybe you need to run recreateoriginalservice.bat
goto :eof
:NavPathOk
IF EXIST "%NAVPATH%%SERVICE%Microsoft.Dynamics.Nav.Server.exe" GOTO serviceexists
C:
CD "%NAVPATH%"
MKDIR "%SERVICE%"
IF ERRORLEVEL 1 GOTO nodir
XCOPY service.org %SERVICE% /s/e
SET SERVICEDIR=%NAVPATH%%SERVICE%
replacestringinfile.vbs #INSTANCE# %SERVICE% "%SERVICEDIR%customsettings.config"
IF '%DBSERVER%' == " GOTO editconfig
replacestringinfile.vbs #DBSERVER# %DBSERVER% "%SERVICEDIR%customsettings.config"
IF '%DATABASE%' == " GOTO editconfig
replacestringinfile.vbs #DATABASE# %DATABASE% "%SERVICEDIR%customsettings.config"
GOTO configdone
:editconfig
NOTEPAD %SERVICEDIR%customsettings.config
:configdone
SC CONFIG NetTcpPortSharing start= demand
SET DEP=
if "%WHICH%" == "ws" goto onlyws
SC CREATE MicrosoftDynamicsNavServer$%SERVICE% binpath= "%SERVICEDIR%Microsoft.Dynamics.Nav.Server.exe $%SERVICE%" DisplayName= "NAV Server %SERVICE%" type= %type% start= %START% obj= "NT AuthorityNetworkService" depend= NetTcpPortSharing
SET DEP=/MicrosoftDynamicsNavServer$%SERVICE%
if "%WHICH%" == "servicetier" goto notws
:onlyws
SC CREATE MicrosoftDynamicsNavWS$%SERVICE% binpath= "%SERVICEDIR%Microsoft.Dynamics.Nav.Server.exe $%SERVICE%" DisplayName= "NAV Server %SERVICE% WS" type= %type% start= %START% obj= "NT AuthorityNetworkService" depend= HTTP/NetTcpPortSharing%DEP%
:notws
IF "%START%" == "demand" GOTO :eof
IF "%START%" == "disabled" GOTO :eof
if "%WHICH%" == "ws" goto startws
SC START MicrosoftDynamicsNavServer$%SERVICE%
if "%WHICH%" == "servicetier" goto :eof
:startws
SC START MicrosoftDynamicsNavWS$%SERVICE%
goto :eof
:serviceexists
ECHO.
ECHO Service already exists
ECHO.
GOTO :eof
:nodir
ECHO.
ECHO Could not create service directory
ECHO.
GOTO :eof
:usage
ECHO.
ECHO Usage:
ECHO.
ECHO CreateService servicetiername [databaseserver] ["databasename"] [demand^|auto^|disabled] [both^|servicetier^|ws]
ECHO.
ECHO.
```

As you can see in the usage section, you can start the .BAT file with 5 parameters – but only the first is mandatory.

The first parameter is the Service Tier name, and this becomes the name of the directory which holds the executable and the configuration file for this Service Tier (therefore – that one cannot be defaulted)

You should think that host and databasename are mandatory, but if you look in the .bat file it will actually start up notepad and ask you to complete the config file if you don’t specify these parameters (this is the reason why RecreateOriginalService.bat calls CreateService with DynamicsNAV dummy dummy – we don’t want notepad – and remember the original config file was still there – so no replacements are made).

CreateService.bat can create both Service Tiers and Web Service listeners and default is to create both (using a shared process). It can set the services to auto start, demand start or to be disabled by default.

The Services created by CreateService.bat are called **MicrosoftDynamicsNavServer$** and **MicrosoftDynamicsNavWS$** and the Service description is **Nav Server** and **Nav Server WS** to make sure that they are listed underneath each other in the services list.

The Web Service listener is created with a dependency to the Service Tier (if you create both), so that when restarting the Service Tier – it automatically restarts the Web Service Listener as well – and both services are created with a dependency to NetTcpPortSharing.

I use the replacestringinfile VB Script (can be found later in this post) to replace the template variables in the config file with the values specifies on the command line.

I guess the best way of describing the functionality of CreateService.bat is to give a bunch of examples – that you can validate against the above source.

`C:\Pro….60>CreateService.bat test`

Creates a Service Tier and a Web Service listener with the instance name test and opens Notepad to allow you to specify databaseserver and database name. Both Services are set to start manually and they share one process.

`C:\Pro….60>CreateService.bat test localhost "Demo Database NAV (6-0)"`

Creates a Service Tier and a Web Service listener with the instance name test, pointing to the demo database on localhost. Both Services are set to start manually and they share one process.

`C:\Pro….60>CreateService.bat test localhost "Demo Database NAV (6-0)" auto servicetier`

Creates a Service Tier with the instance name test, pointing to the demo database on localhost. The Service Tier has its own process and starts automatically.

`C:\Pro….60>CreateService.bat test localhost "Demo Database NAV (6-0)" demand ws`

Creates a Web Service listener with the instance name test, pointing to the demo database on localhost. The Service Tier has its own process and is set to start manually.

```
C:\Pro….60>CreateService.bat test mydbserver "Demo Database NAV (6-0)" auto servicetier
```

`C:\Pro….60>CreateService.bat test mydbserver "Demo Database NAV (6-0)" auto ws`

Creates a Service Tier and a Web Service listener with the instance name test, pointing to the demo database on mydbserver. Both Services are set to start automatically and they each have their own process.

`for /L %p in (1,1,50) DO ( createservice.bat test%p localhost "Demo Database NAV (6-0)" )`

Creates 50 Service Tiers and Web Service listeners pointing to the demo database on localhost. All pairs share a process and all are set to demand load. Yes, I know that I am probably the only one in this world who would do something like this – but I just wanted to see how many Service Tiers I could install.

The result of that investigation is “unlimited” – I didn’t run into any barrier (of course there is a barrier with memory and disk space) by just installing a huge amount of Service Tiers that are set to start manually.

The picture is totally different if I set the Service Tiers to auto start – approx. 50 started Service Tiers managed to eat my available memory and my machine declined to start more services.

### DeleteService.bat

DeleteService really isn’t that bad. The majority of work here is to make sure that it is a service tier before killing the service, deleting it and removing the directory structure of the service without asking for permission. Should be safe though…

```
@ECHO OFF
IF "%1" == "" GOTO usage
SET NAVPATH=%~dp0
IF EXIST "%NAVPATH%service.orgMicrosoft.Dynamics.Nav.Server.exe" GOTO NavPathOK
ECHO.
ECHO Unable to locate original Service directory
ECHO.
ECHO in %NAVPATH%service.org
ECHO.
ECHO Maybe you need to run recreateoriginalservice.bat
goto :eof
:NavPathOk
C:
CD "%NAVPATH%"
IF EXIST "%1Microsoft.Dynamics.Nav.Server.exe" GOTO serviceexists
ECHO.
ECHO Service doesn't exist
GOTO usage
:serviceexists
SC query MicrosoftDynamicsNavServer$%1 | FINDSTR "STOPPED"
IF NOT ERRORLEVEL 1 GOTO dontstop
SC stop MicrosoftDynamicsNavWS$%1
CALL SLEEP.BAT 3
SC stop MicrosoftDynamicsNavServer$%1
CALL SLEEP.BAT 3
:dontstop
SC delete MicrosoftDynamicsNavWS$%1
SC delete MicrosoftDynamicsNavServer$%1
rd %1 /S /Q
GOTO :eof
:usage
ECHO.
ECHO Usage:
ECHO.
ECHO DeleteService servicename
ECHO.
```

A couple of comments to the source:

-   rd %1 /S /Q – removes a directory structure without asking for permission
-   SC query MicrosoftDynamicsNavServer$%1 \| FINDSTR “STOPPED” will check whether the Service Tier is stopped
-   DeleteService does absolutely nothing to the database – it only unhooks a service and removes the directory in which it was installed.

BTW – if you tried to create the 50 Service Tiers with CreateService – you can delete them using:

`for /L %p in (1,1,50) DO ( deleteservice.bat test%p )`

### ReplaceStringInFile.vbs

As you saw, CreateService needs to replace a “variable” in a file with a value – like #DBSERVER# -> localhost etc. and there is no command line tool to do that. But fortunately we have the Internet – and I found a nice VBScript on [http://www.motobit.com/tips/detpg\_replfile/](http://www.motobit.com/tips/detpg_replfile/ "http://www.motobit.com/tips/detpg_replfile/") which does exactly what I want:

```
Dim FileName, Find, ReplaceWith, FileContents, dFileContents
Find         = WScript.Arguments(0)
ReplaceWith  = WScript.Arguments(1)
FileName     = WScript.Arguments(2)

'Read source text file
FileContents = GetFile(FileName)

'replace all string In the source file
dFileContents = replace(FileContents, Find, ReplaceWith, 1, -1, 1)

'Compare source And result
if dFileContents <> FileContents Then
  'write result If different
  WriteFile FileName, dFileContents

  Wscript.Echo "Replace done."
  If Len(ReplaceWith) <> Len(Find) Then 'Can we count n of replacements?
    Wscript.Echo _
    ( (Len(dFileContents) - Len(FileContents)) / (Len(ReplaceWith)-Len(Find)) ) & _
    " replacements."
  End If
Else
  Wscript.Echo "Searched string Not In the source file"
End If

'Read text file
function GetFile(FileName)
  If FileName<>"" Then
    Dim FS, FileStream
    Set FS = CreateObject("Scripting.FileSystemObject")
      on error resume Next
      Set FileStream = FS.OpenTextFile(FileName)
      GetFile = FileStream.ReadAll
  End If
End Function

'Write string As a text file.
function WriteFile(FileName, Contents)
  Dim OutStream, FS

  on error resume Next
  Set FS = CreateObject("Scripting.FileSystemObject")
    Set OutStream = FS.OpenTextFile(FileName, 2, True)
    OutStream.Write Contents
End Function
```

Please go to the Web Site and rate the article if you use the script – I gave it 5 stars:-)

### That’s all good – but how do I populate the combo box with Service Tiers in the Role Tailored Client?

When you try to select a new service tier in the Role Tailored Client, you choose Select Server in the Microsoft Dynamics NAV menu:

[![image](/assets/images/2008/multiple-service-tiers/image_thumb_1.png)](/assets/images/2008/multiple-service-tiers/image_4.png)

This pops up the Select Server and Company dialog:

[![image](/assets/images/2008/multiple-service-tiers/image_thumb_2.png)](/assets/images/2008/multiple-service-tiers/image_6.png)

But you will notice that the drop down with server names is empty and will only get populated every time you successfully connect to a Service Tier.

If you want to populate this list, you will need to alter the Client Configuration file, which is stored locally on each Client – the path of the file is:

**C:\\Documents and Settings\\Local Settings\\Application Data\\Microsoft\\Microsoft Dynamics NAV** on my Windows XP and my Windows 2003 Server box and

**C:\\Users\\AppData\\Local\\Microsoft\\Microsoft Dynamics NAV** on my Vista box.

the config file is called ClientUserSettings.config and the key you want to alter is called UrlHistory. If you modify the key to be:

You will now have these two selections in the dropdown.

### That’s it for now

Having these .BAT files in place will allow you to manage your Service Tiers easier.

Please remember that these .BAT files are just listed here as examples and there is absolutely no guarantee that they will work for the purpose you want them to. That is however the nice thing about .BAT files – they can be modified using notepad.

Oh yes – and the price for reading this far is, that you get to download the .BAT files from a ZIP file here [http://www.freddy.dk/MultipleServiceTiers.zip](http://www.freddy.dk/MultipleServiceTiers.zip "http://www.freddy.dk/MultipleServiceTiers.zip")

I am also working on creating a couple of .BAT files, which should be placed on the Client to allow remote starting of Service Tiers along with startup of the Role Tailored Client (now where we are creating Services as manual start) – more on this in a later post.

I also think that it would be beneficial for people to get info about challenges for installing a Database Server and a Service Tier in a 3T environment, I do think this is described in the Documentation for NAV.

Enjoy

_Freddy Kristiansen_

_PM Architect_

Microsoft Dynamics NAV
