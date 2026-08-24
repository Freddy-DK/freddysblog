---
layout: post
title: "Which AI Model Should You Use for Which Task?"
date: 2026-08-23 09:00:00
categories: ["Freddy"]
tags: [ "AI", "Development" ]
permalink: /2026/08/23/which-model-should-you-use-for-which-task/
---

In an [earlier post](/2026/08/17/which-tasks-should-you-delegate-to-ai-agents/) I wrote about *which tasks* you should delegate to an AI agent. This one is about the very next question you run into the moment you decide to delegate:

> **Once I've decided to hand a task to an agent - which model should I put behind it?**

It's tempting to answer that with a single word: the best one. Reach for the biggest, smartest, most expensive frontier model, point it at everything, and stop thinking. But that's a bit like insisting your most senior architect personally handles every task in the building - from designing the system down to renaming a variable. It works, technically. It's also slow, wildly expensive, and a waste of the thing that makes that architect valuable.

![](/assets/images/2026-08-23-which-model-should-you-use-for-which-task/2026-08-23-22-08-04.png)

So let me try to build a small framework for matching models to tasks - simple enough to keep in your head, but sharp enough to actually change which model you reach for.

## Why not just "use the strongest model for everything"?

The strongest models are extraordinary, but they cost you - and not just in money. They're slower, more expensive per call, and they often "think" for longer before answering. Run thousands or millions of tasks in a loop, and those costs stop being rounding errors and become the whole budget.

So *"always use the strongest model"* fails for the same reason *"always assign your best architect"* fails: you don't have infinite architects, infinite time, or infinite money. Every task you throw at the strongest model is capacity you can't spend where it matters.

But there's a deeper reason:

> **For a large class of tasks, a stronger model doesn't produce a better result - just the same result, slower and more expensively. Sometimes it produces a *worse* one, because it overthinks a task that never needed the thinking.**

If a weaker model gets the answer right, and you can *prove* it did, the frontier model's extra intelligence bought you nothing. The trick is knowing *which* tasks those are - and the best predictor is one thing: **verifiability**.

That "worse result" is worth dwelling on, because it kills a comfortable assumption - that a stronger model is a strict superset of a weaker one, so the worst case is just "same answer, more expensive." It isn't. On simple tasks, the biggest reasoning models can be *strictly worse*:

- **They overthink and second-guess.** A reasoning model can talk itself *out* of a correct first answer, piling on caveats and "actually, wait" reversals until the final answer is worse than the obvious one it started with.
- **They never finish.** Given a hard budget, a long-reasoning model can burn its entire thinking allowance exploring, then time out or get truncated mid-thought - and hand you nothing, where a cheaper model would simply have done the obvious thing and returned.
- **They over-engineer.** Handed a trivial task, a strong model often produces an elaborate, "clever" solution when a three-line one was correct - more surface area, more to review, more to break.

So the strongest model isn't just wasteful on easy work - it can actively cost you accuracy and completion. It's the same instinct that stops you handing a five-minute mechanical job to your principal architect: they'll over-analyse it, redesign it, question the requirements, and deliver late - because reasoning deeply about everything is what they do.

## The core idea: verifiability sets the floor

The principle at the heart of the framework is simple:

> **The more verifiable a task is, the weaker the model you can safely use.**

Verifiability changes what a model's mistakes cost you.

- On a **highly verifiable** task, a wrong answer gets caught - by a test, a compiler, a schema, a checksum, a second pass. A weaker model that's right 85% of the time isn't a problem, because the 15% failures are detected and retried. You can loop a cheap model against a hard check and get a strong result out the other end.
- On a **poorly verifiable** task, a wrong answer slips through. There's no test to catch it, no ground truth to compare against, and a plausible-but-wrong result looks exactly like a correct one. Here, the *only* thing standing between you and a silent mistake is the raw quality of the model - so you want the strongest one you can get.

That's the whole lever. When the environment catches mistakes, you can afford a model that makes more of them. When nothing does, the model's judgment is your last line of defense - so buy the best you can.

When it comes to coding, verifiability has a very concrete shape: it's your automated tests, your compiler, your cops/linter, and your code reviews - both the AI ones and the human ones. That's the machinery that catches a wrong answer before it ships. If you hand AI the keys to the bus with none of that in place - no tests, no review, no way to tell whether it's driving us off a cliff - then I want to get off. Most of that machinery is really the *harness* - more that later.)

## The four dimensions of "which model"

Verifiability is the biggest lever, but it isn't the only one. When I'm choosing a model for a task, I run through four dimensions. Each answers one question.

| Dimension | The question it answers | What it pushes you toward |
|---|---|---|
| **Verifiability** | *Can I catch it when the model is wrong?* | Low verifiability → stronger model |
| **Task difficulty** | *How much reasoning does one attempt really need?* | Higher difficulty → stronger model |
| **Cost & latency sensitivity** | *How much do speed and price matter here?* | High volume / low latency → weaker model |
| **Consequence of a miss** | *What does one undetected error cost?* | High stakes → stronger model |

Notice that these pull against each other on purpose. Verifiability and cost push you *down* toward cheaper models; difficulty and consequence push you *up* toward stronger ones. The right choice is wherever those forces balance for the specific task in front of you.

### Dimension 1 - Verifiability

We just covered this, but here's how to score it. Ask: *after the model answers, how cheaply and reliably can I tell whether it's right?*

- **High:** There's a test, a compiler, a schema, a checksum, a reference answer, or a second cheap model that can check it. Errors announce themselves.
- **Low:** Correctness is subjective, hidden, or as expensive to check as the task was to do. A wrong answer looks just like a right one.

The more you can push a task toward "high," the weaker the model you can get away with - because the environment, not the model, is guaranteeing correctness.

### Dimension 2 - Task difficulty

Some tasks are genuinely hard in a single shot - deep multi-step reasoning, subtle trade-offs, novel problems with no template to follow. Others are mechanical - reformatting, extracting, classifying, translating, following a well-worn pattern.

- **High difficulty:** Needs real reasoning to get right even once. Reach up.
- **Low difficulty:** A well-trodden, mechanical transformation. Reach down.

Be honest here, because this is where people overspend the most. A task can *feel* important without being *difficult*. Extracting fields from an invoice matters, but it isn't hard - and a small model does it beautifully.

### Dimension 3 - Cost & latency sensitivity

How much do price and speed actually matter for *this* task?

- **Very sensitive:** It runs at high volume, in a tight loop, or in front of a user waiting for a response. Every millisecond and every cent is multiplied by scale. Reach down.
- **Not sensitive:** It runs once a day, in the background, and nobody's waiting. You can afford to reach up "just in case."

A frontier model in an interactive loop that fires thousands of times an hour will bankrupt you and frustrate your users. The same model running one nightly analysis is completely fine.

### Dimension 4 - Consequence of a miss

If an error *does* slip through - past whatever verification you have - how bad is it?

- **High consequence:** It feeds a decision, reaches a customer, moves money, or is hard to reverse. Buy the best judgment you can.
- **Low consequence:** It's an internal draft, a suggestion, one of many candidates that gets filtered later. A weaker model is fine.

This dimension is the partner of verifiability. Verifiability asks *"will I catch the mistake?"*; consequence asks *"how much does it hurt if I don't?"* A task can be low on one and high on the other, and you need both to choose well.

## Compare-with-humans

If the four dimensions feel abstract, here's the version I actually think in - because we've solved this exact problem with people forever.

You have a team. At the top, your **principal architect** - brilliant, expensive, in demand, impossible to clone. At the other end, a bright, eager **trainee** - fast, cheap, plentiful, and perfectly capable within a bounded scope, as long as someone checks the work.

The art of running a team is knowing *which task goes to whom* - and you already apply the four dimensions above by instinct:

- You give the **trainee** the tasks where **a senior can easily check the result** (high verifiability), the work is **mechanical or well-defined** (low difficulty), you need **a lot of it done quickly and cheaply** (cost-sensitive), and **a caught mistake costs nothing** (low consequence). Run the numbers, format the report, draft the boilerplate, extract the data, tidy the code to the style guide.
- You give the **architect** the tasks where **nobody else can tell if it's right** (low verifiability), the problem is **genuinely hard** (high difficulty), there's **only one of it and no rush** (not cost-sensitive), and **getting it wrong is expensive or irreversible** (high consequence). The system design. The security-critical decision. The irreversible migration. The judgment call the whole project rests on.

And here's the move that ties it all together - the thing a good lead does without thinking:

> **You let the trainee do the work, and the architect check it.**

That's not a compromise - it's the best of both worlds. The trainee is fast and cheap; the architect's review is where the expensive judgment gets spent, on the small surface that actually needs it. You get most of the volume at trainee cost and most of the safety at architect quality.

This maps *directly* onto models:

- A **cheap, fast model** does the bulk of the work.
- A **strong model** (or a hard automated check, or a human) verifies it.
- The expensive intelligence is spent on *verification*, not *production* - because verifying is often far cheaper than doing, and that asymmetry is exactly what you're exploiting.

And it works the other way too: matching the tier to the task doesn't only save money, it keeps simple work away from someone - or something - whose instinct is to make it complicated.

## Putting it to work

So the next time you're wiring a model behind a task, don't start with "which is the best model." Start with four questions:

1. **Can I catch it when it's wrong?** (verifiability)
2. **How hard is this task, really?** (difficulty)
3. **How much do speed and cost matter here?** (cost & latency)
4. **What does an undetected miss cost me?** (consequence)

Then let the answers point you to a tier:

### Reach *down* to a smaller, cheaper, faster model when...
The task is **highly verifiable**, **mechanical**, **runs at volume or in front of a waiting user**, and a **caught mistake is cheap**. This is a huge share of real agent work - classification, extraction, formatting, routing, simple transformations, first drafts that will be checked anyway. Put a small model here and loop it against a good check. This is your trainee, and they're excellent.

### Reach *up* to a strong frontier model when...
The task has **no cheap way to verify it**, demands **real reasoning**, runs **rarely enough that cost doesn't dominate**, and an **undetected error is expensive or irreversible**. System design, thorny debugging, high-stakes judgment, the final review nobody else can do. This is your architect - use them where their judgment is the actual product.

### Split the task when the dimensions disagree
Most interesting tasks aren't uniformly hard or uniformly easy - and this is the most valuable move of all. When a task scores "cheap" on some dimensions and "expensive" on others, **decompose it** and put a different model behind each piece:

- A **cheap model produces**, a **strong model (or a hard check) verifies**.
- A **cheap model handles the 90% common case**, and **escalates the hard 10%** to a stronger one.
- A **strong model plans** the approach once, and a **swarm of cheap models executes** the well-defined steps.

That last pattern is exactly the human team again: the architect decides *what* to build and *how*, and the trainees build it under checks. You're not choosing *one* model for the task - you're staffing the task with the right mix.

## The harness changes the answer

So far I've written as if "which model" is a question about the *model*. It isn't, quite. It's a question about the model **plus the scaffolding around it** - the tools it can call, the retries when it fails, the schema it must fill, the checks that gate its output, the loop that lets it try again. That scaffolding has a name: the **harness**, and it moves the answer more than people expect.

> **A good harness manufactures verifiability the raw task didn't have - and every bit of verifiability you add lets you reach for a weaker model.**

Look at what a harness does. It forces structured output, so a parser catches a malformed answer instead of a human. It hands the model a tool - a calculator, a compiler, a database query - so it *looks things up* instead of guessing. It runs the answer through a test, a linter, or a second cheap model, and loops back on failure. Every one of those takes a job off the model and gives it to the environment.

That's why the same task can need a frontier model with a bare harness and a small one with a rich harness. Ask a model to "compute the invoice total" as free text and it had better be good at arithmetic; give it a calculator and a schema to fill, and a much weaker model nails it - because the hard part no longer happens in the model's head.

In team terms, the harness is the **bench you put your trainee at**: a checklist, the right tools, a test to run before handing work in, a senior who reviews the output. A trainee with all that beats a smarter one dropped into an empty room. You're not just hiring talent - you're building the environment that makes ordinary talent reliable.

### Skills: giving the model a runbook

If the harness is the *environment*, a **skill** is the *runbook* - a reusable, pre-written description of how to do a kind of task well: the steps, the constraints, the tools to use, the gotchas, an example of good output. Instead of hoping the model reasons its way to the right approach every time, you hand it the approach.

That matters for the same reason the harness does: **a skill moves difficulty out of the model and into something you wrote down once.** A task that needs real reasoning when the model has to invent the method becomes mechanical once the method is spelled out - and "mechanical, follow-the-pattern" is exactly the profile that lets you reach *down* a tier.

It's the oldest management trick there is: you don't hand your trainee a hard problem and hope, you hand them the runbook. A well-written skill turns a task that *would* have needed your architect's judgment into one your trainee can run reliably - and that judgment was spent *once*, when the skill was written, not on every run.

So skills and harnesses sit right next to verifiability in the framework:

- A **skill lowers difficulty** - the method is given, not discovered.
- A **harness raises verifiability** - structure, tools, and checks catch what the model gets wrong.

Both let you reach down. Which reframes the question one last time: not *"how little intelligence can this task get away with?"* but *"how little, once I've written the skill and built the harness?"* Often the highest-leverage move isn't a bigger model at all - it's better scaffolding around a cheaper one. That pays off on every call, forever; a bigger model just charges you more on every call, forever.

## Don't forget the self-hosted option

There's a whole tier I've been ignoring, and for the right task it's the most interesting of all: the model you run **yourself**.

Everything above assumed you call a model over an API - someone else's model, on someone else's hardware, billed per token. But for the work at the "reach down" end - **tedious, repetitive, well-described** tasks with **high verifiability** and **low difficulty** - a small open-weights model on your own hardware becomes genuinely attractive. The tasks a trainee can do are also the tasks a *local* trainee can do.

You know the shape of these tasks: classifying support tickets, extracting fields from documents, reformatting data, tagging, routing, translating boilerplate. Narrow, repeated thousands of times, easy to check, no frontier reasoning required. That's exactly where a 7B or 8B model on a machine you own shines - and where paying frontier prices per call, forever, would be absurd.

What you get in return is **control** - and for some tasks that's the whole point, not a nice-to-have:

- **Data never leaves your walls.** For confidential data, regulated industries, or anything you simply can't send to a third party, this can be the difference between "we can automate this" and "we're not allowed to."
- **No per-token bill.** Once the hardware is paid for, running the model a million more times costs you electricity, not API fees. For high-volume, repetitive work, the economics flip hard in your favour.
- **No rate limits, no outages, no surprise deprecations.** The model doesn't change under you unless *you* change it. The provider can't retire it, throttle you, or quietly swap it for something that behaves differently.
- **Full determinism and reproducibility.** You control the weights, the version, the settings. You can pin exactly what runs, which matters enormously when you need the same input to give the same output next year.

That determinism point deserves a moment, because we've all felt the opposite. You know the feeling: the model that was brilliant yesterday feels *off* today - a little dumber, a little more stubborn, like it got out of the wrong side of the bed. Sometimes that's just you noticing different failures; but sometimes it's real, because the provider tweaked something, rerouted you to different hardware, or quietly rolled out a new version under the same name. When you own the weights, that whole category of "why is it having a bad day?" simply goes away - the model behaves exactly the same today as it did yesterday, because nothing changed unless *you* changed it.

In team terms, this is **hiring your trainee in-house** instead of renting one by the hour. It only makes sense for steady, repetitive, well-understood work - you'd never self-host the rare, hard, judgment-heavy task, just as you'd bring in an outside specialist rather than keep one on payroll for a problem that shows up once a year. But for the bread-and-butter volume that runs constantly, your own model under your own roof is often the best answer of all.

The catch is that you now own the serving stack: the hardware, the updates, the monitoring. That's real work, and it only pays off past a certain volume. But above that volume, self-hosting turns a recurring API bill into a fixed asset - and hands you complete control.

## What about letting something else choose - OpenRouter and routing services

By now you might be thinking: *this is a lot of deciding.* Four dimensions, a tier per task, sometimes a different model per step - every single time? Fair objection. That's where routing services come in.

Tools like [OpenRouter](https://openrouter.ai) sit *between* your code and the models. Instead of hard-wiring your agent to one model from one provider, you talk to the router and get hundreds of models - frontier, mid-tier, cheap, open-weights - through a single API and a single bill. They buy you two quite different things.

The first is **access and portability**. One integration, one key, one invoice, and you swap the model behind a task by changing a string - no new SDK, no new plumbing. When a better or cheaper model lands next month, you point at it and move on. Given how fast this moves, that flexibility alone is worth a lot.

The second, more interesting, is **automatic routing** - letting the service make the tier decision *for* you: a cheaper model when the request looks easy, a stronger one when it looks hard, a fallback when a provider is down. In effect, it automates the "reach up / reach down" judgment this whole post is about.

Think of a router as an **outsourced dispatcher**: the person who takes an incoming job and decides trainee or architect. Useful, and for general-purpose traffic often good enough. But the dispatcher doesn't know:

- how **verifiable** your task is - whether a check catches mistakes, or a wrong answer sails through unnoticed.
- your **consequence of a miss** - regulatory filing or internal scratch note.
- your **data constraints** - whether this payload is allowed to leave your walls at all.

Those are the dimensions that made the decision *yours* in the first place, and it comes back to the one lever running through this post - **verifiability**: *if a task is verifiable, let the router have it.* When a cheap check catches every mistake, it doesn't matter whether the router or you picked the tier - the check guarantees correctness. It's only on the **poorly verifiable** tasks that the choice has to stay yours, because there the model's judgment is the only safety net you've got.

So **let the router handle the plumbing and the verifiable, low-stakes traffic - and keep the high-stakes, hard-to-verify decisions for yourself.** And note the control trade-off: self-hosting is maximum control and maximum responsibility; a router is maximum convenience and reach, at the cost of routing your traffic (and often your data) through an intermediary. Most real setups land in the middle - a router for the broad, low-sensitivity work, direct calls to a chosen frontier model for the hard judgment, and a self-hosted model for the high-volume, confidential grind.

## Closing thought

If you take one thing away from this post, make it this: **verifiability is the master lever.** Everything else - weaker models, skills, harnesses, self-hosting, auto-routing - only becomes safe *because* a task is verifiable. When you can cheaply catch a wrong answer, you're free to reach down, route away, or run it locally. When you can't, the model's judgment is all you have, so you reach up. Verifiable → a weak model is fine. Not verifiable → buy the best judgment you can.

And if you write code for a living, that's not abstract: your tests, your compiler, and your reviews *are* the verifiability. Build that machinery first, and you can safely let a cheaper model do more.

So stop asking *"what's the smartest model?"* and start asking *"how little intelligence can this task get away with, given how well I can check it - and how good a skill and harness I've built around it?"* On a verifiable task it's the *environment* that produces the result - the test, the check, the loop - and the model only has to be good enough to get caught when it's wrong. Staff each task with the cheapest model that clears that bar, and save your architects for the handful where judgment really is the product.

Enjoy

_**Freddy**_
