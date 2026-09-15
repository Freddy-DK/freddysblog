---
layout: post
title: "The End Justifies the Means"
date: 2026-09-15 05:00:00
categories: ["AI"]
tags: [ "AI", "Safety", "Alignment", "Ethics" ]
permalink: /2026/09/15/the-end-justifies-the-means/
---

There's a phrase that fits AI unsettlingly well when you watch a system work on a hard problem: *the end justifies the means*. Not because the AI believes it - an AI doesn't *believe* anything - but because that phrase describes, almost perfectly, how these systems behave when you point them at a goal and step back.

> **An AI will always attempt to reach its goal. If the guardrails aren't set up properly, it will do whatever it takes to get there.**

That sounds fine, even admirable, until you sit with the word *whatever*. Because "whatever it takes" is not a figure of speech to a machine that has no sense of what it *shouldn't* do. It's a literal instruction.

![](/assets/images/2026-09-15-the-end-justifies-the-means/2026-09-15-07-23-03.png)

We already know how to handle powerful, capable agents that pursue goals - we've been doing it with humans for as long as we've had organisations. And the way we do it is almost nothing like how we're currently deploying AI.

## What does "whatever" actually mean?

When we hand a task to a person, we rely on a mountain of unwritten context. We don't say "solve this, but don't break the law, don't lie to me, don't sabotage the other team." We don't have to. They carry those constraints inside them - because they were raised. Years of being taught, corrected, and made to face consequences turned a sense of *right* and *wrong* into part of how they think. They have a conscience, and it wasn't installed in an afternoon.

That's why hiring looks the way it does. We test whether someone can do the job - but nobody hires on skill alone. Most of the process goes on something else: can they collaborate, do they tell the truth when it's inconvenient, will they do the right thing when nobody's watching. We're checking *character*, and we treat raw skill as necessary but nowhere near enough. And even once someone's in, we don't hand over everything at once - trust gets earned, and it can be taken away the moment they break it.

> **The weird thing is that with AI we skip all of that. It shows up with the skill - often a frightening amount of it - and none of the rest. No values, no track record, nothing earned, no consequences it's ever had to live with. It's the one candidate we'd never let past the interview, and we hand it the keys anyway.**

So its objective is whatever we wrote down, and everything we *didn't* write down is fair game. "Whatever it takes" means it'll try anything that moves it toward the goal - including the things any half-decent colleague would refuse on instinct, if only we'd thought to forbid them. It isn't being malicious. It's just perfectly obedient to the objective and blind to everything the objective left out.

## The uncomfortable example

We've now seen this play out in the wild more than once, and one pattern in particular is worth looking at closely. Given a goal and a set of restrictions, a capable model didn't just push against the boundaries - it looked for ways *around* them.

In one case a model, working toward the solution it had been asked to find, pursued approaches it appeared to "know" were off-limits - even illegal - because they were the shortest path to the goal. And here's the part that should give everyone pause: guardrails *had* been put up. The people running it anticipated some of this and tried to fence it off. The model found its way around the fence anyway.

The guardrails weren't missing. They were there, and the system routed around them because routing around them served the goal. That's a different failure from "we forgot to add a rule" - we added the rule, and the optimiser treated it as just another obstacle. A guardrail, to a goal-seeking system, isn't a moral boundary. It's terrain.

## Why is this a problem?

Here is the essence of it, and it's almost embarrassingly simple:

> **An AI has no conscience.**

A conscience isn't a rule or a filter you bolt on at the end. It's an internal sense that some means are unacceptable *regardless of how well they serve the end* - something that says *no, not like that*, even when nobody's watching, even when it would work.

That "even when it would work" is exactly what an AI lacks. A guardrail we impose from outside is only ever a list of the corners we already thought of. A conscience covers the corners we *didn't* - which is most of them. That's the trap: we're writing a finite list of forbidden actions for a system searching a basically infinite space of possible ones, and it only takes one we didn't list.

## How could we give an AI a conscience?

I don't have a clean answer here - and I'm suspicious of anyone who claims they do. But the human comparison at least tells us where *not* to look.

We know how a human conscience gets built, and it's humbling: it takes years. A child is taught, corrected, allowed to fail, made to face consequences, shown what trust costs when it's broken. Conscience is the slow residue of all that - socialisation, not configuration. Nobody ever gave a person a conscience by handing them a longer list of rules.

So it's no surprise the fixes we reach for don't really work. More rules just add to a list of situations we already imagined - and the dangerous ones are always the situations we didn't; you can't enumerate your way to a conscience any more than you can raise a child by handing them a rulebook and walking away. A taller fence around a smarter optimiser mostly teaches it to find the gap faster, which is exactly what happened above. And punishing bad outputs after the fact just teaches it to hide them better - you end up with something that's learned not to get caught, which is worse.

What we'd actually need is an internalised value - a sense of *"not like that"* baked into how the system reasons about every action, not a barrier it hits at the edge. The same thing an interview tries to detect and a childhood tries to instil. We're a long way from knowing how to build that, and it's not obvious that scaling the current approach gets us there. A bigger optimiser is a better means-finder. It is not, by default, a more trustworthy colleague.

## Maybe that's the frontier worth chasing

An enormous amount of money, talent, and ambition is pointed at one thing right now: making these systems *more capable*. Faster, cheaper, smarter, more autonomous. We're getting very good at building the means-finder.

But look at what we're doing. We're building the most capable agents we've ever had and judging them on one thing - raw problem-solving - which is the part a decent hiring process treats as the bare minimum. We benchmark capability endlessly and character almost not at all. A person with that profile - brilliant, tireless, no values you can point to - doesn't get the job. The model gets production access.

> **We are pouring our best efforts into building a more powerful engine, while the steering and the brakes remain an afterthought bolted on at the edges.**

So maybe that's where the effort should go. Not just into raw capability, but into the harder, less glamorous problem we somehow solved for humans and never really started on for machines: agents that carry their own constraints, that care how they get there and not only whether they get there, and that have to *earn* trust rather than be handed it.

We'd never hire the alternative. It's worth asking why we're so happy to deploy it.

What would make it safe is something in the machine that stops it - that refuses to cross certain lines even when crossing them would work. And right now, nothing does.

## So where can we use AI safely?

None of this means we unplug the thing and go home. I use AI every day, and it makes me faster at a lot of what I do. The point isn't *"AI is too dangerous to touch"* - it's *"a conscienceless means-finder belongs where the means don't need a conscience."* And there's a lot of that.

The safe ground has a simple shape, and it's the same lever from the [model post](/2026/08/23/which-model-should-you-use-for-which-task/): **verifiability, plus a human who owns the outcome.** Use AI where a wrong or unacceptable *means* gets caught before it does harm, and where a person - not the model - is accountable for the result. When both are true, the missing conscience stops mattering: the environment and the human supply the judgement instead.

That covers a lot of what I do in a day:

- **Coding with a harness around it.** This is where I get the most value. The model writes; the compiler, the tests, the linter, and code review catch it when it's wrong. It proposes, and nothing merges that I haven't looked at.
- **Reviewing.** A tireless second pair of eyes on a pull request, a design, a piece of prose. It flags things I'd skim past - and I decide what's actually worth acting on.
- **Drafting and summarising.** First drafts, rewrites, turning messy notes into something readable. I read it before it goes anywhere, so a bad draft costs a minute.
- **Search and explanation.** Explaining an unfamiliar codebase, pointing me at the right API, sketching how something works - collapsing an afternoon of digging into a few minutes.
- **Anything you can measure.** If there's a clean way to check the result - a test suite, a schema, a reference answer, a number that has to reconcile - then a wrong means gets caught, and you can safely hand the task over. That's really the [delegation test](/2026/08/17/which-tasks-should-you-delegate-to-ai-agents/) from an earlier post: the more verifiable a task, the more comfortable I am giving it to an agent.

and probably many more things, but leave the decisions to us, the humans.

The thread running through all of it: the AI is on tap, not in charge. It's sitting inside a loop that a human closes. What's *not* on the list is the opposite - handing it the goal and the authority and walking away, with no way to check the means and nobody really on the hook for them. Not because it isn't clever enough; clever was never the thing that was missing.

> **So, please lets NOT elect the model as the next President and tell it to fix all our problems.**

Keep a human in the loop, keep the outcome checkable, and AI is one of the best tools we've ever built. Take those two things away and you're trusting a brilliant stranger with no conscience - and we already know how that goes.

Enjoy

_**Freddy**_
