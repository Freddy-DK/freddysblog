---
layout: post
title: "Handling Sales Orders from Page based Web Services – in NAV 2009SP1 (and RTM)"
date: 2009-05-28 13:06:09
categories: ["Archive"]
tags: ["NAV 2009", "NAV 2009 SP1", "Sales Order", "Web Services"]
permalink: /2009/05/28/handling-sales-orders-from-page-based-web-services-in-nav-2009sp1-and-rtm/
---

First of all, there isn’t going to be a new post on every single record type on how to handle them from Web Services – but Sales Orders are special and the reason for the “(and SP1)” in the titel refers to the fact, that there are changes between RTM and SP1 or maybe a better way to state it is, that the way you could do it in RTM (that might lead to errors) is no longer possible – so you have to do it right.

Secondly, please read the post about the Web Service Changes in SP1 before reading this post – you can find that post [here](http://blogs.msdn.com/freddyk/archive/2009/05/27/web-services-changes-in-nav-2009-sp1.aspx).

### Working with the Sales Orders Page from Web Services in NAV 2009SP1

Just to recap a couple of facts about Page based Web Services – we are using the pages and the code on the pages to work with sales orders, this means that we need to mimic the way the RoleTailored Client works, and the RoleTailored Client doesn’t create the order header and all the lines in one go before writing anything to the database. Instead what really happens is that once you leave the primary key field (the Order No) it creates the Order Header in the table. The same with the lines, they are created and then you type data into them, after which they are updated.

So what we need to do is to create the sales order in 4 steps:

1\. Create the Order Header  
2\. Update the Order Header  
3\. Create an Order Line  
4\. Update an Order Line  
(repeat steps 3-4)

Now this doesn’t mean that you have to do 2 + (no of Orderlines)\*2 roundtrips to the server (fortunately) – but you always need 3 roundtrips.

1\. Create the Order Header  
2\. Update the Order Header and Create all Order Lines  
4\. Update all Order Lines

meaning that you can create all order lines in one go (together with updating header info) and you can update them all in one go.

`a code sample for doing this:`

```
// Create Service Reference
var service = new SalesOrder_Service();
service.UseDefaultCredentials = true;
```

```
// Create the Order header
var newOrder = new SalesOrder();
```

`service.Create(ref newOrder);`

```
// Update Order header
newOrder.Sell_to_Customer_No = "10000";
```

```
// Create Order lines
newOrder.SalesLines = new Sales_Order_Line[2];
for (int idx = 0; idx < 2; idx++)
newOrder.SalesLines[idx] = new Sales_Order_Line();
```

`service.Update(ref newOrder);`

```
// Update Order lines
var line1 = newOrder.SalesLines[0];
line1.Type = SalesOrderRef.Type.Item;
line1.No = "LS-75";
line1.Quantity = 3;
```

```
var line2 = newOrder.SalesLines[1];
line2.Type = NewSalesOrderRef.Type.Item;
line2.No = "LS-100";
line2.Quantity = 3;
```

`service.Update(ref newOrder);`

After invoking Create(ref newOrder) or Update(ref newOrder) we get the updated sales order back from NAV, and we know that all the <field>Specified properties are set to true and all strings which has a value are not null – so we can just update the fields we want to update and call update(ref newOrder) again and utilize that SP1 only updates the fields that actually have changed.

This behavior is pretty different from NAV 2009 RTM web services, where it will write all fields to the database if you don’t set strings fields to NULL or set the <field>Specified to false (as described in my previous post).

### Making the above code run in NAV 2009RTM

What we really want to do here, is to mimic the behavior of NAV 2009 SP1 in RTM – without having to change the logic.

So I went ahead and wrote two functions. One for making a copy of a record (to be invoked right after your Create(ref xx) or Update(ref xx)) – so we now have a copy of the object coming from NAV. Another function for preparing our object for Update (to be invoked right before calling Update(ref xx)) – to compare our object with the old copy and set all the unchanged fields <field>Specified to false and all unchanged string fields to null.

The two functions are listed towards the end of this post.

Our code from above would then look like:

```
// Create Service Reference
var service = new SalesOrder_Service();
service.UseDefaultCredentials = true;
```

```
// Create the Order header
var newOrder = new SalesOrder();
```

```
service.Create(ref newOrder);
SalesOrder copy = (SalesOrder)GetCopy(newOrder);
```

```
// Update Order header
newOrder.Sell_to_Customer_No = "10000";
```

```
// Create Order lines
newOrder.SalesLines = new Sales_Order_Line[2];
for (int idx = 0; idx < 2; idx++)
newOrder.SalesLines[idx] = new Sales_Order_Line();
```

```
PrepareForUpdate(newOrder, copy);
service.Update(ref newOrder);
copy = (SalesOrder)GetCopy(newOrder);
```

```
// Update Order lines
var line1 = newOrder.SalesLines[0];
line1.Type = SalesOrderRef.Type.Item;
line1.No = "LS-75";
line1.Quantity = 3;
```

```
var line2 = newOrder.SalesLines[1];
line2.Type = SalesOrderRef.Type.Item;
line2.No = "LS-100";
line2.Quantity = 3;
PrepareForUpdate(newOrder, copy);
```

`service.Update(ref newOrder);`

and this code would actually run on SP1 as well – and cause smaller packages to be sent over the wire (not that I think that is an issue).

### Deleting a line from an existing Sales Order

Now we have seen how to create a Sales Order with a number of lines – but what if you want to delete a line after having saved the Sales Order. On the Service object you will find a method called Delete\_SalesLines, which takes a key and delete that Sales Line.

`service.Delete_SalesLines(line.Key);`

The only caveat to this is, that if you want to do any more work on the Sales Order, you will have to re-read the Sales Order, else you will get an information that somebody changed the record (and that would be you).

So deleting all lines from a Sales Order could be done by:

```
foreach (Sales_Order_Line line in so.SalesLines)
service.Delete_SalesLines(line.Key);
```

and then you would typically re-read the Sales Order with the following line:

`so = service.Read(so.No);`

That wasn’t so bad.

My personal opinion is that we should change the Delete\_SalesLines to be:

`service.Delete_SalesLines(ref so, line);`

Which is why I created a function that does exactly that:

`void Delete_SalesLines(SalesOrder_Service service,` 

```
ref SalesOrder so, Sales_Order_Line line)
{
Debug.Assert(so.SalesLines.Contains<Sales_Order_Line>(line));
service.Delete_SalesLines(line.Key);
so = service.Read(so.No);
}
```

Note, that Í just re-read the order, loosing any changes you have made to the order or order lines. Another approach here could be to remove the line deleted from the lines collection, but things becomes utterly complicated when we try to mimic a behavior in the consumer that IMO should be on the Server side.

A different approach would be to create a codeunit for deleting lines and expose this as an extension to the page (functions added to the page), but we would gain anything, since we still would have to re-read the order afterwards.

### Adding a line to an existing Sales Order

More complicated is it, when we want to add a line to an existing Sales Order through the Sales Order Page.

Actually it isn’t complicated to add the line – but it is complicated to locate the newly added line after the fact to do modifications, because it is still true that you need to add the line first and then modify the line afterwards (and update the order).

Adding the line is:

```
// Create a new Lines array with only the new line and update (meaning create the line)
so.SalesLines = so.SalesLines.Concat(new [] { new Sales_Order_Line() }).ToArray();
service.Update(ref so);
```

This add’s a new line to the array of lines and update the order.

After invoking update the newly added line is the last in the array (unless somebody messed around in the app-code and made this assumption false).

My personal opinion is that we should add another method to the service called

`service.Add_SalesLines(ref so, ref line);`

so that we would have the newly added line available to modify and the so available for service.Update(ref so), which is why I created a function that does exactly that:

```
Sales_Order_Line AddLine(SalesOrder_Service service, ref SalesOrder so)
{
// Create a new Lines array with only the new line and update (meaning create the line)
so.SalesLines = so.SalesLines.Concat(new [] { new Sales_Order_Line() }).ToArray();
service.Update(ref so);
return so.SalesLines[so.SalesLines.Length-1];
}
```

Again – If this method existed server side, automatically added by NAV WS, it would be able to do the right thing even though people had mangled in the application logic and change the line numbering sequence or whatever.

A different approach would be to create a codeunit for adding lines and expose this as an extension to the page (functions added to the page). It wouldn’t make the consumers job much easier since we would still have to have any updates to the SalesOrder written before calling the function AND we would have to re-read the sales order after calling the function.

Remember, that if you are using this function from NAV 2009 RTM you might want to consider using PrepareForUpdate before AddLine and GetCopy after AddLine, just as you would do with Update. You could even add another parameter and have that done by the function itself.

### Working with other Header/Line objects

Although the samples in this post are using the Sales Orders, the same pattern can be reused for other occurrences of the Header/Line pattern. Just remember that the Header needs to be created first, then you can update the header and create the lines – and last (but not least) you can update the lines.

### GetCopy and PrepareForUpdate

Here is a code-listing of GetCopy and PrepareForUpdate – I have tested these functions on a number of different record types and they should work generically

```
/// <summary>
/// Get a copy of a record for comparison use afterwards
/// </summary>
/// <param name="obj">the record to copy</param>
/// <returns>a copy of the record</returns>
object GetCopy(object obj)
{
Type type = obj.GetType();
object copy = Activator.CreateInstance(type);
foreach (PropertyInfo pi in type.GetProperties())
{
if (pi.PropertyType.IsArray)
{
// Copy each object in an array of objects
Array arr = (Array)pi.GetValue(obj, null);
Array arrCopy = Array.CreateInstance(arr.GetType().GetElementType(), arr.Length);
for (int arrIdx = 0; arrIdx < arr.Length; arrIdx++)
arrCopy.SetValue(GetCopy(arr.GetValue(arrIdx)), arrIdx);
pi.SetValue(copy, arrCopy, null);
}
else
{
// Copy each field
pi.SetValue(copy, pi.GetValue(obj, null), null);
}
}
return copy;
}
```

```
/// <summary>
/// Prepare record for update
/// Set <field> to null if a string field hasn't been updated
/// Set <field>Specified to false if a non-string field hasn't been updated
/// </summary>
/// <param name="obj">record to prepare for update</param>
/// <param name="copy">copy of the record (a result of GetCopy)</param>
void PrepareForUpdate(object obj, object copy)
{
Debug.Assert(obj.GetType() == copy.GetType());
Type type = obj.GetType();
PropertyInfo[] properties = type.GetProperties();
for(int idx=0; idx<properties.Length; idx++)
{
PropertyInfo pi = properties[idx];
if (pi.Name != "Key" &&
```

```
pi.GetCustomAttributes(typeof(System.Xml.Serialization.XmlIgnoreAttribute), false).Length == 0)
{
if (pi.PropertyType.IsArray)
{
// Compare an array of objects – recursively
Array objArr = (Array)pi.GetValue(obj, null);
Array copyArr = (Array)pi.GetValue(copy, null);
for (int objArrIdx = 0; objArrIdx < objArr.Length; objArrIdx++)
{
object arrObj = objArr.GetValue(objArrIdx);
PropertyInfo keyPi = arrObj.GetType().GetProperty("Key");
string objKey = (string)keyPi.GetValue(arrObj, null);
for (int copyArrIdx = 0; copyArrIdx < copyArr.Length; copyArrIdx++)
{
object arrCopy = copyArr.GetValue(copyArrIdx);
if (objKey == (string)keyPi.GetValue(arrCopy, null))
PrepareForUpdate(arrObj, arrCopy);
}
}
}
else
{
object objValue = pi.GetValue(obj, null);
if (objValue != null && objValue.Equals(pi.GetValue(copy, null)))
{
// Values are the same – signal no change
if (pi.PropertyType == typeof(string))
{
// Strings doesn't have a <field>Specified property – set the field to null
pi.SetValue(obj, null, null);
}
else
{
// The <field>Specified is autogenerated by Visual Studio as the next property
```

```
idx++;
PropertyInfo specifiedPi = properties[idx];
// Exception if this assumption for some reason isn't true
```

```
Debug.Assert(specifiedPi.Name == pi.Name + "Specified");
specifiedPi.SetValue(obj, false, null);
}
}
}
}
}
}
```

That’s it, not as straightforward as you could have wished for, but SP1 definitely makes things easier once it comes out.

And I will be pushing for a better programming model for this in v7.

Enjoy

_Freddy Kristiansen  
__PM Architect  
Microsoft Dynamics NAV_
