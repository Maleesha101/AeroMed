# AeroMed — Remediation Guide

## Core rule

> The client may request an operation; only the server may authorize it.

## Vulnerable pattern

Conceptually:

```text
receive action
receive capabilities
if client.capabilities[action] == true:
    perform action
```

The capability is attacker-controlled input.

## Secure pattern

The server should:

1. authenticate the caller;
2. load the current user from trusted server-side identity;
3. load the mission from the database;
4. load the current mission state;
5. calculate authorization using role and clearance;
6. evaluate business rules;
7. perform the action only if the policy allows it;
8. record the authorization decision.

Conceptually:

```text
authorized = policy.allows(user, mission, action)

if authorized:
    perform action
else:
    return 403
```

## Do not rely on

- hidden buttons;
- disabled buttons;
- localStorage values;
- React state;
- response-derived permissions;
- client-side route guards;
- request body capability flags.

These are useful for user experience but are not an authorization boundary.

## Centralize authorization

Use a server-side policy/service such as:

```text
canRequestReroute(user, mission)
```

The policy should be the single source of truth.

## Revalidate current state

Authorization should be evaluated against current state, not stale state returned earlier.

For example:

```text
user clearance
+
mission risk
+
mission status
+
current workflow stage
+
business rules
```

## Audit decisions

Record both successful and rejected sensitive operations.

Useful fields include:

- user;
- mission;
- action;
- authorization result;
- reason/code;
- timestamp;
- request correlation ID.

Avoid recording secrets or unnecessary sensitive information.

## Verification

After remediation, replay the same lab request.

Expected result for the standard dispatcher:

```http
403 Forbidden
```

A privileged user should continue to receive the expected successful response.

The remediation is verified when authorization no longer depends on client-supplied capability state.
