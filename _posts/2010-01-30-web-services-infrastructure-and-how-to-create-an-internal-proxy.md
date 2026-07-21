---
layout: post
title: "Web Services Infrastructure and how to Create an Internal Proxy"
date: 2010-01-30 12:20:40
categories: ["Archive", "Web Services"]
tags: ["C#", "NAV 2009 SP1", "Proxy", "Service", "Service Reference", "WCF", "Web Reference", "Web Services", "Windows Service"]
permalink: /2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/
---

I am NOT an expert in how to setup a secure network and I do NOT know a lot about firewalls, DMZ setup and all of these things, but I have seen a lot in my 25 years of working with computers and the following (absolutely non-exhaustive) gives a good picture of a common network situation of companies, who wants to interact with customers and partners through Web Applications and/or Web Services.

![image\_4](/assets/images/2010/web-services-infrastructure-and-how-to-create-an-internal-proxy/image_4.png)

DMZ can be at the customer site or with a hosting center, and yes – there probably is a firewalls between DMZ and Internet.

As you can imagine, this post is NOT about how to setup firewalls and other network infrastructure elements – it is about the software. Let me explain the different machines in this setup (note that there are many different solutions – this is just one possible setup).

1.  This machine is running a Sharepoint server with a time/entry application, document repository linked together with items in the NAV database and KPI’s from NAV using BDC integration from Bugsy’s blog. This machine would typically connect directly to NAV (due to authentication), but can go through the Internal Proxy for some scenarios. There are a lot of posts on my blog and other places on how to connect to NAV Web Services from an internal application like this.
2.  This machine is the internal proxy. It exposes a high level contract for machines in the DMZ on port 8123. The firewall is open for the 2 specific machines in the DMZ on this specific port to this specific machine. This Web Service listener is open for Anonymous access and the machine connects to NAV with a hardcoded user/password.
3.  This is the NAV Service Tier – database could be on the same machine. The Web Service listener listens on port 7047 and uses Windows Authentication.
4.  This machine is a Web Server hosting the company web site and a web shop for selling our products. The authentication between customer and the Web Application is custom and the customer does not have anything to do with the Active Directory in the Intranet.
5.  This machine is a Web Services listener, allowing our top customers to buy our products directly though a module in their own NAV application. The authentication between customer and the Web Service is custom and the customer does not have anything to do with the Active Directory in the Intranet.

In this post – I will concentrate on machine number 2.

# My Internal Proxy

I have chosen to create this proxy service as a Windows Service, hosting a WCF Service. The other choice would be to create an asp.net web service application hosted by IIS – which probably is easier to find information about on the internet, but that is just because it has been around for a longer period of time and because the general assumption is that it is complicated.

One advantage you have by making a Windows Service is, that you can have that Windows Service running as a domain account with Access to NAV and this way, you don’t have to have username and password in clear text in your web service host.

Another advantage is that it is lightweight and there really isn’t any reason for having IIS loaded on a computer in the intranet unless it is running your local intranet web site.

I have chosen that the contract I want to implement is to get the Customer name of a specific customer and there really isn’t a lot of code in that.

# Writing a WCF service is complicated!

A WCF Service Contract and the service itself is only these few lines of code:

```
[ServiceContract] 
interface IMyInternalProxy 
{ 
    [OperationContract] 
    string GetCustomerName(string No); 
}

[ServiceBehavior(InstanceContextMode=InstanceContextMode.Single)] 
class MyInternalProxy : IMyInternalProxy 
{ 
    #region IMyInternalProxy Members

    public string GetCustomerName(string No) 
    { 
        Customer_Service service = new Customer_Service(); 
        service.Credentials = CredentialCache.DefaultNetworkCredentials; 
        Customer customer = service.Read(No); 
        return customer.Name; 
    }

    #endregion 
}
```

as you can see I chose to connect to NAV using a Web Reference. I could of course have done this using Service Reference as well as described in [this post](/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-code-version/).

# That’s not complicated!

No and actually by hosting the WCF Service in a Windows service you just need to create the ServiceHost and perform the open and close in the proper event handlers like:

```
public partial class Service1 : ServiceBase 
{ 
    string URL = Uri.UriSchemeHttp + Uri.SchemeDelimiter + Environment.MachineName + ":8123"; 
    ServiceHost host;

    public Service1() 
    { 
        InitializeComponent();

        host = new ServiceHost(new MyInternalProxy(), new Uri(URL)); 
        host.AddServiceEndpoint(typeof(IMyInternalProxy), new BasicHttpBinding(), ""); 
        ServiceMetadataBehavior smb = new ServiceMetadataBehavior(); 
        smb.HttpGetEnabled = true; 
        smb.HttpGetUrl = new Uri(URL); 
        host.Description.Behaviors.Add(smb); 
    }

    protected override void OnStart(string[] args) 
    { 
        if (host.State != CommunicationState.Opened && host.State != CommunicationState.Opening) 
            host.Open(); 
    }

    protected override void OnStop() 
    { 
        if (host.State != CommunicationState.Closed && host.State != CommunicationState.Closing) 
            host.Close(); 
    } 
}
```

# Well that’s not complicated either – but something must be?

No, it’s easy – if it fits, it ships!

(Ok, bad joke, People who hasn’t seen USPS commercials might be a bit confused now:-))

# But what about installing, uninstalling etc. etc.

Ok admitted, there is a bit more to writing a Windows Service hosting a WCF service than just writing two classes – but that is the case with IIS hosted Web Services as well. To describe all these things is actually complicated – so instead, I have created a 16 minute video on how to create MyInternalProxy from scratch, install it and create a very small client application for the proxy – and you can download the video here:

[http://www.freddy.dk/MyInternalProxy.wmv](http://www.freddy.dk/MyInternalProxy.wmv)

I tried uploading to Youtube, but they have a max. of 10 minutes:-(

And…, you can download the solution from the video [here](http://www.freddy.dk/MyInternalProxy.zip).

Hope this is helpful

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
