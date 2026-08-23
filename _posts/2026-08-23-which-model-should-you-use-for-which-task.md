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

So let me try to build a small framework for matching models to tasks - simple enough to keep in your head, but sharp enough to actually change which model you reach for.

## Why not just "use the strongest model for everything"?

This is the question that everything else hangs on, so let's take it seriously.

The strongest models are extraordinary, but they come with real costs - and I don't just mean money. They're slower. They're more expensive per call. They often "think" for longer before they answer. And when you're running thousands or millions of tasks in a loop, or inside an agent graph, those costs stop being rounding errors and start being the whole budget.

So the naive rule - *"always use the strongest model"* - fails for the same reason *"always assign your best architect"* fails. You don't have infinite architects, infinite time, or infinite money. Every task you throw at the strongest model is capacity you can't spend somewhere it actually matters.

But there's a deeper reason, and it's the key insight of this whole post:

> **For a large class of tasks, a stronger model doesn't produce a better result - it produces the same result, more slowly and more expensively, or sometimes a *worse* result because it overthinks a task that never needed the extra thinking.**

If a weaker model gets the answer right, and you can *prove* it got the answer right, then the extra intelligence of the frontier model bought you precisely nothing. The trick is knowing *which* tasks those are. And the single best predictor turns out to be one thing: **verifiability**.

And it's worth being blunt about that "worse result," because it breaks the comfortable assumption people carry around - that a stronger model is a strict superset of a weaker one, so the worst case is just "same answer, more expensive." It isn't. On simple tasks, the biggest reasoning models can actually be *strictly worse* in a few recognisable ways:

- **They overthink and second-guess.** A reasoning model can talk itself *out* of a correct first answer, piling on caveats and "actually, wait" reversals until the final answer is worse than the obvious one it started with.
- **They never finish.** Given a hard budget, a long-reasoning model can burn its entire thinking allowance exploring, then time out or get truncated mid-thought - and hand you nothing, where a cheaper model would simply have done the obvious thing and returned.
- **They over-engineer.** Handed a trivial task, a strong model often produces an elaborate, "clever" solution when a three-line one was correct - more surface area, more to review, more to break.

So "always use the strongest model" isn't just wasteful - on the easy, verifiable end of the spectrum it can actively cost you accuracy and completion. That's a sharper argument than money alone, and it's the same instinct that makes you *not* hand a five-minute mechanical job to your principal architect: they'll over-analyse it, redesign it, question the requirements, and deliver late - precisely because reasoning deeply about everything is what they do.

## The core idea: verifiability sets the floor

Here's the principle at the heart of the framework:

> **The more verifiable a task is, the weaker the model you can safely use.**

Why? Because verifiability changes what a model's mistakes cost you.

- On a **highly verifiable** task, a wrong answer gets caught - by a test, a compiler, a schema, a checksum, a second pass. A weaker model that's right 85% of the time isn't a problem, because the 15% failures are detected and retried. You can loop a cheap model against a hard check and get a strong result out the other end.
- On a **poorly verifiable** task, a wrong answer slips through. There's no test to catch it, no ground truth to compare against, and a plausible-but-wrong result looks exactly like a correct one. Here, the *only* thing standing between you and a silent mistake is the raw quality of the model - so you want the strongest one you can get.

That's the whole lever. When the environment can catch mistakes, you can afford a model that makes more of them. When nothing catches mistakes, the model's own judgment is your last line of defense, and you should buy the best judgment available.

![Diagram: verifiability on one axis, model strength on the other](/assets/images/2026-08-23-which-model-should-you-use-for-which-task/verifiability-vs-strength.png)

## The four dimensions of "which model"

Verifiability is the biggest lever, but it isn't the only one. When I'm choosing a model for a task, I run through four dimensions. Each answers one question.

| Dimension | The question it answers | What it pushes you toward |
|---|---|---|
| **Verifiability** | *Can I catch it when the model is wrong?* | Low verifiability → stronger model |
| **Task difficulty** | *How much reasoning does one attempt really need?* | Higher difficulty → stronger model |
| **Cost & latency sensitivity** | *How much do speed and price matter here?* | High volume / low latency → weaker model |
| **Consequence of a miss** | *What does one undetected error cost?* | High stakes → stronger model |

Notice that these pull against each other on purpose. Verifiability and cost push you *down* toward cheaper models; difficulty and consequence push you *up* toward stronger ones. The right choice is wherever those forces balance for the specific task in front of you.

Let me take them one at a time.

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

## The compare-with-humans version

If the four dimensions feel abstract, here's the version I actually think in - because we've been solving this exact problem with people for as long as organizations have existed.

You have a team. At the top is your **principal architect** - brilliant, expensive, in demand, and someone you can't clone. At the other end is a bright, eager **trainee** - fast, cheap, plentiful, and perfectly capable within a bounded scope, as long as their work gets checked.

The whole art of running a team is knowing *which task goes to whom.* And it turns out you already apply exactly the four dimensions above, just by instinct:

- You give the **trainee** the tasks where **a senior can easily check the result** (high verifiability), the work is **mechanical or well-defined** (low difficulty), you need **a lot of it done quickly and cheaply** (cost-sensitive), and **a caught mistake costs nothing** (low consequence). Run the numbers, format the report, draft the boilerplate, extract the data, tidy the code to the style guide.
- You give the **architect** the tasks where **nobody else can tell if it's right** (low verifiability), the problem is **genuinely hard** (high difficulty), there's **only one of it and no rush** (not cost-sensitive), and **getting it wrong is expensive or irreversible** (high consequence). The system design. The security-critical decision. The irreversible migration. The judgment call the whole project rests on.

And here's the move that ties it all together - the thing a good lead does without thinking:

> **You let the trainee do the work, and the architect check it.**

That's not a compromise - it's the best of both worlds. The trainee is fast and cheap; the architect's review is where the expensive judgment gets spent, on the small surface that actually needs it. You get most of the volume at trainee cost and most of the safety at architect quality.

This maps *directly* onto models:

- A **cheap, fast model** does the bulk of the work.
- A **strong model** (or a hard automated check, or a human) verifies it.
- The expensive intelligence is spent on *verification*, not *production* - because verifying is often far cheaper than doing, and that asymmetry is exactly what you're exploiting.

There's a second reason this split works, and it's the one we just talked about: put a mechanical task in front of your architect and they'll over-analyse it and run late - just as a frontier model overthinks a job the trainee would have finished in seconds. Matching the tier to the task doesn't only save money; it avoids handing simple work to someone whose instinct is to make it complicated.

You'd never ask your principal architect to personally rename every variable in the codebase. Don't ask your frontier model to either.

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

## Don't forget the self-hosted option

There's a whole tier I've been quietly ignoring, and for the right task it's the most interesting one of all: the model you run **yourself**.

Everything above assumed you're calling a model over an API - someone else's model, on someone else's hardware, billed per token. But for exactly the kind of task that lives at the "reach down" end - **tedious, well-described, repetitive work** with **high verifiability** and **low difficulty** - a small open-weights model running on your own hardware becomes a genuinely attractive option. And it fits the framework perfectly: the tasks a trainee can do are also the tasks a *local* trainee can do.

Think about what these tasks look like. Classifying support tickets. Extracting fields from documents. Reformatting data. Tagging, routing, normalising, translating boilerplate. They're narrow, they repeat thousands of times, correctness is easy to check, and none of them need frontier-level reasoning. That's precisely the profile where a 7B or 8B model on a machine you own does the job beautifully - and where paying frontier prices per call, forever, would be absurd.

What you get in return is **control** - and for some tasks that's the whole point, not a nice-to-have:

- **Data never leaves your walls.** For confidential data, regulated industries, or anything you simply can't send to a third party, this can be the difference between "we can automate this" and "we're not allowed to."
- **No per-token bill.** Once the hardware is paid for, running the model a million more times costs you electricity, not API fees. For high-volume, repetitive work, the economics flip hard in your favour.
- **No rate limits, no outages, no surprise deprecations.** The model doesn't change under you unless *you* change it. The provider can't retire it, throttle you, or quietly swap it for something that behaves differently.
- **Full determinism and reproducibility.** You control the weights, the version, the settings. You can pin exactly what runs, which matters enormously when you need the same input to give the same output next year.

In team terms, this is like **hiring your trainee in-house** instead of renting one from an agency by the hour. It only makes sense for the steady, repetitive, well-understood work - you'd never self-host for the rare, hard, judgment-heavy task, just as you'd bring in an outside specialist architect rather than keep one on payroll for a problem that shows up once a year. But for the bread-and-butter volume work that runs constantly, having your own model, under your own roof, on your own terms, is often the best answer of all.

The catch, of course, is that you now own the harness: the hardware, the serving stack, the updates, the monitoring. That's real work, and it only pays off past a certain volume. But for a genuinely high-volume, tedious, verifiable task, self-hosting turns a recurring API bill into a fixed asset - and hands you complete control over the agents doing your repetitive work.

## What about letting something else choose - OpenRouter and routing services

By now you might be thinking: *this is a lot of deciding.* Four dimensions, a tier per task, sometimes a different model per step - do I really have to make this call every single time? And that's a fair objection, which is where routing services come in.

Tools like [OpenRouter](https://openrouter.ai) and the various model routers sit *between* your code and the models. Instead of hard-wiring your agent to one specific model from one specific provider, you talk to the router, and it gives you access to hundreds of models - frontier, mid-tier, cheap, open-weights - through a single API and a single bill. There are two quite different things they buy you, and it's worth keeping them apart.

The first is simply **access and portability**. One integration, one key, one invoice, and you can swap the model behind a task by changing a string - no new SDK, no new contract, no new plumbing. When a better or cheaper model lands next month, you point at it and move on. Given how fast this all moves, that flexibility alone is worth a lot: you're not marrying your architecture to a provider.

The second, and more interesting, is **automatic routing** - letting the service make the tier decision *for* you. Some routers can look at a request and send it to a cheaper model when it looks easy, a stronger one when it looks hard, fall back to another provider when one is down, or optimise for price, speed, or context length. In effect, they try to automate the very "reach up / reach down" judgment this whole post is about.

So how does that fit the framework? Think of a router as an **outsourced dispatcher** for your team - the person who takes an incoming job and decides whether it goes to the trainee or the architect. That's genuinely useful, and for a lot of general-purpose traffic it's good enough. But notice what the dispatcher *doesn't* know:

- It doesn't know **how verifiable** your task is - whether you've got a hard check catching mistakes, or whether a wrong answer sails straight through unnoticed.
- It doesn't know your **consequence of a miss** - whether this feeds a regulatory filing or an internal scratch note.
- It doesn't know your **data constraints** - whether this particular payload is allowed to leave your walls at all.

Those are exactly the dimensions that made the decision *yours* in the first place. A generic router optimises for a generic notion of "good answer for the price"; it can't weigh the risk and verifiability that only you can see. So the honest place to land is this, and it comes straight back to the one lever that runs through this whole post - **verifiability**: *if a task is verifiable, let the router (or a weaker model) have it.* When a cheap check is catching every mistake, it genuinely doesn't matter whether an auto-router picked a mid-tier model or you did - the check is what guarantees correctness, so hand the tier decision away with a clear conscience. It's only on the **poorly verifiable** tasks - where a wrong answer slips through unnoticed - that the choice has to stay yours, because there the model's own judgment is the only safety net you've got.

So: **let the router handle the plumbing and all the verifiable, low-stakes traffic - and keep the high-stakes, hard-to-verify tier decisions for yourself.** Use it to reach a hundred models through one door, and to auto-route the bulk, ordinary work. Just don't outsource the judgment on the tasks you *can't* cheaply check - those are the ones you scored by hand for a reason.

There's also a control trade-off worth naming, and it's the mirror image of the self-hosted section: a router is the *opposite* end of that spectrum. Self-hosting gives you maximum control and maximum responsibility; a router gives you maximum convenience and reach, at the cost of routing your traffic (and often your data) through another intermediary. Most real setups end up somewhere in the middle - a router for the broad, low-sensitivity work, direct calls to a chosen frontier model for the hard judgment tasks, and a self-hosted model for the high-volume, confidential, repetitive grind.

## Closing thought

If you take one thing away from this post, make it this: **verifiability is the master lever.** Everything else - weaker models, self-hosting, OpenRouter, auto-routing - only becomes safe *because* a task is verifiable. When you can cheaply catch a wrong answer, the model stops being your safety net and you're free to reach down, route away, or run it locally; when you can't, the model's judgment is all you have, and you reach up. Verifiable → weak model is fine. Not verifiable → buy the best judgment you can.

The instinct to "just use the strongest model" comes from a good place - you want the best result. But it quietly assumes that intelligence is the only thing that produces good results, and it isn't. On a verifiable task, the *environment* produces the good result - the test, the check, the loop - and the model only has to be good enough to get caught when it's wrong.

That's the reframe I'd leave you with. Stop asking *"what's the smartest model?"* and start asking *"how little intelligence can this task get away with, given how well I can check it?"* Push tasks toward verifiability, staff them with the cheapest model that clears the bar, and save your frontier models - your architects - for the handful of tasks where judgment really is the product. And remember that reaching too high isn't a free insurance policy: on a simple task, an over-powered model can overthink it, over-build it, or never finish it - so the cheapest model that clears the bar is often the *best* one, not just the most economical.

You wouldn't run a whole company with nothing but principal architects. Don't run your agents that way either.

Enjoy

_**Freddy**_
