---
layout: post
title: "Search in NAV 2009 – Part 1 (out of 3)"
date: 2008-11-23 07:03:11
categories: ["Archive"]
tags: ["BigText", "Bookmark", "C#", "Javascript", "NAV 2009", "Search", "Service Tier", "URL", "Web Services"]
permalink: /2008/11/23/search-in-nav-2009-part-1-out-of-3/
---

During the partner keynote and during a couple of the other presentations, we showed a small application, which was able to search in NAV 2009 through multiple tables, display a result set and allow people to drill into task pages in NAV 2009 from the search result window.

During the next 3 posts, I will explain how this demo is done and make it available for download. The sample comes with absolutely no warranty, but you can download it and see how things are done and reuse pieces of the sample or the full sample.

In the first part I will describe how the search functionality is done inside NAV, how to expose this as a Web Service and search from Microsoft Infopath, getting an XML result set back.

In the second part, I will describe how to create a small Windows application, which connects to the Web Service from part 1 and uses XSLT to transform the XML to a nice HTML document with links back into NAV 2009.

In the third part, I will describe how to create this as a Microsoft Windows Vista Gadget with a flyout showing the search results.

### Scenario

The demo scenario goes like this:

In the tray of you Windows, there is a small Dynamics Icon

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_4.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_10.png)

If you click this Icon (or use an assigned global hotkey), a small window pops up

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_5.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_12.png)

The user types in what he is looking for and hit ENTER, which closes the Search Window and pops up the Search Result window:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_6.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_14.png)

Now the user can click on the links in the left hand side in order to link back into NAV, or select them with the keyboard.

The Vista Gadget which we will be completing in Part 3 looks like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_7.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_16.png)

The only disadvantage of the Vista Gadget is, that it is doesn’t really support the keyboard very well – I like the System Tray version better:-)

But – much of this is later on – the outcome of the first part is basically the following:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_22.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_46.png)

Admitted – not very useful, but stay tuned for part 2 and 3.

### Table definitions

When doing wildcard search on multiple tables, we of course need some setup tables, which will tell us which tables to search in. We also need to setup which fields in these tables we want to search in – and we need a table definition for a temporary table in which we can store the search result.

The Search Tables table defines which tables to search through.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_4.png)

**Table No** defines a table to search through.  
**Page No** is the card page which should be used for showing a record from the table.  
**Id Field No** is the field number of the ID field in the table.  
**Name Field No** is the field number of the Name field in the table.

I have only one key in the Search Tables table – and that is Table No.

and the Search Fields table defines which fields to search for in these tables:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_2.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_6.png)

**Table No** is the table and **Field No** is a field which should be included in the search.

Also in the Search Fields we only have one key, which includes both Table No and Field No, and last but not least we need a table definition, which we use for a temporary table while doing the search.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_8.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_18.png)

for every match we find in a table, we create one record in this temporary table, where

**Bookmark** is the bookmark (used when launching a page in the Role Tailored Client).  
**Name** is the name field of the record.  
**Table** is the name of the Table in which the record was found.  
**Id** is the id field of the record.  
**Page** is the Page number we want to open in the Role Tailored Client for this record.

It should be clear how the outcome of this can become the XML you see in the InfoPath above – and probably also how this then transforms into the HTML in part 2 and 3.

### The Search Codeunit

Disclaimer: Note, that I am not a trained C/AL developer – meaning that the following code might not be the most efficient – but it works for the purpose for which I use it. If you find things, that can be done smarter, better, faster or just things that are made stupid, let me know so that I can learn something as well.

First of all we create a Codeunit called Search and add the following code. The first section of the DoSearch method is all about searching.

The function loops through all Search Tables and for each Search Table, it loops through the Search Fields – and perform a search. For every match we create a record in the results temporary table (if it isn’t already inserted).

**DoSearch(searchstring : Text\[40\];VAR result : BigText)  
**CLEAR(result);  
results.DELETEALL;  
IF searchtable.FIND(‘-‘) THEN  
BEGIN  
REPEAT  
rec.OPEN(searchtable.”Table No”);  
searchfield.SETRANGE(searchfield.”Table No”, searchtable.”Table No”);  
IF searchfield.FIND(‘-‘) THEN  
BEGIN  
REPEAT  
rec.RESET();  
field := rec.FIELD(searchfield.”Field No”);  
field.SETFILTER(‘\*’ + searchstring + ‘\*’);  
IF rec.FIND(‘-‘) THEN  
BEGIN  
REPEAT  
results.SETRANGE(results.Bookmark, FORMAT(rec.RECORDID,0,10));  
IF NOT results.FIND(‘-‘) THEN  
BEGIN  
results.INIT();  
results.Bookmark := FORMAT(rec.RECORDID,0,10);  
results.Id := rec.FIELD(searchtable.”Id Field No”).VALUE;  
results.Name := rec.FIELD(searchtable.”Name Field No”).VALUE;  
results.Page := searchtable.”Page No”;  
results.Table := rec.NAME;  
results.INSERT();  
END;  
UNTIL rec.NEXT = 0;  
END;  
field.SETFILTER(”);  
UNTIL searchfield.NEXT =0;  
END;  
rec.CLOSE;  
searchfield.SETRANGE(searchfield.”Table No”);  
UNTIL searchtable.NEXT = 0;  
END;

Note, the FORMAT(recid, 0, 10) – which is the way to get a bookmark, which can be used for linking back into NAV 2009.

You probably noticed that the result is defined as a BigText and not as a XMLPort. If I was doing to use this function from C# only, I might have made it as a XMLPort – and used the strongly typed interface – but I also need to connect to this Web Service from Javascript (in part 3), so I will stick with the BigText.

That does however mean, that we manually have to build the XML document based on the temporary results table. The following code is also in the DoSearch function:

results.RESET;  
results.SETCURRENTKEY(results.Table, results.Id);  
IF results.FIND(‘-‘) THEN  
BEGIN  
CREATE(XMLDoc, false, false);  
XMLDoc.async(FALSE);  
TopNode := XMLDoc.createNode(1,’SEARCHRESULT’,”);  
XMLDoc.appendChild(TopNode);  
currentTable := ”;  
REPEAT  
IF results.Table <> currentTable THEN  
BEGIN  
currentTable := results.Table;  
TableNode := XMLDoc.createNode(1,’TABLE’,”);  
TableAttribute := XMLDoc.createAttribute(‘NAME’);  
TableAttribute.value := currentTable;  
TableNode.attributes.setNamedItem(TableAttribute);  
TopNode.appendChild(TableNode);  
END;  
MatchNode := XMLDoc.createNode(1,’MATCH’,”);  
MatchAttribute := XMLDoc.createAttribute(‘PAGE’);  
MatchAttribute.value := results.Page;  
MatchNode.attributes.setNamedItem(MatchAttribute);  
ValueNode := XMLDoc.createNode(1,’BOOKMARK’,”);  
ValueTextNode := XMLDoc.createTextNode(results.Bookmark);  
ValueNode.appendChild(ValueTextNode);  
MatchNode.appendChild(ValueNode);  
ValueNode := XMLDoc.createNode(1,’ID’,”);  
ValueTextNode := XMLDoc.createTextNode(results.Id);  
ValueNode.appendChild(ValueTextNode);  
MatchNode.appendChild(ValueNode);  
ValueNode := XMLDoc.createNode(1,’NAME’,”);  
ValueTextNode := XMLDoc.createTextNode(results.Name);  
ValueNode.appendChild(ValueTextNode);  
MatchNode.appendChild(ValueNode);  
TableNode.appendChild(MatchNode);  
UNTIL results.NEXT = 0;  
result.ADDTEXT(XMLDoc.xml);  
END;

Note that we build the XML in a server side COM object (XMLDoc) and after building the XML Document, I insert that in a BigText in one go.

That statement would fail in the Classic Client (because XMLDoc.xml often is larger than the allowed Text size) – but on the Service Tier, this works just fine because the ADDTEXT takes a string – and there is no size limit on that.

### Populating Search Tables and Search Fields

Before we can test the Search Web Service, we need to define which fields we want the search mechanism to run on.

This can of course be done manually – or we can create a function to do this. I of course went for the second approach and created a function to populate the tables with the following data:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_9.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_20.png) [![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_10.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_22.png)

You can see the code of that function if you download the sample, but basically it scans through metadata for all tables and searches for tables with a card form specified, which has a Search Name/Description in the table.

Now having populated the Search Tables we are ready to expose the code unit as a Web Service and test it. In the Web Service Table we need to expose the Search Codeunit:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_11.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_24.png)

having done this – you should be able to start an Internet Explorer and type in the following URL

[http://localhost:7047/DynamicsNAV/WS/CRONUS\_International\_Ltd/Codeunit/Search](http://localhost:7047/DynamicsNAV/WS/CRONUS_International_Ltd/Codeunit/Search)

giving you the WSDL of the Web Service (given of course that your Service Tier is on localhost, DynamicsNAV is your instance name and you are using the default W1 database.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_12.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_26.png)

### Testing the Web Service from Infopath

Infopath is a nice tool to test your Web Service methods and it is really pretty easy. I have added a walkthrough of how to do case you haven’t tried it before.

Start InfoPath and design a Form Template

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_13.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_28.png)

Base the Form Template on a Web Service

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_14.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_30.png)

Only receive data, since we are not going to alter any data in this case

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_15.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_32.png)

Use the URL pointing to the Search Codeunit Web Service

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_16.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_34.png)

Select the DoSearch operation

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_17.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_36.png)

and give the Data Connection a name

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_18.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_38.png)

Set the title and the Subtitle and drag the search string to the parameters section and the result to the data section

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_19.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_40.png)

Make the result field higher, select TextBox Properties and check the Multiline checkbox, and hit Preview

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_21.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_44.png)

Now the Infopath template launches and can type in cycle in the search string and hit the Run Query button. You probably need to allow Infopath to communicate to your Web Service – but after that you should get the following result

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_thumb_22.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/test_11D91/image_46.png)

That’s it – if you get something similar to this, the search method works.

In part 2 we will create a small Winforms application consuming this Web Service.

You can download a zip file containing the NAV objects in a .fob file and the Infopath template here: [http://www.freddy.dk/Search – Part 1.zip](<http://www.freddy.dk/Search - Part 1.zip> "http://www.freddy.dk/Search - Part 1.zip").

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
