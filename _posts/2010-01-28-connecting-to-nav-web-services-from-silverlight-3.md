---
layout: post
title: "Connecting to NAV Web Services from Silverlight 3"
date: 2010-01-28 13:10:21
categories: ["Archive", "Web Services"]
tags: ["Asynchronous", "C#", "Flash", "NAV Policy Server", "Service Reference", "Silverlight", "Web Services"]
permalink: /2010/01/28/connecting-to-nav-web-services-from-silverlight-3/
---

Please read [this post](/2010/01/19/connecting-to-nav-web-services-from/) to get a brief explanation of the scenario I will implement in Silverlight. Yes, yes – I know it isn’t a fancy graphical whatever as Silverlight should be, but to be honest – I would rather do something crappy on purpose than trying to do something fancy and everybody would find it crappy anyway:-)

# Getting started with Silverlight

[http://silverlight.net/getstarted](http://silverlight.net/getstarted) – is your friend. Go to the web site and click this button:

[![image\_5](/assets/images/2010/connecting-to-nav-web-services-from-silverlight-3/image_5.png)](http://www.microsoft.com/web/gallery/install.aspx?appsxml=&appid=VWD;Silverlight3Tools;SilverlightToolkit;RIAServices)

Or click the image above directly.

Within a few seconds you will find yourself installing all the free tools you need to start developing Silverlight applications.

On the getstarted web site you will also find videos and walkthroughs on how to develop in Silverlight.

# Silverlight is .net and c# so really guys… – how hard can it be?

That was what I thought!

So I just downloaded the Silverlight development platform and started coding and as soon as I tried to connect to NAV Web Services I ran into the showstopper:

![image\_2 (1)](/assets/images/2010/connecting-to-nav-web-services-from-silverlight-3/image_2-1.png)

Meaning that for a Silverlight application to be able to communicate with NAV Web Services – it needs to be deployed in the same location as NAV Web Services – [http://localhost:7047](http://localhost:7047) – that doesn’t really sound like a good idea.

On MSDN i found this article explaining about this in detail: [http://msdn.microsoft.com/en-us/library/cc645032(VS.95).aspx](http://msdn.microsoft.com/en-us/library/cc645032\(VS.95\).aspx "http://msdn.microsoft.com/en-us/library/cc645032(VS.95).aspx")

Silverlight needs permission by the Web Services host to access the Web Service – it kind of seems like overkill due to the fact that our web services are authenticated with Windows Authentication but I guess there are other services where this makes sense.

To make a long story short – if connecting to _**[http://localhost:7047/DynamicsNAV/WS/SystemService](http://localhost:7047/DynamicsNAV/WS/SystemService)**_ – then Silverlight will first try to download _**[http://localhost:7047/clientaccesspolicy.xml](http://localhost:7047/clientaccesspolicy.xml)**_ and check whether this is OK, but as you can imagine – NAV doesn’t do that:-(

# clientaccesspolicy.xml

So if NAV doesn’t support that – how do we get around this obstacle? (of course you know that there is a way – else you wouldn’t be reading this and I wouldn’t be writing it)

The trick is just to create a small windows service that does nothing but host this file. We are lucky that the endpoint of NAV Web Services is **_[http://localhost:7047/DynamicsNAV](http://localhost:7047/DynamicsNAV)_** – and everything underneath that – so I should be able to create a WCF Service hosting just the xml file on _**[http://localhost:7047](http://localhost:7047)**_

# NAV Policy Server

I have created a small project called the NAV Policy Server. It is a Windows Service, hosting a WCF Service that will service a “allow all” version of clientaccesspolicy.xml, making Silverlight 3 able to connect to NAV Web Services.

You can read [here](http://msdn.microsoft.com/en-us/library/9k985bc9\(VS.80\).aspx) about how to create a Windows Service (including how to create Setup functionality in the Service). The main program of the Windows Service is here:

```
using System; 
using System.ComponentModel; 
using System.ServiceProcess; 
using System.ServiceModel; 
using System.ServiceModel.Description; 
using System.Xml; 
using System.Reflection; 
using System.IO;

namespace NAVPolicyServer 
{ 
    public partial class NAVPolicyService : ServiceBase 
    { 
        ServiceHost host;

        public NAVPolicyService() 
        { 
            InitializeComponent();

            string WebServicePort = "7047"; 
            bool WebServiceSSLEnabled = false;

            // Read configuration file 
            XmlDocument doc = new XmlDocument(); 
            doc.Load(Path.Combine(Path.GetDirectoryName(Assembly.GetExecutingAssembly().CodeBase), "CustomSettings.config")); 
            XmlNode webServicePortNode = doc.SelectSingleNode("/appSettings/add[@key='WebServicePort']"); 
            WebServicePort = webServicePortNode.Attributes["value"].Value; 
            XmlNode webServiceSSLEnabledNode = doc.SelectSingleNode("/appSettings/add[@key='WebServiceSSLEnabled']"); 
            WebServiceSSLEnabled = webServiceSSLEnabledNode.Attributes["value"].Value.Equals("true", StringComparison.InvariantCultureIgnoreCase);

            // Base listening address 
            string BaseURL = (WebServiceSSLEnabled ? Uri.UriSchemeHttps : Uri.UriSchemeHttp) + Uri.SchemeDelimiter + System.Environment.MachineName + ":" + WebServicePort;

            // Initialize host 
            this.host = new ServiceHost(new PolicyRetriever(), new Uri(BaseURL)); 
            this.host.AddServiceEndpoint(typeof(IPolicyRetriever), new WebHttpBinding(false ? WebHttpSecurityMode.Transport : WebHttpSecurityMode.None), "").Behaviors.Add(new WebHttpBehavior()); 
        }

        protected override void OnStart(string[] args) 
        { 
            if (host.State != CommunicationState.Opened && host.State != CommunicationState.Opening) 
            { 
                host.Open(); 
            } 
        }

        protected override void OnStop() 
        { 
            if (host.State != CommunicationState.Closed && host.State != CommunicationState.Closing) 
            { 
                host.Close(); 
            } 
        } 
    } 
}
```

As you can see, the Service needs to be installed in the Service Tier directory of the Web Service listener you want to enable for Silverlight as it reads the CustomSettings.config file to find the port number and whether or not it uses SSL.

After this it creates a ServiceHost bases on the _PolicyRetriever_ class with a WebHttpBinding endpoint at the base URL, here _**[http://machine:7047](http://machine:7047)**_. In the endpoint you specify the interface (_IPolicyRetriever_) this endpoint services and this interface is implemented by the _PolicyRetriever_ class.

The actual code is something I found on Carlos’ blog – [http://blogs.msdn.com/carlosfigueira/archive/2008/03/07/enabling-cross-domain-calls-for-silverlight-apps-on-self-hosted-web-services.aspx](http://blogs.msdn.com/carlosfigueira/archive/2008/03/07/enabling-cross-domain-calls-for-silverlight-apps-on-self-hosted-web-services.aspx "http://blogs.msdn.com/carlosfigueira/archive/2008/03/07/enabling-cross-domain-calls-for-silverlight-apps-on-self-hosted-web-services.aspx")

The _IPolicyRetriever_ interface is the contract and it looks like:

```
[ServiceContract] 
public interface IPolicyRetriever 
{ 
    [OperationContract, WebGet(UriTemplate = "/clientaccesspolicy.xml")] 
    Stream GetSilverlightPolicy(); 
    [OperationContract, WebGet(UriTemplate = "/crossdomain.xml")] 
    Stream GetFlashPolicy(); 
}
```

As you can see we host two files – _clientaccesspolicy.xml_ for Silverlight and _crossdomain.xml_ for flash.

The PolicyRetriever class (the Service) itself is implemented as a singleton and looks like:

```
[ServiceBehavior(InstanceContextMode = InstanceContextMode.Single)] 
public class PolicyRetriever : IPolicyRetriever 
{ 
    public PolicyRetriever() 
    { 
    }

    /// <summary> 
    /// Create a UTF-8 encoded Stream based on a string 
    /// </summary> 
    /// <param name="result"></param> 
    /// <returns></returns> 
    private Stream StringToStream(string result) 
    { 
        WebOperationContext.Current.OutgoingResponse.ContentType = "application/xml"; 
        return new MemoryStream(Encoding.UTF8.GetBytes(result)); 
    }

    /// <summary> 
    /// Fetch policy file for Silverlight access 
    /// </summary> 
    /// <returns>Silverlight policy access xml</returns> 
    public Stream GetSilverlightPolicy() 
    { 
        string result = @"<?xml version=""1.0"" encoding=""utf-8""?> 
<access-policy> 
    <cross-domain-access> 
        <policy> 
            <allow-from http-request-headers=""*""> 
                <domain uri=""*""/> 
            </allow-from> 
            <grant-to> 
                <resource path=""/"" include-subpaths=""true""/> 
            </grant-to> 
        </policy> 
    </cross-domain-access> 
</access-policy>"; 
        return StringToStream(result); 
    }

    /// <summary> 
    /// Fetch policy file for Flash access 
    /// </summary> 
    /// <returns>Flash policy access xml</returns> 
    public Stream GetFlashPolicy() 
    { 
        string result = @"<?xml version=""1.0""?> 
<!DOCTYPE cross-domain-policy SYSTEM ""http://www.macromedia.com/xml/dtds/cross-domain-policy.dtd""> 
<cross-domain-policy> 
    <allow-access-from domain=""*"" /> 
</cross-domain-policy>"; 
        return StringToStream(result); 
    } 
}
```

The way you make a WCF service a singleton is by specifying an instance of the class to the ServiceHost and set InstanceContextMode to single in the ServiceBehavior Attribute.

That is actually all it takes, installing and starting this service will overcome the connection issue.

The NAVPolicyServer solution can be downloaded [here](http://www.freddy.dk/NAVPolicyServer.zip) and the compiled .msi (installable) can be downloaded [here](http://www.freddy.dk/NAVPolicyServer.msi.zip).

# Now… – Connecting to NAV Web Services from Silverlight

Having overcome the connection issue – it is really just to write our Silverlight application.

Create a Silverlight application, insert a StackPanel and a ListBox named output in the .xaml file, add service references and write code.

You will quickly notice, that there is nothing called Add Web Reference – only Add Service Reference – and when you have done do, you will notice that all the functions that you normally invoke are missing…

This is because Silverlight only supports Asynchronous Service access – so much for just creating my standard flow of my app.

Another thing that has changed significantly is what you need to do in order to make a Service Reference work. If you look at my earlier posts with [C# and Service References](/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-code-version/), you can see that I need to setup the binding manually and add endpoints etc. Even if I wanted to do it in a config file (like [here](/2010/01/19/connecting-to-nav-web-services-from-c-using-service-reference-config-file-version/)), you needed to make a lot of changes to the config file (adding behaviors etc.)

In Silverlight you just add the Service Reference and start partying like:

```
SystemService_PortClient systemService = new SystemService_PortClient(); 
systemService.CompaniesAsync();
```

works right away, no changes needed – THAT’s nice. In my sample I do however build the URL up dynamically, meaning that my construction of the systemService looks like:

```
SystemService_PortClient systemService = new SystemService_PortClient("SystemService_Port", new EndpointAddress(baseURL + "SystemService"));
```

Which basically just tells it to read the configuration section and overwrite the endpoint address – still pretty simple.

# Async

Whenever you call CompaniesAsync – it returns immediately and after a while the event connected to CompaniesCompleted is triggered. The way I like to do this is to do a inline delegate as an event trigger and just specify my code right there.

My scenario should first list the companies, calculate a customer page URL, read customer 10000 and then read customers with location code BLUE or RED in GB.

```
public partial class MainPage : UserControl 
{ 
    private string baseURL = "http://localhost:7047/DynamicsNAV/WS/"; 
    private string customerPageURL;

    public MainPage() 
    { 
        InitializeComponent();

        SystemService_PortClient systemService = new SystemService_PortClient("SystemService_Port", new EndpointAddress(baseURL + "SystemService")); 
        systemService.CompaniesCompleted += delegate(object sender, CompaniesCompletedEventArgs e) 
        { 
            display("Companies:"); 
            for (int i = 0; i < e.Result.Length; i++) 
                display(e.Result[i]); 
            string cur = e.Result[0];

            this.customerPageURL = baseURL + Uri.EscapeDataString(cur) + "/Page/Customer"; 
            display(" "); 
            display("URL of Customer Page:"); 
            display(customerPageURL);

            FindCustomer10000(); 
        };
        systemService.CompaniesAsync(); 
    }

    void display(string s) 
    { 
        this.output.Items.Add(s); 
    }
}
```

As you can see, I do not call the _FindCustomer10000_ before I am done with step 1.

I could have inserted that call after the call to CompaniesAsync – but then the customerPageURL variable would not be initialized when starting to connect to the customer page.

_FindCustomer10000_ looks like:

```
private void FindCustomer10000() 
{ 
    Customer_PortClient readCustomerService = new Customer_PortClient("Customer_Port", new EndpointAddress(customerPageURL)); 
    readCustomerService.ReadCompleted += delegate(object sender, ReadCompletedEventArgs e) 
    { 
        display(" "); 
        display("Name of Customer 10000: " + e.Result.Name);

        FindCustomers(); 
    };
    readCustomerService.ReadAsync("10000"); 
}
```

Again – when we have data and we are done – call _FindCustomers_, which looks like:

```
private void FindCustomers() 
{ 
    Customer_PortClient readMultipleCustomerService = new Customer_PortClient("Customer_Port", new EndpointAddress(customerPageURL)); 
    readMultipleCustomerService.ReadMultipleCompleted += delegate(object sender, ReadMultipleCompletedEventArgs e) 
    { 
        display(" "); 
        display("Customers in GB served by RED or BLUE warehouse:"); 
        foreach (Customer customer in e.Result) 
            display(customer.Name);
        display(" "); 
        display("THE END");
    }; 
    Customer_Filter filter1 = new Customer_Filter(); 
    filter1.Field = Customer_Fields.Country_Region_Code; 
    filter1.Criteria = "GB"; 
    Customer_Filter filter2 = new Customer_Filter(); 
    filter2.Field = Customer_Fields.Location_Code; 
    filter2.Criteria = "RED|BLUE"; 
    Customer_Filter[] filters = new Customer_Filter[] { filter1, filter2 }; 
    readMultipleCustomerService.ReadMultipleAsync(filters, null, 0); 
}
```

If you try to move the call to _FindCustomers_ up after the call to _FindCustomer10000_ then you will see that it isn’t always determined which of the two methods complete first, meaning that the order of things in the listbox will be “random”.

As you can see, the NAVPolicyServer is really the thing that makes this easy and possible – I will send a mail to my colleague who is the Program Manager for Web Services and ask him to include a way of serving policies from NAV automatically – until then, you will need the policy server (which is free and available right [here](http://www.freddy.dk/NAVPolicyServer.msi.zip)).

Running the Silverlight application will perform the following output:

![image\_6](/assets/images/2010/connecting-to-nav-web-services-from-silverlight-3/image_6.png)

BTW – the Silverlight application can be downloaded [here](http://www.freddy.dk/SilverlightApplication2.zip).

Hopefully this can be used to create some cool visual Silverlight applications:-)

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
