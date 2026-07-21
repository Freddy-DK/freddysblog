---
layout: post
title: "Microsoft Windows Vista Gadget – My “Stuff”"
date: 2008-12-04 23:17:49
categories: ["Archive"]
tags: ["Asynchronous", "Bookmark", "Gadget", "Javascript", "NAV 2009", "Web Services", "XML", "XMLPort"]
permalink: /2008/12/04/microsoft-windows-vista-gadget-my-stuff/
---

This is my second gadget. My first gadget was the Search gadget, which you can find [here](http://blogs.msdn.com/freddyk/archive/2008/11/24/search-in-nav-2009-part-3-out-of-3.aspx).

I won’t repeat any explanation as to what a gadget is, nor will I talk about Javascript – instead I will focus upon the things, which are new in this post:

-   Returning an XMLPort from Web Services to Javascript
-   Running a Web Service method asynchronous
-   Use the same gadget for invoking three different methods
-   Poor mans error handling in Javascript

What I want to do is to create a gadget that can show My Customers, My Vendors or My Items.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_2.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_2.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_6.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_4.png)

The flyout from the gadget should be additional information about the record in the gadget, like

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_3.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_8.png)

Clicking on the customer number should open the Customer Task Page and it should be easy to add additional information to this window (the things that are interesting to quickly get access to for our users).

In the Settings dialog you should be able to control things like

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_4.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_10.png)

I will not discuss My Activities in this post, although it would be cool to have a Vista Gadget with My Activities (must do that some other day).

Ready? – well here we go…

### The GetMyStuff Codeunit

In NAV we need to expose something, through which we can get access to My Customers, My Vendors and My Items – and the easiest thing would of course be to just expose the 3 pages: 9150 (My Customers), 9151 (My Vendors) and 9152 (My Items).

The problem with this approach is, that we need to get a bookmark back (for opening Task Pages) from the Web Service – meaning that we either create another field in these pages – or we create another Web Service Method for returning the bookmark. I really don’t want to modify base objects if I can avoid it and adding another function just multiplies the number of times we invoke Web Service method by a factor, which I am not interested in doing either.

So I am going to go with a Codeunit, 3 XML ports – and the Codeunit basically just have these three functions.

**GetMyCustomers(VAR result : XMLport “My Customers”)  
**mycustomer.SETRANGE(“User ID”,USERID);  
result.SETTABLEVIEW(mycustomer);

**GetMyItems(VAR result : XMLport “My Items”)  
**myitem.SETRANGE(“User ID”,USERID);  
result.SETTABLEVIEW(myitem);

**GetMyVendors(VAR result : XMLport “My Vendors”)  
**myvendor.SETRANGE(“User ID”,USERID);  
result.SETTABLEVIEW(myvendor);

Now returning a XMLPort (which is a NAV construct) from a Web Service sounds weird – and of course we do NOT return the XML Port object to the Web Service Consumer. What we get is the output of the XMLPort – and if you send data into the XMLPort, it is going to run the XMLPort for input.

The XMLPort for My Customers looks like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_6.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_14.png)

and if we take a look at the WSDL for a Codeunit which returns this XMLPort, the schema for the return type very much looks like the above definition

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_5.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_12.png)

and the schema for the method references this as both input and output

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_8.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_18.png)

In the XMLPort properties, I have set the following properties

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_thumb_7.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/MicrosoftWindowsVistaGadgetMyStuff_EDDD/image_16.png)

and in the Table element properties MinOccurs is set to Zero. If MinOccurs is 1 (the default), the XMLPort will return an empty My Customer if the list is empty – we do not want that.

Codebehind on the XMLPort sets the three variables \_Name, \_Bookmark and \_Phone.

**MyCustomer – Export::OnAfterGetRecord()  
**rec.GET(“My Customer”.”Customer No.”);  
ref.GETTABLE(rec);  
\_Name := rec.Name;  
\_Phone := rec.”Phone No.”;  
\_Bookmark := FORMAT(ref.RECORDID,0,10);

Where rec is type Record of Customer and ref is a RecordRef.

The _My Vendors_ and _My Items_ XMLPorts are created following the same pattern and my gadget code will be made in a way, so that additional fields in the XMLPort will be displayed in the flyout – so you can add whatever you want to this XMLPort.

### The Gadget

Again – using Javascript and instead of listing the entire Gadget Sourcecode I will describe the flow.

The main HTML body contains two areas, in which I insert the content via DHTML. These areas are called CAPTION and CONTENT.

CAPTION is for holding the top part of the gadget, where is says My Customer, My Vendor etc. and CONTENT is for holding the list of records. The reason for the caption to be dynamic is that it changes based on the selected type of course.

The first code executed in the script (except for the variable declarations) is the onreadystatechanged event handler – and note that the construct

// Microsoft suggests using onreadystatechange instead of onLoad  
document.onreadystatechange = function()  
{  
if(document.readyState==”complete”)  
{  
// Initialize Settings  
System.Gadget.settingsUI = “settings.html”;  
        System.Gadget.onSettingsClosed = settingsClosed;

        // Initialize flyout  
System.Gadget.Flyout.file = “flyout.html”;  
System.Gadget.Flyout.onShow = flyoutShowing;  

        // Set the Caption of the Gadget  
setCaption();

        // Initialize timer – to refresh every x seconds  
setTimeout( “refresh()”, 1000 );  
}  
}

creates the delegate and assigns it to the event handler in one go.

In the event handler we setup the Settings dialog to use settings.html and the flyout to use flyout.html. setCaption creates a HTML string for the Caption and sets that into the CAPTION DHTML area and lastly, we setup the refresh method to be called in 1 second. This is done in order for the Gadget to be added smoothly and without any delays from connecting to Web Services etc.

The refresh function starts by adding another call to refresh delayed (depending on the RefreshInterval) and call setContent. setConent is the function, which does the actual Web Service method invoke.

// Refresh Gadget  
function refresh()  
{  
// Initialize timer – to refresh every  
setTimeout( “refresh()”, GetRefreshInterval()\*1000 );

    // Set the Content of the Gadget  
setContent();  
}

The three functions: GetRefreshInterval(), GetType() and GetBaseURL() are only for getting variables from the settings dialog. All functions will default the settings to the default value set in the top of the Javascript section, if they are not already defined. The reason for writing the values to the settings file here is that the settings.html becomes much simpler this way.

The settingsClosed event handler is setup in the onreadystatechanged above and is called when the Settings dialog is opened (clicking on the wrench next to the gadget). This event handler will update the Caption and call refresh (in order to force a refresh now – and setup a new timeout).

// Refresh Gadget on Settings Closed  
function settingsClosed(event)  
{  
// User hits OK on the settings page.  
if (event.closeAction == event.Action.commit)  
{  
// Caption might have changed based on settings  
setCaption();

        // Refresh content, update refresh Interval  
setTimeout(“refresh()”, 1000);  
}  
}

The setContent function starts out by setting up some global variables based on the type shown in the gadget

// Setup variables and invoke Web Service method for getting My records  
function setContent()  
{  
RTCpage = ”;  
Type = GetType();  
if (Type == ‘My Vendors’)  
{  
// Settings for My Vendors

        RTCpage = ’26’;  
WSfunction = ‘GetMyVendors’;  
XMLPortResultNS = ‘urn:microsoft-dynamics-nav/xmlports/myvendors’;  
XMLPortResultNode = ‘MyVendor’;  
}  
else if (Type == ‘My Items’)  
{  
// Settings for My Items

        RTCpage = ’30’;  
WSfunction = ‘GetMyItems’;  
XMLPortResultNS = ‘urn:microsoft-dynamics-nav/xmlports/myitems’;  
XMLPortResultNode = ‘MyItem’;  
}  
else if (Type == ‘My Customers’)  
{  
// Settings for My Customers  
RTCpage = ’21’;  
WSfunction = ‘GetMyCustomers’;  
XMLPortResultNS = ‘urn:microsoft-dynamics-nav/xmlports/mycustomers’;  
XMLPortResultNode = ‘MyCustomer’;  
}  
else  
{  
RTCpage = ”;  
}  
// Invoke GetMyStuff Web Service  
try  
{  
xmlhttp = new ActiveXObject(“Msxml2.XMLHTTP.4.0”);  
xmlhttp.open(“POST”, GetBaseURL()+”Codeunit/GetMyStuff”, false, null, null);  
xmlhttp.setRequestHeader(“Content-Type”, “text/xml; charset=utf-8”);  
xmlhttp.setRequestHeader(“SOAPAction”, “GetMyStuff”);

        // Setup event handler when readystate changes  
xmlhttp.onreadystatechange = function()  
{  
if ((xmlhttp.readyState == 4) && (xmlhttp.Status == 200))  
{  
xmldoc = xmlhttp.ResponseXML;  
xmldoc.setProperty(‘SelectionLanguage’, ‘XPath’);  
xmldoc.setProperty(‘SelectionNamespaces’, ‘xmlns:tns=”‘+XMLPortResultNS+'”‘);  
myXML = xmldoc.selectNodes(‘//tns:’+XMLPortResultNode);  
updateGadget(true);  
}  
else  
{  
updateGadget(false);  
}  
}  
xmlhttp.Send(‘<?xml version=”1.0″ encoding=”utf-8″?><soap:Envelope xmlns:soap=”[http://schemas.xmlsoap.org/soap/envelope/&#8221](http://schemas.xmlsoap.org/soap/envelope/&#8221);\><soap:Body><‘+WSfunction+’ xmlns=”urn:microsoft-dynamics-schemas/codeunit/GetMyStuff”><result></result></’+WSfunction+’></soap:Body></soap:Envelope>’);  
}  
catch(e)  
{  
// Something went wrong – display: “Service not available”, indicating that there of course are no bugs in the above code:-)  
updateGadget(false);  
}  
}

After setting up the variables, we initialize the xmlhttp and setup a delegate function for onreadystatechange on the xmlhttp (this gets invoked when the Web Service method is done). After this we invoke Send with a SOAP document conforming to the WSDL for the Web Service.

When the onreadystatechange event handler is executed we read the XML and update the content of the Gadget. If anything goes wrong we call the updateGadget function with false – indicating that it should change the content to an error message.

The UpdateGadget builds a HTML table and inserts this table into the CONTENT area of the gadget. For every row we add a call to showflyout if the user clicks the row in order to get additional information.

// Add a row to newHTML  
newHTML += ‘<tr><td height=”18″ valign=”middle” background=”Images/gadgetmiddle.png”>’;  
newHTML += ‘  

‘;  
newHTML += ‘ [‘+myXML\[o\].childNodes\[1\].text+’](showFlyout\('+o+'\))  
‘;  
newHTML += ‘

</td></tr>’;  
o++;

The showFlyout function is pretty simple

// Show flyout with additional information  
function showFlyout(no)  
{  
System.Gadget.Flyout.show = false;  
flyoutNo = no;  
System.Gadget.Flyout.show = true;  
}

and after this method has been called, the next thing happening is that the flyoutShowing event handler is invoked – and in this event handler we can calculate the content of the flyout and set it in the flyout.

// Flyout Showing event handler  
// Calculate content of flyout  
function flyoutShowing()  
{  
flyoutHTML = ‘<table width=”100%” border=”0″ hspace=”0″ vspace=”0″ cellpadding=”0″ cellspacing=”0″>’;  
flyoutHTML += ‘<tr><td height=”31″ align=”left” valign=”middle” background=”Images/topband.png” nowrap><p><strong><font color=”#FFFFFF” size=”3″ face=”Segoe UI”>&nbsp;’+myXML\[flyoutNo\].childNodes\[1\].text+'</font></strong></p></td></tr>’;  
flyoutHTML += ‘<tr><td valign=”top”><table cellspacing=”5″>’;  
for(i=3; i<myXML\[flyoutNo\].childNodes.length; i++)  
{  
flyoutHTML += ‘<tr><td>’+myXML\[flyoutNo\].childNodes\[i\].nodeName+'</td><td>’;  
if (i==3)  
{  
flyoutHTML += ‘<a href=”dynamicsnav:////runpage?page=’+RTCpage+’&bookmark=’+myXML\[flyoutNo\].childNodes\[2\].text+’&mode=view”>’+myXML\[flyoutNo\].childNodes\[i\].text+'</a>’;  
}  
else  
{  
flyoutHTML += myXML\[flyoutNo\].childNodes\[i\].text;  
}  
flyoutHTML += ‘</td></tr>’;  
}  
flyoutHTML += ‘</table></td></tr></table>’;

    obj = System.Gadget.Flyout.document.getElementById(“CONTENT”);  
obj.innerHTML = flyoutHTML;  
}

Really mostly string manipulation in order to create a string that looks correct. I could probably have done the same with XSLT – but this seems pretty easy. As you can see the function enumerates the content of the childNodes to myXML – and adds everything to the flyout. This means that if you add some fields to the XMLPort, then these will be included in the flyout as well.

The flyout.html only contains an empty HTML document with a CONTENT area, which is set by the code above.

### Settings.html

The settings.html is really simple, with a method for reading the settings and setting them onto the form

// Initialize settings Form  
document.onreadystatechange = function()  
{  
if(document.readyState==”complete”)  
{  
// Read settings and set in form  
URL.value = System.Gadget.Settings.read(“URL”);  
RefreshInterval.value = System.Gadget.Settings.read(“RefreshInterval”);  
Type.value = System.Gadget.Settings.read(“Type”);  
}  
}

A method which for setting the settings back into the settings file.

// Event handler for onSettingsClosing  
System.Gadget.onSettingsClosing = function(event)  
{  
if (event.closeAction == event.Action.commit)  
{  
// Write new URL into settings  
System.Gadget.Settings.writeString(“URL”, URL.value);  
System.Gadget.Settings.writeString(“RefreshInterval”, RefreshInterval.value);  
System.Gadget.Settings.writeString(“Type”, Type.value);

        // State that it is OK to close the settings form  
event.cancel = false;  
}  
}

and the form itself in HTML

<table width=”100%” height=”100%”>  
<tr><td>  
Web Service Base URL:<br>  
<input type=”textbox” id=”URL” maxlength=”250″>  
</td></tr>  
<tr><td>  
My type:<br>  
<select name=”Type” size=”1″>  
<option value=”My Customers”>My Customers</option>  
<option value=”My Vendors”>My Vendors</option>  
<option value=”My Items”>My Items</option>  
</select>  
</td></tr>  
<tr><td>  
Refresh Interval:<br>  
<select name=”RefreshInterval” size=”1″>  
<option value=”10″>5 seconds</option>  
<option value=”30″>30 seconds</option>  
<option value=”60″>1 minute</option>  
<option value=”300″>5 minutes</option>  
</select>  
</td></tr>  
</table>

I will let the code speak for itself.

If you want to see the entire thing, feel free to download it from [http://www.freddy.dk/GetMyStuff.zip](http://www.freddy.dk/GetMyStuff.zip). the Zip file both contains the Gadget (open that and install – or rename to .zip) and a .fob file with the NAV 2009 objects. You will need to expose the GetMyStuff Codeunit as a webservice with the same name.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
