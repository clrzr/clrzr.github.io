---
layout: case-study
title: "Payment Intent Architecture — Case Study"
description: "How I designed a unified payment architecture around a single observation: users always need to see where the money is going before they say how much."
eyebrow: "Case Study · Payments · System Design"
headline: "Payment Intent"
headline_em: "Architecture"
pills:
  - label: "Production"
    live: true
  - label: "Idempotent by design"
  - label: "All payment types unified"
  - label: "Parallel async validation"
decisions:
  - title: "Intent creation synchronous and instant"
    detail: "The user gets a response immediately. Heavy validation begins in background while they're still deciding — not after they've committed to an amount."
  - title: "Validation pipeline is idempotent"
    detail: "Cached data: use it. Missing data: fetch and cache. Same code path regardless of timing. Same guarantees whether the pipeline ran once or three times."
  - title: "PIN + eligibility check run in parallel"
    detail: "Both checks are independent at execution. Running concurrently halves the latency at the most time-sensitive moment in the flow."
  - title: "Intent ID as canonical idempotency key"
    detail: "Every retry, log entry, and reconciliation trace references one ID. Double-execution isn't handled by defensive checks — it is structurally impossible."
  - title: "One model for all payment types"
    detail: "New payment type = new adapter, not a new flow. Frontend always deals with the same primitives. There is only ever one flow to debug."
nav_prev:
  href: "/fraud/"
  label: "Fraud Management System"
nav_next:
  href: "/kyc/"
  label: "Identity, KYC & Device Safety"
---

## What I was trying to solve

I needed a way for a mobile payment application to handle QR payments, transfers, bill payments, and contactless — all within a single unified process flow. The reason I cared about a single flow was what it would mean for the people building on top of it. If the frontend team handles different primitives for every payment type, you end up with fragmented implementation, fragmented testing, fragmented debugging. Every new payment type becomes its own special case to maintain.

What I wanted was simple: the frontend deals with the same primitives regardless of what the user is trying to do, and on the backend, every transaction is traceable through the same shape.

## What I noticed before touching architecture

Before I touched architecture, I spent time watching how users interacted with existing payment products. One pattern became impossible to ignore. The products users loved most consistently did one thing others didn't — they showed the user where the money was going *before* the user entered the amount. Before confirmation. Before any commitment at all.

Transfer, bill payment, QR scan — it didn't matter. The best products always surfaced the destination first. The ones that skipped this step were the ones users didn't trust. That wasn't a UX observation. It was a structural signal about how users think about moving money: *they need to know where it's going before they decide how much.*

If that's always true — and observation said it was — then the architecture should be built around it, not against it.

> Every payment type follows the same fundamental user behaviour: destination first, then amount. That observation became the architectural foundation.

## What I built

Every payment begins with a Payment Intent. The user submits their destination. The system resolves it and returns immediately — an intent ID and the confirmed destination name. The user sees exactly where the money is going before they touch the amount field.

That synchronous exchange is fast by design. And while the user is reading the destination name — while they're still deciding — the system is already running validation in parallel. Transaction limits from the fraud service, wallet balance, daily volume — all fetched concurrently in the background, not one after another.

By the time the user submits an amount, most of the work is already done. At execution, a final parallel check runs: PIN verification and transaction eligibility simultaneously. Both pass, execution happens. The Intent ID is the idempotency key throughout — every retry, audit log, and reconciliation trace references the same ID. Race conditions and double-execution are eliminated by design, not defensive code.

## What this means in practice

For the frontend team: every payment type looks identical. Create an intent, receive a destination name, submit the amount, confirm. The implementation is consistent, testing is consistent, and there is only one flow to trace when something goes wrong. Adding a new payment type means writing a new adapter on the backend — not redesigning the flow.

For the user: in a market where trust in digital payments is still being earned, always knowing where the money is going before committing is not a feature. It is the product.

## The principle

The Intent model works because it separates two things most payment systems conflate: the expression of user intent and the execution of that intent. Keeping those two moments distinct creates space for validation, reduces latency by moving work earlier in the flow, and gives the system a stable reference point for tracing, auditing, and idempotency. One abstraction. Every payment type. No special cases.
