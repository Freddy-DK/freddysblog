---
layout: post
title: "Working with AL-Go for GitHub preview"
date: 2022-06-27 13:34:18
categories: ["CI/CD", "AL-Go for GitHub"]
tags: ["AL-Go for GitHub", "preview"]
permalink: /2022/06/27/working-with-al-go-for-github-preview/
---

If you want to have the latest updates of AL-Go for GitHub, you can update your repository and use [https://github.com/microsoft/al-go-pte@preview](https://github.com/microsoft/al-go-pte@preview) or [https://github.com/microsoft/al-go-appsource@preview](https://github.com/microsoft/al-go-appsource@preview) as your template repository.

This means that you will get new updates before they are released. It also means that the actions you are using are instantly updated when we deploy a new version for preview and thus might be out of sync with your workflows.

One example of this is the preview, which was released yesterday (June 26th, 2022) to fix issue #168 will cause CI/CD workflows to stop publishing artifacts until you have run the Update AL-Go System Files.

Technically, the reason for this is that the RunPipeline action now published artifacts into a folder called .buildartifacts (instead of output) and the earlier CICD.yaml will look in the output folder:

![](/assets/images/2022/working-with-al-go-for-github-preview/image.png)

After updating the AL-Go System Files, everything will work normally again.

Note that this problem will NOT occur for people using the released bits. They can safely update to the latest version after it is released, and they will get workflows and actions updated together.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
