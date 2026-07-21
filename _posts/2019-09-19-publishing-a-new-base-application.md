---
layout: post
title: "Publishing a New Base Application"
date: 2019-09-19 14:02:10
categories: ["AL Development", "Docker", "NavContainerHelper", "PowerShell"]
tags: ["Extension", "Publish"]
permalink: /2019/09/19/publishing-a-new-base-application/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I think I have mentioned before, that I strongly recommend partners to use the extension model and create extensions, which can be published, installed and upgraded much easier then code customizations.

Having said that, I know that there are partners out there for who AL code customizations will be the right stepping stone towards adapting an extension model.

This blog post is very targeted towards these partners. Partners who are creating their own Base Application in 15.x and need to work with that.

# The challenges

In [this blog post](https://freddysblog.com/2019/07/31/preview-of-dynamics-365-business-central-2019-release-wave-2/) it is described how to create a AL Project folder in which you can start to apply your changes.

The Create-AlProjectFolderFromBCContainer function will extract the AL Source and setup a project folder for you. When doing this you have one choice to make: **_Which Id, Publisher, Name and Version should me app have?_**

The Create-AlProjectFolderFromBCContainer can be called with either:

\-useBaseAppProperties

or by specifying:

\-id $appId \`
-publisher $appPublisher \`
-name $appName \`
-version $appVersion

If you decide to use the baseapp properties, your new Base Application will inherit the id, name publisher and version from the existing Microsoft Base Application and you will be able to save the demo data from the container when publishing your new Base Application.

Furthermore, all the apps that are installed in the container (Forecast, PayPal, APIv1, HeadLines, etc.) that have a dependency on the Microsoft Base Application can also be installed on your Base Application (and will be working unless your modifications break something).

The same goes for all the test apps shipped on the DVD or in the container and even part of the test framework. They all have dependencies on the Microsoft Base Application – ID=**437dbf0e-84ff-417a-965d-ed2bb9650972**, Name=**Base Application**, Publisher=**Microsoft** and Version=**15.0.0.0**.

# Your own app identity

If you however need or want your New Base Application to have it’s own app identity, you can imagine that a number of the things I just described will add some challenges.

A short list of the challenges are:

-   You will not be able to save the data.
-   Publishing the built-in apps
-   Running tests

# You will not be able to save the data

It is rather logical if you think about it. You have uninstalled and unpublished one application and now you publish and install another application with a different identity – it cannot just take over the data from a totally different application. Yes, there might be ways to do this, but right now there are not.

If you look into the database of a container with the Microsoft Base Application, you will see:

![tables](/assets/images/2019/publishing-a-new-base-application/tables.png)

All tables belonging to an application will be prefixed with the company name (as it has always been the case) and suffixed with the application Id.

Looking into the database in a container running a Base Application with it’s own identity will of course look different (in this example I also renamed the company):

![tables2.PNG](/assets/images/2019/publishing-a-new-base-application/tables2.png)

And there is currently no way of telling Business Central that after the death of 437dbf0e-84ff-417a-965d-ed2bb9650972, then 88b7902e-1655-4e7b-812e-ee9f0667b01b should inherit everything. This means that when you change the Application properties, you are better off creating a new database.

# Clean-BCContainerDatabase

Clean-BCContainerDatabase was originally created in order to remove the C/AL application objects from the database and prepare the database to have the AL application published.

In order to remove the C/AL objects, it was necessary to uninstall and unpublish applications. At that time it wasn’t possible to carry data forward (upgrading), I will get back to how to move data from C/AL tables to AL extension tables in a separate blog post.

Later the function was extended to uninstall and unpublish the Base Application in order to install a new one – basically the same as from C/AL to AL – just replace one AL Base Application with another. As long as you reuse the application properties, your installation is seen as an upgrade and you can save the data.

If you change the application properties, you might be better off creating a new database. Add the parameter **\-useNewDatabase** to tell the function that you want to:

1.  Create a new database
2.  Import the license file
3.  Create a SUPER user
4.  Publish System symbols
5.  Create a Company
6.  Publish the System Application

and leave the database ready for publishing a new Base Application and other applications.

Obviously this doesn’t allow you to save the data for any applications, so you would have to import and apply a rapid start package in order to make this happen, which I just blogged about here: [http://freddysblog.com/2019/09/19/using-apis-on-containers/](http://freddysblog.com/2019/09/19/using-apis-on-containers/)

**Note, that in order to use the v1.0 APIs you need to install the APIV1 app** (which was installed by default). This can be done manually or automated using the parameter called **\-restoreApps** on **Publish-NewApplicationToBCContainer**.

# Publish-NewApplicationToBCContainer

Like the **Clean-BCContainerDatabase**, the **Publish-NewApplicationToBCContainer** is used when publishing an AL application to a container.

The parameter **\-saveData** controls whether or not data will be saved (controls the **doNotSaveData** flag of **Uninstall-navapp**, which of course doesn’t make sense combined with **\-useNewDatabase**.

The parameter **\-useNewDatabase** is new, it which controls the way it cleans the database and a parameter called **\-restoreApps**, which controls whether or not the apps already installed in the container are downloaded and restored in the new container.

Now you might wonder…

# What about dependencies?

If this function can restore apps and I have changed the app Id of the Base Application, what about dependencies?

In a later version of the platform, you will be able to somehow declare that your Base Application is compatible with the Microsoft Base Application. Today we have to replace the dependencies in the .app with the properties of the new Base Application.

For this, **Publish-NavContainerApp** has a new parameter called **\-replaceDependencies**. In this parameter you can declare which dependencies you want to replace and this way, you can create your own **System Application** or **Base Application** and publish other applications without the need to rebuild them. The **Publish-NavContainerApp** will, if the parameter is specified make a call to the function **Replace-DependenciesInAppFile** with the destination set as a temp. file – and then publishing this temp. file, but never modifying the original.

# Replace-DependenciesInAppFile

This function has 3 parameters. **\-Path** points out the .app file to investigate. **\-Destination** points out the filename where to put the .app file with changed dependencies and a parameter describing which dependencies to replace.

The value for to be specified in **\-replaceDependencies** should be a **hashtable**, where the keys are **AppIds** and the value is a **hashtable** with the **new properties**.

Example:

$appId = "88b7902e-1655-4e7b-812e-ee9f0667b01b"
$appPublisher = "Freddy Kristiansen"
$appName = "MyBaseApp"
$appVersion = "1.0.0.0"
$replaceDependencies = @{
    "437dbf0e-84ff-417a-965d-ed2bb9650972" = @{
        "id" = $appId
        "name" = $appName
        "publisher" = $appPublisher
        "minversion" = $appVersion
    }
}

Transferring this value will replace all dependencies to the Microsoft Base Application (437…) with the values I have defined above.

The **\-replaceDependencies** parameter is of course also surfaced on **Publish-NewApplicationToBCContainer**, which then is used when restoring apps.

The **\-replaceDependencies** is also added to **Import-TestToolkitToBCContainer**, allowing you to publish the Test Framework and the Tests while Just-In-Time replacing dependencies from the Microsoft Base Application to your Base Application.

# The Script

$imageName = "bcinsider.azurecr.io/bcsandbox-master:w1-ltsc2019"
$containerName = "mycontainer"
$auth = "UserPassword"
$credential = New-Object PSCredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "c:\\temp\\license.flf"
$alProjectFolder = "C:\\ProgramData\\NavContainerHelper\\MyBaseApp"

New-BCContainer -accept\_eula \`
                -imageName $imageName \`
                -containerName $containerName \`
                -auth $auth \`
                -Credential $credential \`
                -licenseFile $licenseFile \`
                -updateHosts \`
                -includeAL \`
                -memoryLimit 10G

$myModifiedObjects = @()
$alFileStructure = { Param (\[string\] $type, \[int\] $id, \[string\] $name) 
    if ($myModifiedObjects.Contains("$type$id")) {
        $folder = "Modified"
    }
    elseif (($id -ge 50000 -and $id -le 59999) -or ($id -gt 1999999999)) {
        $folder = "My"
    }
    else {
        $folder = "BaseApp"
    }

    $name = -join ($name.ToCharArray() | Where-Object { \[char\]::IsLetterOrDigit($\_) })
    switch ($type) {
        "dotnet" { "$folder\\$($name).al" }
        ".rdlc"  { "\*\\layouts\\\*.rdlc" }
        ".docx"  { "\*\\layouts\\\*.docx" }
        ".xlf"   { "$folder\\$($name).xlf" }
        default  { "$folder\\$($name).$($type).al" }
    }
}

$appId = "88b7902e-1655-4e7b-812e-ee9f0667b01b"
$appPublisher = "Freddy Kristiansen"
$appName = "MyBaseApp"
$appVersion = "1.0.0.0"

Create-AlProjectFolderFromBCContainer -containerName $containerName \`
                                      -alProjectFolder $alProjectFolder \`
                                      -useBaseLine \`
                                      -id $appId \`
                                      -publisher $appPublisher \`
                                      -name $appName \`
                                      -version $appVersion \`
                                      -alFileStructure $alFileStructure

$replaceDependencies = @{
    "437dbf0e-84ff-417a-965d-ed2bb9650972" = @{
        "id" = $appId
        "name" = $appName
        "publisher" = $appPublisher
        "minversion" = $appVersion
    }
}

$app = Compile-AppInBCContainer -containerName $containerName \`
                                -credential $credential \`
                                -appProjectFolder $alProjectFolder \`
                                -appOutputFolder $alProjectFolder \`
                                -UpdateSymbols

Publish-NewApplicationToBCContainer -containerName $containerName \`
                                    -appDotNetPackagesFolder (Join-Path $alProjectFolder ".netpackages") \`
                                    -appFile $app \`
                                    -credential $credential \`
                                    -useNewDatabase \`
                                    -restoreApps Yes \`
                                    -doNotUseDevEndpoint \`
                                    -replaceDependencies $replaceDependencies

Import-TestToolkitToBCContainer -containerName $containerName \`
                                -includeTestLibrariesOnly \`
                                -credential $credential \`
                                -replaceDependencies $replaceDependencies

After this you can use the script here [http://freddysblog.com/2019/09/19/using-apis-on-containers/](http://freddysblog.com/2019/09/19/using-apis-on-containers/) to import and apply a configuration package.

That’s it – you probably cannot use the script unmodified, but the various pieces should tell you how things works – if you really need this…

# Last, but not least… – the DevEndpoint

One last thing – the various functions also have a parameter called **\-useDevEndpoint** (or **doNot-**). The DevEndpoint is what VS Code uses to publish an app with.

An App can exist in **global scope** or **tenant scope** and from PowerShell you can deploy to any of these. From VS Code (using the **DevEndpoint**) you can only publish to **tenant scope**, and the only thing you really should care about here is, that an app in the global scope **CANNOT** have a dependency to an app in the tenant scope.

Kind of sounds obvious – but now you know.

_**Also you cannot re-publish an app from VS Code if the app was already published from PowerShell without the DevEndpoint.**_

Maybe not quite as obvious, but that’s how it is.

This means that if you decide to publish the Base Application to the Dev Endpoint to enable Rapid Application Development, then you also need to publish the Test Framework and other apps using the Dev Endpoint as well.

And that should all be working just fine…:-)

# Handle with care…

The **\-replaceDependencies** is a temporary hack – until the platform gets support for doing this the right way.

But… – it is a convenient temporary hack, which works for development and testing purposes. Please handle with care…

Enjoy

**Freddy Kristiansen**  
Technical Evangelist
