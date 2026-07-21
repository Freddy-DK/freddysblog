---
layout: post
title: "How is this perf testing thing actually working?"
date: 2017-01-22 16:04:05
categories: ["Load Test"]
tags: ["Azure", "NAV", "NAV 2017", "PERF", "Performance", "Performance test"]
permalink: /2017/01/22/how-is-this-perf-testing-thing-actually-working/
---

This post is #3 in a series of posts about performance testing.

[Post #1](https://freddysblog.com/2016/12/16/so-you-want-to-get-started-on-perf-testing-huh/) was all about setting up an instance of NAV on Azure and get perf tests up running.

[Post #2](https://freddysblog.com/2016/12/17/perf-testing-with-multiple-users/) was all about scaling the number of users and running multi-tenancy.

But what actually happens when running perf tests?

When running a perf test called _OpenCustomerList_ it doesn’t take a lot of thinking to figure out what the test does, but how does it do it?

The _CreateAndPostSalesOrder_ test will Create a Sales Order and Post it, but how?

### The core

The core of perf testing is really to simulate users and user actions. This can of course be done using tools, which can control either the Windows Client or the Web Client in a browser and simulate key presses and mouse events. There are a lot of tools with this functionality, but I would argue that they all rely very much on how the Client is implemented and the how the rendering is done.

Perf testing is slightly different.

If you investigate the settings in the perf test solution you will find a setting called _NAVClientService_, which is set to

https:///NAV/WebClient/cs

cs?

If you try to open this Url in a browser it will fail.

If you remove cs, then you have the Url for the Web Client, so what is this cs?

cs is short for Client Services and in order to communicate with the Client Services endpoint of your NAV Server, you will have to have the _Microsoft.Dynamics.Framework.UI.Client.dll_, which you will find on the DVD, in the _Test Assemblies_ folder:

[![testassemblies](/assets/images/2017/how-is-this-perf-testing-thing-actually-working/08e3f-testassemblies-1.png)](/assets/images/2017/how-is-this-perf-testing-thing-actually-working/08e3f-testassemblies.png)

You should not try to communicate with this endpoint “manually”.

The Client Services endpoint allows you to create a new Client. Unlike Soap or OData Web Services, Client Services will open a session on the server. You will tell the server what action you want to invoke and the server will tell you if you need to display a new page to the user. Doing perf testing we of course do not render the pages physically, but we do everything that’s needed to make the NAV Server think that there are real users behind a real client using the software.

### The Visual Studio Solution in Github

The solution you cloned in part #1, has 3 projects:

-   Microsoft.Dynamics.Nav.LoadTest
-   Microsoft.Dynamics.Nav.TestUtilities
-   Microsoft.Dynamics.Nav.UserSession

The first project is where the scenarios are defined and the test mix.

The other 2 projects are there to make the communication with the Client Services endpoint a little easier.

Lets follow the flow when we run the CreateAndPostSalesOrder:

The Test Method looks like this:

\[TestMethod\]
public void CreateAndPostSalesOrder()
{
    TestScenario.Run(OrderProcessorUserContextManager, TestContext, RunCreateAndPostSalesOrder);
}

_TestScenario.Run_ is a helper function, which takes a _UserContextManager_, a _TestContext_ and the actual test method as parameters.

The _UserContextManager_ is a class, which is responsible for creating Users of a certain Role. TestScenario.Run will only ask the UserContextManager for a new UserContext if no users are available in the pool and the number of simultaneous users haven’t been reached yet.

_TestContext_ is the test context provided by Visual Studio Load Test Framework.

### TestScenario.Run

Let’s look at what TestScenario.Run actually does:

public static void Run(UserContextManager manager, TestContext testContext, Action action, string actionName = null)
{
    var userContext = manager.GetUserContext(testContext);
    var formCount = userContext.GetOpenFormCount();
    action(userContext);
    userContext.CheckOpenForms(formCount);
    userContext.WaitForReady();
    manager.ReturnUserContext(testContext, userContext);
}

Line by line:

1.  Get a UserContext from the UserContextManager, either by getting one from the pool of users or by creating a new user.
2.  Remember the current number of open forms for later
3.  Perform the actual test scenario
4.  Check whether the test scenario closed all the forms that was opened, throw an error if not (ensure session health)
5.  Wait for the UserContext to be ready
6.  return the UserContext to the pool of UserContexts

### The UserContextManager

The test scenario runner consults the _UserContextManager_ twice. Once for getting a _UserContext_ and once for returning the “used” _UserContext_ to the manager. The Github sample implements two UserContextManagers, one with NAVUserPassword Authentication and one with Windows Authentication. In the sample we then instantiate one of these with proper parameters.

The _UserContextManager_ is also responsible for distributing users between multiple tenants (if running multi-tenancy) and for selecting company. The Github sample doesn’t really implement this, but in [Post #2](https://freddysblog.com/2016/12/17/perf-testing-with-multiple-users/) you will see an example of how you could implement this in the _UserContextManager_. In real life load testing you will probably find yourself create at least one new _UserContextManager_ class deriving from one of the existing classes and implementing other ways of managing _users_, _tenants_ and _companies_, you shouldn’t need to modify the base objects in the UserSession project.

The two methods you want to override are

///
/// Get the UserName for the current virtual user
///

/// current test context /// protected abstract string GetUserName(TestContext testContext); ///  
/// Create a new user context for the current virtual user ///

/// current test context /// protected abstract UserContext CreateUserContext(TestContext testContext);

_GetUserName_ is currently only used in _CreateUserContext_.

In the GitHub sample, when using Nav User Password authentication, _GetUserName_ will check whether you are running load tests. If that is the case it will append the load test user id (0, 1, 2, 3,…) to the default username (from settings, ex. admin0, admin1, admin2, …). If you right-click a test and select _Run Selected Test_, the GitHub sample will just connect with the default username from settings.

In the GitHub sample, _CreateUserContext_ will transfer _TenandId_ and _Company_ as static fields from the _UserContextManager_ to the _UserContext_ class. This is where you would implement your own distribution mechanism in your own _UserContextManager_ class.

Note, in NAV 2017 there seem to be a bug, which means that the company selection won’t actually be used. All users will connect to the company they have specified in User Personalization.

### The actual test scenario: RunCreateAndPostSalesOrder

_RunCreateAndPostSalesOrder_ is called with the _UserContext_ and as stated earlier, the actual test scenario will simulate what the user is doing, not by invoking key presses and mouse clicks, but by performing the logical interactions, that the user is doing. If you think about it, the user might be able to do a million things with NAV, but on the interaction level there are only so many things:

-   Enter values in controls
-   Inspect values in controls
-   Activate controls
-   Invoke actions

There are probably more, but for now we will settle with this.

You might be thinking: Hey, on my phone, I can swipe left on a customer and stuff happens, but if you think about it, this is just a different way of invoking an action, which is specific to a phone display target. You cannot perform a swipe through Client Services, but you can invoke the same action as the swipe performs.

Closing a Page is invoking an action (different in different display targets)

Opening a Page is not an interaction the user typically is doing. The typical interaction is to invoke an action, which as a side effect will open a Page (due to some PAGE.RUN code in the action. Yes I know you can open a specific page by changing the URL in the WebClient, but it isn’t the typical navigation paradigm.

With this in mind, lets look at the RunCreateAndPostSalesOrder code:

public void RunCreateAndPostSalesOrder(UserContext userContext)
{
    // Invoke using the new sales order action on Role Center
    var newSalesOrderPage = userContext.EnsurePage(SalesOrderPageId, userContext.RoleCenterPage.Action("Sales Order").InvokeCatchForm());
    // Start in the No. field
    newSalesOrderPage.Control("No.").Activate();
    // Navigate to Customer field in order to create record
    newSalesOrderPage.Control("Customer").Activate();
    var newSalesOrderNo = newSalesOrderPage.Control("No.").StringValue;
    TestContext.WriteLine("Created Sales Order No. {0}", newSalesOrderNo);
    // select a random customer
    var custno = TestScenario.SelectRandomRecordFromListPage(TestContext, CustomerListPageId, userContext, "No.");
    // Set Customer to a Random Customer and ignore any credit warning
    TestScenario.SaveValueAndIgnoreWarning(TestContext, userContext, newSalesOrderPage.Control("Customer"), custno);
    TestScenario.SaveValueWithDelay(newSalesOrderPage.Control("External Document No."), custno);
    userContext.ValidateForm(newSalesOrderPage);
    // Add a random number of lines between 2 and 5
    int noOfLines = SafeRandom.GetRandomNext(2, 6);
    for (int line = 0; line < noOfLines; line++)
    {
        AddSalesOrderLine(userContext, newSalesOrderPage, line);
    }
    // Check Validation errors
    userContext.ValidateForm(newSalesOrderPage);
    PostSalesOrder(userContext, newSalesOrderPage);
    // Close the page
    TestScenario.ClosePage(TestContext, userContext, newSalesOrderPage);
}

The first thing that happens here is:

userContext.RoleCenterPage.Action("Sales Order").InvokeCatchForm()

Locate the Sales Order Action on the Role Center, Invoke it and catch the Form that it opens.

This call is encapsulated in a call to EnsurePage, which basically checks whether the page opened by the action is the SalesOrderPage. If this is not the case, the method will throw an exception.

var newSalesOrderPage = userContext.EnsurePage(SalesOrderPageId, userContext.RoleCenterPage.Action("Sales Order").InvokeCatchForm());

This means that we can continue our test scenario flow, knowing that newSalesOrderPage is indeed the Sales Order Page.

The first thing we do in the newSalesOrderPage is to activate the No. field. It is the responsibility of the display target to activate the first control and since we are the display target, we have to do this:

newSalesOrderPage.Control("No.").Activate();

Next thing is activating the Customer control which, as all NAV users will know, means that the actual record is created and the Sales Order No. is filled out. After activating the Customer control, we can inspect the No. control and get the new Sales Order No. (and write it to the test output).

newSalesOrderPage.Control("Customer").Activate();
var newSalesOrderNo = newSalesOrderPage.Control("No.").StringValue;
TestContext.WriteLine("Created Sales Order No. {0}", newSalesOrderNo);

Next thing is to simulate the user pressing the drop down button and select a random customer. In this sample we don’t actually invoke the drop down but instead we select a random customer from the list page that lies behind the drop down.

var custno = TestScenario.SelectRandomRecordFromListPage(TestContext, CustomerListPageId, userContext, "No.");

Next thing – set the value of the customer in the Customer field:

TestScenario.SaveValueAndIgnoreWarning(TestContext, userContext, newSalesOrderPage.Control("Customer"), custno);

The _SaveValueAndIgnoreWarning_ is a method, which will save the value in a field (with delay) and if that action causes a dialog to popup, it automatically tries to press Ignore. If there isn’t an ignore button on the dialog, the function will throw and the test will fail. This is to ensure that stuff like credit limit doesn’t prevent our tests from running.

After setting the Customer, set the _External Document No._ to the customer no as well (or any random number really):

TestScenario.SaveValueWithDelay(newSalesOrderPage.Control("External Document No."), custno);

_SaveValueWithDelay_ will save the value in a control and sleep for 400ms. This delay is set in _DelayTiming.cs_ and can of course be changed.

The next thing that happens in not really a user interaction, but it is ensuring that we don’t have any validation errors before starting to add lines to the sales order:

userContext.ValidateForm(newSalesOrderPage);

Next up is adding the lines, in the sample we add a random number of lines:

int noOfLines = SafeRandom.GetRandomNext(2, 6);
for (int line = 0; line < noOfLines; line++)
{
    AddSalesOrderLine(userContext, newSalesOrderPage, line);
}

In the AddSalesOrderLine it does really the same things as above. Only difference is getting the current line and adding a think delay after filling out the line.

After this, check for validation errors, post the order and close the page.

### Adding a line

When dealing with lines (repeaters), you need to find the repeater and then find the right line. In the sample project this is done by:

// Get Line
var itemsLine = newSalesOrderPage.Repeater().DefaultViewport\[line\];

If you are going to add more than 5 lines, you will need to scroll down to the desired line (exactly like a user would do in the UI) and then get the desired line. In the NAVLoadTest repository you will find a sample on how this is done:

var repeater = newSalesOrderPage.Repeater();
var rowCount = repeater.Offset + repeater.DefaultViewport.Count;
if (line >= rowCount)
{
    // scroll to the next viewport
    userContext.InvokeInteraction(new ScrollRepeaterInteraction(repeater, 1));
}
var rowIndex = (int)(line - repeater.Offset);
var itemsLine = repeater.DefaultViewport\[rowIndex\];

If you look into the Repeater() method, it is an Extension method to the ClientLogicalForm and finds the first ClientRepeaterControl in the control tree under the page (including sub pages).

return form.ContainedControls.OfType().First();

If you want to find a different repeater (if multiple exists) you will have to write your own extension method to do that.

After getting the Repeater, we need to find the correct line, potentially scrolling down and then get the desired line. The line has controls just like the page, meaning that you can do stuff like this on a line:

// set Type = Item
TestScenario.SaveValueWithDelay(itemsLine.Control("Type"), "Item");

### Posting the Order

Posting the order seems straightforward, but it is a little more complicated than. Locate the **Post…** action and invoke the action:

postConfirmationDialog = newSalesOrderPage.Action("Post...").InvokeCatchDialog();

On the postConfirmationDialog, locate the OK button and press that.

ClientLogicalForm dialog = userContext.CatchDialog(postConfirmationDialog.Action("OK").Invoke);

If pressing OK on the confirmation dialog causes a dialog to popup, press No on that:

if (dialog != null)
{
    // The order has been posted and moved to the posted invoices tab, do you want to open...
    dialog.Action("No").Invoke();
}

You probably got the picture now, every time the user is expected to do something, you need to code that.

### Isn’t there an easier way?

Yes and No. I have helped a few partners write performance tests and I have asked them to describe their scenarios (if possible with a few videos recorded of users doing the actual work) and then we have written the code based on these descriptions/recordings. It does however take a lot of time.

At Directions US 2016, I talked to the guys from ClickLearn ([https://www.clicklearn.dk/dynamics/nav/](https://www.clicklearn.dk/dynamics/nav/)). ClickLearn is a tool, which is specialized at creating documentation and videos based on user scenarios. They demoed a recorder, which could record user interactions in NAV and I proposed that they would make support for generating C# code for the load test framework in their app.

By Directions EMEA 2016, ClickLearn demonstrated that they now were able to create C# scenarios based on their recordings, very cool. I will test this and create a blog post on how to use this for creating the frist stab on creating scenarios and then you can manually fix small issues afterwards.

Next blog post on performance testing will be around how to use/utilize this functionality.

### The NAVLoadTest repository

The NAVLoadTest repository has primarily been maintained by David Worthington and has some cool samples on how to do things:

-   Selecting a customer using the drop down on the customer
-   Filtering a list on a column
-   Scroll the repeater
-   and other things

I don’t think the repository is updated to NAV 2017, but a lot of the things in the repo is still good samples, and the majority of things in the API has not changed. There are however changes in the UI between NAV 2016 and NAV 2017, meaning that the user would have to do slightly different things.

### Videos

There are a few cool videos on Youtube showing how to write load tests. These videos were also created by David:

Enjoy

_**Freddy Kristiansen**_  
Technical Evangelist
