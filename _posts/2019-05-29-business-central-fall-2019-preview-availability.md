---
layout: post
title: "Business Central Fall 2019 Preview Availability"
date: 2019-05-29 14:06:09
categories: ["AL Development", "Docker"]
tags: ["AL", "C/AL", "C/AL to AL", "Docker", "Txt2Al"]
permalink: /2019/05/29/business-central-fall-2019-preview-availability/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

There is a lot of excitement and buzz these days and partners have approached me asking for the insider builds from our master branch.

Hold back on the search, we will NOT be creating Docker builds or preview DVDs from our master branch for a little while. Let me elaborate…

# Why are we not providing insider builds from master at the moment?

As you can imagine, the master branch is undergoing some exciting changes and restructuring.

We are in the process of removing the Windows Client, replacing C/AL with AL and making sure we have the tools in place to work with AL

Sometimes when doing significant changes, you take some temporary steps, which causes disturbance, and it is our assessment that releasing insider builds at the moment, would cause unnecessary confusion and double work, which is best to be avoided and dealt with at a later time.

# When will we resume insider builds from master?

We expect that we will be able to resume insider builds sometime over the summer.

We prioritize delivering the right quality and content, rather than hitting any specific date.

Rest assured that you will be notified with blog posts here and tweets with #msdyn365bc when insider builds resumes.

# What would you actually be using the fall 2019 insider builds for?

This might be a strange question, but please note, that you cannot use insider builds for stuff like converting your C/AL codebase to AL.

As C/AL is not part of fall 2019 – you will need to perform the step from C/AL to AL on a CU of spring 2019 release.

If you are starting a new AL extension, there is no reason that you cannot build this on spring release – fall release will not really give you anything at this time.

If you want to start a new code customized AL solution on fall release, then you might want to rethink – code customized AL should only be used by partners who cannot create AL extensions – primarily due to legacy code customizations.

If you want to see how the base app looks as AL, follow the blog posts from April 15th here: [https://freddysblog.com/category/al-development/](https://freddysblog.com/category/al-development/)

# Which CU should you actually use for converting your C/AL to AL?

Now that is a good question. The blog post above is using spring 2019 RTM, but as I am writing this blog post, spring 2019 CU1 is brewing on our build servers.

We know that CU1 contains some improvements (especially on memory).

We also assume that we will get feedback from people doing conversion from C/AL to AL with spring release and we will be fixing these and backport these fixes to cumulative updates to spring 2019 release.

So,… – start now (with CU1), and keep using the latest cumulative update for the latest in Txt2AL support.

Give Microsoft feedback if something doesn’t work as expected and let’s work together on making the transformation from C/AL to AL easier.

# Finally…

When we then start shipping fall 2019 preview, you can (and should) use this for testing your AL extensions.

For code customized AL, you will have to merge your changes from spring 2019 AL to fall 2019 AL – much like you did with C/AL.

I don’t think partners will be doing that on a daily basis with fall 2019 insider builds.

If you have code customized AL, you should really work on transforming as much of your code customized AL solution to extensions to save time and energy – this transformation should be done on spring 2019 release and will make the effort of moving to fall 2019 easier.

Note, that you don’t really gain anything from waiting until spring 2019 CU5 or CU6, the longer you wait, the less chances you have of influencing anything or getting anything fixed.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
