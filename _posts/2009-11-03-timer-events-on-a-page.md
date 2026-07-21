---
layout: post
title: "Timer events on a page"
date: 2009-11-03 09:13:43
categories: ["Archive"]
tags: ["Add-Ins", "AL", "C#", "Client Tier", "Extensibility", "NAV 2009 SP1", "Timer"]
permalink: /2009/11/03/timer-events-on-a-page/
---

Have you ever wanted to have an event raised every 10th second on a page in the RoleTailored Client?

Wait no more – here is how you can do just that in Microsoft Dynamics NAV 2009SP1.

### A Timer control is a Non-Visual Add-In

I have seen a number of development platforms treat a Timer as a Non-Visual Add-In (including .net) – so I thought I would try to create a non-visual Add-In for NAV – and what better than create the Timer. A Timer should not be visible to the user, but it should be able to raise events.

There are different ways to create a Non-Visual control, but the most obvious method will not work.

Adding a control and setting Visible to FALSE – will cause the control to be optimized away – it will never be created.

You can however create a Non-Visual control in other ways:

-   Set the visible property to a global variable, which is false.
-   Set the size (and MinSize, MaxSize) of the Control to 0, 0.

The first approach would require you to add a variable called something like falsevar on each page you use the Timer Control – and that isn’t really what we want – so I will use the second approach.

### Well – then everything seems pretty simple – right?

Yes and No.

It is very simple to create a non-visual control which instantiates a timer and fires events – Yes, but what if the service tier opens up a modal dialog (like a CONFIRM command) – then I would suggest that we do NOT keep firing events.

For this purpose our control needs to subscribe to two application level events.

`Application.EnterThreadModal`

`Application.LeaveThreadModal`

What is my Application? Well, that is of course the RoleTailored Client. Your WinForms Control gets created as a first class citizen in the RoleTailored Client and of course you have access to the Application events as well. In fact there are all kinds of things you can do and all kinds of things you shouldn’t do.

Always bare in mind that if you start to go outside the control itself – think whether this is necessary, think future compatibility if the RoleTailored Client changes various things and remember to clean up.

For the two events above – they are pretty clear – EnterThreadModal is fired when the application enters Modal state and LeaveThreadModal is fired when the application leaves the modal state.

### Remember to clean up – your mother isn’t here!

When coding in .net you often don’t need to consider cleaning up – the garbage collector will come and clean everything up. Now that isn’t always true.

In the case of the Application Level events – when you subscribe to an event, you actually give the Application object a pointer to your object – telling it to call you whenever something happens. This in fact means that the garbage collector is not allowed to cleanup anymore – it doesn’t matter that the page is closed, your control is gone – the Application object still maintains a reference to your object and therefore it will stay.

Of course this doesn’t apply when you subscribe to events in your own control, since the object holding the reference to your object goes out of scope at the same time as yourself.

Hmmm – admitted – I am probably getting too nerdy now – but it is rather important to understand this in order to avoid memory leaks and these memory leaks will affect the RoleTailored Client – not only your Add-In.

Instead of going further into detail – the curious read can read much more about garbage collection on msdn: [Garbage Collector Basics and Performance Hints](http://msdn.microsoft.com/en-us/library/ms973837.aspx).

### Let’s look at the code

The way I have implemented the Timer control is like this

```
[ControlAddInExport("FreddyK.TimerControl")]
public class TimerControl : StringControlAddInBase, IStringControlAddInDefinition
{
EventHandler EnterThreadModal;
EventHandler LeaveThreadModal;
Timer timer = null;
int interval = 0;
int count = 0;
```

    

```
/// <summary>
/// Constructor – Setup timer and Application event subscriptions
/// </summary>
public TimerControl()
{
EnterThreadModal = new EventHandler(Application_EnterThreadModal);
LeaveThreadModal = new EventHandler(Application_LeaveThreadModal);
Application.EnterThreadModal += EnterThreadModal;
Application.LeaveThreadModal += LeaveThreadModal;
timer = new Timer();
timer.Tick += new EventHandler(timer_Tick);
}
```

    

```
/// <summary>
/// Dispose method – cleanup timer and Application event subscriptions
/// </summary>
protected override void Dispose(bool disposing)
{
base.Dispose(disposing);
if (disposing)
{
Application.EnterThreadModal -= EnterThreadModal;
Application.LeaveThreadModal -= LeaveThreadModal;
if (timer != null)
{
timer.Stop();
timer.Dispose();
timer = null;
}
}
}
```

    

```
/// <summary>
/// Event handler for Application.EnterThreadModal
/// </summary>
void Application_EnterThreadModal(object sender, EventArgs e)
{
timer.Stop();
}
```

    

```
/// <summary>
/// Event handler for Application.LeaveThreadModal
/// </summary>
void Application_LeaveThreadModal(object sender, EventArgs e)
{
if (timer.Interval != 0)
timer.Start();
}
```

    

```
/// <summary>
/// Create the native Add-In Control
/// </summary>
protected override Control CreateControl()
{
// Create a panel with the size 0,0
Panel panel = new Panel();
panel.BorderStyle = BorderStyle.None;
panel.MinimumSize = new Size(0, 0);
panel.MaximumSize = new Size(0, 0);
panel.Size = new Size(0, 0);
return panel;
}
/// <summary>
/// Timer tick handler – raise the Service Tier Add-In Event
/// </summary>
void timer_Tick(object sender, EventArgs e)
{
// Stop the timer while running the add-in Event
timer.Stop();
// Invoke event
this.RaiseControlAddInEvent(this.count++, "");
// Restart the timer
timer.Start();
}
```

    

```
/// <summary>
/// Override to specify that Caption should be omitted
/// </summary>
public override bool AllowCaptionControl
{
get
{
return false;
}
}
```

    

```
/// <summary>
/// Override to specify that value has not changed
/// </summary>
public override bool HasValueChanged
{
get
{
return false;
}
}
```

    

```
/// <summary>
/// Value for the Timer Control – the value is the number of 1/10's of a second between Tick events
/// NOTE: every event is sent from the Client to the Service Tier – meaning that this is not intended
///       for events executing more frequently than 1/10's of a second
/// </summary>
public override string Value
{
get
{
return base.Value;
}
set
{
base.Value = value;
if (!int.TryParse(value, out interval))
{
interval = 0;
}
interval = interval * 100;
if (timer != null && timer.Interval != interval)
{
timer.Interval = interval;
count = 0;
if (interval == 0)
timer.Stop();
else
timer.Start();
```

            

```
}
}
}
}
```

A couple of things to note

-   The Value is set on the Control even it doesn’t seem necessary – that is the reason for checking whether the interval has changed before doing anything.
-   We don’t really use the native control, the Panel(0,0), for anything – it is only there for the RoleTailored Client to have something to hold on to – returning null causes the RoleTailored Client to display an Add-In error.
-   I stop the timer while running the server side event. The primary reason for this is to ensure we don’t get multiple events triggered simultaneously and this causes the interval time to be applied after the event returns – not from the time the event started.
-   If you setup the Timer to trigger an event every 10 seconds – it will do so when there has been 10 seconds without any modal dialogs. If this isn’t what you want, you should setup the trigger to fire every second and look when the Add-In event Index parameter is 10.

### How to use the Control

For a test, we create a sample page like this:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/Timereventsonapage_54E5/image_thumb_5.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/Timereventsonapage_54E5/image_12.png)

with the following global variables:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/Timereventsonapage_54E5/image_thumb_1.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/Timereventsonapage_54E5/image_4.png)

and the following triggers:

```
OnOpenPage()
timer := '10';
```

```
timer – OnControlAddIn(Index : Integer;Data : Text[1024])
count := Index;
```

As you can see, the timer is set to trigger once a second and the Index in the AddIn event actually counts the number of times the trigger has been fired, so the count will be counting.

Now you might wonder – why is the Timer caption **Timer – DO NOT REMOVE**?

The reason for this is, that the RoleTailored Client doesn’t really know about the concept Non-Visual controls and as you probably know, personalization can remove everything from a page – including your timer:

[![image](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/Timereventsonapage_54E5/image_thumb_6.png "image")](https://msdnshared.blob.core.windows.net/media/TNBlogsFS/BlogFileStorage/blogs_msdn/freddyk/WindowsLiveWriter/Timereventsonapage_54E5/image_14.png)

If you remove this control – the Timer will of course stop.

You can find the Visual Studio project and the TimerTest.fob [here](http://www.freddy.dk/TimerControl.zip).

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
