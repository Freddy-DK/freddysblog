---
layout: post
title: "Which Tasks Should You Delegate to AI Agents?"
date: 2026-08-17 09:00:00
categories: ["Freddy"]
permalink: /2026/08/17/which-tasks-should-you-delegate-to-ai-agents/
---

Asking which jobs AI will replace is probably the wrong question. A job is a bundle of dozens of tasks, held together by a title, a team, and a person who takes responsibility for the outcome. An agent doesn't take a job – it takes a task, does it, and hands the result back. So the question worth asking is:

> **Which tasks should I delegate to an AI agent – and which ones should I keep, at least for now?**

![](/assets/images/2026-08-17-which-tasks-should-you-delegate-to-ai-agents/2026-08-17-09-46-02.png)

What follows is a framework for answering that – simple enough to keep in your head, but sharp enough to actually change your decisions. It has four dimensions, each with a question behind it, a way to score them, a rule that ties them together, and three outcomes it produces.

## Why not just "use AI for everything"?

The naive approach is: try to delegate everything, and keep whatever fails. That works, sort of, but it's expensive and occasionally dangerous. Some failures are cheap and obvious – the agent writes a bad paragraph, you notice, you rewrite it. Other failures are silent and costly – the agent confidently produces a number that is wrong, nobody checks it, and it ends up in a report that drives a decision.

The reason "use AI for everything" is a bad rule is that the tasks where delegation goes wrong are exactly the tasks where you *can't easily tell it went wrong*. So we need something better than gut feeling. We need a way to look at a task **before** we delegate it and ask: is this a good candidate, a candidate with conditions, or something I shouldn't hand over yet?

That's what this framework does.

## The core idea: four dimensions

Every task can be scored along four dimensions. Each dimension answers exactly one question:

| Dimension | The question it answers | |
|---|---|---|
| **Economic attractiveness** | *Is it worth delegating?* | Value |
| **Agent feasibility** | *Can an agent realistically do it?* | Capability 
| **Verifiability** | *Can we tell whether it did it correctly?* | Observability |
| **Risk & responsibility** | *Can we safely allow it to do this?* | Safety |

A task can score brilliantly on one and terribly on another, and that's precisely why you need all four – any single one of them, on its own, will mislead you.

For each dimension I'll give the underlying question, the sub-questions that let you score it, and what a high and a low score actually look like.

## Dimension 1 - Economic attractiveness

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
| **Setup cost** | How much effort to get an agent doing it? | Bespoke integration → Point it and go |
| **Running cost** | What does each run cost to operate? | Expensive per run → Negligible |

**A high score** looks like: a frequent, time-consuming, tedious task currently done by people whose time is better spent elsewhere, where doing more of it (or doing it more consistently) would genuinely help – and where the cost to set up and run an agent is comfortably outweighed by that value.

**A low score** looks like: a rare, quick, low-effort task where the setup cost of delegation will never pay itself back. These aren't *unsuitable* for agents – they're just not *worth* the effort.

> Important: a high economic score is what makes a task **interesting**, but on its own it tells you nothing about whether you *should* delegate. Plenty of extremely attractive tasks are terrible delegation candidates because they fail one of the other three dimensions. That's the trap this dimension sets, and the next three are how you avoid it.

## Dimension 2 - Agent feasibility

**The question: *Can an agent realistically do it?***

Forget for a moment whether it's worth it or safe – can the technology, as it exists today, actually perform this task to an acceptable standard?

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

> The crucial thing about feasibility is that it is the **most time-dependent** of the four dimensions. A task that scores low today can score high in six months – not because you did anything, but because models improved, tools matured, or the context finally got connected. We'll come back to this when we talk about *"not yet."*

## Dimension 3 - Verifiability

**The question: *Can we tell whether it did it correctly?***

People consistently underrate this one – and it's often what turns an obvious "yes" into a cautious "not yet."

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

> Here's why verifiability is such a powerful gate: **an agent you can't check is an agent you have to trust blindly.** Blind trust is fine when the stakes are low and fine when the model is perfect – and it's a disaster in every case in between. If you can't verify, you're not really delegating; you're gambling.

## Dimension 4 - Risk & responsibility

**The question: *Can we safely allow it to do this?***

The final dimension is about **consequences and accountability**. Even a feasible, verifiable, economically attractive task can be one you shouldn't hand to an autonomous agent, because of what happens if it goes wrong – or because of who is answerable when it does.

Sub-questions that let you score it (here, *high* means low risk – safe to delegate):

| Sub-question | What it asks | Low → High |
|---|---|---|
| **Blast radius** | How bad is a mistake? | Sends money/deletes data → An unread draft |
| **Reversibility** | Can the action be undone? | Irreversible → Fully reversible |
| **Accountability** | Must a human stay answerable? | Legally required → No external stakes |
| **Human participation** | Is a person's judgment, relationship, or leadership itself part of the value? | Essential → Irrelevant |
| **Sensitive access** | Does it touch confidential data or critical systems? | Highly sensitive → None |
| **Failure containment** | Can one mistake cascade? | Chain reaction → Fully contained |

**A high score** (meaning *low* risk) looks like: a task whose mistakes are minor, reversible, and contained, that doesn't touch sensitive systems, and where no external party is harmed if it's wrong.

**A low score** (meaning *high* risk) looks like: a task that acts irreversibly on the outside world, touches sensitive data or critical systems, or requires a human to remain legally or ethically accountable.

> A useful way to think about this dimension: verifiability asks *"will I notice the mistake?"*, while risk asks *"how much does the mistake cost me before I can undo it?"* They're related, but distinct – and a task can pass one while failing the other.

## The gating rule

**Delegation is not about the average of the four scores.**

It's tempting to add the four scores up, take an average, and delegate anything above some line. That's exactly the wrong thing to do, because it lets a brilliant score on one dimension paper over a fatal weakness on another. Instead, look at the **weakest** dimension:

> **The weakest dimension determines the maximum level of autonomy you can safely grant.**

That turns the four scores into a simple scale:

- **Very weak on any dimension → Not yet.** Don't delegate this autonomously.
- **Medium on the weakest → Delegate with guardrails.** Hand it over, but with human review or restricted autonomy.
- **Strong across all four → Delegate autonomously.**

The average might look great; it doesn't matter. The weakest link sets the ceiling.

This is what turns four vague feelings into a decision. You're no longer asking *"does this feel like a good AI task?"* You're asking four specific questions, and the weakest answer tells you exactly how much autonomy the task can take.

## Not suitable vs. not yet suitable

The gating rule gives you something subtle and valuable: a distinction between **"not suitable"** and **"not yet suitable."**

When a task fails on a dimension, don't just reject it – ask *why*, and *whether that reason is permanent*. Because the four dimensions age very differently:

- A low **economic** score is usually stable. If the task is rare and trivial, it will probably stay rare and trivial. This is the closest thing to "not suitable, period."
- A low **feasibility** score is often temporary. Models get better. Tools mature. The context that wasn't available gets connected. Today's "the agent can't do this" quietly becomes tomorrow's "actually, now it can."
- A low **verifiability** score is frequently *fixable by you*. Add automated tests. Introduce a structured review step. Build a reconciliation check. You can often *engineer* verifiability into a task that didn't have it.
- A low **risk** score is often *fixable by design*. Add an approval gate so a human signs off before anything irreversible happens. Restrict what the agent is allowed to execute. Limit its access to sensitive systems. You can frequently shrink the blast radius until the task becomes safe.

So a failing score isn't a dead end – it's a **diagnosis**. It tells you exactly what would have to change before the task becomes agent-ready, which is what makes the framework useful in practice rather than just a way to say no.

> **NOTE:** If the task fails, decompose it and score the individual steps...

## Putting it to work

You don't need a spreadsheet for this, but if you like scoring things properly you can [download my scorecard here](/assets/files/2026-08-17-which-tasks-should-you-delegate-to-ai-agents/ai-delegation-scorecard.xlsx). In practice, though, the framework works as four questions you run through in your head whenever you're tempted to hand something to an agent – or tempted to dismiss the idea:

1. **Is it worth delegating?** (economic attractiveness)
2. **Can an agent realistically do it?** (feasibility)
3. **Can I verify the result?** (verifiability)
4. **Can I safely allow it?** (risk & responsibility)

Then apply the gate. Put the four answers together with the "not yet" thinking, and every task lands in one of three buckets:

### 1. Delegate now
All four dimensions clear the threshold. The task is worth doing, an agent can do it, you can verify the result, and the risk is acceptable. Hand it over with confidence, and spend your saved time on something an agent *can't* do.

### 2. Delegate with guardrails
The task is attractive and mostly ready, but one or more dimensions need support. This is where most of the interesting real-world tasks live. Let the weak dimension tell you where to put the guardrail. It might mean:

- **A human review step**, because verifiability is only medium – the agent does the work, a person signs off before it counts.
- **Restricted autonomy**, because risk is elevated – the agent can *propose* the action but not *execute* it, or it can act only within tightly bounded limits.
- **An approval gate on the irreversible step**, so everything up to the point of no return is automated, and a human makes the final call.

The task still gets delegated. You just don't grant *full* autonomy – you grant *bounded* autonomy, and you use the weak dimension to decide where the boundary goes.

> **Note:** When I say guardrails, I mean enforced guardrails, like permissions, approval gates, access restrictions, validation checks - not merely instructions in a prompt.

### 3. Not yet
At least one dimension is too weak to make delegation sensible or safe, and you can't (or won't) fix it right now. The right move is not to force it. Instead, note *which* dimension failed and *what would have to change*, and revisit the task when it does. "Not yet" is a parking space, not a rejection.

## Closing thought

The reason I like this framework is that it's honest about time. The technology is moving fast, and a large share of today's *"not yet"* tasks will quietly turn into *"delegate now"* tasks – some because models improve, and some because *you* improved the task by adding tests, review steps, or approval gates.

So the framework isn't a one-time verdict. It's a lens you keep coming back to. Score a task today, act on the score today – and every so often, score it again. The tasks worth delegating are a moving target, and the four questions are how you keep up.

> **Remember:** If the task isn't suited for delegation, try to decompose it and score the individual steps...

AI isn't taking your job. But it's ready to take a growing list of your tasks – and now you have a way to decide exactly which ones.

Enjoy

_**Freddy**_


## Addendum - a worked example or two

Let's run a couple of tasks through the framework to see how it feels in practice.

**Example A - Summarising a batch of support tickets into weekly themes.**

- *Economic:* High. It's done every week, it's tedious, and it currently eats a chunk of a team lead's time. ✔
- *Feasibility:* High. The tickets are text, they're accessible, and models are very good at this. ✔
- *Verifiability:* Medium-high. The summary is easy to skim against the source, and mistakes tend to be visible. ✔
- *Risk:* Low. The output is an internal document nobody acts on blindly, and it's trivially reversible. ✔

All four clear the bar → **Delegate now.**

**Example B - Drafting and sending replies to customer emails.**

- *Economic:* High. High volume, real time saved. ✔
- *Feasibility:* High. Drafting a reply is well within reach. ✔
- *Verifiability:* Medium. A wrong-but-plausible reply can look perfectly fine. ⚠
- *Risk:* Low if it *drafts*, high if it *sends* – an irreversible action reaching a real customer. ✖ (for full autonomy)

The economic and feasibility scores are excellent, but risk on the *send* step vetoes full autonomy. The fix is obvious once you see it: split the task. Let the agent draft; let a human approve the send. → **Delegate with guardrails.**

**Example C - Reconciling month-end financial figures that feed a regulatory filing.**

- *Economic:* High. ✔
- *Feasibility:* Possibly high. ✔
- *Verifiability:* Depends – if there's a hard reconciliation check, high; if not, dangerously low. ⚠
- *Risk:* High. Regulatory accountability means a human must remain answerable, and errors are costly and hard to reverse. ✖

Even with strong economics and feasibility, the risk dimension gates it. You might get to "delegate with guardrails" by having the agent *prepare* and a qualified human *own and sign*, but never to fully autonomous. → **Delegate with guardrails, at most.**

Notice how in all three cases, the *weakest* dimension – not the average – decided the outcome and the shape of the guardrails.
