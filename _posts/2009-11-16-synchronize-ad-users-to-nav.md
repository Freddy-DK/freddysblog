---
layout: post
title: "Synchronize A/D users to NAV"
date: 2009-11-16 09:58:24
categories: ["Archive"]
tags: ["A/D", "C#", "NAV 2009 SP1", "Web Services"]
permalink: /2009/11/16/synchronize-ad-users-to-nav/
---

During my work with demos like Edit In Excel, I wanted to make sure that these things would work in all localized versions of NAV 2009 SP1 – meaning that I needed to install 14 different databases and 14 running Service Tier’s. Having done that, I also wanted to allow my colleagues who needed to check something, access to these service tiers.

For a geek (like me:-)), that problem looks like something you need to write an application for, even though it probably takes more time than it would be to add the users one by one whenever needed, but it certainly is more fun to write the application – and… maybe somebody else can actually use the ideas from this to do something cool.

### How to do?

Basically what I want to do is, to enumerate the Remote Desktop Users of the computer and make sure that these users are SUPER in NAV. Now, I can do this in NAV calling out to a COM object enumerating my users – but that really wouldn’t help me, because I would have to start NAV with every service tier or database and launch that action.

So, first I created a codeunit in NAV, exposed this codeunit as Web Services. Then I created a console application running on a schedule on my server, which enumerates the users and invoke the Web Service function to check all users are created in NAV.

Of course the scheduled application has to run with elevated permissions and that user needs to be able to change users in NAV as well.

### The NAV code

The codeunit contains two functions:

**ImportUserSID(SID : Text\[119\];Role : Code\[20\]) : Boolean  
**IF NOT Winlogin.GET(SID) THEN  
EXIT(FALSE);  
Winlogin.INIT;  
Winlogin.SID := SID;  
Winlogin.INSERT();  
IF NOT WinAccess.GET(SID) THEN  
BEGIN  
WinAccess.INIT;  
WinAccess.”Login SID” := SID;  
WinAccess.”Role ID” := Role;  
WinAccess.INSERT;  
END;  
EXIT(TRUE);

and

**SynchronizeUsers()  
**DATABASE.SYNCHRONIZEALLLOGINS();

BTW. Calling SynchronizeAllLogins running through Web Services didn’t seem to work when called through Web Services and running enhanced database security – whether this is a bug or a problem in my setup – I don’t know, but I am going to file a bug on it.

As you can see, the ImportUserSID does not modify the role if the user is already in the login table – it could of course be modified to do that easily, but it wasn’t necessary for my usage.

### The C# code for enumerating an A/D group

Lets just look at the code:

DirectoryEntry dir = new DirectoryEntry(“WinNT://localhost/Administrators”);  
Console.WriteLine(“Enumerating users”);  
foreach (object obj in (IEnumerable)dir.Invoke(“members”))  
{  
DirectoryEntry user = new DirectoryEntry(obj);  
Console.WriteLine(user.Path);  
}  
Console.WriteLine(“Done”);  
Console.ReadLine();

Running this on my computer outputs the following:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SynchronizeADuserstoNAV_7E8F/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/SynchronizeADuserstoNAV_7E8F/image_2.png)

The next step is, to get the SID’s for each user – now you would think that there should be an SID function on the user object returning a string, but unfortunately it isn’t that simple. Luckily somebody invented the Internet – and luckily somebody was kind enough to post some information for us to use.

Have a look at [http://www.netomatix.com/GetUserSid.aspx](http://www.netomatix.com/GetUserSid.aspx "http://www.netomatix.com/GetUserSid.aspx") where I found the following code, which seems to work for the purpose.

On the user object you have a property collection. One of these properties is called objectSid, which is a byte\[\] and that can be transformed into a SID string using the following function:

private static string ConvertByteToStringSid(Byte\[\] sidBytes)  
{  
StringBuilder strSid = new StringBuilder();  
strSid.Append(“S-“);  
try  
{  
// Add SID revision.  
strSid.Append(sidBytes\[0\].ToString());  
// Next six bytes are SID authority value.  
if (sidBytes\[6\] != 0 || sidBytes\[5\] != 0)  
{  
string strAuth = String.Format  
(“0x{0:2x}{1:2x}{2:2x}{3:2x}{4:2x}{5:2x}”,  
(Int16)sidBytes\[1\],  
(Int16)sidBytes\[2\],  
(Int16)sidBytes\[3\],  
(Int16)sidBytes\[4\],  
(Int16)sidBytes\[5\],  
(Int16)sidBytes\[6\]);  
strSid.Append(“-“);  
strSid.Append(strAuth);  
}  
else  
{  
Int64 iVal = (Int32)(sidBytes\[1\]) +  
(Int32)(sidBytes\[2\] << 8) +  
(Int32)(sidBytes\[3\] << 16) +  
(Int32)(sidBytes\[4\] << 24);  
strSid.Append(“-“);  
strSid.Append(iVal.ToString());  
}

        // Get sub authority count…  
int iSubCount = Convert.ToInt32(sidBytes\[7\]);  
int idxAuth = 0;  
for (int i = 0; i < iSubCount; i++)  
{  
idxAuth = 8 + i \* 4;  
UInt32 iSubAuth = BitConverter.ToUInt32(sidBytes, idxAuth);  
strSid.Append(“-“);  
strSid.Append(iSubAuth.ToString());  
}  
}  
catch (Exception ex)  
{  
return “”;  
}  
return strSid.ToString();  
}

Using this function, you can now write out the SID’s for each user:

DirectoryEntry dir = new DirectoryEntry(“WinNT://localhost/Administrators”);  
Console.WriteLine(“Enumerating users”);  
foreach (object obj in (IEnumerable)dir.Invoke(“members”))  
{  
DirectoryEntry user = new DirectoryEntry(obj);  
Console.WriteLine(user.Path);  
System.DirectoryServices.PropertyCollection col = user.Properties;  
byte\[\] sidBytes = col\[“objectSid”\].Value as byte\[\];  
if (sidBytes != null)  
{  
string strSid = ConvertByteToStringSid(sidBytes);  
if (!string.IsNullOrEmpty(strSid))  
{  
Console.WriteLine(“SID=” + strSid);  
}  
}  
}  
Console.WriteLine(“Done”);  
Console.ReadLine();

Of course we are not in the business of writing SID’s for users in a console application, but you should now have the building blocks for creating whatever mechanism to add A/D users to NAV, exposing the codeunit we talked about at first and then calling these web services functions from the console application.

Good luck

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
