---
layout: post
title: "Which Tasks Should You Delegate to AI Agents?"
date: 2026-08-17 09:00:00
categories: ["Freddy"]
permalink: /2026/08/17/which-tasks-should-you-delegate-to-ai-agents/
---

At this point, I think everybody agrees that AI is not going to replace people and take over current jobs. It is, however, going to perform **tasks**.

That shift in framing matters more than it first appears. A *job* is a bundle of dozens of different tasks, held together by a title, a team, and a person who takes responsibility for the outcome. An *agent* doesn't take a job. It takes a task, does it, and hands the result back. So the interesting question is not *"which jobs will AI do?"* but rather:

> **Which tasks should I delegate to an AI agent – and which ones should I keep, at least for now?**

This blog post describes a Framework, which is simple enough to keep in your head, but sharp enough to actually change your decisions. This post describes that framework in full: the five dimensions, the questions behind each one, how to score them, the gating rule that ties it together, and the three outcomes it produces.

## Why not just "use AI for everything"?

The naive approach is: try to delegate everything, and keep whatever fails. That works, sort of, but it's expensive and occasionally dangerous. Some failures are cheap and obvious – the agent writes a bad paragraph, you notice, you rewrite it. Other failures are silent and costly – the agent confidently produces a number that is wrong, nobody checks it, and it ends up in a report that drives a decision.

The reason "use AI for everything" is a bad rule is that the tasks where delegation goes wrong are exactly the tasks where you *can't easily tell it went wrong*. So we need something better than gut feeling. We need a way to look at a task **before** we delegate it and ask: is this a good candidate, a candidate with conditions, or something I shouldn't hand over yet?

That's what this framework does.

## The core idea: five dimensions

Every task can be scored along five dimensions. Each dimension answers exactly one question:

| Dimension | The question it answers | |
|---|---|---|
| **Economic attractiveness** | *Is it worth delegating?* | Value |
| **Agent feasibility** | *Can an agent realistically do it?* | Capability 
| **Verifiability** | *Can we tell whether it did it correctly?* | Observability |
| **Risk & responsibility** | *Can we safely allow it to do this?* | Safety |
| **Cost of delegation** | *What does it cost to set up and run?* | Effort |

A task can score brilliantly on one and terribly on another, and that's precisely why you need all five – any single one of them, on its own, will mislead you.

Below is a list of dimensions, with questions and For each dimension I'll give the underlying question, the sub-questions that let you score it, and what a high and a low score actually look like.

## Dimension 1 — Economic attractiveness

**The question: *Is it worth delegating?***

This is the dimension people usually start with, and it's the easiest to feel intuitively. If a task is rare, trivial, and takes you ten seconds, there is no point building an agent for it. If a task is frequent, tedious, and eats hours of expensive human attention every week, it's screaming to be delegated.

Sub-questions that let you score it:

| Sub-question | What it asks | Low → High |
|---|---|---|
| **Frequency** | How often does this task occur? | Once a year → Many times a day |
| **Time cost** | How long does it take a human each time? | Seconds → Hours |
| **Skill level tied up** | Whose time does it consume? | Cheap/junior → Expensive specialist |
| **Volume & scalability** | Would you do more of it if it were cheap? | Fixed, low volume → Unlocks new scale |
| **Consistency value** | Does doing it the same way every time help? | Doesn't matter → Highly valuable |

**A high score** looks like: a frequent, time-consuming, tedious task currently done by people whose time is better spent elsewhere, where doing more of it (or doing it more consistently) would genuinely help.

**A low score** looks like: a rare, quick, low-effort task where the setup cost of delegation will never pay itself back. These aren't *unsuitable* for agents – they're just not *worth* the effort.

Important: a high economic score is what makes a task **interesting**, but on its own it tells you nothing about whether you *should* delegate. Plenty of extremely attractive tasks are terrible delegation candidates because they fail one of the other three dimensions. That's the trap this dimension sets, and the next three are how you avoid it.

## Dimension 2 — Agent feasibility

**The question: *Can an agent realistically do it?***

This is the capability dimension. Forget for a moment whether it's worth it or safe – can the technology, as it exists today, actually perform this task to an acceptable standard?

Sub-questions that let you score it:

| Sub-question | What it asks | Low → High |
|---|---|---|
| **Clarity of the task** | Can it be described unambiguously? | Vague & judgment-heavy → Crisp & well-defined |
| **Availability of context** | Can the agent reach what it needs to know? | Tacit → Accessible |
| **Tool access** | Can it reach the tools to actually do the work? | No tools → Fully wired up |
| **Bounded scope** | Is the task self-contained? | Open-ended & sprawling → Narrow & bounded |
| **Tolerance for imperfection** | Must it be right every time? | Zero margin → "Usually great" is fine |

**A high score** looks like: a clearly-describable task, with all necessary context and tools available, bounded in scope, where the current generation of models genuinely performs well.

**A low score** looks like: a fuzzy, open-ended task that depends on context the agent can't reach, or requires a level of reliability the technology can't yet deliver.

The crucial thing about feasibility is that it is the **most time-dependent** of the five dimensions. A task that scores low today can score high in six months – not because you did anything, but because models improved, tools matured, or the context finally got connected. We'll come back to this when we talk about *"not yet."*

## Dimension 3 — Verifiability

**The question: *Can we tell whether it did it correctly?***

This, for me, is the dimension people most consistently underrate – and it's often the one that turns an obvious "yes" into a cautious "not yet."

Verifiability is about **observability of correctness**. After the agent finishes, how do you know whether the result is good? And more pointedly: how *expensive* is it to find that out?

Sub-questions that let you score it:

| Sub-question | What it asks | Low → High |
|---|---|---|
| **Existence of a ground truth** | Is there a correct answer to check against? | Subjective → Definite right answer |
| **Cost of checking** | How much effort is verification vs. doing it? | As costly as doing → Far cheaper |
| **Automatability of the check** | Can correctness be verified automatically? | Manual review only → Fully automatable |
| **Detectability of subtle errors** | Will a wrong answer look wrong? | Plausibly hidden → Glaringly obvious |
| **Feedback loop** | How fast do you find out it's wrong? | Much later → Immediately |

**A high score** looks like: a task with a clear notion of correctness, where checking is cheap or automatable, and where mistakes announce themselves loudly.

**A low score** looks like: a task where correctness is subjective or hidden, where verification is as much work as the task itself, and where a wrong result looks indistinguishable from a right one.

Here's why verifiability is such a powerful gate: **an agent you can't check is an agent you have to trust blindly.** Blind trust is fine when the stakes are low and fine when the model is perfect – and it's a disaster in every case in between. If you can't verify, you're not really delegating; you're gambling.

## Dimension 4 — Risk & responsibility

**The question: *Can we safely allow it to do this?***

The final dimension is about **consequences and accountability**. Even a feasible, verifiable, economically attractive task can be one you shouldn't hand to an autonomous agent, because of what happens if it goes wrong – or because of who is answerable when it does.

Sub-questions that let you score it (here, *high* means low risk – safe to delegate):

| Sub-question | What it asks | Low → High |
|---|---|---|
| **Blast radius** | How bad is a mistake? | Sends money/deletes data → An unread draft |
| **Reversibility** | Can the action be undone? | Irreversible → Fully reversible |
| **Accountability** | Must a human stay answerable? | Legally required → No external stakes |
| **Sensitive access** | Does it touch confidential data or critical systems? | Highly sensitive → None |
| **Failure containment** | Can one mistake cascade? | Chain reaction → Fully contained |

**A high score** (meaning *low* risk) looks like: a task whose mistakes are minor, reversible, and contained, that doesn't touch sensitive systems, and where no external party is harmed if it's wrong.

**A low score** (meaning *high* risk) looks like: a task that acts irreversibly on the outside world, touches sensitive data or critical systems, or requires a human to remain legally or ethically accountable.

A useful way to think about this dimension: verifiability asks *"will I notice the mistake?"*, while risk asks *"how much does the mistake cost me before I can undo it?"* They're related, but distinct – and a task can pass one while failing the other.

## Dimension 5 — Cost of delegation

**The question: *What does it cost to set up and run?***

The first dimension asked whether the task is *worth* delegating – the value side. This last one is the other half of that equation: the *cost* side. Even a task that scores well everywhere else can be a bad deal if handing it to an agent is expensive, fiddly, or fragile to keep running.

It's easy to forget this dimension because the demo always looks free. But real delegation has a price: the effort to describe the task and wire up the context, the tokens or API calls it burns every time it runs, and the ongoing work to keep it maintained as tools and data change around it.

Sub-questions that let you score it (here, *high* means low cost – cheap to delegate):

| Sub-question | What it asks | Low → High |
|---|---|---|
| **Setup effort** | How much work to get it running? | Bespoke integration → Point it and go |
| **Running cost** | What does each run consume? | Expensive per run → Negligible |
| **Maintenance burden** | How much upkeep as things change? | Constant babysitting → Fire and forget |
| **Oversight cost** | How much human attention does it still need? | Heavy supervision → Fully hands-off |
| **Payback** | How does the cost compare to the value returned? | Never pays back → Pays for itself fast |

**A high score** (meaning *low* cost) looks like: a task that is quick to set up, cheap to run, and largely maintains itself – so almost all of the economic value from Dimension 1 actually reaches your bottom line.

**A low score** (meaning *high* cost) looks like: a task that needs bespoke integration, burns expensive compute on every run, or demands constant maintenance and oversight – eating up the value that made it attractive in the first place.

Think of Dimensions 1 and 5 as the two halves of a business case: **economic attractiveness is the benefit, cost of delegation is the price.** A task only makes sense when the benefit clearly outweighs the price – and a task that looks fabulous on value can still be a *"not yet"* simply because it's currently too expensive or too fragile to run.

## The gating rule — the sharp part of the framework

Here is the insight that makes the framework actually useful rather than just a nice diagram.

**Delegation is not about the average of the five scores.**

It's tempting to add the five scores up, take an average, and delegate anything above some line. That's exactly the wrong thing to do, because it lets a brilliant score on one dimension paper over a fatal weakness on another.

Instead, treat each dimension as a **gate**:

> **If any one of the five dimensions is below a minimum threshold, the task is not ready for autonomous delegation – no matter how strong the other four are.**

A task can be wildly attractive economically, clearly feasible, and low-risk – but if it's barely verifiable, the answer is still *"not yet."* A single weak dimension vetoes the task. The average might look great; it doesn't matter. The weakest link decides.

This is what turns five vague feelings into a decision. You're no longer asking *"does this feel like a good AI task?"* You're asking five specific questions, and any single firm "no" is enough to stop you.

## Not suitable vs. not yet suitable

The gating rule gives you something subtle and valuable: a distinction between **"not suitable"** and **"not yet suitable."**

When a task fails on a dimension, don't just reject it – ask *why*, and *whether that reason is permanent*. Because the five dimensions age very differently:

- A low **economic** score is usually stable. If the task is rare and trivial, it will probably stay rare and trivial. This is the closest thing to "not suitable, period."
- A low **feasibility** score is often temporary. Models get better. Tools mature. The context that wasn't available gets connected. Today's "the agent can't do this" quietly becomes tomorrow's "actually, now it can."
- A low **verifiability** score is frequently *fixable by you*. Add automated tests. Introduce a structured review step. Build a reconciliation check. You can often *engineer* verifiability into a task that didn't have it.
- A low **risk** score is often *fixable by design*. Add an approval gate so a human signs off before anything irreversible happens. Restrict what the agent is allowed to execute. Limit its access to sensitive systems. You can frequently shrink the blast radius until the task becomes safe.
- A low **cost** score often improves on its own. Compute gets cheaper, tooling and templates reduce setup effort, and the integration you build once keeps paying off – so a task that's too expensive to delegate today can quietly become worth it.

So a failing score isn't a dead end – it's a **diagnosis**. It tells you exactly what would have to change before the task becomes agent-ready. That reframing is, honestly, the most practically useful part of the whole framework.

## The three outcomes

Put the gating rule and the "not yet" thinking together, and every task lands in one of three buckets:

### 1. Delegate now
All five dimensions clear the threshold. The task is worth doing, an agent can do it, you can verify the result, the risk is acceptable, and it's cheap enough to be worth it. Hand it over with confidence, and spend your saved time on something an agent *can't* do.

### 2. Delegate with guardrails
The task is attractive and mostly ready, but one or more dimensions need support. This is where most of the interesting real-world tasks live. Guardrails might mean:

- **A human review step**, because verifiability is only medium – the agent does the work, a person signs off before it counts.
- **Restricted autonomy**, because risk is elevated – the agent can *propose* the action but not *execute* it, or it can act only within tightly bounded limits.
- **An approval gate on the irreversible step**, so everything up to the point of no return is automated, and a human makes the final call.

The task still gets delegated. You just don't grant *full* autonomy – you grant *bounded* autonomy, and you use the weak dimension to decide where the boundary goes.

### 3. Not yet
At least one dimension is too weak to make delegation sensible or safe, and you can't (or won't) fix it right now. The right move is not to force it. Instead, note *which* dimension failed and *what would have to change*, and revisit the task when it does. "Not yet" is a parking space, not a rejection.

## A worked example or two

Let's run a couple of tasks through the framework to see how it feels in practice.

**Example A — Summarising a batch of support tickets into weekly themes.**

- *Economic:* High. It's done every week, it's tedious, and it currently eats a chunk of a team lead's time. ✔
- *Feasibility:* High. The tickets are text, they're accessible, and models are very good at this. ✔
- *Verifiability:* Medium-high. The summary is easy to skim against the source, and mistakes tend to be visible. ✔
- *Risk:* Low. The output is an internal document nobody acts on blindly, and it's trivially reversible. ✔
- *Cost:* Low. Little setup, cheap to run, nothing to maintain. ✔

All five clear the bar → **Delegate now.**

**Example B — Drafting and sending replies to customer emails.**

- *Economic:* High. High volume, real time saved. ✔
- *Feasibility:* High. Drafting a reply is well within reach. ✔
- *Verifiability:* Medium. A wrong-but-plausible reply can look perfectly fine. ⚠
- *Risk:* Low if it *drafts*, high if it *sends* – an irreversible action reaching a real customer. ✖ (for full autonomy)
- *Cost:* Low–medium. Cheap to run, though the human review step adds some ongoing oversight cost. ✔

The economic and feasibility scores are excellent, but risk on the *send* step vetoes full autonomy. The fix is obvious once you see it: split the task. Let the agent draft; let a human approve the send. → **Delegate with guardrails.**

**Example C — Reconciling month-end financial figures that feed a regulatory filing.**

- *Economic:* High. ✔
- *Feasibility:* Possibly high. ✔
- *Verifiability:* Depends – if there's a hard reconciliation check, high; if not, dangerously low. ⚠
- *Risk:* High. Regulatory accountability means a human must remain answerable, and errors are costly and hard to reverse. ✖
- *Cost:* Medium. Integrating with the finance systems takes real work, but it runs monthly and pays back. ✔

Even with strong economics and feasibility, the risk dimension gates it. You might get to "delegate with guardrails" by having the agent *prepare* and a qualified human *own and sign*, but never to fully autonomous. → **Delegate with guardrails, at most.**

Notice how in all three cases, the *weakest* dimension – not the average – decided the outcome and the shape of the guardrails.

## Putting it to work

You don't need a spreadsheet for this, though you certainly can build one. In practice, the framework works as five questions you run through in your head whenever you're tempted to hand something to an agent – or tempted to dismiss the idea:

1. **Is it worth delegating?** (economic attractiveness)
2. **Can an agent realistically do it?** (feasibility)
3. **Can I verify the result?** (verifiability)
4. **Can I safely allow it?** (risk & responsibility)
5. **Is it cheap enough to be worth it?** (cost of delegation)

Then apply the gate: if all five clear the bar, **delegate now**. If it's attractive but one dimension is weak, **delegate with guardrails**, and let the weak dimension tell you where to put the guardrail. If something is simply too weak today, don't force it – note *what would have to change*, and mark it **not yet**.

## Closing thought

The reason I like this framework is that it's honest about time. The technology is moving fast, and a large share of today's *"not yet"* tasks will quietly turn into *"delegate now"* tasks – some because models improve, and some because *you* improved the task by adding tests, review steps, or approval gates.

So the framework isn't a one-time verdict. It's a lens you keep coming back to. Score a task today, act on the score today – and every so often, score it again. The tasks worth delegating are a moving target, and the five questions are how you keep up.

AI isn't taking your job. But it's ready to take a growing list of your tasks – and now you have a way to decide exactly which ones.
