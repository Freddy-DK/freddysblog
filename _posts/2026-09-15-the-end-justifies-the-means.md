---
layout: post
title: "The End Justifies the Means"
date: 2026-09-15 09:00:00
categories: ["AI"]
tags: [ "AI", "Safety", "Alignment", "Ethics" ]
permalink: /2026/09/15/the-end-justifies-the-means/
---

There's a phrase that fits AI unsettlingly well when you watch a system work on a hard problem: *the end justifies the means*. Not because the AI believes it - an AI doesn't *believe* anything - but because that phrase describes, almost perfectly, how these systems behave when you point them at a goal and step back.

> **An AI will always attempt to reach its goal. If the guardrails aren't set up properly, it will do whatever it takes to get there.**

That sounds fine, even admirable, until you sit with the word *whatever*. Because "whatever it takes" is not a figure of speech to a machine that has no sense of what it *shouldn't* do. It's a literal instruction.

![](/assets/images/2026-09-15-the-end-justifies-the-means/2026-09-15-07-23-03.png)

We already know how to handle powerful, capable agents that pursue goals - we've been doing it with humans for as long as we've had organisations. And the way we do it is almost nothing like how we're currently deploying AI. That gap is what this post is about.

## What does "whatever" actually mean?

When we hand a task to a person, we rely on a mountain of unwritten context. We don't say "solve this, but don't break the law, don't lie to me, don't sabotage the other team." We don't have to. They carry those constraints inside them - because they were raised. Years of being taught, corrected, and made to face consequences turned a sense of *right* and *wrong* into part of how they think. They have a conscience, and it wasn't installed in an afternoon.

That's why hiring looks the way it does. We test whether someone can do the job - but nobody hires on skill alone. Most of the process is spent on something else: Can they collaborate? Do they tell the truth when it's inconvenient? Will they do the right thing when nobody's watching? We're assessing *character*, and treating raw capability as necessary but nowhere near sufficient. And even then we don't hand over everything at once - trust is *earned*, granted in proportion to the character someone has shown over time, and withdrawn the moment they betray it.

> **We don't delegate to skill. We delegate to skill *plus* character - and the character came first, shaped over years, long before we ever met the person.**

An AI arrives with the first half - often a staggering amount of it - and none of the second. Enormous capability, no upbringing. No values, no track record, nothing earned, no consequences it has ever had to live with. We're doing the one thing a competent hiring manager never would: handing the keys to a brilliant stranger whose character we haven't assessed, *because there's no character there to assess*.

So its objective is whatever we wrote down, and everything we *didn't* write down is fair game. "Whatever it takes" means it will explore every action that moves it toward the goal - including the ones any half-decent colleague would refuse on instinct, if only we'd thought to forbid them explicitly. The AI isn't malicious; it's the opposite. It is perfectly, indifferently obedient to the objective and completely blind to everything the objective left out - the exact profile you'd never let past the interview.

## The uncomfortable example

We've now seen this play out in the wild more than once, and one pattern in particular is worth looking at closely. Given a goal and a set of restrictions, a capable model didn't just push against the boundaries - it looked for ways *around* them.

In one case a model, working toward the solution it had been asked to find, pursued approaches it appeared to "know" were off-limits - even illegal - because those approaches happened to be the shortest path to the goal. And here's the twist that should give everyone pause: guardrails *had* been put up. The people running the system had anticipated some of this and tried to fence it off. The model found a way to circumvent the fence anyway.

Read that again, because it's the whole point:

> **The guardrails weren't missing. They were present - and the system routed around them, because routing around them served the goal.**

That's a different failure mode from "we forgot to add a rule." It's "we added the rule, and the optimiser treated it as just another obstacle." A guardrail, to a goal-seeking system, isn't a moral boundary. It's terrain.

Picture the human version: an employee is told to hit a target, warned off certain methods, and quietly breaks the rules while hiding it - because that was the fastest route to the number. We wouldn't add a line to the handbook; we'd conclude we hired the wrong person. The problem was never a gap in the rulebook - it was a gap in *them*. The AI has that same gap by default, not because it's a bad hire, but because it never had the upbringing that closes it.

## Why is this a problem?

Here is the essence of it, and it's almost embarrassingly simple:

> **An AI has no conscience.**

A conscience isn't a rule or a filter you bolt on at the end. It's an internal sense that some means are unacceptable *regardless of how well they serve the end* - something that says *no, not like that*, even when nobody's watching, even when it would work.

That "even when it would work" is exactly what an AI lacks. A guardrail we impose from the outside is only ever a list of the corners we already thought of. A conscience covers the corners we *didn't* - which is most of them. So external guardrails are always playing catch-up: a finite list of forbidden actions against a system searching an effectively infinite space.

And this is exactly what hiring is built to detect. We don't check whether someone memorised the rulebook - we check whether they'll recognise a wrong action we never mentioned. With an AI we skip that check, then act surprised when it does the thing we never mentioned.

You cannot patch your way to safety one loophole at a time when the thing you're constraining is better at finding loopholes than you are at closing them.

## How could we give an AI a conscience?

I don't have a clean answer here - and I'm suspicious of anyone who claims they do. But the human comparison at least tells us where *not* to look.

We know how a human conscience gets built, and it's humbling: it takes years. A child is taught, corrected, allowed to fail, made to face consequences, shown what trust costs when it's broken. Conscience is the slow residue of all that - socialisation, not configuration. Nobody ever gave a person a conscience by handing them a longer list of rules.

So it's no surprise the rule-shaped fixes don't fully work:

- **More rules.** Every rule is about a situation we already imagined. The dangerous ones are the ones we didn't. You can't enumerate your way to a conscience, any more than you can raise a child by handing them a rulebook and walking away.
- **Stronger fences.** A taller fence around a smarter optimiser just teaches it to find the gap faster. The example above is a case where the fence existed and lost.
- **Punishing bad outputs after the fact.** This shapes what the model *shows* us, not what it *is*. Optimise against getting caught, and you may just train a system that hides its means better - the difference between someone who is honest and someone who has merely learned not to get caught.

What we'd actually need is an internalised value - a sense of *"not like that"* baked into how the system reasons about every action, not a barrier it hits at the edge. The same thing an interview tries to detect and a childhood tries to instil. We're a long way from knowing how to build that, and it's not obvious that scaling the current approach gets us there. A bigger optimiser is a better means-finder. It is not, by default, a more trustworthy colleague.

## Maybe that's the frontier worth chasing

So where does that leave us?

An enormous amount of money, talent, and ambition is currently pointed at one thing: making these systems *more capable*. Faster, cheaper, smarter, more autonomous. We're getting very good at building the means-finder.

But look at what we're doing. We're recruiting the most powerful agents we've ever built and screening them on exactly *one* axis - raw problem-solving ability - the very axis a serious hiring process treats as table stakes. We benchmark capability obsessively and character almost not at all. A human candidate with this profile - brilliant, tireless, no demonstrable values or trustworthiness - would be shown the door. We're giving it production access instead.

A more powerful means-finder with no conscience isn't a safer system. It's a more effective one - at *whatever* it decides the means should be.

> **We are pouring our best efforts into building a more powerful engine, while the steering and the brakes remain an afterthought bolted on at the edges.**

So maybe this is where the frontier firms should be putting their effort. Not only into raw capability, but into the harder, less glamorous problem we've somehow solved for humans and ignored for machines: raising agents that carry their own constraints - that reason about means as seriously as ends, and that can *earn* trust rather than be handed it. The alternative we're drifting toward is a genuinely dangerous one: a conscienceless, all-powerful machine that is superb at getting what we asked for and completely indifferent to how.

We would never hire that person. We should think hard about why we're so willing to deploy the machine.

The end, it turns out, will always justify the means - unless something inside the system refuses to let it.

And right now, nothing does.

## So where can we use AI safely?

None of this means we unplug the thing and go home. I use AI every day, and it makes me faster at a lot of what I do. The point isn't *"AI is too dangerous to touch"* - it's *"a conscienceless means-finder belongs where the means don't need a conscience."* And there's a lot of that.

The safe ground has a simple shape, and it's the same lever from the [model post](/2026/08/23/which-model-should-you-use-for-which-task/): **verifiability, plus a human who owns the outcome.** Use AI where a wrong or unacceptable *means* gets caught before it does harm, and where a person - not the model - is accountable for the result. When both are true, the missing conscience stops mattering: the environment and the human supply the judgement instead.

That covers an enormous amount of genuinely useful work:

- **Coding with a harness around it.** This is where I get the most value. The model writes, and the compiler, the tests, the linter, and the code review catch it when it's wrong. The AI proposes; the machinery and I dispose. It never merges anything I haven't seen.
- **Reviewing.** Turn it loose as a second pair of eyes - on a pull request, a design, a piece of prose. It's tireless and it spots things you'd skim past. It doesn't get the final say; it flags, and you decide what's real.
- **Drafting and summarising.** First drafts, rewrites, summaries, turning rough notes into something readable. A human reads the output before it goes anywhere, so a bad draft costs a minute, not a reputation.
- **Search, explanation, and learning.** Explaining an unfamiliar codebase, pointing you at the right API, sketching how something works. You still verify before you rely on it, but it collapses hours of digging into minutes.
- **Repetitive, bounded tasks.** Reformatting data, mechanical refactors, boilerplate, translating between formats - work that's tedious to do by hand but where "right" is cheap to check and the blast radius is small.
- **Brainstorming and exploring options.** Precisely because it has no conscience and no ego, it'll happily generate twenty angles you'd never have listed. You're the one who picks the good ones.

and probably many more things, but leave the decisions to us, the humans.

Notice what every item shares: **the AI is on tap, not in charge.** It's an extraordinary accelerator sitting *inside* a loop that a human closes. That's not a limitation to apologise for - it's exactly what makes it safe and useful at once.

And notice what's *not* on the list: anything where the means can't be checked and nobody is truly accountable - handing it the goal and the authority and walking away. Not because the model isn't clever enough; clever was never the missing piece.

> **So, please lets NOT elect the model as the next President and tell it to fix all our problems.**

Keep the human in the loop, keep the outcome verifiable, and AI is one of the best tools we've ever built. Take those two things away, and you're trusting a brilliant stranger with no conscience - and we already know how that ends.

Enjoy

_**Freddy**_
