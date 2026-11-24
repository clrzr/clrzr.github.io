---
layout: case-study
title: "Kuddle — Business Operating System"
description: "A business operating system that connects sales, fulfillment, delivery, and precise financial records."
eyebrow: "Product / Small Business Operations"
headline: "Kuddle"
headline_em: "Business OS"
permalink: /products/kuddle/
---

For a small team, a good sales day can become an operations problem: more orders bring more fulfillment, delivery, and coordination, leaving less time to serve customers consistently. Kuddle connects the full loop: money received, work fulfilled, and a product or service delivered.

## Build around business activity, not company size

A ledger can record a sale, but it does not coordinate the work that follows it. Kuddle's fit test is operational: a business receives money, fulfills something, and delivers a product or service. That keeps the product grounded in what a business does rather than its headcount, revenue, or industry label.

## Keep the books exact as rules change

The brief required financial records to be accurate from the start and remain accurate as accounting and tax rules change. Storing pre-interpreted values would make those changes difficult to apply consistently. Kuddle stores business facts and keeps interpretation in the service layer; financial events post through a double-entry journal, amounts use integer minor units, and inventory costs are derived from ledger entries rather than stored separately.

The trade-off is a more demanding financial model than a simple collection of editable balances. In return, the system can change how it interprets recorded facts without relying on floating-point math or rewriting the facts themselves.

## Add jurisdictions without reshaping the data

Starting with one jurisdiction would have reduced the initial design effort, but it risked tying the product to one set of business habits and rules. The product brief called for the option to enter new markets as signals emerged. Kuddle therefore stores activity and location as facts, while country-specific tax and commercial rules live in separate code.

That separation costs more up front than hard-coding the first market. It means new jurisdiction rules can be added without redesigning the stored facts. The rules engine contains 11 jurisdictions; onboarding is currently enabled only for Nigeria.

## Introduce complexity when the business needs it

Owners were already overloaded, and there was no useful way to explain the entire system during onboarding. Kuddle starts with six modules—Money, Relationships, Inventory, Projects, My Office, and Organization. Additional modules stay visible but locked until business activity shows readiness; they are not gated behind a paid tier. This keeps the first experience small, at the cost of not making every tool available immediately.

Owners meet each new capability in the context of work they are already doing, leaving onboarding and habit-building to happen over time. Insights are framed to support action, not to lecture or grade the owner.

## Let integrations record events, not own the ledger

Payment and shipping providers can fail, and real transactions do not always happen in the order the software expects. A customer might pay before the cashier enters the order; a provider can also be unavailable while the core service is handling other work.

Kuddle isolates provider integrations in specialized services that record and emit signed payment or shipping facts. The core service remains the authority for interpreting those facts and updating the ledger. This introduces coordination through asynchronous events, but integrations can keep recording activity during core-service outages. A payment that arrives before its order is recorded as an unattributed inflow until the business record catches up, rather than being discarded because the events arrived out of order.

## Enforce tenant boundaries in the database

Multi-branch businesses need fast data entry and retrieval, with permissions enforced and regressions caught before release. Application-only filters would leave too much security responsibility in service code, while an ORM would obscure query behavior and add reflection overhead.

Kuddle uses PostgreSQL row-level security, separate database connection pools, handwritten parameterized SQL compiled with sqlc, and static checks in CI. This requires more explicit query work than relying on an ORM, but moves tenant filtering to the database boundary and catches prohibited query patterns before deployment.

## Keep data sharing under the owner's control

Kuddle is built to change how businesses operate, not to give outside organizations automatic access to their financial activity. Tax or bank data is not shared automatically, and analytics requires consent. This means outside organizations do not receive business data by default; sharing stays the owner's choice, even when it could benefit the business.

## Find the source of slowdowns before optimizing

Users repeatedly described the app as laggy without knowing which action or device caused it. Existing monitoring and analytics showed that reports clustered among iPhone users on Safari, so the investigation focused on that experience instead of applying a broad, unfocused optimization.

The work changed component rendering and deferred resources until they were needed. Standard mobile Largest Contentful Paint fell from 7.8 seconds to 2.4 seconds. In the documented cold 3G test with 20× CPU throttling, it fell from 31.6 seconds to about 10 seconds.
