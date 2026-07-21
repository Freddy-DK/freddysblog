---
layout: post
title: "Integration to Virtual Earth – Part 4 (out of 4)"
date: 2009-06-07 01:18:43
categories: ["Archive"]
tags: ["C#", "Extensibility", "Javascript", "Latitude", "Longitude", "MAP", "NAV 2009 SP1", "URL", "Virtual Earth"]
permalink: /2009/06/07/integration-to-virtual-earth-part-4-out-of-4/
---

(a small change added that simplifies the SmallVEControl class definition)

With the release of NAV 2009 SP1 CTP2 (to MVPs, TAP and BAP) and the official release of the statement of Direction, I can now write about the last part of the integration to Virtual Earth.

People who hasn’t access to NAV 2009 SP1, will unfortunately have to wait until the official release until they can take advantage of this post.

Please not that you should read [Part 1](http://blogs.msdn.com/freddyk/archive/2009/03/18/integration-to-virtual-earth-part-1-out-of-4.aspx), [Part 2](http://blogs.msdn.com/freddyk/archive/2009/03/23/integration-to-virtual-earth-part-2-out-of-4.aspx) and [Part 3](http://blogs.msdn.com/freddyk/archive/2009/03/23/integration-to-virtual-earth-part-3-out-of-4.aspx) of the Integration to Virtual Earth – and you would have to have the changes to the app. described in these posts in order to make this work.

This post will take advantage of a functionality, which comes in NAV 2009 SP1 called Extensibility. Christian explains some basics about extensibility in a post, which you can find [here](http://blogs.msdn.com/cabeln/archive/2009/05/06/add-ins-for-the-roletailored-client-of-microsoft-dynamicsnav-2009-sp1-part1.aspx).

### The Goal

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_2.png)

As you can see on the above picture, we have a control, which is able to show the map in NAV of the customer location, and as you select different customers in the list, the map changes.

The changes in the map happens without any user interference, so that the user can walk up and down in the list without being irritated. In the Actions menu in the part, we will put an action called **Open In Browser**, which will open up a map in a browser as explained in part 3.

Note that the Weather factbox is not shown here.

### What is it?

The Control inside the Customer Map Factbox is basically just a browser control, in which we set a html document (pretty much like the one described in part 3) and leave it to the browser control to connect to Virtual Earth and retrieve the map. I do not connect to web services from the browser control, instead we transfer parameters of the current customer location to the control.

Although the internal implementation is a browser control, we don’t do html in NAV and we don’t give the control any URL’s or other fancy stuff. The way we make this work is to have the control databind to a Text variable (CustomerLocation), which gets set in OnAfterGetRecord:

`CustomerLocation := 'latitude='+FORMAT(Latitude,0,9)+'&longitude='+FORMAT(Longitude,0,9)+'&zoom=15';`

The factbox isn’t able to return any value and there isn’t any reason right now to trigger any events from the control.

So now we just need to create a control, which shows the string “latitude=50&longitude=2&zoom=15” differently than a dumb text.

### How is the control build?

Let’s just go through the creation of the VEControl step by step.

1\. Start Visual Studio 2008 SP1, create a new project of type Class Library and call it VEControl.

2\. Add a reference **System.Windows.Forms** , **System.Drawing** and to the file **C:\\Program Files\\Microsoft Dynamics NAV\\60\\RoleTailored Client\\Microsoft.Dynamics.Framework.UI.Extensibility.dll** – you need to browse and find it. Note that when you copy the VEControl.dll to it’s final location you don’t need to copy this DLL, since it will be loaded into memory from the Client before your DLL is called.

3\. Open Project Properties, go to the Signing tab, and sign your DLL with a new key.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_4.png)

4\. In the Build Events Tab add the following command to the Post-Build Event window:

`copy VEControl.dll "C:\Program Files\Microsoft Dynamics NAV\60\RoleTailored Client\Add-ins"`

this ensures that the Control gets installed in the right directory.

5\. Delete the automatically generated class1.cs and add another class file called VEControl.cs

6\. Add the following class to the file:

```
/// <summary>
/// Native WinForms Control for Virtual Earth Integration
/// </summary>
public class VEControl : WebBrowser
{
private string template;
private string text;
private string html = "<html><body></body></html>";
```

    

```
/// <summary>
/// Constructor for Virtual Earth Integration Control
/// </summary>
/// <param name="template">HTML template for Map content</param>
public VEControl(string template)
{
this.template = template;
this.DocumentCompleted += new WebBrowserDocumentCompletedEventHandler(VEControl_DocumentCompleted);
}
```

    

```
/// <summary>
///
/// </summary>
/// <param name="sender"></param>
/// <param name="e"></param>
void VEControl_DocumentCompleted(object sender, WebBrowserDocumentCompletedEventArgs e)
{
if (this.DocumentText != this.html)
{
this.DocumentText = this.html;
}
}
```

    

```
/// <summary>
/// Property for Data Binding
/// </summary>
public override string Text
{
get
{
return text;
}
set
{
if (text != value)
{
text = value;
if (string.IsNullOrEmpty(value))
{
html = "<html><body></body></html>";
}
else
{
html = this.template;
html = html.Replace("%latitude%", GetParameter("latitude", "0"));
html = html.Replace("%longitude%", GetParameter("longitude", "0"));
html = html.Replace("%zoom%", GetParameter("zoom", "1"));
}
this.DocumentText = html;
}
}
}
```

    

```
/// <summary>
/// Get Parameter from databinding
```

    

```
/// </summary>
/// <param name="parm">Parameter name</param>
/// <param name="defaultvalue">Default Value if the parameter isn't specified</param>
/// <returns>The value of the parameter (or default)</returns>
private string GetParameter(string parm, string defaultvalue)
{
foreach (string parameter in text.Split('&'))
{
if (parameter.StartsWith(parm + "="))
{
return parameter.Substring(parm.Length + 1);
}
}
return defaultvalue;
}
}
```

Note, that you will need a using statement to System.Windows.Forms.

This class gets initialized with a html template (our javascript code) and is able to get values like “latitude=50&longitude=2&zoom=15” set as the Text property and based on this render the right map through the template.

The reason for the DocumentCompleted event handler is, that if we try to set the DocumentText property in the browser before it is done rendering the prior DocumentText, it will just ignore the new value. We handle this by hooking up to the event and if the DocumentText is different from the value we have – then this must have happened and we just set it again. We are actually pretty happy that the control works this way, because the javascript is run in a different thread than our main thread and fetching the map control from Virtual Earth etc. will not cause any delays for us.

Now this is just a standard WinForms Control – how do we tell the Client that this is a control, that it can use inside the NAV Client?

The way we chose to implement this is by creating a wrapper, which is the one we register with the NAV Client and this wrapper is responsible for creating the “real” control. This allows us to use 3rd party controls even if they are sealed and/or we don’t have the source for them.

7\. Add a html page called SmallVEMap.htm and add the following content

```
<html>
<head>
<title></title>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8″ />
        var latitude = parseFloat("%latitude%");
var longitude = parseFloat("%longitude%");
var zoom = parseInt("%zoom%");
map.SetDashboardSize(VEDashboardSize.Tiny);
```

        

```
var position = new VELatLong(latitude, longitude);
map.LoadMap(position, zoom, 'r', false);
shape = new VEShape(VEShapeType.Pushpin, position);
map.AddShape(shape);
}
```

   

</head>  
<body onload=”GetMap();” style=”margin:0; position:absolute; width:100%; height:100%; overflow: hidden”>

</body>  
</html>

8\. Add a Resource file to the project called Resources.resx, open it and drag the SmallVEMap.htm into the resources file.

9\. Add a class called SmallVEControl.cs and add the following classes

```
[ControlAddInExport("SmallVEControl")]
public class SmallVEControl : StringControlAddInBase, IStringControlAddInDefinition
{
protected override Control CreateControl()
{
var control = new VEControl(Resources.SmallVEMap);
control.MinimumSize = new Size(200, 200);
control.MaximumSize = new Size(500, 500);
control.ScrollBarsEnabled = false;
control.ScriptErrorsSuppressed = true;
control.WebBrowserShortcutsEnabled = false;
return control;
}
```

    

```
public override bool AllowCaptionControl
{
get
{
return false;
}
}
}
```

You need to add using statements to **System.Drawing**, **Microsoft.Dynamics.Framework.UI.Extensibility**, **Microsoft.Dynamics.Framework.UI.Extensibility.WinForms** and **System.Windows.Forms**.

The CreateControl is the method called by the NAV Client when it needs to create the actual winforms control. We override this method and create the VEControl and give it the html template.

The reason for overriding the AllowCaptionControl is to specify that our control will not need a caption (else the NAV Client will add a caption control in front of our control).

There are various other methods that can be overridden, but we will touch upon these when needed.

Build your solution and you should now have a VEControl.DLL in the Add-Ins directory under the RoleTailored Client.

### And how do I put this control into use in the NAV Client?

First of all we need to tell the Client that the control is there!

We do that by adding an entry to the **Client Add-In** table (2000000069). You need to specify Control Add-In Name (which would be the name specified in the ControlAddInExport attribute above = SmallVEControl) and the public key token.

But what is the public key token?

Its is the public part of the key-file used to sign the assembly and as you remember, we just asked Visual Studio to create a new key-file so we need to query the key file for it’s public key and we do that by running

`sn –T VEControl.snk`

in a Visual Studio command prompt.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_6.png)

[

Note that this public key is NOT the one you need to use, unless you download my solution below.

](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_6.png)

[](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_6.png)[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_8.png)

Having the Control Registered for usage we need to create a new page and call it Customer Map Factbox. This page has SourceTable set to the Customer table and is contains one control, bound to a variable called CustomerLocation, which gets set in the OnAfterGetRecord.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_thumb_5.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart4outof4_12A7B/image_12.png)

The code in OnAfterGetRecord is

`CustomerLocation := 'latitude='+FORMAT(Latitude,0,9)+'&longitude='+FORMAT(Longitude,0,9)+'&zoom=15';`

The Customer Map Factbox is added as a part to the Customer Card and the Customer List and the SubFormLink is set to **No.=FIELD(No.)**

That’s it guys – I realize this is a little rough start on extensibility – I promise that there will be other and more entry level starter examples on extensibility – I just decided to create an end-to-end sample to show how to leverage the Virtual Earth functionality in a Factbox.

As usual you can download the visual studio project [here](http://www.freddy.dk/VEControl.zip).

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
