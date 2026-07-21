---
layout: post
title: "Running Code on the Client without deploying COM objects"
date: 2009-10-14 10:41:38
categories: ["Archive"]
tags: ["A/D", "Automation", "Client Tier", "COM", "INPUT", "Service Tier", "VBScript", "XP"]
permalink: /2009/10/14/running-code-on-the-client-without-deploying-com-objects/
---

**Yes**, it can be done!

**No**, it isn’t .net code – nor AL code.

### Why?

It started out as me being a little too fast when stating that you could easily download a file to the Client and attach it to Outlook without any user interaction – and as you might know that is true, but you might also know that if you go the recommended route:

```
FILE.DOWNLOAD(FileName, ", '<TEMP>',", ToFile);
Mail.NewMessage(",",Name,",ToFile,TRUE);
FILE.ERASE(FileName);
```

Then you will get an e-mail that looks like this:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_4.png)

People might not assume that this actually is **Invoice no. 10103** in PDF format. What you of course want to have is:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_2.png)

So, how do we get there.

I actually did respond to a post on mibuso a while back ([http://www.mibuso.com/forum/viewtopic.php?f=32&t=29806](http://www.mibuso.com/forum/viewtopic.php?f=32&t=29806 "http://www.mibuso.com/forum/viewtopic.php?f=32&t=29806")) about how this could be done, but that would involve a COM object deployed to all clients and not everybody wants that (although I have posted a method on how to do this automatically).

The problem here is, that file.download always places the file in a temporary directory with a temporary filename – and there is (to my knowledge) no other way to copy a file to the Client.

Assuming that this is correct, how do we then rename a file on the client without having to deploy COM objects?

### I said without deploying COM objects, not without USING COM objects

As you know, we can run COM objects on the server or on the Client and one of the COM objects, which ships with Windows can come in handy here. The Windows Script Host – if we instantiate this COM object we can actually give the component a VB Script to execute in the context of the COM component (which would be either on the Server or on the Client).

### Windows Script Host

Yes, WSH is legacy – but it is widely used and it is included on all Windows versions from Windows XP and up. I am not going to make this a tutorial on VBScript and WSH – for that you can find a number of good posts on the internet – or start by reading msdn

### [http://msdn.microsoft.com/en-us/library/ms950396.aspx](http://msdn.microsoft.com/en-us/library/ms950396.aspx "http://msdn.microsoft.com/en-us/library/ms950396.aspx")

### Creating a script function / method

The method AddCode on the Windows Script Host COM object is used to add sourcecode to the component.

Note, that you need to add an entire function / method in one call and note, that each line needs to be terminated by a CR.

You also need to specify what language you use, the control supports JScript and VBScript.

A VBScript function which returns **Hello <name>** could look like this:

```
function Hello(who)
Hello = "Hello "&who
end function
```

Creating this function in a Client side COM component could look like:

```
IF CREATE(objScript,TRUE,TRUE) THEN
BEGIN
CR := ' '; CR[1] := 13;
objScript.Language := 'VBScript';
objScript.AddCode(
'function Hello(who)'+CR+
'  Hello = "Hello "&who'+CR+
'end function');
END;
```

The way I write this is, that I try to maintain the structure of the VBScript even though it is inside a string in NAV, maybe I am fooling myself, but I think it is more readable.

### Invoking a script function / method

There are two ways of invoking a script method:

**Eval** – used to invoke a function, and get a return value back.

The above function could be called using

`MESSAGE(FORMAT(objScript.Eval('Hello("Freddy")')));`

Note – when calling functions, VBScript wants your parameters embraced by parentheses.

**ExecuteStatement** – used to invoke a method which doesn’t return anything

Let’s rewrite the above function to a method and have the method show a MessageBox:

The VBScript could look like:

```
sub Hello(who)
MsgBox "Hello "&who, 0, "Title"
end sub
```

and creating this function in a COM object and calling the method could look like:

```
IF CREATE(objScript,TRUE,TRUE) THEN
BEGIN
CR := ' '; CR[1] := 13;
objScript.Language := 'VBScript';
```

```
objScript.AddCode(
'sub Hello(who)'+CR+
'  MsgBox "Hello "&who, 0, "Test"'+CR+
'end sub');
objScript.ExecuteStatement('Hello "Freddy"');
END;
```

Note – when calling methods (or sub’s) VBScript does NOT want the parameters embraced by parentheses.

### Some sample scripts

**Rename a temporary file**

```
function RenameTempFile(fromFile, toFile)
set fso = createobject("Scripting.FileSystemObject")
set x = createobject("Scriptlet.TypeLib")
path = fso.getparentfoldername(fromFile)
toPath = path+""+left(x.GUID,38)
fso.CreateFolder toPath
fso.MoveFile fromFile, toPath+""+toFile
RenameTempFile = toPath
end function
```

As you can see, I am doing exactly what I responded on the mibuso thread [here](http://www.mibuso.com/forum/viewtopic.php?f=32&t=29806) – just in VBScript instead – which then requires no client side install.

BTW this function is actually used in ClausL’s post about [sending e-mail with PDF attachments](http://blogs.msdn.com/nav-reporting/archive/2009/10/08/send-email-with-pdf-attachment-in-nav-2009.aspx), which proves that we do talk with our colleagues at Microsoft:-). Note that there is no good way of creating a GUID from VBScript – I (mis)use the fact that every instance of Scriptlet.TypeLib gets assigned a new GUID.

**Get Machine name**

```
function GetComputerName()
set net = createobject("wscript.network")
GetComputerName = net.ComputerName
end function
```

I know, that you also can read an environment variable – but this way you can actually get all kind of information on the network though this.

**Launch an application**

```
sub Notepad()
set shell = createobject("WScript.Shell")
shell.Run "notepad.exe"
end sub
```

Yes, you can do this by using the Shell object directly in NAV, like:

_Shell       Automation       ‘Microsoft Shell Controls And Automation’.Shell_

```
CREATE(objShell,True,true);
objShell.Open('c:\windows\system32\notepad.exe');
```

I just wanted to show that you that stuff like this can be done in VBScript too, and note, that the Shell object in VBScript and in NAV is not the same.

**Asking a simple question**

```
function Input(question, title, default_answer)
Input = InputBox(question, title, default_answer)
end function
```

A couple of partners have told me, that they are unhappy with the discontinuation of INPUT from NAV and having to create pages for even the simplest questions. Running the following code:

```
IF CREATE(objScript,TRUE,TRUE) THEN
BEGIN
CR := ' '; CR[1] := 13;
objScript.Language := 'VBScript';
```

  

```
objScript.AddCode(
'function Input(question, title, default_answer)'+CR+
'  Input = InputBox(question, title, default_answer)'+CR+
'end function');
```

  

```
s := objScript.Eval('Input("How old are you?", "A simple question", "")');
MESSAGE(s);
END;
```

Brings up this dialog on my machine:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_8.png)

Who knows, maybe somebody can use this as an alternative to INPUT.

**Read the RoleTailored Client configuration file**

```
function ReadConfigFile()
set shell = CreateObject("WScript.Shell")
folder = shell.ExpandEnvironmentStrings("%LOCALAPPDATA%")
if folder = "" then folder = shell.ExpandEnvironmentStrings("%USERPROFILE%")&"Local SettingsApplication Data"
```

  

```
filename = folder&"MicrosoftMicrosoft Dynamics NAVClientUserSettings.config"
set fso = createobject("Scripting.FileSystemObject")
set file = fso.OpenTextFile(filename, 1)
ReadConfigFile = file.ReadAll()
end function
```

Note that I have NOT tested this function under Windows XP – I know that LOCALAPPDATA is not defined on Windows XP and I think the line:

  

```
if folder = "" then folder = shell.ExpandEnvironmentStrings("%USERPROFILE%")&"Local SettingsApplication Data"
```

should take care of finding the right folder – if anybody can confirm that, then add that as a comment to this post.

Bringing up a MESSAGE with the outcome of this function on my machine gives me this dialog:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/RunningCodeontheClientwithoutdeployingCO_D0AE/image_6.png)

I don’t know whether that could come in handy, but maybe it can spawn off some good ideas.

### Wrapping up

As you can see, you can do a lot of things in VB Script on the Client (or on the Server). There are a number of scripts you can find on the internet to work with the A/D (create, delete and enumerate users).

Of course there a limitations as to what you can do in VBScript and it isn’t a real alternative to writing a COM component, but for something it is easy and straightforward – and it doesn’t require any client side installation of components and this works in both Classic and RTC.

You can download the rename function from ClausL’s post about [sending e-mail with PDF attachments](http://blogs.msdn.com/nav-reporting/archive/2009/10/08/send-email-with-pdf-attachment-in-nav-2009.aspx). You will need to do copy, paste and maybe modify the other samples in order to use them.

Enjoy

_**Freddy Kristiansen  
**__PM Architect  
Microsoft Dynamics NAV_
