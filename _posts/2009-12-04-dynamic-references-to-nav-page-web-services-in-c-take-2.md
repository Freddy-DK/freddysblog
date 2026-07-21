---
layout: post
title: "Dynamic references to NAV Page Web Services in C# – take 2"
date: 2009-12-04 13:56:26
categories: ["Archive"]
tags: ["C#", "Dynamic", "Excel", "NAV 2009 SP1", "Reflection", "Web Services"]
permalink: /2009/12/04/dynamic-references-to-nav-page-web-services-in-c-take-2/
---

In [this](http://blogs.msdn.com/freddyk/archive/2009/04/27/dynamic-references-to-nav-page-web-services-in-c.aspx) post from April, I explained how to make dynamic references to page based Web Services, but the post really left the developer with a lot of manual work to do using reflection.

So – I thought – why not create a couple of helper classes which makes it easier.

Basically I have created a generic NAVPageServiceHelper class, which encapsulates all the heavy lifting of reflection and leaves the developer with a set of higher level classes he can use.

The service helper will have a collection of classes explaining various information about the fields and has methods for getting or setting the value (and setting the corresponding \_Specified automatically as well).

The primary reason for making this is of course to make Edit In Excel bind to any page without changing anything, but the method can be used in a lot of other scenarios.

### 2 projects: NAVPageFieldInfo and NAVPageDynamicWebReference

I split the PageServiceHelper and the PageFieldInfo into two seperate projects. NAVPageFieldInfo just contains the FieldInfo classes for all the supported field types and a collection class.

**NAVPageFieldInfo** is the abstract base class  
**BooleanFieldInfo** is the field info class for a boolean field  
**OptionFieldInfo** is the field info class for an option field  
**IntFieldInfo** is the field info for…

You get it – all in all, the following types are supported:

**String, Decimal, DateTime, Int, Option, Boolean**

Furthermore, there is a class called **NAVFields**, which derives from **List<NAVFieldInfo>**, for keeping a collection of the fields.

NAVFields has a method called **PopulateFieldsCollection**, which takes an object type and a fields enum type and based on this, instantiates all the NAVFieldInfo classes – let’s look at the code.

/// <summary>  
/// Populate Fields Collection with NAVPageFieldInfo for all properties in the record  
/// Should works with any NAV 2009 Page exposed as WebService  
/// </summary>  
/// <param name=”objType”>Type of Object (typeof(Customer), typeof(Vendor), …)</param>  
/// <param name=”fieldsType”>Type of the Enum holding the property names</param>  
private void PopulateFieldsCollection(Type objType, Type fieldsType)  
{  
// Key property is not part of the Enum  
// Add it manually as the first field  
AddField(“Key”, objType);

    // Run through the enum and add all fields  
foreach (string field in Enum.GetNames(fieldsType))  
{  
AddField(field, objType);  
}  
}

/// <summary>  
/// Add a NAVPageFieldInfo for a field to the fields collection  
/// </summary>  
/// <param name=”field”>Field name</param>  
/// <param name=”objType”>Type of Object in which the field is (typeof(Customer), typeof(Vendor), …)</param>  
private void AddField(string field, Type objType)  
{  
field = VSName(field);  
PropertyInfo pi = objType.GetProperty(field, System.Reflection.BindingFlags.Public | System.Reflection.BindingFlags.Instance);  
if (pi != null)  
{  
NAVPageFieldInfo nfi = NAVPageFieldInfo.CreateNAVFieldInfo(objType, field, pi, objType.Namespace);  
if (nfi != null)  
{  
// If we encounter unknown Field Types, they are just ignored  
this.Add(nfi);  
}  
}  
}

As you can see, the AddField method calls a static method on NAVPageFieldInfo to get a FieldInfo class of the right type created. That method looks like:

/// <summary>  
/// Create a NAVPageFieldInfo object for a specific field  
/// </summary>  
/// <param name=”field”>Name of the property</param>  
/// <param name=”pi”>PropertyInfo for the property on the record object</param>  
/// <param name=”ns”>Namespace for the record object (namespace for the added WebServices proxy class)</param>  
/// <returns>NAVPageFieldInfo or null if the type isn’t supported</returns>  
public static NAVPageFieldInfo CreateNAVFieldInfo(Type objType, string field, System.Reflection.PropertyInfo pi, string ns)  
{  
if (pi.PropertyType == typeof(string))  
{  
// String Property – is it the KeyField  
if (field == “Key”)  
return new KeyFieldInfo(field, pi, null);  
else  
return new StringFieldInfo(field, pi, null);  
}  
PropertyInfo piSpecified = objType.GetProperty(field + “Specified”, System.Reflection.BindingFlags.Public | System.Reflection.BindingFlags.Instance);  
if (pi.PropertyType == typeof(decimal))  
{  
// Decimal Property  
return new DecimalFieldInfo(field, pi, piSpecified);  
}  
if (pi.PropertyType == typeof(int))  
{  
// Integer Property  
return new IntFieldInfo(field, pi, piSpecified);  
}  
if (pi.PropertyType == typeof(bool))  
{  
// Boolean Property  
return new BooleanFieldInfo(field, pi, piSpecified);  
}  
if (pi.PropertyType == typeof(DateTime))  
{  
// DateTime Property  
return new DateTimeFieldInfo(field, pi, piSpecified);  
}  
if (pi.PropertyType.Namespace == ns)  
{  
// Other Property Types, in the same namespace as the object  
// These are enum’s – set up restrictions on OptionFields  
return new OptionFieldInfo(field, pi, piSpecified, Enum.GetNames(pi.PropertyType));  
}  
return null;  
}

No more magic!

And actually the constructor for NAVFields – takes the object type and the field type as parameters for the constructor:

public NAVFields(Type objType, Type fieldsType)  
: base()  
{  
this.PopulateFieldsCollection(objType, fieldsType);  
}

Meaning that all it takes to utilize the NAVFieldInfo subsystem is instantiating the NAVFields class, which doesn’t necessarily need a dynamic web reference helper, but could also be instantiated through:

NAVFields fields = new NAVFields(typeof(Customer), typeof(Customer\_Fields));

If you have some code, which needs to access data loosely coupled, NAVFields is a great way to get going.

The other project is the NAVDynamicPageWebReference – which really is a combination of the Dynamic Web References post from April and a Page Service Helper class.

The way you get a reference to the Dynamic Web Reference is much like in the post from April:

Assembly customerPageRef = NAVPageDynamicWebReference.BuildAssemblyFromWSDL(  
new Uri(“[http://localhost:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd./Page/Customer”)](http://localhost:7047/DynamicsNAV/WS/CRONUS%20International%20Ltd./Page/Customer"\)), 5000);

Based on this, you now instantiate the Service Helper with the Assembly and the name of the Page:

NAVPageServiceHelper serviceHelper = new NAVPageServiceHelper(customerPageRef, “Customer”);

### Using the Page Service Helper

The Page Service Helper then uses NAVFields so that you can do stuff like:

foreach (NAVPageFieldInfo fi in serviceHelper.Fields)  
Console.WriteLine(fi.field + ” ” + fi.fieldType.Name);

The properties currently in the Service Helper are:

**Fields** is a NAVFields (a list of NAVFieldInfo derived classes)  
**PrimaryKeyFields** is an array NAVFieldInfo classes (from Fields) which makes out the primary key of the record  
**GetFieldsType** returns the type of the Field enumeration  
**GetObjectType** returns the type of the records handles through this Service  
**ReadMultiple** reads the records matching an array of filters (calls the ReadMultiple on the Service)  
**CreateFilter** creates a filter spec based on a field and a criteria  
**Read** reads a record based on a primary key (creates a filter spec for the primary key and calls ReadMultiple)  
**Update** updates a record (calls the Update method on the Service)  
**Create** creates a record (calls the Create method on the Service)  
**Delete** deletes a record matching a key  
**ReRead** reads an updated instance of a record (calls the Read method on the Service with the key fields)  
**IsUpdated** checks whether the record is updated (calls the IsUpdated method on the Service)  
**GetFiltersFromView** creates an array of filter specs based on a view (from GETVIEW in AL Code)

An example of how to read customer 10000 and print the name would be:

object cust = serviceHelper.Read(“10000”);  
Console.WriteLine(serviceHelper.Fields\[“Name”\].GetValue(cust));

Note, that you find the Field – and on the field, you call GetValue and specify the record instance.

If you need to Display the name of all customers with location code yellow you would write

ArrayList filters = new ArrayList();  
filters.Add(serviceHelper.CreateFilter(“Location\_Code”, “Yellow”));  
object\[\] customers = serviceHelper.ReadMultiple(filters);  
foreach (object customer in customers)  
Console.WriteLine(serviceHelper.Fields\[“Name”\].GetValue(customer));

Or you could create a Customer by writing

object newcust = System.Activator.CreateInstance(serviceHelper.GetObjectType());  
serviceHelper.Fields\[“Name”\].SetValue(newcust, “Freddy Kristiansen”, DBNull.Value);  
newcust = serviceHelper.Create(newcust);  
Console.WriteLine(serviceHelper.Fields\[“No”\].GetValue(newcust));

As mentioned before, the Page Service Helper was primarily created for making Edit In Excel and other projects, where you are using loosely coupled Page Web Service Access.

For a lot of other usages, this is overkill and you should rather use Web References in Visual Studio and have a strongly typed contract with the Web Service.

You can download the projects and the small test program [here](http://www.freddy.dk/DynamicPageWebService.zip).

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
