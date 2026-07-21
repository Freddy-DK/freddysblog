---
layout: post
title: "Connecting to NAV Web Services from PHP- take 2"
date: 2016-11-06 22:22:30
categories: ["Archive", "Web Services"]
tags: ["NAV", "NAV 2016", "NAV 2017", "PHP", "SOAP", "Web Services"]
permalink: /2016/11/06/connecting-to-nav-web-services-from-php-take-2/
---

Back in the days – January 19th 2010 to be exact, I wrote a blog post on how to connect to NAV Web Services from PHP [here](/2010/01/19/connecting-to-nav-web-services-from-php/).

Web Services was fairly new and there wasn’t a lot of info out there on how to do things.

The “old” blog post has been read over 76.000 times (yes, seventy six thousand times) and I have received numerous comments and emails on whether this works with NAV 2016 and how you would do things today.

### Back then… NTLM and SPNEGO

The biggest challenge back then was authentication. NAV was using Windows authentication with SPNEGO in NAV 2009 and in 2009SP1 we added a special key in the CustomSettings config file called WebServicesUseNTLMAuthentication, which you could set to true and allow products like PHP to connect to NAV Web Services.

PHP didn’t have native support for NTLM, but the post above describes how to use a custom stream to allow PHP to work.

### Today… Basic Authentication

Since then, NAV has added different authentication providers and today, I wouldn’t recommend using Windows Authentication if you are setting up a Web Services for external access. Even if you are using NAV on-prem with Windows Authentication, it is fairly easy to setup an additional Service Tier, with NavUserPassword, which can be used for Web Services, allow access through firewalls.

Setting up a Service Tier with NavUserPassword will allow Web Services to be accessed using Basic authentication, which is natively supported by virtually all programming languages. Basic authentication is where username and password is sent over the wire and if you use this for anything but demo purposes, you of course want to be using a SSL certificate to secure the communication.

A lot of people have asked me about connecting to NAV Web Services from PHP since then and my response have always been, that it is straightforward. Just remove all the code that has to do with NTLM authentication from the old blog post and you should be fine.

Today I decided to prove myself right:-)

### Setting up a test NAV server

[This](/2016/11/05/the-microsoft-dynamics-nav-image-in-the-public-azure-gallery/) blog post describes how to setup a demo environment using the NAV Image in the Azure Gallery. This is by far the easiest way to get started and I will use this as our starting point.

NAV is setup for NavUserPassword authentication, firewalls is open, endpoints have been added and a self-signed certificate is added to secure the communication.

### Installing PHP

For testing purposes, I will install PHP on the same server. I will still communicate to the public dns name of the Virtual Machine instead of localhost to ensure that communication is flowing like it would be if you are NOT on the same box.

PHP is available in the Web Platform Installer. Select the right version and install.

[![web-platform-installer](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/0bfe9-web-platform-installer-1.png)](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/0bfe9-web-platform-installer.png)

The installer will install PHP, including pre-requisites and configure it with IIS. Machine might need a reboot.

### Visual Studio Code

As editor, I will use Visual Studio Code. It is lightweight and is better than notepad when it comes to PHP code editing.

After installing Visual Studio Code, I added the PHP Code Format extension:

[![vscode](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/becfb-vscode-1.png)](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/becfb-vscode.png)

### Hello World

In order to check that PHP was installed and configured correctly, I created a small file called _helloworld.php_ in the folder _C:\\inetpub\\wwwroot\\http._ This folder is the folder, which contains the landing page and is accessible both from within the VM, but also from outside using a browser.

My hello world code looks like this:

```
 
 
 
 
 <!--?php 
  echo "

Hello World

";
 ?--> 
 
```

and connecting to the Web Site reveals that things are up running:

[![helloworld](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/e99a0-helloworld-1.png)](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/e99a0-helloworld.png)

### Connecting to NAV…

Next thing I did was to take the code from the “old” blog post and strip it from all NTLM stuff, using only standard SoapClient available in PHP and it was exactly as easy as I always thought it would be (Well… – sort of:-))

```
 
 
 
 
 <!--?php 
$login = "";
$password = "";
$baseUrl = ""; // ex. https://.cloudapp.net:7047/NAV/WS/

$context = stream_context_create([
 'ssl' => [
 // set some SSL/TLS specific options
 'verify_peer' => false,
 'verify_peer_name' => false,
 'allow_self_signed' => true
 ]
]);

$options = array("login" => $login, 
 "password" => $password, 
 "features" => SOAP_SINGLE_ELEMENT_ARRAYS, 
 "stream_context" => $context);

$client = new SoapClient($baseUrl."SystemService", $options);

// Find the first Company in the Companies 
$result = $client->Companies(); 
$companies = $result->return_value; 
echo "Companies:
"; 
foreach($companies as $company) { 
 echo $company."
"; 
}
$cur = $companies[0];

$pageURL = $baseUrl.rawurlencode($cur)."/Page/Customer";
echo "
URL of Customer page: $pageURL

";

$page = new SoapClient($pageURL, $options);

$params = array("No" => "10000"); 
$result = $page->Read($params); 
$customer = $result->Customer; 
echo "Name of Customer 10000:".$customer->Name."

";

$params = array("filter" => array( 
 array("Field" => "Location_Code", 
 "Criteria" => "RED|BLUE"), 
 array("Field" => "Country_Region_Code", 
 "Criteria" => "GB") 
 ), 
 "setSize" => 0
 ); 
$result = $page->ReadMultiple($params); 
$customers = $result->ReadMultiple_Result->Customer;

echo "Customers in GB served by RED or BLUE warehouse:
"; 
foreach($customers as $cust) { 
 echo $cust->Name."
"; 
} 
?--> 
 
```

I did actually change two things compared to the sample from 2010:

1.  The stream\_context for ignoring self-signed certificate warnings (or errors) is new
2.  The Soap\_single\_element\_arrays ensures that Soap returns an array even when only one element is returned (thanks Dirk)

### The result

Opening the web site in a browser gave me the result I was looking for

[![result](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/f4504-result-1.png)](/assets/images/2016/connecting-to-nav-web-services-from-php-take-2/f4504-result.png)

This example actually runs on a NAV 2016 CU13 virtual machine on Azure and connects to NAV on a NAV 2017 virtual machine, but was also tested on NAV 2016.

### OData, OAuth and other good stuff

I haven’t focused on how to download OData feeds and other stuff in PHP. This is more of a PHP challenge than it is a NAV challenge. NAV today uses standard authentication and setup with the right certificates and configurations, connecting to NAV OData Services should be straight forward. I also haven’t looked into whether or not you can use OAuth with PHP, my assumption is, that if you are creating a web application with AAD authentication and you want to use your identity to connect to NAV from this Web Application using OAuth, then OAuth isn’t your first challenge.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
