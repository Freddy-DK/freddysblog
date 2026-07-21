---
layout: post
title: "Edit In Excel R2 – Part 1 (out of 2)"
date: 2009-04-13 11:22:33
categories: ["Archive"]
tags: ["Automation", "C#", "COM", "Excel", "NAV 2009", "Temporary File", "Web Services", "XML"]
permalink: /2009/04/13/edit-in-excel-r2-part-1-out-of-2/
---

This post assumes that you have read the 4 step walkthrough of how to build the Edit In Excel demo from November 2008. You can find the parts here: [Part 1](http://blogs.msdn.com/freddyk/archive/2008/11/07/edit-in-excel-part-1-out-of-4.aspx), [Part 2](http://blogs.msdn.com/freddyk/archive/2008/11/07/edit-in-excel-part-2-out-of-4.aspx), [Part 3](http://blogs.msdn.com/freddyk/archive/2008/11/08/edit-in-excel-part-3-out-of-4.aspx), [Part 4](http://blogs.msdn.com/freddyk/archive/2008/11/09/edit-in-excel-part-4-out-of-4.aspx) and the [Bug Fix](http://blogs.msdn.com/freddyk/archive/2009/04/12/edit-in-excel-bug-fix-and-r2.aspx).

In this post I will talk about what is needed in order to be able to save an Excel spreadsheet on a local disc, edit it offline and then submit your changes later.

My first assumption was that this was just a few small changes, but I should learn more…

### Goal

The success scenario is the one where a NAV user decides to take the customer table offline in Excel to do modifications. The Excel spreadsheet is saved to a local hard drive and edited while not connected to the network.

When the user reconnects to the network, he can submit his changes, which will be send to NAV through Web Services and validated according to business rules.

### **The location of NAVTemplate.vsto**

Let’s just try to save the spreadsheet in our Documents folder and see what happens if we open the spreadsheet. Not surprisingly we get an error telling us that it was unable to locate NAVTemplate.vsto

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2_134C3/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2_134C3/image_4.png)

An immediate solution is to save the spreadsheet next to the .vsto and then it seems to work better, but the spreadsheet is not attached to NAV anymore, we cannot save changes and reload crashes with an exception.

The .vsto file is our deployment manifest, which is our managed code extension to the spreadsheet and of course this is needed in order for the spreadsheet to work.

You can read more about the architecture of Document-Level Customizations here:

[http://msdn.microsoft.com/en-us/library/zcfbd2sk.aspx](http://msdn.microsoft.com/en-us/library/zcfbd2sk.aspx "http://msdn.microsoft.com/en-us/library/zcfbd2sk.aspx")

Looking at the ServerDocument interface,which we use in the NAVEditInExcel solution, it has a property called [DeploymentManifestUri](http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.tools.applications.serverdocument.deploymentmanifesturl.aspx), which is the location of the .vsto file. Adding the following line to the NAVEditInExcel project

serverDoc.DeploymentManifestUrl = new Uri(System.IO.Path.Combine(System.IO.Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location), “NAVTemplate.vsto”));

will cause the document to be created with an absolute reference to the NAVTemplate.vsto, and this will solve the problem with the spreadsheet not being able to locate the .vsto. In fact if this is a network location, it should even be possible to send the Excel spreadsheet to somebody else who should be able to modify the document and post the changes.

When doing this I decided to make another change in the NAVEditInExcel project. As you know it creates a temporary template in the same location as the DLL and the VSTO – and then it deletes it again. This is really not the preferred location of temporary files – we should create temporary files in the Temp folder so we change the line setting the template name to:

string template = System.IO.Path.Combine(System.IO.Path.GetTempPath(), page + “.xltx”);

since we don’t need to have the .xltx next to the .vsto anymore.

### What do we need to save?

Members of the Sheet1 class marked with \[Cached\] will be saved together with the document – we know that from the way we transfer the page name and the current view to the spreadsheet. Thinking about it, the only things we need to save together with the document is the dataTable (which at any time contains the changes made by the user) and the objects collection (which are the objects returned from the Web Service).

The DataTable class implements IXmlSerializable and the objects collection is (as we know) an array of objects returned from a web Service provider and since these objects where send over the wire from a Web Service they of course also implements IXmlSerializable.

The fields collection cannot be saved, since the NAVFieldInfo class uses the PropertyInfo class, which cannot be serialized. The Service connection of course cannot be serialized either – nor can the datalist class – as the matter of fact, the datalist class shouldn’t be a member at all, it should be moved to the AddDataToExcel method as a local variable.

Problem now is, that if we just mark the dataTable and objects members with \[Cached\] we need to initialize them in the NAVEditInExcel project and there is no way we can instantiate the objects collection at that time with the right type.

A little more on cached data objects in Office documents

[http://msdn.microsoft.com/en-us/library/ms178808(VS.80).aspx](http://msdn.microsoft.com/en-us/library/ms178808\(VS.80\).aspx "http://msdn.microsoft.com/en-us/library/ms178808(VS.80).aspx")

and then how to programmatically add members to the cached objects

[http://msdn.microsoft.com/en-us/library/48b7eyf3(VS.80).aspx](http://msdn.microsoft.com/en-us/library/48b7eyf3\(VS.80\).aspx "http://msdn.microsoft.com/en-us/library/48b7eyf3(VS.80).aspx")

using this knowledge gives us the following two lines, we want to add after we have instantiated the dataTable and the objects array.

// Add dataTable and objects to the Caching collection  
this.StartCaching(“dataTable”);  
this.StartCaching(“objects”);

Now we just need to determine that the spreadsheet was loaded from a file (and not started from NAV) and then act differently.

### Refactoring some code

We need to do a little refactoring of code in order to make things work. In the existing solution the PopulateFieldsCollection method creates the fields collection, but it also creates an empty dataTable class. Since we now store the dataTable class and not the fields collection we need as the first thing in the new spreadsheet to create the fields collection (a lot of things is depending on this). This is the new PopulateFieldsCollection (and AddField):

/// <summary>  
/// Populate Fields Collection with NAVFieldInfo for all properties in the record  
/// Should works with any NAV 2009 Page exposed as WebService  
/// </summary>  
/// <param name=”objType”>Type of Object (typeof(Customer), typeof(Vendor), …)</param>  
/// <param name=”fieldsType”>Type of the Enum holding the property names</param>  
private void PopulateFieldsCollection(Type objType, Type fieldsType)  
{  
this.fields = new List<NAVFieldInfo>();

    // Key property is not part of the Enum  
// Add it manually as the first field  
AddField(“Key”, objType);

    // Run through the enum and add all fields  
foreach (string field in Enum.GetNames(fieldsType))  
{  
AddField(field, objType);  
}  
}

/// <summary>  
/// Create a NAVFieldInfo for a field  
/// </summary>  
/// <param name=”field”>Field name</param>  
/// <param name=”objType”>Type of Object in which the field is (typeof(Customer), typeof(Vendor), …)</param>  
private void AddField(string field, Type objType)  
{  
field = NAVFilterHelper.VSName(field);  
PropertyInfo pi = objType.GetProperty(field, System.Reflection.BindingFlags.Public | System.Reflection.BindingFlags.Instance);  
if (pi != null)  
{  
NAVFieldInfo nfi = NAVFieldInfo.CreateNAVFieldInfo(field, pi, objType.Namespace);  
if (nfi != null)  
{  
// If we encounter unknown Field Types, they are just ignored  
this.fields.Add(nfi);  
}  
}  
}

The Load function in the old spreadsheet does a number of things. It populates the Fields collection, loads the data, populates the dataTable and add the dataTable to the spreadsheet.

We will remove and create one function called LoadDataTable, which will create a dataTable (based on the fields collection), load the data from Web Services and populate the dataTable.

/// <summary>  
/// Load Records from NAV via Web Services  
/// </summary>  
private void LoadDataTable()  
{  
// Create Data Table object based on fields collection  
this.dataTable = new DataTable(this.page);  
foreach (NAVFieldInfo nfi in this.fields)  
{  
this.dataTable.Columns.Add(new DataColumn(nfi.field, nfi.fieldType));  
}

    SetFilters(this.view);  
this.objects = this.service.ReadMultiple();  
// Populate dataTable with data  
foreach (object obj in this.objects)  
{  
DataRow dataRow = this.dataTable.NewRow();  
foreach (NAVFieldInfo nfi in this.fields)  
{  
dataRow\[nfi.field\] = nfi.GetValue(obj);  
}  
this.dataTable.Rows.Add(dataRow);  
}  
this.dataTable.AcceptChanges();  
}

As you can see, this is pieces of the PopulateFieldsCollection, Load and PopulateDataTable functions – as the matter of fact, you can delete the PopulateDataTable function as well. BTW the AcceptChanges call was moved from AddDataToExcel it needs to be together with the code populating the dataTable.

Right now my startup code in my spreadsheet has changed to

if (this.service != null)  
{  
this.service.UseDefaultCredentials = true;

    // Create Fields collection  
this.PopulateFieldsCollection(this.service.GetObjectType(), this.service.GetFieldsType());

    Application.ScreenUpdating = false;

    bool loadData = this.ListObjects.Count == 0;  
if (!loadData)  
{  
MessageBox.Show(“Spreadsheet loaded from disc”);  
}if (loadData)  
{  
// Load Data into dataTable from Web Services  
this.LoadDataTable();  
// Add dataTable to Excel Spreadsheet  
this.AddDataTableToExcel();

        // Add dataTable and objects to the Caching collection  
this.StartCaching(“dataTable”);  
this.StartCaching(“objects”);  
}

    Application.ScreenUpdating = true;  
}

from just invoking Load() before.

So we populate the fields collection and then we check whether or not there is a ListObject in the document (remember the Controls collection was empty). If this is the case we must do something (for now we just display a messagebox).

If we are called from NAV (loadData becomes true) we will load the data and call AddDataTableToExcel (renamed from AddDataToExcel) and that should work.

If we try to compile now, we will see that the Reload() method uses Load() as well. We need to change Reload to

/// <summary>  
/// Reload data from NAV (delete old dataTable, and load new data)  
/// </summary>  
internal void Reload()  
{  
Application.ScreenUpdating = false;

    // Remove List Object  
if (this.dataTable != null)  
this.Controls.RemoveAt(0);  
else  
this.ListObjects\[1\].Delete();

    // Load Data into dataTable from Web Services  
this.LoadDataTable();  
// Add dataTable to Excel Spreadsheet  
this.AddDataTableToExcel();

    // If this reload was in fact a reattach of the spreadsheet, start caching dataTable and objects again  
if (!this.IsCached(“dataTable”))  
{  
// Add dataTable and objects to the Caching collection  
this.StartCaching(“dataTable”);  
this.StartCaching(“objects”);  
}

    Application.ScreenUpdating = true;  
}

Note that we remove the old listobject in two different ways based on whether or not dataTable is set. dataTable is null if the spreadsheet has been detached from NAV – I will touch more upon that later. This is also the reason why we restart Caching the dataTable and objects if in fact this was a reattach.

The solution should work as before now – the only major difference is, that if you save the spreadsheet on the disc and try to load it again it should not give an exception telling you, that you cannot overlap a table with another table, instead it should give you something like:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2_134C3/image_thumb_3.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelR2_134C3/image_10.png)

This is of course not the final solution, but it shows us that we are on the right track.

### Restoring the “State” of the spreadsheet

To make a long story short, the lines we need in order to restore the managed list object are:

// Remove Non-VSTO List Object  
this.ListObjects\[1\].Delete();  
// Create a new VSTO ListObject – data bound  
this.AddDataTableToExcel();

Meaning that we remove the unmanaged ListObject and we add the managed ListObject to Excel, seems pretty easy. But what if the document is saved on disc and you add a field to the Customer Page (+ update your web reference and recompile your NAVTemplate) then the managed extension assembly doesn’t match the saved spreadsheet anymore and the above logic wouldn’t work.

In many cases we could just say that we don’t care – but given the ability to save spreadsheets that are connected to Web Services and reload data adds another dimension to the entire Excel thing. You can have spreadsheets that contain a lot of other things than your dataTable and you might not be pleased with the fact that you loose the NAV Web Service connection if this happens.

I decided to build in a way of determining this and give the user a couple of options:

// If the spreadsheet was detached already – just ignore  
if (this.IsCached(“dataTable”))  
{  
// We have loaded a saved spreadsheet with data  
// Check that the VSTO assembly (fields) matches the spreadsheet  
bool fieldsOK = this.dataTable.Columns.Count == this.fields.Count;  
if (fieldsOK)  
{  
for (int i = 0; i < this.fields.Count; i++)  
if (this.dataTable.Columns\[i\].Caption != this.fields\[i\].field)  
fieldsOK = false;  
}  
if (!fieldsOK)  
{  
// Schema mismatch – cannot link back to NAV  
switch (MessageBox.Show(“Customer Card definition has changed since this spreadsheet was save. Do you want to re-establish link, reload data and loose changes?”, “Error”, MessageBoxButtons.YesNoCancel))  
{  
case DialogResult.Cancel:  
// Quit  
Application.Quit();  
break;  
case DialogResult.Yes:  
// Remove Non-VSTO List Object  
this.ListObjects\[1\].Delete();  
// Signal reload data and reestablish link  
loadData = true;  
                this.StopCaching(“dataTable”);  
this.StopCaching(“objects”);                break;  
case DialogResult.No:// Detach spreadsheet from NAV  
              this.dataTable = null;  
              this.objects = null;  
              this.StopCaching(“dataTable”);  
              this.StopCaching(“objects”);  
                break;  
}

    }  
else  
{  
// Remove Non-VSTO List Object  
this.ListObjects\[1\].Delete();  
// Create a new VSTO ListObject – data bound  
this.AddDataTableToExcel();  
}  
}

3 options – **Cancel** quits the spreadsheet open command and no harm has been done, **Yes** removes the old table and sets the loadData to true (meaning that the spreadsheet will reload data as if it was opened from NAV) and **No** will detatch the spreadsheet from NAV (setting the dataTable to null and stop caching dataTable and objects). Note that if you ever press Reload it will re-attach the spreadsheet and load data from NAV again (the reason for checking whether the dataTable was null in Reload).

Yes or No will of course not touch the original spreadsheet and you can always save in a new name.

BTW – we need one simple check in Save() as well

/// <summary>  
/// Save Changes to NAV via Web Service  
/// </summary>  
internal void Save()  
{  
if (this.dataTable == null)  
{  
MessageBox.Show(“Spreadsheet was detached from NAV, cannot perform Save!”);  
}  
else if (DoSave())  
{  
Reload();  
}  
}

The only extra tihing you need is to add the following line right before the return statement in DoSave():

this.dataTable.AcceptChanges();

Without this line you cannot edit the data, save changes to NAV, edit more data and then save locally.

### Done

We are done – we now have a template you can use for modifying NAV data. You can save the spreadsheet locally and modify it while you are offline and later, when you come back online you can push your changes to NAV and this is when validation happens.

As usual you can download the project here [http://www.freddy.dk/NAVTemplateR2.zip](http://www.freddy.dk/NAVTemplateR2.zip) – there are no changes to what’s needed in NAV, so you should use the objects from the original Edit In Excel walkthrough inside C/AL.

BTW. If you decide to use the source, you probably need to right-click the Web References and invoke Update Web Reference to make sure that the Web Reference matches your Customer, Item and Vendor Card Pages.

### Part 2?

So why is this a part 1 out of  2 you might ask yourself right now?

Reason is that I want to create a better conflict resolution in the spreadsheet. With the solution we have build now, any change to a record by anybody will cause you to loose your changes in Excel. Wouldn’t it be nice if we were able to determine that we only changed the location code in Excel, so just because somebody changed the credit limit on a couple of customers, we should still be able to re-apply our changes without having to type them in again.

This is what R2 step 2 is all about – a better conflict resolution mechanism.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
