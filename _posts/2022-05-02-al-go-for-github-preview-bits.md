---
layout: post
title: "Preview of future AL-Go for GitHub functionality"
date: 2022-05-02 16:58:24
categories: ["AL Development", "CI/CD", "AL-Go for GitHub"]
tags: ["AL-Go for GitHub", "DevOps", "GitHub"]
permalink: /2022/05/02/al-go-for-github-preview-bits/
---

Like everything else these days, AL-Go for GitHub now also is available in a preview version, which you can install/apply and remove as you like. This allows you to get advantage of a bugfix or new functionality faster.

_**NOTE** that when using the AL-Go preview version, you will have to update the AL-Go system files when you are told to, changes to the AL-Go actions might cause your version of the workflows to fail if not updated._

## The template repository

When I create a new repository based on [https://github.com/microsoft/AL-Go-PTE](https://github.com/microsoft/AL-Go-PTE), I will get a repository with some workflows and some settings files.

The repo settings file (for repo wide settings) is .github/AL-Go-Settings.json contains just a few settings, one of them being the url@branch from which this repo was created.

![](/assets/images/2022/al-go-for-github-preview-bits/image.png)

The CI/CD workflow uses this information to check whether there are any updates to the template. If updates are available, it is shown as a warning during the workflow:

![](/assets/images/2022/al-go-for-github-preview-bits/image-1.png)

## Update AL-Go System Files

In order to run the Update AL-Go System Files, you will need to create a secret containing a Personal Access Token with workflow permissions (see [here](https://github.com/microsoft/AL-Go/blob/main/Scenarios/UpdateAlGoSystemFiles.md))

After this, you can always apply the latest updates to the template, which is pointed out by the URL and the branch in the repo settings file, just by running the **Update AL-Go System Files** workflow.

If you want to apply the preview version of AL-Go for GitHub, you can just enter the new URL and branch in the workflow parameters and press **Run Workflow**.

![](/assets/images/2022/al-go-for-github-preview-bits/image-2.png)

After the workflow finishes, you will find that the PR contains changes to all scripts and workflows + a change to the repo settings file to now use the preview branch as the active template.

![](/assets/images/2022/al-go-for-github-preview-bits/image-3.png)

Switching back to the main branch (or any earlier release) can be done simply by specifying the URL and branch you want to use.

Note that the preview branch also has undergone full end-2-end testing before the code makes it to preview.

You can see what’s changed and what’s new under the preview section [here](https://github.com/microsoft/AL-Go/releases).

## Using BcContainerHelper preview

The **AL-Go preview branch** automatically selects the **BcContainerHelper preview** build as well, you do NOT have to modify the settings file for this.

If you want to use the BcContainerHelper preview build from your repository, which is NOT running on AL-Go preview, you can modify a setting called BcContainerHelperVersion in the repo settings file (same as above), but very frequently the AL-G

Edit the file and specify

![](/assets/images/2022/al-go-for-github-preview-bits/image-4.png)

for using the **latest BcContainerHelper preview build**.

## Your own AL-Go for GitHub

You can also create your own local version of AL-Go for GitHub and it is fairly easy and actually the way we in Microsoft are developing for AL-Go for GitHub.

When I follow [this guideline](https://github.com/microsoft/AL-Go/blob/main/Scenarios/Contributing.md), I end up having 3 repositories:

-   [https://github.com/freddydk/AL-Go-PTE](https://github.com/freddydk/AL-Go-PTE)
-   [https://github.com/freddydk/AL-Go-AppSource](https://github.com/freddydk/AL-Go-AppSource)

and the actions in [https://github.com/freddydk/AL-Go-Actions](https://github.com/freddydk/AL-Go-Actions)

Now, **if you are really brave and want to put your DevOps on totally untested code**, go ahead and change your AL-Go-Template to one of these and you will be using my development branches as your production branches. That was of course a joke:-)

**BTW**. You can also specify the value **dev** in **BcContainerHelperVersion**, then you will be using my development branch of ContainerHelper in your pipeline – also not recommended for production usage.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
