---
layout: post
title: "Automatic AppSource Submission of Business Central apps"
date: 2022-09-22 17:33:15
categories: ["BcContainerHelper", "CI/CD", "PowerShell"]
tags: ["AppSource"]
permalink: /2022/09/22/automatic-appsource-submission-of-business-central-apps/
---

A request from many partners have been to be able to automatically submit apps to AppSource Validation from their DevOps setup. The Ingestion API for Partner Center supports all offer types in AppSource and can be used to submit updates to Business Central apps to AppSource.

Unfortunately, the API comes with very little documentation and the sparse documentation doesn’t contain any information about Business Central. If you however want to investigate things by yourself, the primary docs are:

-   [https://docs.microsoft.com/en-us/azure/marketplace/azure-app-apis](https://docs.microsoft.com/en-us/azure/marketplace/azure-app-apis) – which is a one pager, describing how to authenticate and then points to
-   [https://ingestionapi-swagger.azureedge.net/](https://ingestionapi-swagger.azureedge.net/) – for documentation on how to do things.
-   [https://arsenvlad.medium.com/using-partner-center-ingestion-api-for-managing-azure-application-offers-in-azure-marketplace-b47b290dd947](https://arsenvlad.medium.com/using-partner-center-ingestion-api-for-managing-azure-application-offers-in-azure-marketplace-b47b290dd947) – a video describing the API. Unfortunately, the video isn’t up to date.

Instead of describing the API and have all partners create their own code to investigate and submit their apps, the latest BcContainerHelper (4.0.3) contains functions to do just this, using the Partner Center Ingestion API.

## Authentication

In order to get started, you need a way to authenticate to the Partner Center ingestion API. This can be done using Service 2 Service authentication (which is recommended for workflows and pipelines) or you can use User Impersonation.

### Service 2 Service (S2S)

To get started with S2S, you need to complete Step 1 from here: [https://docs.microsoft.com/en-us/azure/marketplace/azure-app-apis](https://docs.microsoft.com/en-us/azure/marketplace/azure-app-apis). After this, you should have a ClientID and a ClientSecret, stored in a KeyVault and the actual authentication is done using the **New-BcAuthContext** function from BcContainerHelper:

$authcontext = New-BcAuthContext \`
    -clientID ($PublisherAppClientIdSecret.SecretValue | Get-PlainText) \`
    -clientSecret $PublisherAppClientSecretSecret.SecretValue \`
    -Scopes "https://api.partner.microsoft.com/.default" \`
    -TenantID "<your AAD tenant>"

### User Impersonation

If you, for some reason, can’t or won’t create an AAD App registration for S2S authentication, then the other option for getting an authcontext is to use user impersonation.

$authcontext = New-BcAuthContext \`
    -includeDeviceLogin \`
    -Scopes "https://api.partner.microsoft.com/user\_impersonation offline\_access" \`
    -tenantID "<your AAD tenant>"

This will invoke the device flow and display a code, which you need to use when authenticating to [https://aka.ms/devicelogin](https://aka.ms/devicelogin)

Now, you will have an authcontext, which contains an access token and a refresh token. The access token can typically be used for authentication for 60 minutes. The refreshtoken can be used to get a new access token for typically 90 days and you can store the refresh token in a keyvault to be able to get a new authContext based on the refreshtoken by using this code:

$authcontext = New-BcAuthContext \`
    -refreshToken $refreshtoken \`
    -Scopes "https://api.partner.microsoft.com/user\_impersonation offline\_access" \`
    -tenantID "<your AAD tenant>"

## AuthContext

The **$authContext** needs to be specified to all functions and you should never need to look insider the Auth Context. Reason for carrying around the AuthContext and not “just” an AccessToken is, that the AccessToken expires after 60 minutes and frequently, pipelines/scripts/tests run longer than 60 minutes. For this reason, the AuthContext contains enough information to renew the AccessToken and all functions in ContainerHelper will check and renew if necessary.  If you ever need to create an authorization header based on the AuthContext, you can do it like this:

$authContext = Renew-BcAuthContext -bcAuthContext $authContext 
$headers += @{ "Authorization" = "Bearer $($authcontext.AccessToken)" }

But again, you won’t need to for the scenarios below, the authcontext is automatically renewed in all functions and uses either the ClientID and ClientSecret or the refresh token to renew.

When you have an AuthContext – you can validate by running:

Get-AppSourceProduct -authContext $authcontext -silent

And you should see a list of all your apps in AppSource.

## Submitting an app to AppSource Validation

Currently the Partner Center Ingestion API doesn’t support submitting an App to production directly. You must promote (click Go Live) the submission after technical validation.

The **New-AppSourceSubmission** function does include an **\-autoPromote** parameter, which waits for the technical validation to complete and after that, it will automatically invoke the promote action. When the API supports **AutoPromote**, the inner workings of the function will change to utilize this instead of waiting.

With your AuthContext from above, you can get all your products by using:

$products = Get-AppSourceProduct -authContext $authcontext -silent

and get the Product Id of the product you want to work with:

$productName = 'BingMaps.AppSource'
$productId = ($products | Where-Object { $\_.name -eq $productName }).id

If you want to get full details of the product, you can use:

$product = Get-AppSourceProduct -authContext $authcontext -productId $productId -includeAll

Now, you can inspect properties of $product. Exaamples:

$product.FeatureAvailability\[0\].marketStates

marketCode state   
---------- -----   
AE         Disabled
AT         Disabled
AU         Disabled
BE         Disabled
CA         Disabled
CH         Disabled
CO         Disabled
CZ         Disabled
DE         Disabled
DK         Enabled 
EE         Disabled
ES         Disabled 
...

$product.Property

resourceType          : AzureProperty
industries            : {}
categories            : {}
additionalCategories  : {}
submissionVersion     : 
productTags           : {}
appVersion            : 3.0.164.0
useEnterpriseContract : False
termsOfUse            : The terms that applies to Microsoft Dynamics 365 Business Central can be downloaded here: 
                        https://www.microsoft.com/en-us/licensing/product-licensing/products.aspx 
                         
                        If you have signed up to the trial version of Microsoft Dynamics 365 Business Central with an organizational email address, you are governed by the Microsoft 
                        Online Service Trial Agreement, which can be found here: https://go.microsoft.com/fwlink/?linkid=828977 
extendedProperties    : {}
hideKeys              : {}
applicableProducts    : {}
marketingOnlyChange   : False
globalAmendmentTerms  : 
customAmendments      : {}
leveledIndustries     : 
leveledCategories     : @{geolocation=System.Object\[\]}
@odata.etag           : "0800a065-0000-0800-0000-62ee043a0000"
id                    : 009f7479-f506-216f-d445-5315d5fb9e62

You can also get information about your latest submission by using:

Get-AppSourceSubmission -authContext $authcontext -productId $productId -includeWorkflowDetails
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions/1152921505695125040
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions/1152921505695125040/workflowdetails

resourceType       : Submission
state              : Published
substate           : InStore
targets            : {@{type=Scope; value=Preview}}
resources          : {@{type=Availability; value=4b8c7499-1cb3-acf6-9b5b-c45d2a7040e2}, @{type=Listing; value=0202f16e-e12b-a657-124e-27ed76542ad5}, @{type=Package; 
                     value=2d212b32-f3f4-4d43-ac9b-5783f7a6099f}, @{type=Property; value=ae0f3f9e-b893-bbde-4dd1-3ac0b50496ef}...}
publishedTimeInUtc : 2022-08-04T13:53:31.2590809Z
pendingUpdateInfo  : @{updateType=Create; status=Completed}
releaseNumber      : 28
friendlyName       : Submission 28
areResourcesReady  : True
id                 : 1152921505695125040
WorkflowDetails    : {@{type=Push; state=Success; targetEnvironment=Preview; workflowSteps=System.Object\[\]; startDateTimeInUtc=2022-08-04T10:37:37.4038218; 
                     completeDateTimeInUtc=2022-08-04T10:51:26.8132496}, @{type=Push; state=Success; targetEnvironment=Live; workflowSteps=System.Object\[\]; 
                     startDateTimeInUtc=2022-08-04T10:51:57.0499235; completeDateTimeInUtc=2022-08-04T13:53:30.8277395}}

The state/substate of your submission can have the following values:

-   **Published/InStore** – your submission was published and the offer is available in AppSource
-   **InProgress/Submitted** – your submission was submitted and validation is running (during this phase, your submission can be cancelled)
-   **Published/ReadyToPublish** – your submission is in preview and ready to promote / Go Live. At this time you can create a new submission if you don’t want to take this version live.
-   ?/? – your submission is in certification/publish phase. This phase cannot be cancelled and you cannot create a new submission until you app has been published.

You can also monitor a submission while running using the Get-AppSourceSubmission function:

$submission = Get-AppSourceSubmission -authContext $authcontext -productId $productId -includeWorkflowDetails
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions/1152921505695131860
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions/1152921505695131860/workflowdetails

PS C:\\> $submission.WorkflowDetails

type                  : Push
state                 : InProgress
targetEnvironment     : Preview
workflowSteps         : {@{name=Automated validation; state=Success; startDateTimeInUtc=2022-08-06T06:38:53.8153199; completeDateTimeInUtc=2022-08-06T06:42:08.3546437}, 
                        @{name=Preview Creation; state=Success; startDateTimeInUtc=2022-08-06T06:42:11.2274863; completeDateTimeInUtc=2022-08-06T06:47:30.8841703}, @{name=Publisher 
                        Signoff; state=NotStarted; startDateTimeInUtc=0001-01-01T00:00:00; completeDateTimeInUtc=0001-01-01T00:00:00}}
startDateTimeInUtc    : 2022-08-06T06:37:47.6696794
completeDateTimeInUtc : 0001-01-01T00:00:00

type                  : Push
state                 : NotStarted
targetEnvironment     : Live
workflowSteps         : {@{name=Certification; state=NotStarted; startDateTimeInUtc=0001-01-01T00:00:00; completeDateTimeInUtc=0001-01-01T00:00:00}, @{name=Publish; 
                        state=NotStarted; startDateTimeInUtc=0001-01-01T00:00:00; completeDateTimeInUtc=0001-01-01T00:00:00}}
startDateTimeInUtc    : 0001-01-01T00:00:00
completeDateTimeInUtc : 0001-01-01T00:00:00

The workflowDetails is an array, consisting of two PSCustomObjects. The first is the details about the preview (before pressing Go Live button) and the second is the details about the Go Live (after pressing Go Live button). A little later, the workflowDetails will look like this:

$submission = Get-AppSourceSubmission -authContext $authcontext -productId $productId -includeWorkflowDetails
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions/1152921505695131860
GET https://api.partner.microsoft.com/v1.0/ingestion/products/5fbe0803-a545-4504-b41a-d9d158112360/submissions/1152921505695131860/workflowdetails

PS C:\\> $submission

resourceType       : Submission
state              : Published
substate           : ReadyToPublish
targets            : {@{type=Scope; value=Preview}}
resources          : {@{type=Availability; value=a179fdd3-9950-5235-493d-ffe6dd02b4aa}, @{type=Listing; value=280ae301-a6ca-293c-88b3-8b49cfe58480}, @{type=Package; 
                     value=6ba6a084-9bba-4c3c-8eae-cea773809696}, @{type=Property; value=009f7479-f506-216f-d445-5315d5fb9e62}...}
publishedTimeInUtc : 2022-08-06T06:47:36.5830633Z
pendingUpdateInfo  : @{updateType=Create; status=Completed}
releaseNumber      : 29
friendlyName       : Submission 29
areResourcesReady  : True
id                 : 1152921505695131860
WorkflowDetails    : {@{type=Push; state=Success; targetEnvironment=Preview; workflowSteps=System.Object\[\]; startDateTimeInUtc=2022-08-06T06:37:47.6696794; 
                     completeDateTimeInUtc=2022-08-06T06:47:35.721944}, @{type=Push; state=NotStarted; targetEnvironment=Live; workflowSteps=System.Object\[\]; 
                     startDateTimeInUtc=0001-01-01T00:00:00; completeDateTimeInUtc=0001-01-01T00:00:00}}
type                  : Push
state                 : Success
targetEnvironment     : Preview
workflowSteps         : {@{name=Automated validation; state=Success; startDateTimeInUtc=2022-08-06T06:38:53.8153199; completeDateTimeInUtc=2022-08-06T06:42:08.3546437}, 
                        @{name=Preview Creation; state=Success; startDateTimeInUtc=2022-08-06T06:42:11.2274863; completeDateTimeInUtc=2022-08-06T06:47:30.8841703}, @{name=Publisher 
                        Signoff; state=Success; startDateTimeInUtc=2022-08-06T06:47:32.6862082; completeDateTimeInUtc=2022-08-06T06:47:35.721944}}
startDateTimeInUtc    : 2022-08-06T06:37:47.6696794
completeDateTimeInUtc : 2022-08-06T06:47:35.721944

type                  : Push
state                 : NotStarted
targetEnvironment     : Live
workflowSteps         : {@{name=Certification; state=NotStarted; startDateTimeInUtc=0001-01-01T00:00:00; completeDateTimeInUtc=0001-01-01T00:00:00}, @{name=Publish; 
                        state=NotStarted; startDateTimeInUtc=0001-01-01T00:00:00; completeDateTimeInUtc=0001-01-01T00:00:00}}
startDateTimeInUtc    : 0001-01-01T00:00:00
completeDateTimeInUtc : 0001-01-01T00:00:00

This means that your partner center UI will look like this:

![](/assets/images/2022/automatic-appsource-submission-of-business-central-apps/image.png)

And you can press **Go Live** (or run **Promote-AppSourceSubmission**) in order to take the app live (or invoke certification / marketing validation)

## New-AppSourceSubmission

At this state, you can submit a new version for validation – you do not need to take the submission live. Submitting a new version of your app is done using the **New-AppSourceSubmission** function:

New-AppSourceSubmission -authContext $authContext -productId $product.Id -appFile $appFile -silent
Extracting C:\\Users\\freddyk\\Downloads\\BingMaps.AppSource-main-Apps-3.0.164.0\\Freddy Kristiansen\_BingMaps.AppSource\_3.0.164.0.app
Automated validation........ Success
Preview Creation.......... Success
Publisher Signoff Success
New AppSource submission succeeded

resourceType       : Submission
state              : Published
substate           : ReadyToPublish
targets            : {@{type=Scope; value=Preview}}
resources          : {@{type=Availability; value=553dfb07-1fe0-6e19-a136-cf6f310127b4}, @{type=Listing; value=383dfff3-5479-dba7-7480-f61f3022ecfd}, @{type=Package; 
                     value=141ca192-e27b-4529-be55-f96de1e1710d}, @{type=Property; value=86c43c9b-6184-ff69-4621-a644cc167977}...}
publishedTimeInUtc : 2022-08-06T07:26:10.2335664Z
pendingUpdateInfo  : @{updateType=Create; status=Completed}
releaseNumber      : 31
friendlyName       : Submission 31
areResourcesReady  : True
id                 : 1152921505695131645

If you include the **\-autoPromote** flag, the function will wait and automatically promote the submission to production / Go Live. If you include the **doNotWait** flag, the function will not wait for completion. If you include the **autoPromote** AND the **doNotWait** flag, the function will (in the current version) **still wait** for the preview creation to be complete and then promote the submission. For a later version of the API (when **IsAutoPromote** is supported for Business Central Apps) the function will be rewritten to utilize the API’s support of autoPromote instead of waiting and calling the API again.

This also means, that if you cancel the function while waiting for preview to complete – the **autoPromote** flag has no effect.

When you specify –**LibraryAppFiles** to **New-AppSourceSubmission**, you can specify an array of files. If you only specify one file, it is uploaded as is to Partner Center. If you specify an array of files, they will be zipped together into a file with the same name as the AppFile specified, followed by **.libraries.zip**, meaning that if you want to control the filename, you need to do the zipping yourself.

If your submission fails, you will be able to see the error message in the portal. Currently, I have not found a way to get the error message through the API. I have submitted a question for the team and expect to know more soon (and change the code)

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
