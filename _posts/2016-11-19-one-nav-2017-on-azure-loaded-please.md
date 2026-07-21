---
layout: post
title: "One NAV 2017 on Azure – Loaded please…"
date: 2016-11-19 12:52:03
categories: ["Archive", "Azure"]
tags: ["DEMO scripts", "Edit In Excel", "Flow", "Gallery", "NAV", "NAV 2017", "PowerApps", "Sharepoint", "That Was Easy"]
permalink: /2016/11/19/one-nav-2017-on-azure-loaded-please/
---

There is a huge difference in ordering a baked potato and a loaded baked potato.

The potato is the same, but it just becomes so much better will all the add-ons.

So, how would you like your NAV 2017 on Azure?

_**NAV 2017 on Azure, Loaded please…**_

of course:-)

This blog post is a visual walkthrough of how to setup NAV 2017 on Azure, Loaded. It consists of a LOT of screenshots explaining what to do…

For assistance on easier navigation, I have added links to the sections here:

1.  [Setup your NAV 2017 DEMO environment](#setup)
2.  [Install the Self Signed Certificate](#certificate)
3.  [Access NAV 2017 using the Web Client](#webclient)
4.  [Add NAV 2017 to the Waffle Menu](#waffle)
5.  [Create a PowerBI dashboard with data from NAV](#powerbi)
6.  [Embedding PowerBI](#embeddingpowerbi)
7.  [Configure the Outlook/Office Add-in](#configuringoutlook)
8.  [Setup Email](#email)
9.  [Setup Email Logging](#emaillogging)
10.  [Do you want to Edit In Excel](#editinexcel)
11.  [Create your PowerApps](#powerapps)
12.  [Create a Flow](#flow)
13.  [Access NAV 2017 via a SharePoint Portal](#sharepoint)
14.  [View a world map of your customers](#map)
15.  [Connect your Phone or Tablet to NAV 2017](#device)
16.  [Access NAV 2017 using the Windows Client](#windowsclient)
17.  [Connect to your Virtual Machine](#connect)
18.  [That was easy](#easy)

You can also choose to go through the entire story-like post right here:

### Setup your NAV 2017 DEMO environment

Go to [http://aka.ms/navdemodeploy](http://aka.ms/navdemodeploy)

[![1](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e8d5c-120-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e8d5c-120.png)

The system will redirect you to the new Azure Portal, and you will be asked to login and fill out a quesionnaire. You need to at least fill out Resource Group, VM Name and the admin password.

In order to get your NAV 2017 Fully Loaded, you also want to fill out the _BingMaps Key_ (get one [here](https://msdn.microsoft.com/da-dk/library/ff428642.aspx)), select _Yes_ to ClickOnce, select _yes_ to PowerBI and type in the administrator username (email) and password for the Office 365 where you want to integrate NAV 2017. Also say Yes to allow NAV to create a Portal with NAV 2017 Content.

Accept the terms and check Pin to dashboard.  
[![2](/assets/images/2016/one-nav-2017-on-azure-loaded-please/ac226-214-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/ac226-214.png)

Now deployment will start and in approx. 10 minutes you will be looking at this:  
[![5](/assets/images/2016/one-nav-2017-on-azure-loaded-please/105e8-53-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/105e8-53.png)

When Deployment is done, the new Portal will show the deployment and display 1 Succeeded. In the unlikely event it should say 1 Failed, you can click on the link to see what went wrong. It isn’t easy, but you will probably get there. Then remove the Resource Group and try again:  
[![6](/assets/images/2016/one-nav-2017-on-azure-loaded-please/0af3f-64-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/0af3f-64.png)

When you click the Public IP line, the portal will show the DNS name. Click the DNS Name and copy it to the clipboard:  
[![7](/assets/images/2016/one-nav-2017-on-azure-loaded-please/b0c52-73-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/b0c52-73.png)

Open a new Tab in the browser and navigate to the URL. The browser will probably show you this:  
[![8](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d1d62-83-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d1d62-83.png)

But after you refresh a few times, you should end up having the landing page:  
[![9](/assets/images/2016/one-nav-2017-on-azure-loaded-please/046a4-92-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/046a4-92.png)

The landing page here tells you that Installation is still running and you can view the installation status by clicking the link on the right:  
[![10](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f5e22-102-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f5e22-102.png)

After around 10-20 minutes, the status should say Installation  Complete:  
[![11](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88be7-1110-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88be7-1110.png)

and your Landing page should now be complete:  
[![12](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122.png)

### Install the Self Signed Certificate

If you are using a self-signed certificate (if you followed the process here, you are), click the Download Certificate link and follow the process for installing the certificate in the trusted root certificate authorities:  
[![20](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a3bfb-20-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a3bfb-20.png)  [![21](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d1b2b-215-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d1b2b-215.png)[![22](/assets/images/2016/one-nav-2017-on-azure-loaded-please/3cf89-222-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/3cf89-222.png) [![23](/assets/images/2016/one-nav-2017-on-azure-loaded-please/3d73a-231-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/3d73a-231.png) [![24](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a69a0-241-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a69a0-241.png)

Note, if you are going to connect to the Virtual Machine with your tablet or your phone, you would need to install the certificate on the device also. You can scan the QR code on the landing page on your device, to navigate to the landing page and download the certificate and the app for your device (link is at the bottom of the landing page).

### Access NAV 2017 using the Web Client

On the landing page, in the section with Accessing NAV 2017 using Office 365 authentication, you will find a link to _Access Web Client_:  
[![12](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122.png)

You will be redirected to the sign-in page. Enter your Office 365 credentials and press Sign in:  
[![25](/assets/images/2016/one-nav-2017-on-azure-loaded-please/92412-251-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/92412-251.png)

The first time you login, you will be asked to give permissions to sign you into the NAV Web Client with your credentials:  
[![26](/assets/images/2016/one-nav-2017-on-azure-loaded-please/9093d-261-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/9093d-261.png)

And after that you will open the Web Client:  
[![28](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e9cb5-281-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e9cb5-281.png)

### Add NAV 2017 to the Waffle Menu

If you want to have NAV 2017 in your Waffle Menu in Office 365, click the “Waffle” Menu in NAV 2017, which will navigate to your apps. Click the … menu next to your NAV 2017 app and say Pin to App Launcher:  
[![29](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf96-291-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf96-291.png)

### Create a PowerBI dashboard with data from NAV

Navigate to [https://powerbi.microsoft.com/en-us/](https://powerbi.microsoft.com/en-us/) and sign in with your Office 365 username and password:  
[![46](/assets/images/2016/one-nav-2017-on-azure-loaded-please/52c7c-462-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/52c7c-462.png)

and press Start:  
[![47](/assets/images/2016/one-nav-2017-on-azure-loaded-please/67d93-471-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/67d93-471.png)

In PowerBI, under Content Pack Library, Services, click Get:[![49](/assets/images/2016/one-nav-2017-on-azure-loaded-please/21a08-491-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/21a08-491.png)

Search for the NAV Content pack and press GET:  
[![50](/assets/images/2016/one-nav-2017-on-azure-loaded-please/36efb-50-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/36efb-50.png)

Now, for the PowerBI content pack, you will need the OData URL. Open your landing page and click the Get OData Feed Url:[![51](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d4c76-511-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d4c76-511.png)

Copy the Feed Url from the dialog and close it:  
[![52](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4d9ee-521-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4d9ee-521.png)

and in the setup dialog of the PowerBI, you need to paste the URL. Select authentication method Basic and put in your username and password for your admin user. Note: this is NOT your Office 365 username and password, it is the username and password for your NAV user. You can also create a Web Services key in the users dialog in NAV and use that instead of the password.

_**NOTE: if you are using the UNSECURE endpoint, your username and password will be sent over the wire in clear text and you should never do that with your production system:**_  
[![53](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f7913-531-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f7913-531.png)

After a short while, you will have a PowerBI dashboard based on NAV 2017 Data:  
[![54](/assets/images/2016/one-nav-2017-on-azure-loaded-please/b7acc-541-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/b7acc-541.png)

### Embedding PowerBI

Now where you have PowerBI, you want to setup embedded PowerBI in NAV 2017. Start your Web Client like described in the section about [starting the Web Client](#webclient). In the Role Center, Click Actions and select _Assisted Setup & Tasks_.  
[![31](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315.png)

In the list of Assisted Setup, select _Setup Azure Active Directory_ and go through the Wizard. All values should be pre-populated.

[![40](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c02af-40-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c02af-40.png) [![41](/assets/images/2016/one-nav-2017-on-azure-loaded-please/3f069-412-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/3f069-412.png)

[![42](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8d611-421-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8d611-421.png)

On the Role Center, locate the _Power BI Reports_ at the bottom and click _Get Started with Power BI_.

[![43](/assets/images/2016/one-nav-2017-on-azure-loaded-please/66dc6-431-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/66dc6-431.png) [![44](/assets/images/2016/one-nav-2017-on-azure-loaded-please/63ef6-442-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/63ef6-442.png)

Now, click the Power BI Reports selection and you will see a window with your Power BI Reports. The Reports are by default not enabled.  
[![55](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8fae5-55-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8fae5-55.png)

Click the Home menu, Select Edit List  
[![56](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c39bb-56-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c39bb-56.png)

Enable the Report and press OK  
[![57](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e8ba2-57-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e8ba2-57.png)

And you will now have an embedded PowerBI report on your Role Center  
[![58](/assets/images/2016/one-nav-2017-on-azure-loaded-please/97491-58-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/97491-58.png)

### Configure the Outlook/Office Add-in

Start your Web Client like described in the section about [starting the Web Client](#webclient). In the Role Center, Click Actions and select _Assisted Setup & Tasks_.  
[![31](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315.png)

Select Set up Outlook for Financials to start the setup wizard:  
[![32](/assets/images/2016/one-nav-2017-on-azure-loaded-please/41368-321-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/41368-321.png)

Go through the wizard, select your mail (or your organization) and type in your Office 365 username and password:  
[![33](/assets/images/2016/one-nav-2017-on-azure-loaded-please/32900-332-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/32900-332.png) [![34](/assets/images/2016/one-nav-2017-on-azure-loaded-please/62271-341-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/62271-341.png)

[![35](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a39b8-351-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a39b8-351.png) [![36](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c4983-361-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c4983-361.png)

Now navigate to Outlook in Office 365 and send and email to yourself to test the NAV 2017 Add-in.  
[![37](/assets/images/2016/one-nav-2017-on-azure-loaded-please/fc57c-371-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/fc57c-371.png)

When you click Dynamics NAV the first time, it will ask you to login to NAV 2017 using your Office 365 username and password:  
[![39](/assets/images/2016/one-nav-2017-on-azure-loaded-please/7ab16-391-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/7ab16-391.png)

### Setup Email

Setting up E-mail from NAV 2017 is really easy. Start your Web Client like described in the section about [starting the Web Client](#webclient). In the Role Center, Click Actions and select _Assisted Setup & Tasks_.  
[![31](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315.png)

In the _Assisted Setup_ list, Select _Setup Email_. This will guide you through a very easy 4-step wizard, which sets up the E-Mail connection. On Step 3, it is recommended that you click Send Test Mail to check your setup.  
[![65](/assets/images/2016/one-nav-2017-on-azure-loaded-please/da4d1-65-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/da4d1-65.png)  [![66](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f4037-66-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f4037-66.png)

[![67](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d7d8e-67-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/d7d8e-67.png)  [![68](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f12e9-68-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f12e9-68.png)

### Setup Email Logging

Setting up E-mail Logging from NAV 2017 is really easy. Start your Web Client like described in the section about [starting the Web Client](#webclient). In the Role Center, Click Actions and select _Assisted Setup & Tasks_.  
[![31](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/88e7e-315.png)

In the _Assisted Setup_ list, Select _Setup Email Logging_. This will guide you through an easy 3 step wizard, which sets up Email logging.  
[![69](/assets/images/2016/one-nav-2017-on-azure-loaded-please/61e8f-69-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/61e8f-69.png) ![70](/assets/images/2016/one-nav-2017-on-azure-loaded-please/70.png)

[![71](/assets/images/2016/one-nav-2017-on-azure-loaded-please/09a55-711-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/09a55-711.png)

### Do you want to Edit In Excel

**Good news:** Everything has already been setup for you. Start your Web Client like described in the section about [starting the Web Client](#webclient). In the Navigation pane on the Role Center, click Customers and select Edit In Excel in the ribbon.  
[![59](/assets/images/2016/one-nav-2017-on-azure-loaded-please/61fca-59-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/61fca-59.png)

Now the devil is always in the detail. You might run into issues if Excel is installed with a different Office 365 account. I had to install Excel on a different machine, using the Office 365 credentials used to setup NAV 2017, which was pretty annoying. When Excel opens, you will have to enable editing:  
[![60](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4c817-60-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4c817-60.png)

and then sign in to the Excel Add-in:  
[![61](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a8a42-611-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/a8a42-611.png)

Like with embedded PowerBI, you will have to sign in and give permissions to the Excel Add-in:  
[![62](/assets/images/2016/one-nav-2017-on-azure-loaded-please/7d796-621-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/7d796-621.png)  [![63](/assets/images/2016/one-nav-2017-on-azure-loaded-please/fcc20-631-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/fcc20-631.png)

and then you can edit your customers in Excel:  
[![64](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4a906-641-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4a906-641.png)

### Create your PowerApps

So, you want to create PowerApps… – wait no more. Navigate to [https://powerapps.microsoft.com/en-us/](https://powerapps.microsoft.com/en-us/) and sign in with your Office 365 account:  
[![73](/assets/images/2016/one-nav-2017-on-azure-loaded-please/1a3ee-731-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/1a3ee-731.png)

and start your free trial:  
[![74](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4ed93-74-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/4ed93-74.png)

Wait a few minutes…  
[![75](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8f2fe-75-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8f2fe-75.png)

And now you are ready. Start by adding a New Connection (top right hand corner):  
[![76](/assets/images/2016/one-nav-2017-on-azure-loaded-please/097af-76-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/097af-76.png)

In the new connections dialog, add your UNSECURE OData endpoint. On the landing page you can right click the _View UNSECURE OData Web Services_ and copy the link address to the clipboard.

As the Username and password you use your NAV admin username and the password (NOT your Office 365 credentials). Company name is the Company name from NAV, if you are using W1, the company name is _CRONUS International Ltd.  
_[![77](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8ba1c-77-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8ba1c-77.png)

Having created your Connection, it is time to create your first PowerApp. Press _New app_ and select to use PowerApps Studio for web:[![79](/assets/images/2016/one-nav-2017-on-azure-loaded-please/1918b-79-1.png)  
](/assets/images/2016/one-nav-2017-on-azure-loaded-please/1918b-79.png)

In the create an app window, press the white right arrow and locate your data/connection:  
[![80](/assets/images/2016/one-nav-2017-on-azure-loaded-please/78c3f-80-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/78c3f-80.png)

Select your connection and your dataset:  
[![81](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8248b-811-1.png)  
](/assets/images/2016/one-nav-2017-on-azure-loaded-please/8248b-811.png)

and select the Customer table:  
[![82](/assets/images/2016/one-nav-2017-on-azure-loaded-please/857f5-821-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/857f5-821.png)

and wait…  
[![83](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e6452-831-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/e6452-831.png)

Voila, your first PowerApp is ready. Press the _Play_ button in the top right corner:  
[![84](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2046d-84-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2046d-84.png)

and… here you are – your first PowerApp:  
[![85](/assets/images/2016/one-nav-2017-on-azure-loaded-please/080de-85-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/080de-85.png)

### Create a Flow

Flows lives in the same area as PowerApps. In order to create a Flow, you need to have a Connection created and you can find information how to do this in the previous section about [PowerApps](#powerapps). Click Flows:  
[![86](/assets/images/2016/one-nav-2017-on-azure-loaded-please/69088-86-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/69088-86.png)

Create a new Flow and select the Template: _Dynamics NAV: When a record is created._  
[![87](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2de82-87-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2de82-87.png)

Select the Customer table  
[![88](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c2300-88-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/c2300-88.png)

As the next step, select the send email step:  
[![89](/assets/images/2016/one-nav-2017-on-azure-loaded-please/be0e5-89-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/be0e5-89.png)

Specify the reciever of the email and a text. In the text you can embed fields from the connection:  
[![90](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2d326-90-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2d326-90.png)

Your new flow is running…  
[![92](/assets/images/2016/one-nav-2017-on-azure-loaded-please/5ee8a-921-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/5ee8a-921.png)

and after creating a customer in NAV 2017, I got this email:  
[![93](/assets/images/2016/one-nav-2017-on-azure-loaded-please/05d6b-93-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/05d6b-93.png)

If this was a session with an audience, I would ask: _Does anybody knows why the customer name is empty?_ and if the audience had been working with NAV for more than just a short time, they would all answer: _Because the created trigger is invoked when the record is created, not when you have filled out the fields and pressed OK on the customer card._ It might have been a better demo to hook on to the customer modified event in Flow.

### Access NAV 2017 via a SharePoint Portal

On the landing page, click the link called _Access SharePoint Site_.  
[![12](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122.png)

and you will be taken to a SharePoint Portal that looks like this:  
[![95](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f3421-95-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/f3421-95.png)

In the waffle menu, you will find links to your Apps (PowerApps and Flow are still being setup?)  
[![96](/assets/images/2016/one-nav-2017-on-azure-loaded-please/af1f4-96-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/af1f4-96.png)

And in the Web Client you will find a button in the top right corner, which also will take you to your SharePoint portal:  
[![96](/assets/images/2016/one-nav-2017-on-azure-loaded-please/6b54a-961-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/6b54a-961.png)

### View a world map of your customers

On the landing page, click the link called _Show Customer Map_.  
[![12](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122.png)

The browser might ask for permission to use your physical location and then show a customer map centered in your current location.  
[![98](/assets/images/2016/one-nav-2017-on-azure-loaded-please/432c4-98-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/432c4-98.png)

When you zoom out, you will see all customers and when you hover over a customer, you will get a link for navigating to the customer in the Web Client.  
[![100](/assets/images/2016/one-nav-2017-on-azure-loaded-please/89668-100-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/89668-100.png)

### Connect your Phone or Tablet to NAV 2017

In the top left corner of your landing page, you will see a QR code, with the device you want to use, scan that barcode:  
[![barcode](/assets/images/2016/one-nav-2017-on-azure-loaded-please/15a49-barcode-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/15a49-barcode.png)

Your device will now navigate to your landing page. At the bottom of the landing page, you will find links to download the NAV 2017 app for your device:  
[![94](/assets/images/2016/one-nav-2017-on-azure-loaded-please/9306b-94-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/9306b-94.png)

Before you open your app and connect to your NAV 2017, you will need to install the self-signed certificate on your device as well. At the top of the landing page, you will find a description on how to do and a link to download the self-signed certificate.

Last, but not least, the easiest way to configure the app on your device to your instance of NAV 2017 is to click the Configure App link on the landing page. It should autoconfigure the app to the URL for NAV 2017.

_**Note: there are two configure App links. One with Office 365 authentication and one with username/password authentication.  
**_[![12](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/2bf24-122.png)

### Access NAV 2017 using the Windows Client

The Windows Client is available for ClickOnce deployment on the NAV 2017 environment. On the landing page, click the _Install Windows Client_ link under Office 365 authentication (or under username/password authentication). The system will open a page, on which you need to accept the license terms and press _install now_.

[![101](/assets/images/2016/one-nav-2017-on-azure-loaded-please/75321-1011-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/75321-1011.png)

The Windows Client will now be installed on your PC using ClickOnce. Read more [here](https://msdn.microsoft.com/en-us/library/71baz9ah\(v=vs.140\).aspx).  
[![install1](/assets/images/2016/one-nav-2017-on-azure-loaded-please/20529-install1-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/20529-install1.png)  [![install2](/assets/images/2016/one-nav-2017-on-azure-loaded-please/879a8-install2-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/879a8-install2.png)

When the installation is done, you will be asked to authentication and you will now be running the Windows Client locally against a NAV Service Tier on Azure.  
![windowsclient](/assets/images/2016/one-nav-2017-on-azure-loaded-please/windowsclient.png)

### Connect to your Virtual Machine

If you need to connect to your Virtual Machine, then you will find a link to a Remote Desktop connection to your Virtual Machine (_server1_) on the landing page. Click that and login with your Virtual Machine Administrator username and password.  
[![14](/assets/images/2016/one-nav-2017-on-azure-loaded-please/029fb-141-1.png)](/assets/images/2016/one-nav-2017-on-azure-loaded-please/029fb-141.png)

![easy](/assets/images/2016/one-nav-2017-on-azure-loaded-please/50aa3-easy.png)

[thatwaseasy.m4a](/assets/images/2016/one-nav-2017-on-azure-loaded-please/665aa-thatwaseasy.m4a)

BTW – if you ever succeeded in setting up all of this without the Gallery image, please write in the comments how much time it took, from start to finish… (or maybe how much time it took to get some of the way:-)) – we can multiply this with a lot of people – to see how much time people actually save by using the NAV 2017 image on Azure.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
