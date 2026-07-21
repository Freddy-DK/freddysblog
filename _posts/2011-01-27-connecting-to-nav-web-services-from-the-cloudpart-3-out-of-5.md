---
layout: post
title: "Connecting to NAV Web Services from the Cloud–part 3 out of 5"
date: 2011-01-27 05:46:55
categories: ["Archive", "Web Services"]
tags: ["Azure", "Proxy", "Service Tier", "Servicebus", "WCF", "Web Services"]
permalink: /2011/01/27/connecting-to-nav-web-services-from-the-cloudpart-3-out-of-5/
---

If you haven’t already read part 2 you should do so [here](/2010/12/02/connecting-to-nav-web-services-from-the-cloudpart-2-out-of-5/), before continuing to read this post.

In part 2 I talked about how to connect to my locally installed NAV Web Service Proxy from anywhere in the world and towards the end, I promised that I would explain how the proxy was build. Problem was, that while writing this post I ran into a bug in the Servicebus – which was really annoying.

The bug causes my service to stop listening after a number of hours or days – and as such, I couldn’t create a reliable way to host a service on the Servicebus and I didn’t want to post information about how to do stuff like this and then stand the risk of having mislead a number of people with all kinds of problems to follow.

In the beginning I thought this problem was caused by inactivity on the service, but after several tests (which took days each of these) I found that the problem only occurs when hosting a metadata endpoint in the cloud. Now one can argue that a metadata endpoint is only for development purposes – yes, but we are also doing development here, so I have to do that.

Anyway – knowing that I have a workaround, I will now explain how Proxy1 is created.

# Proxy1

The solution I will create consists of 2 projects. One project compiles to a DLL and contains the ServiceHost and the actual Proxy. The other project is a Console application, which is used as a host application for the DLL. Later on I will add 2 projects more – a Windows Service and an installer – much like explained in this post [https://freddysblog.com/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/](/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/ "https://freddysblog.com/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/")

The Proxy1 Windows Service or Console Application will be running on the local network, next to the NAV Service Tier and the application/Service should be running as a user, who has access to NAV. In one of the later posts, I will explain about authentication and security and ways to make this safer.

First of all – we need to define our Service Contract:

\[ServiceContract\] 
public interface IProxyClass 
{ 
    \[OperationContract\] 
    string GetCustomerName(string No); 
}

and secondly the implementation of this Proxy:

\[ServiceBehavior(InstanceContextMode = InstanceContextMode.Single, IncludeExceptionDetailInFaults = true)\] 
public class ProxyClass : IProxyClass 
{ 
    public ProxyClass() 
    { 
    }

    public string GetCustomerName(string No) 
    { 
        Debug.Write(string.Format("GetCustomerName(No = {0}) = ", No)); 
        CustomerCard\_Service service = new CustomerCard\_Service(); 
        service.UseDefaultCredentials = true; 
        CustomerCard customer = service.Read(No); 
        if (customer == null) 
        { 
            Debug.WriteLine("Customer not found"); 
            return string.Empty; 
        } 
        Debug.WriteLine(customer.Name); 
        return customer.Name; 
    } 
}

as you can see, the implementation has a reference to the Customer Card in my NAV – and I am using default authentication (current user) and just calling into NAV to read a Customer and then return the name – almost as simple as a Hello World sample.

This part is more or less exactly the same proxy as described in this post [https://freddysblog.com/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/](/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/ "https://freddysblog.com/2010/01/30/web-services-infrastructure-and-how-to-create-an-internal-proxy/") – now the question is, how do we make this proxy accessible from everywhere in the world.

# Hosting a WCF Service on the Servicebus

In my prior post about creating an internal proxy, we would use the following lines to create a ServiceHost:

host = new ServiceHost(new MyInternalProxy(), new Uri(URL)); 
host.AddServiceEndpoint(typeof(IMyInternalProxy), new BasicHttpBinding(), ""); 
ServiceMetadataBehavior smb = new ServiceMetadataBehavior(); 
smb.HttpGetEnabled = true; 
smb.HttpGetUrl = new Uri(URL); 
host.Description.Behaviors.Add(smb);

This would of course create a host and listen on the URL. One could think that you could replace the URL with a magic URL on the service bus and then everything would work. Well – it is not THAT simple.

In my samples I have created a ServiceClass, which then is used from my Console App and from my Windows Service.

public class ServiceClass 
{ 
    ServiceHost serviceHost = null; 
    string instanceId = "Proxy1"; 
    bool includeMex;

    public ServiceClass(bool includeMex) 
    { 
        this.includeMex = includeMex; 
    }

    void InitializeServiceHost() 
    { 
        serviceHost = new ServiceHost(new ProxyClass());

        // sb:// binding 
        Uri sbUri = ServiceBusEnvironment.CreateServiceUri("sb", "navdemo", instanceId); 
        var sbBinding = new NetTcpRelayBinding(EndToEndSecurityMode.Transport, RelayClientAuthenticationType.None); 
        serviceHost.AddServiceEndpoint(typeof(IProxyClass), sbBinding, sbUri);

        // https:// binding (for Windows Phone etc.) 
        Uri httpsUri = ServiceBusEnvironment.CreateServiceUri("https", "navdemo", "https/" + instanceId); 
        var httpsBinding = new BasicHttpRelayBinding(EndToEndBasicHttpSecurityMode.Transport, RelayClientAuthenticationType.None); 
        serviceHost.AddServiceEndpoint(typeof(IProxyClass), httpsBinding, httpsUri);

        if (this.includeMex) 
        { 
            // sb:// Metadata endpoint 
            Uri mexUri = new Uri(sbUri.AbsoluteUri + "mex"); 
            var mexBinding = new NetTcpRelayBinding(EndToEndSecurityMode.Transport, RelayClientAuthenticationType.None); 
            ServiceMetadataBehavior smb = new ServiceMetadataBehavior(); 
            serviceHost.Description.Behaviors.Add(smb); 
            serviceHost.AddServiceEndpoint(typeof(IMetadataExchange), mexBinding, mexUri); 
        }

        // Setup Shared Secret Credentials for hosting endpoints on the Service Bus 
        string issuerName = "name"; 
        string issuerSecret = "secret"; 
        TransportClientEndpointBehavior sharedSecretServiceBusCredential = new TransportClientEndpointBehavior(); 
        sharedSecretServiceBusCredential.CredentialType = TransportClientCredentialType.SharedSecret; 
        sharedSecretServiceBusCredential.Credentials.SharedSecret.IssuerName = issuerName; 
        sharedSecretServiceBusCredential.Credentials.SharedSecret.IssuerSecret = issuerSecret;

        // Set credentials on all endpoints on the Service Bus 
        foreach (ServiceEndpoint endpoint in serviceHost.Description.Endpoints) 
        { 
            endpoint.Behaviors.Add(sharedSecretServiceBusCredential); 
        }

        Debug.WriteLine(string.Format("{0} Initialized", this.GetType().FullName)); 
    }

    public void StartHosts() 
    { 
        if (this.serviceHost == null) 
        { 
            Debug.WriteLine(string.Format("{0} Initializing...", this.GetType().FullName)); 
            InitializeServiceHost(); 
        } 
        if (this.serviceHost != null && serviceHost.State != CommunicationState.Opened && serviceHost.State != CommunicationState.Opening) 
        { 
            Debug.WriteLine(string.Format("{0} Opening...", this.GetType().FullName)); 
            this.serviceHost.Open(); 
            Debug.WriteLine(string.Format("{0} Opened", this.GetType().FullName)); 
        } 
    }

    public void StopHosts() 
    { 
        if (this.serviceHost != null && serviceHost.State != CommunicationState.Closed && serviceHost.State != CommunicationState.Closing) 
        { 
            Debug.WriteLine(string.Format("{0} Closing...", this.GetType().FullName)); 
            this.serviceHost.Close(); 
            Debug.WriteLine(string.Format("{0} Closed", this.GetType().FullName)); 
        } 
        this.serviceHost = null; 
    } 
}

and yes – it is slightly more complicated, but if you look twice it isn’t that different. For each endpoint you will host, you will need a Type of the Service Contract, a binding (describing the communication protocol – BasicHttpBinding in the “old” sample), and a URL where the Service is hosted.

// sb:// binding 
Uri sbUri = ServiceBusEnvironment.CreateServiceUri("sb", "navdemo", instanceId); 
var sbBinding = new NetTcpRelayBinding(EndToEndSecurityMode.Transport, RelayClientAuthenticationType.None); 
serviceHost.AddServiceEndpoint(typeof(IProxyClass), sbBinding, sbUri);

The bindings for the Servicebus are Relay bindings and you will find these when referencing the Microsoft.Servicebus DLL.

The NetTcpRelayBinding class takes two parameters – securitymode (which is set to Transport here – meaning that we will have a secure line) and a RelayAuthenticationType, which is set to None – everybody can access this endpoint. Setting the RelayClientAuthenticationType to RelayAccessToken means that an application connecting to this endpoint will have to specify a specify RelayAuthenticationToken to connect to this endpoint, which again means that your proxy will never get called unless people can get by the 1st level authentication done on the endpoint.

My other binding is https – and again RelayAuthenticationType is set to None. The primary reason for the https endpoint is Windows Phone – Windows Phone doesn’t know about the sb:// protocol and it also doesn’t know anything about RelayAuthenticationToken – so this needs to be None.

After setting up the endpoints, we add SharedSecretServicebusCredential behaviors to all endpoints. Reason for this is that the ServiceBus require authentication in order to host an endpoint on the Servicebus. People cannot just go and host endpoints on my Servicebus domain – and then leave the bill to me.

The value of these variables:

string issuerName = "name";
string issuerSecret = "secret";

is given from the Windows Azure AppFabric account when you sign up.

Furthermore – the name “navdemo” (when creating the ServiceUri is registered by me. In order to use this you will have to sign up for a Windows Azure account and register your own service namespace.

# The Console App

is very simple, just an app. hosting the ServiceClass. The only thing complicating this is, that I want to restart the servicehost for every hour or so if we are hosting a metadata endpoint – in order to avoid the Servicebus bug.

class Program 
{ 
     static Proxy1.ServiceClass proxy1; 
     static bool includeMex = true;

     static void Main(string\[\] args) 
     { 
         Console.WindowWidth = 120; 
         Console.WindowHeight = 50; 
         Debug.Listeners.Add(new TextWriterTraceListener(System.Console.Out));

         if (includeMex) 
         { 
             Timer timer = new Timer(3600000); // 1 hour in milliseconds 
             timer.Elapsed += new ElapsedEventHandler(timer\_Elapsed); 
             timer.Start(); 
             Console.WriteLine("Timer started - listening for 1 hour"); 
         }

         proxy1 = new Proxy1.ServiceClass(includeMex); 
         proxy1.StartHosts(); 
         Console.ReadLine(); 
         proxy1.StopHosts(); 
     }

     static void timer\_Elapsed(object sender, ElapsedEventArgs e) 
     { 
         Console.WriteLine("Timer elapsed - restart listener"); 
         proxy1.StopHosts(); 
         proxy1.StartHosts(); 
     } 
}

When running in production, includeMex should be false.

You can download the entire solution [here](http://www.freddy.dk/Proxy1.zip) – but of course you cannot run the solution before you have a Windows Azure account.

More information about Servicebus (Windows Azure AppFabric) can be found here:

[Windows Azure AppFabric general information (and free trial)](http://www.microsoft.com/windowsazure/appfabric/overview/default.aspx)

[AppFabric Service Bus Tutorial and information about how to create and account and a namespace](http://msdn.microsoft.com/en-us/library/ee706736.aspx)

[Windows Azure AppFabric on MSDN](http://msdn.microsoft.com/en-us/library/gg577633.aspx)

[http://blogs.msdn.com/b/windowsazureappfabric/](http://blogs.msdn.com/b/windowsazureappfabric/ "http://blogs.msdn.com/b/windowsazureappfabric/")

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
