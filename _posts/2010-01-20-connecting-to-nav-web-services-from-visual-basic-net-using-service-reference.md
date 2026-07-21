---
layout: post
title: "Connecting to NAV Web Services from Visual Basic .net using Service Reference"
date: 2010-01-20 09:51:28
categories: ["Archive", "Web Services"]
tags: ["NAV 2009 SP1", "Service Reference", "Visual Basic", "Web Services"]
permalink: /2010/01/20/connecting-to-nav-web-services-from-visual-basic-net-using-service-reference/
---

This post is really just a Visual Basic version of [this post](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-config-file-version.aspx) and [this post](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-code-version.aspx) combined, please read those posts before continuing this post.

As described in the other posts, there are two ways to work with Service References – one is to keep the configuration in a .config file and the other is to do everything from code.

### Using the .config file

First I create a Visual Basic Console application and add the two service references as explained in the other posts. Then I copied the ServiceModel part of the .config file from [this post](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-config-file-version.aspx) and pasted it into (and overwrite) the ServiceModel part of my new .config file.

After that, it is really just to write the code

`Module Module1`

    

```
Sub Main()
Dim baseURL As String = "
```

[`http://localhost:7047/DynamicsNAV/WS/"`](http://localhost:7047/DynamicsNAV/WS/")

First, connect to the System Web Service and list all companies:

        

```
Dim systemService As New SystemServiceRef.SystemService_PortClient("SystemService_Port", baseURL + "SystemService")
Dim companies() As String = systemService.Companies()
Console.WriteLine("Companies:")
For Each company As String In companies
Console.WriteLine(company)
Next
Dim cur As String = companies(0)
```

Now I have the company I want to use in _cur_ and the way I create a URL to the Customer page is by doing:

        

```
Dim customerPageURL As String = baseURL + Uri.EscapeDataString(cur) + "/Page/Customer"
Console.WriteLine(vbCrLf + "URL of Customer Page: " + customerPageURL)
```

and then I can create a Service Class to the Customer Page:

        `Dim customerService As New CustomerPageRef.Customer_PortClient("Customer_Port", customerPageURL)`

and using this, I read customer 10000 and output the name:

        

```
Dim customer10000 As CustomerPageRef.Customer = customerService.Read("10000")
Console.WriteLine(vbCrLf + "Name of Customer 10000: " + customer10000.Name)
```

Last, but not least – lets create a filter and read all customers in GB that has Location Code set to RED or BLUE:

        

```
Dim filter1 As New CustomerPageRef.Customer_Filter()
filter1.Field = CustomerPageRef.Customer_Fields.Country_Region_Code
filter1.Criteria = "GB"
```

        

```
Dim filter2 As New CustomerPageRef.Customer_Filter()
filter2.Field = CustomerPageRef.Customer_Fields.Location_Code
filter2.Criteria = "RED|BLUE"
```

        

```
Console.WriteLine(vbCrLf + "Customers in GB served by RED or BLUE warehouse:")
Dim filters() As CustomerPageRef.Customer_Filter = New CustomerPageRef.Customer_Filter(1) {filter1, filter2}
Dim customers() As CustomerPageRef.Customer = customerService.ReadMultiple(filters, Nothing, 0)
For Each customer As CustomerPageRef.Customer In customers
Console.WriteLine(customer.Name)
Next
```

        

```
Console.WriteLine(vbCrLf + "THE END")
Console.ReadLine()
End Sub
```

`End Module`

### Using Visual Basic code

If we want to avoid the .config file, the trick is very much like [this post](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-service-reference-code-version.aspx) to create the configuration in code.

Basically with the above solution, delete the app.config file and change the code to

`Module Module1`

    

```
Sub Main()
Dim baseURL As String = "
```

[`http://localhost:7047/DynamicsNAV/WS/"`](http://localhost:7047/DynamicsNAV/WS/")

        

```
Dim navWSBinding As New System.ServiceModel.BasicHttpBinding()
navWSBinding.Security.Mode = ServiceModel.BasicHttpSecurityMode.TransportCredentialOnly
navWSBinding.Security.Transport.ClientCredentialType = ServiceModel.HttpClientCredentialType.Windows
```

        

```
Dim systemService As New SystemServiceRef.SystemService_PortClient(navWSBinding, New System.ServiceModel.EndpointAddress(baseURL + "SystemService"))
systemService.ClientCredentials.Windows.AllowedImpersonationLevel = Security.Principal.TokenImpersonationLevel.Delegation
systemService.ClientCredentials.Windows.AllowNtlm = True
```

        

```
Dim companies() As String = systemService.Companies()
Console.WriteLine("Companies:")
For Each company As String In companies
Console.WriteLine(company)
Next
Dim cur As String = companies(0)
```

        

```
Dim customerPageURL As String = baseURL + Uri.EscapeDataString(cur) + "/Page/Customer"
Console.WriteLine(vbCrLf + "URL of Customer Page: " + customerPageURL)
```

        

```
Dim customerService As New CustomerPageRef.Customer_PortClient(navWSBinding, New System.ServiceModel.EndpointAddress(customerPageURL))
customerService.ClientCredentials.Windows.AllowedImpersonationLevel = Security.Principal.TokenImpersonationLevel.Delegation
customerService.ClientCredentials.Windows.AllowNtlm = True
```

        

```
Dim customer10000 As CustomerPageRef.Customer = customerService.Read("10000")
Console.WriteLine(vbCrLf + "Name of Customer 10000: " + customer10000.Name)
```

        

```
Dim filter1 As New CustomerPageRef.Customer_Filter()
filter1.Field = CustomerPageRef.Customer_Fields.Country_Region_Code
filter1.Criteria = "GB"
```

        

```
Dim filter2 As New CustomerPageRef.Customer_Filter()
filter2.Field = CustomerPageRef.Customer_Fields.Location_Code
filter2.Criteria = "RED|BLUE"
```

        

```
Console.WriteLine(vbCrLf + "Customers in GB served by RED or BLUE warehouse:")
Dim filters() As CustomerPageRef.Customer_Filter = New CustomerPageRef.Customer_Filter(1) {filter1, filter2}
Dim customers() As CustomerPageRef.Customer = customerService.ReadMultiple(filters, Nothing, 0)
For Each customer As CustomerPageRef.Customer In customers
Console.WriteLine(customer.Name)
Next
```

        

```
Console.WriteLine(vbCrLf + "THE END")
Console.ReadLine()
End Sub
```

`End Module`

In both cases you can change the user used to connect to Web Services by setting _service.ClientCredentials.Windows.ClientCredential_ to an instance of System.Net.NetworkCredential like:

`systemService.ClientCredentials.Windows.ClientCredential = New System.Net.NetworkCredential("user", "password", "domain")`

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
