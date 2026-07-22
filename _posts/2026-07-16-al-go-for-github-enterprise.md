---
layout: post
title: "AL-Go for GitHub Enterprise"
date: 2026-07-16 06:58:50
categories: ["AL-Go for GitHub"]
tags: ["AL-Go for GitHub", "data-residency", "ghe-com", "GitHub", "github-enterprise"]
permalink: /2026/07/16/al-go-for-github-enterprise/
---

GitHub.com is the standard multi-tenant public platform where anyone can host personal or org repos, public or private. GHE.com refers specifically to GitHub Enterprise Cloud with data residency — a separate product tier where your enterprise gets its own dedicated subdomain (e.g., octocorp.ghe.com) instead of living on github.com/octocorp.

## The Challenge

At this time, AL-Go for GitHub doesn’t work on GHE.COM enterprises, but there might be a solution for this ahead. I was contacted by Fellowmind in Sweden, who wanted to have this functionality and I guess there is likely nobody better qualified (outside Microsoft) to build this, than me, so… I accepted the challenge and I am working with [Niklas Silvergrund](https://www.linkedin.com/in/niklas-silvergrund-243445) from Fellowmind to test the solution.

If you want to try this out yourself, there is a version of AL-Go in my Freddy-DK organization, which supports GHE.COM. Documentation on how to get started is here: [https://github.com/Freddy-DK/AL-Go/blob/main/Scenarios/UsingGitHubEnterprise.md](https://github.com/Freddy-DK/AL-Go/blob/main/Scenarios/UsingGitHubEnterprise.md)

I will keep this repo up-to-date with Microsofts AL-Go for GitHub and work with Microsoft to get support for GHE.COM into the main product, but I can obviously not promise any date.

Try it out, and report any problems by creating an issue in my fork, and I will make sure to fix them, before the functionality is merged.

## Main differences between github.com and GHE.COM

**Isolation & access**: On GHE.com, managed user accounts can only access resources within your enterprise’s dedicated instance – no interacting with the broader github.com ecosystem (no personal accounts, no public open-source repos mixed in). GitHub.com is fully open and interconnected.

**Data residency**: GHE.com lets you choose the geographic region where your code and data are stored, which is the main compliance driver for adopting it. GitHub.com doesn’t offer this control – your data lives wherever GitHub’s standard infrastructure puts it.

**API endpoints**: Integrations must hit your enterprise-specific URL (e.g., api.octocorp.ghe.com) rather than the standard api.github.com.

**Identity**: GHE.com uses managed user accounts tied to your enterprise’s identity provider, separating enterprise work cleanly from any open-source/personal GitHub activity.

**Purpose**: GitHub.com is the general-purpose platform for individuals and organizations of any size. GHE.com exists for enterprises with strict data residency or compliance requirements who still want a cloud-hosted (not self-managed) solution – as opposed to GitHub Enterprise Server, which is the fully self-hosted on-prem option.

## Changes needed in BcContainerHelper

BcContainerHelper is used by AL-Go for GitHub for handling GitHub packages (NuGet), and this functionality needed some changes due to the GitHub packages NuGet feed URL is different – **https://nuget.<enterprisename>.ghe.com/<organizationname>** vs. **https://nuget.github.com/<organizationname>** also the GitHub API URL is different, so this is deducted from the NuGet Server Url.

The PR can be found here: [https://github.com/microsoft/navcontainerhelper/pull/4167](https://github.com/microsoft/navcontainerhelper/pull/4167)

## Changes needed in AL-Go for GitHub

Most changes come from the fact that the URL is different, **https://<enterprisename>.ghe.com/<organizationname>** vs. **https://github.com/<organizationname>**. This also changes the API URL by adding api in front of the hostname.

Inside GitHub workflows, we can use environment variables like GITHUB\_SERVER\_URL and GITHUB\_API\_URL, which takes care of most of these things.

A bigger challenge is that the GITHUB\_TOKEN in workflows on GHE cannot be used for authenticating towards github.com – not even for public repositories and since AL-Go for GitHub still lives on github.com and every enterprise shouldn’t have to have their own version of AL-Go, this poses a few challenges when running Update AL-Go System Files.

Another problem is that you cannot use a template from github.com on ghe.com, so how do I create my repo with [https://aka.ms/algopte](https://aka.ms/algopte) – if I cannot select my enterprise. The solution here is to create an indirect AL-Go template in your enterprise and keep that up-to-date (I have created a small tool for that) and then your “normal” repositories will use a PTE or AppSource template from your own enterprise and everything is normal.

Beside this, a few other fixes for github.com hardcoded stuff has been fixed, but everything should be working now…

The PR can be found here: [https://github.com/microsoft/AL-Go/pull/2309](https://github.com/microsoft/AL-Go/pull/2309)

## Sources

-   [About GitHub Enterprise Cloud with data residency](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/about-github-enterprise-cloud-with-data-residency)
-   [Network details for GHE.com](https://docs.github.com/en/enterprise-cloud@latest/admin/data-residency/network-details-for-ghecom)
-   [About GitHub Enterprise Cloud](https://docs.github.com/en/enterprise-cloud@latest/admin/overview/about-github-enterprise-cloud)

Enjoy

**_/Freddy_**
