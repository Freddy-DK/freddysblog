---
layout: post
title: "Edit In Excel R2 – Part 2 (out of 2) – the final pieces"
date: 2009-12-11 11:24:45
categories: ["Archive"]
tags: ["C#", "Client Tier", "COM", "Deployment", "Excel", "Service Tier", "Setup", "Web Services"]
permalink: /2009/12/11/edit-in-excel-r2-part-2-out-of-2-the-final-pieces/
---

It is time to collect the pieces.

The full Edit In Excel R2 solution looks like this

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_2.png)

Slightly more complicated than the first version – but let me try to explain the pieces

**NAVEditInExcel** is the COM object, which we use from within NAV. This actually hasn’t changed a lot, the only small change is, that the EditInExcel method now takes a base URL, a company, a page and a view (compared to just a page and a view earlier).  
**NAVPageDynamicWebReference** is the Dynamic Web Reference class and the NAVPageServiceHelper class – described [here](http://blogs.msdn.com/freddyk/archive/2009/12/04/dynamic-references-to-nav-page-web-services-in-c-take-2.aspx).  
**NAVPageFieldInfo** contains the NAVFieldInfo class hierarchy for handling type weak pages, described [here](http://blogs.msdn.com/freddyk/archive/2009/12/04/dynamic-references-to-nav-page-web-services-in-c-take-2.aspx) and used in the Conflict resolution dialog [here](http://blogs.msdn.com/freddyk/archive/2009/12/09/conflict-resolution-when-working-with-web-services.aspx).  
**NAVPageMergeForm** is the conflict resolution dialog, described [here](http://blogs.msdn.com/freddyk/archive/2009/12/09/conflict-resolution-when-working-with-web-services.aspx).  
**NAVTemplate** is the actual Excel Add-In which of course now makes use of Dynamic Page References and conflict resolution. It really haven’t changed a lot since the version described [here](http://blogs.msdn.com/freddyk/archive/2009/04/13/edit-in-excel-r2-part-1-out-of-2.aspx) – the major change is the pattern for handling conflict resolution.  
**EditInExcel Setup** is the Client Setup program, this setup program needs to be run on all Clients  
**EditInExcelDemo** is the Server Setup program, this setup program contains the Client Setup msi and places it in the ClientSetup folder for the ComponentHelper (which you can read about here) to autodeploy to clients. This setup also contains the .fob with the EditInExcel objects.

### The Client Setup Program

Lets have a closer look at the Client Setup Program

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_thumb_2.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_6.png)

This setup project includes primary output from the COM component and the Excel Add-in and calculated dependencies from that.

Note, that when deploying add-ins you have to add the .vsto and the .manifest files to the setup project yourself, the dependency finder doesn’t discover those. Also note, that all the vsto runtime dll’s etc are excluded from the install list, as we do not want to copy those DLL’s.

Instead I have built in a Launch condition for VSTO runtime 3.0, which is done in 2 steps:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_thumb_4.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_10.png)

First a Search on the Target Machine for component ID {AF68A0DE-C0CD-43E1-96DD-CBD9726079FD} (which is the component installation ID for VSTO 3.0 Runtime) and a launch condition stating that that search needs to return TRUE – else a message will appear with a URL for installing VSTO, which is:

[http://www.microsoft.com/downloads/details.aspx?FamilyId=54EB3A5A-0E52-40F9-A2D1-EECD7A092DCB&displaylang=en](http://www.microsoft.com/downloads/details.aspx?FamilyId=54EB3A5A-0E52-40F9-A2D1-EECD7A092DCB&displaylang=en "http://www.microsoft.com/downloads/details.aspx?FamilyId=54EB3A5A-0E52-40F9-A2D1-EECD7A092DCB&displaylang=en")

One more thing needed in the Client Setup program is to register the COM object. Now the Setup actually has a property you can set, indicating that the object should be registered as COM, but I couldn’t get that to work, so I added custom install actions to the NAVEditInExcel COM object:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_thumb_5.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2Part2outof2thefinalpieces_A079/image_12.png)

and the code for the class, which is called by the installer looks like:

\[RunInstaller(true)\]  
public partial class RegasmInstaller : Installer  
{  
public RegasmInstaller()  
: base()  
{  
}

    public override void Commit(IDictionary savedState)  
{  
base.Commit(savedState);  
Regasm(false);  
}

    public override void Rollback(IDictionary savedState)  
{  
base.Rollback(savedState);  
}

    public override void Uninstall(IDictionary savedState)  
{  
base.Rollback(savedState);  
Regasm(true);  
}

    private void Regasm(bool unregister)  
{  
string parameters = “/tlb /codebase”;  
if (unregister)  
parameters += ” /unregister”;  
string regasmPath = RuntimeEnvironment.GetRuntimeDirectory() + @”regasm.exe”;  
string dllPath = this.GetType().Assembly.Location;  
if (!File.Exists(regasmPath))  
throw new InstallException(“Registering assembly failed”);  
if (!File.Exists(dllPath))  
return;

        Process process = new Process();  
process.StartInfo.CreateNoWindow = true;  
process.StartInfo.UseShellExecute = false; // Hides console window  
process.StartInfo.FileName = regasmPath;  
process.StartInfo.Arguments = string.Format(“”{0}” {1}”, dllPath, parameters);  
process.Start();

        // When uninstalling we need to wait for the regasm to finish,  
// before continuing and deleting the file we are unregistering  
if (unregister)  
{  
process.WaitForExit(10000);  
try  
{  
System.IO.File.Delete(System.IO.Path.ChangeExtension(dllPath, “tlb”));  
}  
catch  
{  
}  
}  
}  
}

All of the above is captured in the NAVEditInExcelR2.msi – which is the output from the Edit In Excel Setup project. Running this .msi on a client will check pre-requisites, install the right DLL’s, register the COM and you should be good to go.

### The Server Setup Program

The Server Setup program actually just needs to place the Client Setup Program in a ClientSetup folder and the .fob (NAV Objects) in the ServerSetup folder.

There are no pre-requisites, no actions no nothing – just copy the files.

After Copying the files on the Server – you need to import the .fob, run the setup code unit and you should be good to go.

Note, that this requires ComponentHelper1.03 (which you can read about [here](http://blogs.msdn.com/freddyk/archive/2009/12/09/auto-deployment-of-client-side-components-take-2.aspx) and download [here](http://www.freddy.dk/ComponentHelper1.03.zip)) to run.

### Wrapping up…

So, what started out as being a small garage project, ended up being somewhat more complicated and way more powerful. It runs with Office 2007 and Office 2010 (even though you cannot modify the project when Office 2010 beta2 is installed) and even though you might not need the actual Edit In Excel functionality – there are pieces of this that can be used for other purposes.

The source for the entire thing can be downloaded [here](http://www.freddy.dk/EditInExcelR2.zip) and the EditInExcel Demo msi can be downloaded [here](http://www.freddy.dk/EditInExcelDemo.zip).

Happy holidays

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
