---
layout: post
title: "Connecting to NAV Web Services from Windows Mobile 6"
date: 2010-01-22 14:14:30
categories: ["Archive", "Web Services"]
tags: ["C#", "Compact", "NAV 2009 SP1", "Web Reference", "Web Services", "Windows Mobile 6"]
permalink: /2010/01/22/connecting-to-nav-web-services-from-windows-mobile-6/
---

I have created my very first Windows Mobile App!

![image\_2 (3)](/assets/images/2010/connecting-to-nav-web-services-from-windows-mobile-6/image_2-3.png)

This is running in an Emulator using the Professional SDK.

I also tried to deploy the solution to my physical device (my Smartphone), which also worked:

![Smartphone\_2](/assets/images/2010/connecting-to-nav-web-services-from-windows-mobile-6/smartphone_2.jpg)

To be honest, the biggest challenge is to setup everything so that you can get going.

# A couple of useful links to get going

Location to download Windows Mobile 6 SDK: [http://www.microsoft.com/downloads/details.aspx?FamilyID=06111a3a-a651-4745-88ef-3d48091a390b&DisplayLang=en](http://www.microsoft.com/downloads/details.aspx?FamilyID=06111a3a-a651-4745-88ef-3d48091a390b&DisplayLang=en "http://www.microsoft.com/downloads/details.aspx?FamilyID=06111a3a-a651-4745-88ef-3d48091a390b&DisplayLang=en")

The Windows Mobile 6 SDK is not included in Visual Studio 2008, you will have to download and install it.

Create your first WM6 App: [http://code.msdn.microsoft.com/WM6YourFirstApp](http://code.msdn.microsoft.com/WM6YourFirstApp "http://code.msdn.microsoft.com/WM6YourFirstApp")

A good video to help you get started.

Windows Mobile Development Center: [http://msdn.microsoft.com/en-us/windowsmobile/default.aspx](http://msdn.microsoft.com/en-us/windowsmobile/default.aspx "http://msdn.microsoft.com/en-us/windowsmobile/default.aspx")

This contains a lot of good links for getting started and how to do things.

Security Configuration Manager: **C:\\Program Files\\Windows Mobile 6 \\SDK\\Tools\\Security\\Security Powertoy**

When you have installed the SDK – go to this location and install the security configuration manager to be able to setup your device so that you can deploy your solution and debug it.

Note: I did struggle quite a bit to get network access up running on the device and on the emulator, but once I got the emulator setup to have network access (connected to Internet – not Work) and I had access through the firewall to my host machine – then everything worked fine.

# The scenario

Please read [this post](/2010/01/19/connecting-to-nav-web-services-from/) to get a brief explanation of the scenario I will implement on a Windows Mobile Device.

# .net 3.5

We will use 3.5 of the compact .net framework to build our application and whether you select Professional (first picture) or Standard (second picture) really doesn’t matter. First thing I do is to create two Web References from my app to the two Web Services i use in my scenario – SystemService (SystemServiceRef) and Customer Page (CustomerPageRef).

These Web References are pretty similar to .net 2.0 Web References from the normal .net framework (look [this post](/2010/01/19/connecting-to-nav-web-services-from-c-using-web-reference/)). One thing to note is, that you do not have UseDefaultCredentials in the compact framework so you need to specify user and password when connecting to NAV Web Services.

The project type is a Device Application and the code on the form is:

```
using System; 
using System.Windows.Forms; 
using SmartDeviceProject4.SystemServiceRef; 
using SmartDeviceProject4.CustomerPageRef; 
using System.Net;

namespace SmartDeviceProject4 
{ 
    public partial class Form1 : Form 
    { 
        public Form1() 
        { 
            InitializeComponent();

            string baseURL = ":7047/DynamicsNAV/WS/";'>http://<IP>:7047/DynamicsNAV/WS/"; 
            NetworkCredential credentials = new NetworkCredential(user, password, domain);

            SystemService systemService = new SystemService(); 
            systemService.Credentials = credentials; 
            systemService.Url = baseURL + "SystemService"; 
            systemService.PreAuthenticate = true;

            display("Companies:"); 
            string[] companies = systemService.Companies(); 
            foreach (string company in companies) 
                display(company); 
            string cur = companies[0];

            string customerPageURL = baseURL + Uri.EscapeDataString(cur) + "/Page/Customer"; 
            display(""); 
            display("URL of Customer Page:"); 
            display(customerPageURL);

            Customer_Service customerService = new Customer_Service(); 
            customerService.Credentials = credentials; 
            customerService.Url = customerPageURL; 
            customerService.PreAuthenticate = true;

            Customer customer10000 = customerService.Read("10000"); 
            display(""); 
            display("Name of customer 10000:"); 
            display(customer10000.Name);

            Customer_Filter filter1 = new Customer_Filter(); 
            filter1.Field = Customer_Fields.Country_Region_Code; 
            filter1.Criteria = "GB";

            Customer_Filter filter2 = new Customer_Filter(); 
            filter2.Field = Customer_Fields.Location_Code; 
            filter2.Criteria = "RED|BLUE";

            display(""); 
            display("Customers in GB served by RED or BLUE warehouse:"); 
            Customer_Filter[] filters = new Customer_Filter[] { filter1, filter2 }; 
            Customer[] customers = customerService.ReadMultiple(filters, null, 0); 
            foreach (Customer customer in customers) 
                display(customer.Name);

            display(""); 
            display("THE END"); 
        }

        private void display(string s) 
        { 
            this.textBox1.Text += s + "\r\n"; 
        } 
    } 
}
```

As you can see 99% of the code is similar to the post about C# and Web References (found [here](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-web-reference.aspx)). Major differences are that the baseURL of course isn’t localhost (since localhost would be the mobile device itself) and I have to setup credentials in the beginning.

# But… It is Very Slow!

Having done this and finally have everything working, you will probably find that Web Services from a mobile device is extremely slow.

Call a service to get Customer 10000 takes approx. 1.5 second – and it is not getting faster if you do it 100 times.

If you set service.preAuthenticate to true – then the time is down to 1.2 second, but still – slower than I would like.

I tried to create a standard .net Web Service on my host computer (asmx web service – just the Hello World sample) and tried to call this method 100 times and in this case, the time was down to around 0.5 second pr. call – still very slow, but more acceptable.

When running some of the other applications a call to a webservice (including authorization) is only around 0.04 seconds on my computer so we are looking at around 30 times slower pr. web service call from a mobile device.

I also tried to make my Hello World application return a 10k string – this didn’t affect the performance at all – and when upping the size of the string to 40k – the time climbed to around 0.7 second pr. call – it seems like the biggest problem is latency (only guessing).

I will do some more investigation on this – including contacting the mobile team in Microsoft to figure out why and how to fix this (if possible).

For now the solution seems to be to create some proxy (with a very limited instruction set = one method for each high level thing the mobile device is capable of doing) running with no authentication and then have the mobile devices communicate with that – maybe using some kind of poor mans authentication – or simply having IP security on the Web Service.

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
