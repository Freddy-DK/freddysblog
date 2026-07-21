---
layout: post
title: "The Customer MAP demo"
date: 2009-03-15 10:29:09
categories: ["Archive"]
tags: ["MAP", "Virtual Earth"]
permalink: /2009/03/15/the-customer-map-demo/
---

My newest demo was shown at the NAV Partner Keynote and also on the NAV Customer General session.

I also did a very short walkthrough of the demo at the NAV06 concurrent session at Convergence – and I promised a number of people that I would add a walkthrough of how the demo is done on my blog.

This post is only an appetizer – the real stuff is going to be out here over the next couple of days – and will be a 3 step walkthrough.

1.  How to geocode the customers in your customer database using a Microsoft Virtual Earth Web Service
2.  How to use this geocode information from an intranet application using Microsoft Virtual Earth
3.  Adding an action to the Customer page to view other customers in the area

A screenshot of the demo can be seen here

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/TheCustomerMAPdemo_6874/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/TheCustomerMAPdemo_6874/image_2.png)

and basically what happens is that I added a Latitude and a Longitude field to the Customer table – and created a small automation object, which can geocode adresses. Having this information in the customer table enables a lot of cool scenarios – the above one is just the first simple one, that springs into mind.

Beside the geocoding the above solution requires a Web Service codeunit to be exposed and a small intranet application.

The Web Service codeunit contains a function that returns all the customers in a rectangle of the world (given by the bottom left and the top right latlong coordinates) – and the small intranet application just calls this webservice every time you zoom or pan the map.

Stay tuned

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
