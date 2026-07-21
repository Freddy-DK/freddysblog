---
layout: post
title: "Branching strategies for your AL-Go for GitHub repo"
date: 2022-05-03 08:35:16
categories: ["AL Development", "CI/CD"]
tags: ["AL-Go for GitHub", "Automated testing", "Branch", "Branching", "Code Review", "Pull Request"]
permalink: /2022/05/03/branching-strategies-for-your-al-go-for-github-repo/
---

If you teach yourself to follow a fairly simple set of rules, you will see that the health of your project will increase dramatically, and you will be in a better place with your project development.

So, I will at the ripe age of 56 bestow upon you these 5 rules, which can be applied to any project, using any DevOps setup. In the following I will explain how to implement these rules using AL-Go for GitHub:

1.  Use Pull Requests
2.  Use Code Reviews
3.  Use automated testing
4.  Use Feature branches
5.  Use Releases and release branches

Godspeed!

## Use Pull Requests

Using Pull Requests (PRs) when developing means that no developer can check-in code directly to the main branch. For some, it might not seem like a problem to check-in code directly to main, but it has a number of risks associated – especially if you are multiple developers working on the same repository.

Let me give one example. Developer 1 checks in a change, which renames a function. He of course also changes the code wherever the function was called. The CI/CD pipeline will kick off and succeed. Developer 2 checks in a change, using the old function name. His check-in will succeed, and the CI/CD pipeline will kick off and fail. At this time your main branch is broken and any developer working on the project will be unable to work until the situation is resolved. Needless to say, the situation can be much more complex and harder to discover. Using pull requests helps you ensuring a stable main branch at all times.

When using pull requests, developers will create a fork or a branch in which they will complete their work. When creating the pull request, their change will be merged with main, and a CI/CD workflow will be run on the merged code before the code is pushed into the repository. You can also have fellow developers review your code, add comments and suggestions and every time you make a change to your feature branch, the PR will re-run the CI/CD workflow with these changes, minimizing those chances of your main branch being broken, not guaranteeing that it never happens.

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-6.png)

## Use code reviews

As you can see, in this repository I also setup that one code reviewer is required and of course I cannot add my own review. Other developers on the project will have this in the PR:

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-7.png)

When all required branch rules have been met, you can merge the PR and delete the feature branch.:

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-8.png)

Note that the AL-Go CI/CD pipeline running on the Pull Request will **skip the Deploy step**. Only completed PRs are deployed after pushed to the repo (else people could submit a PR from a fork and end up with running code in a sandbox environment without anybody approving anything. For this, the deploy step looks like this:

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-11.png)

I strongly encourage you to enforce Pull Requests on your repository and it is fairly easy to do. In your repository, click **Settings** -> **Branches** and next to **Branch protection rules**, press **Add rule**. In the Branch name pattern, write **main** and check the box next to **Require a pull request before merging**.

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-5.png)

There are a lot of other settings you can check in order to harden the security on your branch even more but require pull requests is IMO the most important.

### Reverting pull requests

Another very nice thing with pull requests is the simplicity of which you can revert a PR. If you discover that you approved a PR on your repository, which didn’t do what you expected, you can always go back to the closed PR and click Revert.

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-16.png)

Reverting a PR will create a new PR, which undo’s what the other PR did. It is probably not something you will use a lot, but very nice when needed.

## Use feature branches

The AL-Go CI/CD workflow is defined to trigger on **any pull request** and any **push** to the **main** branch and to any branches, where the name matches **release/\*** or **feature/\***.

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-9.png)

This means that if you are working on larger features in a team, you could name the branch **feature/something**, in order to enable CI on that branch as well (Remember, deploy will only run on main), but you would ensure that everything compiles, all tests are run and when the feature is done, you can create a PR to merge it into main.

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-10.png)

You can use a different naming algorithm (like **user/feature**) for branches, where you specifically do not want to run a CI pipeline on every check-in but want to do this on the PR when requesting to merge your feature into main.

## Use automated testing

If you aren’t already, you should, and if you don’t know how to, I recommend [this book](https://subscription.packtpub.com/product/business_and_other/9781801816427) from my dear friend and Microsoft MVP Luc van Vugt

Luc is Mr. Automated Testing, and you will find his blog [here](https://www.fluxxus.nl/).

In AL-Go for GitHub all apps specified in the testFolders setting (or if no folders are specified in settings, all apps with a dependency on the test framework or including codeunits with subtype test) will be compiled as test apps, published and all tests in these test apps will be executed.

If there are test failures, they will be displayed directly in the workflow and the workflow will fail:

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-12.png)

AL-Go for GitHub unfortunately doesn’t have a nice UI in which the test results are displayed, instead you have to download and inspect the test results file, where you will be able to see the error, the stack trace etc.

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-13.png)

You can also see the test results directly in the build output, but it might be hard to navigate through the output:

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-14.png)

We will add a nicer experience for viewing test results, seeing test statistics etc. in the future and as an AL-Go for GitHub user, you will just download and apply this new functionality to your repository when available.

If you have tests, which should be disabled, you can place a disabledTests.json file in the folder of the test app. The format of the file is straightforward:

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-15.png)

We are also planning to enable you to run the Microsoft Test Suite as part of your pipeline, simply by stating that you want to do that.

We will also be adding code coverage, automated performance testing and many more features to AL-Go for GitHub going forward for you to download and apply to your repository.

## Use releases and release branches

When you are done with the development of your app, release it. Not just by compiling the app on your laptop and upload it to your customer or to AppSource – create a release.

In AL-Go there is a workflow called **Create Release**. In the Create Release you will specify a number of parameters:

![](/assets/images/2022/branching-strategies-for-your-al-go-for-github-repo/image-17.png)

In this example, I will create a release with the name **v1.0**. The tag **1.0.0** needs to follow semantic versioning 2.0 rules ([https://semver.org](https://semver.org)) – GitHub standard and it ensures correct sorting of releases. I recommend using **Major** and **Minor** from your repo version number and **0 in the patch field**. If you ever need to ship a hotfix for this release, you **increase the patch field**.

You can create a draft or a pre-release of your app to indicate that you are doing final acceptance testing on the app and you can create a release branch in which you will finalize or hotfix the released app.

As you saw earlier, the CI/CD workflow will also run on check-ins to the release branch and the version of the app, which will be used as the “previous app” is the released version in this branch. No automatic deployment will happen on a release branch.

I also recommend that you type in the **new version number in main** right here and update the version number in main. The reason for updating the version number in main is that the automatic version numbering will continue in the release branch and if the main branch and the release branch share the same major.minor version number, **you will run into issues**, where a build from the release branch suddenly is seen as newer than a build from main – which isn’t correct.

For PTEs, the **Publish To Environment** will by default grab the **current version** (which is the latest released version) and publish that to the online environments specified. Some time in the future when automated publish to AppSource is possible, the AppSource template will get a workflow for publishing to AppSource directly from your repository, at no cost to you.

When you need to hand carry bits to customers or to AppSource – **always grab them from the latest release**.

With that,…

Enjoy

**_Freddy Kristiansen_**  
Technical Evangelist
