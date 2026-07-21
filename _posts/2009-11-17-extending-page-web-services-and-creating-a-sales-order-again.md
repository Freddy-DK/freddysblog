---
layout: post
title: "Extending page Web Services (and creating a Sales Order again)"
date: 2009-11-17 06:07:35
categories: ["Archive"]
tags: ["C#", "NAV 2009 SP1", "Page Extension", "Sales Order", "Web Services"]
permalink: /2009/11/17/extending-page-web-services-and-creating-a-sales-order-again/
---

It has been working in the same way since NAV 2009, but I still get asked often how this works, so why not write up a quick post on this. I also realize that my prior post on how to create Sales Orders through Web Services was made very complex due to compatibility with NAV 2009.

This post only works in NAV 2009 SP1 and will show how to extend the Order page with a Post function and show how to Create a Sales Order from C# and post it.

### Extending the page

First of all, we need to create a codeunit with the function, we want to add to the Order page.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ExtendingpageWebServicesandcreatingaSale_561D/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ExtendingpageWebServicesandcreatingaSale_561D/image_2.png)

Then we expose this codeunit with the same name as the page we want to extend, without putting a check in the published column

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ExtendingpageWebServicesandcreatingaSale_561D/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ExtendingpageWebServicesandcreatingaSale_561D/image_4.png)

**Note:** All functions in the codeunit needs to have the first parameter be of the same type as the base record as the page you want to extend, else the page will no longer be available and you will get an error in the event log on the Service Tier.

Now taking a look at the WSDL in a browser will show us the new function as a first class citizen

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ExtendingpageWebServicesandcreatingaSale_561D/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ExtendingpageWebServicesandcreatingaSale_561D/image_6.png)

and we can start using this.

### Creating a Sales Order through Web Services

This might seem like repeating myself from a prior [post](http://blogs.msdn.com/freddyk/archive/2009/05/28/handling-sales-orders-from-page-based-web-services-in-nav-2009sp1-and-rtm.aspx), but that post did contain a lot of other information, which really isn’t necessary if you only target SP1.

Creating an order is a 3 step process:

1.  Create the Order Header
2.  Fill out the Order Header and create the Order lines
3.  Fill out the Order lines

### Creating the Order header

Is really simple

Order\_Service service = new Order\_Service();  
service.UseDefaultCredentials = true;

Order order = new Order();  
service.Create(ref order);

After this we have a Order Number and an empty order – exactly like leaving the order No. field on the Sales Order Page.

### Fill out the Order Header and create the Order lines

In this sample I will just fill out the Sell\_to\_Customer\_No – a number of the other Order Header fields will be auto-updated when updating the order

order.Sell\_to\_Customer\_No = “10000”;

Then we need to create the Order lines – in this sample I will create 5. BTW – It is NOT trivial to add an order line after the fact, so I suggest you add the needed number of lines in one go:

order.SalesLines = new Sales\_Order\_Line\[5\];  
for (int i = 0; i < 5; i++)  
order.SalesLines\[i\] = new Sales\_Order\_Line();  
service.Update(ref order);

### Fill out the Order lines

In this sample, I will just create 5 lines with green ROME guest chairs.

for (int i = 0; i < 5; i++)  
{  
order.SalesLines\[i\].Type = OrderPageRef.Type.Item;  
order.SalesLines\[i\].No = “1960-S”;  
order.SalesLines\[i\].Quantity = 1;  
}  
service.Update(ref order);

That’s it – the order is created and you can find it in the Client.

### And at last… – Post the order

Having created the order, now it is time to post the order

service.PostOrder(order.Key);

As you can see, the function takes a Record parameter, but we give it a Key.

Note, that calling a function does not make an implicit Update – meaning that if you have done changes to the record in C# and call the function, you will get an error when calling update later. Reason – the PostOrder function has changed the record and will tell you that the record was changed by another user.

After calling a function on a page you will need to Re-read the record if you need to do more work.

That’s it

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
