---
layout: post
title: "Transferring binary data to/from WebServices (and to/from COM (Automation) objects)"
date: 2008-11-04 16:58:50
categories: ["Archive"]
tags: ["Automation", "BigText", "Binary", "C#", "Client Tier", "COM", "Data", "NAV 2009", "Service Tier", "Temporary File", "Web Services"]
permalink: /2008/11/04/transferring-binary-data-tofrom-webservices-and-tofrom-com-automation-objects/
---

A number of people have asked for guidance on how to transfer data to/from COM and WebServices in NAV 2009.

In the following I will go through how to get and set a picture on an item in NAV through a Web Service Connection.

During this scenario we will run into a number of obstacles – and I will describe how to get around these.

First of all – we want to create a Codeunit, which needs to be exposed to WebServices. Our Codeunit will contain 2 functions: GetItemPicture and SetItemPicture – but what is the data type of the Picture and how do we return that value from a WebService function?

The only data type (supported by Web Services) that can hold a picture is the BigText data type.

We need to create the following two functions:

GetItemPicture(No : Code\[20\];VAR Picture : BigText)  
SetItemPicture(No : Code\[20\]; Picture : BigText);

BigText is capable if holding binary data (including null terminals) up to any size. On the WSDL side these functions will have the following signature:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/TransferringdatatofromCOMAutomationobjec_C581/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/TransferringdatatofromCOMAutomationobjec_C581/image_2.png)

As you can see BigText becomes string – and strings in .net are capable of any size and any content.

The next problem we will face is, that pictures typically contains all kinds of characters, and unfortunately when transferring strings to/from WebServices there are a number of special characters that are not handled in the WebServices protocol.

(Now you wonder whether you can have <> in your text – but that isn’t the problem:-)

The problem is LF, CR, NULL and other characters like that.

So we need to base64 (or something like that) encode our picture when returning it from Web Services. Unfortunately I couldn’t find any out-of-the-box COM objects that was able to do base64 encoding and decoding – but we can of course make one ourselves.

Lets assume for a second that we have a base64 COM object – then this would be our functions in AL:

**GetItemPicture(No : Code\[20\];VAR Picture : BigText)  
**CLEAR(Picture);  
Item.SETRANGE(Item.”No.”, No, No);  
IF (Item.FINDFIRST()) THEN  
BEGIN  
  Item.CALCFIELDS(Item.Picture);  
// Get Temp FileName  
TempFile.CREATETEMPFILE;  
FileName := TempFile.NAME;  
TempFile.CLOSE;

  // Export picture to Temp File  
Item.Picture.EXPORT(FileName);

  // Get a base64 encoded picture into a string  
CREATE(base64);  
Picture.ADDTEXT(base64.encodeFromFile(FileName));

  // Erase Temp File  
FILE.ERASE(FileName);  
END;

**SetItemPicture(No : Code\[20\];Picture : BigText)  
**Item.SETRANGE(Item.”No.”, No, No);  
IF (Item.FINDFIRST()) THEN  
BEGIN  
// Get Temp FileName  
TempFile.CREATETEMPFILE;  
FileName := TempFile.NAME;  
TempFile.CLOSE;

  // Decode the bas64 encoded image into the Temp File  
CREATE(base64);  
base64.decodeToFile(Picture, FileName);

  // Import picture from Temp File  
Item.Picture.IMPORT(FileName);  
Item.Modify();  
// Erase Temp File  
FILE.ERASE(FileName);  
END;

A couple of comments to the source:

-   The way we get a temporary filename in NAV2009 is by creating a temporary file, reading its name and closing it. CREATETEMPFILE will always create new GUID based temporary file names – and the Service Tier will not have access to write files in e.g. the C:\\ root folder and a lot of other places.
-   The base64 automation object is loaded on the Service Tier (else it should be CREATE(base64, TRUE, TRUE);) and this is the right location, since the exported file we just stored is located on the Service Tier.
-   The base64.encodeFromFile reads the file and returns a very large string which is the picture base64 encoded.
-   The ADDTEXT method is capable of adding these very large strings and add them to a BigText (BTW – that will NOT work in the classic client).
-   We do the cleanup afterwards – environmental protection:-)

So, why does the ADDTEXT support large strings?

As you probably know, the ADDTEXT takes a TEXT and a position as parameter – and a TEXT doesn’t allow large strings, but what happens here is, that TEXT in C# becomes string – and the length-checking of TEXT variables are done when assigning variables or transferring parameters to functions and the ADDTEXT doesn’t check for any specific length (which comes in handy in our case).

The two lines in question in C# looks like:

base64.Create(DataError.ThrowError);  
picture.Value = NavBigText.ALAddText(picture.Value, base64.InvokeMethod(@”encodeFromFile”, fileName));

Note also that the base64.decodeToFile function gets a BigText directly as parameter. As you will see, that function just takes an object as a parameter – and you can transfer whatever to that function (BigText, Text, Code etc.). You actually also could give the function a decimal variable in which case the function would throw an exception (str as string would return NULL).

So now you also know how to transfer large strings to and from COM objects:

1.  To the COM object, you just transfer a BigText variable directly to an object parameter and cast it to a string.
2.  From the COM object to add the string return value to a BigText using ADDTEXT.
3.  You cannot use BigText as parameter to a by-ref (VAR) parameter in COM.

In my WebService consumer project I use the following code to test my WebService:

// Initialize Service  
CodeUnitPicture service = new CodeUnitPicture();  
service.UseDefaultCredentials = true;

// Set the Image for Item 1100  
service.SetItemPicture(“1100″, encodeFromFile(@”c:\\MandalayBay.jpg”));

// Get and show the Image for Item 1001  
string p = “”;  
service.GetItemPicture(“1001″, ref p);  
decodeToFile(p, @”c:\\pic.jpg”);  
System.Diagnostics.Process.Start(@”c:\\pic.jpg”);

and BTW – the source code for the two functions in the base64 COM object are here:

public string encodeFromFile(string filename)  
{  
FileStream fs = File.OpenRead(filename);  
BinaryReader br = new BinaryReader(fs);  
int len = (int)fs.Length;  
byte\[\] buffer = new byte\[len\];  
br.Read(buffer, 0, len);  
br.Close();  
fs.Close();  
return System.Convert.ToBase64String(buffer);  
}

public void decodeToFile(object str, string filename)  
{  
FileStream fs = File.Create(filename);  
BinaryWriter bw = new BinaryWriter(fs);  
bw.Write(Convert.FromBase64String(str as string));  
bw.Close();  
fs.Close();  
}

If you whish to download and try it out for yourself – you can download the sources here:

The two Visual Studio solutions can be downloaded from [http://www.freddy.dk/VSDemo.zip](http://www.freddy.dk/VSDemo.zip "http://www.freddy.dk/VSDemo.zip") (the base64 COM object and the VSDemo test project)

The NAV codeunit with the two functions above can be downloaded from [http://www.freddy.dk/VSDemoObjects.fob](http://www.freddy.dk/VSDemoObjects.fob "http://www.freddy.dk/VSDemoObjects.fob").

Remember that after importing the CodeUnit you would have to expose it as a WebService in the WebService table:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/TransferringdatatofromCOMAutomationobjec_C581/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/TransferringdatatofromCOMAutomationobjec_C581/image_4.png)

And…. – remember to start the Web Service listener (if you are running with an unchanged Demo installation).

_The code shown in this post comes with no warranty – and is only intended for showing how to do things. The code can be reused, changed and incorporated in any project without any further notice._

Comments or questions are welcome.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
