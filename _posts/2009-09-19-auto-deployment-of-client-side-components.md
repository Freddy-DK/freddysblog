---
layout: post
title: "Auto deployment of Client Side Components"
date: 2009-09-19 03:13:00
categories: ["Archive"]
tags: ["Add-Ins", "Client Tier", "COM", "Deployment", "Extensibility", "NAV 2009 SP1", "Service Tier", "Web Services"]
permalink: /2009/09/19/auto-deployment-of-client-side-components/
---

NOTE: There is an updated post on Auto deployment of Client Side Components [here](http://blogs.msdn.com/freddyk/archive/2009/12/09/auto-deployment-of-client-side-components-take-2.aspx).

When you install the RoleTailored Client on a number of clients, you might need to install a number of Client side components as well. This might not sound as too much of a problem when you need to install the client anyway – but lets say you install an ISV Add-on with a live customer, who already have 100 clients install – and now you need to install the objects to the database – AND you need to run to 100 computers and install Client side components.

Yes, you can do this with system management policies, but not all customers are running SMS and it would just be way easier if everything could be handled from the ISV Add-On and the Client Components could be auto deployed.

When doing this – it is still important, that IF the customer is running SMS and decide to deploy the Client Side components through system policies – then the auto deployment should just pick this up and accept that things are Ok.

### Two kinds of Client Side Components

NAV 2009 SP1 supports Add-Ins (Client Extensibility Controls) and Client side COM components (as NAV 2009) and the way these components are installed is very different.

Add-Ins needs to be placed in the **Add-Ins** folder under the RoleTailored Client folder on the Client and COM components can be installed wherever on the Client, but needs to registered in the registry with **regasm**.

Both Add-Ins and COM components might rely on other client side components, so it is important that we don’t just create a way of copying files to the Client – but we should instead create a way of launching a setup program on the client, which then installs the components. In my samples, I have one Setup program for every component, but an ISV could easily package all components together in one installation program and install them all in one go.

To install a client side component really isn’t that difficult – use FILE.DOWNLOAD with an MSI and that’s it. But how do we detect whether or not the component is installed already?

We cannot keep a list on the server side, since the computer might get re-installed or restored – we need a way of discovering whether a component is installed.

### Detecting whether a Client side COM component is installed

I will start with the COM component (since it will take a COM component to check whether an Add-In is installed). The COM component needs a CREATE statement to be initialized and if you check the return value of the CREATE statement – you know whether or not the COM component is executable. If not we launch a FILE.DOWNLOAD

```
IF NOT CREATE(mycomponent, TRUE, TRUE) THEN
FILE.DOWNLOAD(mycomponentinstaller);
```

Almost too simple right?

Now – I know that some people will say – well, what if I have an updated version of the COM component and it needs to be deployed?

My answer to that would be to change the COM signature, in effect making it a different COM component and allow them to be installed side-by-side. This would in effect mean that you might have multiple versions of COM components installed on a client, but they typically don’t take up a lot of space, and they don’t run if nobody uses them.

You could also create a function for checking the version number of the component like:

```
IF NOT CREATE(mycomponent, TRUE, TRUE) THEN
FILE.DOWNLOAD(mycomponentinstaller)
ELSE IF NOT mycomponent.CheckVersion(100) THEN
FILE.DOWNLOAD(mycomponentinstaller);
```

problem with this approach is, that NAV keeps a lock on the Client side component (event if you CLEAR(mycomponent)) due to performance reasons and your mycomponentinstaller will have to close the NAV client in order to update the component.

I like the solution better, where you just create a new GUID and thus a new component – so that is what I will describe here.

### Detecting whether an Add-In is installed on the Client

If you have installed the Server pieces of the Virtual Earth Integration (look here), but have a Client without the VEControl Add-In, this is how the FactBox will look:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_2.png)

Not very informative when you were expecting this:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_4.png)

But as you might know, we actually didn’t write any code to plugin the control and the Control Add-In error above is handled by the Client without actually notifying the Service Tier that anything is missing.

What we need to do, is to create one line of code in the INIT trigger of all pages, which uses an Add-In:

`ComponentHelper.CheckAddInNameKey('FreddyK.LargeVEControl','1c9f7ad47dba024b');`

and then of course create a function that actually checks that the Add-In is there and does a FILE.DOWNLOAD(addininstaller) if it isn’t.

Problem here is that we need a COM component in order to check the existence of an Add-In, and this COM component will have to run Client side (how else could it inspect the Add-ins folder – doh).

The INIT trigger is executed before anything is sent off to the Client and thus we can install the component and continue opening the page after we have done that. BTW the FILE.DOWNLOAD is NOT going to wait until the user actually finishes the setup program, so we will have to bring up a modal dialog telling the user to confirm that he has completed the setup.

BTW as you probably have figured out by now, the above line requires a registration of Add-Ins like:

`ComponentHelper.RegisterAddIn('FreddyK.LargeVEControl','1c9f7ad47dba024b','NAV Large Virtual Earth Control', 'NavVEControl.msi');`

In order to specify what file to download. Now I could have added this to the Check function to avoid a table – but I actually don’t think it belongs there.

### The ComponentHelper

So, what I have done is to collect some functionality that I find I use all the time in various samples in a Component called the ComponentHelper.

The functions are:

1.  Installation of Client side COM components (used by the majority of samples)
2.  Installation of Client side Add-Ins (used by all samples with Add-ins)
3.  Ability to Escape and Unescape strings (the method Web Services uses for encoding of company name – used in the Virtual Earth Integration)
4.  Ability to register a codeunit or page as Web Service from code (used by all samples using Web Services)
5.  Global information about the URL to my IIS and Web Service tier (used in Edit In Excel and Virtual Earth Integration)
6.  Modify metadata programmatically (all samples)

In fact I am hoping that these basic pieces of functionality will find their way into the base product in the future, where they IMO belong.

### Installation of Client side COM components

Every time you use a self built COM component (in this case the NAVAddInHelper), which you want to auto-deploy, you should create a function like this:

```
LoadAddInHelper(VAR NAVAddInHelper : Automation "'NAVAddInHelper'.NAVAddInHelper") Ok : Boolean
Ok := FALSE;
WHILE NOT CREATE(NAVAddInHelper,TRUE,TRUE) DO
BEGIN
IF NOT AskAndInstallCOMComponent('NAV AddIn Helper', 'NAVAddInHelper.msi') THEN
EXIT;
END;
Ok := TRUE;
```

and always invoke this when you want to create an instance of the Component (instead of having CREATE(NAVAddInHelper,TRUE,TRUE) scattered around the code.

```
AskAndInstallCOMComponent(Description : Text[80];InstallableName : Text[80]) Retry : Boolean
Retry := FALSE;
IF CONFIRM(STRSUBSTNO(TXT_InstallCOMComponent, Description)) THEN
BEGIN
Retry := InstallComponent(InstallableName);
END;
```

```
InstallComponent(InstallableName : Text[80]) Retry : Boolean
Retry := TRUE;
toFile := InstallableName;
fromFile := APPLICATIONPATH + 'ClientSetup'+InstallableName;
IF NOT FILE.EXISTS(fromFile) THEN
BEGIN
fromFile := APPLICATIONPATH + '..ClientSetup'+InstallableName;
END;
IF FILE.DOWNLOAD(fromFile, InstallableName, ", ", toFile) THEN
BEGIN
Retry := CONFIRM(TXT_PleaseConfirmComplete);
END;
```

as you can see from the code, the function will try to create the component until it succeeds or the user says No, I do not want to install the component. At this time I would like to mention a small bug in NAV 2009 SP1 – when you try to CREATE a COM component client side and it isn’t there, the Client will still ask you whether or not you want to run a client side component, but since the Control isn’t installed – it doesn’t know what to call it, meaning that you will get:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_8.png)

Now it is OK for the user to cancel this window because he doesn’t know what it is, but if he says **Never Allow** (silly choice to give the user:-)), he will have to delete personalization settings for automation objects to get this working again.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_thumb_5.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/AutodeploymentofClientSideComponents_72DD/image_12.png)

BTW If the user declines running a COM component – our code will see this as the component is not installed and ask him to install it.

### Installation of Client side Add-Ins

To check whether an Add-in is installed, we first check whether it is registered in the Client’s add-in table.

```
CheckAddInNameKey(AddInName : Text[220];PublicKeyToken : Text[20]) Found : Boolean
Found := FALSE;
IF NOT AddIn.GET(AddInName,PublicKeyToken) THEN
BEGIN
MESSAGE(STRSUBSTNO(TXT_AddInNotRegisterd, AddInName, PublicKeyToken));
EXIT;
END;
Found := CheckAddIn(AddIn."Control Add-in Name", AddIn."Public Key Token", AddIn.Description);
```

Without anything here – nothing works. After this we check our own table (in which we have information about what executable to download to the client)

```
CheckAddIn(AddInName : Text[220];PublicKeyToken : Text[20];Description : Text[250]) Found : Boolean
IF Description = " THEN
BEGIN
Description := AddInName;
END;
Found := FALSE;
IF LoadAddInHelper(NAVAddInHelper) THEN
BEGIN
WHILE NOT NAVAddInHelper.CheckAddIn(AddInName, PublicKeyToken) DO
BEGIN
IF NOT InstallableAddIn.GET(AddInName, PublicKeyToken) THEN
BEGIN
IF NOT CONFIRM(STRSUBSTNO(TXT_AddInNotFound, Description)) THEN
BEGIN
EXIT(FALSE);
END;
END
ELSE
EXIT(AskAndInstallAddIn(Description, InstallableAddIn.InstallableName));
END;
Found := TRUE;
END;
```

and last but not least – the method that installs the Add-In

```
AskAndInstallAddIn(Description : Text[80];InstallableName : Text[80]) Retry : Boolean
Retry := FALSE;
IF CONFIRM(STRSUBSTNO(TXT_InstallAddIn, Description)) THEN
BEGIN
Retry := InstallComponent(InstallableName);
END;
```

BTW, the method to register Add-Ins to this subsystem is

```
RegisterAddIn("Control Name" : Text[220];"Public Key Token" : Text[20];Description : Text[128];InstallableName : Text[80])
IF NOT AddIn.GET("Control Name", "Public Key Token") THEN
BEGIN
AddIn.INIT();
AddIn."Control Add-in Name" := "Control Name";
AddIn."Public Key Token" := "Public Key Token";
AddIn.Description := Description;
AddIn.INSERT(TRUE);
END;
IF NOT InstallableAddIn.GET("Control Name", "Public Key Token") THEN
BEGIN
InstallableAddIn.INIT();
InstallableAddIn."Control Add-in Name" := "Control Name";
InstallableAddIn."Public Key Token" := "Public Key Token";
InstallableAddIn.InstallableName := InstallableName;
InstallableAddIn.INSERT(TRUE);
END;
```

As you can see I could have extended the AddIn table – but I decided to go for adding a table instead, it doesn’t really matter.

### Ability to Escape and Unescape strings

In the Virtual Earth sample, I need to construct a URL, which contains the company name from NAV. Now with NAV 2009SP1 we use standard Escape and Unescape of strings in the URL, so I have added functions to ComponentHelper to do this. In fact, they just call a function in the C# COM component, which contains these functions.

### Ability to register a codeunit or page as Web Service from code

Instead of having to ask partners and/or users to register web services in the Web Service table or form, I have created this small function in the ComponentHelper to do this.

```
RegisterWebService(isPage : Boolean;"Object ID" : Integer;"Service Name" : Text[80];Published : Boolean)
IF isPage THEN
BEGIN
ObjType := WebService."Object Type"::Page;
END ELSE
BEGIN
ObjType := WebService."Object Type"::Codeunit;
END;
```

```
IF NOT WebService.GET(ObjType, "Service Name") THEN
BEGIN
WebService.INIT();
WebService."Object Type" := ObjType;
WebService."Object ID" := "Object ID";
WebService."Service Name" := "Service Name";
WebService.Published := Published;
WebService.INSERT();
COMMIT;
END ELSE
BEGIN
IF (WebService."Object ID" <> "Object ID") OR (WebService.Published<>Published)  THEN
BEGIN
WebService."Object ID" := "Object ID";
WebService.Published := Published;
WebService.MODIFY();
COMMIT;
END;
END;
```

### Global information about the URL to my IIS and Web Service tier

Again – a number of the samples I create will integrate from the RoleTailored Client to an application or a web site, which then again uses Web Services. I found out, that I needed a central way to find the URL of the right Web Service listener and the best way was to create a table in which I store the base URL (which would be ://WS/”>://WS/”>http://<server>:<port>/<instance>/WS/ (default [http://localhost:7047/DynamicsNAV/WS/](http://localhost:7047/DynamicsNAV/WS/)).

Also in the Virtual Earth I spawn up a browser (with HYPERLINK) and I need a location for the intranet server on which an application like the MAP would reside.

### Modify Metadata programmatically

I found that all my samples worked fine in the W1 version of NAV 2009 SP1, but as soon as I started to install them on other localized version, the pages on which I added actions etc. had been modified by local functionality and since there is no auto merge of pages, people would have to merge page metadata or find themselves loosing local functionality when they installed my samples.

I have added 4 functions:

`GetPageMetadata(Id : Integer;VAR Metadata : BigText)`

`SetPageMetadata(Id : Integer;Metadata : BigText)`

`AddToMetadata(Id : Integer;VAR Metadata : BigText;Before : Text[80];Identifier : Text[80];Properties : Text[800]) result : Boolean`

`AddToPage(Id : Integer;VersionList : Text[30];Before : Text[80];Identifier : Text[80];Properties : Text[800]`

where the last function just call the three other (Get, Add, Set metadata).

I am not very proud of the way these functions are made – they just search for a line in the exported text file and inserts some metadata but they meet the needs.

`As an example on how these functions are used you will find:`

```
// Read Page Metadata
ComponentHelper.GetPageMetadata(PAGE::"Customer Card", Metadata);
```

```
// Add Map Factbox
ComponentHelper.AddToMetadata(PAGE::"Customer Card", Metadata, '    { 1900383207;1;Part   ;',
'    { 66031;1  ;Part      ;',
' SubFormLink=No.=FIELD(No.); PagePartID=Page66030 }')
OR
```

```
// Add View Area Map Action
ComponentHelper.AddToMetadata(PAGE::"Customer Card", Metadata, '      { 82      ;1   ;ActionGroup;', '      { 66030   ;2   ;Action    ;',
' CaptionML=[ENU=View Area Map]; OnAction=VAR MAP : Codeunit 66032; BEGIN MAP.OpenCustomerMAPInBrowser(Rec); END; }');
```

```
// Write Page Metadata back
ComponentHelper.SetPageMetadata(PAGE::"Customer Card", Metadata);
```

So basically – it reads the metadata for the page, checks whether the action already has been added (the string ‘      { 66030   ;2   ;Action    ;’ exists already). If not it searches for the string ‘      { 82      ;1   ;ActionGroup;’ and inserts the action below that. Not pretty – but it works.

### The Visual Studio piece

As mentioned earlier a couple of functions are needed in a client side COM component.

The Escape and Unescape functions really doesn’t do anything:

```
public string EscapeDataString(string str)
{
return Uri.EscapeDataString(str);
}
```

```
public string UnescapeDataString(string str)
{
return Uri.UnescapeDataString(str);
}
```

and the essence of the CheckAddIn is the code found in the LoadAddIn function of the AddIn class:

`Assembly assembly = Assembly.LoadFrom(dll);`

```
this.publicKey = "";
foreach (byte b in assembly.GetName().GetPublicKeyToken())
{
this.publicKey += string.Format("{0:x2}", b);
}
```

```
Type[] types = assembly.GetTypes();
foreach (Type type in types)
{
foreach (System.Attribute att in System.Attribute.GetCustomAttributes(type))
{
ControlAddInExportAttribute expAtt = att as ControlAddInExportAttribute;
if (expAtt != null && !string.IsNullOrEmpty(expAtt.Name))
{
if (!isAddIn)
{
this.controlNames = new List<string>();
isAddIn = true;
}
this.controlNames.Add(expAtt.Name);
}
}
}
```

Which loads an Add-In, finds the public key token and the registered controls. The rest is really simple – check whether one of the Add-Ins in fact is the one we are looking for – else install it…

`The Visual Studio solution also contains a setup project for generating the .msi file which needs to be placed in the ClientSetup folder.`

### `Putting it all together`

`So, now we have a .fob file and an .msi file which we need to install on the Service Tier – so why don't we create a Setup project, which contains this .fob (install that in a ServerSetup folder) and the .msi (install that in the ClientSetup folder).`

Doing this makes installing the ComponentHelper a 3 step process:

1.  `Install ComponentHelper.msi on the Service Tier`
2.  `Import a .fob from the ServerSetup folder`
3.  `Run a codeunit which registers the necessary stuff`

`In fact I am trying to make all the demos and samples installable like the ComponentHelper itself – so that anybody can download cool samples and get a sexy Microsoft Dynamics NAV 2009 SP1 – to work with.`

ComponentHelper1.01.zip (which contains ComponentHelper1.01.msi) can be downloaded [here](http://www.freddy.dk/ComponentHelper1.01.zip).

If you don’t fancy downloading the .msi (for whatever reason) – the source to NAVAddHelper can be downloaded [here](http://www.freddy.dk/NAVAddInHelper1.01.zip) and the ComponentHelper objects can be downloaded [here](http://www.freddy.dk/ComponentHelperObjects1.01.zip).

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
