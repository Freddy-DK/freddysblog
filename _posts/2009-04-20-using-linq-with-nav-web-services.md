---
layout: post
title: "Using LINQ with NAV Web Services"
date: 2009-04-20 20:02:30
categories: ["Archive", "Web Services"]
tags: ["C#", "LINQ", "NAV 2009", "Web Services"]
permalink: /2009/04/20/using-linq-with-nav-web-services/
---

In .NET 3.5 Microsoft released LINQ (Language INtegrated Query) – which is a way of writing select statements strongly typed inside your C# code.

Wouldn’t it be nice if we could leverage this technology when connecting to NAV Web Services?

I thought YES – so I set out to find out what it took…

The goal was to be able to replace code like this:

```
CustomerRef.Customer_Service cservice = new CustomerRef.Customer_Service();
cservice.UseDefaultCredentials = true;
```

```
CustomerRef.Customer_Filter filter = new CustomerRef.Customer_Filter();
filter.Field = CustomerRef.Customer_Fields.Location_Code;
filter.Criteria = "=YELLOW|=BLUE";
CustomerRef.Customer_Filter[] filters = new CustomerRef.Customer_Filter[] { filter };
```

`CustomerRef.Customer[] customers = cservice.ReadMultiple(filters, null, 0);`

```
foreach (CustomerRef.Customer customer in customers)
{
// do stuff
}
```

with code like this:

```
CustomerRef.Customer_Service cservice = new CustomerRef.Customer_Service();
cservice.UseDefaultCredentials = true;
```

```
var cquery = from c in new FreddyK.NAVPageQuery<CustomerRef.Customer>(cservice)
where c.Location_Code == "YELLOW" || c.Location_Code == "BLUE"
select c;
```

```
foreach (CustomerRef.Customer customer in cquery)
{
// do stuff
}
```

Which I personally find more readable and has a lot of other advantages:

1.  Strongly typed criterias – if fields change type you get a compile error after updating the Web Reference
2.  Don’t have to think about CultureInfo (NumberFormat and DateFormat) when doing your criteria
3.  Don’t have to duplicate quotes for your search string
4.  Don’t have to convert booleans into Yes/No
5.  Separation of Query and Values in the Query
6.  and probably a lot more…

I always heard that you only needed 3 good reasons for doing something – so this should be more than enough.

### Understanding LINQ

Lets look at the above code. The following

```
CustomerRef.Customer_Service cservice = new CustomerRef.Customer_Service();
cservice.UseDefaultCredentials = true;
```

initializes the Service (as usually) and

```
var cquery = from c in new FreddyK.NAVPageQuery<CustomerRef.Customer>(cservice)
where c.Location_Code == "YELLOW" || c.Location_Code == "BLUE"
select c;
```

creates a query (which contains an expression tree of the where clause) and

```
foreach (CustomerRef.Customer customer in cquery)
{
// do stuff
}
```

invokes the query (calls Web Services ReadMultiple) and returns the customers.

FreddyK.NAVPageQuery is a class, which is capable of acting as a LINQ provider for all pages exposed as Web Services.

The NAVPageQuery class is a generic class where you specify the type returned by the service – and you specify the service object as a parameter to the constructor. No need to say, that these two must match, else you will get an exception.

For a class to be usable in a LINQ query it needs to implement [IQueryable<type>](http://msdn.microsoft.com/en-us/library/bb351562.aspx) (which is a combination of [IQueryable](http://msdn.microsoft.com/en-us/library/system.linq.iqueryable.aspx), [IEnumerable<type>](http://msdn.microsoft.com/en-us/library/9eekhta0.aspx) and [IEnumerable](http://msdn.microsoft.com/en-us/library/system.collections.ienumerable.aspx)) – and typically you will then have another class, which implements [IQueryProvider](http://msdn.microsoft.com/en-us/library/system.linq.iqueryprovider.aspx). In my sample, I have implemented both interfaces in the NAVPageQuery.

Instead of listing the interfaces, I will list what happens when the query is build (when the line with var cquery = is executed):

1.  The NAVPageQuery gets instantiated
2.  LINQ calls the method `IQueryProvider IQueryable.Provider` to get the IQueryProvider class
3.  LINQ calls the method `System.Linq.Expressions.Expression IQueryable.Expression` to get the initial expression
4.  Now LINQ builds the expression tree and calls `IQueryable<S> IQueryProvider.CreateQuery<S>(Expression expression)` from which it expects a IQueryable<type> object.

Now we are done building the query and our variable cquery is set to be of type IQueryable<Customer>

When we then do a foreach on the cquery a little later, the following this happens:

1.  foreach calls into the IEnumerable interface `IEnumerator<T> IEnumerable<T>.GetEnumerator()` and ask for an enumerator
2.  In this method, I call the Execute method (`TResult IQueryProvider.Execute<TResult>(Expression expression)`) on the Provider (which is ‘this’)
3.  In the Execute method I run through the Expression tree and builds a number of Filter objects, collect them into an array of filters and call ReadMultiple.

You could argue that it would be a good idea to run through the expression tree while building the query – but the primary reason for NOT doing this is that you could imagine queries like:

`decimal amount = 10000;`

```
var cquery = from c in new FreddyK.NAVPageQuery<CustomerRef.Customer>(cservice)
where c.Balance_LCY > amount
select c;
```

In this case you want to separate building the query and creating the filters, since you could:

```
foreach (CustomerRef.Customer customer in cquery)
{
// do stuff
}
```

```
amount = 100000;
foreach (CustomerRef.Customer customer in cquery)
{
// do other stuff
}
```

this demonstrates the need for having the ability to decouple the query and the values used by the query and this is the reason for not creating the filter specs while building the query – but keeping the expression tree.

Another sample:

```
cquery = from c in new FreddyK.NAVPageQuery<CustomerRef.Customer>(cservice)
where c.Last_Date_Modified >= DateTime.Today.AddDays(-5)
select c;
```

At the time of execution, this query will give you customers modified the last 5 days (imagine building that one with filters easy)

### Important things to notice

It is important to realize that NAVPageQuery doesn’t give you any functionality you didn’t have before. With the original Filter class you cannot OR things together (unless it is the same field) – you also cannot do that using LINQ (towards NAV Web Services)

If you try to type in a where clause like

```
where c.Location_Code == "YELLOW" || c.Country_Region_Code == "US"
```

It will fail – intentionally!

Of course I could have done client side filtering – but this would only fool the developer, since you would take the performance hit when calling ReadMultiple and you wouldn’t really notice – I wanted the developer to be aware when he makes clever or not so clever choices.

So – the where clause needs to be simple enough so that I can convert it into a filter specification.

The error will be thrown when you try to use the query and it will typically be an exception stating “Query too complex” (even though it might not be complex at all).

Another thing to notice is that AND’s and OR’s cannot be combined (as this isn’t possible in the filter spec):

```
where c.Location_Code == "YELLOW" && (c.Balance_LCY > amount || c.Location_Code == "BLUE")
```

is not allowed, but

```
where c.Balance_LCY > amount && (c.Location_Code == "YELLOW" || c.Location_Code == "BLUE") && c.No != ""
```

works just fine.

### What about updating, deleting and creating?

Nothing new there – you still have your service object and the Customer objects that are returned from the LINQ query is the same object you would get from reading through Readmultiple directly.

This class is only designed to make queries towards web services easier and more error safe.

### That’s all good – but how is it made?

For once, I am not going to go through how NAVPageQuery is created in detail – I have commented the source code pretty detailed and if you need to understand how it is build, you can look at the sourcecode combined with the information on how things are executed (from above) and it should be possible to decrypt the source:-)

If you download the solution .zip file [here](http://www.freddy.dk/NavPageQuery.zip), you will find a class library with one class (NAVPageQuery) and a Console application called TestQuery. The console app. can be deleted if you don’t want it and it has a number of samples – some of which are listed above.

When downloading the test app. – you need to update the web references (so that they match your exposed Pages) – I know that I added the Last\_Date\_Modified to the Customer Page in order to have access to that field through web services – there might be more.

Questions and comments are welcome, I prefer to answer questions on [mibuso](http://www.mibuso.com) instead of here – since it typically makes the answer available to more people.

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
