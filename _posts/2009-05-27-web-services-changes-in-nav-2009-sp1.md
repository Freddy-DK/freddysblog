---
layout: post
title: "Web Services changes in NAV 2009 SP1"
date: 2009-05-27 12:52:32
categories: ["Archive", "Web Services"]
tags: ["NAV 2009", "NAV 2009 SP1", "Web Services", "XML"]
permalink: /2009/05/27/web-services-changes-in-nav-2009-sp1/
---

NAV 2009 SP1 is being released later this year, so why write about it now?

The main reason is, that NAV 2009 SP1 comes out with a couple of changes, you might want to take into consideration when writing towards NAV 2009 Web Services.

Except for some performance enhancements, the major changes are:

### New standard encoding of the Company name

I think one of the questions that has been asked the most is, how does my company name look in the URL: _http://server:7047/DynamicsNAV/WS/<company>/_…

we all know that “CRONUS International Ltd.” becomes “CRONUS\_International\_Ltd”. Lesser known is it that “CRONUS USA, Inc.” becomes “CROUNS\_USA\_x002C\_\_Inc” and there are a lot of special cases in this encoding algorithm.

In NAV 2009 SP1 we change to a standard Uri EscapeDataString function, meaning that

_CRONUS International Ltd._ becomes _CRONUS%20International%20Ltd._

_CRONUS USA, Inc._ becomes _CRONUS%20USA%2C%20Inc._

and

_CRONUS+,æøå_ becomes _CRONUS%2B%2C%C3%A6%C3%B8%C3%A5_

fact is that you actually just can type in the Company name in the URL with the special characters in the browser and it will (in most cases) figure out to select the right company, even in Visual Studio when making your Web Reference (this won’t work if you have / or ? in the company name).

btw you can escape all the characters if you wish to

_[http://localhost:7047/DynamicsNAV/WS/%43RONUS%20International%20Ltd./Services](http://localhost:7047/DynamicsNAV/WS/%43RONUS%20International%20Ltd./Services)_

is a perfectly good company name – if you prefer Hex over characters.

This change has also affected the return values of the Companies function in SystemService – it now returns the un-encoded company names (= clear text). You can not any longer use the output from the companies function to build your URL – you need to escape the name.

Note: There is no backwards compatibility, trying to access webservices with a URL from NAV 2009 will fail, you need to change the company name encoding.

### Schema changes in ReadMultiple and UpdateMultiple

Microsoft Infopath couldn’t modify multiple records using the NAV2009 page based Web Services due to a schema incompatibility. In NAV 2009 SP1 the schema changes for these methods. If you are using proxy based Web Service access (the add service or web reference in Visual Studio) you should just update the reference. If you are using XML Web Services you might have to modify the code used to parse the XML.

I will of course modify the samples on my blog where I use XPath to query the XML.

### Updating records in Page based web services only updates the fields that you actually changed

The basics of XML Web Services is, that you send an XML document to a WebServices telling what you want to change. Visual Studio makes it easy to create a reference to a Web Service and get Strongly typed access to f.ex. Customers and Sales Orders through pages.

But how do we tell Web Services which fields actually changed?

For this, Visual Studio autogenerates a <field>Specified boolean property for all non-string fields from NAV and we will change ALL the fields, where <field>Specified is true or where a string is not NULL – NULL in a string value doesn’t mean clear the field, it means don’t update the field.

If you want to clear a field, set the value to String.Empty (“”).

In some cases this have caused problems. Primarily because when you read a customer record in order to change his name, it comes back from the Get function with all <field>Specified set to TRUE and all string fields have content. Changing the Name of a customer – writes the NAME and since the SEARCHNAME is included in the data sent to Web Services that gets updated as well (meaning that NAME and SEARCHNAME could be out of sync).

In NAV 2009 SP1 that has changed. Visual Studio of course still uses <field>Specified and string field <> NULL to determine what comes over the wire, but on the NAV side we only persist what you actually changed, so in NAV 2009 SP1 you can do:

Customer customer = custService.Read(“10000”);  
customer.Name = “The Cannon Group, Inc.”;  
custService.Update(ref customer);

and it will only update the name of the Customer. In NAV 2009 you would have to either set all the other fields in the customer to NULL or <field>Specified to false in order to get the same behavior – OR you could do like this:

Customer readCustomer = custService.Read(“10000”);  
Customer updateCustomer = new Customer();  
updateCustomer.Key = readCustomer.Key;  
updateCustomer.Name = “The Cannon Group, Inc.”;  
custService.Update(ref updateCustomer);

Which also will update only the name (just a small trick, instantiating a new Customer() will have all string fields set to NULL and <field>Specified for other fields set to false – and now we can just set the fields we want to change. Remember setting <field>Specified to true for all non-string fields.).

Note that this will of course work in SP1 as well and the advantage here is that you actually only send the new customer name over the wire to the Web Service.

### Changes to how you Create and Updating Sales Orders through a page based Web Service

Actually the way you need to work with Sales Orders in NAV 2009 SP1 through a page based Web Service will also work in NAV 2009 – but the other way around is a problem. In NAV 2009 you could create a sales order with lines with just one call to Web Services, but in reality this didn’t work, you need to do this with a couple of roundtrips.

This is because application code (CurrPage.UPDATE) relies on one kind of transactional behavior (the new order is inserted and committed before any validation trigger starts), but Web Services enforce a different kind (every call to server is wrapped into one atomic transaction that is either committed or rolled back entirely – meaning that insert is not committed until all validation triggers passed).

I will create a special post on how to work with Sales Orders from Web Services – and try to show a way, which works for NAV 2009 SP1 (the same method will work for NAV 2009 as well – so you should consider this early).

### Web Services doesn’t change the users default company

A Web Service consumer application would change the users default company in NAV 2009, but since Web Services doesn’t really use the notion of a default company for anything this seemed confusing – and made it impossible for a web service consumer application to call a web service method to request the users default company. In NAV 2009 SP1 – invoking a Web Service method does not change the default company for the user.

### Blob fields in Page based Web Services are ignored

In NAV 2009 you couldn’t have a BLOB field on a page (image or whatever), which you exposed as a Web Service.

In NAV 2009 SP1, this has changed. This doesn’t mean that NAV transfers the content of the BLOB to the web service consumer – the field is just ignored.

If you want access to the value of the Blob you will need to write some code, which you can read something about here [:](http://blogs.msdn.com/freddyk/archive/2008/11/04/transferring-data-to-from-com-automation-objects-and-webservices.aspx "http://blogs.msdn.com/freddyk/archive/2008/11/04/transferring-data-to-from-com-automation-objects-and-webservices.aspx")

[http://blogs.msdn.com/freddyk/archive/2008/11/04/transferring-data-to-from-com-automation-objects-and-webservices.aspx](http://blogs.msdn.com/freddyk/archive/2008/11/04/transferring-data-to-from-com-automation-objects-and-webservices.aspx)

### Codeunits ignores methods with unsupported parameters

Like Pages with unsupported fields (BLOB), also codeunits can be exposed as Web Services even though they contain methods that use parameter types, that are not supported by Web Services. This could be streams, automation, blobs and more.

### In SP1 you can connect to NAV Web Services from both PHP and Java

I won’t cover the details in this post, but it should be clear that NAV WebServices are accessible from PHP and Java come SP1. As soon as I have a build, which supports this – I will write a couple of posts on how to do this.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
