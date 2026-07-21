---
layout: post
title: "Connecting to NAV Web Services from C# using Service Reference (config file version)"
date: 2010-01-19 23:53:33
categories: ["Archive", "Web Services"]
tags: ["C#", "NAV 2009 SP1", "Service Reference", "Web Services"]
permalink: /2010/01/19/connecting-to-nav-web-services-from-c-using-service-reference-config-file-version/
---

##### Prerequisites

Please read [this post](http://blogs.msdn.com/freddyk/archive/2010/01/19/connecting-to-nav-web-services-from.aspx) to get a brief explanation of the scenario I will implement in C# using Web References.

For C# we can leave the Service Tier running Negotiate or we can use Ntlm as PHP and Java. In this example I will assume that the Service Tier is running SPNEGO (which is the default)

BTW. Basic knowledge about C# is required to understand the following post:-)

##### Version and download

I am using Visual Studio 2008 professional with SP1 when writing this sample, to be honest I have NOT tried to see whether this will work in the Express versions of Visual Studio, but they do have Service Reference and Web Reference so I cannot see why not.

### What is the difference between a Web Reference and a Service Reference?

In short, the Web Reference is a .net 2.0 compatible Web Service reference, the Service Reference is a .net 3.5 WCF based Service Reference.

**Add Web Reference** is a wrapper over [wsdl.exe](http://msdn.microsoft.com/en-us/library/7h3ystb6\(VS.80\).aspx) and can be used to create proxies for .NET 1.1 or 2.0 clients. Of course this means when you are pointing to a WCF service you have to be pointing to an endpoint that uses basicHttpBinding.

**Add Service Reference** is a wrapper over [svcutil.exe](http://msdn.microsoft.com/en-us/library/aa347733.aspx) and also creates clients proxies. These proxies, however, can only be consumed by .NET 3.5 clients.

In this post I will describe how to use Service References, where all settings are stored in the .config file.

### How to add a Service Reference

Select **Add Service Reference**

[![image\_thumb\[13\]](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image_thumb%5B13%5D_5006ce0b-67d9-4af0-a923-0c8b1d5feab6.png "image_thumb[13]")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image%5B27%5D.png)

Type the URL and namespace for the SystemService Service Reference:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image_2.png)

and for the Customer Page Service Reference:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image_4.png)

### The .config file

After having added these Service References, all the properties and settings about the references are stored in app.config (which gets autocreated by Visual Studio). The Proxy classes are generated and everything seems fine until you start using it.

You have to change a couple of things in the app.config file before using these.

Under every binding configuration setting you will find a section like this:

```
<security mode="None">
<transport clientCredentialType="None" proxyCredentialType="None"
realm="">
<extendedProtectionPolicy policyEnforcement="Never" />
</transport>
<message clientCredentialType="UserName" algorithmSuite="Default" />
</security>
```

this does not match whats needed for NAV Web Services. NAV Web Services absolutely do not run without security. You will have to change this section with:

```
<security mode="TransportCredentialOnly">
<transport clientCredentialType="Windows"  />
</security>
```

which matches the security mode and transport of the binding used by NAV when using Windows Authentication (SPNEGO). If the Service Tier is setup to run Ntlm – the ClientCredentialType needs to be “Ntlm” in the config file.

Furthermore you will have to add a behavior indicating that the Web Service Listener is allowed to use Delegation on your credentials (between </bindings> and <client>:

```
<behaviors>
<endpointBehaviors>
<behavior name="allowDelegation">
<clientCredentials>
<windows allowedImpersonationLevel="Delegation"
allowNtlm="true"/>
</clientCredentials>
</behavior>
</endpointBehaviors>
</behaviors>
```

and last, but not least you will have to add this behavior to all endpoints like:

`<endpoint address="`[`http://localhost:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd/Page/Customer"`](http://localhost:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd/Page/Customer")  
          

```
binding="basicHttpBinding"
bindingConfiguration="Customer_Binding"
behaviorConfiguration="allowDelegation"
contract="CustomerPageRef.Customer_Port"
name="Customer_Port" />
```

If we strip away all the unnecessary defaults and modify a couple of things by hand, the **ENTIRE** config file could look like this:

```
<?xml version="1.0″ encoding="utf-8″ ?>
<configuration>
<system.serviceModel>
<bindings>
<basicHttpBinding>
<binding name="NavWSBinding">
<security mode="TransportCredentialOnly">
<transport clientCredentialType="Windows"  />
</security>
</binding>
</basicHttpBinding>
</bindings>
<behaviors>
<endpointBehaviors>
<behavior name="allowDelegation">
<clientCredentials>
<windows allowedImpersonationLevel="Delegation"
allowNtlm="true"/>
</clientCredentials>
</behavior>
</endpointBehaviors>
</behaviors>
<client>
<endpoint address="
```

[`http://localhost:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd/Page/Customer"`](http://localhost:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd/Page/Customer")  
                

```
binding="basicHttpBinding"
bindingConfiguration="NavWSBinding"
behaviorConfiguration="allowDelegation"
contract="CustomerPageRef.Customer_Port"
name="Customer_Port" />
<endpoint address="
```

[`http://localhost:7047/DynamicsNAV/WS/SystemService"`](http://localhost:7047/DynamicsNAV/WS/SystemService")  
                

```
binding="basicHttpBinding"
bindingConfiguration="NavWSBinding"
behaviorConfiguration="allowDelegation"
contract="SystemServiceRef.SystemService_Port"
name="SystemService_Port" />
</client>
</system.serviceModel>
</configuration>
```

Confused?

### The code

First a couple of using statements (including the two reference namespaces) and the main body of a console app:

```
using System;
using System.Net;
using testAppWCF2.SystemServiceRef;
using testAppWCF2.CustomerPageRef;
```

```
namespace testAppWCF2
{
class Program
{
static void Main(string[] args)
{
// main program code
}
}
}
```

The main code follows.

First, connect to the System Web Service and list all companies:

`string baseURL = "`[`http://localhost:7047/DynamicsNAV/WS/";`](http://localhost:7047/DynamicsNAV/WS/";)

```
// Create the SystemService Client
SystemService_PortClient systemService = new SystemService_PortClient("SystemService_Port", baseURL + "SystemService");
```

```
Console.WriteLine("Companies:");
string[] companies = systemService.Companies();
foreach (string company in companies)
Console.WriteLine(company);
string cur = companies[0];
```

Note, that when creating a System Service Client, I specify the name of an endpoint configuration and a URL. I didn’t have to specify anything, then all defaults would be taken from the Config file, but I like to show how you can calculate the URL and specify that at runtime.

Now I have the company I want to use in _cur_ and the way I create a URL to the Customer page is by doing:

```
string customerPageURL = baseURL + Uri.EscapeDataString(cur) + "/Page/Customer";
Console.WriteLine("nURL of Customer Page: " + customerPageURL);
```

and then I can create a Service Class to the Customer Page by specifying the config section and a URL again:

```
// Create the SystemService Client
Customer_PortClient customerService = new Customer_PortClient("Customer_Port", customerPageURL);
```

and using this, I read customer 10000 and output the name:

```
Customer customer10000 = customerService.Read("10000");
Console.WriteLine("nName of Customer 10000: " + customer10000.Name);
```

Last, but not least – lets create a filter and read all customers in GB that has Location Code set to RED or BLUE:

```
Customer_Filter filter1 = new Customer_Filter();
filter1.Field = Customer_Fields.Country_Region_Code;
filter1.Criteria = "GB";
```

```
Customer_Filter filter2 = new Customer_Filter();
filter2.Field = Customer_Fields.Location_Code;
filter2.Criteria = "RED|BLUE";
```

```
Console.WriteLine("nCustomers in GB served by RED or BLUE warehouse:");
Customer_Filter[] filters = new Customer_Filter[] { filter1, filter2 };
Customer[] customers = customerService.ReadMultiple(filters, null, 0);
foreach (Customer customer in customers)
Console.WriteLine(customer.Name);
```

```
Console.WriteLine("nTHE END");
Console.ReadLine();
```

As you can see the code is actually as simple as the Web Reference version, but the config file complicates things a lot. All of the above will output the following to a console prompt (on my machine running NAV 2009SP1 W1)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingServ_13F7D/image_6.png)

### Authentication

BTW – this sample will by default run Windows Authentication. If you want to specify a different user you will need to set the ClientCredential property like this:

`customerService.ClientCredentials.Windows.ClientCredential = new NetworkCredential("user", "password", "domain");`

You would need to set this on each Service Client you create.

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
