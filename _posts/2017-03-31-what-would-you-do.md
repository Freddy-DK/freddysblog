---
layout: post
title: "What would you do?"
date: 2017-03-31 07:31:15
categories: ["Freddy"]
tags: ["Business Model", "Customizations", "Event Based Architecture", "Events", "Extensions", "NAV 2017", "NAV Tenerife", "Visual Studio Code"]
permalink: /2017/03/31/what-would-you-do/
---

Lately, a number of  ISVs have asked me this question:

_If you where me, what would you do?_

To give the question a little context, we are talking about Solutions, Customizations, Extensions, Modifications or whatever name you use. We are talking On-prem, Hosted, Managed Service, Dynamics 365 for financials.

What is the magic bullet? What should the ISV do in order to make sure that you are ready for the future, without wasting a lot of time.

The answer can really be given from a technical or from a business model perspective, but since I am a Technical Evangelist (and not a Business Model Evangelist:-)), I have decided to see things from the technical side. I claim, that even if you would see this from the business model side, you will reach the same conclusions.

This post is my response to these ISVs.

### Definitions!

Let me first start by defining a number of different “states” an ISV solution can be in. This list is not be complete, people can be in between or have more than one solution, but work with me on this, don’t get caught up on the details:

1.  Solutions are customer specific, every customer is running on their own version of the app on their own server (on-prem or hosted) and the customer pays to get upgraded to new versions of NAV. Best practices are followed as to tagging code customizations and a merge tool is used to merge the latest update from Microsoft with the customer solution when the customer decides to do so.
2.  Like 1 – except the solution is a standard solution following the guidelines of the Road to Repeatability program. Customers still decide when to upgrade to a new version, but when they do, they get the latest version, which is the same for all.
3.  Like 2 – except that the solution is running in a multi-tenant environment and all customers are in fact running on the same versions of the app. When upgrading, a new multi-tenant environment is spun up and customers are moved over.
4.  Like 2 or 3 – except that the solution is using event based architecture, using events to modify base behavior where possible and if no events are available, a new event is introduced in the base app as the only code modification to merge. A merge tool or the merge Cmdlets are used to automagically merge updates from Microsoft with the solution.
5.  Like 4 – except that the solution is actually packaged as an extension, meaning that all customers will be using the same base app and the individual tenants can install the extension if they like. Upgrade is done by rolling the extension off and on again, restoring the data where needed.
6.  Like 5 – except that the extension is Saasified and uploaded to AppSource and Cutomers using Dynamics 365 for Financials (always multi-tenant) can install the extension, try it out for free and the app can request a registration and/or payment to use full functionality. Upgrades are handled automagically every month.

These six models really showcase the transformation solutions in NAV has taken – just over the last approx. 5 years and no wonder that it can be hard to follow along – and now Microsoft is talking about Extensions v2 and Visual Studio Code????

You can probably easily identify where you are on the stairway – and maybe you even have a clear plan on where to do?

[![stairways](/assets/images/2017/what-would-you-do/f6651-stairways-1.png)](/assets/images/2017/what-would-you-do/f6651-stairways.png)

### Challenges

For every step you take up the stairway, you will meet challenges. Some are easy to overcome, some harder. Some challenges might even be to the point, where you decide not to take that step, but please be aware, that you normally cannot skip any steps on a stairway. You need somehow to get over it or be stuck below that step.

Here are a few of the challenges you will meet on your way:

step 3 – multi-tenancy means people cannot write to the app database, some scenarios are harder.

step 5 – real extensions means that you will have to live with the events that are exposed by the base app, you cannot modify and add new events.

step 6 – no server side DLLs, no file access, only limited .net interop, no Windows Client.

and when we talk about extensions v2 and Visual Studio Code, there will be more challenges, which Microsoft will try to provide good workarounds for (either by extending the language or making built in support for stuff like Azure Functions and Web APIs)

Note, that it is important for Microsoft to hear about the challenges from Partners. Microsoft cannot foresee every single obstacle every partner will ever run into, so feedback is important, whether it is missing events, missing functionality or other things that prevents partners from taking the next step up the stairways.

### So, what would I do?

I assume that you want to move up the stairway – all the way to the top of the [**_stairway to heaven_**…](https://www.youtube.com/watch?v=9Q7Vr3yQYWQ) (sweet music:-))

I assume that you eventually want to be standing on the top with an app in AppSource but what should you do to get there? Should you wait until Dynamics 365 for Financials is in your country? and what if that date isn’t even planned yet?

If my solution was on step 1-3 on the stairway – I would refactor it in order to get to **step 4 – Event Based Architecture**.

Getting to step 4 will put you in a better place and doesn’t really add any challenges that cannot easily be overcome. It is just a good way of writing your code, which makes updates and merges easier and will enable you to identify missing events and know more about what it takes to move to an extension. This work can be done in NAV 2017 using the Classic Development environment.

Once at step 4 or above you can decide to continue up the stairway using Extensions v1 and the classic development environment or you can switch to Extensions v2 and use Visual Studio Code – and this is really determined by timing.

### How does your code look at step 4

When refactoring your code for step 4 you should think about the next steps, but you shouldn’t limit yourself due to challenges you see today.

-   Use events to intercept base app functionality and add your own behavior.
-   Introduce new events in the base app if you cannot find events that meets your needs – this will be a code modification causing a delta and a challenge when moving to the next step. Postpone the resolution of that challenge until you take the next step, but check NAV Tenerife to see whether that event is coming – else notify Microsoft about the missing event.
-   If you have code which you know will cause challenges at higher steps (could be Web Services communication, calling a DLL or other things), then encapsulate this in functions and use these functions throughout. Postpone the resolution to that challenges until you take the next step, but check NAV Tenerife to see if and how this challenge can be solved.
-   Do not limit yourself to whats possible in Extensions v1 or in Extensions v2 today. If you meet a challenge, encapsulate the challenge, document, check NAV Tenerife, tell Microsoft and then postpone.
-   Saasify your solution, use assisted setup, wizards, notifications and other tools to make sure, that your app is easy to use and doesn’t require a lot of training hours for users to understand.
-   Make good use of design patterns in your implementation.

You can (and should) really perform a lot of the refactoring of your solution at step 4, when refactoring for the event based architecture. At this step, you can still ship, test and develop your software. You are in a safe haven.

### Extensions v1 vs Extensions v2?

I hope that it is clear by now, that step 4, the event based architecture is the place to get to and that this has absolutely nothing to do with Extensions v1 or Extensions v2. It also has nothing to do with whether or not Dynamics 365 is available in your country – it is just a good step to be on and take a rest. Furthermore… **it is a necessary step.**

The two options you have now is Extensions v1 or Extensions v2. Extensions v1 is out there in NAV 2017 and apps on AppSource today is based on Extensions v1. Extensions v2 is in preview and you probably won’t be able to publish apps on AppSource before summer of 2017 using Extensions v2. Extensions v1 is likely to be deprecated in AppSource some time during 2017.

New events are available in CTP drops of NAV Tenerife – both for Extensions v1 and Extensions v2.

Extensions v1 and Extensions v2 will be supported in NAV Tenerife (on-prem). Extensions v1 will likely be deprecated in a future NAV release.

_Whether Classic NAV Development will be deprecated at some point in time in the future – I have no idea and that is totally unrelated to this blog post._

Extensions v1 has limitations on where you can add code and it is cumbersome to work with. Extensions v2 is still in preview and doesn’t have full functionality yet.

With this info, it isn’t really a question about one or the other. It is more a question on **_when do you move to Extensions v2?_** and **_whether you are going to use Extensions v1?_**

### Should you use Extensions v1 (C/SIDE)?

Maybe you are already on step 5 with extensions v1 or maybe you have an app in AppSource and you are at step 6, then you of course would use extensions v1 until you select to move to extensions v2.

If not, then IMO, you should have a business reason for taking step 5 with extensions v1. That reason could be that you need to ship something to AppSource tomorrow – then your only option really is Extensions v1.

### When should you move to Extensions v2 (Visual Studio Code and In-Client Designer)?

If you haven’t already done so, you should spend some time on the [NAV Developer Preview](http://aka.ms/navdeveloperpreview).

You should familiarize yourself with the new development environment, read the blog posts on the [NAV Team Blog](https://blogs.msdn.microsoft.com/nav/), follow the questions and [issues on GitHub](https://github.com/microsoft/al/issues), Investigate the [samples on GitHub](https://github.com/microsoft/al) and the earlier you get started, the better your chances of influence the shape of Extensions v2.

The conversion tool is likely to be shipped in the April update of the NAV Developer Preview and should be able to help you convert from Step 4, 5 or 6 to the same step using the new Development environment – allowing you to take the final steps in Extensions v2.

The conversion tool is not going to take you all the way, you will probably have to do some manual work too. You might also find that it would be best to refactor a few things in C/SIDE before the conversion to make the conversion run more smooth. Do that, run some test conversions and move over to Extensions v2 and Visual Studio Code once you have a business reason for making the move.

### Conclusion

If you want to move your solution forward and up the stairway, you need to consider a few things.

If you are not there yet, you need to refactor your code to step 4 – Event Based Architecture.

While at step 4, make sure that you have everything in place for the next steps before going there.

-   Identify challenges and engage with app solution providers or Microsoft to solve these.
-   Easy to use user interface (how do users get a WOW experience in just 5 minutes of using your app)
-   Test automation (when upgrades and release are automatic, you need to have high code quality)
-   Sustainable Dev Structure (agile development, source code management, release management etc.)
-   Good alignment between Marketing and Engineering

When you have all these things in place the next steps up the stairway are easy to see and easier to overcome.

I hope this is helpful

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
