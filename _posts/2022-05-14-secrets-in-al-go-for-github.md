---
layout: post
title: "Secrets in AL-Go for GitHub"
date: 2022-05-14 07:24:53
categories: ["CI/CD"]
tags: ["Azure KeyVault", "Environments", "Forks", "GitHub Secrets", "Organization", "Repository", "Secrets"]
permalink: /2022/05/14/secrets-in-al-go-for-github/
---

This blog post will not reveal any secrets in AL-Go for GitHub:-)

Instead, it will explain ways for you to store secrets, which are used for AL-Go for GitHub. In almost every DevOps setup, you will have to store some keys, passwords, tokens or like. In GitHub, these are called secrets and AL-Go will look for a set of secrets by their name.

This blog post will also touch upon how you can use GitHub organizations and environments with your customer projects.

## GitHub secrets vs. Azure KeyVault

AL-Go for GitHub supports two ways of storing your secrets, **GitHub secrets** or Azure KeyVault. If you store your secrets in an **Azure KeyVault** you need to provide access to the Azure KeyVault using a GitHub secret. This is done by following [this guideline](https://docs.microsoft.com/en-us/azure/developer/github/github-key-vault) including the **Add a role assignment** step. The location of the GitHub secret created determines the access to the KeyVault, meaning that you can have and use multiple KeyVaults if you like.

GitHub secrets can be **repository secrets**, created for a single **repository**, they can be **organization secrets**, created one or more repositories in the **organization** or they can be **environment secrets**, created for a **single environment**.

You can set GitHub secrets through the GitHub API (or GH command line), but you cannot access the secrets from your local machine.

## Repository secrets

Repository secrets are available to all repositories (private or public, personal or organizational). If a secret is created for a repository, it is only accessible within GitHub actions for that repository. Repository secrets are not included when forking a repository (obviously).

For repositories created under your personal account, this is really the only place you can store secrets for the workflows. For repositories created under an organization, you can also use organization secrets.

## Organization secrets

If a secret is created for an organization, you can select which repositories the secret should be available for:

![](/assets/images/2022/secrets-in-al-go-for-github/image-43.png)

The **InsiderSasToken** is a good example of an organization secret, which typically is available to all repositories and when you renew the value of the token every 6 months, it automatically renews for all repos.

The **LicenseFileUrl** is an example of an organization secret, which typically will be available to a selected number of repositories. If you have two different License files, which each should be available to a number of repositories, you specify which secret to use in the settings of the individual repository. More about this later.

## Environment secrets

If a secret is created for an environment, the secret is only available during deployment to that environment. Environments (and thus also environment secrets) are only available for public repositories or for repositories in organizations with Teams or Enterprise subscription.

Currently, you can place 3 secrets under the environment:

-   **EnvironmentName** contains the actual Business Central Environment name. If not specified, the name of the environment in GitHub is used.
-   **AuthContext** contains the authorization context for the environment.
-   **Projects** is only used if the repository is multi-project. The secret contains a comma separated list of the projects to be published to the environment. Default is all. (Note that every project can have multiple apps, which are all included)

If you are using a private repository in a free GitHub subscription, where environments are not available, you can create environments in the settings file and you can create these secrets as **<environmentname>\_EnvironmentName** and **<environmentname>\_AuthContext**.

At the time writing this blog post, you cannot specify projects unless using environment secrets. In a future version, it will be possible to specify projects as a setting instead.

## Priority order

If secrets exist in multiple places, the order of priority is

1.  Environment secret
2.  Repository secret
3.  Organization secret
4.  Azure KeyVault specified in environment secret
5.  Azure KeyVault specified in repository secret
6.  Azure KeyVault specified in organization secret

As earlier stated, environment secrets are only available during deployment to the specific environment.

## Forks

If you fork a GitHub repository to your personal or organization account, you obviously do NOT get a copy of the secrets from that repository. GitHub actions in the main repository, also cannot access the secrets while running a CI/CD workflow based on a Pull Request from a fork (**An evil PR could reveal the secrets**).

This means that if your repository contains an AppSource App (which currently always requires a license file URL), you will always get this error during the CI/CD workflow run on the Pull Request from a fork:

![](/assets/images/2022/secrets-in-al-go-for-github/image-45.png)

and in the details:

![](/assets/images/2022/secrets-in-al-go-for-github/image-44.png)

If you review the code and decide to merge anyway, the CI/CD workflow will re-run based on the commit and will now have access to the secrets.

For this and other reasons, [the recommendation is to use feature branches](/2022/05/03/branching-strategies-for-your-al-go-for-github-repo/), from which the Pull Request will have access to the necessary secrets and the ability to build, publish and test your app.

In a future version AL-Go we will modify the CI/CD workflow to still perform the compile during the Pull Request but skip the publish step and the test run step and replace the error with a warning stated that the app has not been published and tested.

## Named secrets

AL-Go for GitHub uses settings and secrets to enable functionality.

If the Code Sign certificate secrets are available, then the CI/CD workflow automatically signs your app, if the LicenseFileUrl secret is available, it is used during build, if a StorageContext secret is available, the CI/CD workflow will automatically publish build artifacts to a storage account, etc. etc.

Below is a list of named secrets you can create in order to get and use additional functionality.

<table><tbody><tr><td>GhTokenWorkflow</td><td>Whenever you run the <strong>Update AL-Go System Files</strong> workflow, you will need a secret called GhTokenWorkflow containing a personal access token with workflow permissions, like described <a rel="noreferrer noopener" href="https://github.com/microsoft/AL-Go/blob/main/Scenarios/UpdateAlGoSystemFiles.md" target="_blank">here</a>.</td></tr><tr><td>InsiderSasToken</td><td>The Insider Sas Token is available for partners on <a rel="noreferrer noopener" href="https://aka.ms/collaborate" target="_blank">https://aka.ms/collaborate</a> to allow access to insider (future) builds of Business Central. This secret is needed for the <strong>Test Next Minor </strong>and <strong>Test Next Major </strong>workflows.</td></tr><tr><td>LicenseFileUrl</td><td>For <strong>AppSource apps</strong>, you need to specify a direct download URL to your license file in this secret.</td></tr><tr><td>CodeSignCertificateUrl<br>and<br>CodeSignCertificatePassword</td><td>For <strong>AppSource apps</strong>, you need to specify a code signing certificate. The secret named CodeSignCertificateUrl needs to contain a direct download URL to your code signing certificate in .pfx format and a secret named CodeSignCertificatePassword needs to contain the pfx password.</td></tr><tr><td>KeyVaultCertificateUrl,<br>KeyVaultCertificatePassword<br>and<br>KeyVaultClientId<br></td><td>If your<strong> AppSource app</strong> needs KeyVault access, you need to provide these three secrets according to the requirements described <a rel="noreferrer noopener" href="https://github.com/microsoft/AL-Go/blob/preview/Scenarios/EnableKeyVaultForAppSourceApp.md" target="_blank">here</a>.</td></tr><tr><td>StorageContext</td><td>If you specify a StorageContext secret, your app artifacts are copied to a storage account using the specifications in the StorageContext secret.</td></tr><tr><td>&lt;environmentName&gt;_AuthContext<br>&lt;environmentName&gt;-AuthContext<br>AuthContext</td><td>If your <strong>authorization context</strong> is specified as an environment secret, it can be named AuthContext, else you need to create a secret with the environment name followed by underscore or dash and AuthContext, like described <a rel="noreferrer noopener" href="https://freddysblog.com/2022/05/06/deployment-strategies-and-al-go-for-github/" target="_blank">here</a>.</td></tr><tr><td>&lt;environmentName&gt;_EnvironmentName<br>&lt;environmentName&gt;-EnvironmentName<br>EnvironmentName</td><td>If your actual <strong>environment name </strong>is different from the environment name in AL-Go for GitHub, you need an EnvironmentName secret as described <a rel="noreferrer noopener" href="https://freddysblog.com/2022/05/06/deployment-strategies-and-al-go-for-github/" target="_blank">here</a>.</td></tr></tbody></table>

When adding new functionality to AL-Go for GitHub, more secrets might be added.

## Secret name indirections

As already mentioned, secrets are located by name and secrets can be shared between multiple repositories. Let’s say you have two License files. One license file should be used for 5 of your repositories and another license file should be used for another 5 of your repositories. You can however not create two organization secrets called LicenseFileUrl and assign them to 5 repositories each and you don’t really want to add the secrets to the individual repositories, as this means you would have to modify every single repo when you have an update to the secret.

For this, you can use secret name indirections. In the individual repository, you can create a setting called **LicenseFileUrlSecretName** containing the value **ContosoLicenseFile**, then the secret called ConsotoLicenseFile is used as LicenseFileUrl in this repository.

The same mechanism can be used for all secrets. Create a setting in the repository called **<secretname>SecretName** and your secret will be read from that name instead.

## Recommendations

My recommendation when it comes to organizations and secrets are as follows:

-   **One GitHub organization per partner (use Teams or Enterprise subscription)**
    -   for IP owned by the partner
    -   Use organizational secrets for developer license, certificates etc.
    -   Use environment secrets for authorization to customer environments
    -   Setup self-hosted agents for the organization to improve build performance
-   **One GitHub organization per customer (Free or Teams license)**
    -   for IP owned by or shared with the customer
    -   use organization secrets for authorization to customer environments
    -   Optionally use self-hosted agents for performance

Hope this helps

**_Freddy Kristiansen_**  
Technical Evangelist
