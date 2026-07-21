---
layout: post
title: "NAV and Multi Factor Authentication (MFA)"
date: 2017-01-23 19:35:02
categories: ["Archive", "Azure", "Demo Environments"]
tags: ["AAD", "ACS", "ADFS", "Authentication", "Azure", "Azure AD", "MFA", "Multi-factor Authentication", "NAV", "NAV 2016", "NAV 2017"]
permalink: /2017/01/23/nav-and-multi-factor-authentication-mfa/
---

Over the last weeks I have gotten an increasing number of inquiries around MFA. To be honest, I had never tried to setup MFA before, but that didn’t stop me from answering.

My typical answer would be the following:

_NAV itself does not have any knowledge about multi factor authentication, but we do support claims based authentication through authentication providers and if these authentication providers are setup for MFA, then NAV should support MFA through the authentication provider._

Having answered the same thing a number of times, I decided it was time to try it out.

### Setting up a NAV DEMO environment

As always, I started out by creating a NAV DEMO environment using [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy) and in the section on Office 365 I added an Office 365 administrator email address and password. Whether you want to create the portal or not does not matter.

[![o365props](/assets/images/2017/nav-and-multi-factor-authentication-mfa/907ba-o365props-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/907ba-o365props.png)

After waiting for some time your landing page will reveal that the installation is Complete and you can get going.

Two different authentication methods

On the landing page for your demo environment you will find two sections talking about authentication. One talking about how to access your demo environment using UserName/Password Authentication:

[![o365-2](/assets/images/2017/nav-and-multi-factor-authentication-mfa/62405-o365-2-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/62405-o365-2.png)

and one talking about how to access your demo environment using AAD or O365 authentication:

[![o365-1](/assets/images/2017/nav-and-multi-factor-authentication-mfa/3b00c-o365-1-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/3b00c-o365-1.png)

**You CANNOT setup MFA for the UserName/Password authentication model**. You can remove this option of authentication if you like and ONLY have AAD / O365 authentication,

**You CAN setup MFA for AAD / O365 authentication** and you do NOT have to tell NAV anything about that. It is your little secret, and NAV just supports it.

So basically when we decide to out-source the authentication to somebody else, they can do it as they want to – as long as they bring a claim to us, describing who the user is, which we can map to a user in the NAV user table (the Authentication E-Mail address).

### AMFA (Azure Multi Factor Authentication)

Before starting to setup MFA, you should read [this](https://docs.microsoft.com/en-us/azure/multi-factor-authentication/multi-factor-authentication-get-started-cloud).

And, if you want to read more about how it actually works, go [here](https://docs.microsoft.com/en-us/azure/multi-factor-authentication/multi-factor-authentication-how-it-works).

If you just want to set it up, follow this process:

Navigate to [http://portal.office.com](http://portal.office.com) with the Office 365 administrator email and password, which you used to setup your demo environment.

Click the Waffle Icon (top left corner) and go to Admin:

[![o365-3](/assets/images/2017/nav-and-multi-factor-authentication-mfa/cddd6-o365-3-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/cddd6-o365-3.png)

Select Users, Active users:

[![o365-4](/assets/images/2017/nav-and-multi-factor-authentication-mfa/e726c-o365-4-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/e726c-o365-4.png)

Under More, select **Setup Azure multi-factor auth**

[![o365-5](/assets/images/2017/nav-and-multi-factor-authentication-mfa/3e24b-o365-5-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/3e24b-o365-5.png)

Select the user(s) you want to enable for MFA and select Enable under quick steps

[![o365-6](/assets/images/2017/nav-and-multi-factor-authentication-mfa/80bde-o365-6-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/80bde-o365-6.png)

In the dialog that pops up, select enable multi-factor auth

[![o365-7](/assets/images/2017/nav-and-multi-factor-authentication-mfa/ca407-o365-7-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/ca407-o365-7.png)

After this, when you sign out and back in you will be asked to setup MFA:

[![o365-9](/assets/images/2017/nav-and-multi-factor-authentication-mfa/2d872-o365-9-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/2d872-o365-9.png)

When you click Set it up now, you will be taken to the additional security verification setup web site:

[![o365-10](/assets/images/2017/nav-and-multi-factor-authentication-mfa/333ff-o365-10-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/333ff-o365-10.png)

You can also get to this web site by navigating to [https://aka.ms/MFASetup](https://aka.ms/MFASetup) and select profile -> Additional security verification.

After the setup of MFA is complete, locate the URL on your landing page for accessing the Web Client using AAD or O365 authentication and after providing your password you will get a text message and will be asked to provide that.

[![o365-8](/assets/images/2017/nav-and-multi-factor-authentication-mfa/86602-o365-8-1.png)](/assets/images/2017/nav-and-multi-factor-authentication-mfa/86602-o365-8.png)

So, as you can see MFA does not require any changes in NAV, it is something that is handled by the authentication provider and there is actually nothing you can setup in NAV for that.

I have also tested that the Windows Client works with the same setup.

### Other authentication providers

If you are using ADFS or other authentication providers supported by NAV (we do not have an exhaustive list) then it is the same deal. We redirect to the authentication provider for authentication and if that would use any number of authentication validations before returning an authentication email in a claim to NAV, we don’t really care:-)

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
