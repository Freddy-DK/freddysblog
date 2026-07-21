---
layout: post
title: "Edit In Excel – Part 2 (out of 4)"
date: 2008-11-07 02:36:26
categories: ["Archive"]
tags: ["C#", "COM", "Data", "Excel", "NAV 2009", "Web Services"]
permalink: /2008/11/07/edit-in-excel-part-2-out-of-4/
---

If you haven’t read [Part 1](http://blogs.msdn.com/freddyk/archive/2008/11/07/edit-in-excel-part-1-out-of-4.aspx), you should do so before continuing here.

In Part 1 we saw how easy it is to white .net code inside Excel, and get it executed based on an event in Excel, and how easy it is to fill values into cells. But in order to make this really useful we need to go a different way around.

First of all, we need to know more about the record we are working with. We could of course hard code everything – but that is not the preferred way to go. It needs to be flexible.

When referencing the Customer Page in Visual Studio, Visual Studio creates a proxy class (a wrapper for the Customer).

This Proxy class contains members for all fields on the Page and properties to access them, like:

public partial class Customer  
{  
private string keyField;  
private string noField;  
private string nameField;  
… etc …

  
public string Key  
{  
get { return this.keyField; }  
set { this.keyField = value; }  
}  
  
public string No  
{  
get { return this.noField; }  
set { this.noField = value; }  
}  

    public string Name{  
get { return this.nameField; }  
set { this.nameField = value; }  
}

… etc …  
}

Meaning that if we have a variable of type Customer, we can get the value of the fields by accessing cust.Name etc., as we saw in part 1. There is no collection of fields, field types and getters/setters we can call – all we have is a class with a number of properties and an Enum with all the field names.

So, in order to be able to build a list of fields, look at their types and get their values without having to hard code everything, we need to use reflection.

### Setting up definitions

I wont go into details about reflection here – but basically we need a method like this:

/// <summary>  
/// Populate Fields Collection with NAVFieldInfo for all properties in the record  
/// Should works with any NAV 2009 Page exposed as WebService  
/// </summary>  
/// <param name=”objType”>Type of Object (typeof(Customer), typeof(Vendor), …)</param>  
/// <param name=”fieldsType”>Type of the Enum holding the property names</param>  
private void PopulateFieldsCollection(Type objType, Type fieldsType)  
{  
// Create columns in Datatable  
this.dataTable = new DataTable(objType.Name);  
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

This method enumerates all field names in the fieldsType Enum and call AddField for every field in the record object.

AddField then looks like this:

/// <summary>  
/// Add a Column to the DataTable  
/// And create a corresponding NAVFieldInfo  
/// </summary>  
/// <param name=”field”>Field name</param>  
/// <param name=”objType”>Type of Object in which the field is (typeof(Customer), typeof(Vendor), …)</param>  
private void AddField(string field, Type objType)  
{  
PropertyInfo pi = objType.GetProperty(field, System.Reflection.BindingFlags.Public | System.Reflection.BindingFlags.Instance);  
NAVFieldInfo nfi = NAVFieldInfo.CreateNAVFieldInfo(field, pi, objType.Namespace);  
// If we encounter unknown Field Types, they are just ignored  
if (nfi != null)  
{  
this.fields.Add(nfi);  
this.dataTable.Columns.Add(new DataColumn(field, nfi.fieldType));  
}  
}

It uses reflection to get the information about the property on the record object (the PropertyInfo).

After calling the PopulateFIeldsCollection, we will have a collection of fields in the **this.fields** variable and we will have the corresponding columns in the **this.dataTable** variable, all created with the right names and types on the columns.

For every different field type there is a different class which all derives from NAVFieldInfo. This class holds the field name, the field type and the PropertyInfo class (which is used for invoking the getter and setter on the record object later).

The static CreateNAVFieldInfo which is called from AddField then creates the right NAVFieldInfo class as shown below:

/// <summary>  
/// Create a NAVFieldInfo object for a specific field  
/// </summary>  
/// <param name=”field”>Name of the property</param>  
/// <param name=”pi”>PropertyInfo for the property on the record object</param>  
/// <param name=”ns”>Namespace for the record object (namespace for the added WebServices proxy class)</param>  
/// <returns>NAVFieldInfo or null if the type isn’t supported</returns>  
internal static NAVFieldInfo CreateNAVFieldInfo(string field, System.Reflection.PropertyInfo pi, string ns)  
{  
if (pi.PropertyType == typeof(string))  
{  
// String Property – is it the KeyField  
if (field == “Key”)  
return new KeyFieldInfo(field, pi);  
else  
return new StringFieldInfo(field, pi);  
}  
if (pi.PropertyType == typeof(decimal))  
{  
// Decimal Property  
return new DecimalFieldInfo(field, pi);  
}  
if (pi.PropertyType == typeof(int))  
{  
// Integer Property  
return new IntFieldInfo(field, pi);  
}  
if (pi.PropertyType == typeof(bool))  
{  
// Boolean Property  
return new BooleanFieldInfo(field, pi);  
}  
if (pi.PropertyType == typeof(DateTime))  
{  
// DateTime Property  
return new DateTimeFieldInfo(field, pi);  
}  
if (pi.PropertyType.Namespace == ns)  
{  
// Other Property Types, in the same namespace as the object  
// These are enum’s – set up restrictions on OptionFields  
return new OptionFieldInfo(field, pi, Enum.GetNames(pi.PropertyType));  
}  
// Unsupported – ignore  
return null;  
}

Meaning that there are classes for KeyFieldInfo, StringFieldInfo, DecimalFieldInfo, IntFieldInfo, BooleanFieldInfo, DateTimeFieldInfo and OptionFieldInfo.

On these classes we have 3 virtual methods:

1.  Get the value of the property from a Record Object (GetValue)
2.  Set the value of the property on the Record Object (SetValue)
3.  Format the Excel column used to show this type (AdjustColumn)

Example of the AdjustColumn on the StringFieldInfo is:

/// <summary>  
/// Adjust formatting and properties of an Excel column  
/// </summary>  
/// <param name=”column”>Excel Range of cells to set formatting on</param>  
internal override void AdjustColumn(Microsoft.Office.Interop.Excel.Range column)  
{  
column.EntireColumn.NumberFormat = “@”;  
column.EntireColumn.HorizontalAlignment = Microsoft.Office.Interop.Excel.Constants.xlLeft;  
column.EntireColumn.AutoFit();  
}

Which will set the format, the horizontal alignment and make the column with adjust to the size of the content.

The GetValue is defined as

/// <summary>  
/// Get the value from the record object by calling the property getter on the object  
/// </summary>  
/// <param name=”obj”>The record object</param>  
/// <returns>The value in the type specified in fieldType</returns>  
internal virtual object GetValue(object obj)  
{  
return this.pi.GetValue(obj, null);  
}

in the base class – and is only overwritten in the BooleanFieldInfo class. The SetValue is defined in the base class is:

/// <summary>  
/// Set the value to the record object by calling the property setter on the object  
/// </summary>  
/// <param name=”obj”>The record object</param>  
/// <param name=”value”>The new value for the field</param>  
internal virtual void SetValue(object obj, object value)  
{  
    if (value == DBNull.Value)  
{  
if (!string.IsNullOrEmpty(this.pi.GetValue(obj, null) as string))  
{  
this.pi.SetValue(obj, “”, null);  
}  
}  
else  
{  
this.pi.SetValue(obj, value, null);  
}  
}

has is overwritten in OptionFieldInfo and BooleanFieldInfo.

The reason for overwriting these functions is, that the property type in the Record Object is different from the type we set in the DataTable. For Boolean – we want to have a Yes/No option in Excel – but the type in the Record Object is a Boolean – not a string.

For Option Fields – the Property Type of the Record Object is an Enumeration defined in the same namespace as the Record Object and the OptionFieldInfo class uses reflection to enumerate the enumeration names.

### Loading the data

After having information about all fields ready on our fingertips and the DataTable initialized and ready to recieve data – we just need to read the data and add it to the datatable (and then of course add the datatable to the Spreadsheet).

I have created a method called Load() – which does all of these things:

/// <summary>  
/// Load Customers from NAV via Web Services  
/// </summary>  
private void Load()  
{  
PopulateFieldsCollection(typeof(CustomerRef.Customer), typeof(CustomerRef.Customer\_Fields));  
CustomerRef.Customer\_Service service = new CustomerRef.Customer\_Service();  
service.UseDefaultCredentials = true;  
this.objects = service.ReadMultiple(null, “”, 0);  
PopulateDataTable();  
    AddDataToExcel();  
}

First of all, call the PopulateFieldsCollection method. Then create a Web Service Connection and read all record objects in an array of objects (note that I am NOT using strongly typed class references, as I want this code to be reusable when I work with other pages via Web Services).

The PopulateDataTable method became amazingly simple:

/// <summary>  
/// Populate DataTable based on array of objects  
/// </summary>  
private void PopulateDataTable()  
{  
// Populate DataTable with data  
foreach (object obj in this.objects)  
{  
DataRow dataRow = this.dataTable.NewRow();  
foreach (NAVFieldInfo nfi in this.fields)  
{  
dataRow\[nfi.field\] = nfi.GetValue(obj);  
}  
this.dataTable.Rows.Add(dataRow);  
}  
}

Run through all rows and for all rows – run through all columns and get the value from the Record Object into the Data Table.

and the AddDataToExcel really isn’t that hard either:

/// <summary>  
/// Add a DataList to Excel  
/// This function should work with any Page exposed as Web Service  
/// </summary>  
/// <param name=”objects”>Array of records to add</param>  
private void AddDataToExcel()  
{  
Application.ScreenUpdating = false;

    // Populate Excel Spreadsheet with data  
this.dataList = this.Controls.AddListObject(this.Range\[this.Cells\[1, 1\], this.Cells\[this.dataTable.Rows.Count + 1, this.dataTable.Columns.Count + 1\]\], this.dataTable.TableName);  
this.dataList.AutoSetDataBoundColumnHeaders = true;  
this.dataList.DataSource = this.dataTable;  
this.dataTable.AcceptChanges();

    // Adjust columns in excel with the right formatting based on Field Info  
int col = 1;  
foreach (NAVFieldInfo nfi in this.fields)  
{  
nfi.AdjustColumn(this.dataList.Range\[1, col++\] as Microsoft.Office.Interop.Excel.Range);  
}

    Application.ScreenUpdating = true;  
}

First, we disable ScreenUpdating, setup a ListObject with the created DataTable, call AdjustColumn on the correct NAVFieldInfo class for the corresponding column and enable screenupdating again.

Running the project with these things gives us:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart2outof4_12A8B/image_thumb.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart2outof4_12A8B/image_2.png)

Now – that’s more like it. We have data in a table, formatting on fields and even options in option fields:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart2outof4_12A8B/image_thumb_1.png)](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/EditInExcelPart2outof4_12A8B/image_4.png)

### and… Saving the changes!

The only thing we need to do more is to be able to write data back into NAV, if we change something. But wait – we would of course also like to be able to create records and delete records in the spreadsheet – lets see what is needed for that….

We use DataView to look at a subset of a DataTable (deleted, added or modified) and then we need to do react accordingly.

First we run through the deleted records and delete them from NAV – if you want to add a warning, you should do so before calling service.Delete.

// Run through records marked for deletion and delete those  
DataView dv = new DataView(this.dataTable, “”, “”, DataViewRowState.Deleted);  
foreach (DataRowView drv in dv)  
{  
object obj = GetRecordObject((string)drv\[0\]);  
if (obj != null)  
{  
service.Delete(((CustomerRef.Customer)obj).Key);  
}  
}

Then we run through the added records and add them to NAV:

// Run through added records and add them  
dv = new DataView(this.dataTable, “”, “”, DataViewRowState.Added);  
foreach (DataRowView drv in dv)  
{  
CustomerRef.Customer customer = new CustomerTemplate.CustomerRef.Customer();  
foreach (NAVFieldInfo nfi in this.fields)  
{  
if (nfi.field != “Key”)  
{  
nfi.SetValue(customer, drv\[nfi.field\]);  
}  
}  
service.Create(ref customer);  
}

and finally we run through the modified records and update them in NAV:

// Run through modified records and update them  
dv = new DataView(this.dataTable, “”, “”, DataViewRowState.ModifiedCurrent);  
foreach (DataRowView drv in dv)  
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
CustomerRef.Customer customer = (CustomerRef.Customer)obj;  
service.Update(ref customer);  
}  
}

and after this – we reload the data.

so… – that was a LOT of code and some explaining. I hope you are with me so far.

I didn’t explain how to add a Ribbon to Excel, nor did I list all the NAVFieldInfo classes in this post, but you can download the solution as source from [http://www.freddy.dk/CustomerTemplate.zip](http://www.freddy.dk/CustomerTemplate.zip "http://www.freddy.dk/CustomerTemplate.zip") and see how these things are done. You can play around with things and/or use pieces of the code in your own solution.

_The code shown in this post comes with no warranty – and is only intended for showing how to do things. The code can be reused, changed and incorporated in any project without any further notice._

### So that’s all good, but what’s next?

We now have a spreadsheet, which can read customers via Web Services, allow you to modify, add or delete and save the changes through Web Services, which is pretty impressive – but we have 2 more posts on the topic:-)

Next, we want to integrate this into the Customer List Place and get the filter from the List Place to the spreadsheet. To do this we need a Client side COM object, which transfers parameters to the Excel Spreadsheet and launches Excel.

And last, we will make the Edit In Excel work without the Web Reference – meaning that we can just add an action to the pages on which we want the Edit In Excel to work.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
