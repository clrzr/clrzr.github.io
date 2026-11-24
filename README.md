# Michael Ikirodah — Technical Architecture

Technical architecture monograph and infrastructure specifications built with **Jekyll** and hosted on **GitHub Pages**.

Live: [clrzr.github.io](https://clrzr.github.io)

---

## Financial Infrastructure

All specifications live in `_case_studies/` (rendered via Jekyll collections):

| Specification | Path | Focus |
|---|---|---|
| **A Ledger's Core Model** | `_case_studies/ledger.md` (`/engineering/fintech/ledger/`) | Double-entry zero-sum equilibrium, pooled omnibus accounts, universal hold clearing, and state recovery. |
| **Payment Intent Architecture** | `_case_studies/payments.md` (`/engineering/fintech/payments/`) | Decoupled intent declaration, destination-first binding, parallel pre-validation, and canonical idempotency. |
| **Continuous Signal Risk Engine** | `_case_studies/fraud.md` (`/engineering/fintech/fraud/`) | Asynchronous behavioral signal aggregation, kinematic Haversine delta-velocity detection, and real-time risk caching. |
| **Device Boundary & Identity Engine** | `_case_studies/kyc.md` (`/engineering/fintech/kyc/`) | Decoupled asynchronous KYC state machines, multi-provider failover, blind-index HMAC PII containment, and single-device invalidation. |

---

## Diagrams

Diagrams use **Mermaid.js**. They are rendered client-side on all pages using the `system` layout. Simply write standard fenced Mermaid blocks:

````markdown
```mermaid
flowchart TD
  A --> B
```
````

---

## Project Structure

```
_case_studies/  ← Architecture specifications
_data/          ← Site metadata and navigation
_layouts/       ← Shared layouts (base.html, system.html)
_includes/      ← Shared components (nav.html)
images/         ← Visual assets
index.html      ← Selected specifications index
about/          ← Background, focus, and architectural principles
_config.yml     ← Jekyll settings and collection permalinks
Gemfile         ← Ruby dependencies
```

---

## Deployment

Push to `main`. GitHub Pages detects `_config.yml`, runs Jekyll, and deploys automatically.

