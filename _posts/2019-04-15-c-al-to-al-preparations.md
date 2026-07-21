---
layout: post
title: "C/AL to AL – preparations…"
date: 2019-04-15 11:45:09
categories: ["AL Development", "NavContainerHelper"]
tags: ["AL", "C/AL", "C/AL to AL"]
permalink: /2019/04/15/c-al-to-al-preparations/
---

**Update 2021/2/10:** Microsoft stopped creating images for Docker in the summer of 2020. We now publish artifacts, which can be used to spin up containers and BcContainerHelper has replaced NavContainerHelper. This blog post reflects the old way of using NAV/BC on Docker and references NavContainerHelper, which is outdated.

A few weeks ago at Directions Asia it was announced that Dynamics 365 Business Central Spring 2019 was the last release based on C/AL. Fall 2019 release is to be based on AL alone. In this blog post series I will try to give my view on how partners can go through this process.

# Extensions or not Extensions?, that’s not the question!

Business Central fall 2019 release will support 2 ways of customizing Business Central:

1.  Extensions (target External for online, target External or Internal for on premises)
2.  Code customizations (for on-premises only)

Yes, that is exactly the same as we support for the current release of Business Central, the biggest difference being that in Fall 2019 you will be performing your code customizations in AL and not in C/AL.

**Note, I love extensions and I think everybody should move to extensions as soon as possible, but I also realize that some partners do have solutions that are hard to move to extensions today.**

In this blog post series I will describe some tools available to you for moving your C/AL code to **BOTH** an AL extension **AND** a code customized AL solution.

For the sake of demoing these tools, I have created a small C/AL solution in NAV 2017 CU3 and will show the process of converting this solution to an AL extension and an AL code customized solution.

# My “demo” solution

The demo solution I am going to transfer to Business Central is a small solution, which implements a session list for Directions and includes a number of typical modifications:![demosolution](/assets/images/2019/c-al-to-al-preparations/demosolution.png)

-   A field (Directions Sponsor) is added to the customer table and code to initialize the field.
-   A new table (Directions Sessions) is added
-   2 new pages (Directions Session Card and Directions Session List) are added
-   The new field (Directions Sponsor) is added to the Customer Card Page
-   An Action is added to the Customer Card Page to open the Session List of sessions assigned to this customer
-   Initailization code of session list is added to codeunit 1
-   Directions Session List is added to the menusuite

I have placed the solution as a .txt file here: [https://bcdocker.blob.core.windows.net/public/DemoSolution.txt](https://bcdocker.blob.core.windows.net/public/DemoSolution.txt)

# Create a NAV 2017 CU3 NA development environment

If you like, you can follow along in the process with this demo solution before trying with your own solution. The solution was created using NAV 2017 CU3 NA. The first thing I need to do is to create a container with that exact version of NAV and import the objects.

On a machine with the latest NavContainerHelper installed, I run this script:

\# Settings
$auth = "NavUserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "C:\\temp\\license.flf"
$demoSolutionPath = "C:\\ProgramData\\NavContainerHelper\\DemoSolution.txt"

# Create NAV 2017 CU3 NA
New-NavContainer -accept\_eula \`
                 -imageName "mcr.microsoft.com/dynamicsnav:2017-cu3-na" \`
                 -containerName "nav2017" \`
                 -licenseFile $licenseFile \`
                 -auth $auth \`
                 -Credential $Credential \`
                 -updateHosts \`
                 -includeCSide
#
# Import and compile objects
#
if (!(Test-Path $demoSolutionPath)) {
    Download-File -sourceUrl "https://bcdocker.blob.core.windows.net/public/DemoSolution.txt" -destinationFile $demoSolutionPath
}
Import-ObjectsToNavContainer -containerName "nav2017" -objectsFile $demoSolutionPath
Compile-ObjectsInNavContainer -containerName "nav2017" -filter "Modified=Yes"

Now, I have a set of shortcuts on my desktop and I can start the Windows Client or the Web Client and see my NAV 2017 solution.

# Move my demo solution to Business Central Spring 2019 release

On this step, most partners knows more than me, as this is something they do frequently. I won’t try to educate anybody, but I want to re-state what I wrote two years ago in this blog post: [https://freddysblog.com/2017/03/31/what-would-you-do/](/2017/03/31/what-would-you-do/) – get your solution to step 4, where you minimize base app customizations and move your solution to an event based architecture easier to merge and easier to move to Business Central.

There might even be newer technologies since then, but the essence is clear – the more cleanup work you do before the move, the easier the move.

**Note: If you are going to move your solution to a code customized AL solution, you can of course leave code customizations there, but it is always a good idea to minimize the use of code customizations.**

I will run the following command to generate the deltas for my solution from NAV 2017:

Export-ModifiedObjectsAsDeltas -containerName "nav2017" -openFolder

This will open a folder with the my modifications:![Screenshot 2019-04-13 22.38.37](/assets/images/2019/c-al-to-al-preparations/screenshot-2019-04-13-22.38.37.png)

All TXT files are new objects, which should transfer directly, all DELTA files are modified objects, which might or might not be problematic.

Looking through my delta files, the codeunit 1 modifications must be refactored (as Codeunit 1 is no more) and the menusuite needs to be implemented in a different way.

In NAV 2017, I will remove my changes in Codeunit 1 to an event subscriber on OnCompanyOpen, still using Codeunit 1 as event publisher. Also the code modifications in the OnInsert trigger in Table 18 can be moved to an event subscriber on table 18 on the OnAfterInsertEvent.

After having refactored the necessary deltas, I will run this code to create a Business Central container and import my deltas.

\# Settings
$imageName = "mcr.microsoft.com/businesscentral/onprem:1904-rtm"
$auth = "NavUserPassword"
$credential = New-Object pscredential 'admin', (ConvertTo-SecureString -String 'P@ssword1' -AsPlainText -Force)
$licenseFile = "C:\\temp\\license.flf"

# Create Business Central container
New-NavContainer -accept\_eula \`
                 -imageName $imageName \`
                 -containerName "bc" \`
                 -licenseFile "C:\\temp\\license.flf" \`
                 -auth $auth \`
                 -Credential $Credential \`
                 -updateHosts \`
                 -includeCSide

# Import and compile Deltas
Import-DeltasToNavContainer -containerName "bc" -deltaFolder "C:\\ProgramData\\NavContainerHelper\\Extensions\\nav2017\\delta"
Compile-ObjectsInNavContainer -containerName "bc" -filter "Modified=Yes"

After the conversion, I will have to change the event publisher on the OnAfterCompanyOpen to codeunit 40 instead of codeunit 1.

As part of this conversion, I need to make sure that everything works, all tests run and all scenarios works as expected. Any error or mistake at this place will just carry over when converting to AL.

After this – I remove my NAV 2017 container

Remove-NavContainer -containerName nav2017

and now I am ready for the next step which is the actual move to AL.

When moving to AL, you can select

-   [C/AL to AL extension](/2019/04/15/c-al-to-al-extension/)

or

-   [C/AL to AL code customizations](/2019/04/15/c-al-to-al-code-customizations/)

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist

‘
