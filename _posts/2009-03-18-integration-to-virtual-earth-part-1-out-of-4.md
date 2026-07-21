---
layout: post
title: "Integration to Virtual Earth – Part 1 (out of 4)"
date: 2009-03-18 18:12:54
categories: ["Archive"]
tags: ["Geocode", "Latitude", "Longitude", "Mappoint", "Virtual Earth"]
permalink: /2009/03/18/integration-to-virtual-earth-part-1-out-of-4/
---

### Disclaimer

Please note that this is an example on how to do geocoding and Virtual Earth integration. There are probably a lot of different ways to do the same, and this might not work in all areas of the world. I do however think that there is enough information in the post that people can make it work anywhere and the good thing about the samples and demos I post in my blog is, that they are free to be used as partners and customers see fit. My samples doesn’t come with any warranty and if installed at customer site, the partner and/or customer takes full responsibility.

Following this sample also does not release you from following any license rules of the products used in the blog (note that I don’t know these rules)

### Mappoint or Virtual Earth

As you probably know there already is an integration to Mappoint in NAV 2009. This integration makes it possible to open a map for a given customer (or create route description on how to get there). The way it works is, that when you request an online map, a URL is created which will open a map centered on the requested customer. It is a one way integration – meaning that we can see a map and/or calculate routes.

But… – it is a one way integration. Wouldn’t it be cool if we could request NAV for all customers in a range of 10 miles from another customer – display all customers on a map and have information included directly on Virtual Earth with phone numbers, orders and other things.

That is what this is all about…

But in order to do that, we need more information on our customers than just an address, a city and a country, as this information is hard to query. How would NAV know that Coventry is near Birmingham – if we don’t tell it.

The idea is off course to add geocode information to all customers in our customer table.

We can do this the hard way (typing them in), the other “easy” way (create an automation object which does the trick for you).

### Latitude and Longitude

I am not (and I wouldn’t be capable of) trying to describe in details what Latitude and Longitude is – if you want this information you should visit

[http://en.wikipedia.org/wiki/Latitude](http://en.wikipedia.org/wiki/Latitude "http://en.wikipedia.org/wiki/Latitude")

and

[http://en.wikipedia.org/wiki/Longitude](http://en.wikipedia.org/wiki/Longitude "http://en.wikipedia.org/wiki/Longitude")

Not that it necessarily helps a lot, but there you have it.

A simpler explanation can be found on [http://www.worldatlas.com](http://www.worldatlas.com), which is also, where this image is from

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_7.png "image")](http://www.worldatlas.com/aatlas/imageg.htm)

_Click the image to take you directly to the description._

In this map, coordinates are described as degrees, minutes and seconds + a direction in which this is from the center (N, S, E, W).

There are different ways to write a latitude and a longitude – in my samples I will be using decimal values, where Latitude is the distance from equator (positive values are on the northern hemisphere and negative values are on the southern) and Longitude is the distance from the prime meridian (positive values are going east and negative values are going west). This is the way Microsoft Virtual Earth uses latitude and longitude in the API.

Underneath you will find a map with a pushpin in 0,0.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_9.png)

Another location (well known to people in the Seattle area) is Latitude = 47.6 and Longitude = -122.33, which on the map would look like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_thumb_4.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_11.png)

Yes – the Space Needle.

I think this is sufficient understanding to get going.

### Preparing your customer table

First of all we need to create two fields in the customer table, which will hold the geocode information of the customer.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_thumb_5.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_13.png)

Set the Decimalplaces for both fields to 6:8 and remember to create a key, including the two fields (else your searches into the customer table will be slow)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_6.png)

You also need to add the fields to the Customer Task Page, in order to be able to edit the values manually if necessary.

### Virtual Earth Web Services

In order to use the Microsoft Virtual Earth Web Services you need an account. I do not know the details about license terms etc., but you can visit

[https://mappoint-css.live.com/MwsSignup](https://mappoint-css.live.com/MwsSignup)

for signing up and/or read about the terms. I do know that an evaluation developer license is free – so you can sign up for getting one of these – knowing of course that you probably cannot use this for your production data – please contact [maplic@microsoft.com](mailto:maplic@microsoft.com) for more information on this topic.

Having signed up for a developer account you will get an account ID and you will set a password which you will be using in the application working with the Virtual Earth Web Services. This account ID and Password is used in your application when connecting to Web Services and you manage your account and/or password at

[https://mappoint-css.live.com/CscV3](https://mappoint-css.live.com/CscV3 "https://mappoint-css.live.com/CscV3")

You will also find a site in which you can type in your Account ID and password to test whether it is working.

On these sites there are a number of links to Mappoint Web Services – this is where the confusion started for me…

I wrote some code towards Mappoint and I quickly ran into problems having to specify a map source (which identifies the continent in which I needed to do geocoding). I really didn’t want this, as this would require setup tables and stuff like that in my app. After doing some research I found out that Virtual Earth exposes Web Services to the Internet, which are different from Mappoint (I do not have any idea why). A description of the Virtual Earth Web Services API can be found here:

[http://msdn.microsoft.com/en-us/library/cc980922.aspx](http://msdn.microsoft.com/en-us/library/cc980922.aspx)

and a description of the geocode service can be found here:

[http://msdn.microsoft.com/en-us/library/cc966817.aspx](http://msdn.microsoft.com/en-us/library/cc966817.aspx)

and yes, your newly assigned account and password for Mappoint services also works for Virtual Earth Web Services.

The way it works is, that when connecting to Virtual Earth Web Services you need to supply a valid security token. This token is something you request from a different web service and when requesting this token, you specify the number of minutes the token should be valid (Time-To-Live).

Confused?

### Creating a COM automation object for geocoding addresses

If you think you have the grasp around the basics of the Virtual Earth Web Services – let’s get going…

First of all – fire up your Visual Studio 2008 SP1 and create a new Class Library (I called mine NavMaps).

Add the CLSCompliant(true) to the AssemblyInfo.cs file (I usually to this after the ComVisible(false) line).

```
// Setting ComVisible to false makes the types in this assembly not visible
// to COM components.  If you need to access a type in this assembly from
// COM, set the ComVisible attribute to true on that type.
[assembly: ComVisible(false)]
[assembly: CLSCompliant(true)]
```

After this you need to sign the assembly. Do this by opening properties of project, go to the Signing TAB,  check the “Sign the assembly” checkbox and select new – type in a filename and password protect the key file if you want to (I usually don’t).

Next thing is to create the COM interface and Class – the interface we want is:

```
[ComVisible(true)]
[Guid("B1F26FE7-0EA0-4883-BD6A-0398F8D2B139"), InterfaceType(ComInterfaceType.InterfaceIsDual)]
public interface INAVGeoCode
{
string GetLocation(string query, int confidence, ref double latitude, ref double longitude);
}
```

For the implementation we need a Service Reference to:

[http://staging.dev.virtualearth.net/webservices/v1/geocodeservice/geocodeservice.svc?wsdl](http://staging.dev.virtualearth.net/webservices/v1/geocodeservice/geocodeservice.svc?wsdl "http://staging.dev.virtualearth.net/webservices/v1/geocodeservice/geocodeservice.svc?wsdl ")

called

**GeocodeService**

Note that the configuration of this Service Reference will be written in app.config – I will touch upon this later.

The implementation could be:

```
[ComVisible(true)]
[Guid("9090DF4C-FB24-4a4b-9E49-3924353A6040"), ClassInterface(ClassInterfaceType.None)]
public class NAVGeoCode : INAVGeoCode
{
private string token = null;
private DateTime tokenExpires = DateTime.Now;
```

    

```
/// <summary>
/// Geocode an address and return latitude and longitude
/// Low confidence is used for geocoding demo data – where the addresses really doesn't exist:-)
/// </summary>
/// <param name="query">Address in the format: Address, City, Country</param>
/// <param name="confidence">0 is low, 1 is medium and 2 is high confidence</param>
/// <param name="latitude">returns the latitude of the address</param>
/// <param name="longitude">returns the longitude of the address</param>
/// <returns>Error message if something went wrong</returns>
public string GetLocation(string query, int confidence, ref double latitude, ref double longitude)
{
try
{
// Get a Virtual Earth token before making a request
string err = GetToken(ref this.token, ref this.tokenExpires);
if (!string.IsNullOrEmpty(err))
return err;
```

            `GeocodeService.GeocodeRequest geocodeRequest = new GeocodeService.GeocodeRequest();`

            

```
// Set the credentials using a valid Virtual Earth token
geocodeRequest.Credentials = new GeocodeService.Credentials();
geocodeRequest.Credentials.Token = token;
```

            

```
// Set the full address query
geocodeRequest.Query = query;
// Set the options to only return high confidence results
GeocodeService.ConfidenceFilter[] filters = new GeocodeService.ConfidenceFilter[1];
filters[0] = new GeocodeService.ConfidenceFilter();
switch (confidence)
{
case 0:
filters[0].MinimumConfidence = GeocodeService.Confidence.Low;
break;
case 1:
filters[0].MinimumConfidence = GeocodeService.Confidence.Medium;
break;
case 2:
filters[0].MinimumConfidence = GeocodeService.Confidence.High;
break;
default:
return "Wrong value for confidence parameter";
}
```

            

```
GeocodeService.GeocodeOptions geocodeOptions = new GeocodeService.GeocodeOptions();
geocodeOptions.Filters = filters;
```

            `geocodeRequest.Options = geocodeOptions;`

            ```
// Make the geocode request
GeocodeService.IGeocodeService geocodeService = new ChannelFactory<GeocodeService.IGeocodeService>(new BasicHttpBinding(), new EndpointAddress("http://staging.dev.virtualearth.net/webservices/v1/geocodeservice/GeocodeService.svc")).CreateChannel();
````GeocodeService.GeocodeResponse geocodeResponse = geocodeService.Geocode(geocodeRequest);`

            

```
if (geocodeResponse.Results.Length == 0 || geocodeResponse.Results[0].Locations.Length == 0)
{
return "No locations found";
}
latitude = geocodeResponse.Results[0].Locations[0].Latitude;
longitude = geocodeResponse.Results[0].Locations[0].Longitude;
```

            

```
return "";
}
catch (Exception ex)
{
return ex.Message;
}
}
}
```

Before we add the last function – **GetToken** – I would like to draw attention to the line:

`GeocodeService.IGeocodeService geocodeService = new ChannelFactory<GeocodeService.IGeocodeService>(new BasicHttpBinding(), new EndpointAddress("http://staging.dev.virtualearth.net/webservices/v1/geocodeservice/GeocodeService.svc&#8221;)).CreateChannel();`

This isn’t normally the way you would instantiate the service class. In fact normally you would see:

`GeocodeService.GeocodeServiceClient geocodeService = new GeocodeService.GeocodeServiceClient();`

which is simpler, looks nicer and does the same thing – so why bother?

### Which configuration file to use?

The primary reason is to avoid using the configuration file. The standard way of instantiating the Service Client is looking for a number of settings in the appSettings section in the config file – and you wouldn’t think that should be a problem – but it is. The problem is that it uses the application configuration file – NOT the DLL config file, and I couldn’t find any way to make it read the DLL config file for these settings.

So if my NavMaps.dll should be accessible from the classic client, I would have to create a finsql.exe.config with the right configuration. Microsoft.Dynamics.Nav.Client.exe.config would be the configuration file for the Roletailored Client (if we are running the automation client side) and I would have to find a way to merge the config settings into the service tier configuration if we are running the automation server side.

So, to avoid all that crap (and severe deployment problems), I instantiate my WCF Client manually through code – meaning that no app.config is necessary.

### Requesting a Virtual Earth Security Token

First you need to add a Web Reference to

[https://staging.common.virtualearth.net/find-30/common.asmx?wsdl](https://staging.common.virtualearth.net/find-30/common.asmx?wsdl "https://staging.common.virtualearth.net/find-30/common.asmx?wsdl")

called

**TokenWebReference**

The code for the GetToken could look like this:

```
/// <summary>
/// Check validity of existing security token and request a new
/// Security Token for Microsoft Virtual Earth Web Services if necessary
/// </summary>
/// <param name="token">Security token</param>
/// <param name="tokenExpires">Timestamp for when the token expires</param>
/// <returns>null if we have a valid token or an error string if not</returns>
```

```
private string GetToken(ref string token, ref DateTime tokenExpires)
{
if (string.IsNullOrEmpty(token) || DateTime.Now.CompareTo(tokenExpires) >= 0)
{
// Set Virtual Earth Platform Developer Account credentials to access the Token Service
TokenWebReference.CommonService commonService = new TokenWebReference.CommonService();
commonService.Credentials = new System.Net.NetworkCredential("<your account ID>", "<your password>");
```

        

```
// Set the token specification properties
TokenWebReference.TokenSpecification tokenSpec = new TokenWebReference.TokenSpecification();
IPAddress[] localIPs = Dns.GetHostAddresses(Dns.GetHostName());
foreach (IPAddress IP in localIPs)
{
if (IP.AddressFamily == System.Net.Sockets.AddressFamily.InterNetwork)
{
tokenSpec.ClientIPAddress = IP.ToString();
break;
}
}
// Token is valid an hour and 10 minutes
tokenSpec.TokenValidityDurationMinutes = 70;
```

        

```
// Get a token
try
{
// Get token
token = commonService.GetClientToken(tokenSpec);
// Renew token in 1 hour
tokenExpires = DateTime.Now.AddHours(1);
}
catch (Exception ex)
{
return ex.Message;
}
}
return null;
}
```

Note that I am giving the token a TTL for 70 minutes – but renew after 60 – that way I shouldn’t have to deal with tokens being expired.

<your account ID> should be replaced by your account ID and <your password> should be replaced with your password.

**Using the geocode automation object**

Having build the assembly we need to put it to play.

After building the assembly we need to place it in the “right” folder – but what is the right folder?

It seems like we have 4 options

1.  In the Classic folder
2.  In the RoleTailored Client folder
3.  In the Service Tier folder
4.  Create a Common folder (next to the Classic, RoleTailored and Service Tier folders) and put it there

Let me start by excluding the obvious choice, the Service Tier folder. The reason is, that if you have multiple Service Tiers, they will all share the same COM automation object and you don’t really know which directory the assembly is read from (the one where you did the last regasm). You could accidently delete the Service Tier in which the registered assembly is located and thus end up having denial of service.

RoleTailored Client folder also seems wrong – there is absolutely no reason for running this object Client side – it should be on the Service Tier and there might not be a RoleTailored Client on the Service Tier. The same is case with the Classic folder – so I decided to create a Common folder and put the DLL there. Another reason for selecting the Common folder is, that you can install it on the Client (for being able to run it in the Classic Client) in a similar way as on the server.

Your situation might be different and you might select a different option.

After copying the DLL to the Common folder, we need to register the DLL and make it available as a COM automation object.

`C:\Windows\Microsoft.NET\Framework\v2.0.50727\regasm NAVMaps.dll /codebase /tlb`

Is the command to launch.

### Creating a codeunit for geocoding your customers

What I normally do in situations like this is, to create a codeunit, which initializes itself when you Run it.

In this case Running the codeunit should run through all customers and geocode them.

```
OnRun()
IF cust.FIND('-') THEN
BEGIN
REPEAT
err := UpdateLatitudeAndLongitude(cust);
IF (err <> ") THEN
BEGIN
IF NOT CONFIRM('Customer '+cust."No."+ ' – Error: '+err + ' – Continue?') THEN EXIT;
END;
UNTIL cust.NEXT = 0;
END;
```

Whether or not you want an error here or not is kind of your own decision.

The function, that does the job looks like:

```
UpdateLatitudeAndLongitude(VAR cust : Record Customer) error : Text[1024]
```

```
CREATE(NavMaps, TRUE, FALSE);
country.GET(cust."Country/Region Code");
query := cust.Address+', '+cust."Address 2″+', '+cust.City+', '+', '+cust."Post Code"+', '+country.Name;
error := NavMaps.GetLocation(query, 2, cust.Latitude, cust.Longitude);
IF (error = ") THEN
BEGIN
cust.MODIFY();
END ELSE
BEGIN
query := cust.City + ', ' + country.Name;
error := NavMaps.GetLocation(query, 0, cust.Latitude, cust.Longitude);
IF (error = ") THEN
BEGIN
cust.MODIFY();
END;
END;
```

Local variables looks like this

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_15.png)

Now we can discuss whether this is the right way to go around it – problem for me is, that the demodata are not real addresses – meaning that the street names are fine, city names are fine – but the street and number doesn’t exist in the city.

So – what I do here is to start building a query with Address, City, Post code, Country Name – and tell Virtual Earth to make a High confidence search – this will return the location of correct addresses. In the demo database, this returns absolutely nothing. Next thing is to just give me the location of the city in the country – whether you want to do this for your customer table is kind of your own decision, but it works for the demo data.

BTW – i tried this with addresses in Denmark (my former) and the US (my current) – and the format seems to suit Virtual Earth in these countries, but I didn’t try a lot of other addresses. The advantage of using the query is really to give Virtual Earth the freedom to interpret and look at the address – and it does a pretty good job. If anybody experiences problems with this in certain countries – please let me know (with a suggestion as to what should be changed) and I will include this.

### Can’t make it work?

If (for some reason) this code doesn’t work for you – but you really want to get on with the next two parts of this walkthrough, I have included a table here with Latitudes and Longitudes for all the customers in the W1 database.

Copy the entire table into a clean excel spreadsheet and use Edit In Excel to copy the data from the two columns to the two columns in your customer table in one go and save it back to NAV. Note that the Latitude and Longitude is only available in Excel if you

1.  Have added the fields to the customer card
2.  Updated the Web Reference in the Edit In Excel project to the customer card after doing step 1 (if you already had Edit In Excel running)

If you don’t have the Edit In Excel – you can find it here [http://blogs.msdn.com/freddyk/archive/tags/Excel/default.aspx](http://blogs.msdn.com/freddyk/archive/tags/Excel/default.aspx "http://blogs.msdn.com/freddyk/archive/tags/Excel/default.aspx")

<table border="0" width="624" cellspacing="0" cellpadding="0"><tbody><tr><td width="79"><strong>No</strong></td><td width="261"><strong>Name</strong></td><td width="140"><strong>Latitude</strong></td><td width="142"><strong>Longitude</strong></td></tr><tr><td width="79">01121212</td><td width="261">Spotsmeyer’s Furnishings</td><td width="140">25.72898470</td><td width="142">-80.23741968</td></tr><tr><td width="79">01445544</td><td width="261">Progressive Home Furnishings</td><td width="140">41.86673200</td><td width="142">-87.70100800</td></tr><tr><td width="79">01454545</td><td width="261">New Concepts Furniture</td><td width="140">33.77397700</td><td width="142">-84.38731800</td></tr><tr><td width="79">01905893</td><td width="261">Candoxy Canada Inc.</td><td width="140">48.38170029</td><td width="142">-89.24547985</td></tr><tr><td width="79">01905899</td><td width="261">Elkhorn Airport</td><td width="140">49.97715327</td><td width="142">-101.23461867</td></tr><tr><td width="79">01905902</td><td width="261">London Candoxy Storage Campus</td><td width="140">42.98689485</td><td width="142">-81.24621458</td></tr><tr><td width="79">10000</td><td width="261">The Cannon Group PLC</td><td width="140">52.47865520</td><td width="142">-1.90859489</td></tr><tr><td width="79">20000</td><td width="261">Selangorian Ltd.</td><td width="140">52.42190337</td><td width="142">-1.53778508</td></tr><tr><td width="79">20309920</td><td width="261">Metatorad Malaysia Sdn Bhd</td><td width="140">3.08329985</td><td width="142">101.64999984</td></tr><tr><td width="79">20312912</td><td width="261">Highlights Electronics Sdn Bhd</td><td width="140">3.15021023</td><td width="142">101.71284467</td></tr><tr><td width="79">20339921</td><td width="261">TraxTonic Sdn Bhd</td><td width="140">1.54907294</td><td width="142">110.34416981</td></tr><tr><td width="79">21233572</td><td width="261">Somadis</td><td width="140">34.01504517</td><td width="142">-6.83272026</td></tr><tr><td width="79">21245278</td><td width="261">Maronegoce</td><td width="140">33.60242486</td><td width="142">-7.61274353</td></tr><tr><td width="79">21252947</td><td width="261">ElectroMAROC</td><td width="140">33.91666643</td><td width="142">-6.91666670</td></tr><tr><td width="79">27090917</td><td width="261">Zanlan Corp.</td><td width="140">-26.35520540</td><td width="142">27.40158677</td></tr><tr><td width="79">27321782</td><td width="261">Karoo Supermarkets</td><td width="140">-29.11835074</td><td width="142">26.22492447</td></tr><tr><td width="79">27489991</td><td width="261">Durbandit Fruit Exporters</td><td width="140">-29.83637005</td><td width="142">30.94218850</td></tr><tr><td width="79">30000</td><td width="261">John Haddock Insurance Co.</td><td width="140">53.47962007</td><td width="142">-2.24880964</td></tr><tr><td width="79">31505050</td><td width="261">Woonboulevard Kuitenbrouwer</td><td width="140">52.14019530</td><td width="142">6.19148992</td></tr><tr><td width="79">31669966</td><td width="261">Meersen Meubelen</td><td width="140">51.98542312</td><td width="142">5.90462968</td></tr><tr><td width="79">31987987</td><td width="261">Candoxy Nederland BV</td><td width="140">52.37311967</td><td width="142">4.89319481</td></tr><tr><td width="79">32124578</td><td width="261">Nieuwe Zandpoort NV</td><td width="140">51.17786638</td><td width="142">4.83266278</td></tr><tr><td width="79">32656565</td><td width="261">Antarcticopy</td><td width="140">51.22171506</td><td width="142">4.39739518</td></tr><tr><td width="79">32789456</td><td width="261">Lovaina Contractors</td><td width="140">50.88170014</td><td width="142">4.71750006</td></tr><tr><td width="79">33000019</td><td width="261">Francematic</td><td width="140">48.81669022</td><td width="142">1.94925517</td></tr><tr><td width="79">33002984</td><td width="261">Parmentier Boutique</td><td width="140">48.85692470</td><td width="142">2.34120972</td></tr><tr><td width="79">33022842</td><td width="261">Livre Importants</td><td width="140">48.90484169</td><td width="142">2.81284869</td></tr><tr><td width="79">34010100</td><td width="261">Libros S.A.</td><td width="140">41.38566770</td><td width="142">2.16993861</td></tr><tr><td width="79">34010199</td><td width="261">Corporación Beta</td><td width="140">39.43432257</td><td width="142">-0.38737597</td></tr><tr><td width="79">34010602</td><td width="261">Helguera industrial</td><td width="140">40.41576270</td><td width="142">-3.70385108</td></tr><tr><td width="79">35122112</td><td width="261">Bilabankinn</td><td width="140">64.11111474</td><td width="142">-21.90939903</td></tr><tr><td width="79">35451236</td><td width="261">Gagn &amp;Gaman</td><td width="140">64.92900006</td><td width="142">-18.96200001</td></tr><tr><td width="79">35963852</td><td width="261">Heimilisprydi</td><td width="140">64.13533777</td><td width="142">-21.89521417</td></tr><tr><td width="79">38128456</td><td width="261">MEMA Ljubljana d.o.o.</td><td width="140">46.05124690</td><td width="142">14.50306222</td></tr><tr><td width="79">38546552</td><td width="261">EXPORTLES d.o.o.</td><td width="140">46.05124690</td><td width="142">14.50306222</td></tr><tr><td width="79">38632147</td><td width="261">Centromerkur d.o.o.</td><td width="140">46.55813813</td><td width="142">15.65098330</td></tr><tr><td width="79">40000</td><td width="261">Deerfield Graphics Company</td><td width="140">51.86390832</td><td width="142">-2.24978395</td></tr><tr><td width="79">41231215</td><td width="261">Sonnmatt Design</td><td width="140">47.42380030</td><td width="142">8.55140001</td></tr><tr><td width="79">41497647</td><td width="261">Pilatus AG</td><td width="140">47.05957649</td><td width="142">8.30785449</td></tr><tr><td width="79">41597832</td><td width="261">Möbel Scherrer AG</td><td width="140">47.69385177</td><td width="142">8.63503763</td></tr><tr><td width="79">42147258</td><td width="261">BYT-KOMPLET s.r.o.</td><td width="140">49.03722882</td><td width="142">17.81008579</td></tr><tr><td width="79">42258258</td><td width="261">J &amp;V v.o.s.</td><td width="140">48.98017220</td><td width="142">17.21433230</td></tr><tr><td width="79">42369147</td><td width="261">PLECHKONSTRUKT a.s.</td><td width="140">48.85557219</td><td width="142">16.05438888</td></tr><tr><td width="79">43687129</td><td width="261">Designstudio Gmunden</td><td width="140">47.91861087</td><td width="142">13.79814081</td></tr><tr><td width="79">43852147</td><td width="261">Michael Feit – Möbelhaus</td><td width="140">47.58900024</td><td width="142">14.13999975</td></tr><tr><td width="79">43871144</td><td width="261">Möbel Siegfried</td><td width="140">48.16780201</td><td width="142">16.35428691</td></tr><tr><td width="79">44171511</td><td width="261">Zuni Home Crafts Ltd.</td><td width="140">52.49931332</td><td width="142">-2.13111123</td></tr><tr><td width="79">44180220</td><td width="261">Afrifield Corporation</td><td width="140">51.27380021</td><td width="142">0.52508533</td></tr><tr><td width="79">44756404</td><td width="261">London Light Company</td><td width="140">52.20986970</td><td width="142">0.11156514</td></tr><tr><td width="79">45282828</td><td width="261">Candoxy Kontor A/S</td><td width="140">56.15704469</td><td width="142">10.20700961</td></tr><tr><td width="79">45282829</td><td width="261">Carl Anthony</td><td width="140">56.15704469</td><td width="142">10.20700961</td></tr><tr><td width="79">45779977</td><td width="261">Ravel Møbler</td><td width="140">55.31117991</td><td width="142">10.79238497</td></tr><tr><td width="79">45979797</td><td width="261">Lauritzen Kontormøbler A/S</td><td width="140">57.03462996</td><td width="142">9.92748722</td></tr><tr><td width="79">46251425</td><td width="261">Marsholm Karmstol</td><td width="140">56.67225949</td><td width="142">12.85753034</td></tr><tr><td width="79">46525241</td><td width="261">Konberg Tapet AB</td><td width="140">57.78593438</td><td width="142">14.22523925</td></tr><tr><td width="79">46897889</td><td width="261">Englunds Kontorsmöbler AB</td><td width="140">58.59301685</td><td width="142">16.17726378</td></tr><tr><td width="79">47523687</td><td width="261">Slubrevik Senger AS</td><td width="140">59.85794865</td><td width="142">10.47694914</td></tr><tr><td width="79">47563218</td><td width="261">Klubben</td><td width="140">59.91503946</td><td width="142">10.56067966</td></tr><tr><td width="79">47586954</td><td width="261">Sjøboden</td><td width="140">71.07307523</td><td width="142">24.70469333</td></tr><tr><td width="79">49525252</td><td width="261">Beef House</td><td width="140">51.21562980</td><td width="142">6.77605525</td></tr><tr><td width="79">49633663</td><td width="261">Autohaus Mielberg KG</td><td width="140">53.55334528</td><td width="142">9.99244496</td></tr><tr><td width="79">49858585</td><td width="261">Hotel Pferdesee</td><td width="140">50.04764177</td><td width="142">8.57851513</td></tr><tr><td width="79">50000</td><td width="261">Guildford Water Department</td><td width="140">51.23708010</td><td width="142">-0.57051592</td></tr><tr><td width="79">60000</td><td width="261">Blanemark Hifi Shop</td><td width="140">51.51777074</td><td width="142">-0.15552994</td></tr><tr><td width="79">61000</td><td width="261">Fairway Sound</td><td width="140">51.50632493</td><td width="142">-0.12714475</td></tr><tr><td width="79">62000</td><td width="261">The Device Shop</td><td width="140">51.50632493</td><td width="142">-0.12714475</td></tr><tr><td width="79">IC1020</td><td width="261">Cronus Cardoxy Sales</td><td width="140">56.10199973</td><td width="142">9.55599993</td></tr><tr><td width="79">IC1030</td><td width="261">Cronus Cardoxy Procurement</td><td width="140">53.55334528</td><td width="142">9.99244496</td></tr></tbody></table>

Note that some of the customers have the same latitude, longitude – this is due to the demo data.

### Next steps

Ok admitted – that was a long post – but hopefully it was helpful (and hopefully you can make it work)

As usual – you can download the [objects](http://www.freddy.dk/NavMaps1.zip) and the [Visual Studio solution](http://www.freddy.dk/NavMaps.zip) – and please do remember that this is NOT a solution that is installable and will work all over – you might have issues or problems and I would think the best place to post questions is mibuso (where my answer will get read by more people).

In the next post I will create a website, which connects to NAV Web Services to get customer location information – and in the third post I will show how to create an action in the customer List and Card to open up an area map centered around a specific customer.

So… – after step 2 you will see this

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/IntegrationtoVirtualEarthPart1outof3_8487/image_17.png)

Stay tuned!

The forth post in this series is a surprise – and will require NAV 2009 SP1 in order to work – so stay tuned for this one…

Enjoy and good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
