---
layout: post
title: "TimeStampServer – you had one job!"
date: 2020-12-31 06:59:23
categories: ["BcContainerHelper", "CI/CD"]
tags: ["BcContainerHelper", "Sign-AppInBcContainer", "Signing", "TimeStampServer"]
permalink: /2020/12/31/timestampserver-you-had-one-job/
---

The important part of this blog post: _**Please make sure to upgrade to BcContainerHelper 1.0.18 or reconfigfure the timestampServer in your existing version.**_

December 30th 2020 we got a number of apps for validation failing due to a wrong timestamp signature. Some partners discovered this before submitting and filed an issue on github [here](https://github.com/microsoft/navcontainerhelper/issues/1579), complaining about this error: “**The timestamp signature and/or certificate could not be verified or is malformed.**“

I was enjoying a nice walk in the forest with my dogs, but my phone would reveal increased activity on github and emails from partners running into this strange issue.

# The Signing process

I don’t now the inner workings of the signtool, but I do know that it uses a code signing certificate, a digest algorithm, a timestamp server and a crypto provider for Business Central to sign a file. Every one of these have a job to do in the signing process and the task of the timestamp server is to return a valid and accurate timestamp for the signing process.

Yesterday, the timestamp server we have been using as the default timestamp server since like forever, started returning malformed timestamps – really – you had one job! The big problem was really that everything seemed ok, no error, but the app could not be published afterwards. Just tested today (December 31st 2020) – and the problem persists.

I did not go into much investigation as to why the timestampserver was failing (it might be that it is no longer active), instead I tested a different timestamp server and fortunately in BcContainerHelper, the timestampserver is a configuration setting, so it was fairly easy to change.

BcContainerHelper 1.0.18 uses a different timestamp server ([http://timestamp.digicert.com](http://timestamp.digicert.com/)) and it uses the SHA256 digest algorithm instead of the outdated SHA1 algortihm used before as the default.

So, if you are using BcContainerHelper to sign your app, please update BcContainerHelper to 1.0.18 – or configure it to use a different timestampserver, else your app will fail submission.

# BTW – Run-AlValidation did catch this

Partners who did run Run-AlValidation before submitting to AppSource Validation did catch this themselves as Run-AlValidation will try to publish and install the app and this did fail.

Sorry for the inconvenience, but these things are out of my control.

**_Freddy Kristiansen_**  
Technical Evangelist
