---
layout: post
title: "AL-Go for GitHub"
date: 2022-04-26 10:12:48
categories: ["CI/CD"]
tags: ["AL-Go for GitHub", "DevOps", "GitHub"]
permalink: /2022/04/26/al-go-for-github/
---

It has been a while since my last blog post and the reason behind this is quite simple: I have been busy. Busy creating [AL-Go for GitHub](https://github.com/microsoft/AL-Go).

[AL-Go for GitHub](https://github.com/microsoft/AL-Go) is plug-and-play DevOps for Business Central PTEs or AppSource apps on GitHub. A tool, which does NOT require you to modify PowerShell scripts, change .yaml workflows or pipelines, but still allows you to setup and maintain full DevOps for your Business Central projects with a click of a button.

[AL-Go for GitHub](https://github.com/microsoft/AL-Go) will only be supported on GitHub and we have no plans of migrating that to Azure DevOps. We also will not ship any new versions of the Hands-On-Lab with focus on Azure DevOps.

BcContainerHelper, docker and a number of the tools that is used in [AL-Go for GitHub](https://github.com/microsoft/AL-Go) can still be used on Azure DevOps, but our focus will be on GitHub.

## Why GitHub?

Since the acquisition of GitHub in 2018, Microsoft has been investing heavily in the functionality of GitHub, constantly adding features and functionalities to actions and workflows. In many ways, GitHub is to Azure DevOps what VS Code is to Visual Studio, the leaner and more agile brother.

It was also important for us that both [ALOps](https://github.com/HodorNV/ALOps) and [Cosmo](https://marketplace.cosmoconsult.com/product/?id=345E2CCC-C480-4DB3-9309-3FCD4065CED4) have offerings on Azure DevOps, giving a real alternative to people who prefer Azure DevOps, allowing us to provide a free and open-source tool on GitHub.

Functionality wise, there are still places, where Azure DevOps have more functionalities built in than GitHub. A graphical view of build and release pipelines are some of the areas, where [AL-Go for GitHub](https://github.com/microsoft/AL-Go) is lacking behind similar offerings on Azure DevOps. Rest assured that GitHub will catch up.

## The Business Central Launch Event presentation

[AL-Go for GitHub](https://github.com/microsoft/AL-Go) has been in preview for some months and officially released on April 1st, 2022. During Directions North America I had a session about AL-Go, which was very similar to the session, which was recorded and is available on the [Business Central Launch Event](https://aka.ms/bcle). It is free to register and watch the video, which is called: **Introducing: Do-it-yourself CI/CD made easy with GitHub**.

## Maintaining your repository

As you might remember, I wrote a blog post 1½ year ago on [how to setup CI/CD on GitHub](https://freddysblog.com/2020/11/14/using-github-for-devops/) using the HelloWorld template. This was also just 5 minutes to WOW – but then you had a copy of a repository, including a number of scripts and it was a steep learning curve to maintain, update and fix issues in your workflows.

The single biggest advantage of [AL-Go for GitHub](https://github.com/microsoft/AL-Go) is, that upgrading your repository to the latest version of [AL-Go for GitHub](https://github.com/microsoft/AL-Go) is done by running a workflow. Until you run this workflow, your repository will keep running on the version you currently have, using the version of the actions that your repo has been using all the time.

Regular updates and innovations added to [AL-Go for GitHub](https://github.com/microsoft/AL-Go), will not break your pipeline and can be applied and removed as any other Pull Request, allowing you to have a stable DevOps setup at all times. The explanation of how this is possible is at the end of the Launch Event presentation.

## Documentation and Usage scenarios

You will find the official documentation on [docs](https://docs.microsoft.com/en-us/dynamics365/business-central/dev-itpro/al-go/algo-overview) and you will find a Hands-On-Lab like walkthrough of scenarios on [GitHub](https://github.com/microsoft/AL-Go#readme).

## Support and questions

Support is provided on [GitHub](https://github.com/microsoft/AL-Go/issues). You can also ask questions in our [Yammer group for partners](https://www.yammer.com/dynamicsnavdev/#/threads/inGroup?type=in_group&feedId=106205503488&view=all).

## One-Size

[AL-Go for GitHub](https://github.com/microsoft/AL-Go) is one-size, but it is not necessarily one-size fits all. What you gain in improved user experience – you might lack in flexibility. Not that [AL-Go for GitHub](https://github.com/microsoft/AL-Go) isn’t flexible (everyone who has worked with any of my tools know that there are a ton of hooks), but if you need to bend over backwards in order to make AL-Go for GitHub meet your needs, then AL-Go might not be the right tool for you.

Instead of forcing AL-Go to do exactly what you want, you might want to read some of the upcoming blog posts explaining why AL-Go was created like it is.

## Next steps

My next blog post will be on how to migrate from a DevOps repository created on GitHub or Azure DevOps using the old Hands-On-Lab. What do you need to do to get up running?

In the near future, I will also ship an [AL-Go for GitHub](https://github.com/microsoft/AL-Go) wizard – kind of like New-BcContainerWizard, where you answer a number of questions in a wizard-like environment and you will get a repo setup for you.

I will also write up some blog posts about branching and versioning of apps to explain what lies behind the thinking that went into [AL-Go for GitHub](https://github.com/microsoft/AL-Go).

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
