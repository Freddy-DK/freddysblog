---
layout: post
title: "Connecting to NAV Web Services from C# using Web Reference"
date: 2010-01-19 22:24:07
categories: ["Archive", "Web Services"]
tags: ["C#", "NAV 2009 SP1", "Web Reference", "Web Services"]
permalink: /2010/01/19/connecting-to-nav-web-services-from-c-using-web-reference/
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

### How to add Web References

Select **Add Service Reference**

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_2.png)

Click **Advanced**

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_6.png)

Click **Add Web Reference**

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_8.png)

Type in the **URL** and specify the **namespace** for the SystemService Web Reference

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_thumb_5.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_12.png)

and for the Customer Page Web Reference

![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_thumb_4.png "image")

When adding a Web Reference, Visual Studio will create a config file in which it stores stuff like the URL for the Reference. In my samples I will set the URL in code and due to this, the config file is not needed.

### Authentication

In the following sample I use Windows Authentication. In Web References you just need to set the property _UseDefaultCredentials_ in the service class to _true_, then .net will automatically try to use your windows credentials to connect to the Web Service.

If you want to connect to a Web Reference using a specific username/password you need to exchange this line:

`someService.UseDefaultCredentials = true;`

with this:

`someService.Credentials = new NetworkCredential("user", "password", "domain");`

### The code

`First a couple of using statements (including the two reference namespaces) and the main body of a console app:`

```
using System;
using System.Net;
using testApp.CustomerPageRef;
using testApp.SystemServiceRef;
```

```
namespace testApp
{
class Program
{
static void Main(string[] args)
{
// main program code
```

        

```
}
```

    

```
}
```

`}`

`The main code follows`

First, connect to the System Web Service and list all companies:

`string baseURL = "`[`http://localhost:7047/DynamicsNAV/WS/";`](http://localhost:7047/DynamicsNAV/WS/";)

```
SystemService systemService = new SystemService();
systemService.Url = baseURL + "SystemService";
systemService.UseDefaultCredentials = true;
```

```
Console.WriteLine("Companies:");
string[] companies = systemService.Companies();
foreach (string company in companies)
Console.WriteLine(company);
string cur = companies[0];
```

Now I have the company I want to use in _cur_ and the way I create a URL to the Customer page is by doing:

```
string customerPageURL = baseURL + Uri.EscapeDataString(cur) + "/Page/Customer";
Console.WriteLine("nURL of Customer Page: " + customerPageURL);
```

and then I can create a Service Class to the Customer Page:

```
Customer_Service customerService = new Customer_Service();
customerService.Url = customerPageURL;
customerService.UseDefaultCredentials = true;
```

and using this, I read customer 10000 and output the name:

```
Customer cust10000 = customerService.Read("10000");
Console.WriteLine("nName of Customer 10000: " + cust10000.Name);
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

All of the above will output the following to a console prompt (on my machine running NAV 2009SP1 W1)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_thumb_6.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromCusingWebR_13860/image_14.png)

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
