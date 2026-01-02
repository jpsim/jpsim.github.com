---
layout: post
title: "Mobile Development in the Age of AI"
date: 2026-01-02 15:00
comments: true
categories: mobile ai development product
---

After working in mobile engineering for about a decade and a half, across
multiple eras of iOS and Android tooling, languages, and architectural
approaches, I've never been more excited about the landscape than I am right
now.

When I started making apps in 2011, it was thrilling to go from an idea to
something you could physically touch and hold in your hand within hours. Today,
that same feeling exists—but the iteration cycles are now an order of magnitude
faster.

The fundamentals haven’t changed. What *has* changed is the speed at which
intent turns into working software.

## Translation Is (Basically) a Solved Problem

A lot of where AI applications go wrong today comes from anthropomorphizing
models and ascribing to them a kind of human-style intelligence. As
[Andrej Karpathy][karpathy-ghosts] puts it: *“We’re summoning ghosts, not
building animals.”*

But one category of tasks these LLM “ghosts” excel at is translation.

That’s obvious when translating natural language like English to French, but
it’s equally true for:

- TypeScript to Swift
- Jetpack Compose to SwiftUI
- Web APIs to platform SDKs
- Architectural patterns across platforms

See [this example](https://simonwillison.net/2025/Dec/15/porting-justhtml/)
where Simon Willison ported JustHTML from Python to JavaScript with Codex CLI
and GPT-5.2 in 4.5 hours.

If you operate in an environment where your web app, iOS app, and Android app
are separate codebases with baseline functional parity, AI fundamentally changes
the economics of one codebase for each platform.

Instead of rewriting features, you’re translating intent.

A web PR becomes a reference implementation. Existing mobile code becomes
executable context. Asking an agent to “take a first pass at implementing this
natively” is now a normal—and productive—workflow, especially for apps with
large CRUD-style surface areas and well-defined business logic.

## Context Beats Code

[Simon Willison][context-engineering] has written about *context engineering*:
“the art of providing all the context for the task to be plausibly solvable by
the LLM.” The limiting factor in AI-assisted development isn’t the model itself,
but how clearly the problem is framed.

That aligns closely with what actually matters in software engineering:

- Understanding the domain
- Choosing the right abstractions
- Designing the architecture
- Encoding business rules correctly
- Setting up the right conditions for QA and validating core behavior and edge cases

These are platform-agnostic, one-time costs. Once you’ve paid them, the knowledge
transfers cleanly across platforms.

This is also where the distinction between **process-driven** and
**outcome-driven** engineers becomes important. As
[Ben Werdmuller][outcome-process] put it, we’re
likely to see a real split between people who are outcome-driven and excited to
test their work with users faster, and people who are process-driven and derive
meaning primarily from the engineering itself.

LLMs disproportionately benefit outcome-driven engineers—those who know what
“good” looks like and can steer toward it. The tools reward clarity of intent,
not ceremony.

## What Mobile Development Actually Looks Like Now

In practice, most of the code I’ve written over the past twelve months hasn’t
been typed directly into Xcode or Android Studio.

A common setup today looks like this:

- A coding agent (e.g. Claude Code, Cursor, etc.) on one side of the screen
- Xcode or Android Studio on the other
- The bulk of code editing happening in the agent
- The IDE acting as a platform-native harness: breakpoints, debugger, previews,
  and deeper integrations with simulators and emulators

With a modest amount of investment, the need for an IDE as a primary input tool
almost disappears. The more you can hoist your compiler and language server out
of the IDE and into terminal-ready commands, the more you can augment the
capabilities of your coding agent.

## Native Platforms Pair Well With AI, Despite Training Gaps

A fair concern is that frontier models still have less deep knowledge of
iOS- and Android-specific APIs than they do of web or Python ecosystems.

[Matt Gallagher][swift-llms] recently wrote about the state of Swift and iOS
knowledge in frontier models at the end of 2025. He notes that *“most LLMs are
trained on data that is 2+ years old, and their Swift style often feels older
still.”*

The conclusion isn’t that models are perfect, but that modern native platforms
provide strong constraints:

- Strong static typing
- Clear compiler diagnostics
- Declarative UI frameworks
- Deterministic tooling

These constraints matter more than raw training data volume. What matters day to
day isn’t whether an LLM can one-shot a perfect solution, but whether it can
iterate quickly with tight feedback. Strong typing in Swift, Kotlin, and
TypeScript helps enormously here.

I don’t know about you, but I don’t typically write perfect code on the first try
either.

## A Brief Note on React Native

It’s absolutely possible to build excellent mobile apps using React Native or
fully native stacks.

What’s often underestimated is the organizational overhead and depth of
expertise required to do React Native well at scale. Teams that succeed tend to
invest heavily in platform infrastructure, tooling, and internal knowledge.
Shopify is a well-known example of what “doing it right” looks like.

AI shifts this tradeoff. When translation becomes cheap, the value of
lowest-common-denominator abstractions drops. Platform-idiomatic code no longer
implies slower delivery.

Side note: this is a large part of why Ramp’s mobile engineering team is much
leaner than people expect given the breadth of the apps' capabilities.

## The Centaur Model, Applied

This way of working maps closely to what Cory Doctorow calls the
[*Centaur model*][centaur]. In automation theory, “a centaur” is a person
assisted by a machine.

Humans and machines work together, each focused on what they do best:

- Humans handle judgment, architecture, domain understanding, and taste
- Machines handle repetition, translation, and acceleration

AI doesn’t remove the need for software engineers. It removes the least
interesting parts of the job and sharpens the most important ones.

## Why This Reinforces Native Teams

With AI-assisted translation:

- Separate codebases no longer incur the same maintenance costs as they once did
- Platform fidelity no longer trades off against velocity
- Engineers can build where they’re strongest and port confidently

This is exactly how we’re working today—and why Ramp is hiring [iOS engineers]
and [Android engineers].

If you care about building platform-idiomatic mobile apps, moving quickly
without sacrificing quality, and spending more time on judgment than
boilerplate, this is a particularly good moment to be doing native mobile
engineering.

## Closing

The tools changed.  
The job didn’t.

If anything, it got more interesting.

[context-engineering]: https://simonwillison.net/2025/jun/27/context-engineering/
[outcome-process]: https://simonwillison.net/2026/Jan/2/ben-werdmuller/
[swift-llms]: https://www.cocoawithlove.com/blog/llms-twelve-months-later.html
[centaur]: https://pluralistic.net/2025/12/05/pop-that-bubble/#u-washington
[karpathy-ghosts]: https://www.youtube.com/watch?v=lXUZvyajciY
[iOS engineers]: https://jobs.ashbyhq.com/ramp/4859cd5e-f2a9-44d7-81f7-8bfc0e62369f
[Android engineers]: https://jobs.ashbyhq.com/ramp/f564dcf9-9390-4a3f-896f-8047a5086040
