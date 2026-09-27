# AeroMed — Testing Guide

## Scope

Perform testing only against the local AeroMed training environment.

## Functional checks

Verify:

- login;
- mission listing;
- mission details;
- workflow retrieval;
- route review;
- permitted actions;
- audit logging.

## Security test matrix

| User | Action | Expected secure result |
|---|---|---|
| Dispatcher | View route | Allowed |
| Dispatcher | Request emergency corridor | Allowed when policy permits |
| Dispatcher | Request reroute | 403 |
| Senior Operator | Request reroute | Allowed |
| Supervisor | Request reroute | Allowed |
| Auditor | Request reroute | 403 |

## Response-manipulation test

1. Authenticate as the seeded dispatcher.
2. Open mission MD-48291.
3. Capture the workflow response.
4. Identify the route-change capability.
5. Trace how the frontend uses it.
6. Observe the action request.
7. In vulnerable mode, alter the client-controlled workflow state in the local lab.
8. Submit the resulting request.
9. Confirm the vulnerable implementation accepts the unauthorized action.
10. Inspect the audit log.
11. Switch to secure mode.
12. Repeat the same test.
13. Confirm the server returns 403.

## Burp Suite

The API uses ordinary JSON over HTTP in the local lab, making it suitable for:

- Proxy;
- HTTP history;
- Repeater;
- request/response comparison.

Keep all testing within the authorized laboratory.

## Automated tests

The project should include tests for:

### Vulnerable mode

A standard dispatcher with client-controlled reroute capability should demonstrate the intentionally vulnerable behavior.

### Secure mode

The identical authorization attempt must be rejected.

### Privileged user

An authorized senior operator or supervisor must remain able to perform the permitted action.

## Evidence checklist

For a security report, preserve:

- endpoint;
- HTTP method;
- relevant request;
- relevant response;
- authenticated role;
- expected authorization;
- observed result;
- audit event;
- secure-mode result.

Do not include real credentials or unrelated sensitive data.
