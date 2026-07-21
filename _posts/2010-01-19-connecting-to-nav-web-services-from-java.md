---
layout: post
title: "Connecting to NAV Web Services from Java"
date: 2010-01-19 14:09:39
categories: ["Archive", "Web Services"]
tags: ["Java", "NAV 2009 SP1", "Web Services"]
permalink: /2010/01/19/connecting-to-nav-web-services-from-java/
---

### Prerequisites

Please read [this post](http://blogs.msdn.com/freddyk/archive/2010/01/19/connecting-to-nav-web-services-from.aspx) to get an explanation on how to modify the service tier to use NTLM authentication and for a brief explanation of the scenario I will implement in Java.

BTW. Basic knowledge about Java is required to understand the following post:-)

### Version and download

Java does not natively have support for the SPNEGO authentication protocol, but it does have support for NTLM authentication, so you will have to change the Web Services listener to use that.

I downloaded the Java developer toolkit from [http://www.java.com/en/download/manual.jsp](http://www.java.com/en/download/manual.jsp "http://www.java.com/en/download/manual.jsp") and installed it under c:\\sun – and made sure that c:\\sun\\SDK\\bin is added to the path.

### wsimport – import WSDL

To make Java able to see and use webservices I use wsimport to create proxy classes in Java. In a directory, I have created two empty folders (generated and source) and use the following command:

`C:\java>wsimport -d generated -s source http://localhost:7047/DynamicsNAV/WS/SystemService`

this will create a series of .class files under

`C:\java\generated\schemas\dynamics\microsoft\nav\system`

and a set of source files under

`C:\java\source\schemas\dynamics\microsoft\nav\system`

note, that the folder structure is the namespace of the web service.

### javac – compile the program

Assuming that I create a test program in c:\\java\\test.java, the following command will compile the program:

`c:\java>c:\sun\sdk\jdk\bin\javac -cp .;generated test.java`

this creates a c:\\java\\test.class, which we can run.

### java – run the program

The statement used to run the java class is:

`c:\java>c:\sun\sdk\jdk\bin\java -cp .;generated test`

enough about setup – lets code…

### Authentication

By default, java supports Windows Authentication and if the class is running in context of a windows user, these credentials are automatically used to authenticate towards NAV Web Services.

If you want to use different credentials (connect as a different user) you will need to create a class derived from _Authenticator_ and implement _getPasswordAuthentication_ like:

```
static class MyAuthenticator extends Authenticator {
public PasswordAuthentication getPasswordAuthentication() {
return (new PasswordAuthentication("domainuser", new char[] {'p','a','s','s','w','o','r','d'}));
}
}
```

and then as the very first statement in the class

`Authenticator.setDefault(new MyAuthenticator());`

### test.java

First of all, I have a number of imports. Not sure that they all are needed and then a main function body:

```
import java.net.*;
import java.util.*;
import java.io.*;
import javax.xml.namespace.QName;
import schemas.dynamics.microsoft.nav.system.*;
import schemas.dynamics.microsoft.page.customer.*;
```

```
public class test {
public static void main(String[] args) {
```

        `// main program code`

    

```
}
}
```

The main code is:

```
try {
```

`String baseURL = "`[http://localhost:7047/DynamicsNAV/WS/&#8221](http://localhost:7047/DynamicsNAV/WS/&#8221);;

The following code allows you to connect to NAV Web Services system service in Java and output the companies available on the service tier:

  

```
URL systemServiceURL = new URL(baseURL + "SystemService");
QName systemServiceQName = new QName("urn:microsoft-dynamics-schemas/nav/system/", "SystemService");
SystemService systemService = new SystemService(systemServiceURL, systemServiceQName);
SystemServicePort systemPort = systemService.getSystemServicePort();
List companies = systemPort.companies();
System.out.println("Companies:");
for(String company : companies) {
System.out.println(company);
}
String cur = companies.get(0);
```

Now I have the company I want to use in _cur_ and the way I create a URL to the Customer page is by doing:

  

```
URL customerPageURL = new URL(baseURL+URLEncoder.encode(cur,"UTF-8″).replace("+","%20″)+"/Page/Customer");
QName customerPageQName = new QName("urn:microsoft-dynamics-schemas/page/customer", "Customer_Service");
```

  `System.out.println("nURL of Customer page:" + customerPageURL.toString());`

and then I can create a Service Client to the Customer Page:

  

```
CustomerService customerService = new CustomerService(customerPageURL, customerPageQName);
CustomerPort customerPort = customerService.getCustomerPort();
```

and using this, I read customer 10000 and output the name:

  

```
Customer cust = customerPort.read("10000");
System.out.println("nName of Customer 10000:" + cust.getName());
```

Last, but not least – lets create a filter and read all customers in GB that has Location Code set to RED or BLUE:

  

```
CustomerFilter filter1 = new CustomerFilter();
filter1.setField(CustomerFields.fromValue("Location_Code"));
filter1.setCriteria("RED|BLUE");
```

  

```
CustomerFilter filter2 = new CustomerFilter();
filter2.setField(CustomerFields.fromValue("Country_Region_Code"));
filter2.setCriteria("GB");
```

  

```
List filters = new ArrayList();
filters.add(filter1);
filters.add(filter2);
```

  

```
System.out.println("nCustomers in GB served by RED or BLUE warehouse:");
CustomerList customers = customerPort.readMultiple(filters, null, 0);
for (Customer c : customers.getCustomer()) {
System.out.println(c.getName());
}
```

  `System.out.println("nTHE END");`

```
} catch(MalformedURLException e) {
} catch(UnsupportedEncodingException e) {
}
```

All of the above will output the following in a prompt (on my machine running NAV 2009SP1 W1)

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromJava_C712/image_thumb.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromJava_C712/image_2.png)

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
