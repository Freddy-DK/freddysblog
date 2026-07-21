---
layout: post
title: "Dynamic references to NAV Page Web Services in C#"
date: 2009-04-27 01:16:00
categories: ["Archive", "Web Services"]
tags: ["C#", "Dynamic", "Excel", "Web Services"]
permalink: /2009/04/27/dynamic-references-to-nav-page-web-services-in-c/
---

Note: There is an updated post about Dynamic references to NAV Page Web Services [here](http://blogs.msdn.com/freddyk/archive/2009/12/04/dynamic-references-to-nav-page-web-services-in-c-take-2.aspx).

When creating the very first (never published) version of Edit In Excel, it was loosely coupled, meaning that I did not have any Web references in the project to the Customer Page, Vendor Page or other pages. I read the WSDL and used XPath to traverse the XML and build up structures and I was able to attach to any Page Web Service.

The code wasn’t nice, and I was afraid that I would confuse more people than necessary if I posted that source. So I decided to go with a version, where I had a Web References in VS for every page you can connect to Excel.

The caveat of this approach is that everytime you customize the Customer Page, you need to recompile your Edit In Excel solution – AND you need to do some work if you want to add additional pages. Wouldn’t it be nice if we could avoid this?

If you study the code in the Edit In Excel, you will find that the type-strong web references really aren’t used that much, the majority of the code uses the type of the service, the type of the field enumeration etc.

Last week I stumbled over a very interesting blog called [crowsprogramming](http://www.crowsprogramming.com/). A special thanks to the author for this post:

[C# – Dynamically Invoke Web Service At Runtime](http://www.crowsprogramming.com/archives/66)

which in details shows how to ask the read the WSDL, build a service description and compile it into an assembly, and it basically consists of 3 methods:

```
/// <summary>
/// Builds an assembly from a web service description.
/// The assembly can be used to execute the web service methods.
/// </summary>
/// <param name="webServiceUri">Location of WSDL.</param>
/// <returns>A web service assembly.</returns>
public static Assembly BuildAssemblyFromWSDL(Uri webServiceUri)
```

```
/// <summary>
/// Builds the web service description importer, which allows us to generate a proxy class based on the
/// content of the WSDL described by the XmlTextReader.
/// </summary>
/// <param name="xmlreader">The WSDL content, described by XML.</param>
/// <returns>A ServiceDescriptionImporter that can be used to create a proxy class.</returns>
private static ServiceDescriptionImporter BuildServiceDescriptionImporter(XmlTextReader xmlreader)
```

```
/// <summary>
/// Compiles an assembly from the proxy class provided by the ServiceDescriptionImporter.
/// </summary>
/// <param name="descriptionImporter"></param>
/// <returns>An assembly that can be used to execute the web service methods.</returns>
private static Assembly CompileAssembly(ServiceDescriptionImporter descriptionImporter)
```

When you call the first method with our NAV WebServices URL, it reads the WSDL, creates a CodeDom of the proxy classes, compiles the proxy and returns an assembly, which you can reflect over – all it takes is the following line of code:

```
// create an assembly from the web service description
Assembly webServiceAssembly = BuildAssemblyFromWSDL(
```

`new Uri("`[`http://localhost:7047/DynamicsNAV/WS/CRONUS_International_Ltd/Page/Customer"));`](http://localhost:7047/DynamicsNAV/WS/CRONUS_International_Ltd/Page/Customer"\)\);)

### Type weak

So, what can you really do with an Assembly in your hand…

You cannot use statements like:

`if (customer.No == "10000")`

if the webservice isn’t added to the project. How should Visual Studio know that there is a No field in customer. So any Web References you add dynamically will only be there to reflect over and use via reflection, but you can do everything using reflection – it is just harder.

First of all – we can enumerate the public types in the assembly:

```
// Create Service Reference
Type[] types = webServiceAssembly.GetExportedTypes();
foreach (Type type in types)
Console.WriteLine(type.ToString());
```

running this, will output the following:

```
Customer_Service
Customer
Blocked
Copy_Sell_to_Addr_to_Qte_From
Application_Method
Reserve
Shipping_Advice
Customer_Filter
Customer_Fields
```

Which are the public types from an assembly, which is the proxy to a NAV Customer Page Webservice. Knowing that all pages follows the same pattern and that everything in Edit In Excel uses reflection over these classes anyway, it really became too compelling to rip out the Web Service References and make everything dynamic (that post will follow this one).

### Working with reflection

I am not going to go into detail about how reflection works and what you can do with reflection, but I will show some examples of how to work with the dynamic assembly. First of all we want to create our service class:

```
Type serviceType = webServiceAssembly.GetType("Customer_Service");
object service = Activator.CreateInstance(serviceType);
```

if using static web references, this would be `Customer_Service service = new Customer_Service();`

Now we need to set the UseDefaultCredentials property to true:

```
PropertyInfo useDefaultCredentials = service.GetType().GetProperty("UseDefaultCredentials");
useDefaultCredentials.SetValue(service, (object)true, new object[] { });
```

in other words, get the info-class about the property based on the type, and call the setValue on the propertyinfo, specifying the object instance you want to set the value in, the value and an empty array, specifying that there are no parameters for this call.

Using static web references, this would be `service.UseDefaultCredentials = true;`

Next thing we want to do, is to call ReadMultiple and get all customers:

```
MethodInfo readMultiple = service.GetType().GetMethod("ReadMultiple");
object[] customers = (object[])readMultiple.Invoke(service, new object[] { null, null, 0 });
```

You see the picture – get the method info-class based on the type, and invoke the instance based method, specifying the instance and an object\[\] which contains the parameters you want to use.

In the static world this would be `Customer[] customers = service.ReadMultiple(null, null, 0);`

Now, we have an array of objects and the objects are of type Customer – but we don’t know about the customer type – only from reflection, so if we want to write the names of all customers we have to do something like:

```
Type customerType = webServiceAssembly.GetType("Customer");
PropertyInfo no = customerType.GetProperty("No");
PropertyInfo name = customerType.GetProperty("Name");
foreach (object customer in customers)
Console.WriteLine(no.GetValue(customer, new object[] { }) + " " + name.GetValue(customer, new object[] { }));
```

Which in a static implementation would be `foreach(Customer customer in customers) Console.Writeline(customer.No + " " + customer.Name);`

### When to use dynamic Web References?

So, by now you got it – and yes, it is WAY easier to work with static Web References, inserted in the solution and using the strongly typed classes and methods, so when would you use dynamic Web References?

My answer to this is: Whenever you want to make something generic, where you can connect to different pages and/or where you don’t mind that the pages gets customized. In the scenario, where you have a fixed contract for requesting order information from a web service in NAV, there is absolutely no reason to use dynamic web references. In cases where you are connecting to a page based web reference where you have control over the page, it is easier (and probably safer due to type checking) to use the type strong web service access and maybe [using LINQ with NAV Web Services](http://blogs.msdn.com/freddyk/archive/2009/04/21/using-linq-with-nav-web-services.aspx) from my last post.

But… – for something like Edit In Excel, dynamic Web References is a gift – and is really useful. I also think that it could be very useful with [Bugsy’s Sharepoint sample](http://blogs.msdn.com/pchriste/archive/2009/04/14/showing-nav-data-in-sharepoint-using-business-data-catalog-part-2-of-2.aspx) – I need to investigate that…

### Privileges

I was concerned whether stuff like this would require elevated privileges, but it turns out, that as long as the DLL you are creating / calling is going to run in the same context as your application, this doesn’t require anything. I tested this out running as non-administrator with UAC (Windows Vista User Access Control) turned on.

As usual, you can download the DynamicWebReference solution [here](http://www.freddy.dk/DynamicWebReference.zip).

The next thing I will do, is to extract some of the code from the Edit In Excel and create a set of classes, which makes working with dynamic web references easier. This will of course then be rolled back into the Edit In Excel R2.

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
