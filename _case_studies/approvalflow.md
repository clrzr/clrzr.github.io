---
layout: case-study
title: "ApprovalFlow — Document Approval"
description: "A document workflow shaped around feedback, role-based access, and institution-specific data separation."
eyebrow: "Product / Workflow Design"
headline: "Approval"
headline_em: "Flow"
permalink: /products/approvalflow/
---

Built for a government institution, ApprovalFlow treats document approval as a process. Requests move through defined review and approval steps, record who acted and when, and can add more review when needed.

<figure class="case-image">
  <img src="/images/approvalflow.png" alt="ApprovalFlow sign-in screen" />
  <figcaption>ApprovalFlow sign-in screen</figcaption>
</figure>

## Give feedback a formal path

Informal requests for feedback were often not prioritized. The institution wanted feedback to follow a visible, tracked process without making it the final decision. Review collects feedback; approval decides. They also wanted to revisit that distinction after seeing how people used it.

## Correct a request without starting over

Requesters may not know every step a process requires. When a request reaches someone authorized to approve it, that person can correct the route by adding a reviewer or another step, rather than sending the request back to the beginning.

## Match access to real responsibilities

A paper and an annual budget do not belong in front of the same people. Access is therefore part of the workflow design: people see files and actions that match their responsibilities. Requesters can focus on submitting work; approvers can focus on decisions, without being asked to choose between roles they do not perform.

## Isolate each university's data

The institution needed one self-hosted environment for its own work and another hosted environment for universities it serves. Each university needed its own data enclosure. Rules inside one shared database did not provide the separation they wanted, so the system gives each university a separate PostgreSQL database. A control plane signs users in and routes each request to the right database; identity changes are coordinated across those databases, with rollback and retries if an update fails. With a known, finite user base, strict separation mattered more than designing for sharding.

## Let storage follow the institution's needs

The institution wanted to start with local file storage, then move to cloud storage or back if its budget or operating capacity changed. ApprovalFlow supports local and S3-compatible storage, with migration designed to avoid specialist work.

## Keep document work in the platform

Users wanted to work with PDFs, Office files, images, audio, and video without leaving the approval workflow. PDFs and images use built-in support; Office document editing uses an integration with ONLYOFFICE.

## Prevent duplicate work at upload

The intended users were not necessarily technical, and uploading the wrong file could start a chain of notifications and reviews. ApprovalFlow prevents the same person from uploading identical content twice. An optional workspace-wide check also catches a document already in another approval process, so users can find the existing request instead of starting parallel work.

## Make responsiveness visible

The institution wanted responsiveness to inform internal performance reviews. Reports track how long requests and individual steps take, giving managers information they can use in that process.
