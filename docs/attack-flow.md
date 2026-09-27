# AeroMed — Attack Flow

This document describes the intended **authorized local-lab investigation flow**.

## 1. Discover the workflow API

Open the mission and observe the workflow request:

```http
GET /api/missions/MD-48291/workflow
```

Inspect the JSON for security-sensitive workflow fields.

## 2. Identify response-derived authorization state

Look for fields such as:

```text
capabilities
request_reroute
request_emergency_corridor
override_weather_restriction
clearance
workflow stage
```

The important question is not simply whether these fields exist, but whether later authorization depends on them.

## 3. Trace the frontend

Determine how the frontend stores and uses the workflow response.

The expected discovery is:

```text
API response
    -> browser state
    -> action availability
    -> action request
```

## 4. Compare the action request

Observe:

```http
POST /api/missions/MD-48291/actions
Content-Type: application/json
```

The vulnerable lab intentionally sends authorization-related capability state in this request.

## 5. Controlled manipulation

Inside the local lab, change the client-controlled capability representation so that the action appears available to the dispatcher.

The goal is to establish whether the server independently evaluates authorization or trusts the client state.

## 6. Observe the result

In vulnerable mode, an unauthorized route-change request is accepted.

The audit log records the action.

In secure mode, the same authorization attempt must return:

```http
403 Forbidden
```

## 7. Root cause

The flaw is a broken trust boundary:

> The server treats client-controlled response-derived state as an authority instead of treating it as untrusted input.

## Expected evidence

A complete finding should contain:

- original workflow response;
- relevant capability field;
- action request;
- evidence that the client influenced the capability;
- server response;
- resulting audit event;
- secure-mode comparison.
