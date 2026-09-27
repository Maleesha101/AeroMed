# AeroMed Mission Control

> **Security Training Lab — Response Manipulation / Client-Trusted Authorization State**

AeroMed Mission Control is an intentionally vulnerable, local-only enterprise simulation for learning how security-sensitive workflow state returned by an API can become dangerous when a backend later trusts that state from the client.

## Scenario

AeroMed Logistics coordinates emergency medical-drone delivery missions between fictional medical facilities. Operators review routes and request operational changes according to role, clearance, mission risk, weather, and workflow stage.

The central lab mission is **MD-48291**, an emergency insulin delivery requiring elevated operational privileges for a route change.

The vulnerable workflow is deliberately subtle:

1. The workflow API returns capabilities to the browser.
2. The frontend uses those capabilities to construct the UI and later action request.
3. The action request sends the capability state back to the API.
4. Vulnerable server-side code trusts the client-supplied capability.
5. A standard dispatcher can therefore cause an action that should require elevated clearance.
6. Secure mode recalculates authorization from trusted server-side state and rejects the same request.

This is intentionally **not** a simple isAdmin=true exercise.

## Learning objectives

- Map a multi-step API workflow.
- Identify authorization data inside normal JSON responses.
- Trace response-derived state through frontend code.
- Distinguish authentication from authorization.
- Identify a broken trust boundary.
- Demonstrate why hidden/disabled UI controls are not authorization.
- Compare vulnerable and server-authoritative authorization.
- Verify remediation using the same test request.
- Understand useful audit evidence for authorization decisions.

## Architecture

    Browser / Burp Suite
            |
            v
         Nginx
         /   \
        v     v
      React  Laravel API
                 |
                 v
             PostgreSQL

## Intended roles

| Role | Clearance | Route change |
|---|---|---|
| Dispatcher | STANDARD | Denied |
| Senior Operator | ELEVATED | Allowed |
| Supervisor | CRITICAL | Allowed |
| Auditor | STANDARD | Denied |

The exact policy is enforced server-side in secure mode.

## Vulnerable trust boundary

    Server calculates capabilities
              |
              v
         API response
              |
              v
           Browser
              |
         client state
              |
              v
         API action request
              |
              v
    Vulnerable backend trusts the
    client-supplied capability state

The secure architecture is:

    API action request
            |
            v
    Authenticate user
            |
            v
    Load mission from server
            |
            v
    Load role/clearance from server
            |
            v
    Evaluate current mission state
            |
            v
    Apply authorization policy
            |
       +----+----+
       |         |
      Allow     Deny

## Repository documentation

- [Lab scenario](docs/lab-scenario.md)
- [Architecture](docs/architecture.md)
- [Attack flow](docs/attack-flow.md)
- [Vulnerability analysis](docs/vulnerability-analysis.md)
- [Remediation](docs/remediation.md)
- [Testing guide](docs/testing-guide.md)

## Lab safety

This project is designed for local, authorized security education. It uses fictional missions, synthetic records, and simulated operational actions. It does not control real drones, medical systems, airspace, payment systems, or external infrastructure.

Do not deploy the intentionally vulnerable mode to a public environment.

## Intended completion condition

A learner completes the lab when they can explain:

> A response can contain information needed by the UI, but information returned to an untrusted client must never become the authority for a future server-side security decision.

The final implementation should include both vulnerable and secure modes and automated tests proving the difference.
