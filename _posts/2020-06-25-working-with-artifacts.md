---
layout: post
title: "Working with artifacts"
date: 2020-06-25 22:12:14
categories: ["Docker", "PowerShell"]
tags: ["Artifacts", "Docker", "Download-Artifacts", "Get-BcArtifactUrl", "Get-NavArtifactUrl", "New-BcContainer", "New-BcImage"]
permalink: /2020/06/25/working-with-artifacts/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

As a follow up to [this blog post](/2020/06/25/changing-the-way-you-run-business-central-in-docker/) (make sure you have read this first), this blog post will share some details about working with artifacts. How do you find them, what do they contain and what happens when you try to use them. It is primarily a list of the new functionality in NavContainerHelper for working with artifacts.

I will also provide a blog post later, which describes how to work with artifacts when using docker run (raw docker commands)

# Get-BcArtifactUrl

Originally submitted by Waldo and slightly modified to fit the needs of everybody (hopefully). The parameters are:

-   **storageAccount** (bcinsider or **bcartifacts**). The storage account determines where to get the artifacts. bcartifacts is the storage account, which contains all public artifacts with Business Central and NAV.
-   **type** (onprem or **sandbox**) determines whether to get onprem artifacts (shipped builds) or sandbox artifacts (Saas builds). If you are looking for a build, which matches your saas version, you need to use sandbox.
-   **version** (default is blank = all versions) is the version you are filtering for. You can specify a full version number or a partial version number. The function will try to match your request.
-   **country** (default is blank = all countries) is the localization you are filtering for. Onprem countries follows onprem releases (na for north america), sandbox countries follows saas countries (us, ca and mx)._Note that in 0.7.0.7 there is a bug, which returns the platform as a country. The platform cannot be used as a country:-(_
-   **select** (all, closest, secondToLastMajor or **latest**) determines which artifact urls you get.
    -   **latest** (which is the default) will return the latest image in the list
    -   **all** will return an array of all artifact urls matching the filter
    -   **closest** requires you to supply a full version number and the function will return the closest version (where Major and Minor version MUST match).
    -   **secondToLastMajor** is used in CI/CD scenarios to identify next minor release.
-   **sasToken** is the shared access signature token to use when requesting artifacts from a secured storage account (like bcinsider).

try to run this PowerShell snippet and investigate the return values.

Write-Host -ForegroundColor Yellow "Get US sandbox artifact url for current version (Latest)"
Get-BCArtifactUrl -country "us"

Write-Host -ForegroundColor Yellow "Get all US sandbox artifact urls"
Get-BCArtifactUrl -country "us" -select All

Write-Host -ForegroundColor Yellow "Get US sandbox artifact url for a version closest to 16.2.13509.13700"
Get-BCArtifactUrl -country "us" -version "16.2.13509.13700" -select Closest

Write-Host -ForegroundColor Yellow "Get latest 16.1 US sandbox artifact url"
Get-BCArtifactUrl -country "us" -version "16.1"

Write-Host -ForegroundColor Yellow "Get latest 15.x US sandbox artifact url"
Get-BCArtifactUrl -country "us" -version "15"

Write-Host -ForegroundColor Yellow "Get all Danish NAV and Business Central artifact urls"
Get-BCArtifactUrl -type OnPrem -country "dk" -select All

With Business Central, we do not use CU numbers, we use version numbers like 14.7, 15.4 and 16.2 and with this function you can easily find the versions available with a major, minor version.

# Get-NavArtifactUrl

For people in need of a NAV 2016 – NAV 2018 artifact, these are also available. You can get NAV 2018 artifacts using Get-BcArtifactUrl with version 11.0 and type onprem – but all NAV 2018 artifacts with be versioned 11.0.<build>.0 and if you can remember the build number you will probably just use Get-BcArtifactUrl.

If you however need an artifact url for a specify CU, you can use Get-NavArtifactUrl, which has 4 parameters:

-   **nav** (2016, 2017 or **2018**) specifies which NAV version you want to find artifacts for.
-   **cu** (can be rtm, cu1, cu2, cu3… or 0, 1, 2, 3) determines the cumulative update you need
-   **country** (the localization you need, **w1** is default).
-   **select** (all, closest, secondToLastMajor or **latest**) determines which artifact urls you get – meaning of this parameter is the same as in **Get-BcArtifactUrl**.

a couple of examples in how to use Get-NavArtifactUrl:

Write-Host -ForegroundColor Yellow "Get all NAV 2017 DK artifact Urls"
(Get-NavArtifactUrl -nav 2017 -country 'dk' -select all).count

Write-Host -ForegroundColor Yellow "Get latest NA onprem artifact Url"
Get-BCArtifactUrl -country "na" -type OnPrem

having the aritfact url, you can try to download it.

# Download-Artifacts

Download-Artifacts is a function, which does exactly what the name describes: Downloads the artifacts pointed out by an artifact url.

The parameters are:

-   **artifactUrl** is the artifact url returned by Get-BcArtifactUrl or Get-NavArtifactUrl
-   **includePlatform** indicates whether you want to download the platform as well
-   **force** indicates that you want to download the artifacts even though
-   **basePath** (default **c:\\bcartifacts.cache**)

You can see a sample of the command and the output here:

PS c:\\temp> Download-Artifacts -artifactUrl (Get-BCArtifactUrl -country "us") -includePlatform

Downloading application artifact /sandbox/16.2.13509.14082/us
Downloading C:\\Users\\freddyk\\AppData\\Local\\Temp\\9f550271-a1c8-4125-96c5-2b781e2b9a3e.zip
Unpacking application artifact
c:\\bcartifacts.cache\\sandbox\\16.2.13509.14082\\us
https://bcartifacts.azureedge.net/sandbox/16.2.13509.14082/platform
Downloading platform artifact /sandbox/16.2.13509.14082/platform
Downloading C:\\Users\\freddyk\\AppData\\Local\\Temp\\45959ba6-b934-470f-9603-8867135a3dcd.zip
Unpacking platform artifact
Downloading Prerequisite Components
Downloading c:\\bcartifacts.cache\\sandbox\\16.2.13509.14082\\platform\\Prerequisite Components\\Open XML SDK 2.5 for Microsoft Office\\OpenXMLSDKv25.msi
Downloading c:\\bcartifacts.cache\\sandbox\\16.2.13509.14082\\platform\\Prerequisite Components\\Microsoft Report Viewer 2015\\ReportViewer.msi
Downloading c:\\bcartifacts.cache\\sandbox\\16.2.13509.14082\\platform\\Prerequisite Components\\IIS URL Rewrite Module\\rewrite\_2.0\_rtw\_x64.msi
Downloading c:\\bcartifacts.cache\\sandbox\\16.2.13509.14082\\platform\\Prerequisite Components\\Microsoft Report Viewer 2015\\SQLSysClrTypes.msi
c:\\bcartifacts.cache\\sandbox\\16.2.13509.14082\\platform

It will download the artifacts and return the path. If you specify -includePlatform it will return two urls – if not, only one. Looking into the filesystem on the host, you will find the artifacts extracted in the bcartifacts.cache folder:

![](/assets/images/2020/working-with-artifacts/cache1.png)

In the US folder you will find a manifest.json + all the things that are specific to the US application (database, applications, configuration packages etc.)

The manifest.json contains:

{
    "version":  "16.2.13509.14082",
    "platformUrl":  "sandbox/16.2.13509.14082/platform",
    "licenseFile":  "",
    "isBcSandbox":  true,
    "country":  "us",
    "platform":  "16.0.13440.13997",
    "database":  "database\\\\database.bak"
}

In this specific sample, the cronus licensefile is already in the database and is not in the artifact. The platformUrl tag might be missing in which case the platform is grabbed by replacing the country with platform in the artifact url.

Note that we also have a sandbox country called base. base is the onprem w1 demo data running in sandbox mode. I am aware that a lot of partners have used the base docker image for running tests and decided that we had to support this in artifacts as well.

Looking into the platform artifact, you will find everything from the platform + a number of w1 things (not the database). The reason for the w1 stuff is that a lot of the localizations uses the w1 things and instead of adding these to all localizations, we added them to the platform.

![](/assets/images/2020/working-with-artifacts/plat.png)

Basically a lot of stuff from the DVD.

# Flush-ContainerHelperCache

This function is not new, but it does have a new parameter and a new option.

Flush-ContainerHelperCache -cache bcartifacts -keepDays 7

will flush the artifacts cache but keep the artifacts, which has been used the last 7 days.

Even though downloading from CDN is crazy fast, I still thought that this function could safely run on agents on a schedule to avoid the hard drive in filling up with artifacts.

As you can see, your cache folder will contain a file called lastused, which indicates to the containerhelper when this cache folder was used last. I could also do other things like deleting oldest cache folders until the cache is less than 10Gb – but for now – the days is all that is implemented.

## New-BcImage

What is this? You just thought you got rid of images and a new function appears, which can build an image????

Well yes, if you are using the same image again and again – it is much faster to run an image, which has been built, than just-in-time installation of the artifacts. So, you can build an image based in the artifacts and use it again and again.

The function has a number of parameters like baseImage, isolation, memory which are defaulted to use the baseimage, which matches your OS, but you can if you so desire use a different base image.

Mostly you will use these parameters:

-   **artifactUrl** – yes you guessed it, the artifact url you got from Get-BcArtifactUrl.
-   **imageName** – the name of your new docker image. This docker image can be used with new-bccontainer afterwards by specifying -imageName.
-   **myscripts** – exactly like you can add files to this in new-bccontainer which will end up in the c:\\run\\my folder, you can do it here as well and get them copied into the image, they will however not end up in the c:\\run\\my folder, but instead get copied into the c:\\run folder, overriding any existing files there (in 0.7.0.7 of ContainerHelper has a bug here)

Try

$artifactUrl = Get-BCArtifactUrl -country "us" -version "15"
New-BcImage -artifactUrl $artifactUrl -imageName myownimage:latest
docker images
docker inspect myownimage:latest

Yes, this is an image, which is totally like the images you have been used to use.

This means that you can build an image and publish it to a private repository if you like – or you can leave it to people to build images on the fly. It does however seem cumbersome that you have to build an image and run it. Seems like some complex programming need to ensure that things are in sync – so why don’t we make it easy…

# New-BcContainer

As already mentioned, New-BcContainer has a new parameter called artifactUrl and that is more or less everything you need to know.

However…

If you specify BOTH artifactUrl AND imagename – then New-BcContainer will test whether imageName is based on artifactUrl and the current OS etc. etc. and if that is the case – it will run the image. If not it will call New-BcImage and build imageName from artifactUrl – and run that.

Wait… – what?

Yes, try this twice:

Remove-NavContainer test
Measure-Command {
    $artifactUrl = Get-BCArtifactUrl -version 16.1 -select Latest -country us
    $credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
    New-NavContainer \`
        -accept\_eula \`
        -containerName test \`
        -artifactUrl $artifactUrl \`
        -Credential $credential \`
        -auth UserPassword \`
        -updateHosts \`
        -imagename myown
}

The first run takes 8 minutes 35 seconds on my machine – the second run only a little more than 3 minutes and after the first run, I have a docker image called **myown:sandbox-16.1.12629.14076-us**. (BTW – you need ContainerHelper 0.7.0.8 or higher for this to work)

If you don’t want to have the image named after the version, you just specify a tag yourself.

# Get-BcContainerArtifactUrl

This function can be used on a running container to discover the artifactUrl used to create the container. If the container was started using a docker image (no artifact), this function will return blank.

That’s it – I think this describes the new functionality added to the containerHelper to help people working with Business Central artifacts in docker.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
