---
layout: case-study
title: "Identity, KYC & Device Safety — Case Study"
description: "Three architectural decisions that started as product and safety calls: one-device policy, async KYC, and PII isolation with a single owner."
eyebrow: "Case Study · Identity · KYC · Device Safety"
headline: "Identity, KYC"
headline_em: "& Device Safety"
pills:
  - label: "Production"
    live: true
  - label: "Async KYC pipeline"
  - label: "One-device policy"
  - label: "PII isolation"
decisions:
  - title: "One device at a time — hard enforcement"
    detail: "New device activation immediately invalidates the previous one. Full re-auth required. 24-hour transaction cap on new devices. Built from a user story, not a compliance checklist."
  - title: "Async KYC with automatic provider failover"
    detail: "Local format and duplicate check runs instantly. KYC job runs in background. Provider error triggers automatic switch to backup — the user never sees it happen."
  - title: "Partial access over hard blocking on KYC failure"
    detail: "Wrong KYC data doesn't eject the user. They can explore the app, can't transact, and see a persistent correction prompt. The system keeps the door open."
  - title: "Collect contact details before KYC resolves"
    detail: "The async flow means onboarding completes before verification finishes. Email and phone are captured regardless of KYC outcome — so provider downtime never means a user is lost silently."
  - title: "Single PII owner — hash for index, AES for storage"
    detail: "No other service queries identity data directly. Blast radius structurally contained. Searchability and security both preserved without tradeoff."
nav_prev:
  href: "/payments/"
  label: "Payment Intent Architecture"
nav_next:
  href: "/infrastructure/"
  label: "Platform Infrastructure"
---

## Three questions that drove the architecture

When I was thinking through identity, onboarding, and user data for the platform, I kept coming back to three specific questions that the standard "pass KYC, get access" model didn't answer well.

**What happens to a user's money if they lose their phone?** **What happens to a user's relationship with the product if KYC goes down during onboarding?** **What's the blast radius if something goes wrong elsewhere in the system?** Each question led somewhere interesting.

## Problem one — the lost phone scenario

This one started with a user story I couldn't stop thinking about: a user loses their main device. Someone picks it up. The app is still authenticated. What happens to their money?

If the app is still authenticated on that device, the answer is uncomfortable. So before writing a single line of architecture, I had to answer a harder question: when a user migrates to a new device, should the old one retain any access at all? The answer was no. Obviously no. But most systems at the time weren't enforcing it — the standard pattern was to let multiple devices stay active, with deactivation as an optional user action. Relying on a panicked user to remember to deactivate a lost device is not a safety mechanism. It is the absence of one.

The solution was a strict one-device policy. A mobile financial services application is only active on one device at a time. The moment a user activates on a new device, the previous one is automatically invalidated — no action required from the user, no grace period, no overlap. Migration requires full re-authentication: not just a PIN, but the kind of verification that confirms the person holding the new device is actually the account owner. On a new device, a ₦20,000 transaction cap applies for the first 24 hours. Trust is re-established gradually, not assumed.

> This was a product and safety decision before it became regulation. The CBN formally mandated exactly this pattern in March 2026. The system had been built this way for years prior — not because the regulation existed, but because the user story demanded it.

## Problem two — KYC downtime was a retention problem

KYC providers in Nigeria go down. When they do, the standard response is: user cannot proceed, come back later. Most teams treat this as an infrastructure problem. It is actually a retention problem — and the numbers are worse than they look.

A user who just downloaded the app, completed onboarding, and hit a wall at KYC has no context for what went wrong. They don't know a provider is down. They don't know it's temporary. What they know is that the app stopped working. That impression — *this app is broken* — is almost impossible to recover from. Most users will not come back.

There's also a quieter problem: because the failure is synchronous, the system never collected anything useful from the user. No email address. No phone number. No way to reach them if the issue resolved itself an hour later. The relationship ends before it begins, and the system has no record of who was lost or why.

The insight was simple: decouple onboarding from KYC synchronicity. When a user submits their identity data — BVN, NIN — the system runs two fast local checks immediately. Does this ID already exist in the system? Does it look structurally valid? Both checks run against already-indexed data and return in milliseconds. If they pass, the user moves through onboarding without waiting.

The KYC verification job runs in the background as an async process. Three outcomes are handled cleanly: if the provider succeeds, the user is fully provisioned. If the provider fails due to a system error, the job automatically switches to a backup provider and retries — the user never knows. If the user's data is wrong, they proceed with limited access and a persistent notification guiding them to correct it. When they fix it, verification runs again.

The consequence that matters most: by letting the user move through onboarding before KYC resolves, the system successfully collects their email address and phone number. If something goes wrong downstream, the system can now reach them. The door stays open. A synchronous failure loses the user silently. An async flow keeps the conversation going.

## Problem three — PII needs its own architecture

User PII — names, identity numbers, addresses, government IDs — is the most sensitive data in a financial system. The default approach spreads it around: a field here, a reference there, queried directly by whatever service needs it at the time. The blast radius of any compromise is the entire system.

I gave PII a single owner. One service. One access path. Every other service routes requests through it — no other service queries the identity database directly, no PII fields stored elsewhere. Within that service, data is hashed for indexing (searches are fast without exposing raw values) and AES-encrypted at rest. Searchability and security are not in tension. Both are preserved.

If something fails elsewhere in the system, user data is not part of the collateral. The blast radius is contained by the architecture, not managed after the fact.

## The through line

Three different problems. Three different solutions. But the same underlying instinct in each one: don't make the user pay for system failures, and don't make safety an afterthought dressed up as a feature. Device safety was a product decision before it became regulation. Async KYC was a retention decision before it was a technical pattern. PII isolation was a trust decision before it was a compliance requirement. The architecture is just what those decisions look like in code.
