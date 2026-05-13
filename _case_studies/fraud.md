---
layout: case-study
title: "Fraud Management System — Case Study"
description: "How I built a fraud system that watches continuously in the background, so the risk score is already there by the time a transaction reaches execution."
eyebrow: "Case Study · Fraud Prevention · Risk Systems"
headline: "Fraud Management"
headline_em: "System"
pills:
  - label: "Production"
    live: true
  - label: "Continuous risk profiling"
  - label: "GPS velocity detection"
  - label: "Async signal pipeline"
decisions:
  - title: "Fraud service async — never on the critical path"
    detail: "Synchronous fraud checks add latency to every transaction. Running continuously in the background means execution is a cache lookup, not a computation."
  - title: "Cached score as primary path, live fallback if cold"
    detail: "Fast by default. If the cache is cold, a direct call to the fraud service is the fallback. Either way, the payment system honours the result without override."
  - title: "Profile recalculated on every signal, not on request"
    detail: "Every incoming signal updates the profile and emits a new score downstream. Dependent services always have the most current view."
  - title: "Velocity check on location delta, not location alone"
    detail: "Distance without time is meaningless. The check is whether the distance is physically possible given elapsed time — targeting compromise, not travel."
nav_prev:
  href: "/"
  direction: "Back"
  label: "All Work"
nav_next:
  href: "/payments/"
  label: "Payment Intent Architecture"
---

## What I was trying to fix

The approach I inherited was a checkpoint model: at the moment a user tried to do something, the system checked a static list and either let them through or blocked them. It was reactive, narrow, and had a fundamental weakness — it only knew what it had been explicitly told to look for. A novel attack pattern that didn't match any existing rule passed straight through.

The second problem was timing. By the time a user reached a payment execution checkpoint, the system was seeing that action in isolation. It had no record of the login five minutes earlier, the location the intent was created from, or the device shift in between. It was making a binary decision with almost no context about what had led to that moment.

## The question I reframed

I reframed what the system needed to ask. Instead of *is this transaction suspicious?* the question became: *is this transaction suspicious for this user, right now, given everything we know about them?*

That shift — from static rules to contextual understanding — changed everything about the architecture. If the system needed to evaluate actions in context, it had to be continuously building a picture of who the user was. Their patterns. Their behaviours. Their normal. And that picture had to update at every meaningful action, not just at payment time.

> The fraud system is not a checkpoint. It is a continuously running background process that happens to inform checkpoints.

## What I built

Every meaningful action in the system emits a signal to the fraud service — not the raw data, but the signal. The fact of what happened, when, and the context around it. A user logs in: timestamp, device identifier, location. A user initiates a payment: intent creation and location at that moment. A user confirms a transaction: location again, at confirmation.

None of this puts the fraud service on the critical path of those actions. It receives signals asynchronously, processes them, and updates the user's risk profile in the background. As signals arrive, the profile recalculates in real time. The service doesn't wait to be asked — it emits the updated score downstream the moment a profile changes. The payment service receives it and caches it.

By the time a transaction reaches execution, the fraud service has already been watching. The risk score the payment system consults is not computed at the moment of execution. It's the result of everything that happened before.

## The GPS velocity check

When a payment intent is created, the device location is captured. When the user confirms the transaction, the location is captured again. The system evaluates the distance between those two points relative to the time elapsed between them.

If a user created a payment intent in Lagos and confirmed it from Abuja four minutes later, something is wrong. The check isn't about distance alone — it's about whether that distance is physically possible given the time between the two events. No human moves 500km in four minutes.

This catches a specific attack: device compromise, where an attacker on a separate device attempts to confirm a transaction the legitimate user initiated. The legitimate user is still in Lagos. The attacker, with access to the session, is somewhere else entirely. The velocity check surfaces that discrepancy as a risk signal before execution reaches the payment provider.

## The difference in practice

The difference between the original model and this one is the difference between what the system was told and what the system has observed. By execution time, this system has seen the login, processed the GPS signals, and watched the session pattern unfold. The risk score isn't a spot check — it's a continuous assessment that surfaces at the moment it's needed.

That is the difference between a fraud system that reacts and one that actually understands risk.
