---
layout: post
title: "Edit In Excel – Part 1 (out of 4)"
date: 2008-11-06 20:07:43
categories: ["Archive"]
tags: ["C#", "Data", "Excel", "NAV 2009", "Web Services"]
permalink: /2008/11/06/edit-in-excel-part-1-out-of-4/
---

For the last 6-9 months, Microsoft have been showing a demo of how a user could create a filter on a list place and invoke an action called Edit In Excel. This would open Microsoft Excel with the same records, the user selected in the List Place and allow the user to edit values and post modifications back to Microsoft Dynamics NAV, showing how the user would get a runtime error if he was trying to violate validation logic from NAV.

I will divide this post into 4 sections on how to achieve this

1.  Create an Excel Spreadsheet, which reads the entire Customer table and show it in Excel.
2.  Create an Excel Spreadsheet, which reads the entire Customer table, show it and allow the user to modify, delete or add records directly in Excel – and save the changes back to NAV.
3.  Hook this spreadsheet up to the Customer List Place and take the filter defined on the Customer List Place and apply that to Excel.
4.  Make the damn thing loosely coupled – allowing it to be placed on any List Place if you fancy.

I chose to divide it, in order to allow people better to understand the processes and I think that every one of these four steps will take you through a number of new things.

This first post is all about creating a spreadsheet, which reads the entire Customer table and show it in Excel. We could do this from inside NAV (populate an XML document and send it to the Client) but that would put us into a blind alley when going towards post number 2.

So – we are going to use VSTO

### What is VSTO?

VSTO (Visual Studio Tools for Office) came out as an add-on to Visual Studio 2005 and in Visual Studio 2008, it is an integrated part of the professional SKU. Basically it allows you to write code inside Excel, Word, Outlook etc. – add toolbars, menu items, subscribe to triggers and do all kinds of things using your favorite .net language – and mine is C#.

VSTO is NOT in Visual Studio Express SKU’s.

I am not going to make this a tutorial on how to use VSTO – but I will just show a couple of basics. When creating a new project in Visual Studio, you have new options:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_2.png)

and when you select to create an Excel Template – it actually opens up a spreadsheet inside of Visual Studio.

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_4.png)

Now Visual Studio might not be your favorite environment for using Excel, but that isn’t the point. If you look to the right you will see a solution and a project with a number of C# files under your .xltx “folder”. These are files, which contains code behind for the different sheets.

Right click on Sheet1.cs and select View Code and you will see:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_thumb_2.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_6.png)

Which kind of indicates where we are going here…

I can write code in the Sheet1\_Startup, which connects to our Microsoft Dynamics NAV Web Services and read data into the Spreadsheet – could it really be that simple?

### Yes, it really is that simple – but…

… when you see the code beneath in a moment you can see that this is really not scaling – and it really wouldn’t allow us to edit the data – but hey, that is in post number 2 – this was the simple version, let’s continue.

You of course need to add a Web Reference to the Customer page (Page 21 – exposed as Customer) using the following URL (if the NAV is installed as default):

[http://localhost:7047/DynamicsNAV/WS/CRONUS\_International\_Ltd/Page/Customer](http://localhost:7047/DynamicsNAV/WS/CRONUS_International_Ltd/Page/Customer)

and call the reference CustomerRef.

Add the following code to Sheet1\_Startup and run your solution.

// Postpone Screen updating  
Application.ScreenUpdating = false;

// Initialize the Service  
CustomerRef.Customer\_Service service = new CustomerSimple.CustomerRef.Customer\_Service();  
service.UseDefaultCredentials = true;

// Read the customers  
CustomerRef.Customer\[\] customers = service.ReadMultiple(null, null, 0);

// Populate the header line  
int row = 1;  
this.Cells\[row, 1\] = “No”;  
this.Cells\[row, 2\] = “Name”;  
this.Cells\[row, 3\] = “Address”;  
this.Cells\[row, 4\] = “Address\_2”;  
// etc.

// Fill the spreadsheet  
foreach (CustomerRef.Customer customer in customers)  
{  
row++;  
this.Cells\[row, 1\] = customer.No;  
this.Cells\[row, 2\] = customer.Name;  
this.Cells\[row, 3\] = customer.Address;  
this.Cells\[row, 4\] = customer.Address\_2;  
// etc.  
}

// Set formatting for the added cells  
Microsoft.Office.Interop.Excel.Range range = this.Range\[this.Cells\[1, 1\], this.Cells\[row, 4\]\];  
range.EntireColumn.NumberFormat = “@”;  
range.EntireColumn.HorizontalAlignment = Microsoft.Office.Interop.Excel.Constants.xlLeft;  
range.EntireColumn.AutoFit();

// Update the Screen  
Application.ScreenUpdating = true;

That’s it and that’s that! This should bring up Excel looking like this:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_thumb_3.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart1outof4_E5A0/image_8.png)

Not quite the end-goal, but I hope you get the picture.

As usual – you can download the solution here [http://www.freddy.dk/CustomerSimple.zip](http://www.freddy.dk/CustomerSimple.zip "http://www.freddy.dk/CustomerSimple.zip")

### So where do we go from here?

In the next post we will start by removing the code we just wrote and write some new and better code in C#. We will still not touch NAV (only from Web Services). The goal here is to read the data from NAV into a table, with knowledge about the field types and add the NAV Key to a hidden column – all in order to be able to post data back to NAV.

My next post will also add a couple of buttons to the Excel toolbar and add the Save and Reload functions. It will still be hard coded to the Customer table though.

In my third post I will explain how to get parameters from code on the Service Tier into our Spreadsheet (like the filter) and hook things up accordingly.

Last, but not least, I will explain how we can do this without having a Web Reference to the Customer or other pages – how can we do this dynamically.

Stay tuned, enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
