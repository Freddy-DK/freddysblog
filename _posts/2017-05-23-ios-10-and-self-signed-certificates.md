---
layout: post
title: "iOS 10 and self signed certificates"
date: 2017-05-23 09:13:33
categories: ["Not Archived"]
tags: ["Apple", "Azure", "Certificates", "DEMO", "iPad", "iPhone", "NAV", "NAV 2017", "Self-Signed"]
permalink: /2017/05/23/ios-10-and-self-signed-certificates/
---

With the release of iOS 10, Apple have changed the way self-signed certificates works and since self-signed certificates are a vital part of the Microsoft Dynamics NAV Demo Environment setup, I thought I would describe how to connect to a Demo Environment, signed by a self-signed certificate from an iPad or an iPhone.

It is no longer sufficient to install the profile (self-signed certificate), you also have to trust the certificate…

# Install the Self-Signed Certificate from the Landing page

After deploying a Demo Environment using [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy), you open the landing page of you Demo Environment and install the Certificate by clicking the Download Certificate link in the top right corner.

[![img\_0456](/assets/images/2017/ios-10-and-self-signed-certificates/4094c-img_0456-1.png)](/assets/images/2017/ios-10-and-self-signed-certificates/4094c-img_0456.png)

Now follow the installation process and note especially the warning when installing the certificate.

[![warning](/assets/images/2017/ios-10-and-self-signed-certificates/355b0-warning-1.png)](/assets/images/2017/ios-10-and-self-signed-certificates/355b0-warning.png)

It actually tells us, that the certificate will not work for trusted websites unless we enable it in Certificate Trust Settings.

But nobody reads these things, right?

You just click install and expect everything to work – but it doesn’t.

And where is this Certificate Trust Settings???

# Setting->General->About->Certificate Trust Settings

Hidden under the About menu in the General section of the Settings app – you will find the Certificate Trust Settings and there you will have to enable your self-signed certificate

[![img\_0462](/assets/images/2017/ios-10-and-self-signed-certificates/5d4b5-img_0462-1.png)](/assets/images/2017/ios-10-and-self-signed-certificates/5d4b5-img_0462.png)

Move the slider over and say continue to the question on whether you want to enable this root certificate,

Now you should be able to click the Configure App link on the landing page and connect your iPad App to your Demo Environment again:

[![configure](/assets/images/2017/ios-10-and-self-signed-certificates/7165a-configure-1.png)](/assets/images/2017/ios-10-and-self-signed-certificates/7165a-configure.png)

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
