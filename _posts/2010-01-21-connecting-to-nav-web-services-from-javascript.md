---
layout: post
title: "Connecting to NAV Web Services from Javascript"
date: 2010-01-21 12:57:33
categories: ["Archive", "Web Services"]
tags: ["Javascript", "NAV 2009 SP1", "Web Services", "XML"]
permalink: /2010/01/21/connecting-to-nav-web-services-from-javascript/
---

### Prerequisites

Please read [this post](http://blogs.msdn.com/freddyk/archive/2010/01/19/connecting-to-nav-web-services-from.aspx) to get a brief explanation of the scenario I will implement in Javascript.

BTW. Basic knowledge about Javascript and XML is required to understand the following post:-)

### Browser compatibility

The sample in this post will work with Internet Explorer. I have only tried this with IE8, but according to documentation it should work from IE5 and up.

I actually also tried to download Mozilla FireFox 3.6 and Opera 10.10 – but I failed to make the script work – not due to incompatibility in Javascript, but I simply couldn’t get any of those browsers to authenticate towards NAV WebServices (whether I was running NTLM or SPNEGO – see [this post](http://blogs.msdn.com/freddyk/archive/2010/01/19/connecting-to-nav-web-services-from.aspx)).

This might be due to the XmlHttpRequest being unable to connect to Web Services X-Domain (explained [here](http://www.xml.com/pub/a/2005/11/09/fixing-ajax-xmlhttprequest-considered-harmful.html)) – if you really need to make this work, it seems like a possible solution is to create a proxy (another Web Service in any serviceside language) which is hosted on the same site as the Javascript app and forwards all requests.

IE seems to not care if you have trusted the web site (which is the case here). Script based Web Services access should only be used in your local intranet anyway – you should never try to go across the internet and connect to a NAV server somewhere.

### Other posts with Javascript code

On my blog there are a number of posts, where I use Javascript to connect to Web Services. All the Gadgets and the Virtual Earth web site are using Javascript to connect to Web Services and in all of these I am using the MSXML ActiveXObject and all of the other samples are using CodeUnit access. In this sample, I will show how this can be done without using ActiveX and going towards a page.

### It’s all about XML

Javascript does not natively have support for strongly typed proxy class generation like C#, VB and Java nor does it have support for interpretation of Soap based webservices like PHP – it is all about XML.

In the end all of the other languages ends up creating a XML document (basically just a string), which is send over the wire to the the Web Service host who then replies back with a XML Document (again just a formatted string).

String manipulation is possible in Javascript and Javascript does have an object called XmlHttpRequest which can communicate with a XML Web Service host – we should be good.

The way NAV 2009 (and SP1) handles Web Services is using SOAP. You can read more about the basics here: [http://en.wikipedia.org/wiki/SOAP](http://en.wikipedia.org/wiki/SOAP "http://en.wikipedia.org/wiki/SOAP"). This image describes pretty well how your message is put into a body and inserted into an envelope, which the gets send.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromJavascript_AE5C/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromJavascript_AE5C/image_2.png)

In the scenario I am implementing here, there are 3 roundtrips to Web Services:

1.  Get the Companies supported on a Web Service Listener
2.  Get a specific Customer
3.  Get all Customers matching a specific filter

### Get the Companies supported on a Web Service Listener

In C# we create a reference to the SystemService, which then creates some proxy classes and we just call a method called Companies on these classes.

Underneath that magic, .net will create a XML string that looks much like:

`<Soap:Envelope xmlns:Soap="``http://schemas.xmlsoap.org/soap/envelope/`

```
">
<Soap:Body>
```

```
<Companies xmlns="urn:microsoft-dynamics-schemas/nav/system/">
</Companies>
</Soap:Body>
</Soap:Envelope>
```

As you can see, the only thing I actually send is <Companies /> (with a namespace).

The return value from the Companies method is again

`<Soap:Envelope xmlns:Soap="http://schemas.xmlsoap.org/soap/envelope/&#8221;`

```
>
<Soap:Body>
```

```
<Companies_Result xmlns="urn:microsoft-dynamics-schemas/nav/system/">
<return_value>CRONUS International Ltd.</return_value>
<return_value>TEST</return_value>
</Companies_Result>
</Soap:Body>
</Soap:Envelope>
```

In a later post I will show how you can hook into .net and see what XML actually gets sent and what gets received underneath the nice .net surface.

### Get a specific Customer

The XML for getting a specific customer looks like:

`<Soap:Envelope xmlns:Soap="``http://schemas.xmlsoap.org/soap/envelope/&#8221;`

```
>
<Soap:Body>
<Read xmlns="urn:microsoft-dynamics-schemas/page/customer">
<No>10000</No>
</Read>
</Soap:Body>
</Soap:Envelope>
```

and the return XML from the NAV Customer Page could be:

`<Soap:Envelope xmlns:Soap="``http://schemas.xmlsoap.org/soap/envelope/&#8221;`

```
>
<Soap:Body>
<Read_Result xmlns="urn:microsoft-dynamics-schemas/page/customer">
<Customer>
<Key>… some huge key …</Key>
<No>10000</No>
<Name>The Cannon Group PLC</Name>
<Address>192 Market Square</Address>
<Address_2>Address no. 2</Address_2>
… all the other fields …
</Customer>
</Read_Result>
</Soap:Body>
</Soap:Envelope>
```

I haven’t included all fields – you probably get the picture.

### Get all Customers matching a specific filter

The XML for getting all customers matching a specific filter could be:

`<Soap:Envelope xmlns:Soap="``http://schemas.xmlsoap.org/soap/envelope/&#8221;`

```
>
<Soap:Body>
<ReadMultiple xmlns="urn:microsoft-dynamics-schemas/page/customer">
<filter><Field>Country_Region_Code</Field><Criteria>GB</Criteria></filter>
<filter><Field>Location_Code</Field><Criteria>RED|BLUE</Criteria></filter>
</ReadMultiple>
</Soap:Body>
</Soap:Envelope>
```

and the returned XML something like

`<Soap:Envelope xmlns:Soap="``http://schemas.xmlsoap.org/soap/envelope/&#8221;`

```
>
<Soap:Body>
<ReadMultiple_Result xmlns="urn:microsoft-dynamics-schemas/page/customer">
<ReadMultiple_Result>
<Customer>
… one customer …
</Customer>
<Customer>
… another customer …
</Customer>
<Customer>
… a third customer …
</Customer>
</ReadMultiple_Result>
</ReadMultiple_Result>
</Soap:Body>
</Soap:Envelope>
```

### Enough about the XML – lets see some code

Instead of splitting up the script – I will specify the entire script here and do some explanation beneath.

`<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "`[`http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"`](http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd")

```
>
<html>
<head>
<title></title>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8″ />
```

var baseURL = ‘[`http://localhost:7047/DynamicsNAV/WS/';`](http://localhost:7047/DynamicsNAV/WS/';)  
    

```
var cur;
var SystemServiceURL = baseURL + 'SystemService';
var CustomerPageURL;
```

    `var SoapEnvelopeNS = '`[`http://schemas.xmlsoap.org/soap/envelope/';`](http://schemas.xmlsoap.org/soap/envelope/';)  
    

```
var SystemServiceNS = 'urn:microsoft-dynamics-schemas/nav/system/';
var CustomerPageNS = 'urn:microsoft-dynamics-schemas/page/customer';
```

    

```
// Function to Invoke a NAV WebService and return data from a specific Tag in the responseXML
function InvokeNavWS(URL, method, nameSpace, returnTag, parameters) {
var result = null;
try {
var xmlhttp;
if (window.XMLHttpRequest) {// code for IE7+, Firefox, Chrome, Opera, Safari
xmlhttp = new XMLHttpRequest();
}
else {// code for IE6, IE5
xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
```

            

```
var request = " +
" +
" +
parameters +
" +
" +
";
```

            

```
// Use Post and non-async
xmlhttp.open('POST', URL, false);
xmlhttp.setRequestHeader('Content-type', 'text/xml; charset=utf-8');
xmlhttp.setRequestHeader('Content-length', request.length);
xmlhttp.setRequestHeader('SOAPAction', method);
```

            

```
// Setup event handler when readystate changes
xmlhttp.onreadystatechange = function() {
if (xmlhttp.readyState == 4) {
if (xmlhttp.status == 200) {
xmldoc = xmlhttp.responseXML;
xmldoc.setProperty('SelectionLanguage', 'XPath');
xmldoc.setProperty('SelectionNamespaces', 'xmlns:tns="' + nameSpace + '"');
result = xmldoc.selectNodes('//tns:' + returnTag);
}
}
}
```

            

```
// Send request will return when event has fired with readyState 4
xmlhttp.send(request);
}
catch (e) {
}
return result;
}
```

    

```
// Get the Company list
function SystemService_Companies() {
return InvokeNavWS(SystemServiceURL, 'Companies', SystemServiceNS, 'return_value', ");
}
```

    

```
function CustomerPage_Read(no) {
return InvokeNavWS(CustomerPageURL, 'Read', CustomerPageNS, 'Customer',
" + no + ");
}
```

    

```
function CustomerPage_ReadMultiple(filters) {
return InvokeNavWS(CustomerPageURL, 'ReadMultiple', CustomerPageNS, 'Customer', filters);
}
```

```
</head>
<body>
```

    

```
var companies = SystemService_Companies();
document.writeln('Companies:
');
for (var i = 0; i         document.writeln(companies[i].text + '
');
}
cur = companies[0].text;
```

    

```
CustomerPageURL = baseURL + encodeURIComponent(cur) + '/Page/Customer';
document.writeln('
URL of Customer Page: ' + CustomerPageURL + '
');
```

    

```
var Customer10000 = CustomerPage_Read('10000');
document.writeln('
Name of Customer 10000: ' +
Customer10000[0].childNodes[2].firstChild.nodeValue + '
');
```

    

```
document.writeln('
Customers in GB served by RED or BLUE warehouse:
');
var Customers = CustomerPage_ReadMultiple(
'Country_Region_CodeGB'+
```

                    `'`

```
Location_CodeRED|BLUE');
for (i = 0; i         document.writeln(Customers[i].childNodes[2].firstChild.nodeValue + '
');
```

    

```
document.writeln('
THE END');
```

```
</body>
</html>
```

This is the entire Default.htm file.

Most of the “magic” happens inside InvokeNavWS, which really just builds up the XML document based on the method, the namespace and the parameters for the method. This XML document is then sent to the URL and the XML document we get back is read into an XmlDoc and we use XPath to get the return value (the ReturnTag specifies which tags we are interested in).

On top of this method I have created some high level functions so that we can call CustomerPage\_Read(‘10000’) and get a Customer back.

Note that Read and ReadMultiple returns a XML NodeList – and the childNodes under the top level nodes are the fields and under the fields you get the field value by saying firstChild.nodeValue.

In this sample I have hardcoded the Name field to be in childNodes\[2\], this is probably not the way to get the name – but then again, this is only to show how to get a connection up running.

The output of this on my installation looks like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromJavascript_AE5C/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromJavascript_AE5C/image_4.png)

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
