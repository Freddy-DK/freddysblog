---
layout: post
title: "Connecting to NAV Web Services from the Cloud–Part 1 out of 5"
date: 2010-11-30 23:59:13
categories: ["Archive", "Web Services"]
tags: ["Azure", "Servicebus", "WCF", "Web Services"]
permalink: /2010/11/30/connecting-to-nav-web-services-from-the-cloudpart-1-out-of-5/
---

My last post in January 2010 ([here](/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/)) talked about how to create a Proxy for connecting to NAV Web Services from outside the local intranet, by creating a local proxy and exposing this through the firewall to your DMZ.

It still isn’t simple though and there are a lot of tasks to complete in order to make this work:

1.  Define the API which you need to expose to the external application – and create the internal proxy
2.  Decide what authentication method to use (maybe this needs some coding)
3.  Decide what security measures you need to put in place (external proxy, firewall rules, IP level security, etc.)
4.  Setup servers for proxys, modify firewalls etc. etc.

All in all – if you are creating an application, which should be easy to install – all of this really doesn’t make things easier. You need to demand a number of things of the network infrastructure from the customer before you will be able to make things work.

Also, a number of NAV customers might not have the infrastructure at all, which will make things more expensive.

# What then?

Imagine if you could create a proxy, which could expose a service in the Cloud in a secure way, without any of the infrastructure problems mentioned above? A Proxy for which you could create an installer and you would just have to give authentication tokens to the people who needs to connect to your service and then communication would flow problem free. You wouldn’t have to change firewalls, you wouldn’t have to setup VPN’s, IP security etc. etc.

That almost sounds too good to be true right?

# Windows Azure AppFabric (Servicebus)

As you can imagine, it is possible to expose services in the Cloud problem free – and one mechanism for doing this is using Windows Azure AppFabric – or the Servicebus.

Actually it isn’t rocket science – it is pretty simple and the technology has existed for quite some time.

If you are from the generation, who remembers, that a phone once looked like this:

![8182.phone\_30C96CAE](/assets/images/2010/connecting-to-nav-web-services-from-the-cloudpart-1-out-of-5/8182.phone_30c96cae.jpg)

then you also know that you would have to pick up the phone, turn the lever on the right side a couple of rounds and then some nice lady with a switchboard would ask who you wanted to be connected to.

![6076.switchboard\_653A1929](/assets/images/2010/connecting-to-nav-web-services-from-the-cloudpart-1-out-of-5/6076.switchboard_653a1929.jpg)

The lady would then physically connect your phone to the phone of the person you want to dial and make the phone ring.

All of this of course runs automatically today, but the basic functionality is the same – you connect to a switchboard which automatically connects you to the person you are calling. The same method is used in network switches and routers of course and if you are running Live Messenger or any other chat program, these programs actually often work in the same way.

You start a program on your computer and now suddenly everybody you know can send instant messages to your computer, which will popup on your screen. This is not because the chat software is polling a service in the cloud 10 times a second, nor is it because the software modifies your firewall to allow other people to connect to you. What happens is, that the software will open up a communication channel to the chat server (everybody is doing that), and then the chat server is working as a router for messages between one computer and the other.

![8055.image\_46BFB273](/assets/images/2010/connecting-to-nav-web-services-from-the-cloudpart-1-out-of-5/8055.image_46bfb273.png)

The same goes for the Servicebus – it works as a meeting point in the Cloud for WCF services – allowing you to expose a service in the Cloud by opening a communication channel to the Servicebus and then allowing authenticated users to communicate to you through this channel, only difference from the Chat Server is really that you decide on the API of the service in the Cloud.

# The hen or the egg

We need to start with either exposing services in the cloud or consuming services in the cloud). In order to see something quickly, I have created a service in the cloud, which is connected to a local installation of Microsoft Dynamics NAV 2009 R2 (could have been SP1) in order to showcase how consuming services hosted on Azure works. This service has a limited API, in fact it is just a cloud version of the Proxy post from back in January (can be found [here](http://blogs.msdn.com/b/freddyk/archive/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy.aspx)). This way you can try these samples and actually see that you can call a NAV Service over the Internet.

**Note: I will try to keep the service running, but there is absolutely no guarantee that this will be running all the time.**

My next blog post will then describe how to actually create this proxy, but for now it is all about consuming.

The service we are testing against only have one method, which returns the Customer Name, given the Customer No. I also exposed a metadata endpoint in the cloud, which you can use to create a service reference to my service, the URL is [sb://navdemo.servicebus.windows.net/Proxy1/mex](//navdemo.servicebus.windows.net/Proxy1/mex "sb://navdemo.servicebus.windows.net/Proxy1/mex")

# But wait… – what is sb:// ?

sb is an abbreviation of ServiceBus and you need to install the Windows Azure AppFabric SDK in order to have that available. You can download this SDK [here](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=39856a03-1490-4283-908f-c8bf0bfad8a5&displaylang=en).

Now you might think – does this mean that all clients will have to install this SDK in order to communicate with my service bus hosted service? and the answer is No, you can connect to a service bus hosted service using standard WCF bindings, but in order to do this, you need to expose an HTTPS endpoint. If you want to connect to a SB endpoint, you will need the SDK.

This also means that on the developer machine (on which you create the reference to the metadata endpoint) you will need the SDK installed.

# Consuming a Servicebus hosted service from C#

Lets just try the easiest possible way to consume my cloud hosted service and see that it actually works.

Start Visual Studio and create a console application.

Add a Service Reference to the URL: [sb://navdemo.servicebus.windows.net/Proxy1/mex](//navdemo.servicebus.windows.net/Proxy1/mex "sb://navdemo.servicebus.windows.net/Proxy1/mex") (set the Namespace to Proxy1). If this doesn’t work it is probably because you are missing the SDK mentioned above.

![2781.Proxy1-Service-Reference\_53A96CB7](/assets/images/2010/connecting-to-nav-web-services-from-the-cloudpart-1-out-of-5/2781.proxy1-service-reference_53a96cb7.png)

Write the following two lines of code:

```
var client = new Proxy1.ProxyClassClient("NetTcpRelayBinding_IProxyClass"); 
Console.WriteLine(client.GetCustomerName("10000"));
```

Run the application and you should get:

![7506.Proxy1-output\_4E2AD646](/assets/images/2010/connecting-to-nav-web-services-from-the-cloudpart-1-out-of-5/7506.proxy1-output_4e2ad646.png)

Kind of fun to think about that every single call you make to this service will run a code snippet in my locally hosted Microsoft Dynamics NAV 2009 – feel free to try – it contains all the customers from the W1 demo database.

In the next post I will go into some more details about how to consume this service without the SDK. I will also explain how to do this from Microsoft Dynamics NAV 2009 R2 and from a Windows Phone 7.

Stay tuned.

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
