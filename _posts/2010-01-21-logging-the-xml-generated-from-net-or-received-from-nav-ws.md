---
layout: post
title: "Logging the XML generated from .net or received from NAV WS"
date: 2010-01-21 14:06:43
categories: ["Archive"]
tags: ["C#", "Log", "NAV 2009 SP1", "Service Reference", "Web Services", "XML"]
permalink: /2010/01/21/logging-the-xml-generated-from-net-or-received-from-nav-ws/
---

When working with Web Services using languages who doesn’t natively have Web Services support (like Javascript and NAV self) you have to create a SOAP envelope yourself in the correct format.

Of course you can do so by looking at the WSDL, understanding SOAP and using theory – or… – you can create a small C# application, invoke the Web Service you want to and see what XML .net creates for this.

You can also see what XML you get back before .net makes this into classes and feed you something high level.

I only know how to do this with a .net 3.5 application using Service References. I don’t know how to hook into an application using Web References.

I will use the application I created in [this post](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-code-version.aspx), and basically what we have to do is, to create and plug-in a behavior that prints out the XML.

### Endpoint Behaviors

An Endpoint Behavior is a class implementing the _[IEndpointBehavior](http://msdn.microsoft.com/en-us/library/system.servicemodel.description.iendpointbehavior.aspx)_ interface. This interface consists of 4 methods:

-   ```
    public void AddBindingParameters(ServiceEndpoint endpoint,
    System.ServiceModel.Channels.BindingParameterCollection bindingParameters)
    ```
    
-   ```
    public void ApplyClientBehavior(ServiceEndpoint endpoint,
    System.ServiceModel.Dispatcher.ClientRuntime clientRuntime)
    ```
    
-   ```
    public void ApplyDispatchBehavior(ServiceEndpoint endpoint,
    System.ServiceModel.Dispatcher.EndpointDispatcher endpointDispatcher)
    ```
    
-   `public void Validate(ServiceEndpoint endpoint)`

In my implementation I will leave all empty except for ApplyClientBehavior, where I get access to the ClientRuntime object.

This object has a collection of MessageInspectors – and this is where I want to hook up.

### A Client Message Inspector

A Client Message Inspector is a class implementing the _[IClientMessageInspector](http://msdn.microsoft.com/en-us/library/system.servicemodel.dispatcher.iclientmessageinspector.aspx)_ interface. This interface consists of 2 methods:

-   ```
    public void AfterReceiveReply(ref System.ServiceModel.Channels.Message reply,
    object correlationState)
    ```
    
-   ```
    public object BeforeSendRequest(ref System.ServiceModel.Channels.Message request,
    System.ServiceModel.IClientChannel channel)
    ```
    

and what I want to do in both these methods is to print the contents of the reply/request to the console.

Now there is a special consideration to take here – as soon as you have taken a copy of the message, you have to replace the message with a new one (even if it is a copy of the old one) – else you will get the following error:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/LoggingtheXMLgene.netorreceivedfromNAVWS_C66F/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/LoggingtheXMLgene.netorreceivedfromNAVWS_C66F/image_2.png)

Actually a very explanatory error message, but kind of weird.

### The Code

I created a .cs file and added the following two classes to the file:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.ServiceModel.Description;
using System.ServiceModel.Dispatcher;
using System.ServiceModel.Channels;
using System.Xml;
using System.IO;
```

```
namespace testAppWCF
{
public class MyBehavior : IEndpointBehavior
{
```

        `#region IEndpointBehavior Members`

        

```
public void AddBindingParameters(ServiceEndpoint endpoint, System.ServiceModel.Channels.BindingParameterCollection bindingParameters)
{
}
```

        

```
public void ApplyClientBehavior(ServiceEndpoint endpoint, System.ServiceModel.Dispatcher.ClientRuntime clientRuntime)
{
clientRuntime.MessageInspectors.Add(new MyMessageInspector());
}
```

        

```
public void ApplyDispatchBehavior(ServiceEndpoint endpoint, System.ServiceModel.Dispatcher.EndpointDispatcher endpointDispatcher)
{
}
```

        

```
public void Validate(ServiceEndpoint endpoint)
{
}
```

        

```
#endregion
}
```

    

```
public class MyMessageInspector : IClientMessageInspector
{
#region IClientMessageInspector Members
```

        

```
public void AfterReceiveReply(ref System.ServiceModel.Channels.Message reply, object correlationState)
{
MessageBuffer buffer = reply.CreateBufferedCopy(Int32.MaxValue);
reply = buffer.CreateMessage();
Message msg = buffer.CreateMessage();
StringBuilder sb = new StringBuilder();
XmlWriter xw = XmlWriter.Create(sb);
msg.WriteBody(xw);
xw.Close();
Console.WriteLine("Received:n{0}", msg.ToString());
Console.WriteLine("Body:n{0}", sb.ToString());
}
```

        

```
public object BeforeSendRequest(ref System.ServiceModel.Channels.Message request, System.ServiceModel.IClientChannel channel)
{
MessageBuffer buffer = request.CreateBufferedCopy(Int32.MaxValue);
request = buffer.CreateMessage();
Console.WriteLine("Sending:n{0}", buffer.CreateMessage().ToString());
return null;
}
```

        

```
#endregion
}
}
```

Note that the AfterReceiveReply takes special consideration as the actual body comes as a stream and in order to output that I create a XmlWriter and write the body to a string through that one before outputting it to the console.

### Adding the behavior to the endpoint

In the main application, we then have to add this to the endpoint, which is done by adding the following line:

`systemService.Endpoint.Behaviors.Add(new MyBehavior());`

to the code after initialization of the systemService and

`customerService.Endpoint.Behaviors.Add(new MyBehavior());`

to the code after initialization of the customerService.

### F5

When running the application, our console now looks somewhat different.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/LoggingtheXMLgene.netorreceivedfromNAVWS_C66F/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/LoggingtheXMLgene.netorreceivedfromNAVWS_C66F/image_6.png)

If you look closely you can find Companies: at the top and a list of the companies 3/4’s down.

Everything in between is what is send AND what is received from Web Services. Of course dumping this to the Console isn’t necessarily useful but I assume that you can find ways to dump this to something else if you need.

Enjoy

_**Freddy Kristiansen**  
_PM Architect  
Microsoft Dynamics NAV
