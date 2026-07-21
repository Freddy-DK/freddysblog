---
layout: post
title: "Integration to Virtual Earth – Part 3 (out of 4)"
date: 2009-03-23 16:04:03
categories: ["Archive"]
tags: ["Latitude", "Longitude", "MAP", "Mappoint", "NAV 2009", "Virtual Earth", "Web Services", "XMLPort"]
permalink: /2009/03/23/integration-to-virtual-earth-part-3-out-of-4/
---

If you haven’t read [Part 1](http://blogs.msdn.com/freddyk/archive/2009/03/18/integration-to-virtual-earth-part-1-out-of-4.aspx) and [Part 2](http://blogs.msdn.com/freddyk/archive/2009/03/23/integration-to-virtual-earth-part-2-out-of-4.aspx), you should do so before continuing here. In Part 1 we saw how to add geographical information to your customer table and how to connect to the Virtual Earth geocode web service, and in part 2 we created a simple web site showing a map and adding pushpins.

Now we want to add an action to the customer card, showing us the location of the customer and other customers in the surrounding area.

### Parameters to the website

The web site we create in part 2 takes 3 optional parameters: Latitude, Longitude and Zoom

If you try to type in

[http://localhost/mysite/default.htm?latitude=0&longitude=0&zoom=6](http://localhost/mysite/default.htm?latitude=0&longitude=0&zoom=6)

you will get something like this:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart3outof4_B6EE/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart3outof4_B6EE/image_2.png)

Which is location 0,0 and a zoom factor of 5.

Now the only thing we need to do in NAV is to create an action, which launches this web site with the location set to the coordinates of the customer and set the zoom factor to e.g. 10. The web site will then connect back to web services and place push pins where the customers are and that part already works.

### Adding the action

In the customer card open the site actions and add an action called View Area Map:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart3outof4_B6EE/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart3outof4_B6EE/image_6.png)

Switch to code view and add the following line to the action:

`HYPERLINK('http://localhost/mysite/default.htm?latitude='+FORMAT(Latitude,0,2)+'&longitude='+FORMAT(Longitude,0,2)+'&zoom=10&#8217;);`

FORMAT(xxx,0,2) ensures that the latitude and longitude are in the correct format and when you launch the action on the Cannon Group you will get:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart3outof4_B6EE/image_thumb_4.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart3outof4_B6EE/image_10.png)

A nice map of the surroundings and if we hover over the customers we can see more information about the customer.

I did not upload the binaries for this to any site – since it really is just adding this one line of code to a new action.

Part 4 of the Virtual Earth Integration will have to wait a little. It uses SP1 features and we are not allowed to blog about SP1 features until the Statement of Direction is published and that should be the case end of March 2009 – so stay tuned for the round-up of Virtual Earth Integration beginning of April 2009.

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
