# AeroMed Mission Control — Architecture

## Logical architecture

```mermaid
flowchart LR
    B[Browser / Security Testing Proxy]
    N[Nginx]
    F[React Frontend]
    A[Laravel REST API]
    D[(PostgreSQL)]
    B --> N
    N --> F
    N --> A
    A --> D
```

## Security boundary

```mermaid
flowchart TD
    S[Server-side identity and business state]
    R[Workflow API response]
    C[Browser state]
    Q[Action request]
    P[Vulnerable authorization decision]

    S --> R
    R --> C
    C --> Q
    Q --> P
```

In vulnerable mode, the server incorrectly lets browser-controlled workflow state influence authorization.

## Secure authorization path

```mermaid
flowchart TD
    Q[Action request]
    I[Authenticate identity]
    M[Load mission]
    U[Load role and clearance]
    W[Evaluate workflow and business rules]
    Z{Authorized?}
    Y[Perform action]
    N[403 + audit event]

    Q --> I --> M --> U --> W --> Z
    Z -->|Yes| Y
    Z -->|No| N
```

## Components

### Frontend

React + TypeScript + Vite. Responsible for presentation, workflow interaction, and user experience. It is untrusted from an authorization perspective.

### API

Laravel/PHP REST API. Owns authentication, business state, authorization, workflow transitions, and audit events.

### Database

PostgreSQL stores users, missions, routes, workflow sessions, actions, and audit logs.

### Reverse proxy

Nginx provides a simple local entry point and routes browser traffic to the frontend/API services.

## Trust model

Trusted:

- authenticated identity established by the server;
- server-side database state;
- server-side authorization policy.

Untrusted:

- browser memory;
- local storage;
- hidden UI controls;
- JSON fields supplied by the browser;
- capability values copied from an earlier response.
