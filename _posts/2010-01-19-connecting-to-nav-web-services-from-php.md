---
layout: post
title: "Connecting to NAV Web Services from PHP"
date: 2010-01-19 12:18:53
categories: ["Archive", "Web Services"]
tags: ["NAV 2009 SP1", "PHP", "Web Services"]
permalink: /2010/01/19/connecting-to-nav-web-services-from-php/
---

### Prerequisites

Please read [this post](http://blogs.msdn.com/freddyk/archive/2010/01/19/connecting-to-nav-web-services-from.aspx) to get an explanation on how to modify the service tier to use NTLM authentication and for a brief explanation of the scenario I will implement in PHP.

BTW. Basic knowledge about PHP is required to understand the following post:-)

### Version and download

In my sample I am using PHP version 5.2.11, which I downloaded from [http://www.php.net](http://www.php.net), but it should work with any version after that.

In order to make this work you need to make sure that SOAP and CURL extensions are installed and enabled in php.ini.

PHP does not natively have support for NTLM nor SPNEGO authentication protocols, so we need to implement that manually. Now that sounds like something, which isn’t straightforward and something that takes an expert.   Fortunately there are a lot of these experts on the internet and I found this [post](http://rabaix.net/en/articles/2008/03/13/using-soap-php-with-ntlm-authentication) (written by Thomas Rabaix), which explains about how what’s going on, how and why. Note that this implements NTLM authentication and you will have to change the Web Services listener to use that.

### License of the code

The code can be used freely as long as you include Thomas’ copyright notice in the code.

```
/*
* Copyright (c) 2008 Invest-In-France Agency http://www.invest-in-france.org
*
* Author : Thomas Rabaix
*
* Permission to use, copy, modify, and distribute this software for any
* purpose with or without fee is hereby granted, provided that the above
* copyright notice and this permission notice appear in all copies.
*
* THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
* WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
* MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
* ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
* WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
* ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
* OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
*/
```

### “My” Version

I have modified Thomas’ version slightly (primarily removing debug echo’s etc.).

I have also changed the way the username and password is specified to be a script constant:

`define('USERPWD', 'domainuser:password');`

The Stream wrapper now looks like:

```
class NTLMStream
{
private $path;
private $mode;
private $options;
private $opened_path;
private $buffer;
private $pos;
/**
* Open the stream
*
* @param unknown_type $path
* @param unknown_type $mode
* @param unknown_type $options
* @param unknown_type $opened_path
* @return unknown
*/
public function stream_open($path, $mode, $options, $opened_path) {
$this->path = $path;
$this->mode = $mode;
$this->options = $options;
$this->opened_path = $opened_path;
$this->createBuffer($path);
return true;
}
/**
* Close the stream
*
*/
public function stream_close() {
curl_close($this->ch);
}
/**
* Read the stream
*
* @param int $count number of bytes to read
* @return content from pos to count
*/
public function stream_read($count) {
if(strlen($this->buffer) == 0) {
return false;
}
$read = substr($this->buffer,$this->pos, $count);
$this->pos += $count;
return $read;
}
/**
* write the stream
*
* @param int $count number of bytes to read
* @return content from pos to count
*/
public function stream_write($data) {
if(strlen($this->buffer) == 0) {
return false;
}
return true;
}
/**
*
* @return true if eof else false
*/
public function stream_eof() {
return ($this->pos > strlen($this->buffer));
}
/**
* @return int the position of the current read pointer
*/
public function stream_tell() {
return $this->pos;
}
/**
* Flush stream data
*/
public function stream_flush() {
$this->buffer = null;
$this->pos = null;
}
/**
* Stat the file, return only the size of the buffer
*
* @return array stat information
*/
public function stream_stat() {
$this->createBuffer($this->path);
$stat = array(
'size' => strlen($this->buffer),
);
return $stat;
}
/**
* Stat the url, return only the size of the buffer
*
* @return array stat information
*/
public function url_stat($path, $flags) {
$this->createBuffer($path);
$stat = array(
'size' => strlen($this->buffer),
);
return $stat;
}
/**
* Create the buffer by requesting the url through cURL
*
* @param unknown_type $path
*/
private function createBuffer($path) {
if($this->buffer) {
return;
}
$this->ch = curl_init($path);
curl_setopt($this->ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($this->ch, CURLOPT_HTTP_VERSION, CURL_HTTP_VERSION_1_1);
curl_setopt($this->ch, CURLOPT_HTTPAUTH, CURLAUTH_NTLM);
curl_setopt($this->ch, CURLOPT_USERPWD, USERPWD);
$this->buffer = curl_exec($this->ch);
$this->pos = 0;
}
}
```

The NTLM SOAP Client also uses the USERPWD constant defined above and looks like:

```
class NTLMSoapClient extends SoapClient {
function __doRequest($request, $location, $action, $version) {
$headers = array(
'Method: POST',
'Connection: Keep-Alive',
'User-Agent: PHP-SOAP-CURL',
'Content-Type: text/xml; charset=utf-8',
'SOAPAction: "'.$action.'"',
);
$this->__last_request_headers = $headers;
$ch = curl_init($location);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_POST, true );
curl_setopt($ch, CURLOPT_POSTFIELDS, $request);
curl_setopt($ch, CURLOPT_HTTP_VERSION, CURL_HTTP_VERSION_1_1);
curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_NTLM);
curl_setopt($ch, CURLOPT_USERPWD, USERPWD);
$response = curl_exec($ch);
return $response;
}
```

    

```
function __getLastRequestHeaders() {
return implode("n", $this->__last_request_headers)."n";
}
}
```

Putting this into the PHP script now allows you to connect to NAV Web Services system service in PHP and output the companies available on the service tier:

```
// we unregister the current HTTP wrapper
stream_wrapper_unregister('http');
// we register the new HTTP wrapper
stream_wrapper_register('http', 'NTLMStream') or die("Failed to register protocol");
```

```
// Initialize Soap Client
$baseURL = '
```

[`http://localhost:7047/DynamicsNAV/WS/';`](http://localhost:7047/DynamicsNAV/WS/';)  
`$client = new NTLMSoapClient($baseURL.'SystemService');`

```
// Find the first Company in the Companies
$result = $client->Companies();
$companies = $result->return_value;
echo "Companies:<br>";
if (is_array($companies)) {
foreach($companies as $company) {
echo "$company<br>";
}
$cur = $companies[0];
}
else {
echo "$companies<br>";
$cur = $companies;
}
```

Note that is return value is an array if there are multiple companies, but a company name if there is only one. I have NO idea why this is or whether I can write the code differently to avoid this.

Now I have the company I want to use in $cur and the way I create a URL to the Customer page is by doing:

```
$pageURL = $baseURL.rawurlencode($cur).'/Page/Customer';
echo "<br>URL of Customer page: $pageURL<br><br>";
```

and then I can create a Soap Client to the Customer Page:

```
// Initialize Page Soap Client
$page = new NTLMSoapClient($pageURL);
```

and using this, I read customer 10000 and output the name:

```
$params = array('No' => '10000');
$result = $page->Read($params);
$customer = $result->Customer;
echo "Name of Customer 10000:".$customer->Name."<br><br>";
```

Last, but not least – lets create a filter and read all customers in GB that has Location Code set to RED or BLUE:

```
$params = array('filter' => array(
array('Field' => 'Location_Code',
'Criteria' => 'RED|BLUE'),
array('Field' => 'Country_Region_Code',
'Criteria' => 'GB')
),
'setSize' => 0);
```

```
$result = $page->ReadMultiple($params);
$customers = $result->ReadMultiple_Result->Customer;
```

Note that Bookmark is an optional parameter and doesn’t need to be specified.

Now echo the customers and restore the http protocol – again, the return value might be an array and might not.

`echo "Customers in GB served by RED or BLUE warehouse:<br>";`

```
if (is_array($customers)) {
foreach($customers as $cust) {
echo $cust->Name."<br>";
}
}
else {
echo $customers->Name."<br>";
}
```

```
// restore the original http protocole
stream_wrapper_restore('http');
```

All of the above will output this when the script is opened in a browser (on my machine running NAV 2009SP1 W1)

[![image\_thumb\[3\]](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromPHP_9D82/image_thumb%5B3%5D_8bf25a39-e2e5-47de-8449-b676783dbd42.png "image_thumb[3]")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/ConnectingtoNAVWebServicesfromPHP_9D82/image%5B5%5D.png)

I hope this is helpful.

Good luck

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
