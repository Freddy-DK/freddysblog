---
layout: post
title: "NAV on Docker version 0.0.4.1"
date: 2017-12-02 06:28:25
categories: ["Docker", "Not Archived", "PowerShell"]
tags: ["Docker", "Generic Image", "NAV on Docker", "navinstall", "navstart", "Specific image"]
permalink: /2017/12/02/nav-on-docker-version-0-0-4-1/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers. This blog post reflects the old way of using NAV/BC on Docker.

Some of you might already know what lies behind this cryptic title, some of you might not care. This post describes what changed in the Generic image version 0.0.4.1, which today is the foundation of all images on the Docker hub and of course also of the generic image on the docker hub.

# Support for database credentials

The biggest visible change is the support for specifying database credentials and allow to easily setup NAV to use external SQL Server. In the end, this change was only a few lines of code, which was already described in the how-to document. Primary reason for not including these in the very first version was that back then, we didn’t have a secure way of handling credentials, meaning that credentials for your external SQL Server would be available for grabs by anyone. Today passwords are handled securely and database credentials can be transferred safely.

**Note**: You need to use databaseSecurePassword (or the navcontainerhelper) in order to safely transfer credentials.

# Ability to run NAV 2013, NAV 2013R2 and NAV 2015

After launching NAV on Docker, one of the requests we got a lot was support for older versions. As described [here](https://freddysblog.com/2017/11/29/can-i-run-nav-2015-and-earlier-on-docker/), we do not support older versions, but with 0.0.4.1 we allow you to run older versions on Docker. The reason for doing this is, to allow people to create an infrastructure which will allow developers to develop and test on newer versions like NAV 2018, even though they might be working on a customer project on NAV 2015. It also becomes much easier to test out a NAV 2015 customers solution on NAV 2017 because you can spin up any version in minutes.

So, we didn’t add this only to make it easier for you to work on NAV 2015 or earlier versions, we want you to move forward and we think this makes it easier.

# Split installation and runtime scripts

The biggest change however is clearly the refactoring of navstart.ps1. In the first versions of the generic image, navstart was a multi-purpose script, which would do the installation if needed at the same time as starting NAV. While this was fairly simple in the beginning, it soon became complicated as more and more functionality was added.

With the requirement of being able to install earlier versions than NAV 2016 it became evident, the installation scripts needed to be separated from the running scripts. The installation scripts can still be used “on-the-fly” when running the generic image, meaning that the functionality really didn’t change a lot.

In order for you to understand how the NAV on Docker image works, I have created a small walk-through of what happens when you start a NAV on Docker image. This covers both the Generic image (microsoft/dynamics-nav:generic) and the Specific images (with a specific version of NAV installed ex. microsoft/dynamics-nav:2017-cu5-dk).

The source for the NAV on Docker images is available on github under [https://github.com/microsoft/nav-docker](https://github.com/microsoft/nav-docker)

# start.ps1

When starting a NAV on Docker image, the primary entry point is c:\\run\\start.ps1 and on a high level, this is what start.ps1 does.

1.  If not NAV is installed?
    1.  If not C:\\NAVDVD exists?
        1.  Fail
    2.  Copy the content of the version folder (c:\\run\\90 for NAV 2016) to c:\\run
    3.  Launch **c:\\run\\navinstall.ps1** (or rather **c:\\run\\my\\navinstall.ps1** if it exists)
2.  Include **c:\\run\\HelperFunctions.ps1** (or rather **c:\\run\\my\\HelperFunctions.ps1** if it exists)
3.  Run **c:\\run\\navstart.ps1** (or rather **c:\\run\\my\\navstart.ps1** if it exists)
4.  Run **c:\\run\\mainloop.ps1** (or rather **c:\\run\\my\\mainloop.ps1** if it exists)

In the github repo, you will find version folders for NAV 2013 (70), NAV 2013R2 (71), NAV 2015 (80), NAV 2016 (90), NAV 2017 (100) and NAV 2018 (110).

The my folder does not exist in the image by default, the idea is, that when running the image you can share a folder from the host to the c:\\run\\my folder and override functionality as you like.

# navstart.ps1

navstart is the main runner for starting the image and it will sequence a number of other scripts, which all have a specific purpose. When sequencing these scripts, navstart will check whether the script exists in c:\\run\\my and launch that instead of the original script. This allows you to override the scripts and determine whether or not you want the base functionality or totally take over.

Two variables will be calculated in the beginning of navstart:

**$restartingInstance** will be set to true if this is a restart of the container. If this indeed is a restart, there are a number of things that can be omitted (setting up the Web Client, importing license file, creating users)

**$newPublicDnsName** will be set to true if the publicDnsName of the container has changed, either by this being the first time you run the container or if a restart caused a new PublicDnsName (happens if you use docker commit to create your own image and clone that).

The sequence of the various scripts are as follows:

1.  Include **HelperFunctions.ps1**
2.  Run **SetupVariables.ps1** to read all parameters from environment variables into PowerShell variables and doing defaulting etc. SetupVariables will also decrypt encrypted password transferred to the container and remove the encryption key if requested, leaving the secure passwords as temporary securestrings only.
3.  If the user did not accepted the EULA, fail
4.  If the image is outdated and the user did not accept to run outdated images, fail
5.  Start **SQL Server Express** if $databaseServer is localhost
6.  Start **Internet Information Server** unless you requested no WebClient and no http download site.
7.  Run **SetupDatabase.ps1** to setup the database. This will by default do nothing if this is a restart, restore a bakfile if requested or setup database credentials for external SQL Server access if databaseCredentials have been specified.
8.  If $newPublicDnsName
    1.  If using SSL
        1.  Run **SetupCertificate.ps1** to setup a self-signed certificate. Override this to use a trusted certificate.
    2.  Run SetupConfiguration.ps1 to setup the configuration of NAV (customSettings.config, port ACL’s etc.)
9.  Run **setupAddIns.ps1** only if this is not a restart.
10.  Start the NAV Service Tier
11.  Run **SetupLicense.ps1** if you are using a local SQL Server. It is assumed that license is in place if connecting to an external SQL Server. Default behavior for SetupLicense is to do nothing if this is a restart.
12.  If $newPublicDnsName is true and you didn’t skip the WebClient
     1.  Run **SetupWebClient.ps1**
     2.  Run **SetupWebClientConfiguration.ps1**
13.  If not restarting
     1.  If you didn’t skip the Http Download site
         1.  Create Web Site
         2.  Run **SetupFileShare.ps1**
     2.  Run **SetupWindowsUsers.ps1**
     3.  If using Local SQL Server (it is assumed that SQL and NAV users are in place if connecting to an external SQL Server)
         1.  Run **SetupSqlUsers.ps1**
         2.  Run **SetupNavUsers.ps1**
14.  If ClickOnce is requested (and you didn’t skip http download site)
     1.  Run **SetupClickOnce.ps1** which in its default implementation will run **SetupClickOnceDirectory.ps1** to setup the files to include.
15.  Run **AdditionalSetup.ps1** which by default is empty, but allows you to do additional setup at the very end of the setup.
16.  Write container info output
17.  Run **AdditionalOutput.ps1** which by default is empty, but allows you to write additional output to the user
18.  Write “Ready for connections!”

There you have it. It might seem complicated, but it offers extreme flexibility and you can really hook in a lot of places and change the behavior of small things if you need to.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
