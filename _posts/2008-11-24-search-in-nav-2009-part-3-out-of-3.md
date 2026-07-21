---
layout: post
title: "Search in NAV 2009 – Part 3 (out of 3)"
date: 2008-11-24 18:44:52
categories: ["Archive"]
tags: ["BigText", "Bookmark", "Gadget", "Javascript", "NAV 2009", "Search", "URL", "Web Services", "XML", "XSLT"]
permalink: /2008/11/24/search-in-nav-2009-part-3-out-of-3/
---

If you haven’t read [part 2](http://blogs.msdn.com/freddyk/archive/2008/11/24/search-in-nav-2009-part-2-out-of-3.aspx) and [part 1](http://blogs.msdn.com/freddyk/archive/2008/11/23/search-in-nav-2009-part-1-out-of-3.aspx) of the Search in NAV 2009 posts, you should do so before continuing.

This is the 3rd and final part of the Search in NAV 2009 post. In this section I will show how to create a Windows Vista Gadget and have this gadget connect to NAV through Web Services and search in NAV (like the System Tray version in part 2).

We will create an installable Gadget like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_2.png)

and when installed the user should be able to perform searches like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_4.png)

Giving the user the opportunity to click on items and link into NAV 2009.

As you will notice the results window is different from the results window in part 2 and the main reason for this is, that I couldn’t get intra document links (that be <A HREF=”#Customers”> and <A NAME=”Customers”>) to work in a flyout. Every time you would click a link which should reposition yourself in the document – it would reload the page and leave me with a blank page.

After having struggled with this for some hours I decided that that piece was primarily done in order to help keyboard users – and since Gadgets kind of require the mouse I decided to remove the intra document links.

If anybody finds a way to do this, feel free to add a comment and make me smarter! 🙂

I actually think the sample shows that the strategy of having the Web Service just return a result set and have the consumer format this in the way that fits the consumer is the right decision.

### What is a .Gadget file?

If you ever downloaded a file called **Something.Gadget** and opened it, you might get a warning like this

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_thumb_2.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_6.png)

and if you say Install, then the Gadget gets installed – very easy indeed, but what is this?

As a hint – try to rename the file to **Something.Gadget.zip** – and you will see.

The file extension Gadget is known by Windows Sidebar, which will look for a Gadget.xml in the .zip file and if a correct Gadget.xml is present, it will display this installation dialog and if you select to Install, the .zip file is unpacked into a directory under

C:\\Users\\<username>\\AppData\\Local\\Microsoft\\Windows Sidebar\\Gadgets

and add the gadget to the sidebar.

Note that the .zip file should NOT contain the outer directory – only the content.

### And what is the Gadget then?

The Gadget is actually just a small html document, which can contain Javascript, VB Script code or other client side code, supported in html. The first file read by the Sidebar is the Gadget.xml file, which in my Search example looks like:

<?xml version=”1.0″ encoding=”utf-8″ ?>  
<gadget>  
<name>NAV Search Gadget</name>  
<namespace>NAVsearch</namespace>  
<version>1.0</version>  
<author name=”Freddy Kristiansen”>  
<info url=”[http://blogs.msdn.com/freddyk”](http://blogs.msdn.com/freddyk) />  
</author>  
<copyright>None, feel free to use!</copyright>  
<description>Freddys NAV Search Gadget</description>  
<icons>  
<icon height=”48″ width=”48″ src=”Images/Navision.ico” />  
</icons>  
<hosts>  
<host name=”sidebar”>  
<base type=”HTML” apiVersion=”1.0.0″ src=”gadget.html” mce\_src=”gadget.html” />  
<permissions>full</permissions>  
<platform minPlatformVersion=”0.3″ />  
</host>  
</hosts>  
</gadget>

So, this is where you define Name, Namespace, Version, Author, etc.  But also Icon to display in the Add Gadget and the html document to display in the sidebar (in this case gadget.html).

In my sample I will be using Javascript – not because I by any means is an expert in Javascript – but I did some Javascript coding back around year 2000 – so I guess it is time to refresh my memory. I use notepad as my editor – and the biggest problem I have run into is, that whenever I make a mistake (like misspell something btw. Javascript is case sensitive) execution of Javascript will just stop without giving any form of error. I guess there are better way to write Javascript than this – I just haven’t found it.

### Gadget.html

Note, that this is not an HTML tutorial, I expect you to know the basic constructs of HTML and Javascript – you should be able to find a LOT of content around these things on the Internet.

The body section of my gadget looks like this:

<body bgcolor=”0″ leftmargin=”0″ topmargin=”0″ >  
<g:background opacity=”100″></g:background>  
<table width=”100%” height=”100%” border=”0″ hspace=”0″ vspace=”0″ cellpadding=”0″ cellspacing=”0″>  
<tr>  
<td height=”36″ align=”left” valign=”top” background=”Images/gadgettop.png” nowrap><p style=”margin-top: 10px”><strong><font color=”#FFFFFF” size=”3″ face=”Segoe UI”>&nbsp;NAV Search</font></strong></p></td>  
</tr>  
<tr>  
<td height=”22″ valign=”middle” background=”Images/gadgetmiddle.png”>  
<input type=”textbox” id=”SearchText” onFocus=”hideFlyout();”><input type=”image” src=”Images/search.png” id=”doSearch” onClick=”search();”>  
</td>  
</tr>  
<tr>  
<td height=”28″ border=”0″ background=”Images/gadgetbottom.png”>  

![Microsoft Dynamics NAV](Images/weelogo.png)

</td>  
</tr>  
</table>  
</body>

As you can see, most of this is HTML in order to make the Gadget look right. The only two Javascript methods that are called is

-   hideFlyout() – when the textbox receives focus.
-   search() – when you click the search icon (or press enter in the text box)

and of course our Gadget has references to some images from an Images folder.

The main search function looks like this

// Main search function  
// search after the content in the textbox  
function search()  
{  
// If flyout is shown, hide it  
if (System.Gadget.Flyout.show)  
{  
hideFlyout();  
}

    // Get search string  
str = document.getElementById(“SearchText”).value;  
if (str != “”)  
{  
// Perform search  
result = doSearch(str);  
if (result != “”)  
{  
// Store HTML to use when flyout pops out  
newHTML = result;  
// Display result in flyout  
System.Gadget.Flyout.show = true;  
}  
}  
}

System.Gadget.Flyout is part of the Gadget Framework and gives you access to set a document used for flyouts, show the flyout and hide it again.

The flyout is (as you can imagine) also just a HTML document – even though it doesn’t behave totally like a normal browser showing a HTML document – more about that later.

As you can see, the function, which will be doing the Web Service connection and the “real” search is doSearch:

// the “real” search function  
function doSearch(searchstring)  
{  
// Get the URL for the NAV 2009 Search Codeunit  
var URL = GetBaseURL() + “Codeunit/Search”;

// Create XMLHTTP and send SOAP document  
xmlhttp = new ActiveXObject(“Msxml2.XMLHTTP.4.0”);  
xmlhttp.open(“POST”, URL, false, null, null);  
xmlhttp.setRequestHeader(“Content-Type”, “text/xml; charset=utf-8”);  
xmlhttp.setRequestHeader(“SOAPAction”, “DoSearch”);  
xmlhttp.Send(‘<?xml version=”1.0″ encoding=”utf-8″?><soap:Envelope xmlns:soap=”[http://schemas.xmlsoap.org/soap/envelope/”](http://schemas.xmlsoap.org/soap/envelope/)\><soap:Body><DoSearch xmlns=”urn:microsoft-dynamics-schemas/codeunit/Search”><searchstring>’+searchstring+'</searchstring><result></result></DoSearch></soap:Body></soap:Envelope>’);  
  
// Find the result in the soap result and return the rsult  
xmldoc = xmlhttp.ResponseXML;  
xmldoc.setProperty(‘SelectionLanguage’, ‘XPath’);  
xmldoc.setProperty(‘SelectionNamespaces’, ‘xmlns:soap=”[http://schemas.xmlsoap.org/soap/envelope/”](http://schemas.xmlsoap.org/soap/envelope/) xmlns:tns=”urn:microsoft-dynamics-schemas/codeunit/Search”‘);  
result = xmldoc.selectSingleNode(“/soap:Envelope/soap:Body/tns:DoSearch\_Result/tns:result”).text;

// Load result into XML Document  
xmldoc = new ActiveXObject(“Msxml2.DOMDocument.4.0”);  
xmldoc.loadXML(result);

    // Load XSL document  
xsldoc = new ActiveXObject(“Msxml2.DOMDocument.4.0”);  
xsldoc.load(“SearchResultToHTML.xslt”); 

    // Transform  
return xmldoc.transformNode(xsldoc);  
}

Wow – a lot of code.

This is actually the only code in the Gadget connecting to Web Services – all the other code is housekeeping and has as such nothing to do with NAV 2009. Basically we just get the URL for the Web Service and use XMLHTTP to connect to the Web Service and get a SOAP response back. We use XPath to find the XML result from our codeunit. Load this into a XML Document. Load the XSLT into another XML Document and transform the XML using the XSLT – somehow similar to the way we did it in C# in part 2.

I will post other examples of Gadgets communicating with NAV Web Services, stay tuned.

The basic initialization of the gadget is done in

// Microsoft suggests using onreadystatechange instead of onLoad  
document.onreadystatechange = function()  
{  
if(document.readyState==”complete”)  
{  
// Initialize Settings and Flyout  
System.Gadget.settingsUI = “settings.html”;  
System.Gadget.Flyout.file = “flyout.html”; 

        // Add eventhandler for Flyout onShow  
System.Gadget.Flyout.onShow = flyoutShowing;  

// Write default Base URL in settings if not already done  
GetBaseURL();  
}  
}

When setting the value of settingsUI on System.Gadget the gadget will get a small [![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_thumb_3.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SearchinNAV2009Part3outof3_495/image_8.png) icon when you hover over the Gadget and an Options menu item in the Context menu. Both these options will open the HTML defined in settingsUI.

There is no code in the Flyout – the only special thing is with the flyout is that it contains an IFRAME element, which loads the content.html document in order to get a scrollbar if the content of the flyout becomes too big.

The GetBaseURL function is used under startup – and when we need to connect.

// Get the Base Web Services URL  
function GetBaseURL()  
{  
// Read the URL from settings  
var URL = System.Gadget.Settings.readString(“URL”);  
if (URL == “”)  
{  
// No settings in the settings.ini – write the default URL  
URL = defaultURL;  
System.Gadget.Settings.writeString(“URL”, URL);  
}  
// Always terminate with /  
if (URL.substr(URL.length-1,1) != “/”)  
{  
URL = URL + “/”;  
}  
return URL;  
}

The reason for calling the function at startup is, that we set the settings to the default URL if it isn’t already defined. It is better that the settings dialog comes up with “some” default than just a blank URL – IMO.

The settings.html contains two functions for doing the housekeeping of the settings:

// Initialize settings Form  
document.onreadystatechange = function()  
{  
if(document.readyState==”complete”)  
{  
// Read settings and set in form  
URL.value = System.Gadget.Settings.read(“URL”);  
}  
}

// Event handler for onSettingsClosing  
System.Gadget.onSettingsClosing = function(event)  
{  
if (event.closeAction == event.Action.commit)  
{  
// Write new URL into settings  
System.Gadget.Settings.writeString(“URL”, URL.value);

        // State that it is OK to close the settings form  
event.cancel = false;  
}  
}

I will let the code speak for itself.

The System.Gadget.Settings read and write functions stores the settings in

C:\\Users\\<username>\\AppData\\Local\\Microsoft\\Windows Sidebar\\settings.ini

and the settings will be stored in clear text, you will actually be able to modify this file as well.

That’s it for the NAV Search demo – I hope you like it, you can download the Gadget from [http://www.freddy.dk/Search – Part 3.zip](<http://www.freddy.dk/Search - Part 3.zip>). Note that this download cannot stand alone – you need the NAV piece of this, which you can find in [Part 1](http://blogs.msdn.com/freddyk/archive/2008/11/23/search-in-nav-2009-part-1-out-of-3.aspx).

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
