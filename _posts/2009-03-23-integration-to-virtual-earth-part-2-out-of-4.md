---
layout: post
title: "Integration to Virtual Earth – Part 2 (out of 4)"
date: 2009-03-23 14:10:16
categories: ["Archive"]
tags: ["Bookmark", "Geocode", "Javascript", "Latitude", "Longitude", "MAP", "Mappoint", "NAV 2009", "Virtual Earth", "Web Services", "XMLPort"]
permalink: /2009/03/23/integration-to-virtual-earth-part-2-out-of-4/
---

If you haven’t read [Part 1](http://blogs.msdn.com/freddyk/archive/2009/03/18/integration-to-virtual-earth-part-1-out-of-4.aspx), you should do so before continuing here. In Part 1 we saw how to add geographical information to your customer table and how to connect to the Virtual Earth geocode web service. Now we want to put this information to play.

First of all, we all know Virtual Earth (if not – try it at [http://maps.live.com](http://maps.live.com)). Less commonly known is it, that the map control used in maps.live.com actually is available for public usage in other web pages.

### The Virtual Earth Interactive SDK

If you navigate to [http://dev.live.com/virtualearth/sdk/](http://dev.live.com/virtualearth/sdk/ "http://dev.live.com/virtualearth/sdk/") you will get a menu in the left side looking like this:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_2.png)

and a map looking like this:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_6.png)

(depending on where in the world you are)

Try looking at the different tabs and you can see the source code for getting a map like this – and you can select other menu items.

Try clicking at the menu item for “Show a specific map”. The map now shows the space needle in Seattle and if you click the source code tab you will see the code for creating that map:

<!DOCTYPE html PUBLIC “-//W3C//DTD XHTML 1.0 Transitional//EN” “[http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd”&gt](http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd”&gt);  
<html>  
<head>  
<title></title>  
<meta http-equiv=”Content-Type” content=”text/html; charset=utf-8″>  
[http://dev.virtualearth.net/mapcontrol/mapcontrol.ashx?v=6.2](http://dev.virtualearth.net/mapcontrol/mapcontrol.ashx?v=6.2)

var map = null;  
function GetMap()  
{  
map = new VEMap(‘myMap’);  
map.LoadMap(new VELatLong(47.6, -122.33), 10 ,’h’ ,false);  
}  

</head>  
<body onload=”GetMap();”>

</body>  
</html>

So – HTML and Javascript again. A few things which are important to know is that this line:

[http://dev.virtualearth.net/mapcontrol/mapcontrol.ashx?v=6.2](http://dev.virtualearth.net/mapcontrol/mapcontrol.ashx?v=6.2)

Imports the mapcontrol library from the Virtual Earth Web Site and allows you to use all the controls and functions exposed by this library (like a C# reference and using statement).

var map = null;  
function GetMap()  
{  
map = new VEMap(‘myMap’);  
map.LoadMap(new VELatLong(47.6, -122.33), 10 ,’h’ ,false);  
}  

Is the actual code – for manipulating the map. 47.6, –122.33 should be latitude and longitude for the Space Needle, 10 is the Zoom level, ‘h’ is the style (h==hybrid) if you click the reference tab on the dev sdk window you can find the documentation to [LoadMap](http://msdn.microsoft.com/en-us/library/bb412546.aspx) and other stuff.

Turns out that the Space Needle is not exactly in that position – the right position would be 47.620564, -122.349577 – anyway you get the picture,

The instantiation of the Map also tells you where on the web site, the map should be placed (‘myMap’) which references a

area in the body:

This

area specifies the location, flow definition and the size of the control.

The last thing we need to know is how our function (GetMap) in Javascript is executed – and that happens as an event handler to the onload event on the body tag.

Note that Javascript is executed on the Client and there is no serverside code in this example.

Knowing that we can communicate with NAV Web Services from Javascript (as explained in this [post](http://blogs.msdn.com/freddyk/archive/2008/11/24/search-in-nav-2009-part-3-out-of-3.aspx)) we should be able to create a site, which loads a map and displays pushpins for all our customers – more about this in a second…

### Deployment

Knowing that there are a million different ways to deploy web sites I will not go into detail about how this should be done – but only explain how it can be done (aka what I did on my laptop).

What I did was to install IIS (Internet Information Server) on my laptop. After this I fired up Visual Studio, selected File –> New –> Web Site and specified that I wanted to create an Empty Web Site called **[http://localhost/mysite](http://localhost/mysite)**

In the empty web site I right clicked on the project and selected add new item, selected HTML page and called it default.htm:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_8.png)

C# / Visual Basic choice really didn’t matter as I wasn’t going to make any server side code.

In this default.htm I replaced the content with the following HTML

[http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&#8221](http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd&#8221);\>

</body>  
</html>

and when I open the site in my browser i get:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_thumb_6.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_14.png)

[

Voila, the Space Needle.

So far so good and if you succeeded in showing this map in a browser from your own Web Site – we know that all the basics are in place.

### What’s needed in NAV

Nothing comes for free – and just because we added the latitude and longitude to the customer table doesn’t mean that we can query on them.

We need a function in NAV, exposed as a Web Service, which we can call and ask for all customers in a rectangle of the Earth. Now sceptics might say that there are no such thing as a rectangle of the earth – but programming towards the Virtual Earth API – there is. For returning the customers through Web Services I have created a XML port:

](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_14.png)

[](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_14.png)[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_thumb_7.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_16.png)

and the two lines of Code behind this

**Customer – Export::OnAfterGetRecord()  
**ref.GETTABLE(“<Customer>”);  
\_Bookmark := FORMAT(ref.RECORDID,0,10);

for making sure that the bookmark is correctly formatted.

The function that we want to expose as a webservice looks like this:

**GetCustomersWithin(latitude1 : Decimal;latitude2 : Decimal;longitude1 : Decimal;longitude2 : Decimal;VAR result : XMLport CustomerLocation)  
**customers.SETRANGE(Latitude, latitude1, latitude2);  
customers.SETRANGE(Longitude, longitude1, longitude2);  
result.SETTABLEVIEW(customers);

So this is the reason why it is good to remember to have a key on the latitude and longitude fields in the customer table.

I just added this function to the NavMaps codeunit and made sure that the other function (from post 1) is private – meaning that this function is the only function exposed and you need to expose the NavMaps codeunit as a web service in the Web Service table

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_thumb_8.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_18.png)

I called it Maps – and this is used in the code below.

### Combining things

In the menu you can find code for how to add shapes (pushpins and other things). You can also find code for how to subscribe to events and we are going to do this for two of the events from the mapcontrol (onendzoom and onendpan) as they change the current viewing rectangle of the control.

<!DOCTYPE html PUBLIC “-//W3C//DTD XHTML 1.0 Transitional//EN” “[http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd”](http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd")\>  
<html>  
<head>  
<title></title>  
<meta http-equiv=”Content-Type” content=”text/html; charset=utf-8″ />  
[http://localhost:7047/DynamicsNAV/WS/CRONUS\_International\_Ltd/&#8221;;](http://localhost:7047/DynamicsNAV/WS/CRONUS_International_Ltd/&#8221)  
    var XMLPortResultNS = ‘urn:microsoft-dynamics-nav/xmlports/customerlocation’;  
var XMLPortResultNode = ‘Customer’;  
var resultSet;

    // Event handler for endzoom event  
function EndZoomHandler(e) {  
UpdatePushPins();  
}

    // Event handler for endpan event  
function EndPanHandler(e) {  
UpdatePushPins();  
}

    // Initialize the map  
function GetMap() {  
map = new VEMap(‘myMap’);

        // center and zoom are passable as parameters  
var latitude = parseFloat(queryString(“latitude”, “0”));  
var longitude = parseFloat(queryString(“longitude”, “0”));  
var zoom = parseInt(queryString(“zoom”, “2”));

        // use normal dashboard  
map.SetDashboardSize(VEDashboardSize.Normal);  
// load the map  
var position = new VELatLong(latitude, longitude);  
map.LoadMap(position, zoom, ‘r’, false);  
// hook events  
map.AttachEvent(“onendzoom”, EndZoomHandler);  
map.AttachEvent(“onendpan”, EndPanHandler);  
// Place pushpins  
UpdatePushPins();  
}

    // Update all pushpins on the map  
function UpdatePushPins() {  
map.DeleteAllShapes();  
// Get the view rectangle  
var botlft = map.PixelToLatLong(new VEPixel(1, myMap.offsetHeight-1));  
var toprgt = map.PixelToLatLong(new VEPixel(myMap.offsetWidth-1, 1));

        // Get customers within rectangle  
GetCustomersWithin(botlft.Latitude, toprgt.Latitude, botlft.Longitude, toprgt.Longitude);  
if (resultSet != null) {  
i = 0;  
while (i                 var shape = new VEShape(VEShapeType.Pushpin, new VELatLong(resultSet\[i\].childNodes\[2\].text, resultSet\[i\].childNodes\[3\].text));  
shape.SetTitle(resultSet\[i\].childNodes\[0\].text + ” ” + resultSet\[i\].childNodes\[1\].text);  
shape.SetDescription(”

” +  
”  
” +  
”  
” +  
”  
” +  
”

<table><tbody><tr><td>Contact</td><td></td><td>” + resultSet[i].childNodes[5].text + “</td></tr><tr><td>Phone</td><td></td><td>” + resultSet[i].childNodes[6].text + “</td><td>Sales</td><td></td><td>” + resultSet[i].childNodes[7].text + “</td><td>Profit</td><td></td><td>” + resultSet[i].childNodes[8].text + “</td><td colspan="3"><a href="////runpage?page=21&amp;mode=view&amp;bookmark=&quot; + resultSet[i].childNodes[4].text + &quot;">Open Customer Page</a></td></tr></tbody></table>

“);  
map.AddShape(shape);  
i++;  
}  
}  
}  
// Helper function for querying parameters to the site  
function queryString(parameter, defaultvalue) {  
var loc = location.search.substring(1, location.search.length);  
var param\_value = false;  
var params = loc.split(“&”);  
for (i = 0; i             param\_name = params\[i\].substring(0, params\[i\].indexOf(‘=’));  
if (param\_name == parameter) {  
param\_value = params\[i\].substring(params\[i\].indexOf(‘=’) + 1)  
}  
}  
if (param\_value) {  
return param\_value;  
}  
else {  
return defaultvalue;  
}  
}

    // Get Base URL  
function GetBaseURL() {  
return defaultURL;  
}

    // Get Customers within specified rectangle by connecting to NAV WebService  
function GetCustomersWithin(latitude1, latitude2, longitude1, longitude2) {  
resultSet = null;  
try {  
// Instantiate XMLHTTP object  
xmlhttp = new ActiveXObject(“Msxml2.XMLHTTP.6.0”);  
xmlhttp.open(“POST”, GetBaseURL() + “Codeunit/Maps”, false, null, null);  
xmlhttp.setRequestHeader(“Content-Type”, “text/xml; charset=utf-8”);  
xmlhttp.setRequestHeader(“SOAPAction”, “GetCustomersWithin”);

            // Setup event handler when readystate changes  
xmlhttp.onreadystatechange = function() {  
// Inline function for handling response  
if ((xmlhttp.readyState == 4) && (xmlhttp.Status == 200)) {  
var xmldoc = xmlhttp.ResponseXML;  
xmldoc.setProperty(‘SelectionLanguage’, ‘XPath’);  
xmldoc.setProperty(‘SelectionNamespaces’, ‘xmlns:tns=”‘ + XMLPortResultNS + ‘”‘);  
resultSet = xmldoc.selectNodes(‘//tns:’ + XMLPortResultNode);  
}  
}  
// Send request  
xmlhttp.Send(‘[http://schemas.xmlsoap.org/soap/envelope/&#8221](http://schemas.xmlsoap.org/soap/envelope/&#8221);\>’ + latitude1 + ” + latitude2 + ” + longitude1 + ” + longitude2 + ”);  
}  
catch (e) {  
alert(e.message);  
}  
}

  
</head>  
<body onload=”GetMap();” style=”margin:0; width:100%; height:100%; overflow: hidden”>

</body>  
</html>

I will let the code speak for itself and direct attention to some of my other Javascript posts and/or information generally available on Javascript on the Internet.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_thumb_9.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart2outof4_9B82/image_20.png)

The defaultURL variable in the start should be set to the URL of your web services listener and if you want to have more fields when you hover over a customer on the map, you will need to add these fields to the customerLocation XML port and add them to the SetDescription call in the code.

I have only tried this with the demo data – and of course you could imagine that if you have a very high number of customers the pushpin placement routine would be too slow. If this is the case, we can either limit the rectangle (if it spans too much – then ignore) or we can check the size of the returned and only place pushings for the firs xx number.

### Epilog

When I said nothing comes for free, it wasn’t entirely true – this blog post is free, the usage of the information in this post is free include code snippets etc. – so some things are free:-)

I realize that this was a LOT of non-NAV code – but I do think it opens up some very compelling scenarios for integration when we combine the strength of different products using Web Services.

As always the NAV objects and the default.htm from above is available for download [here](http://www.freddy.dk/NavMaps2.zip).

In step 3 I will show how we can add an action to the Customer Card for showing the area map for a given customer – and you probably already know how to do this, if you have followed this post carefully.

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
