---
layout: post
title: "Edit In Excel – Part 4 (out of 4)"
date: 2008-11-08 20:46:49
categories: ["Archive"]
tags: ["Automation", "C#", "Client Tier", "COM", "Data", "Excel", "Multiple", "NAV 2009", "Web Services"]
permalink: /2008/11/08/edit-in-excel-part-4-out-of-4/
---

If you haven’t read [part 3](http://blogs.msdn.com/freddyk/archive/2008/11/08/edit-in-excel-part-3-out-of-4.aspx), [part 2](http://blogs.msdn.com/freddyk/archive/2008/11/07/edit-in-excel-part-2-out-of-4.aspx) (and [part 1](http://blogs.msdn.com/freddyk/archive/2008/11/07/edit-in-excel-part-1-out-of-4.aspx)), you should do so before continuing here.

We have seen how to put code inside Excel, using VSTO and connect to NAV 2009 Web Services. We have seen how to add this to a table inside Excel and how to write data back to NAV through Web Services. We can delete, add and modify records in Excel and we can even do so with both Customers, Vendors and Items. We have added support for NAV filters, error handling and the only thing we are missing to have a very good experience is integrating the whole thing into NAV.

So we better do that now!

### Disclaimer

Most of the people reading this are probably superior AL coders compared to me. I really only started coding in AL last year and I am still struggling to understand how things works. So, if I do something stupid in AL – just tell me in a comment how to do it smarter – thanks.

### Actions in NAV 2009

What we want to do is

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart4outof4_CF21/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart4outof4_CF21/image_2.png)

Add an action to the menu, which launches Excel, reads the same data as we have in the list place and allows the user to modify that data.

But we need to start in a different area – we need a COM object, which our action can invoke.

### Creating a COM object

First of all we will create a COM object, which contains one function.

`public void EditInExcel(string page, string view)`

I do think there are a number of tutorials that explains how to do this, so I will run over the steps very quickly.

1.  In the same solution as the NAVTemplate, create a new project – type Class Library and name the project NAVEditInExcel
2.  Rename class1.cs to NAVEditInExcel.cs – and say Yes to the question whether you want to rename the class as well.
3.  Select Properties on the project (not the solution)
    1.  On the Build tab, set the output path to `..NAVTemplatebinDebug` in order to share the output path the the Excel Spreadsheet
    2.  On the Build events tab, we need to register the COM object to make it visible to NAV. Add the following Post Build Event: `C:\Windows\Microsoft.NET\Framework\v2.0.50727\regasm NAVEditInExcel.dll /codebase /tlb`
    3.  On the Signing tab, check the Sign the Assembly checkbox and select New key in the combo box, name the key and protect it with a password if you fancy.
4.  Open the AssemblyInfo.cs (under Properties in the Solution Explorer)
    1.  Add `using system;` to the using statements
    2.  Add `[assembly: CLSCompliant(true)]` under the line with `[assembly: ComVisible(false)]`.
5.  Open the source for the NavEditInExcel.cs
    1.  Add `using System.Runtime.InteropServices;` to the using statements
    2.  Create an Interface and change the class to be:

```
[ComVisible(true)]
[Guid("A2C51FC8-671E-4135-AD27-48EDC491E76E"), InterfaceType(ComInterfaceType.InterfaceIsDual)]
public interface INAVEditInExcel
{
void EditInExcel(string page, string view);
}
```

```
[ComVisible(true)]
[Guid("233E0C7F-2276-4142-929C-D6BA8725D7B4"), ClassInterface(ClassInterfaceType.None)]
public class NAVEditInExcel : INAVEditInExcel
{
public void EditInExcel(string page, string view)
{
```

        `// Code goes here…`    

```
}
}
```

Now you should be able to build the COM object and see it inside NAV when adding a variable of type automation.

### Adding the Action in NAV

Open up the Classic Client and design the Customer List Place.

Insert an Action on the Customer List Place called Edit In Excel and edit the code for that (btw. the Image Name for the Excel icon is **Excel**)

In the code for that Action – create a local variable called NAVEditInExcel of type Automation and select the NAVEditInExcel.NAVEditInExcel COM object to use and add the following code:

```
CREATE(NAVEditInExcel, TRUE, TRUE);
NAVEditInExcel.EditInExcel(TABLENAME, GETVIEW(TRUE));
```

That’s it on the NAV side, but of course we didn’t make all the code necessary in the COM object yet.

If you try to launch the Action you will be met by the security dialog

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart4outof4_CF21/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart4outof4_CF21/image_4.png)

Which you of course want to hit always allow to – else you will get this dialog every time you say Edit In Excel.

BTW – If you hit Never allow – you will never be allowed to Edit in Excel – not until you have deleted your PersonalizationStore.xml at least.

### Completing the COM object

Having that hooked up we really just need to launch that damn spreadsheet with the right parameters.

We need to add 3 .NET references to the COM object:

-   `System.Windows.Forms`
-   `Microsoft.Office.Interop.Excel`
-   `Microsoft.VisualStudio.Tools.Applications.ServerDocument.v9.0`

and the following 3 using statements:

```
using Microsoft.VisualStudio.Tools.Applications;
using System.Windows.Forms;
using System.Reflection;
```

and last but not least, add the following EditInExcel method:

```
public void EditInExcel(string page, string view)
{
try
{
// Copy the original template to a new template using the page name as name!
string originalTemplate = System.IO.Path.Combine(System.IO.Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location), "NAVTemplate.xltx");
if (!System.IO.File.Exists(originalTemplate))
{
MessageBox.Show(string.Format("The template: '{0}' cannot be found!", originalTemplate), "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
return;
}
string template = System.IO.Path.Combine(System.IO.Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location), page + ".xltx");
while (System.IO.File.Exists(template))
{
try
{
System.IO.File.Delete(template);
}
catch (System.IO.IOException)
{
if (MessageBox.Show(string.Format("The template: '{0}' is locked, cannot open spreadsheet", template), "Error", MessageBoxButtons.RetryCancel, MessageBoxIcon.Error) != DialogResult.Retry)
{
return;
}
}
}
System.IO.File.Copy(originalTemplate, template);
```

        

```
// Open the new template and set parameters
ServerDocument serverDoc = new ServerDocument(template);
CachedDataHostItem host = serverDoc.CachedData.HostItems[0];
host.CachedData["page"].SerializeDataInstance(page);
host.CachedData["view"].SerializeDataInstance(view);
serverDoc.Save();
serverDoc.Close();
```

        

```
// Create a new spreadsheet based on the new template
Microsoft.Office.Interop.Excel.ApplicationClass excelApp = new Microsoft.Office.Interop.Excel.ApplicationClass();
excelApp.Visible = true;
excelApp.Workbooks.Add(template);
```

        

```
// Erase template
System.IO.File.Delete(template);
}
catch (Exception e)
{
System.Windows.Forms.MessageBox.Show(e.Message, "Critical error", MessageBoxButtons.OK, MessageBoxIcon.Error);
}
}
```

This method really does 4 things:

1.  Copy the NAVTemplate.xltx to a new template called Customer.xltx (that is if the page name is customer) which is a temporary template
2.  Open the template as a ServerDocument and set the parameters
3.  Ask Excel to create a new spreadsheet based on this template
4.  Erase the template

That was easy!

Oh – there is one things I forgot to say, you need to specify in the Excel Spreadsheet that the page and view variables are cached data (meaning their value are saved with Excel) – this is done by adding an attribute to the variables:

```
[Cached]
public string page;
```

```
[Cached]
public string view;
```

Having done this, you can open the spreadsheet as a Serverdocument, get and set the value of these parameters and save the document again, pretty sweet way of communicating parameters to Excel or Word – and this will definitely come in handy later.

### Adding the action other pages

Having verified that we can edit customers in Excel we can now add the same action as above to the Vendor and the Item List Places.

You can either follow the same steps as above – or you can copy the action and paste it on the other List Places.

Note that you cannot build the Visual Studio solution while you have NAV 2009 open. When NAV loads the COM object, it keeps a reference to it until you close NAV.

Last but not least – this should work from the classic client as well – if you want to add the functionality there – I haven’t tried it.

### That’s it folks

That completes the Edit In Excel in Part 1 through 4

As always, there is absolutely no warranty that this code works for the purpose you need it to, but these posts show how to do some things and feel free to use pieces of this or use it as a base to build your own solution using Excel – the code is free – a referral to my blog is always a good way of acknowledgement.

I hope you can make it work, that it is useful and you can download the final solution here: [http://www.freddy.dk/NAVTemplate\_Final.zip](http://www.freddy.dk/NAVTemplate_Final.zip "http://www.freddy.dk/NAVTemplate_Final.zip")

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
