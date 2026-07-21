---
layout: post
title: "A “new” learning opportunity – the Hackathon at NAVUG Focus 18"
date: 2018-05-16 04:36:22
categories: ["AL Development"]
tags: ["AL", "Azure", "Hackathon", "VS Code"]
permalink: /2018/05/16/a-new-learning-opportunity-the-hackathon-at-navug-focus-18/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

I know, a Hackathon is not something new. Hackathons have existed at least half a decade, but how can a Hackathon be a learning opportunity?

Typically we see a Hackathon as an event where people get together to create some prototype or proof of concept of an idea, but the Wikipedia description of a Hackathon is actually as simple as:

_“A hackathon, a hacker neologism, is an event when programmers meet to do collaborative computer programming.”_

# The idea

A few months ago, Mark ([@**GatorRhodie**](https://twitter.com/GatorRhodie)) contacted me and told me about [NAVUG FOCUS 18](https://www.ugfocus.com/home). He told me that one of the themes of this years NAVUG FOCUS is “A Brave New World” – about AL development, Docker, VS Code, Azure etc.

He told me that he wanted to conduct a Hackathon during the event. We had a few calls and discussed various approaches and ended up agreeing that the best approach would be to create some ideas/challenges, which people can work on in groups if they don’t have ideas of their own. The challenges should be things of common usage, things that people can go back and look at as a reference on how to do things.

We brainstormed some ideas and with great help of Jesper ([@**JesperSchulz**](https://twitter.com/JesperSchulz)) we ended up with a set of challenges, which we think are appropriate for the event.

# The event

The event takes place on Monday, may 21st evening from 5:30PM to 11PM (not sure how my jetlag is going to cope with that:-)) and the idea is that people can choose one or more of “our” challenges to work on – or they can work on ideas of their own.

For every challenge there is a description, an expected result, some steps, some hints and some cheat sheets. We will have some people in the room to help out if people get stuck, but the primary idea is, that people help each other. People working on “our” challenges can request a cheat sheet if they cannot figure out how to solve a specific issue.

Depending on the outcome of this event, we might use the same mechanism at other conferences. I am also considering whether our challenges can be made public somehow so that people can conduct their own Hackathon events for social learning/programming.

# A sample challenge

Below, you will find one of the challenges in its full form (but without the cheat sheets). This challenge is a level 1 challenge.

# Auto-fill company information on the customer card

As a new customer is entered in Dynamics 365 Business Central, the user can decide to enter a domain name instead of the name, which leads to the system looking up the information for the company associated with this domain name from a Web Service and filling out the remaining fields on the customer card with information obtained from the Web Service.

To complete this challenge, you will need:

-   A Dynamics 365 Business Central Sandbox Environment
    -   Use [http://aka.ms/bcsandbox](http://aka.ms/bcsandbox) to create an Azure VM if you do not have a sandbox environment.
-   Visual Studio Code with the AL Extension installed
    -   Azure VMs will have VS Code pre-installed
-   An API Key from [http://www.fullcontact.com](http://www.fullcontact.com)

Expected result:

[![](/assets/images/2018/a-new-learning-opportunity-the-hackathon-at-navug-focus-18/23925-challenge1-1.gif)](/assets/images/2018/a-new-learning-opportunity-the-hackathon-at-navug-focus-18/23925-challenge1.gif)

Steps:

-   Create an empty app
-   Create a page extension for the customer card
-   On the _OnAfterValidate_ trigger on the _Name_ field, check whether the entered value is a domain name
-   Ask the user whether he wants to lookup information about the company associated with this domain name
-   Call the fullcontact Web API and assign field values

Hints:

-   In VS Code, use Ctrl+Shift+P and type AL GO and remove the customerlist page extension
-   Use the _tpageext_ snippet
-   Use _EndsWith_ to check whether the name is a domain name
-   Use the _Confirm_ method to ask whether the user want to download info
-   Use _HttpClient_ to communicate with the Web Service
-   Use Json types (_JsonObject_, _JsonToken_, _JsonArray_ and _JsonValue_) to extract values from the Web Service result

Cheat Sheets:

-   Create an empty app
-   Create a page extension
-   Code for communicating with Web Service
-   Update the customer

See you in Indianapolis.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
