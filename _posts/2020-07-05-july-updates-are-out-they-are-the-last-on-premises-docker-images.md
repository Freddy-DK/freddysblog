---
layout: post
title: "July updates are out – they are the last on-premises docker images"
date: 2020-07-05 11:26:37
categories: ["Docker", "NavContainerHelper"]
tags: ["Artifacts", "Docker", "Image", "NavContainerHelper"]
permalink: /2020/07/05/july-updates-are-out-they-are-the-last-on-premises-docker-images/
---

**Update 2021/2/10:** BcContainerHelper has replaced NavContainerHelper. This blog post still reference NavContainerHelper, which is outdated.

Business Central and NAV July 2020 on-premises updates was published over the weekend and this is going to be the last on-premises version of Business Central and NAV released as a docker image. The existing docker images will stay on mcr.microsoft.com for some time still, but no new on premises images will be added.

If this is news to you, you should read [https://freddysblog.com/2020/06/25/changing-the-way-you-run-business-central-in-docker/](/2020/06/25/changing-the-way-you-run-business-central-in-docker/) and [https://freddysblog.com/2020/06/25/working-with-artifacts/](/2020/06/25/working-with-artifacts/) (+ some more blog posts on how to find out what the changes are)

# July on-premises updates images and artifacts

Below you will see a table of the july update artifacts (how to get the url) and what the corresponding image names are.

<table><tbody><tr><td></td><td><strong>NAV 2016 CU57</strong></td></tr><tr><td>artifacts</td><td>Get-NavArtifactUrl -nav 2016 -cu cu57 -country w1<br><a href="https://bcartifacts.azureedge.net/onprem/9.0.51900.0/w1" rel="nofollow">https://bcartifacts.azureedge.net/onprem/9.0.51900.0/w1</a></td></tr><tr><td>image</td><td>mcr.microsoft.com/dynamicsnav:2016-cu57-w1</td></tr><tr><td></td><td><strong>NAV 2017 CU44</strong></td></tr><tr><td>artifacts</td><td>Get-NavArtifactUrl -nav 2017 -cu cu44 -country w1<br><a href="https://bcartifacts.azureedge.net/onprem/10.0.30301.0/w1" rel="nofollow">https://bcartifacts.azureedge.net/onprem/10.0.30301.0/w1</a></td></tr><tr><td>image</td><td>mcr.microsoft.com/dynamicsnav:2017-cu44-w1</td></tr><tr><td></td><td><strong>NAV 2018 CU31</strong></td></tr><tr><td>artifacts</td><td>Get-NavArtifactUrl -nav 2018 -cu cu31 -country w1<br><a href="https://bcartifacts.azureedge.net/onprem/11.0.43796.0/w1" rel="nofollow">https://bcartifacts.azureedge.net/onprem/11.0.43796.0/w1</a></td></tr><tr><td>image</td><td>mcr.microsoft.com/dynamicsnav:2018-cu31-w1</td></tr><tr><td></td><td><strong>Business Central 14.15</strong></td></tr><tr><td>artifacts</td><td>Get-BCArtifactUrl -type OnPrem -version 14.15 -country w1<br><a href="https://bcartifacts.azureedge.net/onprem/14.15.43800.0/w1" rel="nofollow">https://bcartifacts.azureedge.net/onprem/14.15.43800.0/w1</a></td></tr><tr><td>image</td><td>mcr.microsoft.com/businesscentral/onprem:1904-cu14-w1</td></tr><tr><td></td><td><strong>Business Central 15.8</strong></td></tr><tr><td>artifacts</td><td>Get-BCArtifactUrl -type OnPrem -version 15.8 -country w1<br><a href="https://bcartifacts.azureedge.net/onprem/15.8.43801.0/w1" rel="nofollow">https://bcartifacts.azureedge.net/onprem/15.8.43801.0/w1</a></td></tr><tr><td>image</td><td>mcr.microsoft.com/businesscentral/onprem:1910-cu8-w1</td></tr><tr><td></td><td><strong>Business Central 16.3</strong></td></tr><tr><td>artifacts</td><td>Get-BCArtifactUrl -type OnPrem -version 16.3 -country w1<br><a href="https://bcartifacts.azureedge.net/onprem/16.3.14085.14238/w1" rel="nofollow">https://bcartifacts.azureedge.net/onprem/16.3.14085.14238/w1</a></td></tr><tr><td>image</td><td>mcr.microsoft.com/businesscentral/onprem:2004-cu3-w1</td></tr></tbody></table>

# August 2020 on-premises updates will be artifacts only

As already mentioned, August updates will be artifacts only, and looking at the table above, you can easily imagine how you will be able to retrieve the url for the artifacts when it ships.

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
