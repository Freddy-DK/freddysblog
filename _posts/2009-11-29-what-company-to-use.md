---
layout: post
title: "What COMPANY to use?"
date: 2009-11-29 10:22:33
categories: ["Archive"]
tags: ["COMPANY", "Gadget", "NAV 2009 SP1", "URL", "Web Services"]
permalink: /2009/11/29/what-company-to-use/
---

As you know, when creating an application consuming NAV Web Services you need to specify the Company name as part of the URL to the Web Service, but what company should you be using?

Some applications are web front-ends placing data from the web application into NAV. For applications like this you typically would have a config file in which you specify what company things needs to go to. For these applications, this post adds no further value.

Other applications are integration applications, like a lot of the applications you can see on my blog:

-   Search
-   “My” gadgets
-   MAP
-   Edit In Excel

for all of these applications, it really doesn’t make sense to run with a different company than the users default company.

Example – if you search through your NAV data – you really want to search through the data in the active company – not just any company.

Wouldn’t it be nice if you could type in the URL

/Codeunit/Search”>http://localhost:7047/DynamicsNAV/WS/<default>/Codeunit/Search

and then the **<default>** would be replaced by the authenticated users default company – unfortunately this doesn’t work (I have suggested this feature for v7 though:-)). Instead, we have to do the work in the Web Service consuming application. Easiest solution is of course to create a Codeunit with a function, returning the default company of a user, call that and then build your URL for calling the Page / Codeunit web service.

A function like that could be:

```
GetDefaultCOMPANY() : Text[30]
Session.SETRANGE("My Session",TRUE);
Session.FINDFIRST;
WindowsLogin.SETRANGE(ID,Session."User ID");
WindowsLogin.FINDFIRST;
UserPers.SETRANGE("User SID",WindowsLogin.SID);
UserPers.FINDFIRST;
EXIT(UserPers.Company);
```

The problem with this approach is (as you probably already figured out) that every call to a Web Service will require 2 roundtrips instead of one and for Page based Web Service access there really isn’t much you can do better.

For Codeunit based Web Service access you can however avoid a lot of these roundtrips by using a very simple pattern in the way you write your functions. I have rewritten my search method to return a Text\[30\] and start off with the following lines of code:

```
company := GetDefaultCOMPANY();
IF company <> COMPANYNAME THEN
EXIT(company);
```

and the consumer will have to build up the URL for the Web Service in code with whatever company (the first in the list of companies would be just fine), call the web service and if it returns a different company than the one used to invoke the web service, build a new URL and try again.

In the Search gadget this would look like (the lines in Red are the important changes)

```
// the "real" search function
function doSearch(searchstring) {
    specifiedCompany = GetCompany();
usedCompany = specifiedCompany;
if (specifiedCompany == "default") {
if (myCompany == "") {
Companies = GetCompanies();
if (Companies != null)
myCompany = Companies[0].text;
}
usedCompany = myCompany;
}
```

    

```
// Get the URL for the NAV 2009 Search Codeunit
var URL = GetBaseURL() + encodeURIComponent(usedCompany) + "/Codeunit/Search";
```

    

```
// Create XMLHTTP and send SOAP document
xmlhttp = new ActiveXObject("Msxml2.XMLHTTP.6.0");
xmlhttp.open("POST", URL, false, null, null);
xmlhttp.setRequestHeader("Content-Type", "text/xml; charset=utf-8");
xmlhttp.setRequestHeader("SOAPAction", "DoSearch");
xmlhttp.Send('<?xml version="1.0″ encoding="utf-8″?><soap:Envelope xmlns:soap="' + SoapEnvelopeNS + '"><soap:Body><DoSearch xmlns="' + CodeunitSearchNS + '"><searchstring>' + searchstring + '</searchstring><result></result></DoSearch></soap:Body></soap:Envelope>');
```

    

```
// Find the result in the soap result and return the rsult
xmldoc = xmlhttp.ResponseXML;
xmldoc.setProperty('SelectionLanguage', 'XPath');
xmldoc.setProperty('SelectionNamespaces', 'xmlns:soap="' + SoapEnvelopeNS + '" xmlns:tns="' + CodeunitSearchNS + '"');
```

    

```
userCompany = xmldoc.selectSingleNode('//tns:return_value');
myCompany = userCompany.text;
```

    

```
if ((specifiedCompany == "default") && (myCompany != usedCompany)) {
// Default company has changed – research
return doSearch(searchstring);
}
```

   `… do the actual searching`

`}`

In this sample I use three variables:

**specifiedCompany** is the company specified in the config file (_default_ means use users default company)

**usedCompany** is the company used to invoke the last WS method

**myCompany** is my current belief of the users current company, which gets replaced if a method returns a new default company.

Using a pattern like this will help lowering the number of round trips and still allow your consuming application to use the users default company.

This “trick” is only possible in NAV 2009 SP1. NAV 2009 RTM will change the users default company to the company you use to invoke the Web Service with – which again will cause the above function to always return the same company name as the one you invoke the Web Service with.

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
