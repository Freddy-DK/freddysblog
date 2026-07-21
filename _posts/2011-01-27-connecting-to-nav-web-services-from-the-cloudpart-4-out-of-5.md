---
layout: post
title: "Connecting to NAV Web Services from the Cloud–part 4 out of 5"
date: 2011-01-27 07:55:33
categories: ["Archive", "Web Services"]
tags: ["AppFabric", "Authentication", "Azure", "Proxy", "Service Reference", "Service Tier", "Servicebus", "WCF", "Web Services", "Windows Phone 7"]
permalink: /2011/01/27/connecting-to-nav-web-services-from-the-cloudpart-4-out-of-5/
---

If you haven’t already read part 3 you should do so [here](/2011/01/27/connecting-to-nav-web-services-from-the-cloudpart-3-out-of-5/), before continuing to read this post.

By now you have seen how to create a WCF Service Proxy connected to NAV with an endpoint hosted on the Servicebus (Windows Azure AppFabric). By now, I haven’t written anything about security yet and the Proxy1 which is hosted on the Servicebus is available for everybody to connect to anonymously.

In the real world, I cannot imagine a lot of scenarios where a proxy which can be accessed anonymously will be able to access and retrieve data from your ERP system, so we need to look into this and there are a lot of different options – but unfortunately, a lot of these doesn’t work from Windows Phone 7.

# ACS security on Windows Azure AppFabric

The following links points to documents describing how to secure services hosted on the Servicebus through the Access Control Service (ACS) built into Windows Azure AppFabric.

[An Introduction to Microsoft .NET Services for Developers](http://download.microsoft.com/download/6/9/6/696C233B-1AC8-4F26-A55C-A65F8D3BDFA0/An%20Introduction%20to%20Windows%20Azure%20AppFabric%20for%20Developers.docx "An Introduction to Microsoft .NET Services for Developers")

[A Developer’s Guide to the Service Bus](http://download.microsoft.com/download/D/6/F/D6F4E54C-F6A7-48FA-AE03-4C543B14B41A/A%20Developer's%20Guide%20to%20Service%20Bus%20in%20Windows%20Azure%20AppFabric.docx "A Developer’s Guide to the Service Bus")

[A Developer’s Guide to the .NET Access Control Service](http://download.microsoft.com/download/5/A/7/5A7703E2-32DA-4D53-9EFD-A200E1E93562/A%20Developer's%20Guide%20to%20Access%20Control%20in%20Windows%20Azure%20AppFabric.docx "A Developer’s Guide to the .NET Access Control Service")

[Windows Azure platform](http://www.microsoft.com/windowsazure/)

A lot of good stuff and if you (like me) try to dive deep into some of these topics, you will find that using this technology, you can secure services without changing the signature of your service. Basically you are using claim based security, asking a service for a token with which you can connect to a service. You get a token back (which will be valid for a given timeframe) and while this token is valid you can call the service by specifying this token when calling the service.

This functionality has two problems:

1.  It doesn’t work with Windows Phone 7 (nor any other mobile device) – YET! In the future, it probably will – but nobody made a servicebus assembly for Windows Phone, PHP, Java, iPhone or any of those. See also: [http://social.msdn.microsoft.com/Forums/en-US/netservices/thread/12660485-86fb-4b4c-9417-d37fe183b4e1](http://social.msdn.microsoft.com/Forums/en-US/netservices/thread/12660485-86fb-4b4c-9417-d37fe183b4e1 "http://social.msdn.microsoft.com/Forums/en-US/netservices/thread/12660485-86fb-4b4c-9417-d37fe183b4e1")
2.  Often we need to connect to NAV with different A/D users depending on which user is connecting from the outside.

I am definitely not a security expert – but whatever security system I want to put in place must support the following two basic scenarios:

-   My employees, who are using Windows Phone 7 (or other applications), needs to authenticate to NAV using their own domain credentials – to have the same permissions in NAV as they have when running the Client.
-   Customers and vendors should NOT have a username and a password for a domain user in my network, but I still want to control which permissions they have in NAV when connecting.

# Securing the access to NAV

I have created the following poor mans authentication to secure the access to NAV and make sure that nobody gets past the Proxy unless they have a valid username and password. I will of course always host the endpoint on a secure connection ([https://](https://)) and the easiest way around adding security is to extend the methods in the proxy to contain a username and a password – like:

```
[ServiceContract] 
public interface IProxyClass 
{ 
    [OperationContract] 
    string GetCustomerName(string username, string password, string No); 
}
```

This username and password could then be a domain user and a domain password, but depending on the usage this might not be the best of ideas.

In the case of the Windows Phone application, where a user is connecting to NAV through a proxy, I would definitely use the domain username and password (like it is the case with Outlook on the Windows Phone). In the case, where an application on a customer/vendor site should connect to your local NAV installation over the Servicebus – it doesn’t seem like a good idea to create a specific domain user and password for this customer/vendor and maintain that.

In my sample here – I will allow both. Basically I will just say, that if the username is of the format domainuser – then I will assume that this is a domain user and password – and use that directly as credentials on the service connection to NAV.

If the username does NOT contain a backslash, then I will lookup in a table for this username and password combination and find the domain, username and password that should be used for this connection. This table can either be a locally hosted SQL Express – or it could be a table in NAV and then I could access this through another web service. I have done the latter.

In NAV, I have created a table called AuthZ:

[![image](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-4-out-of-5/3884.image_19f43fe6.png?w=625&h=284 "image")](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-4-out-of-5/3884.image_19f43fe6.png)

This table will map usernames and passwords to domain usernames and passwords.

As a special option I have added a UseDefault – which basically means that the Service will use default authentication to connect to NAV, which again would be the user which is running the Service Host Console App or Windows Service. I have filled in some data into this table:

[![image](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-4-out-of-5/8686.image_1977e724.png?w=594&h=196 "image")](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-4-out-of-5/8686.image_1977e724.png)

and created a Codeunit called AuthZ (which I have exposed as a Web Service as well):

```
GetDomainUserPassword(VAR Username : Text[80];VAR Password : Text[80];VAR Domain : Text[80];VAR UseDefault : Boolean) : Boolean 
IF NOT AuthZ.GET(Username) THEN 
BEGIN 
  AuthZ.INIT; 
  AuthZ.Username := Username; 
  AuthZ.Password := CREATEGUID; 
  AuthZ.Disabled := TRUE; 
  AuthZ.Attempts := 1; 
  AuthZ.AllTimeAttempts := 1; 
  AuthZ.INSERT(); 
  EXIT(FALSE); 
END; 
IF AuthZ.Password <> Password THEN 
BEGIN 
  AuthZ.Attempts := AuthZ.Attempts + 1; 
  AuthZ.AllTimeAttempts := AuthZ.AllTimeAttempts + 1; 
  IF AuthZ.Attempts > 3 THEN 
    AuthZ.Disabled := TRUE; 
  AuthZ.MODIFY(); 
  EXIT(FALSE); 
END; 
IF AuthZ.Disabled THEN 
  EXIT(FALSE); 
IF AuthZ.Attempts > 0 THEN 
BEGIN 
  AuthZ.Attempts := 0; 
  AuthZ.MODIFY(); 
END; 
UseDefault := AuthZ.UseDefault; 
Username := AuthZ.DomainUsername; 
Password := AuthZ.DomainPassword; 
Domain := AuthZ.DomainDomain; 
EXIT(TRUE);
```

As you can see, the method will log the number of attempts done on a given user and it will automatically disable a user if it has more than 3 failed attempts. In my sample, I actually create a disabled account for attempts to use wrong usernames – you could discuss whether or not this is necessary.

In my proxy (C#) I have added a method called Authenticate:

```
private void Authenticate(SoapHttpClientProtocol service, string username, string password) 
{ 
    if (string.IsNullOrEmpty(username)) 
        throw new ArgumentNullException("username"); 
    if (username.Length > 80) 
        throw new ArgumentException("username");

    if (string.IsNullOrEmpty(password)) 
        throw new ArgumentNullException("password"); 
    if (password.Length > 80) 
        throw new ArgumentException("password");

    string[] creds = username.Split('\\');

    if (creds.Length > 2) 
        throw new ArgumentException("username");

    if (creds.Length == 2) 
    { 
        // Username is given by domain\user - use this 
        service.Credentials = new NetworkCredential(creds[1], password, creds[0]); 
    } 
    else 
    { 
        // Username is a simple username (no domain) 
        // Use AuthZ web Service in NAV to get the Windows user to use for this user 
        AuthZref.AuthZ authZservice = new AuthZref.AuthZ(); 
        authZservice.UseDefaultCredentials = true; 
        string domain = ""; 
        bool useDefault = true; 
        if (!authZservice.GetDomainUserPassword(ref username, ref password, ref domain, ref useDefault)) 
            throw new ArgumentException("username/password"); 
        if (useDefault) 
            service.UseDefaultCredentials = true; 
        else 
            service.Credentials = new NetworkCredential(username, password, domain); 
    } 
}
```

The only task for this function is to authenticate the user and set the Credentials on a NAV Web Reference (SoapHttpClientProtocol derived). If something goes wrong, this function will throw and exception and will return to the caller of the Service.

The implementation of GetCustomerName is only slightly different in relation to the one implemented in part 3 – only addition is to call Authenticate after creating the service class:

```
public string GetCustomerName(string username, string password, string No) 
{ 
    Debug.Write(string.Format("GetCustomerName(No = {0}) = ", No)); 
    CustomerCard_Service service = new CustomerCard_Service(); 
    try 
    { 
        Authenticate(service, username, password); 
    } 
    catch (Exception ex) 
    { 
        Debug.WriteLine(ex.Message); 
        throw; 
    } 
    CustomerCard customer = service.Read(No); 
    if (customer == null) 
    { 
        Debug.WriteLine("Customer not found"); 
        return string.Empty; 
    } 
    Debug.WriteLine(customer.Name); 
    return customer.Name; 
}
```

So, based on this we now have an endpoint on which we will have to specify a valid user/password combination in order to get the Customer Name – without having to create / give out users in the Active Directory.

Note that with security it is ALWAYS good to create a threat model and depending on the data you expose might want to add more security.

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
