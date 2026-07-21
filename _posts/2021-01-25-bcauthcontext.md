---
layout: post
title: "BcAuthContext"
date: 2021-01-25 10:52:20
categories: ["BcContainerHelper", "BcSaaS"]
tags: ["AAD", "AAD App", "Access Token", "BcAuthContext", "Refresh Token"]
permalink: /2021/01/25/bcauthcontext/
---

The latest version of BcContainerHelper ([BcContainerHelper version 2.0.1 | Freddys blog](/2021/01/24/bccontainerhelper-version-2-0-1/)) comes with a new concept called a BcAuthContext. A BcAuthContext is really just a hashtable with authentication information for a Business Central online tenant.

This blog post describes the concept and how to obtain and refresh a BcAuthContext. Subsequent blog posts will describe how to use them.

A BcAuthContext is created using New-BcAuthContext. New-BcAuthContext supports 4 different OAuth2 flows:

1.  **Devicecode** – this flow is where you will be asked to open a browser on [http://aka.ms/devicelogin](http://aka.ms/devicelogin) and enter a code, which is shown by the flow and then login using your AAD credentials. Multi factor authentication is supported using this flow.
2.  **Refresh\_token** – this flow is used to get an access token based on a refresh token. The refresh token is returned by the devicecode flow and the password flow.
3.  **Password** – this flow supports getting an access token based on the Username and Password from a user. This flow does not work when your user is setup for multi factor authentication (MFA). It is not recommended to use this flow.
4.  **Client\_Credentials** – this flow is a service 2 service flow, where you specify a ClientID and a ClientSecret of an AAD App with access to the resources. Service 2 service flows can today be used for some of the Business Central automation APIs, but we are working on covering all CI/CD scenarios with this flow.

Once you create a BcAuthContext, the access token is valid for 60 minutes. The BcAuthContext contains information about the expiration timestamp AND the BcAuthContext contains enough information to get a new access token. All BcContainerHelper functions, that takes a BcAuthContext as a parameter will implicitly call Renew-BcAuthContext, which will renew the access token if the validity period is less than 10 minutes. Renewing the auth context will return immediately if the access token is still valid.

# Recommendation

For pipelines connecting to Business Central online environments, I recommend using the devicecode flow to create an authcontext manually and then store the refresh token in a keyvault accessible from the pipeline. This refresh token must be renewed every 3 months (validity 90 days). The devicecode and refresh\_token flows are both enabled on the default AAD App (ClientID = “1950a258-227b-4e31-a9cf-717495945fc2”) which is a Microsoft Azure PowerShell well known app.

You can use Set-AzKeyVaultSecret to store the refresh token in a keyvault for access from your pipeline.

# The devicecode flow

The easiest way to create a BcAuthContext is by issuing this command:

```
$authContext = New-BcAuthContext -includeDeviceLogin
```

which should give an output like this:

![](/assets/images/2021/bcauthcontext/auth1-1.png)

Now you can open a browser using [https://aka.ms/devicelogin](https://aka.ms/devicelogin) or [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin), enter the code DEM276827 and then login with your AAD credentials. You will see a dialog like this:

![](/assets/images/2021/bcauthcontext/auth3.png)

Indicating that you will be signed in using an AAD App called Microsoft Azure PowerShell. After the login you will see something like:

![](/assets/images/2021/bcauthcontext/auth2-3.png)

And if you display the content of the **$authContext** variable you will see something like:

![](/assets/images/2021/bcauthcontext/auth4-1.png)

By default, the devicecode flow will wait 5 minutes for the device login to complete. You can specify a different wait time using the deviceLoginTimeout parameter (default \[TimeSpan\]::FromMinutes(5))

# The refresh\_token flow

With the refresh token from the devicecode flow, you can create a new BcAuthContext (in your pipeline) using:

```
$authContext2 = New-BcAuthContext -refreshToken $refreshToken
```

which should give an output like this:

![](/assets/images/2021/bcauthcontext/auth5-1.png)

You have to refresh the refresh token in your keyvault for every 90 days. Note that the refresh token is invalidated if the password of the authenticated user changes.

# The password flow

With the password flow, you can authenticate using a AAD username and password. Storing your username and password in an app or in a keyvault is not recommended and multi factor authentication (MFA) is NOT supported, but for testing purposes (or for getting a refresh token), you can do like this:

```
$authContext7 = New-BcAuthContext -credential $credential
```

which should give an output like this:

![](/assets/images/2021/bcauthcontext/auth7.png)

$authcontext7.RefreshToken can stored in a Keyvault and can be used subsequently to get a new accesstoken.

# The client\_credentials flow

The client\_credentials flow is currently only used for some automation APIs. Over time, all CI/CD tasks should be able to run using this flow, but today only some automation APIs can be reached with this. You need to create an AAD App as described here. [https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/automation-apis-using-s2s-authentication](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/administration/automation-apis-using-s2s-authentication) and specify the ClientID and ClientSecret for this app to the New-BcAuthContext:

```
$authcontext4 = New-BcAuthContext `
    -clientID $PublisherAppClientIdSecret.SecretValueText `
    -clientSecret $PublisherAppClientSecretSecret.SecretValue
```

which should give an output like this:

![](/assets/images/2021/bcauthcontext/auth6.png)

Note that service 2 service doesn’t authenticate as a specific user in Business Central and therefore this flow currently cannot be used for all purposes.

# Renewing the BcAuthContext

Once you have a BcAuthContext, the Access Token is only valid for 60 minutes, but you can refresh the access token by running:

```
$authContext = ReNew-BcAuthContext -bcAuthContext $authContext
```

If the auth context is still valid, the function returns immediately. If not, you might see something like:

![](/assets/images/2021/bcauthcontext/auth8.png)

The renew function will renew the access token if the validity period is below 300 seconds. You can specify a different minimum validity period if you need to.

# Other parameters for New-BcAuthContext

Beside the parameters for the various flows described above, there are few other parameters indicating which resources you are authenticating to and what authority you are using as authenticator. By default these parameters are set to authenticate towards online Business Central tenants. If you are authenticating towards docker instances or onprem instances, you would have to modify these parameters. The parameters and their default values are:

**ClientID** is used to specify which AAD App is used for the authentication. The default value is 1950a258-227b-4e31-a9cf-717495945fc2, which is a well known AAD App used by the Microsoft Azure PowerShell module.

**Resource** and **Scopes** is used to specify the Resource/Scopes you are authenticating to. The client\_credentials flow uses Scopes, the other flows are using Resource. Both are defaulted to [https://api.businesscentral.dynamics.com/](https://api.businesscentral.dynamics.com/) (Scopes have an added .default)

**TenantID** is set to Common by default, meaning that you will be authenticating your user towards all AAD tenants and can use the obtained access token to access any Business Central tenant your user has access to. You can specify the AAD tenant ID by providing the Guid or the tenant domain (69cb4a05-4ea8-412d-9f34-10fb5cf7db05 or demo.onmicrosoft.com in the examples above).

**Authority** is the authority used to authenticate the request. Default value is [https://login.microsoftonline.com/$TenantID](https://login.microsoftonline.com/$TenantID) (where $TenantID is the value of the TenantId parameter)

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
