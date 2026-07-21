---
layout: post
title: "NavContainerHelper – Authentication"
date: 2018-03-20 03:33:07
categories: ["Docker", "NavContainerHelper"]
tags: ["Authentication", "Docker", "NAV on Docker", "NavContainerHelper"]
permalink: /2018/03/20/navcontainerhelper-authentication/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](https://freddysblog.com/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

This post covers the different ways of setting up authentication for your Container.

## Specify username and password for your NAV SUPER user

The parameter needed to specify username and password for your NAV Super user is

```
-credential $credential
```

The credentials are of type System.Management.Automation.PSCredential and can be created like this

```
$securePassword = ConvertTo-SecureString -String "P@ssword1" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -argumentList "admin", $securePassword
```

or, if you want to ask the user to enter the credentials in a dialog

```
$credential = get-credential
```

The Nav-ContainerHelper transfers the password to the container as an encrypted string and the key to decrypt the password is shared in a file and deleted afterwards. This allows you to use Windows Authentication with your domain credentials in a secure way.

Example:

```
if ($credential -eq $null -or $credential -eq [System.Management.Automation.PSCredential]::Empty) {
    $credential = get-credential -UserName "admin" -Message "Enter NAV Super User Credentials"
}
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -auth NavUserPassword `
                 -imageName "microsoft/dynamics-nav" `
                 -Credential $credential
```

**Note**, if you use docker run to run your container, you will transfer credentials in clear text to the container and can be retrieved by a simple docker inspect on the host. If you want to transfer the password securely, you need to encrypt the password and transfer a file containing the encryption key to the container using the two environment variables securepassword, passwordkeyfile and removepasswordkeyfile, this is what the NavContainerHelper is doing.

## Setup Windows Authentication with the Windows User on the host computer

The parameter used to specify that you want to use Windows Authentication is

```
-auth Windows
```

Which also is the default if you do not specify -auth. A container doesn’t have its own Active Directory and cannot be joined into an AD, but you can still setup Windows Authentication by sharing your domain credentials to the container.

Example:

```
New-NavContainer -accept_eula `
                 -containerName "test" `
                 -auth Windows `
                 -imageName "microsoft/dynamics-nav" `
                 -Credential $credential
```

**Note**, if the host computer cannot access the domain controller, Windows authentication might not work properly.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
