---
layout: post
title: "The BingMaps Extension (and some tips and tricks)"
date: 2017-01-19 13:59:10
categories: ["Archive"]
tags: ["Azure", "BingMaps", "Extension", "Image", "NAV", "NAV 2017"]
permalink: /2017/01/19/the-bingmaps-extension-and-some-tips-and-tricks/
---

On the Azure VM, you will find a couple of extensions: BingMaps, O365 and MSBand.

These extensions are primarily there in order to show how you would do certain things in extensions. It is my intention to create an app in AppSource for each of these apps as well when time allows. In this post, I will describe how the BingMaps extension was built and how it works.

### Publishing the BingMaps extension

When you deploy a DEMO environment using [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy) you will have the option of entering a BingMaps Key in the template and the BingMaps extension will be published, installed and configured for you. This blog post expects you to NOT specify a BingMaps key in the Azure Deployment template.

In order to publish the BingMaps extension, you need to run C:\\DEMO\\Install BingMaps.ps1 on the VM and select the default (press ENTER) answer to both questions.

1.  Specify Region Format
2.  BingMaps Key

[![bingmapsinstaller2](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/8fe5e-bingmapsinstaller2-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/8fe5e-bingmapsinstaller2.png)

When deploying throught the Azure deployment template, the same script will run, but the questions will be answered with the parameters from the template.

If you specify a BingMaps Key to the script, the script will publish, install and configure the App. If you leave the BingMaps Key empty, it will only publish the app, which on a conceptual level is the same as publishing to AppSource and then allowing people to install it afterwards.

### Installing the BingMaps extension

In the Actions Ribbon, locate Services and Extensions and select Extensions.

You should now see the BingMaps integration extension as _Not Installed_ in the extension management page:

[![bingmapsnotinstalled](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/35b35-bingmapsnotinstalled-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/35b35-bingmapsnotinstalled.png)

If you click the extension, it will launch a wizard, which ends up with an _Install_ button, which you can press to install the extension.

After installing the extension, the extension will automatically open the BingMaps Integration Setup page, in which you can configure the extension:

[![bingmapssetup](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/a5fb8-bingmapssetup-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/a5fb8-bingmapssetup.png)

You can find information about how to pop up a dialog to _configure the app upon successful installation_ a little later in this post.

If you do not configure the App at this time, the BingMaps extension will be installed, but will not be working. It cannot geocode customer addresses if no BingMaps Key is specified. In this case, the BingMaps app will notify you on your role center, that the BingMaps extension isn’t properly setup yet:

[![bingmapsisntsetup](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/cb3e8-bingmapsisntsetup-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/cb3e8-bingmapsisntsetup.png)

You can find information about how to _display notification that app is not configured on your role center_ a little later in this blog post.

Clicking the link in the notification will take you to the setup dialog, where you specify your BingMaps key, test it and click _Geocode All Customers_.

Go [here](http://msdn.microsoft.com/en-us/library/ff428642.aspx) if you don’t have a BingMaps Key, it is free for demo usage and the assistedit button on the BingMaps key field actually takes you to this web site.

Geocoding of customers will happen in the background and you can continue working while this happens.

You can always get back to the BingMaps Integration Setup dialog if you navigate to Services & Extensions -> Service Connections – there you will also find the BingMaps Integration Setup.

All in all – this is the kind of experience I think users should have when installing an app, help them as much as possible succeeding in using the app.

### Lets look at the source

If you look in the folder C:\\DEMO\\BingMaps\\Sources on the VM you will find a number of files:

-   .zip files are Control Add-ins which will be installed with the extension
-   .xml files are definitions on web services to be published, when the app is installed
-   .delta files are the actual AL source files
-   .txt files are translations

In order to develop the extension, we need to create a development environment for the extension and for this I am currently using the Extension Development Shell, which also is on the VM.

Run _C:\\DEMO\\Install Extensions Development Shell.ps1_ and specify your developer license. After this, open the Extension Development Shell and use this command:

```
new-devinstance -devinstance dev -appfolder "C:\DEMO\BingMaps"
```

[![bingmapsnewdev](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/7653f-bingmapsnewdev-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/7653f-bingmapsnewdev.png)

After this, you should have a few extra shortcuts on the desktop for the _dev_ developer instance.

If you start the “dev Development Environment” you will get a development environment with all the objects for the BingMaps Extension.

[![bingmapsobjects](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/63290-bingmapsobjects-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/63290-bingmapsobjects.png)

### Modified standard objects

3 objects have been modified: Customer table, Customer Card page and Customer List page.

The Customer table has 4 new fields: Geocoded, Latitude, Longitude and Zoom.

Both pages have a new fact box inserted in the list of fact boxes.

### New objects

#### Table::BingMaps Setup

Contains setup information for the BingMaps extension, which is the BingMaps Key and a boolean field specifying whether or not the key has been tested and found OK.

#### Codeunit::Geocode

Contains the code to call the BingMaps API to geocode customers. The GeoCode function is a TryFunction.

#### Codeunit::BingMaps Setup

Contains code for getting and setting the BingMaps Key a method to geocode all customers synchronously. This codeunit is exposed as Web Services and called from the PowerShell scripts to automate the configuration of the BingMaps when using [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy).

#### Codeunit::BingMaps Events

Contains event subscribers for events.

#### Codeunit::BingMaps Notifications

Contains the code and event subscribers needed to popup the notificationon the Role Center.

#### Page::CustomerMap

Is the fact box showing the Customer Map.

#### Page::CustomerLocation

The CustomerLocation page is exposed as Web Services and used to retrieve information about customers and show them on a map.

#### Page::BingMaps Integration Setup

The BingMaps Integration Setup dialog.

Instead of going through every single object, I will describe how to achieve some of the things, the BingMaps extension does.

### Update Latitude and Longitude for the customer when a customer is modifying

In the _BingMaps Events_ codeunit, you will find two methods hooked up to events on the _Customer_ table.

One is hooked up to the _OnBeforeInsertEvent_ and one to the _OnBeforeModifyEvent_.

```
LOCAL [EventSubscriber] InsertCustomer(VAR Rec : Record Customer;RunTrigger : Boolean)
Rec.Geocoded := 0;
Geocode.GeocodeCustomer(Rec);

LOCAL [EventSubscriber] ModifyCustomer(VAR Rec : Record Customer;VAR xRec : Record Customer;RunTrigger : Boolean)
Rec.Geocoded := 0;
Geocode.GeocodeCustomer(Rec);
```

In both cases clear the flag that indicates that the Customer has been geocoded and then call the GeocodeCustomer function.

The GeocodeCustomer function will try geocoding the specific address and if that fails, it will geocode the city and country.

Reason for this is, that some of the street names in the NAV demo data are not real, or rather they are real, but they don’t exist in that city.

The actual geocoding is done by calling the unstructured REST API from BingMaps, documentation is available here: [https://msdn.microsoft.com/en-us/library/ff701714.aspx](https://msdn.microsoft.com/en-us/library/ff701714.aspx)

Reason for the unstructured approach is to leave it to BingMaps to figure out how to decode the address fields. I have not tested this with addresses from all countries.

### Make the BingMaps Integration Setup appear in Service Connections

Services can register themselves in the Service Connection dialog :

[![bingmapsserviceconnection](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/d8fd9-bingmapsserviceconnection-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/d8fd9-bingmapsserviceconnection.png)

This is done by hooking up to the _OnRegisterServiceConnection_ on the _Service Connection_ Table:

[![bingmapsprop3](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/c3806-bingmapsprop3-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/c3806-bingmapsprop3.png)

and the code for adding the BingMaps setup to the Service Connection looks like this:

```
LOCAL [EventSubscriber] RegisterServiceConnection(VAR ServiceConnection : Record "Service Connection")
IF NOT BingMapsSetup.GET THEN
BEGIN
  IF NOT BingMapsSetup.WRITEPERMISSION THEN
    EXIT;
  BingMapsSetup.INIT;
  BingMapsSetup.INSERT;
END;
RecRef.GETTABLE(BingMapsSetup);
ServiceConnection.Status := ServiceConnection.Status::Disabled;
IF BingMapsSetup."BingMaps Key OK" THEN
  ServiceConnection.Status := ServiceConnection.Status::Enabled;
ServiceConnection.InsertServiceConnection(ServiceConnection, RecRef.RECORDID, txtBingMapsIntegrationSetup, '', PAGE::"BingMaps Integration Setup");
```

At first we check permissions and then we add the line with status set to Disabled or enabled depending on whether or not the BingMaps Key is OK.

### Configure the app upon successful installation

When you have installed an app, you can automatically popup a dialog and ask the user to configure the app:

[![bingmapssetup](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/a5fb8-bingmapssetup-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/a5fb8-bingmapssetup.png)

This is done by hooking up to the _OnAfterActionEvent_ on the _Extension Details_ page and specifying that it is the _Install_ action you want to hook up to.

[![bingmapsprop2](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/ad905-bingmapsprop2-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/ad905-bingmapsprop2.png)

The code for your event subscriber should check whether the extension was installed and if this is the case, it should invoke your dialog:

```
LOCAL [EventSubscriber] Installed(VAR Rec : Record "NAV App")
IF (Rec.Name = 'BingMaps Integration') THEN
  BingMapsSetup.RUNMODAL;
```

Remember to check for the name of the extension installed, else your dialog will popup whenever you install any extension.

### Display notification that app is not configured on your role center

When you open your role center without having specified a valid and tested BingMaps Key in the BingMaps Integration Setup dialog you will see this:

[![bingmapsisntsetup](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/cb3e8-bingmapsisntsetup-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/cb3e8-bingmapsisntsetup.png)

This is handled in _Codeunit::BingMaps Notifications._

This method:

```
LOCAL ShowBingMapsSetupWarning()
BingMapsSetupOk := BingMapsSetup.FINDFIRST;
IF BingMapsSetupOk THEN BEGIN
  BingMapsSetupOk := BingMapsSetup."BingMaps Key OK";
END;
IF NOT BingMapsSetupOk THEN BEGIN
  Notification.ID('3EBC1525-C2D4-4797-8B28-BA2D0C6294B5');
  Notification.SCOPE(NOTIFICATIONSCOPE::LocalScope);
  Notification.MESSAGE(txtBingMapsIntegrationNotSetup);
  Notification.ADDACTION(txtSetupBingMaps, CODEUNIT::"BingMaps Notifications", 'SetupBingMapsIntegration');
  Notification.SEND;
END;
```

Will check whether the BingMaps Key is OK and if not, create a notification, which will be displayed in the current window.

In order for this notification to get displayed in the Role Center, we need to hook up events on all RoleCenters. Unfortunately there are no OnGlobalOpenPage yet, meaning that you will find a number of event subscriptions:

```
LOCAL [EventSubscriber] O365ActivitiesOnOpenPage(VAR Rec : Record "Activities Cue")
ShowBingMapsSetupWarning;

LOCAL [EventSubscriber] AccountManagerActivitiesOnOpenPage(VAR Rec : Record "Finance Cue")
ShowBingMapsSetupWarning;

LOCAL [EventSubscriber] AccPayablesActivitiesOnOpenPage(VAR Rec : Record "Finance Cue")
ShowBingMapsSetupWarning;
```

The reason for hooking up the event to the Activities part is, that you cannot hook up _OnOpenPage_ events to the Role Center itself because it doesn’t have a _SourceTable_ defined.

Sample subscription properties:

[![bingmapsprop1](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/86934-bingmapsprop1-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/86934-bingmapsprop1.png)

The SetupBingMapsIntegration method, which is called when the user clicks the action in the notification will simply run the BingMaps Integration Setup dialog:

```
SetupBingMapsIntegration(Notification : Notification)PAGE.RUNMODAL(PAGE::"BingMaps Integration Setup");
```

### Display a map with all customers

The BingMaps extension also has support allowing a web site to show all customers on a map

[![bingmapsmap](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/90801-bingmapsmap-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/90801-bingmapsmap.png)

The CustomerLocation page includes all information needed for displaying customers on the map (including a URL to open the customer page on a specific customer).

[![bingmapscustomerlocation](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/3828a-bingmapscustomerlocation-1.png)](/assets/images/2017/the-bingmaps-extension-and-some-tips-and-tricks/3828a-bingmapscustomerlocation.png)

The URL is a calculated field, which is calculated in the _OnAfterGetRecord_ trigger.

```
OnAfterGetRecord()
URL := GETURL(CLIENTTYPE::Web, COMPANYNAME, OBJECTTYPE::Page, PAGE::"Customer Card", Rec);
```

When using the Azure Image, the BingMaps script will place a website called _map.aspx_ in your WebClient folder in order to give you this functionality. When this app is published to AppSource, I cannot place an .aspx file on the server, meaning that I would have to create a publicly hosted web site, which can get data from the NAV version it is attached to.

The map.aspx uses C# as server side scripting and the code for getting all customers is:

```
public string GetCustomers()
{
  try
  {
    var wc = new WebClient();
    wc.Credentials = new NetworkCredential(GetUserName(), GetWsKey());
    return wc.DownloadString(GetPublicODataBaseUrl() + "CustomerLocation?$format=json&$callback=dataloaded&$filter=Geocoded%20eq%201");
  } catch {
    return "";
  }
}
```

Which basically creates a _Javascript_ function call to _dataloaded_ with all customers in a json string (called JSONP). This means that all data retrieval is done server side and only the needed data will be transferred over the wire. You could set filters on the Latitude and Longitude fields if you want to only retrieve the customers visible right now, but then you would have to use several calls to the NAV server to get customers when ever you zoom in/out or move the viewport.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
