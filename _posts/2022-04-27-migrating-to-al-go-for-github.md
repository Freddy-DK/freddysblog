---
layout: post
title: "Migrating to AL-Go for GitHub"
date: 2022-04-27 05:57:29
categories: ["AL Development", "CI/CD"]
tags: ["AL-Go for GitHub", "CI/CD", "Migrating"]
permalink: /2022/04/27/migrating-to-al-go-for-github/
---

As explained in the first blog post about [AL-Go for GitHub](/2022/04/26/al-go-for-github/) the next post would be all about how to migrate your repository to AL-Go for GitHub.

Whether you have a setup based on the first CI/CD Hands-On-Lab or you have the latest generation, it should be fairly easy to migrate to AL-Go and get all the benefits with that, but it is a manual process.

The following scenarios are described in this post:

1.  From a CI/CD HOL based repo on GitHub
2.  From a CI/CD HOL based repo on Azure DevOps
3.  From GitHub (if you are “just” using it as a source code repository)
4.  From Azure DevOps (if you are “just” using it as a source code repository)
5.  From nothing (if you just have the source code on your laptop)
6.  From nothing (if you just have some .app files, but not the source code)

Last, but not least there are some common questions you need to consider when using any DevOps setup really.

## From a CI/CD HOL based repo on GitHub

In this example, I have a repository called freddydk/MyOldRepo, which was created using the old Hello World template ([https://github.com/businesscentralapps/Old.HelloWorld](https://github.com/businesscentralapps/Old.HelloWorld)), used in the latest version of the CI/CD Hands-On-Lab. In order to keep my development history from the current repository, I will just make the change as a PR and apply that.

In my case, this is a PTE, but the process is the same for AppSource apps. My repository looks like this on GitHub:

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-12.46.27.png)

and like this in VS Code:

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-12.39.07.png)

Make sure that you do not have any pending changes to checkin. Create a new branch for applying AL-Go and push the branch to the repo.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-12.44.13.png)

Delete the .AzureDevOps folder and the .github folder. Download the [PTE template](https://github.com/microsoft/AL-Go-PTE/archive/refs/heads/main.zip) to your computer and unpack the .zip file. Use the [AppSource template](https://github.com/microsoft/AL-Go-AppSource/archive/refs/heads/main.zip) if your app is an AppSource App. Copy the two folders: .AL-Go and .github

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-12.54.34.png)

and paste them into the root folder of your repository

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-12.57.05.png)

Now you need to transfer values from your old settings file (scripts\\settings.json) to the AL-Go settings file (.AL-Go\\settings.json). A few things to note:

appFolders, testFolders and additionalCountries are now a .json array and not a comma separated string.

previousApps are automatically taken from the latest release from the repository, you would need to create an initial release as the first thing after moving to AL-Go and use this as a baseline

installTestRunner, installTestFramework, installTestLibraries and installPerformanceToolkit can be specified, but they are auto calculated based on dependencies.

installApps and installTestApps are also .json arrays instead of comma separated values – AL-Go supports a different mechanism when specifying dependencies which you can add later.

change the country setting in the AL-Go settings file to be your primary development country and then remove the scripts folder.

Lastly – modify the .code-workspace file to reflect the new folders.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-14.17.15.png)

**Commit**, **Push**, create **Pull Request**, **Confirm Merge** and watch the **CI/CD** pipeline start…

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-14.21.24.png)

After a while, the PR CI/CD should complete.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-14.33.53.png)

With this, there are some common topics you need to consider. Please go to the last section for this.

## From a CI/CD HOL based repo on Azure DevOps

In this example, I am using the Old HelloWorld sample from Azure DevOps ([https://dev.azure.com/businesscentralapps/Old.HelloWorld/\_git/Old.HelloWorld](https://dev.azure.com/businesscentralapps/Old.HelloWorld/_git/Old.HelloWorld)). Instead of cloning the repo, I download the content of the repo as a .zip file and extract the content.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-15.37.56.png)

giving me a file structure like this

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-15.39.08.png)

In your browser, navigate to [https://aka.ms/al-go-pte](https://aka.ms/al-go-pte) (for PTEs) or [https://aka.ms/al-go-appsource](https://aka.ms/al-go-appsource) (for AppSource apps) and click the Use this template button.

Select an organization, a name and the visibility of the project.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-15.42.30.png)

Looking like this in VS Code on your local box.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-15.44.01.png)

Now Copy and paste all folders (except for .AzureDevOps, .github and scripts) from your old repository (the extracted .zip file).

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-15.48.01.png)

and paste them into the new repo

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-15.49.29.png)

Remove the al.code-workspace and modify your .code-workspace file to reflect the new folders.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-15.52.46.png)

Now you need to transfer values from your old settings file (.zip-file\\scripts\\settings.json) to the AL-Go settings file (.AL-Go\\settings.json). A few things to note:

appFolders, testFolders and additionalCountries are now a .json array and not a comma separated string.

previousApps are automatically taken from the latest release from the repository, you would need to create an initial release as the first thing after moving to AL-Go and use this as a baseline

installTestRunner, installTestFramework, installTestLibraries and installPerformanceToolkit can be specified, but they are auto calculated based on dependencies.

installApps and installTestApps are also .json arrays instead of comma separated values – AL-Go supports a different mechanism when specifying dependencies which you can add later.

change the country setting in the AL-Go settings file to be your primary development country and then remove the scripts folder.

Commit, push, create Pull Request, Confirm Merge and watch the CI/CD pipeline start and finish…

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.22.49.png)

After a while, the PR CI/CD should complete. With this, there are some common topics you need to consider. Please go to the last section for this.

## From GitHub (if you are “just” using it as a source code repository)

In this example, I have a repository called freddydk/MyGitHubApps, which contains 2 apps and a test app, and I want to add AL-Go.

In my case, this is a PTE, but the process is the same for AppSource apps. My repository looks like this on GitHub:

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-27-06.24.10.png)

and like this in VS Code:

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-27-06.24.37.png)

Download the [PTE template](https://github.com/microsoft/AL-Go-PTE/archive/refs/heads/main.zip) to your computer and unpack the .zip file. Use the [AppSource template](https://github.com/microsoft/AL-Go-AppSource/archive/refs/heads/main.zip) if your app is an AppSource App. Copy the two folders: .AL-Go and .github

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-12.54.34.png)

and paste them into your folder.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-27-06.29.22.png)

Modify the .gitignore file in VS Code and ensure that it contains these lines:

```
*.app
*.flf
TestResults.xml
BuildOutput.txt
rad.json
.output
.alcache/
.altemplates/
cache_*
```

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-27-06.32.24.png)

Now, **Stage**, **Commit** and **Push** your changes to the repo.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-27-06.33.57.png)

Go to **Actions**, select **CI/CD** and click **Run Workflow**

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-27-06.34.47.png)

The reason why the workflow didn’t start automatically is, that the workflow wasn’t there at the time of checkin. Every subsequent push to the repository will cause a CI/CD workflow to run.

After a while, you should see that the CI/CD pipeline completes and your artifacts are generated…

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-27-06.51.22.png)

## From Azure DevOps (if you are “just” using it as a source code repository)

In your browser, navigate to [https://aka.ms/al-go-pte](https://aka.ms/al-go-pte) (for PTEs) or [https://aka.ms/al-go-appsource](https://aka.ms/al-go-appsource) (for AppSource apps) and click the Use this template button.

Select an organization, a name and the visibility of the project.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.49.24.png)

Download the content of your repo and place the .zip file on Dropbox or somewhere else, where you can get a direct download url to the file (read how-to [here](/2017/02/26/create-a-secure-url-to-a-file/)).

Under **Actions**, select **Add existing app or test app**, press **Run workflow** and paste the direct download url to your .zip file (you can try [this url](https://dev.azure.com/businesscentralapps/Old.HelloWorld/_apis/git/repositories/Old.HelloWorld/items/items?path=/&resolveLfs=true&$format=zip&api-version=5.0&download=true) is you like). Select N to Direct Commit and click **Run Workflow**.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.50.11.png)

After a few minutes, the workflow is done and you can see the **Pull request** added under **Pull Requests**.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.54.29.png)

**Merge the Pull Request** and you will see that a **CI/CD** workflow is started automatically and will complete successfully. You should also see that the pipeline builds all apps and runs the tests in all test apps.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-21.27.38.png)

## From nothing (if you just have the source code on your laptop)

In your browser, navigate to [https://aka.ms/al-go-pte](https://aka.ms/al-go-pte) (for PTEs) or [https://aka.ms/al-go-appsource](https://aka.ms/al-go-appsource) (for AppSource apps) and click the Use this template button.

Select an organization, a name and the visibility of the project.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.58.29.png)

Locate your source files on your laptop

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-21.01.12.png)

and simply drag and drop the files into your GitHub project

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-21.02.41.png)

Commit the changes and you should see that the CI/CD workflow automatically starts and completes if all apps can build and tests can be run.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-21.03.15.png)

Navigating into the pipeline output, you will see familiar output of compiling, publishing and running tests

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-21.25.46.png)

## From nothing (if you just have some .app files, but not the source code)

In your browser, navigate to [https://aka.ms/al-go-pte](https://aka.ms/al-go-pte) (for PTEs) or [https://aka.ms/al-go-appsource](https://aka.ms/al-go-appsource) (for AppSource apps) and click the Use this template button.

Select an organization, a name and the visibility of the project.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.28.28.png)

Place your .app files in a .zip file and place it on Dropbox or somewhere else, where you can get a direct download url to the file (read how-to [here](/2017/02/26/create-a-secure-url-to-a-file/)).

Under **Actions**, select **Add existing app or test app**, press **Run workflow** and paste the direct download url to your .zip file (you can try with [this url](https://businesscentralapps.blob.core.windows.net/bingmaps-pte/BingMaps.PTE_3.0.27.0.zip) if you like). Select N to Direct Commit and click **Run Workflow**.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.45.28.png)

After a few minutes, the workflow completed, and you can see the Pull request added under **Pull Requests**.

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.38.42.png)

**Merge the Pull Request** and you will see that a **CI/CD** workflow is started automatically, and when completed, it makes all build artifacts available on the build

![](/assets/images/2022/migrating-to-al-go-for-github/screenshot-2022-04-26-20.55.57.png)

## Common questions you need to consider

As you might guess, migrating to AL-Go is not just to move your code and make it build. You also need to ask yourself some questions on how to do things. First and foremost – the build part of the version number of your app is autoincremented, starting at build #1 – you might have a higher version our there already. Below are some questions, which each will have their own blog post.

### The structure of your repositories

What apps should be in the same repository and which apps should be in individual repositories – how do you want to structure your repositories.

_Stay tuned for a blog post on structuring your GitHub repositories._

### branch name

Your main branch must be named main. The CI/CD pipeline automatically triggers on the main branch and on Pull Requests. If your main branch is called master, you need to rename it (Settings -> branches -> rename default branch). Beside the main branch, you need to decide on how you develop, do you allow people to just check-in code directly or do you do feature branches, code reviews and pull requests

_Stay tuned for a blog post on branching strategies_

### Versioning of your app

AL-Go is automatically applying build and release number to your apps. The build number is the RUN\_NUMBER assigned from GitHub which with the move to AL-Go for GitHub gets reset. This might mean that your app will get version numbers which you already shipped. You can add a setting called **RunNumberOffset** and set that to the previously build app build number +1 – or you can increase the minor or major part of your app version.

_Stay tuned for a blog post on versioning and release strategy_

### Secrets

In every DevOps setup, there will be a number of secrets. That could be the license file, authentication token to Business Central Admin Center, the Insider Sas Token or other things. Where do you store those securely. AL-Go for GitHub supports GitHub secrets or Azure KeyVaults, but what is the right choice for you?

_Stay tuned for a blog post on secrets_

### Deployments

How are you going to deploy your app? Are you going to make continuous deployment of your app to QA environments and how do you go around doing that?

_Stay tuned for a blog post on deployment_

With that – I have plenty of things to write about over the upcoming days…

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
