---
layout: post
title: "Connecting to NAV Web Services from VBScript"
date: 2010-01-29 10:52:43
categories: ["Archive", "Web Services"]
tags: ["NAV 2009 SP1", "VBScript", "Web Services", "XML"]
permalink: /2010/01/29/connecting-to-nav-web-services-from-vbscript/
---

The Connecting to NAV Web Services series is coming to an end. I think I have covered the majority of platforms from which you would like to connect and use NAV Web Services – some things are easy and some things are a little harder. I did not cover Flash nor did i cover things like the iPhone or iPod Touch, primarily because I don’t think the demand is there. If I have forgotten any platform/language please let me know and if the demand is there I might make something work.

# Why VBScript?

Including VBScript makes it possible to do Web Services from scripting like login, shutdown and maintenance scripts. I know that VBScript can also be used from certain browsers but the real idea behind including VBScript here is to enable command line scripts.

Please read [this post](/2010/01/19/connecting-to-nav-web-services-from/) to get a brief explanation of the scenario I will implement in VBScript.

Please read [this post](/2010/01/21/connecting-to-nav-web-services-from-javascript/) about how to connect to NAV Web Services from Javascript to get an overall explanation about XML Web Services and how to do things without having proxy classes generated etc.

The primary difference between Javascript and VBcript is actually syntax – most of the things are done in a similar way.

# The Script file

I created a file called _TestWS.vbs_ and the code to implement the scenario looks like:

function InvokeNavWS(URL, method, nameSpace, returnTag, parameters) 
    Set xmlhttp = CreateObject("MSXML2.XMLHTTP")
    request = "<Soap:Envelope xmlns:Soap="""+SoapEnvelopeNS+"""><Soap:Body><"+method+" xmlns="""+nameSpace+""">"+parameters+"</"+method+"></Soap:Body></Soap:Envelope>"

    ' Use Post and non-async 
    xmlhttp.open "POST", URL, false 
    xmlhttp.setRequestHeader "Content-type", "text/xml; charset=utf-8" 
    xmlhttp.setRequestHeader "Content-length", len(request) 
    xmlhttp.setRequestHeader "SOAPAction", method

    ' send request synchronously 
    xmlhttp.send request

    ' 200 == OK 
    if xmlhttp.status = 200 then 
        Set xmldoc = xmlhttp.responseXML 
        xmldoc.setProperty "SelectionLanguage", "XPath" 
        xmldoc.setProperty "SelectionNamespaces", "xmlns:tns="""+nameSpace+"""" 
        Set InvokeNavWS = xmldoc.selectNodes("//tns:"+returnTag) 
    else 
        Set InvokeNavWS = nothing 
    end if

end function

' Get the Company list 
function SystemService\_Companies() 
    Set SystemService\_Companies = InvokeNavWS(systemServiceURL, "Companies", systemServiceNS, "return\_value", "") 
end function

' Read one customer 
function CustomerPage\_Read(no) 
    Set CustomerPage\_Read = InvokeNavWS(CustomerPageURL, "Read", CustomerPageNS, "Customer", "<No>"+no+"</No>") 
end function

' Read Customers 
function CustomerPage\_ReadMultiple(filters) 
    Set CustomerPage\_ReadMultiple = InvokeNavWS(CustomerPageURL, "ReadMultiple", CustomerPageNS, "Customer", filters) 
end function

sub display(str) 
    WScript.echo str 
end sub

baseURL = "http://localhost:7047/DynamicsNAV/WS/" 
systemServiceURL = baseURL + "SystemService"

soapEnvelopeNS = "http://schemas.xmlsoap.org/soap/envelope/" 
systemServiceNS = "urn:microsoft-dynamics-schemas/nav/system/" 
customerPageNS = "urn:microsoft-dynamics-schemas/page/customer"

Set Companies = SystemService\_Companies() 
display "Companies:" 
for i = 0 to Companies.length-1 
    display Companies(i).text 
next 
cur = Companies(0).text

customerPageURL = baseURL+escape(cur)+"/Page/Customer" 
display "" 
display "URL of Customer Page:" 
display customerPageURL

Set Customer10000 = CustomerPage\_Read("10000") 
display "" 
display "Name of Customer 10000: "+Customer10000(0).childNodes(2).firstChild.nodeValue

Set Customers = CustomerPage\_ReadMultiple("<filter><Field>Country\_Region\_Code</Field><Criteria>GB</Criteria></filter><filter><Field>Location\_Code</Field><Criteria>RED|BLUE</Criteria></filter>") 
display "" 
display "Customers in GB served by RED or BLUE warehouse:" 
for i = 0 to Customers.length-1 
    display Customers(i).childNodes(2).firstChild.nodeValue 
next

display "" 
display "THE END"

The similarity to the Javascript sample is huge (since I am using the same object model), the biggest differences are:

-   The way to encode a URL component in VBScript is by calling _escape()_ – note that escape also exists in Javascript and .net – but there it works **differently**.
-   Displaying things are done using WScript.echo – this will result in a messagebox if you are using WScript to run the script and a commandline output if you are using CScript (I use CScript)

# Running the script

Using the command:

C:\\users\\freddyk>SCript /nologo testws.vbs

I get the following:

![image\_2](/assets/images/2010/connecting-to-nav-web-services-from-vbscript/image_2.png)

and of course you can now do things as redirecting the output to a file and typing or searching in that file:

![image\_4 (1)](/assets/images/2010/connecting-to-nav-web-services-from-vbscript/image_4-1.png)

This is something network administrators are experts in doing – I won’t try to compete in any way.

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
