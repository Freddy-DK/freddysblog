---
layout: post
title: "Edit In Excel – Part 3 (out of 4)"
date: 2008-11-08 13:05:41
categories: ["Archive"]
tags: ["C#", "COM", "Data", "Excel", "Multiple", "NAV 2009", "Web Services"]
permalink: /2008/11/08/edit-in-excel-part-3-out-of-4/
---

If you haven’t read [part 2](http://blogs.msdn.com/freddyk/archive/2008/11/07/edit-in-excel-part-2-out-of-4.aspx) (and [part 1](http://blogs.msdn.com/freddyk/archive/2008/11/07/edit-in-excel-part-1-out-of-4.aspx)), you should do so before continuing here.

In Part 1 and 2, we have seen how easy it is to add a Web Service Reference inside Excel, and use it to get Data. In Part 2 we even had the ability to modify data and send this back to NAV. The original intend was that part 3 would be all about integrating this to NAV on the Client side and part 4 would be to make this loosely coupled – but I have changed my mind on this.

Part 3 will remove the direct dependency on the Customer Web Service from most of the code – and thus allowing us to modify both Customer, Vendor or Item data in Excel with very few tweaks to the code. Also I will add support for parsing a filter string and applying this to the list. I will also add error handling of the save process.

Part 4 will then be to add the Action in NAV and hook that up to set the parameters in Excel.

I will still post the source of the original loosely coupled XMLHTTP based Edit In Excel, but I will not use it for anything.

To prepare ourselves for part 4 we need the following variables:

/// <summary>  
/// Page which is going to be used for Edit In Excel  
/// Customer, Vendor, Item, etc…  
/// The card page for this record needs to be exposed as webservice with that name  
/// </summary>  
string page;

/// <summary>  
/// The filters to apply (format: GETVIEW(TRUE))  
/// Sample: “SORTING(No.) WHERE(Balance (LCY)=FILTER(>10,000))”  
/// </summary>  
string view;

These are the parameters, which we in part 4 will transfer values to Excel in – for now we will build the Spreadsheet to use those.

BTW – I changed the Project name from CustomerTemplate to NAVTemplate (actually I created a new project and copied over some of the files and changed the namespace).

Then I have moved the service connection initialization away from Load – and into Sheet\_Startup, the new Sheet1\_Startup code looks like this

private void Sheet1\_Startup(object sender, System.EventArgs e)  
{  
switch (this.page)  
{  
case “Customer”:  
this.service = new CustomerRef.Customer\_Service();  
break;  
case “Vendor”:  
this.service = new VendorRef.Vendor\_Service();  
break;  
case “Item”:  
this.service = new ItemRef.Item\_Service();  
break;  
default:  
MessageBox.Show(string.Format(“Page {0} is not setup for Edit In Excel. Please contact your system administrator”, this.page), “Microsoft Dynamics NAV”, MessageBoxButtons.OK, MessageBoxIcon.Error);  
break;  
}  
if (this.service != null)  
{  
this.service.UseDefaultCredentials = true;  
Load();  
}  
}

and I have added references to all 3 services.

This is the only place I have a switch on the page – the rest of the code is made to work with all – but wait… – how is that possible?

Service Connection classes code generated from Visual Studio doesn’t implement any common interface and we cannot change the code generated proxy classes (or rather – we don’t want to). We can, however, add something to the existing service. Looking at the code generated proxy class we will notice that the Customer\_Service class is defined as a partial class – meaning that we can actually write another part of the class just by creating a new class (with the keyword partial)

Looking through my code I really need the Customer\_Service to implement an interface like this:

public interface INAVService  
{  
bool UseDefaultCredentials {get; set; }  
System.Net.ICredentials Credentials {get; set; }

    object\[\] ReadMultiple();  
void Update(object obj);  
void Create(object obj);  
bool Delete(string key);

    Type GetFieldsType();  
Type GetObjectType();

    void ClearFilters();  
void AddFilter(string field, string criteria);  
}

Some of these methods are already implemented by all Service Proxy classes and I use this to allow my code to look at the Service Connection via this interface only and the service variable I have in the sheet is actually type INAVService, flip side of this idea is, that for every new Page I want to add – I need to create a class like this:

public partial class Customer\_Service : INAVService  
{  
List<Customer\_Filter> filters;

    #region INAVService Members

    public object\[\] ReadMultiple()  
{  
return this.ReadMultiple(this.filters.ToArray(), null, 0);  
}

    public void Update(object obj)  
{  
Customer customer = (Customer)obj;  
this.Update(ref customer);  
}

    public void Create(object obj)  
{  
Customer customer = (Customer)obj;  
this.Create(ref customer);  
}

    public Type GetObjectType()  
{  
return typeof(Customer);  
}

    public Type GetFieldsType()  
{  
return typeof(Customer\_Fields);  
}

    public void ClearFilters()  
{  
this.filters = new List<Customer\_Filter>();  
}

    public void AddFilter(string field, string criteria)  
{  
Customer\_Filter filter = new Customer\_Filter();  
filter.Field = (Customer\_Fields)Enum.Parse(typeof(Customer\_Fields), field, true);  
filter.Criteria = criteria;  
this.filters.Add(filter);  
}

    #endregion  
}

Not really nice – but it beats having a series of switch statements scattered around in the source files.

So, whenever we want to add a record object type, which we want to be able to Edit In Excel – we add a source file like this (search and replace Customer with <newtype>), we add an extra outcome in the switch statement above and we expose the page to Web Services in NAV 2009.

BTW – In my solution, I have added the classes to the solution in a folder called Services.

### Applying Filters and Load

The Load method now looks like this:

/// <summary>  
/// Load Records from NAV via Web Services  
/// </summary>  
private void Load()  
{  
PopulateFieldsCollection(this.service.GetObjectType(), this.service.GetFieldsType());  
SetFilters(this.view);  
this.objects = this.service.ReadMultiple();  
PopulateDataTable();  
AddDataToExcel();  
}

Note that we ask the Service connection class for the Object Type, the Fields Enum Type and we call the ReadMultiple on the Service Connection (all through the interface we just implemented).

After generating fields collection and the DataTable we call SetFilters – which in effect just parses the view variable (sample: “SORTING(No.) WHERE(Balance (LCY)=FILTER(>10,000))” – without the quotes) and calls AddFilter a number of times (in the sample only once) on the Service Connection Interface.

I added a NAVFilterHelper static class with 3 static helper methods – GetBlock, WSName and VSName.

GetBlock parses the string for a block (a keyword followed by a parentheses with stuff in it) – SORTING(No.) is one and the WHERE clause is another. The FILTER  clause is another block inside the WHERE block.

WSName takes a name like “**Balance (LCY)”** and puts it through name mangling to get the Visual Studio generated identifier name (this is the name used in Enum – **Balance\_LCY**)

VSName takes the enum identifier and removes special characters to get the Property name of the record object (there are no special characters in **Balance\_LCY**)

Confused? – well look at this:

test  &&//(())==??++–\*\*test – is a perfectly good (maybe stupid) field name in NAV

test\_\_x0026\_\_x0026\_\_\_x003D\_\_x003D\_\_x003F\_\_x003F\_\_x002B\_\_x002B\_\_\_x002A\_\_x002A\_test is the same identifier in the xx\_Fields enum (from the WSDL)

test\_\_\_test is the same identifier as property in Visual Studio (the code generated proxy class)

and yes – you can generate fields, which will cause Web Services to fail. In fact, CTP4 (the US version) has an Option field in Customer and Vendor (Check Seperator), where the options causes Customer and Vendor to fail when exposed to Web Services. This special case is fixed for RTM – and the WSName in my sample contains the same name mangling as NAV 2009 RTM, but you can still create field names, which will end up having identical names in VS – and then your WebService proxy won’t work.

WSName and VSName works for my usage – they might not work for all purposes.

There is really nothing fancy about the SetFilters code, but it works for the purpose:

/// <summary>  
/// Parse the view and apply these filters to the Service Connection  
/// </summary>  
/// <param name=”view”>View to parse (from AL: GETVIEW(TRUE))</param>  
private void SetFilters(string view)  
{  
this.service.ClearFilters();  
if (string.IsNullOrEmpty(view))  
return;  
string sorting = NAVFilterHelper.GetBlock(“SORTING”, ref view);  
string where = NAVFilterHelper.GetBlock(“WHERE”, ref view);  
do  
{  
int e = where.IndexOf(“=FILTER”);  
if (e < 0)  
break;  
string field = NAVFilterHelper.WSName(where.Substring(0, e));  
string criteria = NAVFilterHelper.GetBlock(“FILTER”, ref where);  
this.service.AddFilter(field, criteria);

        if (where.StartsWith(“,”))  
where.Remove(0, 1);  
}  
while (true);  
}

Yes, yes – as you of course immediately spotted – this code doesn’t work if you have a field with **\=FILTER** in the field name – so don’t!

The PopulateDataTable and AddDataToExcel methods haven’t changed.

### Save

One thing we didn’t get done in part 2 was error handling. If anybody tried to modify a Location Code to an illegal Location Code and save it back to Excel – you will have noticed that Excel just ignored your request.

Reason for this is, that Excel swallows the Exception, and just ignores it.

So – I have changed the Save() method to:

/// <summary>  
/// Save Changes to NAV via Web Service  
/// </summary>  
internal void Save()  
{  
if (DoSave())  
{  
Reload();  
}  
}

and then created the DoSave() – with most of the content from Save() – but refactored inside one loop with error handling (Abort, Retry, Ignore).

/// <summary>  
/// Delete, Add and Update Records  
/// </summary>  
internal bool DoSave()  
{  
// Run through records marked for delete, create or modify  
DataView dv = new DataView(this.dataTable, “”, “”, DataViewRowState.Deleted | DataViewRowState.Added | DataViewRowState.ModifiedCurrent);  
foreach (DataRowView drv in dv)  
{  
bool retry;  
do  
{  
retry = false;  
try  
{  
if (drv.Row.RowState == DataRowState.Deleted)  
{  
object obj = GetRecordObject((string)drv\[0\]);  
if (obj != null)  
{  
if (!service.Delete((string)drv\[0\]))  
{  
throw new Exception(string.Format(“Unable to delete record”));  
}  
}  
}  
else if (drv.Row.RowState == DataRowState.Added)  
{  
object obj = Activator.CreateInstance(this.service.GetObjectType());  
foreach (NAVFieldInfo nfi in this.fields)  
{  
if (nfi.field != “Key”)  
{  
nfi.SetValue(obj, drv.Row\[nfi.field\]);  
}  
}  
this.service.Create(obj);  
}  
else  
{  
object obj = GetRecordObject((string)drv\[0\]);  
if (obj != null)  
{  
foreach (NAVFieldInfo nfi in this.fields)  
{  
if (nfi.field != “Key”)  
{  
nfi.SetValue(obj, drv\[nfi.field\]);  
}  
}  
this.service.Update(obj);  
}  
}  
}  
catch (Exception e)  
{  
DialogResult reply = MessageBox.Show(string.Format(“{0} {1} {2}nn{3}”, this.dataTable.TableName, this.dataTable.Columns\[1\].Caption, drv\[1\].ToString(), e.Message), “Microsoft Dynamics NAV”, MessageBoxButtons.AbortRetryIgnore, MessageBoxIcon.Error);  
if (reply == DialogResult.Abort)  
return false;  
if (reply == DialogResult.Retry)  
retry = true;  
}  
} while (retry);  
}  
return true;  
}

oh yes, and beside that – you can see that I now use the methods on the Service Connection Interface directly and I do not use the type safe references to Customer – but instead just object. A couple of comments:

The DataView is the only way (I know of) that we can see which records have been deleted in the DataTable.

The Line

object obj = Activator.CreateInstance(this.service.GetObjectType());

does the same as using

object obj = new CustomerRef.Customer();

if the object type of the Service Connection is Customer.

So if you change the location code on a customer to something stupid you now get this error:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart3outof4_201/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart3outof4_201/image_4.png)

Of course it doesn’t make much sense to Retry this one. Abort aborts the save – and doesn’t reload. Ignore continues the Save and ends up reloading and you loose the changes, we couldn’t save. Note that Abort is a little strange – it cannot abort the changes that has happened up until you abort – and since it doesn’t reload, it leaves the spreadsheet in a bad state.

Maybe we should just remove the abort option – it just seems like a bad thing only to be able to retry or press ignore on every single record. If I come up with a better way of handling Abort, I will post that.

### Can’t wait for part 4?

Part 4 is where we integrate this into NAV and create Actions to call out to open this Excel Spreadsheet on the Client and that includes changes in NAV, a Client Side COM object and a mechanism to transfer parameters to an Excel spreadsheet – stay tuned.

In my original Edit In Excel sample, I use XMLHTTP to retrieve page information and look through that – I will still post the original Edit In Excel source code with part 4 – but note, that there are a LOT of pitfalls in that – and, it doesn’t support Adding records or deleting records, and I have stopped working on that code, even though the need for using XMLHTTP might still be relevant.

The safer way is to use the sample solutions posted with this walk through.

BTW – the NAVTemplate solution can be downloaded here: [http://www.freddy.dk/NAVTemplate.zip](http://www.freddy.dk/NAVTemplate.zip "http://www.freddy.dk/NAVTemplate.zip")

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
