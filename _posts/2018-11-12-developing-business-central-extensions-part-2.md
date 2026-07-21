---
layout: post
title: "Developing Business Central Extensions (part 2) – Repository/Environment"
date: 2018-11-12 15:44:35
categories: ["AL Development", "CI/CD"]
tags: ["AL", "Business Central Sandbox", "CD", "CI", "Continuous Delivery", "Continuous Integration", "Extensions"]
permalink: /2018/11/12/developing-business-central-extensions-part-2/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

If you haven’t read [part 1](/2018/11/12/developing-business-central-extensions-part-1/), you should do so before continuing here. [Part 1](/2018/11/12/developing-business-central-extensions-part-1/) contains all the prerequisites you need in order to continue with this post.

# Create your organization and your first project

Navigate to [https://devops.azure.com](https://devops.azure.com) and login to your DevOps account. Create your organization, which is the location in which you will create your projects. In your organization, create your first project:

[![](/assets/images/2018/developing-business-central-extensions-part-2/c3517-firstproject-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/c3517-firstproject.png)

In the project navigate to the **Repos** -> **Files** area, click Import and enter [https://dev.azure.com/businesscentralapps/HelloWorld/\_git/HelloWorld](https://dev.azure.com/businesscentralapps/HelloWorld/_git/HelloWorld) in the Clone URL field.

[![](/assets/images/2018/developing-business-central-extensions-part-2/911a3-import-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/911a3-import.png)

# Inspect the content of the repository

[![](/assets/images/2018/developing-business-central-extensions-part-2/2b681-myfirstapp-repo-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/2b681-myfirstapp-repo.png)

-   **app** is the Hello World App folder
-   **test** is the Test App folder
-   **scripts** contains a set of scripts for build, test and CI/CD
-   **settings.json** defines the settings for your project and users
-   **Local-Sandbox.ps1** is a script, which will create a local Sandbox environment (based on Docker)
-   **AzureVM-Sandbox.ps1** is a script, which will create an AzureVM with a Sandbox environment (based on Docker and ARM templates)
-   **Local-Build.ps1** is a script, which will perform a full build and test run locally
-   **Initialize.ps1** is an initialization script used from the other scripts in this folder.
-   **CI.yml** is the CI pipeline sample

**Note:** The template will constantly be improved and the content of the template repository might vary.

If you want to add multiple apps to the project, the idea is to create folders for each app in the root folder.

After the import, Click the **Clone** button in the upper right Corner and select **Clone in VS Code**.

Accept the app change, select a location for the repository and sign in to your Azure DevOps account if asked to do so.

**Note:** If authentication fails with a NullReferenceException, it is likely because you need to install a new version of the Credential Manager as explained in Part 1.

# Modify Settings.json

Open Repository and open **settings.json**:

[![](/assets/images/2018/developing-business-central-extensions-part-2/c8bab-settings.json_-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/c8bab-settings.json_.png)

-   **name** is the name of the project.
-   **startupObjectId**, **startupObjectType** and **breakOnError** are values, which will be inserted in launch.json if present.
-   **navContainerHelperPath** is a local path to NavContainerHelper. If not set, NavContainerHelper will be installed from the PowerShell Gallery.
-   **versions** describes the three different versions of Business Central you will be building against.
-   **profiles** describes the Azure connection and resource naming, location, sizing etc. for a specific profile. Every user will typically have their own profile and the build process will also have a profile of its own.

Please modify the default profile to match the **subscription Id** and the **Key Vault Name** from part 1 of this blog post series. If you are going to create Azure VMs for your Business Central Sandbox environment, you also need to set the **Resource Group Name**, the **Location** and the **Properties** you want to set in the ARM template deployment. The properties section can contain all properties known by the ARM template (described by the template parameter).

# Modify App app.json

Expand the app folder, open app.json, mark , press Ctrl+Shift+P and select **Insert GUID** (from the Insert GUID extension we installed in VS Code in Part 1).

[![](/assets/images/2018/developing-business-central-extensions-part-2/4e1c7-app.json_-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/4e1c7-app.json_.png)

Copy the GUID to the clipboard, you need it for the dependency in the Test App. Close app.json.

# Modify Test app.json

Expand the test folder, open app.json, replace under dependencies with the App Id GUID from the App app.json.

Replace the with a new GUID. Close app.json.

# Check in your changes

In this blog post we are primarily working on local builds and local development. For that it is not necessary to checkin your changes.

But, in the next blog post we will be looking at setting up hosted builds and our configuration needs to be in the depot for that.

Click the Source Control icon and press **+** on the changes line to stage all your changes.

[![](/assets/images/2018/developing-business-central-extensions-part-2/582f4-changes1-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/582f4-changes1.png)

Now enter your commit message and press **Ctrl+Enter**

[![](/assets/images/2018/developing-business-central-extensions-part-2/fdf91-staged-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/fdf91-staged.png)

In the bottom left side, press the **synchronize symbol** to **push** your changes to the depot.

[![](/assets/images/2018/developing-business-central-extensions-part-2/0d847-push-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/0d847-push.png)

# Create your Sandbox environment

In Part 1, we installed the PowerShell extension in VS Code.

In the repository you will find two scripts for setting up sandbox environments:

-   **Local-Sandbox.ps1** is a script, which will create a local Sandbox environment (based on Docker)
-   **AzureVM-Sandbox.ps1** is a script, which will create an AzureVM with a Sandbox environment (based on Docker and ARM templates)

## Create a local Sandbox environment

Open **Local-Sandbox.ps1** and press **F5**.

Select the current version, login to your azure subscription if asked to do so and wait for the script to complete.

[![](/assets/images/2018/developing-business-central-extensions-part-2/7beac-local-sandbox-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/7beac-local-sandbox.png)

After completion of the script, you should see

[![](/assets/images/2018/developing-business-central-extensions-part-2/2be8b-completed-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/2be8b-completed.png)

**Note:** That all app folders in the root folder automatically gets a **Local Sandbox** server created pointing to the local sandbox environment.

## Create an AzureVM Sandbox environment

**Note**: In order to create an AzureVM Sandbox environment, you need to correctly configure settings.json as described above (**Resource Group Name**, **VM Name** and **Contact email for Lets Encrypt**)

**Note also**: The Resource Group holding the Azure VM should not be used for other resources. Creating an AzureVM-Sandbox will **remove the Resource Group if it already exists**.

Open **AzureVM-Sandbox.ps1** and press **F5**.

Note

Select the current version, login to your azure subscription if asked to do so and wait for the script to complete. This will likely take 10-15 minutes and after that you should see:

[![](/assets/images/2018/developing-business-central-extensions-part-2/2c717-deploy-vm-done-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/2c717-deploy-vm-done.png)

**Note:** The deployment is still running inside the Azure VM. The landing page should open automatically and you should see:

[![](/assets/images/2018/developing-business-central-extensions-part-2/97b7e-landing-page-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/97b7e-landing-page.png)

This content of this page will change to the landing page when the installation is complete:

[![](/assets/images/2018/developing-business-central-extensions-part-2/23c07-landing-page-final-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/23c07-landing-page-final.png)

**Note:** That all app folders in the root folder automatically gets a **AzureVM Sandbox** server created pointing to the AzureVM sandbox environment.

# Build your app

In VS Code, close the folder.

Select **File** -> **Add folder to workspace** select the app folder in the repository and add this to the workspace.

Do the same with the **test** folder and save the Workspace in the Root folder.

Open your **app** project in the workspace (not under myfirstapp), click **HelloWorld.al**

[![](/assets/images/2018/developing-business-central-extensions-part-2/88311-app-project-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/88311-app-project.png)

Click the Download Symbols, select appropriate server if you have created both environments (**Local Sandbox** if running Local, **AzureVM Sandbox** if running Azure VM). Enter the credentials you stored in the keyvault and you should see:

[![](/assets/images/2018/developing-business-central-extensions-part-2/f88b0-symbols-downloaded-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/f88b0-symbols-downloaded.png)

Press **F5** and you should see:

[![](/assets/images/2018/developing-business-central-extensions-part-2/65b2e-hello-world-1.png)](/assets/images/2018/developing-business-central-extensions-part-2/65b2e-hello-world.png)

# What’s next

Now we are done setting up our repository, we have build the app and successfully tested it.

In the next Blog Post we will setup build agents and create build pipelines to build the app in Azure DevOps.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
