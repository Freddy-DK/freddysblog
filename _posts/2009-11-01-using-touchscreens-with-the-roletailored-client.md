---
layout: post
title: "Using touchscreens with the RoleTailored Client"
date: 2009-11-01 10:00:20
categories: ["Archive"]
tags: ["Add-Ins", "BigText", "C#", "Client Tier", "Extensibility", "Javascript", "NAV 2009 SP1", "Touch Screen"]
permalink: /2009/11/01/using-touchscreens-with-the-roletailored-client/
---

I LOVE the RoleTailored Client, I LOVE the fact that everything is metadata driven and i LOVE what this will give us (us being everybody using NAV) going forward. As a result of the investments leading to NAV 2009, NAV has by far the most modern UX and the new framework allows us to innovate faster and more consistent than any other ERP solution out there.

We can change the UX to follow Microsoft Office 2010 if we decide to, without having to do a wash through all pages and modify those to follow the UX. We can create new UI paradigms and allow the existing pages to be reused and we will make sure that the UX is consistent throughout NAV.

I do however also acknowledge that sometimes, love just isn’t enough – for some scenarios, the RoleTailored Client doesn’t make things easier for us and we need to consider what to do.

In this post I will try to explain a way to handle one of these scenarios – creating a page with buttons that can be used from a Touch Screen like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/UsingtouchscreenswiththeRoleTailoredClie_116BE/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/UsingtouchscreenswiththeRoleTailoredClie_116BE/image_2.png)

As you might  guess – this requires Visual studio:-)

### Button Panels

I have collected a number of screenshots from various applications using touch screens – and it is very common to have one or more panels of buttons and other information from NAV. It is no secret that you could of course just create a button panel like this in Visual Studio using WinForms and you would be on your way, but the problem here is, that you would put the decision on location, size, captions and visuals of the buttons into your Visual Studio solution.

You would have to have a way of describing the looks and the functionality of the button panel from NAV in order to capture your business logic in one place. Thinking more about this – I found myself trying to describe something I had seen before…

A “string” that would describe the visuals, the flow, the positions and the functionality of a panel – that sounds a lot like HTML and Javascript, so if I decided to go with a browser using HTML and Javascript – how in earth would I raise an event on the Service Tier from inside my browser?

### Escaping from Javascript

I decided to go forward with the browser idea and try to find out how to escape from Javascript – and it turned out to be pretty simple actually.

On the WebBrowser Control there is a property called [ObjectForScripting](http://msdn.microsoft.com/en-us/library/system.windows.forms.webbrowser.objectforscripting.aspx). By setting that property you are now able to escape to that object from Javascript using **window.external.myfunction(myparameters);**. In Fact – all the methods in the class you specify in ObjectForScripting are now available from Javascript.

### Show me the code!!!

If you haven’t created Microsoft Dynamics Add-Ins before, you might want to read some of the basics on [Christian’s blog](http://blogs.msdn.com/cabeln), especially the following post explains the basics pretty well:

[http://blogs.msdn.com/cabeln/archive/2009/05/06/add-ins-for-the-roletailored-client-of-microsoft-dynamicsnav-2009-sp1-part1.aspx](http://blogs.msdn.com/cabeln/archive/2009/05/06/add-ins-for-the-roletailored-client-of-microsoft-dynamicsnav-2009-sp1-part1.aspx "http://blogs.msdn.com/cabeln/archive/2009/05/06/add-ins-for-the-roletailored-client-of-microsoft-dynamicsnav-2009-sp1-part1.aspx")

Assuming that you are now a shark in creating Add-Ins – we can continue:-)

Let’s first of all create the native WinForms Control. We can use the WebBrowser unchanged – although the WebBrowser comes with an error, which sometimes surfaces in NAV. If you set the DocumentText in the browser control before it is done rendering the last value of DocumentText – it will ignore the new value. Frankly I want an implementation where the last value wins – NOT the first value. I handle that by subscribing to the DocumentCompleted event and check whether there is a newer value available. I also don’t want to set the value in the WebBrowser if it hasn’t changed.

```
public class MyWebBrowser : WebBrowser
{
private string text;
private string html = Resources.Empty;
```

    

```
/// <summary>
/// Constructor for WebBrowser Control
/// </summary>
public MyWebBrowser()
{
this.DocumentCompleted += new WebBrowserDocumentCompletedEventHandler(MyWebBrowser_DocumentCompleted);
}
```

    

```
/// <summary>
/// Handler for DocumentCompleted event
/// If we are trying to set the DocumentText while the WebBrowser is rendering – it is ignored
/// Catching this event to see whether the DocumentText should change fixes that problem
/// </summary>
void MyWebBrowser_DocumentCompleted(object sender, WebBrowserDocumentCompletedEventArgs e)
{
if (this.DocumentText != this.html)
{
this.DocumentText = this.html;
}
}
```

    

```
/// <summary>
/// Get/Set the Text of the WebBrowser
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
html = Resources.Empty;
}
else
{
html = text;
}
this.DocumentText = html;
}
}
}
}
```

and now the Add-In part of the Control.

```
[ControlAddInExport("FreddyK.BrowserControl")]
public class BrowserControl : StringControlAddInBase, IStringControlAddInDefinition
{
MyWebBrowser control;
```

    

```
protected override Control CreateControl()
{
control = new MyWebBrowser();
control.MinimumSize = new Size(16, 16);
```

```
control.MaximumSize = new Size(4096, 4096);
control.IsWebBrowserContextMenuEnabled = false;
control.ObjectForScripting = new MyScriptManager(this);
control.ScrollBarsEnabled = false;
control.ScriptErrorsSuppressed = true;
control.WebBrowserShortcutsEnabled = false;
control.Dock = DockStyle.Fill;
return control;
}
```

    

```
public void clickevent(int i, string s)
{
this.RaiseControlAddInEvent(i, s);
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
```

    

```
public override bool HasValueChanged
{
get
{
return false;
}
}
```

    

```
public override string Value
{
get
{
return base.Value;
}
set
{
base.Value = value;
if (this.control != null)
this.control.Text = value;
}
}
}
```

Things to note:

-   I am using DockStyle.Fill to specify that the Control should take up whatever space is available.
-   ObjectForScripting is set to an instance of the MyScriptManager class
-   the clickevent method raises the Add-In Event on the Service Tier with the parameters coming from the caller.

The MyScriptManager could look like this:

```
[ComVisible(true)]
public class MyScriptManager
{
BrowserControl browserControl;
```

    

```
public MyScriptManager(BrowserControl browserControl)
{
this.browserControl = browserControl;
}
```

    

```
public void clickevent(int i, string s)
{
browserControl.clickevent(i, s);
}
}
```

and as you might have guessed – this allows Javascript in the WebBrowser to invoke statements like:

`window.external.clickevent(i, s);`

Note that you need to have **ComVisible(true)** on the ScriptManager class.

Of course you need to sign the DLL, copy the DLL to the Add-Ins folder and create an entry in the Client Add-Ins table.

You can download the source to the Visual Studio project [here](http://www.freddy.dk/NAVBrowserControl.zip) – and if you use this, the public key token for this add-in is **58e587b763c2f132** and the Control Add-In Name is **FreddyK.BrowserControl**.

### Let’s put the BrowserControl to work for us

Assuming that we have built the BrowserControl, copied and registered it – we will not build a page with two fields:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/UsingtouchscreenswiththeRoleTailoredClie_116BE/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/UsingtouchscreenswiththeRoleTailoredClie_116BE/image_6.png)

and of course create two global Variables (HTML as BigText and Value as Decimal).

The pagetype of the page is set to CardPart (in order to avoid the menus – I know this kind of bends the rules of the RoleTailored Client, but since this is a page that wasn’t supposed to be – I think we should manage).

on the Value field – set the DecimalPlaces to **0:10** and on the browser field – set the ControlAddIn property to point to our Browser Control: **FreddyK.BrowserControl;PublicKeyToken=58e587b763c2f132**.

Now in the OnOpenPage of the page – put the following lines:

```
OnOpenPage()
CLEAR(HTML);
HTML.ADDTEXT('<html><body>Hello World</body><html>');
```

this should give us the following page when running:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/UsingtouchscreenswiththeRoleTailoredClie_116BE/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/UsingtouchscreenswiththeRoleTailoredClie_116BE/image_8.png)

A couple of things to think about when writing the “real” code:

-   We do not want to work directly in our HTML global variable, since any change in this would cause the UI to request an update.
-   If we want to use images in the HTML code, these images needs to be copied to the Client Tier – I do that using DownloadTempFile from the 3-Tier Management codeunit (varibale called TT).

The code to download the 3 images used (normal button, wide button and tall button) could be:

```
buttonurl := 'file:///'+CONVERTSTR(TT.DownloadTempFile(APPLICATIONPATH + 'button.png'),",'/');
tallbuttonurl := 'file:///'+CONVERTSTR(TT.DownloadTempFile(APPLICATIONPATH + 'tallbutton.png'),",'/');
widebuttonurl := 'file:///'+CONVERTSTR(TT.DownloadTempFile(APPLICATIONPATH + 'widebutton.png'),",'/');
```

and the code to create the HTML/Javascript code could look like this:

```
CLEAR(TEMP);
TEMP.ADDTEXT('<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" ');
TEMP.ADDTEXT('"
```

[`http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"`](http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd")

```
>');
TEMP.ADDTEXT('<html xmlns="
```

[`http://www.w3.org/1999/xhtml"`](http://www.w3.org/1999/xhtml") 

```
>');
TEMP.ADDTEXT('<head>');
```

```
// Create Stylesheet for the visuals
TEMP.ADDTEXT('<style type="text/css">');
TEMP.ADDTEXT('  td { width:64px; font-size:xx-large; background-image:url("'+buttonurl+"') }');
TEMP.ADDTEXT('  tr { height:64px }');
TEMP.ADDTEXT('  a { color:#000000; text-decoration:none }');
TEMP.ADDTEXT('  body { margin:0px; background-color:#FAFAFA }');
TEMP.ADDTEXT('</style>');
```

```
// Create Javascript function for invoking AL Event
TEMP.ADDTEXT(");
TEMP.ADDTEXT('  function click(i, s) {');
TEMP.ADDTEXT('    window.external.clickevent(i, s);');
TEMP.ADDTEXT('  }');
TEMP.ADDTEXT(");
```

```
TEMP.ADDTEXT('</head>');
TEMP.ADDTEXT('<body>');
```

```
// Create Table with Controls
TEMP.ADDTEXT('<table cellpadding="0″ cellspacing="5″>');
TEMP.ADDTEXT('<tr>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(7, "")">7</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(8, "")">8</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(9, "")">9</a></td>');
tempstyle := 'background-image:url("'+tallbuttonurl+"')';
TEMP.ADDTEXT('<td style="'+tempstyle+'" rowspan="2″ align="center"><a href="javascript:click(-1, "+")">+</a></td>');
TEMP.ADDTEXT('</tr>');
TEMP.ADDTEXT('<tr>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(4, "")">4</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(5, "")">5</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(6, "")">6</a></td>');
TEMP.ADDTEXT('</tr>');
TEMP.ADDTEXT('<tr>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(1, "")">1</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(2, "")">2</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(3, "")">3</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(-1, "-")">-</a></td>');
TEMP.ADDTEXT('</tr>');
TEMP.ADDTEXT('<tr>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(-1, ".")">.</a></td>');
TEMP.ADDTEXT('<td align="center"><a href="javascript:click(0, "")">0</a></td>');
tempstyle := 'width:133px; background-image:url("'+widebuttonurl+"')';
TEMP.ADDTEXT('<td style="'+tempstyle+'" colspan="2″ align="center"><a href="javascript:click(-1, "=")">=</a></td>');
TEMP.ADDTEXT('</tr>');
TEMP.ADDTEXT('</table>');
```

```
TEMP.ADDTEXT('</body>');
TEMP.ADDTEXT('</html>');
HTML := TEMP;
```

Meaning that every click on any button is routed back to the Add-In Event – and the actual calculator is then implemented in AL Code.

I am not going to go in detail about how to create a calculator, since this is pretty trivial and really not useful – the thing to take away from this sample is how to create button panels in HTML and have every button pressed routed to NAV for handling.

The Calculator .fob file (one page) and the 3 images used in this example can be downloaded [here](http://www.freddy.dk/Calculator.zip) – but again – this is just a “stupid” example. I do think that the technology can come in handy in some cases.

Now, I am aware, that this is not going to solve all issues and you shouldn’t try to twist this to hold all your forms in order to be able to manage colors and font sizes – but it can be used in one-off pages, where you have a page that needs to be used in a warehouse or other locations where you might want huge fonts or touch screen button panels.

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
