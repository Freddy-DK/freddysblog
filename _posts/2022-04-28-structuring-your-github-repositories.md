---
layout: post
title: "Structuring your AL-Go for GitHub repositories"
date: 2022-04-28 21:16:53
categories: ["AL Development", "CI/CD"]
tags: ["AL-Go for GitHub", "App Development", "CI/CD", "GitHub"]
permalink: /2022/04/28/structuring-your-github-repositories/
---

When developing apps for Business Central, you very frequently will have more than one app. You might split one customer specific app into multiple smaller apps or you might have a common app, which contains common functionality for all your customer specific or AppSource apps.

This blog post describes the thinking behind AL-Go for GitHub and how we recommend you setup your GitHub repository structure and why…

## High Level

Use the shipping vehicle to determine the repository/project structure.

On a high level, the recommendation is to place apps, which are shipped together, in the same repository/project.

Apps which are shipped together with more than one other app (like a common library), but never alone should be placed in another repository/project – potentially as two apps in one project or two projects in one repository.

More about multiple projects in one repository later.

### Example

Let’s say that our App1 offering consists of 4 apps:

-   App1 is the main app, which has dependencies on all other apps
-   LibraryApp1 is a library app with functionality specific to app1, which depends on Common
-   Licensing is a licensing app, containing functionality for licensing for all your apps, which depends on Common
-   Common is a library app, containing common functionality for multiple apps

![](/assets/images/2022/structuring-your-github-repositories/image-1.png)

![](/assets/images/2022/structuring-your-github-repositories/image-2.png)

In this case, the recommendation is to place **Licensing** and **Common** in one repository and **App1** and **LibraryApp1** in another repository, with a dependency on the first repository.

You can then decide whether you place Licensing and Common in the same project in the first repository (built together), or you place them in two projects (built seperately), more about this later.

In AL-Go for GitHub, you can create dependencies in two ways

1.  Create a dependency on a repository
2.  Add a direct download URL to the app(s) you want to download and install

In both cases, AL-Go will only actually install the apps, which are included in the dependencies of the apps in the main repository (unless you set **installOnlyReferencedApps** to false in your settings file).

## Repository dependencies

In AL-Go for GitHub, you can create dependencies on other AL-Go repositories. The way you do that is by specifying **appDependencyProbingPaths** in the settings file, example:

```
"appDependencyProbingPaths": [
        {
            "repo":  "https://github.com/BusinessCentralApps/Contoso-Common",
            "release_status":  "latestBuild",
            "version":  "latest",
            "projects":  "*",
            "AuthTokenSecret":  "CommonAppsRepoToken"
        }
    ]
```

In this example, my repository will (during build) download all apps from the latest successful build from the **Contoso-Common** repository and install the apps, which are in the dependency tree of my main app.

Note that the **AuthTokenSecret** is not the secret itself, but instead the name of the secret containing the authorization token.

The default value for **release\_status** is **release** – meaning that it will grab apps from the release, that matches the version. Version can be set to latest or to a specific version and projects determines which projects to include if the repository is multi-project. Note that project is not the same as an app.

Right now, it is not a requirement to release your project, that the dependency on other apps must be released bits as well. This means that your app could be built with a dependency on a non-released app. My recommendation is that unless you specifically need new functionality from another repo, you should always have release\_status set to release.

![](/assets/images/2022/structuring-your-github-repositories/image-9.png)

You can see the Contoso-App1 sample [here](https://github.com/BusinessCentralApps/Contoso-App1) and check the settings and dependencies. Contoso-Common can be found [here](https://github.com/BusinessCentralApps/Contoso-Common).

## Direct download URL

The second way of adding dependencies in AL-Go for GitHub is to create a setting called **InstallApps**. InstallApps is an array and can contain any number of direct download URLs (or relative paths inside the repository) to .app files or .zip files including any number of .app files.

```
"installApps": [
        "https://my.azureedge.net/myapp/latest/apps.zip"
    ]
```

or if the app is included in a folder in the repository:

```
"installApps": [
        "myapp/latest.app"
    ]
```

Again, only the apps that are actually references by apps in the repository are installed.

## Multi-project repositories

The default setup in AL-Go is to have one project in a repository. One project can contain multiple apps, but they are all compiled, published and tested in the same version of Business Central. A repository can also contain multiple projects, where each project is one or more apps.

All apps in one project will be built towards the same country version and the same Business Central build. The lowest compatible Business Central version (the application dependency) is determined by looking at the application dependency of all apps in the project and grabbing the highest.

### Example:

Now, let’s say you have one app for the world-wide market (w1) and then you have an über-app for some of the country versions (dk and it). Your main app consists of:

-   App2.w1 is the main app for w1 (on AppSource)
-   App2.library is a library app for App2 (with all the functionality really)

and then you have two additional apps

-   App2.dk
-   App2.it

both with a dependency on App2.library

Furthermore, you have the Common and the Licensing apps from the first example as well.

![](/assets/images/2022/structuring-your-github-repositories/image-3.png)

and

![](/assets/images/2022/structuring-your-github-repositories/image-6.png)

plus

![](/assets/images/2022/structuring-your-github-repositories/image-7.png)

In this case, the recommendation is to place Common and Licensing in one repository and the other apps in another repository with **3** projects, one for **App2.dk**, one for **App2.it** and one for **App2.w1+App2.library**.

You could create **4** projects and split **App2.w1** from **App2.Library**, but since it is easier to develop apps in the same repo/project, it might be easier to keep **3** projects.

You could also place **App2.w1** and **App2.Library** in one repository alone and then have another repo with projects for **App2.dk** and **App2.it**.

You cannot put **App2.dk** and **App2.it** into the same project as **App2.w1**, since these are having different dependencies on the base application.

When building a multi-project repository, all projects are built simultaneously and if you modify a file in one project in the repository, then only that project will be built.

![](/assets/images/2022/structuring-your-github-repositories/image-8.png)

You can see the Contoso-App2 sample [here](https://github.com/BusinessCentralApps/Contoso-App2) and check the settings and dependencies.

## GenerateDependencyArtifact

When building projects with dependencies to other repositories, you can add a setting called GenerateDependencyArtifacts and set it to true. With this setting, every build will now include an artifact with all the external dependency apps used to build the app.

For contoso-app1, the dependency artifact looks like this:

![](/assets/images/2022/structuring-your-github-repositories/image-10.png)

And for Contoso-app2, you will have 3 dependency artifacts, next to the Apps and the BuildOutput artifacts. If you have test apps, there will also be a test apps artifact and a test results artifact.

![](/assets/images/2022/structuring-your-github-repositories/image-11.png)

## Other app configurations?

If you have other app configurations, which wasn’t directly or indirectly described in this blog post here, then please let me know about them and I will try to add a chapter explaining how (if possible) you can do this in AL-Go for GitHub.

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
