---
layout: post
title: "Connecting to NAV Web Services from the Cloud–part 5 out of 5"
date: 2011-03-15 06:49:13
categories: ["Archive", "Web Services"]
tags: ["AppFabric", "Authentication", "Azure", "C#", "Proxy", "Service Reference", "Servicebus", "Silverlight", "WCF", "Web Services", "Windows Phone 7", "XAML"]
permalink: /2011/03/15/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/
---

If you haven’t already read part 4 (and the prior parts) you should do so [here](/2011/01/27/connecting-to-nav-web-services-from-the-cloudpart-4-out-of-5/), before continuing to read this post.

In this post, I am going to create a small Windows Phone 7 application, which basically will be a phone version of the sidebar gadgets from [this post](/archive/2008/12/05/microsoft-windows-vista-gadget-my-stuff.aspx). When we are done, your Windows Phone 7 will look like:

[![WP7](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/3036.wp7_45a150ec.png?w=287&h=538 "WP7")](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/3036.wp7_45a150ec.png)

During the other posts, I have been describing how to make the Proxy and one way of securing this.

For this sample, I have created a new Proxy (Proxy2) and added 3 functions to this proxy:

```
Customer[] GetMyCustomers(string username, string password)
Vendor[] GetMyVendors(string username, string password)
Item[] GetMyItems(string username, string password)
```

You can imagine that these functions are implemented in the Proxy simply by authenticating and calling the corresponding function in the MyStuff codeunit from [this post](/archive/2008/12/05/microsoft-windows-vista-gadget-my-stuff.aspx), I will not go into further detail about how the Proxy is done.

## A small Console Application

Try the following:

-   In Visual Studio 2010, create a Windows Console Application
-   Add a service reference to [sb://navdemo.servicebus.windows.net/Proxy2/mex](//navdemo.servicebus.windows.net/Proxy2/mex "sb://navdemo.servicebus.windows.net/Proxy2/mex") and use the namespace Proxy2 (note that you need the Windows Azure AppFabric SDK to do this and you can download that [here](http://www.microsoft.com/downloads/en/details.aspx?FamilyID=39856a03-1490-4283-908f-c8bf0bfad8a5&displaylang=en)).
-   Use the following code in main:

```
static void Main(string[] args) 
{ 
    Proxy2.ProxyClassClient client = new Proxy2.ProxyClassClient("NetTcpRelayBinding_IProxyClass"); 
    foreach(Proxy2.Customer customer in client.GetMyCustomers("freddy", "password")) 
        Console.WriteLine(string.Format("{0} {1}", customer.Name, customer.Phone)); 
    Console.ReadLine(); 
}
```

-   Run the app, and you should get something like:

[![image](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/8838.image_326ffe73.png?w=681&h=346 "image")](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/8838.image_326ffe73.png)

This in effect calls my NAV proxy2 (which is placed in Redmond) through the Service bus and returns My Customers. I will try to keep the server running, but please understand that it might be down for various reasons (I might be doing development work on the Proxy).

## A Windows Phone 7 application

Now for the real thing.

In order to create solutions for Windows Phone 7, you will need the developer tools and they can be downloaded for free here: [http://create.msdn.com/en-us/home/getting\_started](http://create.msdn.com/en-us/home/getting_started "http://create.msdn.com/en-us/home/getting_started"):

There are three steps to the install process:

1.  [Download and install the Windows Phone Developer Tools](http://download.microsoft.com/download/1/7/7/177D6AF8-17FA-40E7-AB53-00B7CED31729/vm_web.exe) ([Release Notes](http://download.microsoft.com/download/1/7/7/177D6AF8-17FA-40E7-AB53-00B7CED31729/Release%20Notes%20-%20WPDT%20RTM.htm))
2.  [Download and install the Windows Phone Developer Tools January 2011 Update](http://download.microsoft.com/download/6/D/6/6D66958D-891B-4C0E-BC32-2DFC41917B11/WindowsPhoneDeveloperResources_en-US_Patch1.msp) ([Release Notes](<http://download.microsoft.com/download/6/D/6/6D66958D-891B-4C0E-BC32-2DFC41917B11/Release Notes - January 2011 Update.htm>)) \[Note: Installation may take several minutes and is complete when the install dialog box closes.\]
3.  [Download and install the Windows Phone Developer Tools Fix](http://download.microsoft.com/download/6/D/6/6D66958D-891B-4C0E-BC32-2DFC41917B11/VS10-KB2486994-x86.exe)

![Trist smiley](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/0003.wlemoticon-sadsmile_407ec7a3.png)

Anyway – when done – you are ready.

I created a Windows Phone Panorama Application from the templates:

[![image](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/7752.image_401294ae.png?w=862&h=599 "image")](/assets/images/2011/connecting-to-nav-web-services-from-the-cloudpart-5-out-of-5/7752.image_401294ae.png)

After this, I add a service reference to the Proxy ([sb://navdemo.servicebus.windows.net/Proxy2/mex](//navdemo.servicebus.windows.net/Proxy2/mex "sb://navdemo.servicebus.windows.net/Proxy2/mex")) with the namespace Proxy2.

Looking at the ServiceReferences.ClientConfig, you will see that only the http:// and the https:// endpoints are mentioned here as the Windows Phone doesn’t support sb://.

In Windows Phone applications it is common to have a ViewModel, which is the data for the app. The templates comes with a default ViewModel, which we need to modify:

Declaring the collections:

```
/// 
/// Collections for My stuff objects. 
///
```

public ObservableCollection MyCustomers { get; private set; } public ObservableCollection MyVendors { get; private set; } public ObservableCollection MyItems { get; private set; }

Initializing the ViewModel:

```
public MainViewModel() 
{ 
    this.MyCustomers = new ObservableCollection(); 
    this.MyVendors = new ObservableCollection(); 
    this.MyItems = new ObservableCollection(); 
}
```

Loading the data:

```
/// 
/// Load data into collections 
/// 
public void LoadData() {
    BasicHttpBinding binding;
    EndpointAddress endpoint;

    // Emulator doesn't support HTTPS reliably 
    binding = new BasicHttpBinding(BasicHttpSecurityMode.None); 
    endpoint = new EndpointAddress("http://navdemo.servicebus.windows.net/http/Proxy2/"); 
  
    Proxy2.ProxyClassClient client = new Proxy2.ProxyClassClient(binding, endpoint);

    client.GetMyCustomersCompleted += new EventHandler(client_GetMyCustomersCompleted); 
    client.GetMyCustomersAsync("freddy", "password");

    client.GetMyVendorsCompleted += new EventHandler(client_GetMyVendorsCompleted); 
    client.GetMyVendorsAsync("freddy", "password");

    client.GetMyItemsCompleted += new EventHandler(client_GetMyItemsCompleted); 
    client.GetMyItemsAsync("freddy", "password");

    this.IsDataLoaded = true; 
}

void client_GetMyCustomersCompleted(object sender, Proxy2.GetMyCustomersCompletedEventArgs e) 
{ 
    foreach (Proxy2.Customer customer in e.Result) 
        this.MyCustomers.Add(customer); 
}

void client_GetMyVendorsCompleted(object sender, Proxy2.GetMyVendorsCompletedEventArgs e) 
{ 
    foreach (Proxy2.Vendor vendor in e.Result) 
        this.MyVendors.Add(vendor); 
}

void client_GetMyItemsCompleted(object sender, Proxy2.GetMyItemsCompletedEventArgs e) 
{ 
    foreach (Proxy2.Item item in e.Result) 
        this.MyItems.Add(item); 
}
```

As you can see, the data is loaded using asynchronous data access – as everything on the phone works this way.

In the SampleData folder, I have modified the Sample Data xaml for the ViewModel to:

```
<local:MainViewModel 
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"       
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml" 
    xmlns:local="clr-namespace:MyStuff" 
    xmlns:proxy2="clr-namespace:MyStuff.Proxy2"> 
    
    <local:MainViewModel.MyCustomers> 
        <proxy2:Customer No="1" Name="Freddy Kristiansen" Phone="(425) 111-2222" EMail="freddy@cronus.com" /> 
        <proxy2:Customer No="2" Name="Pernille Kristiansen" Phone="(425) 222-3333" EMail="pernille@cronus.com" /> 
    </local:MainViewModel.MyCustomers>

    <local:MainViewModel.MyVendors> 
        <proxy2:Vendor No="1" Name="Niklas Kristiansen" Phone="(425) 333-4444" /> 
        <proxy2:Vendor No="2" Name="Mads Kristiansen" Phone="(425) 444-5555" /> 
        <proxy2:Vendor No="3" Name="Jonas Kristiansen" Phone="(425) 555-6666" /> 
    </local:MainViewModel.MyVendors>

    <local:MainViewModel.MyItems> 
        <proxy2:Item No="1" Description="Bike"  /> 
        <proxy2:Item No="2" Description="Car" /> 
    </local:MainViewModel.MyItems>

</local:MainViewModel>
```

Also the xaml for the panorama control in the MainPage has been modified to include 3 PanoramaItems, which binds to Customers, Vendors and Items:

```
<!--Panorama control--> 
<controls:Panorama Title="my stuff"> 
    <controls:Panorama.Background> 
        <ImageBrush ImageSource="PanoramaBackground.png"/> 
    </controls:Panorama.Background>

    <!-- My Customers --> 
    <controls:PanoramaItem Header="My Customers"> 
        <ListBox Name="lbMyCustomers" Margin="0,0,-12,0" ItemsSource="{Binding MyCustomers}" ManipulationStarted="ListBox_ManipulationStarted" ManipulationCompleted="ListBox_ManipulationCompleted" MouseLeftButtonUp="ListBox_MouseLeftButtonUp" ManipulationDelta="ListBox_ManipulationDelta"> 
            <ListBox.ItemTemplate> 
                <DataTemplate> 
                    <StackPanel Margin="0,0,0,17" Width="432"> 
                        <TextBlock Text="{Binding Name}" TextWrapping="NoWrap" Style="{StaticResource PhoneTextExtraLargeStyle}"/> 
                        <TextBlock Text="{Binding Phone}" TextWrapping="NoWrap" Margin="12,-6,12,0" Style="{StaticResource PhoneTextSubtleStyle}"/> 
                    </StackPanel> 
                </DataTemplate> 
            </ListBox.ItemTemplate> 
        </ListBox> 
    </controls:PanoramaItem>

    <!-- My Vendors --> 
    <controls:PanoramaItem Header="My Vendors"> 
        <ListBox Name="lbMyVendors" Margin="0,0,-12,0" ItemsSource="{Binding MyVendors}" ManipulationDelta="lbMyVendors_ManipulationDelta" ManipulationStarted="lbMyVendors_ManipulationStarted" MouseLeftButtonUp="lbMyVendors_MouseLeftButtonUp"> 
            <ListBox.ItemTemplate> 
                <DataTemplate> 
                    <StackPanel Margin="0,0,0,17" Width="432"> 
                        <TextBlock Text="{Binding Name}" TextWrapping="NoWrap" Style="{StaticResource PhoneTextExtraLargeStyle}"/> 
                        <TextBlock Text="{Binding Phone}" TextWrapping="NoWrap" Margin="12,-6,12,0" Style="{StaticResource PhoneTextSubtleStyle}"/> 
                    </StackPanel> 
                </DataTemplate> 
            </ListBox.ItemTemplate> 
        </ListBox> 
    </controls:PanoramaItem>

    <!-- My Items --> 
    <controls:PanoramaItem Header="My Items"> 
        <ListBox Name="lbMyItems" Margin="0,0,-12,0" ItemsSource="{Binding MyItems}"> 
            <ListBox.ItemTemplate> 
                <DataTemplate> 
                    <StackPanel Margin="0,0,0,17" Width="432"> 
                        <TextBlock Text="{Binding Description}" TextWrapping="NoWrap" Style="{StaticResource PhoneTextExtraLargeStyle}"/> 
                        <TextBlock Text="{Binding No}" TextWrapping="NoWrap" Margin="12,-6,12,0" Style="{StaticResource PhoneTextSubtleStyle}"/> 
                    </StackPanel> 
                </DataTemplate> 
            </ListBox.ItemTemplate> 
        </ListBox> 
    </controls:PanoramaItem>

</controls:Panorama>
```

That’s it – running the application in the Windows Phone Emulator should give you the desired output.

If you want to download the entire MyStuff solution and try it out, you can download it [here](http://www.freddy.dk/MyStuff.zip).

Note, that this application doesn’t do anything for local caching or updating of data, nor does it do anything when the App is tombstoned – it merely shows how to access data. The solution does however also support Dialing customers or vendors just by clicking on the line on the phone.

Enjoy

_Freddy Kristiansen  
_PM Architect  
Microsoft Dynamics NAV
