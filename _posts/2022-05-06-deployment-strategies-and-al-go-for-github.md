---
layout: post
title: "Deployment strategies and AL-Go for GitHub"
date: 2022-05-06 12:08:07
categories: ["AL Development", "CI/CD"]
tags: ["AL-Go for GitHub", "Continuous Deployment", "FAT", "PROD", "Publish", "Publish-BcContainerApp", "Publish-PerTenantExtensionApps", "QA"]
permalink: /2022/05/06/deployment-strategies-and-al-go-for-github/
---

When you are done developing your app, it needs to be deployed to your customers if it is a PTE or to AppSource if it is an AppSource app.

Currently we don’t have any automated way of publishing your app to AppSource – that is something we are working on and some day in the future, when it becomes possible, you will get a workflow in AL-Go for GitHub which handles this automatically.

This blog post will talk about PTEs, and what features are available in AL-Go for GitHub to assist you deploying your PTEs to customers.

## Environments

In this blog post I will distinguish between 4 different types of environments:

The **DEV environment** (Development) which currently can be a local docker based environment or an online sandbox environment, but in the future, you will also be able to use hosted docker based environment or Azure Container Instances or Azure VMs. Stay tuned for this functionality coming to a repository near you.

The **QA environment** (Quality Assurance) is an online sandbox environment, which always holds the latest successful build of your app. This environment contains standard demo data from Business Central and your app and is the environment for which continuous deployment is setup.

The **FAT environment** (Final Acceptance Test) is an online sandbox environment, which is a copy of the customers production environment and is used to allow the customer to test the solution before releasing to production.

The **PROD environment** (Production) is your customers live production environment. Only released bits makes it to the production environment, which is the life nerve of your customer.

You might have different terminologies, or you might have different needs, but the essence of these 3 environments is, that the QA environment is subject to continuous deployment for every successful build from main. The FAT environment is where you publish your release candidate to allow your customer hands on and the PROD environment is the holy grail.

The actual creation of the environments is not part of AL-Go for GitHub – the environments are created in the Dynamics 365 Business Central admin center.

In this blog post, I will explain how to setup deployment for these 3 types of environments for one project to one or two different customers, both using the Environments feature in GitHub, which comes with paid licenses or the settings file option from AL-Go for GitHub.

In my case, my GitHub organization (**BusinessCentralApps**) is a paid Team license:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-19.png)

## Setting up environments using GitHub Environments

I created these three environments in my **Admin Center**:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-18.png)

In my repository (**[Contoso-App3](https://github.com/BusinessCentralApps/Contoso-App3)**), under [**Settings** -> **Environments**](https://github.com/BusinessCentralApps/Contoso-App3/settings/environments), I have added 3 environments, with no additional settings.

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-21.png)

**Note** that I have added an annotation to the final acceptance test environment and the production environment. All environments with annotations like **(FAT)**, **(PROD)**, **(Final Acceptance Test)** or **(Production)** will be ignored by the continuous deployment step in the CI/CD workflow. Environments created as production environments in Admin Center cannot be used for continuous deployment (even if the environment doesn’t have any annotation).

If the name of the environment in the admin center is different from name in environments, you can create an environment secret called **EnvironmentName** under the environment, containing the actual environment name:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-22.png)

Note, that it you enter the value in Upper Case, the actual environment name will be hidden in the output of the workflow.

You can also create a repository secret called **MyFAT\_EnvironmentName** if you prefer to have your secrets in the repository instead of under the Environment.

You also need to setup authentication to the environments. For this, you can setup S2S like described here, or you can use delegation and create your AUTHCONTEXT secret value using this PowerShell script (on a machine with the **BcContainerHelper 3.0.8 or higher**):

```
$AuthContext = New-BcAuthContext -includeDeviceLoginGet-ALGoAuthContext -bcAuthContext $AuthContext | Set-Clipboard
```

In the repository, under Settings -> Secrets -> Actions click New Repository Secret and create a secret called AUTHCONTEXT with this value.

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-23.png)

After this, I can run the **CI/CD** workflow and see that my **QA** environment (and only my QA environment) receives my App3.

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-24.png)

Every time you add features and functionality to our app and complete a pull request to the main branch, the CI/CD branch will run and deploy to the QA environment.

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-25.png)

After the development is done, you run the **Publish To Environment** workflow, select which build to publish (**latest** is the latest build) and enter the name of your **Final Acceptance Test** name (**MyFAT** in our case):

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-27.png)

After the workflow completes

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-28.png)

We can investigate the deploy step and see the actual name of our environment

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-29.png)

Now, the customer will perform final acceptance test of the solution and if he discovers any bugs, we will fix the bugs and redeploy to the FAT environment.

If you look in the output, you will notice that when publishing to environments of type **Sandbox**, AL-Go for Github will use the **development endpoint**.

When the solution has been accepted by the customer, we create a release of the app:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-30.png)

when done, we can publish to PROD

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-31.png)

and looking at the workflow output, you will see that the **automation API** is used for the deployment:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-32.png)

at the same time, the **Create Release** workflow also updated the version number in the repository, and you can now work on the next version of your app.

## Environment protection rules

When using the environments feature in GitHub, you can setup environment protection to ensure that the customer (or customer responsible) approves the deployment before it actually happens.

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-33.png)

## Individual AUTHCONTEXT

If the **PROD** environment uses a **different authorization context** than the **QA** and **FAT** environments, you can create an environment secret (a secret defined under the Environment in the repository) called **AUTHCONTEXT** and this will override the global **AUTHCONTEXT** secret when deploying to **PROD**. You can also create a repository secret called **PROD\_AUTHCONTEXT**, which also will take precedence over the global AUTHCONTEXT for the PROD environment.

## Setup environments when using a free GitHub tier

Even though the environments feature is not available when using a free GitHub subscription, AL-Go for GitHub still allows you to publish to environments.

Obviously, you cannot setup **environment protection rules** and you cannot have **environment secrets**, but you can create your environments in the repository settings file and you can use repository secrets like **<environmentName>\_<secretName>** to add “environment secrets”. These secrets will obviously be available throughout the workflows, but AL-Go for GitHub will use them during deployment.

In a different Business Central tenant (another customer who also want our app), I have added 3 new environments (DE):

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-34.png)

When you have multiple customers for one repository, you will likely name the environments in AL-Go for GitHub after the name of the customer or like.

I have added 3 additional environments from a different customer to our repository using the repository settings file (.github\\AL-Go-Settings.json):

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-35.png)

and added repository secrets for the actual name and the authorization context for these environments:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-36.png)

and now, when I run the CI/CD workflow, I can see that it deploys to two QA environments:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-37.png)

and when done, both environments will now contain this new build

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-40.png)

When publishing to environments, I can use a naming pattern like **\*(FAT)**:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-38.png)

to deploy to **all final acceptance test environments**:

![](/assets/images/2022/deployment-strategies-and-al-go-for-github/image-39.png)

and **\*(Production)** or any other naming patterns based on the environment name in GitHub or settings. to deploy to all production environments.

## The very very short version

The very very short story of this long blog post is, that you can define **any number of environments** in **GitHub environments** or **AL-Go-Settings** and the ones **without annotations** (and are **not production environments**) will be **deployed during CI/CD**. The remaining environments can be deployed to using the **Publish To Environment** workflow, by specifying the **environment name** or pattern.

The **authorization context** and the **actual environment name** (if different from the GitHub name) must be added as **secrets** (**AUTHCONTEXT** and **EnvironmentName**) **under the environment** or following the naming pattern **<environmentname>\_<SecretName>**.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
