---
layout: post
title: "Word Management"
date: 2009-09-20 12:51:36
categories: ["Archive"]
tags: ["AL", "BigText", "Client Tier", "Merge", "Performance", "Service Tier", "TAP", "Temporary File", "Word"]
permalink: /2009/09/20/word-management/
---

As with the release of Microsoft Dynamics NAV 2009, I was also deeply involved in the TAP for Microsoft Dynamics NAV 2009 SP1. My primary role in the TAP is to assist ISVs and partners in getting a customer live on the new version before we ship the product.

During this project we file a lot of bugs and the development team in Copenhagen are very responsive and we actually get a lot of bugs fixed – but… not all – it happens that a bug is closed with “By Design”, “Not Repro” or “Priority too low”.

As annoying as this might seem, I would be even more annoyed if the development team would take every single bug, fix it, run new test passes and punt the releases into the unknown. Some of these bugs then become challenges for me and the ISV / Partner to solve, and during this – it happens that I write some code and hand off to my contact.

Whenever I do that, two things are very clear

1.  The code is given as is, no warranty, no guarantee
2.  The code will be available on my blog as well, for other ISV’s and partners to see

and of course I send the code to the development team in Copenhagen, so that they can consider the fix for the next release.

### Max. 64 fields when merging with Word

One of the bugs we ran into this time around was the fact that when doing merge with Microsoft Word in a 3T environment, word would only accept 64 merge fields. Now in the base application WordManagement (codeunit 5054) only uses 48 fields, but the ISV i was working with actually extended that to 100+ fields.

The bug is in Microsoft Word, when merging with file source named .HTM – it only accepts 64 fields, very annoying.

We also found that by changing the filename to .HTML, then Word actually could see all the fields and merge seemed to work great (with one little very annoying aberdabei) – the following dialog would pop up every time you open Word:

[![clip\_image002](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/WordManagement_8AA5/clip_image002_thumb.jpg "clip_image002")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/WordManagement_8AA5/clip_image002_2.jpg)

Trying to figure out how to get rid of the dialog, I found the right parameters to send to Word.OpenDataSource, so that the dialog would disappear – but… – then we are right back with the 64 fields limitation.

The reason for the 64 field limitation is, that Word loads the HTML as a Word Document and use that word document to merge with and in a document, you cannot have more than 64 columns in a table (that’s at least what they told me).

I even talked to PM’s in Word and got confirmed that this behavior was in O11, O12 and would not be fixed in O14 – so no rescue in the near future.

### Looking at WordManagement

Knowing that the behavior was connected to the merge format, I decided to try and change that – why not go with a good old fashion .csv file instead and in my quest to learn AL code and application development, this seemed like a good little exercise.

So I started to look at WordManagement and immediately found a couple of things I didn’t like

MergeFileName := RBAutoMgt.ClientTempFileName(Text029,’.HTM’);  
IF ISCLEAR(wrdMergefile) THEN  
CREATE(wrdMergefile,FALSE,TRUE);  
// Create the header of the merge file  
CreateHeader(wrdMergefile,FALSE,MergeFileName);  
<find the first record>  
REPEAT  
// Add Values to mergefile – one AddField for each field for each record  
  wrdMergefile.AddField(<field value>);  // Terminate the line  
wrdMergefile.WriteLine;  
UNTIL <No more records>  
// Close the file  
wrdMergefile.CloseFile;

now wrdMergefile is a COM component of type ‘Navision Attain ApplicationHandler’.MergeHandler and as you can see, it is created Client side, meaning that for every field in every record we make a roundtrip to the Client (and one extra roundtrip for every record to terminate the line) – now we might not have a lot of records nor a lot of fields, but I think we can do better (said from a guy who used to think about clock cycles when doing assembly instructions on z80 processors back in the start 80’s – WOW I am getting old:-))

One fix for the performance would be to create the file serverside and send it to the Client in one go – but that wouldn’t solve our original 64 field limitation issue. I could also create a new COM component, which was compatible with MergeHandler and would write a .csv instead – but that wouldn’t solve my second issue about wanting to learn some AL code.

### Creating a .csv in AL code

I decided to go with a model, where I create a server side temporary file for each record, create a line in a BigText and write it to the file. After completing the MergeFile, it needs to be downloaded to the Client and deleted from the service tier.

The above code would change into something like

MergeFileName := CreateMergeFile(wrdMergefile);  
wrdMergefile.CREATEOUTSTREAM(OutStream);  
CreateHeader(OutStream,FALSE); // Header without data  
<find the first record>  
REPEAT  
CLEAR(mrgLine);  
// Add Values to mergefile – one AddField for each field for each record  
AddField(mrgCount, mrgLine, <field value>);  
  // Terminate the line  
mrgLine.ADDTEXT(CRLF);  mrgLine.WRITE(OutStream);  
CLEAR(mrgLine);  
UNTIL <No more records>  
// Close the file  
wrdMergeFile.Close();  
MergeFileName := WordManagement.DownloadAndDeleteTempFile(MergeFileName);

As you can see – no COM components, all server side. A couple of helper functions are used here, but no rocket science and not too different from the code that was.

CreateMergeFile creates a server side temporary file.

**CreateMergeFile(VAR wrdMergefile : File) MergeFileName : Text\[260\]  
**wrdMergefile.CREATETEMPFILE;  
MergeFileName := wrdMergefile.NAME + ‘.csv’;  
wrdMergefile.CLOSE;  
wrdMergefile.TEXTMODE := TRUE;  
wrdMergefile.WRITEMODE := TRUE;  
wrdMergefile.CREATE(MergeFileName);

AddField adds a field to the BigText. Using AddString, which again uses DupQuotes to ensure that “ inside of the merge field are doubled.

**AddField(VAR count : Integer;VAR mrgLine : BigText;value : Text\[1024\])  
**IF mrgLine.LENGTH = 0 THEN  
BEGIN  
count := 1;  
END ELSE  
BEGIN  
count := count + 1;  
mrgLine.ADDTEXT(‘,’);  
END;  
mrgLine.ADDTEXT(‘”‘);  
AddString(mrgLine, value);  
mrgLine.ADDTEXT(‘”‘);

**AddString(VAR mrgLine : BigText;str : Text\[1024\])  
**IF STRLEN(str) > 512 THEN  
BEGIN  
mrgLine.ADDTEXT(DupQuotes(COPYSTR(str,1,512)));  
str := DELSTR(str,1,512);  
END;  
mrgLine.ADDTEXT(DupQuotes(str));

**DupQuotes(str : Text\[512\]) result : Text\[1024\]  
**result := ”;  
REPEAT  
i := STRPOS(str, ‘”‘);  
IF i <> 0 THEN  
BEGIN  
result := result + COPYSTR(str,1,i) + ‘”‘;  
str := DELSTR(str,1,i);  
END;  
UNTIL i = 0;  
result := result + str;

and a small function to return CRLF (line termination for a merge line)

**CRLF() result : Text\[2\]  
**result\[1\] := 13;  
result\[2\] := 10;

When doing this I did run into some strange errors when writing both BigTexts and normal Text variables to a stream – that is the reason for building everything into a BigText and writing once pr. line.

and last, but not least – a function to Download a file to the Client Tier and delete it from the Service Tier:

**DownloadAndDeleteTempFile(ServerFileName : Text\[1024\]) : Text\[1024\]  
**IF NOT ISSERVICETIER THEN  
EXIT(ServerFileName);

FileName := RBAutoMgt.DownloadTempFile(ServerFileName);  
FILE.ERASE(ServerFileName);  
EXIT(FileName);

It doesn’t take much more than that… (beside of course integrating this new method in the various functions in WordManagement). The fix doesn’t require anything else than just replacing codeunit 5054 and the new WordManagement can be downloaded [here](http://www.freddy.dk/NewWordManagement.zip).

Question is now, whether there are localization issues with this. I tried changing all kinds of things on my machine and didn’t run into any problems – but if anybody out there does run into problems with this method – please let me know so.

### What about backwards compatibility

So what if you install this codeunit into a system, where some of these merge files already have been created – and are indeed stored as HTML in blob fields?

Well – for that case, I created a function that was able to convert them – called

ConvertContentFromHTML(VAR MergeContent : BigText) : Boolean

It isn’t pretty – but it seems to work.

### Feedback is welcome

I realize that by posting this, I am entering a domain where I am the newbie and a lot of other people are experts. I do welcome feedback on ways to do coding, things I can do better or things I could have done differently.

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
