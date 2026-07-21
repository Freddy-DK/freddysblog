---
layout: post
title: "hyperv isolation to the rescue!"
date: 2020-02-14 12:18:49
categories: ["Docker", "NavContainerHelper"]
tags: ["Docker", "Security Upate"]
permalink: /2020/02/14/hyperv-isolation-to-the-rescue/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

# Update

More information on this topic can be found here: [http://freddysblog.com/2020/02/26/the-world-after-february-18th/](http://freddysblog.com/2020/02/26/the-world-after-february-18th/)

# February 11th 2020

On this date, the February security updates for Windows was released, and over the next days, Windows 10 computers and Windows Servers all over the world would receive this update. I am a true believer in securing my Windows Computers and my Windows Servers and would never leave my servers unprotected so I follow the guidelines and update my machines.

99% of the time this works flawlessly and just once in a while – something goes wrong. This day was this time. Not that the update failed, it didn’t – it was applied nicely, but it had some negative side effects on people running Docker on Windows (or at least people running NAV and Business Central containers on Windows).

# The discovery

The first indicator I got of this issue was that my next major pipeline failed

![nextmajor](/assets/images/2020/hyperv-isolation-to-the-rescue/nextmajor.png)

Looking at the issue Wednesday, I could see that the container was created successfully and compilation of my app failed:

![compileapp](/assets/images/2020/hyperv-isolation-to-the-rescue/compileapp.png)

\-alcOutput was a parameter I added in NavContainerHelper 0.6.4.26 and immediately my attention was on that. Tried to repro locally with no luck – everything worked fine and no other issues was reported anywhere and no matter how much I looked at the code – it couldn’t be that change. I even ended up sending a message to the Modern Dev. team asking them whether they changed the output from ALC.EXE. They said no!

The issues came on Thursday. Some reported that they couldn’t sign apps using the Sign-AppInNavContainer function. Other people couldn’t compile. Other people couldn’t create the container, Web Client wouldn’t install etc. etc. etc. issues, e-mails, phone calls with different problems had no end.

Thursday late afternoon I finally found out that the February security update was to blame. My build agent for the BingMaps project was updated on Tuesday – and my next major build started failing Wednesday morning. Our Docker image build servers was updated on Wednesday causing them to create images, which didn’t work properly, which caused the nightly build between Wednesday and Thursday to basically invalidate the latest insider builds.

Some people were running images which didn’t work, some people were running good images on servers which had been updated and now failed compilation, signing, SQL stuff etc. etc. – other people didn’t see the problem at all – only with the insider builds.

I even tweeted that the first person who could point me to the direction of the error would win a beer. Even that didn’t resolve the issue…

# The root cause

With the February update on the host, attempting to run executables inside the container might randomly fail if you are running process isolation. It seems like there are other problems with the February update when used in the container as well, but we didn’t uptake the February update inside our images yet.

It wasn’t due to unblock-file and I do not have a fix for this. Running things in hyperv isolation works – uninstalling February Security Update also works.

I have changed all our build server to run all containers using hyperv isolation and request a rebuild of the latest insider builds – should be done in a few hours.

This should take care of the images.

# What you need to do

If you have pulled insider images the last two days, you need to pull new ones. Latest master build (next major) is 16.0.11119.0, latest 15.x build (next minor) is 15.4.40820.0 (should be ready within the next hour or two).

Now you might think – isn’t next minor 15.3 – yes, that is correct, but that has branched off for release and we don’t build docker images from release branches. If you want something close to what becomes 15.3 you need to use Get-BcContainerImageTags from bcinsider.azurecr.io/bcsandbox and grab the latest 15.3 image.

You also need to run your containers under hyperv isolation (add **\-isolation hyperv** to New-BCContainer) or you need to uninstall February security update.

Note that using hyperv containers is known to cause issues when using non-unicode apps. A lot of effort is put into NavContainerHelper to make sure that text files are handled correctly, but you might still have issues. If you use containers for C/AL development, best option might be to uninstall February Security Update.

**Sorry for the inconvenience!**

_**Freddy Kristiansen**_  
Technical Evangelist
