---
layout: post
title: "Edit In Excel – bug fix and R2"
date: 2009-04-12 13:51:26
categories: ["Archive"]
tags: ["Bug", "C#", "Excel", "NAV 2009", "Web Services"]
permalink: /2009/04/12/edit-in-excel-bug-fix-and-r2/
---

If you haven’t read the 4 step walkthrough of how to Edit In Excel from Microsoft Dynamics NAV, you should do so [here](http://blogs.msdn.com/freddyk/archive/2008/11/09/edit-in-excel-part-4-out-of-4.aspx) this post is a follow up to the original posts.

I have received a number of suggestions to what you could do with the Edit In Excel and a single bug. In this post I will fix the bug and I will explain what R2 is all about.

### The Bug

The bug description is, that if you take my binaries and use them on a machine without regional settings = US – you will get an exception (Thanks Waldo).

Whether this is my bug or whether Excel behaves strange I will leave to the reader, but fact is, that if I have created a VSTO template for Excel on a machine with regional settings = US – then you cannot create a spreadsheet based on that template from code if your computer is not setup the same way.

The easiest fix I found to this problem (and now somebody might say that I am a hacker) is to temporarily set the current culture to en-US while opening the spreadsheet, then everything seems to work.

So, change this line:

`excelApp.Workbooks.Add(template);`

to

```
// Set the current culture to en-US when adding the template to avoid an exception if running on a non-US computer
System.Globalization.CultureInfo orgCulture = System.Threading.Thread.CurrentThread.CurrentCulture;
System.Threading.Thread.CurrentThread.CurrentCulture = new System.Globalization.CultureInfo("en-US");
```

```
// Create a new spreadsheet based on the new template
excelApp.Workbooks.Add(template);
```

```
// Restore culture in current thread
System.Threading.Thread.CurrentThread.CurrentCulture = orgCulture;
```

I have updated the original binaries with this fix and they can be downloaded from [http://www.freddy.dk/NAVTemplate\_Final.zip](http://www.freddy.dk/NAVTemplate_Final.zip)

### R2

One suggestion I have received a number of times is whether it is possible to save the spreadsheet with customers and be able to have a local spreadsheet connected to your NAV that you can load and work with.

So – I set out to fix this – shouldn’t be a biggy (I thought), but it turned out that there are a number of things you need to take into consideration. It isn’t a total rewrite, but it is a pretty significant change, so I decided that it was time for Edit In Excel R2.

Stay tuned – Edit In Excel R2 coming up in a few days.

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
