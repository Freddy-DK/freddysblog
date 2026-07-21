---
layout: post
title: "Managing secrets in your scripts"
date: 2021-03-02 10:20:04
categories: ["BcContainerHelper", "CI/CD", "PowerShell"]
tags: ["Key Vault", "Plain Text", "SecureString"]
permalink: /2021/03/02/managing-secrets-in-your-scripts/
---

Most people have tried it. You are writing a script, which connects to a service using a user name and a password or you are using a Shared Access Service token to access insider builds from Microsoft and you just put the “secret” values right there in your source code. After all – it is only you who can access the data on your computer right? and you will of course remember to mask out these secrets if you ever need to share the script…

Well some times we forget and some times hackers to get access to files on your computer and suddenly you might have leaked data or secrets to public places, which you shouldn’t – or sometimes we don’t even realize that we have exposed a secret.

# My story

Yes, I have of course done this myself.

How many people are aware that for more than a year, the nugetkey for publishing modules in my name to the PowerShell Gallery was actually part of NavContainerHelper. Just have a look in the settings.ps1 file in version 0.6.0.0 of NavContainerHelper ([PowerShell Gallery | settings.ps1 0.6.0.0](https://www.powershellgallery.com/packages/navcontainerhelper/0.6.0.0/Content/settings.ps1)) – a nice and clear text nugetkey. The settings file was only on my build machine and was never checked into the depot, so I thought I was safe. I just didn’t know that Publishing PowerShell packages took all files in the folder. When I found out, I changed the key and checked whether the key was ever misused – it wasn’t.

# My solution

I use a KeyVault for my secrets and it is almost as easy as writing the secret directly in the source code. Using the Az PowerShell module, I have created a KeyVault like this:

New-AzKeyVault \`
    -ResourceGroupName "freddyk" \`
    -Name "FreddysSecrets" \`
    -Location "West Europe"

This creates a KeyVault for me to access.

Storing a password in my KeyVault, I can do it like this:

Set-AzKeyVaultSecret \`
    -VaultName "FreddysSecrets" \`
    -Name "Password" \`
    -SecretValue (ConvertTo-SecureString 'P@ssw0rd' -AsPlainText -Force)

All my scripts then starts with this code:

if (!($PasswordSecret)) {
    Get-AzKeyVaultSecret "FreddysSecrets" | % {
        Write-Host $\_.Name
        Set-Variable \`
            -Name "$($\_.Name)Secret" \`
            -Value (Get-AzKeyVaultSecret "FreddysSecrets" -Name $\_.Name)
    }
}

which ends up giving me a variable called

$passwordSecret

in the current session.

In my script, I can now use the secret value as a secretString by specifying:

$passwordSecret.SecretValue

or if I need it as a string, I can use

$passwordSecret.SecretValueText

I have read that SecretValueText has been deprecated in the latest Az PowerShell module, so I have added a function in the next BcContainerHelper called Get-PlainText, which I can use like this:

($passwordSecret.SecretValue | Get-PlainText)

Not sure this was what they had in mind, but some of these secrets (like licensefile) are to be transferred in clear text to the functions, and the construct to convert securestring to plain text is just not something I can remember every time I need it.

Another benefit of the Key Vaults is that Azure DevOps pipelines can access Key Vaults and use values as variables as well.

BTW – I have removed the FreddysSecrets Key Vault.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
