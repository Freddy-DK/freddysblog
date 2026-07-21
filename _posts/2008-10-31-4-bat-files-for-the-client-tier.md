---
layout: post
title: "4 BAT files for the Client Tier"
date: 2008-10-31 17:31:40
categories: ["Archive"]
tags: ["Client Tier", "Multiple", "NAV 2009", "Service", "Service Tier"]
permalink: /2008/10/31/4-bat-files-for-the-client-tier/
---

If you haven’t done so, please read the post about Multiple Service Tiers before reading this:-)

In my previous post (Multiple Service Tier) I promised to talk about what to do on the Client Tier if you have installed a number of services on a Service Tier computer that all are set to start manually.

Do you really have to go to the Service Tier machine in order to start the services or is there a simpler way?

As you might have guessed – there is (else you wouldn’t be reading this post)

The SC command can actually start services on a different box with the syntax:

SC \\machine start servicename

Assuming that we are using the CreateService.bat to create our Service Tier we know what the service name is based on the instance name and we should be able to create 4 BAT files, which enables us to:

1\. Start Service Tiers  
2\. Stop Service Tiers  
3\. Restart Service Tiers  
4\. Start the Role Tailored Client using a specific Service Tier (and start the Service Tier if necessary)

The four scripts really makes things easier when dealing with multiple Service Tiers – and here they go…

### StartService.bat

@ECHO OFF  
IF “%1” == “” GOTO usage  
SETLOCAL  
SET SERVICETIER=%2  
IF NOT “%SERVICETIER%” == “” SET SERVICETIER=\\%SERVICETIER%  
SET NAVPATH=%~dp0  
SC %SERVICETIER% query MicrosoftDynamicsNavServer$%1 > nul  
IF ERRORLEVEL 1 GOTO :eof  
SC %SERVICETIER% query MicrosoftDynamicsNavServer$%1 | FINDSTR “RUNNING”  
IF NOT ERRORLEVEL 1 GOTO :eof  
SC %SERVICETIER% start MicrosoftDynamicsNavServer$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
SC %SERVICETIER% start MicrosoftDynamicsNavWS$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
GOTO :eof  
:usage  
ECHO.  
ECHO Usage:  
ECHO.  
ECHO startservice instancename \[servicetier\]  
ECHO.

SETLOCAL means that the changes we do to environment variables here are not reflected outside this .BAT file.  
The .BAT files checks whether the service exists and whether it already has been started – if that is the case, there is no real reason for starting it.  
Note BTW that you will need the Sleep.bat file described in the Multiple Service Tiers post here as well.

### StopService.bat

@ECHO OFF  
IF “%1” == “” GOTO usage  
SETLOCAL  
SET SERVICETIER=%2  
IF NOT “%SERVICETIER%” == “” SET SERVICETIER=\\%SERVICETIER%  
SET NAVPATH=%~dp0  
SC %SERVICETIER% query MicrosoftDynamicsNavServer$%1 > nul  
IF ERRORLEVEL 1 GOTO :eof  
SC %SERVICETIER% query MicrosoftDynamicsNavServer$%1 | FINDSTR “STOPPED”  
IF NOT ERRORLEVEL 1 GOTO :eof  
SC %SERVICETIER% stop MicrosoftDynamicsNavWS$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
SC %SERVICETIER% stop MicrosoftDynamicsNavServer$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
GOTO :eof  
:usage  
ECHO.  
ECHO Usage:  
ECHO.  
ECHO stopservice instancename \[servicetier\]  
ECHO.

Kind of the same story as StartService.bat – if the Service is stopped, there is no real reason to try and stop it.

### RestartService.bat

You probably guessed by now what this .BAT file is doing – so no reason in explaining.

@ECHO OFF  
IF “%1” == “” GOTO usage  
SETLOCAL  
SET SERVICETIER=%2  
IF NOT “%SERVICETIER%” == “” SET SERVICETIER=\\%SERVICETIER%  
SET NAVPATH=%~dp0  
SC %SERVICETIER% query MicrosoftDynamicsNavServer$%1 > nul  
IF ERRORLEVEL 1 GOTO :eof  
SC %SERVICETIER% query MicrosoftDynamicsNavServer$%1 | FINDSTR “STOPPED”  
IF NOT ERRORLEVEL 1 GOTO dontstop  
SC %SERVICETIER% stop MicrosoftDynamicsNavWS$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
SC %SERVICETIER% stop MicrosoftDynamicsNavServer$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
:dontstop  
SC %SERVICETIER% start MicrosoftDynamicsNavServer$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
SC %SERVICETIER% start MicrosoftDynamicsNavWS$%1  
CALL “%NAVPATH%SLEEP.BAT” 3  
GOTO :eof  
:usage  
ECHO.  
ECHO Usage:  
ECHO.  
ECHO restartservice instancename \[servicetier\]  
ECHO.

The .BAT file more or less just does a StopService followed by a StartService.

### RTC.bat

Now this is the fun stuff – this .BAT file starts the Role Tailored Client connecting to a specific Service Tier – but first it starts the Service Tier in question if it wasn’t already started.

@ECHO OFF  
IF “%1” == “” GOTO usage  
SETLOCAL  
SET COMPANY=%3  
REM IF ‘%COMPANY%’ == ” SET COMPANY=”CRONUS International Ltd.”  
IF ‘%COMPANY%’ == ” SET COMPANY=”CRONUS USA, Inc.”  
ECHO.%COMPANY%  
SET COMPANY=%COMPANY:”=%  
SET COMPANY=%COMPANY:,=#%  
:again  
SET BEFORE=%COMPANY%  
FOR /F “tokens=1\* delims= ” %%A IN (‘ECHO.%COMPANY%’) DO (  
IF NOT “%%B” == “” SET COMPANY=%%A%%20%%B  
)  
IF NOT “%BEFORE%” == “%COMPANY%” GOTO again  
SET COMPANY=%COMPANY:#=,%  
SET MACHINE=%2  
SET SERVICETIER=%MACHINE%  
IF “%SERVICETIER%” == “” SET SERVICETIER=localhost  
CALL STARTSERVICE.BAT %1 %SERVICETIER%  
START dynamicsnav://%SERVICETIER%/%1/%COMPANY%/  
GOTO :eof  
:usage  
ECHO.  
ECHO Usage:  
ECHO.  
ECHO RTC instancename \[servicetier\] \[“Company”\]  
ECHO.

As you can see in the usage section, RTC can be started specifying the Service Tier instance name, the Service Tier machine and the Company name.

Launching a Role Tailored Client pointing to the default installed Service Tier would be:

C:\\Prog…60>RTC DynamicsNAV localhost “CRONUS International Ltd.”

or just

C:\\Prog…60>RTC DynamicsNAV

because localhost and CRONUS are the default values for the 2nd and 3rd parameters. No real reason for that except for the fact that this has been working for me.

Note, that the FOR loop in the .BAT file replaces spaces in the COMPANY variable with %20. Another thing to notice is, that I replace commas with # before the loop (since commas will change the loop) and replace them back afterwards, meaning that you cannot have # in your Company names. You also cannot have ” in your company name.

If you have other special characters or if you do have # in your company names you would have to change the name mangling on the COMPANY name.

I use this .BAT file for a number of shortcuts on my desktop to launch Role Tailored Clients to specific Service Tiers and on my Service Tier box I run a command every night shutting down all my service Tiers:

for /f %D in (‘dir /ad/b’) do ( CALL STOPSERVICE.BAT %D )

The .BAT files can of course also be used on the Service Tier (except for the RTC one) for stopping and starting services – but they are a big help on the Client Tier – the way I am running things at least.

Once again – the people who has read all the way to the end will be able to download a ZIP file containing the .BAT files from [http://www.freddy.dk/4ForTheClientTier.zip](http://www.freddy.dk/4ForTheClientTier.zip) this ZIP file also contains a .BAT file called StopServices.bat (which actually only runs on the Service Tier = Stops all Running NAV Service Tiers).

I promise this is my last post with .BAT files:-)

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
