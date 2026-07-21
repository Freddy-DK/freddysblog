---
layout: post
title: "Connecting to NAV Web Services from Microsoft Dynamics NAV 2009 SP1"
date: 2010-01-21 21:57:22
categories: ["Archive", "Web Services"]
tags: ["NAV 2009 SP1", "URL", "Web Services", "XML"]
permalink: /2010/01/21/connecting-to-nav-web-services-from-microsoft-dynamics-nav-2009-sp1/
---

Please read [this post](http://blogs.msdn.com/freddyk/archive/2010/01/19/connecting-to-nav-web-services-from.aspx) to get a brief explanation of the scenario I will implement in Microsoft Dynamics NAV 2009 SP1. Please also read [this post](http://blogs.msdn.com/freddyk/archive/2010/01/21/connecting-to-nav-web-services-from-javascript.aspx) in order to understand how Web Services works using pure XML and no fancy objects.

Like Javascript, NAV 2009 SP1 does not natively have support for consuming Web Services. It does however have support for both Client and Server side COM automation and XmlHttp (which is compatible with XmlHttpRequest which we used in the Javascript sample [here](http://blogs.msdn.com/freddyk/archive/2010/01/21/connecting-to-nav-web-services-from-javascript.aspx)) is available in Microsoft XML.

### Client or Serverside

When running XmlHttp under the RoleTailored Client we have to determine whether we want to run XmlHttp serverside or clientside. My immediate selection was Serverside (why in earth do this Clientside) until I tried it out and found that my server was not allowed to impersonate me against a web services which again needs to impersonate my against the database.

The double hub issue becomes a triple hub issue and now it suddenly becomes clear that XmlHttp in this sample of course needs to run clientside:-)

### Compatibility

The sample below will run both in the RoleTailored Client and in the Classic Client.

### InvokeNavWS

As in the Javascript example, I will create a function called InvokeNavWS – and in this function I will do the actual Web Service method invocation. In Javascript we setup an event to be called when the send method was done and as you might know, this is not doable on the Roletailored Client.

Fortunately, we are using synchronous web services, meaning that it is actually not necessary to setup this event. We can just check the status when send returns.

xmlhttp.send allows you to send either a string or a XML Document. Having in mind that a string in NAV Classic is max. 1024 characters, I decided to go with a XML Document. In the RoleTailored Client I could have used BigText, but that doesn’t work in Classic.

Creating a XML Document will take slightly more time than building up a large string, but it is the safest way to go. Start by adding an Envelope, a body, a method and then transfer the parameter nodes one by one (there might be smarter ways to do this:-)

The return value is always a nodeList and we only look at the responseXML property of the xmlhttp (which is an XML document).

The Code for InvokeNavWS looks like this:

**InvokeNavWS(URL : Text\[250\];method : Text\[20\];nameSpace : Text\[80\];returnTag : Text\[20\];parameters : Text\[1024\];VAR nodeList : Automation “‘Microsoft XML, v6.0’.IXMLDOMNodeList”) result : Boolean  
**result := FALSE;  
// Create XML Document  
CREATE(xmldoc,TRUE,TRUE);  
// Create SOAP Envelope  
soapEnvelope := xmldoc.createElement(‘Soap:Envelope’);  
soapEnvelope.setAttribute(‘xmlns:Soap’, ‘[http://schemas.xmlsoap.org/soap/envelope/’);](http://schemas.xmlsoap.org/soap/envelope/'\);)  
xmldoc.appendChild(soapEnvelope);  
// Create SOAP Body  
soapBody := xmldoc.createElement(‘Soap:Body’);  
soapEnvelope.appendChild(soapBody);  
// Create Method Element  
soapMethod := xmldoc.createElement(method);  
soapMethod.setAttribute(‘xmlns’, nameSpace);  
soapBody.appendChild(soapMethod);  
// Transfer parameters by loading them into a XML Document and move them  
CREATE(parametersXmlDoc,TRUE,TRUE);  
parametersXmlDoc.loadXML(‘<parameters>’+parameters+'</parameters>’);  
IF parametersXmlDoc.firstChild.hasChildNodes THEN  
BEGIN  
WHILE parametersXmlDoc.firstChild.childNodes.length>0 DO  
BEGIN  
node := parametersXmlDoc.firstChild.firstChild;  
node := parametersXmlDoc.firstChild.removeChild(node);  
soapMethod.appendChild(node);  
END;  
END;  
// Create XMLHTTP and SEND  
CREATE(xmlhttp, TRUE, TRUE);  
xmlhttp.open(‘POST’, URL, FALSE);  
xmlhttp.setRequestHeader(‘Content-type’, ‘text/xml; charset=utf-8’);  
xmlhttp.setRequestHeader(‘SOAPAction’, method);  
xmlhttp.send(xmldoc);  
// If status is OK – Get Result XML  
IF xmlhttp.status=200 THEN  
BEGIN  
xmldoc := xmlhttp.responseXML;  
xmldoc.setProperty(‘SelectionLanguage’,’XPath’);  
xmldoc.setProperty(‘SelectionNamespaces’,’xmlns:tns=”‘+nameSpace+'”‘);  
nodeList := xmldoc.selectNodes(‘//tns:’+returnTag);  
result := TRUE;  
END;

and the local variables for InvokeNavWS are

Name              DataType      Subtype                                 Length  
xmlhttp           Automation    ‘Microsoft XML, v6.0’.XMLHTTP  
xmldoc            Automation    ‘Microsoft XML, v6.0’.DOMDocument  
soapEnvelope      Automation    ‘Microsoft XML, v6.0’.IXMLDOMElement  
soapBody          Automation    ‘Microsoft XML, v6.0’.IXMLDOMElement  
soapMethod        Automation    ‘Microsoft XML, v6.0’.IXMLDOMElement  
node              Automation    ‘Microsoft XML, v6.0’.IXMLDOMNode  
parametersXmlDoc  Automation    ‘Microsoft XML, v6.0’.DOMDocument   

As in the Javascript sample I have create a couple of “high” level functions for easier access:

**SystemService\_Companies(VAR nodeList : Automation “‘Microsoft XML, v6.0’.IXMLDOMNodeList”) result : Boolean  
**result := InvokeNavWS(systemServiceURL, ‘Companies’, SystemServiceNS, ‘return\_value’, ”, nodeList);

**CustomerPage\_Read(No : Text\[20\];VAR nodeList : Automation “‘Microsoft XML, v6.0’.IXMLDOMNodeList”) result : Boolean  
**result := InvokeNavWS(customerPageURL, ‘Read’, CustomerServiceNS, ‘Customer’, ‘<No>’+No+'</No>’, nodeList);

**CustomerPage\_ReadMultiple(filters : Text\[1024\];VAR nodeList : Automation “‘Microsoft XML, v6.0’.IXMLDOMNodeList”) result : Boolean  
**result := InvokeNavWS(customerPageURL, ‘ReadMultiple’, CustomerServiceNS, ‘Customer’, filters, nodeList);

### The “main” program

**OnRun()**  
baseURL := ‘[http://localhost:7047/DynamicsNAV/WS/’;](http://localhost:7047/DynamicsNAV/WS/';)  
systemServiceURL := baseURL + ‘SystemService’;  
SoapEnvelopeNS := ‘[http://schemas.xmlsoap.org/soap/envelope/’;](http://schemas.xmlsoap.org/soap/envelope/';)  
SystemServiceNS := ‘urn:microsoft-dynamics-schemas/nav/system/’;  
CustomerServiceNS := ‘urn:microsoft-dynamics-schemas/page/customer’;

CLEAR(nodeList);  
IF SystemService\_Companies(nodeList) THEN  
BEGIN  
DISPLAY(‘Companies:’);  
FOR i:=1 TO nodeList.length DO  
BEGIN  
node := nodeList.item(i-1);  
DISPLAY(node.text);  
IF i=1 THEN cur := node.text;  
END;

  customerPageURL := baseURL + EncodeURIComponent(cur) + ‘/Page/Customer’;  
DISPLAY(‘URL of Customer Page: ‘+ customerPageURL);

  IF CustomerPage\_Read(‘10000’, nodeList) THEN  
BEGIN  
DISPLAY(‘Name of Customer 10000: ‘ + nodeList.item(0).childNodes.item(2).firstChild.text);  
END;

  IF CustomerPage\_ReadMultiple(‘<filter><Field>Country\_Region\_Code</Field><Criteria>GB</Criteria></filter>’+  
‘<filter><Field>Location\_Code</Field><Criteria>RED|BLUE</Criteria></filter>’, nodeList) THEN  
BEGIN  
DISPLAY(‘Customers in GB served by RED or BLUE warehouse:’);  
FOR i:=1 TO nodeList.length DO  
BEGIN  
node := nodeList.item(i-1);  
DISPLAY(node.childNodes.item(2).firstChild.text);  
END;  
END;

  DISPLAY(‘THE END’);

END;

with the following local variables:

Name       DataType      Subtype                                 Length  
nodeList   Automation    ‘Microsoft XML, v6.0’.IXMLDOMNodeList  
node       Automation    ‘Microsoft XML, v6.0’.IXMLDOMNode  
i          Integer       

As it was the case in the Javascript sample I am using simple xml nodelist code to navigate and display various values. baseURL, cur, SystemServiceURL etc. are all global Text variables used as constants.

DISPLAY points to a function that just does a IF CONFIRM(s) THEN ; to display where we are and running this on the RoleTailored Client will display the following Confirm Dialogs:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_2.png) [![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_4.png) [![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_6.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_8.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_4.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_10.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_5.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_12.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_6.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_14.png) [![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_7.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_16.png) [![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_8.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_18.png) [![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_9.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_20.png)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_thumb_10.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromMicrosoftD_E44B/image_22.png)

Note that the URL of the Customer Page is different from all the other examples. This is because NAV doesn’t have a way of Encoding an URL, so I have to do the company name encoding myself and when I encode a company name, I just encode all characters, that works perfectly:

**EncodeURIComponent(uri : Text\[80\]) encodedUri : Text\[240\]  
**// No URI Encoding in NAV – we do it ourself…  
HexDigits := ‘0123456789ABCDEF’;  
encodedUri := ”;  
FOR i:=1 TO STRLEN(uri) DO  
BEGIN  
b := uri\[i\];  
encodedUri := encodedUri + ‘%  ‘;  
encodedUri\[STRLEN(encodedUri)-1\] := HexDigits\[(b DIV 16)+1\];  
encodedUri\[STRLEN(encodedUri)\] := HexDigits\[(b MOD 16)+1\];  
END;

(Again, there might be smarter ways to do this – I just haven’t found it).

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
