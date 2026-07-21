---
layout: post
title: "So, you want to import your Office 365 users, huh???"
date: 2016-12-16 07:10:58
categories: ["PowerShell"]
tags: ["Azure", "Azure AD", "Import", "NAV", "NAV 2017", "O365"]
permalink: /2016/12/16/so-you-want-to-import-your-office-365-users-huh/
---

Yesterday evening when I was writing the blog post about performance testing, I got a question on whether I had a script, which could import users from Office 365 into your NAV.

I didn’t, but hey, 10 minutes after, I did:-)

Here’s the script. It is designed to run on the Azure Image, but it is really only the locationof the NavAdminTool.ps1 and the serverInstanc name (NAV), which are hardcoded to the Azure Image. You can easily change these for your own environment.

\# O365 admin username and password
$Username = ""
$Password = ""

# Import Admin Tools
. ("c:\\program files\\Microsoft Dynamics NAV\\100\\Service\\NavAdminTool.ps1") | Out-Null

# Login to Azure Ad
$credential = New-Object PSCredential -ArgumentList $UserName, (ConvertTo-SecureString -String $password -AsPlainText -Force)
$account = Add-AzureRmAccount -Credential $credential
$aadTenant = $account.Context.Account.Tenants\[0\]
if (!($account.Context.Tenant)) {
    Set-AzureRmContext -TenantId $aadTenant | Out-Null
}

# Enumerate Users and add them to NAV
Get-AzureRmAdUser | % {
    $navemail = $\_.UserPrincipalName
    $navusername = $navemail.Split('@')\[0\]
    $navfullname = $\_.DisplayName
    Write-Host "$navusername, $navfullname \[$navemail\]"
    if (!(Get-NAVServerUser -ServerInstance NAV | Where-Object { $\_.Username -eq $navusername -or $\_.AuthenticationEMail -eq $navemail })) {
        Write-Host "Add user"
        New-NavServerUser -ServerInstance NAV -UserName $navusername -FullName $navfullname -LicenseType Full -AuthenticationEmail $navemail
        New-NAVServerUserPermissionSet -ServerInstance NAV -UserName $navusername -PermissionSetId SUPER 
    } else {
        Write-Host "User already exists"
    }
}

I might add this to the O365 Integration script in NAV 2017 CU2 with a question on whether you want to import your existing Office 365 users, seems like a good idea, thanks @DagFrS.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
