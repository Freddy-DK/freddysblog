---
layout: post
title: "NavContainerHelper – Certificates"
date: 2018-03-20 03:29:51
categories: ["Docker", "NavContainerHelper", "Not Archived"]
tags: ["Certificates", "Docker", "NAV on Docker", "NavContainerHelper"]
permalink: /2018/03/20/navcontainerhelper-certificates/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read the [first post](/2018/03/20/navcontainerhelper-1/) about the NavContainerHelper, you should do so.

If you want to secure the communication to your container, you need to use a certificate.

## Use SSL with a self-signed certificate

I you want to add a certificate to a container started by New-NavContainer, you ccan use the parameter:

```
-useSSL
```

Example:

```
New-NavContainer -accept_eula `
                 -useSSL `
                 -containerName "test" `
                 -auth NavUserPassword `
                 -imageName "microsoft/dynamics-nav"
```

You will notice, that the output section looks slightly different:

```
Container IP Address: 172.19.145.168
Container Hostname  : test
Container Dns Name  : test
Web Client          : https://test/NAV/
Dev. Server         : https://test
Dev. ServerInstance : NAV

Files:
http://test:8080/al-0.12.17720.vsix
http://test:8080/certificate.cer
```

The Web Client and Dev. Server are both secured with a self-signed certificate (https) and the link to download and trust this certificate on your computer is displayed underneath.

**Note**, The -useSSL parameter causes a New-NavContainer to add _–env useSSL=Y_ to the docker run command.

**Note**, if you are planning to expose your container outside the boundaries of your own machine, you should always use SSL.

## Use SSL with a LetsEncrypt certificate

LetsEncrypt is a certificate provider which issues free SSL certificates for services. Furthermore, there is a PowerShell module, which enables you to do this automatically.

This PowerShell module is being used in the NAV ARM Templates (like [http://aka.ms/getnav](http://aka.ms/getnav)) and the code can be found [here](https://github.com/Microsoft/nav-arm-templates/blob/master/initialize.ps1) (search for LetsEncrypt).

The code to import and use the certificate is the same as you use when using a certificate issued by a trusted authority.

## Use a certificate, issued by a trusted authority

There are no parameters in which you can specify a certificate directly. Instead, you will have to override the SetupCertificate script in the Docker image.

Overriding scripts is done by specifying adding a parameter called

```
-myScripts @(file1, file2)
```

The files specified in myScripts will be copied to a folder, which is shared to the container as c:\\run\\my. You can see more about overriding scripts laer in this document.

Create a script file called SetupCertificate.ps1 with this content:

```
$certPfxFile = Join-Path $PSScriptRoot "navdemo.net.pfx"
$certPfxPassword = "Password"
$dnsidentity = "navdemo.net"
$publicDnsname = "${hostname}.${dnsidentity}"

$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $certPfxFile,$certPfxPassword
$certificateThumbprint = $cert.Thumbprint

Write-Host "Certificate File Thumbprint $certificateThumbprint"
if (!(Get-Item "Cert:LocalMachinemy$certificateThumbprint" -ErrorAction SilentlyContinue)) {
    Write-Host "Import Certificate to LocalMachinemy"
    $certPfxSecurePassword = ConvertTo-SecureString -String $certPfxPassword -AsPlainText -Force
    Import-PfxCertificate -FilePath $certPfxFile -CertStoreLocation "cert:localmachinemy" -Password $certPfxSecurePassword | Out-Null
}
```

If the certificate you use isn’t issued by an authority, which is in the Trusted Root Certification Authorities, then you will have to import the pfx file to LocalMachineroot as well as LocalMachinemy, using this line:

```
Import-PfxCertificate -FilePath $certPfxFile -CertStoreLocation "cert:localmachineroot" -Password $certPfxSecurePassword | Out-Null
```

Example:

```
New-NavContainer -accept_eula `
                 -useSSL `
                 -myscripts @("c:\temp\SetupCertificate.ps1", "C:\temp\navdemo.net.pfx") `
                 -containerName "test" `
                 -auth NavUserPassword `
                 -imageName "microsoft/dynamics-nav"
```

**Note**, for this to work, the publicdnsname (here test.navdemo.net) specified in the script needs to exist in the DNS and point to the host computer, typically this is done by creating a CNAME record.

**Note**, New-NavContainer creates a folder for the files specified in -myscripts and shares this folder to the c:\\run\\my folder in a container using _–volume :c:\\run\\my_.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
