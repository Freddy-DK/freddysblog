---
layout: post
title: "Organizing your .al files"
date: 2019-08-02 07:56:29
categories: ["AL Development", "Docker", "NavContainerHelper"]
tags: ["AL", "Docker", "Git", "new-navcontainer", "Txt2Al"]
permalink: /2019/08/02/organizing-your-al-files/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

This blog post is primarily directed towards partners who wants to do AL code customization in **Microsoft Dynamics 365 2019 release wave 2**. Even though the tools can be used for extension development as well, typically people working on extensions already have organized their files.

So…, in the old days, we didn’t really have to bother about source file structure. Everything was in the database and the Object Designer was the tool to use. Having said that, most serious partners are exporting objects to text files and storing those in some kind of source control, but the structure of these files were typically just placed in one directory since it was only for storage/source control.

When moving to AL the situation is different. The file structure will surface directly in VS Code and you will want to arrange your files in the way you want to work with them and placing all 6000+ source files in one folder might not be the right approach.

# How do you want to arrange your files?

If I asked 10 partners about how source files should be named and structured, I might end up with 10 different answers.

-   Should the file name contain the ID?
-   Should files be structured after type?
-   Should the file name contain the type?
-   Should files be structured after functional area?
-   In which order should type, name and ID appear in the name?
-   etc. etc.

and some people will be rather locked on their opinion, which I think is totally fine.

# Converting the baseapp to AL

Since the spring 2019 release of Business Central, it has been possible to convert the baseapp to AL, make modifications and publish your own baseapp to a container as described in this blog post: [https://freddysblog.com/2019/04/15/c-al-to-al-code-customizations/](/2019/04/15/c-al-to-al-code-customizations/)

Running this script:

```
$imageName = "mcr.microsoft.com/businesscentral/onprem:1904-rtm-ltsc2019"
$containerName = "test"
$auth = "UserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "c:\temp\mylicense.flf"

New-BCContainer -accept_eula -accept_outdated `
                -imageName $imageName `
                -containerName $containerName `
                -auth $auth `
                -credential $credential `
                -licenseFile $licenseFile `
                -updateHosts `
                -includeAL

$alProjectFolder = "C:\ProgramData\NavContainerHelper\AL\BaseApp-rtm"
Create-AlProjectFolderFromNavContainer -containerName $containerName `
                                       -alProjectFolder $alProjectFolder `
                                       -useBaseLine `
                                       -addGIT `
                                       -useBaseAppProperties
```

will give you a folder with all files in one big pile. Running the same script on 1904-cu1 will give you a folder with cu1 files and if you compare the content of the rtm al folder and the cu1 al folder, you will see that 556 files have changed and a few have been added:

![rtmvscu1](/assets/images/2019/organizing-your-al-files/rtmvscu1.png)

This might seem manageable, but you can of course ask yourself why you need to know – I will get back to this later.

Lets try to run the above script on newly created 15.x container and perform the same compare:

![cu1vs15x](/assets/images/2019/organizing-your-al-files/cu1vs15x.png)

Horror – not a single file is unchanged – everything is different…

In fact the structure has changed. Looking into the folder, you will see:

![structure1](/assets/images/2019/organizing-your-al-files/structure1.png)

But that’s not all – under src, you will also see some folders and the source for the objects, but the observant reader will quickly notice that file names are significantly different from the file names in 14.x:

![structure2.PNG](/assets/images/2019/organizing-your-al-files/structure2.png)

# Why?

Why did Microsoft do this? Are we just trying to make life miserable for partners?

Of course not. There are in fact multiple reasons for doing this and making life miserable for partners is definitely NOT one of them. In fact, I will try to explain ways, you (the partner) can make your life easier, just wait for it.

It is no secret, that we want to refactor the application to become more extensible. One task on this journey is to move functionality to an AL Application Foundation layer (see [https://github.com/microsoft/ALAppExtensions](https://github.com/microsoft/ALAppExtensions)), and as you can imagine, this will cause some disruption of the baseapp source.

We will also work on making the file structure more manageable and make it easier for our developers to work with AL files and we don’t want to be stuck with a naming, which really was done because of C/SIDE.

So, you will see disruption, you will see a LOT of changes and for some of you, this will mean work.

# But,… how do I compare?

Before answering this question,  we need to discuss **what you need to compare?**

Do you really need to go through all modified files? or do you only need to concentrate on the files you have changed? Remember that this is really only needed because you do code customizations. If you are creating extensions, you might still want to know what is going on behind the scenes, but if your extension compiles and your tests are running, there is really no reason in going through all code modifications in the baseapp.

Lets look at your solution. The objects in your code customized solution can be split into three:

-   Objects you have added
-   Objects you have modified
-   Objects you haven’t touched

I assume that nobody are deleting objects.

The only objects you really need to compare and concentrate about are the objects you have modified, which also have baseapp modifications. I know that customized applications also might have dependencies from added or modified objects to unmodified objects and this contract might be broken, but this is kind of the same situation you would be in if you are developing an extension.

In fact if you think about it, **one of the most important task partners will have over the next years is to make sure that the middle bucket above (objects you have modified) gets empty**. This makes the path to becoming an extension much easier and will save you time.

As you probably already guessed, I would structure my files in these three buckets, and I would work on removing files from the modified folder – probably by creating new files in the added folder (page modification becomes page extensions etc.). I would only if impossible to avoid allow additional files from the unmodified folder to be modified (stop the bleeding) and try to find ways around this.

# Applying a file structure

The **Create-AlProjectFolderFromNavContainer** function from above has a parameter called **\-alFileStructure**, where you can describe the file structure.

The alFileStructure parameter is NOT a set of pre-defined file structure constants, instead it is a function definition, which gets called for each single object during the copy to determine where to put the file.

You do need a list of your modified objects, but I will share a PowerShell script, which can give you that later.

The function definition looks like this:

```
$myModifiedObjects = @("table18","page21","report10", ".rdlc10")
$alFileStructure = { Param ([string] $type, [int] $id, [string] $name)
    if ($myModifiedObjects.Contains("$type$id")) {
        $folder = "Modified"
    }
    elseif (($id -ge 50000 -and $id -le 99999) -or ($id -gt 1999999999)) {
        $folder = "My"
    }
    else {
        $folder = "BaseApp"
    }
    "$folder\$($name).$($type).al"
}
```

If you have number ranges or naming conventions indicating functional areas of you added objects you can even arrange your added objects into functional folders very easily.

At this time, the $type will be one of:

-   enum
-   page
-   table
-   codeunit
-   report
-   query
-   xmlport
-   profile
-   dotnet
-   enumextension
-   pageextension
-   tableextension
-   .xlf

More might be added. I suggest to create a switch statement, which throws an error if an unknown type occurs (if you are placing files based on the types of course).

Re-running the above script with the alFileStructure defined, suddenly the output of 14 cu1 and 15.x looks similar (same naming, same structure).

We can now concentrate on the 4 files in our modified folder, but when we open up the difference between the Customer Card we don’t really get what we expect/hoped for:![compare](/assets/images/2019/organizing-your-al-files/compare-1.png)

A lot of the differences are indentation changes due to the conversion from txt2al in cu1 creates the files with a different indentation than we use today. Also the 15.x folder has spaces between operators etc. and the compare tool becomes confused. Confused to the point where it is almost not useful.

# and why is that?

The answer to this (as is the answer to almost all these things) is innovation. In 15.x the compiler contains a code analysis tool, which can perform formatting of the source and of course, we want to take advantage of this going forward to increase developer productivity and product quality.

# OK, fine – but what can we do about this?

Thankfully, the Modern Dev team created a small and very smart feature, which will ship in either the next CU (or the one after that) of 14.x, which will contain an updated txt2al that uses the code analysis DLL to format the documents. You can wait for the next CU, move your solution to this CU, perform the AL conversion and the syntactical differences should be gone.

Also, the latest insider build of 15.x also contains an updated txt2al with the updated codeanalysis – even though 15.x doesn’t contain C/AL or .txt files, the tool is still there.

I do realize that not everybody wants to wait for the next CU, so I decided to add a small **hack** to New-NavContainer. With a new parameter (called **\-runTxt2AlInContainer**) you can specify that you want to use the txt2al tool from a different container when doing the conversion from Text to AL.

I realize that you have to spin up two containers, but the second container (the txt2al container) can be removed shortly after – and hey – you probably need the 15.x source anyway.

In the following script, I use a specific build of the insider 15.x container, where I know the tool is available. Note, that you will need NavContainerHelper version 0.6.2.91 in order to run this.

```
$myModifiedObjects = @("table18","page21","report10", ".rdlc10")
$alFileStructure = { Param ([string] $type, [int] $id, [string] $name)
    if ($myModifiedObjects.Contains("$type$id")) {
        $folder = "Modified"
    }
    elseif (($id -ge 50000 -and $id -le 99999) -or ($id -gt 1999999999)) {
        $folder = "My"
    }
    else {
        $folder = "BaseApp"
    }
    "$folder\$($name).$($type).al"
}

$auth = "UserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "c:\temp\mylicense.flf"

$imageName15 = "bcinsider.azurecr.io/bcsandbox-master:15.0.34731.0-w1-ltsc2019"
$containerName15 = "test15"
New-BCContainer -accept_eula -accept_outdated `
                -imageName $imageName15 `
                -containerName $containerName15 `
                -auth $auth `
                -credential $credential `
                -licenseFile $licenseFile `
                -updateHosts `
                -includeAL

$alProjectFolder = "C:\ProgramData\NavContainerHelper\AL\BaseApp-15x"
Create-AlProjectFolderFromNavContainer -containerName $containerName15 `
                                       -alProjectFolder $alProjectFolder `
                                       -useBaseLine `
                                       -addGIT `
                                       -useBaseAppProperties `
                                       -alFileStructure $alFileStructure

$imageName14 = "mcr.microsoft.com/businesscentral/onprem:1904-cu1-ltsc2019"
$containerName14 = "test14"
New-BCContainer -accept_eula -accept_outdated `
                -imageName $imageName14 `
                -containerName $containerName14 `
                -auth $auth `
                -credential $credential `
                -licenseFile $licenseFile `
                -updateHosts `
                -includeAL `
                -runTxt2AlInContainer $containerName15

$alProjectFolder = "C:\ProgramData\NavContainerHelper\AL\BaseApp-14cu1"
Create-AlProjectFolderFromNavContainer -containerName $containerName14 `
                                       -alProjectFolder $alProjectFolder `
                                       -useBaseLine `
                                       -addGIT `
                                       -useBaseAppProperties `
                                       -alFileStructure $alFileStructure
```

Only difference from the first script is that we use the 15.x container for running txt2al in the 14.x container and voila:

![compare2](/assets/images/2019/organizing-your-al-files/compare2.png)

That is totally what I expected / hoped for.

# Conclusion

Being able to carry your code customized solution forward to a structure and format, which is comparable with 15.x is key to a lot of partners.

In this blog post you have seen how this can be done using Docker Containers, but you can of course grab the ideas and learning from this to create a setup without Docker if you are a sworn enemy of Docker. I definitely do not want to force anything upon anybody, but it makes life a LOT easier for me describing these things using Docker. I can do more in less time for the benefit of all, so bear with me that I do not describe how to do this without Docker, nor will I spend time on helping people do this. Everything is open source, feel free to find and borrow any code/idea you like.

# What’s next

Next up is how this can be used together with GIT. How can you move your code customized solution to 15.x and have GIT help you identify your changes on top of 15.x and not your changes on top of 14.x against 15.x changes.

Hope that makes sense.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
