---
layout: post
title: "Connecting to NAV Web Services from C# using Service Reference (code version)"
date: 2010-01-20 00:22:59
categories: ["Archive", "Web Services"]
tags: ["C#", "NAV 2009 SP1", "Service Reference", "Web Services"]
permalink: /2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-code-version/
---

You should read the post about [connecting to NAV Web Services from C# using Service Reference (config file version)](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-config-file-version.aspx) before continuing here.

### Code is king

As you saw in the other post, the config file was pretty complicated and although it is editable by hand and as such could be modified at installtime or whatever, I actually prefer to capture a number of these settings in code and only have very specific values in a config file (like f.ex. the base URL).

In NAV you would never have the full Service URL on all services in a config file. This would mean that you would have to change company in a number of locations in a config file – that just doesn’t fly.

If we have a look at the config file once more, you will see that there is a Binding configuration, specifying a BasicHttpBinding with a couple of settings. If we want to create this binding in code, it would look like:

// Create a NAV comatible binding  
BasicHttpBinding navWSBinding = new BasicHttpBinding();  
navWSBinding.Security.Mode = BasicHttpSecurityMode.TransportCredentialOnly;  
navWSBinding.Security.Transport.ClientCredentialType = HttpClientCredentialType.Windows;

Having this binding class, we can now create a systemService Service Client simply by:

SystemService\_PortClient systemService = new SystemService\_PortClient(navWSBinding, new EndpointAddress(baseURL + “SystemService”));

Specifying the endpoint address to the constructor.

The only other thing we need to specify is the endpoint behavior to allow delegation. This is done in code by:

systemService.ClientCredentials.Windows.AllowedImpersonationLevel =  
System.Security.Principal.TokenImpersonationLevel.Delegation;  
systemService.ClientCredentials.Windows.AllowNtlm = true;

Using code like this actually makes the app.config obsolete and you can delete the config file totally when running the below application.

### The entire application

The following is a listing of the full console application using code to set all properties and no app.config is necessary (nor used at all):

using System;  
using System.Net;  
using testAppWCF.SystemServiceRef;  
using testAppWCF.CustomerPageRef;  
using System.ServiceModel;

namespace testAppWCF  
{  
class Program  
{  
static void Main(string\[\] args)  
{  
string baseURL = “[http://localhost:7047/DynamicsNAV/WS/”;](http://localhost:7047/DynamicsNAV/WS/";)

            // Create a NAV compatible binding  
BasicHttpBinding navWSBinding = new BasicHttpBinding();  
navWSBinding.Security.Mode = BasicHttpSecurityMode.TransportCredentialOnly;  
navWSBinding.Security.Transport.ClientCredentialType = HttpClientCredentialType.Windows;

            // Create the SystemService Client  
SystemService\_PortClient systemService = new SystemService\_PortClient(navWSBinding, new EndpointAddress(baseURL + “SystemService”));  
systemService.ClientCredentials.Windows.AllowedImpersonationLevel = System.Security.Principal.TokenImpersonationLevel.Delegation;  
systemService.ClientCredentials.Windows.AllowNtlm = true;

            Console.WriteLine(“Companies:”);  
string\[\] companies = systemService.Companies();  
foreach(string company in companies)  
Console.WriteLine(company);  
string cur = companies\[0\];

            string customerPageURL = baseURL + Uri.EscapeDataString(cur) + “/Page/Customer”;  
Console.WriteLine(“nURL of Customer Page: “+customerPageURL);

            // Create the Customer Page Service Client  
Customer\_PortClient customerService = new Customer\_PortClient(navWSBinding, new EndpointAddress(customerPageURL));  
customerService.ClientCredentials.Windows.AllowedImpersonationLevel = System.Security.Principal.TokenImpersonationLevel.Delegation;  
customerService.ClientCredentials.Windows.AllowNtlm = true;

            Customer customer10000 = customerService.Read(“10000”);  
Console.WriteLine(“nName of Customer 10000: “+customer10000.Name);

            Customer\_Filter filter1 = new Customer\_Filter();  
filter1.Field = Customer\_Fields.Country\_Region\_Code;  
filter1.Criteria = “GB”;

            Customer\_Filter filter2 = new Customer\_Filter();  
filter2.Field = Customer\_Fields.Location\_Code;  
filter2.Criteria = “RED|BLUE”;

            Console.WriteLine(“nCustomers in GB served by RED or BLUE warehouse:”);  
Customer\_Filter\[\] filters = new Customer\_Filter\[\] { filter1, filter2 };  
Customer\[\] customers = customerService.ReadMultiple(filters, null, 0);  
foreach (Customer customer in customers)  
Console.WriteLine(customer.Name);

            Console.WriteLine(“nTHE END”);  
Console.ReadLine();  
}  
}  
}

If you need to specify a different username / password it is done in the same way as it was done for Service References using config files.

This application will output exactly the same as the application using Service References and a config file, in the end the config file is just a number of settings which will be used to instantiate a number of classes from – giving the same result.

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
