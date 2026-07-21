---
layout: post
title: "Conflict resolution when working with Web Services"
date: 2009-12-09 07:19:35
categories: ["Archive"]
permalink: /2009/12/09/conflict-resolution-when-working-with-web-services/
---

It’s time to wrap up on Edit In Excel R2 – but before I do that, I will explain about another important feature in Edit In Excel R2 – conflict resolution.

In my last post on Edit In Excel R2 (found [here](http://blogs.msdn.com/freddyk/archive/2009/04/13/edit-in-excel-r2-part-1-out-of-2.aspx)) – I explained how to make Edit In Excel capable of taking data offline for editing. It doesn’t make much sense to do that unless you also have some sort of conflict resolution when you then try to save data.

Another important post, which you should read before continuing is this [post](http://blogs.msdn.com/freddyk/archive/2009/12/04/dynamic-references-to-nav-page-web-services-in-c-take-2.aspx), explaining about how to use dynamic web references and explains a little about some helper classes for enumerating fields and stuff.

This post is all about creating a conflict resolution dialog like this

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConflictresolutionwhenworkingwithWebServ_C455/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConflictresolutionwhenworkingwithWebServ_C455/image_4.png)

which can be used in Edit In Excel – or in other scenarios, where you want the user to do conflict resolution.

### Type weak

This dialog of course needs to be type weak – we are not about to write a conflict resolution dialog for each page we use as web services.

For this, I will use the NAVFieldInfo class hierarchy from this [post](http://blogs.msdn.com/freddyk/archive/2009/12/04/dynamic-references-to-nav-page-web-services-in-c-take-2.aspx) and produce the following static method in a Windows Forms Form called NAVPageMergeForm:

/// <summary>  
/// Resolve conflicts in a record  
/// </summary>  
/// <param name=”fields”>NAVFields collection</param>  
/// <param name=”orgObj”>Original record</param>  
/// <param name=”obj”>Record with your changes</param>  
/// <param name=”theirObj”>Record with changes made by other users</param>  
/// <param name=”mergedObj”>This parameter receives the merged object if the user presses accept</param>  
/// <returns>DialogResult.Retry if the user pressed Accept Merged Values  
/// DialogResult.Ignore if the user chose to ignore this record  
/// DialogResult.Abort if the user chose to abort</returns>  
public static DialogResult ResolveConflict(NAVFields fields, object orgObj, object myObj, object theirObj, out object mergedObj)  
{  
NAVPageMergeForm form = new NAVPageMergeForm();  
// And set the values  
form.SetValues(fields, orgObj, myObj, theirObj);  
DialogResult response = form.ShowDialog();  
if (response == DialogResult.Retry)  
mergedObj = form.GetMergedObj();  
else  
mergedObj = null;  
return response;  
}

The method takes a Field Collection and three objects. The object as it looked before i started to make my changes (orgObj), the object with my changes (myObj) and the “new” object including the changes somebody else made in the database (theirObj). The last parameter is an out parameter meaning that this is where the method returns the merged object.

The return value of the dialog is a DialogResult, which can be Abort, Ignore og Retry.

### Why Retry?

Why not OK?

Well, if you think of it, when you have merged the record you now have a new record which you have constructed – but while you were merging this record, somebody else might have changed the record again – meaning that we have to retry the entire thing and this might result in a new conflict resolution dialog popping up for the same record.

### How to use the ResolveConflict method

As you can imagine, the usage of the method needs to follow a certain pattern and in my small sample app I have done this like:

bool retry;  
do  
{  
retry = false;  
try  
{  
service.Update(ref myChanges);  
}  
catch (Exception e)  
{  
string message = e.InnerException != null ? e.InnerException.Message : e.Message;  
object theirChanges = null;  
try  
{  
if (service.IsUpdated(customer2.Key))  
theirChanges = service.Read(customer2.No);  
}  
catch { } // Ignore errors when re-reading  
if (theirChanges != null)  
{  
object merged;  
retry = (NAVPageMergeForm.ResolveConflict(fields, customer2, myChanges, theirChanges, out merged) == DialogResult.Retry);  
if (retry)  
myChanges = (Customer)merged;  
}  
else  
retry = (MessageBox.Show(string.Format(“{0}nn{3}”, “Customer ” + customer2.No, message), “Unable to Modify Record”, MessageBoxButtons.RetryCancel, MessageBoxIcon.Error) == DialogResult.Retry);  
}  
} while (retry);

Compared to just saying service.Update(ref myChanges) – this is of course more complicated, but it adds huge value.

In Edit-In-Excel, this is of course captured i a method called UpdateRecord.

### What happens in SetValues?

SetValues basically enumerates the field collection and adds values to a grid as you see in the image above, comparing the changes made by the various people and automatically merging values only changed by one user – displaying conflicts if the same field was modified by both.

/// <summary>  
/// Set records in merge form  
/// </summary>  
/// <param name=”fields”>Field collection</param>  
/// <param name=”orgObj”>Original record</param>  
/// <param name=”obj”>Modified record</param>  
/// <param name=”theirObj”>Changed record from WS</param>  
internal void SetValues(NAVFields fields, object orgObj, object myObj, object theirObj)  
{  
this.mergedObj = theirObj;  
this.fields = fields;

    foreach (NAVPageFieldInfo field in fields)  
{  
if (field.field != “Key”)  
{  
object orgValue = field.GetValue(orgObj);  
if (orgValue == null) orgValue = “”;  
object myValue = field.GetValue(myObj);  
if (myValue == null) myValue = “”;  
object theirValue = field.GetValue(theirObj);  
if (theirValue == null) theirValue = “”;  
object mergedValue;

            DataGridViewCellStyle myStyle = this.normalStyle;  
DataGridViewCellStyle theirStyle = this.normalStyle;  
DataGridViewCellStyle mergedStyle = this.normalStyle;

            bool iChanged = !orgValue.Equals(myValue);  
bool theyChanged = !orgValue.Equals(theirValue);

            if (iChanged && theyChanged)  
{  
// Both parties changed this field  
myStyle = modifiedStyle;  
theirStyle = modifiedStyle;  
if (myValue.Equals(theirValue))  
{  
mergedValue = myValue;  
mergedStyle = this.modifiedStyle;  
}  
else  
{  
mergedValue = “”;  
mergedStyle = this.conflictStyle;

                }  
}  
else if (theyChanged)  
{  
// “They” changed something – I didn’t  
mergedValue = theirValue;  
theirStyle = this.modifiedStyle;  
mergedStyle = this.modifiedStyle;  
}  
else if (iChanged)  
{  
// I changed something – “they” didn’t  
mergedValue = myValue;  
myStyle = this.modifiedStyle;  
mergedStyle = this.modifiedStyle;  
}  
else  
{  
// Nobody changed anything – merged value is ok  
mergedValue = orgValue;  
}  
int rowno = this.mergeGridView.Rows.Add(field.field, orgValue, myValue, theirValue, mergedValue);  
this.mergeGridView\[2, rowno\].ValueType = field.fieldType;  
this.mergeGridView\[3, rowno\].ValueType = field.fieldType;  
this.mergeGridView\[4, rowno\].ValueType = field.fieldType;

            this.mergeGridView\[2, rowno\].Style = myStyle;  
this.mergeGridView\[3, rowno\].Style = theirStyle;  
this.mergeGridView\[4, rowno\].Style = mergedStyle;  
if (mergedStyle == this.conflictStyle)  
{  
if (this.mergeGridView.CurrentCell == null)  
this.mergeGridView.CurrentCell = this.mergeGridView\[0, rowno\];  
}  
}  
}  
UpdateConflicts();  
}

The rest is really manipulating button status depending on selection, setting values if you press My Changes, Their Changes or Original and in the end when the user pressed Accept changes we just return Retry and the caller will call and get the Merged Object, which basically just is

/// <summary>  
/// Get Merged object  
/// </summary>  
/// <returns>the Merged record</returns>  
internal object GetMergedObj()  
{  
int rowno = 0;  
foreach (NAVPageFieldInfo field in fields)  
{  
if (field.field != “Key”)  
{  
field.SetValue(this.mergedObj, this.mergeGridView\[4, rowno\].Value, field.GetValue(this.mergedObj));  
rowno++;  
}  
}  
return this.mergedObj;  
}

### A small Test App

Here is a small test app, demonstrating how the conflict resolution can be provoked

Console.WriteLine(“Initialize Service…”);  
Customer\_Service service = new Customer\_Service();  
service.UseDefaultCredentials = true;

Console.WriteLine(“Read Customer 10000… – twice (two users)”);  
// Read customer twice  
Customer customer1 = service.Read(“10000”);  
Customer customer2 = service.Read(“10000”);

Console.WriteLine(“One user changes Customer 10000…”);  
// Change customer 1  
customer1.Phone\_No = “111-222-3333”;  
customer1.Address\_2 = “an address”;  
service.Update(ref customer1);

Console.WriteLine(“Other user tries to change Customer 10000”);  
NAVFields fields  = new NAVFields(typeof(Customer), typeof(Customer\_Fields));  
Customer myChanges = (Customer)GetCopy(customer2);

myChanges.Phone\_No = “222-333-4444”;  
myChanges.Name = “The Cannon Group, Inc.”;

Write the data according to the pattern shown above – using conflict resolution and all (and after that – clean up the data, so that we can run the test app again)

Console.WriteLine(“Reset Data…”);

// Clear any updated data  
Customer customer = service.Read(“10000”);  
customer.Phone\_No = “”;  
customer.Address\_2 = “”;  
customer.Name = “The Cannon Group PLC”;  
service.Update(ref customer);

Console.WriteLine(“nTHE END”);  
Console.ReadLine();

BTW – the GetCopy method looks like this:

private static object GetCopy(object obj)  
{  
Type typ = obj.GetType();  
MethodInfo mi = typ.GetMethod(“MemberwiseClone”, BindingFlags.Instance | BindingFlags.NonPublic);  
return mi.Invoke(obj, null);  
}

The ConflictResolution solution can be downloaded [here](http://www.freddy.dk/ConflictResolution.zip).

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
