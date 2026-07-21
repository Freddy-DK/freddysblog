---
layout: post
title: "NAV 2009 and Unicode!"
date: 2008-11-10 05:13:03
categories: ["Archive"]
tags: ["BigText", "Binary", "C#", "COM", "NAV 2009", "Service Tier", "Unicode", "Web Services"]
permalink: /2008/11/10/nav-2009-and-unicode/
---

The title might be a bit misleading, but I am writing this post as a response to a problem, which a partner ran into with NAV 2009 – and the problem is caused by Unicode. I am not a Unicode expert, so bare with me if I am naming some things wrong.

As you know, NAV 2009 is a 3T architecture and the Service Tier is 95% managed code (only the lower pieces of the data stack is still unmanaged code). You probably also know, that managed code supports Unicode natively – in fact a string in C# or VB.Net is by default a Unicode string.

In C# you use byte\[\] if you need to work with binary data. My earlier post about transferring binary data between the Service Tier and Client Side COM or Web Service consumers (you can find it [here](http://blogs.msdn.com/freddyk/archive/2008/11/04/transferring-data-to-from-com-automation-objects-and-webservices.aspx)) I read the file content into a byte\[\] and use a base64 encoding to encode this content into a string, which is transferable between platforms, code pages etc.

### Is NAV 2009 then Unicode compliant?

No, we cannot claim that. There are a lot of things, that stops us from having a fully Unicode compliant product – but we are heading in that direction. Some of the things are:

-   The development environment and the classic client is not Unicode. When you write strings in AL code they are in OEM.
-   The import/export formats are OEM format.
-   I actually don’t know what the format inside SQL is, but I assume it isn’t Unicode (again because we have mixed platforms).

Lets take an example:

In a codeunit I write the following line:

txt := ‘ÆØÅ are three Danish letters, ß is used in German and Greek.’;

Now, I export this as a .txt file, and view this in my favorite DOS text viewer:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/NAV2009andUnicode_1F2E/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/NAV2009andUnicode_1F2E/image_2.png)

As you can see – the Ø has changed and a quick look at my codepage reveals that I am running 437 – the codepage that doesn’t have an Ø.

Opening the exported file in Notepad looks different:

Txt := ‘’î are three Danish letters, á is used in German and Greek.’;

Primary reason for this is, that Notepad assumes Unicode and the .txt file is OEM. When I launch the RoleTailored Client, and take a look at that line in the generated C# source file in Notepad:

txt = new NavText(1024, @”ÆØÅ are three Danish letters, ß is used in German and Greek.”);

Nice – and we know that Notepad is Unicode compliant and C# uses Unicode strings, so the AL -> C# compiler converts the string to Unicode – how does this look in my favorite DOS text viewer:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/NAV2009andUnicode_1F2E/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/NAV2009andUnicode_1F2E/image_4.png)

Clearly my string has been encoded.

### But wait…

If NAV 2009 converts my text constants to Unicode – what if I write this string to a file using my well known file commands in AL? – well, let’s try.

I added the following code to a Codeunit and ran it from both the Role Tailored Client and from the Classic Client.

file.CREATETEMPFILE();  
filename := file.NAME;  
file.CLOSE();  
file.CREATE(filename);  
file.CREATEOUTSTREAM(OutStr);  
OutStr.WRITETEXT(Txt);  
file.CLOSE();

and don’t worry, the results are identical – both platforms actually writes the file in OEM format (we might have preferred Unicode, but for compatibility reasons this is probably a good thing).

Another thing we should try, is to call a managed COM object – and take a look at the string we get there – and again we get the same string from the Classic and from the Role Tailored Client – but in this case, we do NOT get the string in OEM format – we get a Unicode string.

### MSXML

Now if we call an unmanaged COM object (like MSXML2 – XMLHTTP) we actually get the OEM string when invoked from the Classic Client and a Unicode string when invoked from the Role Tailored Client. Typically XMLHTTP is used with ASCII only – but in some cases, they do have binary data – which might be in the 128-255 character range.

Our problem now is, that our binary data (which didn’t have any relation to any special characters) gets converted to Unicode – and the Web Service provider doesn’t stand a chance to guess what we mean with the data we send.

The next problem is that the Role Tailored Client doesn’t support a byte\[\] type (binary) – in which we could have build a command and send it over. I tried a number of things, but didn’t find a way to send any binary data (above 128) to the Send command of XMLHTTP.

The third problem with XMLHTTP is that the only way we can get a result back is by reading the ResponseText – and that is treated as a Unicode string and gets crumbled before we get it into NAV.

Remember that these problems will not occur if the web service provider uses XML to transfer data back and forth – since XML is ASCII compliant.

My first proposal if you are having problems with a Web Service provider, using a binary communication is to query the provider and ask for an XML version of the gateway. If this isn’t possible – you have a couple of options (which both include writing a new COM object).

### Create a proxy

You could create a Web Service proxy as a COM component (probably server side) and have a higher level function you call. This would remove the XMLHTTP glue code from NAV and put that into the COM object.

Example – you have a Web Service provider who can verify credit card numbers – and normally you would build up a string in AL and send this to the Send command – and then parse the ResponseStream you get back to figure out whether everything was A OK for not.

Create a function in a new COM object which might be named:

int CheckCreditCard(string CreditCardNo, string NameOnCard, int ExpireMonth, int ExpireYear, int SecurityCode)

Then your business logic in AL would just call and check – without a lot of XML parsing and stuff.

This would be my preferred choice – but it does involve some refactoring and a new COM object that needs to be installed on the server.

### Use temporary files to transfer data

As mentioned earlier – writing a file in NAV 2009 with binary data, creates a file in OEM format – which would be the same binary content as we are typing in the AL editor.

So, you could create the string you want to send to XMLHTTP in a temporary file, create a new COM object which contains a function which sends the content of a file to a XMLHTTP object and writes the response back into the same file for NAV to read.

The idea here is that files (and byte\[\]) are binary data – strings are not.

The function in the COM object could look like:

public int SendFileContent(object NAVxmlhttp, string filename)  
{  
MSXML2.ServerXMLHTTP xmlHttp = NAVxmlhttp as MSXML2.ServerXMLHTTP;  
FileStream fs = File.OpenRead(filename);  
BinaryReader br = new BinaryReader(fs);  
int len = (int)fs.Length;  
byte\[\] buffer = new byte\[len\];  
br.Read(buffer, 0, len);  
br.Close();  
fs.Close();

    xmlHttp.send(buffer);

    buffer = xmlHttp.responseBody as byte\[\];  
fs = File.Create(filename);  
BinaryWriter bw = new BinaryWriter(fs);  
bw.Write(buffer);  
bw.Close();  
fs.Close();

    return xmlHttp.status;  
}

If you went for this approach your AL code would change from:

XmlHttp.Send(Txt);

(followed by opening the ResponseStream) to

file.CREATETEMPFILE();  
filename := file.NAME;  
file.CLOSE();  
file.CREATE(filename);  
file.CREATEOUTSTREAM(OutStr);  
OutStr.WRITETEXT(Txt);  
file.CLOSE();

myCOMobject.SendFileContent(XmlHttp, filename);

(followed by opening the file <filename> again – reading the result).

The second approach would also work from the Classic client – so you don’t have to use IF ISSERVICETIER THEN to do the one of the other.

In the Edit In Excel – Part 4, you can see how to create a COM object – should be pretty straightforward.

### Build a binary string that doesn’t get encoded

You could create a function in a COM object, which returns a character based on a numeric value:

public string GetChar(int ch)  
{  
return “”+(char)ch;  
}

Problem with this direction is, that this function should ONLY be used when running in the Role Tailored Client.

Calling this function from the Classic Client will case this string to be seen as Unicode and converted back into OEM – and that really wouldn’t make sense at all.

### Convert to/from Unicode using strings instead of files

So what if the data you need to send is confidential and you cannot write that to a file?

Well – you can create a function, which converts the string back to OEM (making it the same binary image) – send it over the wire – and then convert the response to UniCode (so that when the string comes into NAV – it will be converted back to OEM again again).

Seems like a lot of conversion back and forth – but it would actually work from both the Classic and the Role Tailored Client, the code for that goes here:

public class MyCOMobject : IMyCOMobject  
{  
private static Byte\[\] oem2AnsiTable;  
private static Byte\[\] ansi2OemTable;

    /// <summary>  
/// Initialize COM object  
/// </summary>  
public MyCOMobject()  
{  
oem2AnsiTable = new Byte\[256\];  
ansi2OemTable = new Byte\[256\];  
for (Int32 i = 0; i < 256; i++)  
{  
oem2AnsiTable\[i\] = (Byte)i;  
ansi2OemTable\[i\] = (Byte)i;  
}  
NativeMethods.OemToCharBuff(oem2AnsiTable, oem2AnsiTable, oem2AnsiTable.Length);  
NativeMethods.CharToOemBuff(ansi2OemTable, ansi2OemTable, ansi2OemTable.Length);  
// Remove “holes” in the convertion structure  
Int32 ch1 = 255;  
Int32 ch2 = 255;  
for (;; ch1–, ch2–)  
{  
while (ansi2OemTable\[oem2AnsiTable\[ch1\]\] == ch1)  
{  
if (ch1 == 0)  
break;  
else  
ch1–;  
}  
while (oem2AnsiTable\[ansi2OemTable\[ch2\]\] == ch2)  
{  
if (ch2 == 0)  
break;  
else  
ch2–;  
}  
if (ch1 == 0)  
break;  
oem2AnsiTable\[ch1\] = (Byte)ch2;  
ansi2OemTable\[ch2\] = (Byte)ch1;  
}  
}

    /// <summary>  
/// Convert Unicode string to OEM string  
/// </summary>  
/// <param name=”str”>Unicode string</param>  
/// <returns>OEM string</returns>  
private byte\[\] UnicodeToOem(string str)  
{  
Byte\[\] buffer = Encoding.Default.GetBytes(str);  
for (Int32 i = 0; i < buffer.Length; i++)  
{  
buffer\[i\] = ansi2OemTable\[buffer\[i\]\];  
}  
return buffer;  
}

    /// <summary>  
/// Convert OEM string to Unicode string  
/// </summary>  
/// <param name=”oem”>OEM string</param>  
/// <returns>Unicode string</returns>  
private string OemToUnicode(byte\[\] oem)  
{  
for (Int32 i = 0; i < oem.Length; i++)  
{  
oem\[i\] = oem2AnsiTable\[oem\[i\]\];  
}  
return Encoding.Default.GetString(oem);  
}

    /// <summary>  
/// Send data through XMLHTTP  
/// </summary>  
/// <param name=”NAVxmlhttp”>XmlHttp object</param>  
/// <param name=”data”>string containing data (in Unicode)</param>  
/// <returns>The response from the XMLHTTP Send</returns>  
public string Send(object NAVxmlhttp, string data)  
{  
MSXML2.ServerXMLHTTP xmlHttp = NAVxmlhttp as MSXML2.ServerXMLHTTP;  
byte\[\] oem = UnicodeToOem(data);  
xmlHttp.send(oem);        return OemToUnicode((byte\[\])xmlHttp.responseBody);  
}  
}

internal static partial class NativeMethods  
{  
#region Windows OemToChar/CharToOem imports

    \[DllImport(“user32”, EntryPoint = “OemToCharBuffA”)\]  
internal static extern Int32 OemToCharBuff(Byte\[\] source, Byte\[\] dest, Int32 bytesize);

    \[DllImport(“user32”, EntryPoint = “CharToOemBuffA”)\]  
internal static extern Int32 CharToOemBuff(Byte\[\] source, Byte\[\] dest, Int32 bytesize);

    #endregion  
}

Unfortunately I have not found a way to do this without having a COM object in play.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
