---
layout: post
title: "Connecting to NAV Web Services from Visual Basic .net using Web Reference"
date: 2010-01-20 08:15:24
categories: ["Archive", "Web Services"]
tags: ["NAV 2009 SP1", "Visual Basic", "Web Reference", "Web Services"]
permalink: /2010/01/20/connecting-to-nav-web-services-from-visual-basic-net-using-web-reference/
---

This post is really just a Visual Basic version of [this post](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-web-reference.aspx), please read that post before continuing.

Note, this is my very first Visual Basic application. I don’t think there are any ways to do this easier – but then again – how should I know.

I am creating a Visual Basic Console application and adding two Web References (like it is done in [this post](http://blogs.msdn.com/freddyk/archive/2010/01/20/connecting-to-nav-web-services-from-c-using-web-reference.aspx)) and then it is really just writing the code

### The code

`Module Module1`

    `Sub Main()`

First, connect to the System Web Service and list all companies:

        `Dim baseURL As String = "`[`http://localhost:7047/DynamicsNAV/WS/"`](http://localhost:7047/DynamicsNAV/WS/")

        

```
Dim systemService As New SystemServiceRef.SystemService()
systemService.Url = baseURL + "SystemService"
systemService.UseDefaultCredentials = True
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

        

```
Dim customerService As New CustomerPageRef.Customer_Service()
customerService.Url = customerPageURL
customerService.UseDefaultCredentials = True
```

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

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
