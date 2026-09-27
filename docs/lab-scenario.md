# AeroMed Mission Control — Lab Scenario

## Business context

AeroMed Logistics operates a fictional emergency medical-drone coordination service. The web application is an internal mission-control console used by dispatchers, senior operators, supervisors, and auditors.

The laboratory focuses on one security failure: **authorization state returned to an untrusted client is later trusted by the server**.

## Primary mission

**Mission:** MD-48291  
**Cargo:** Emergency insulin  
**Origin:** Colombo Distribution Hub  
**Destination:** Kandy Regional Hospital  
**Priority:** CRITICAL  
**Risk:** HIGH  
**Stage:** ROUTE_REVIEW

A standard dispatcher can review the route and request an emergency corridor, but cannot request a route change. A senior operator or supervisor can request a route change.

## Intended workflow

1. Authenticate.
2. Open the mission.
3. Request the workflow state.
4. Display actions based on returned capabilities.
5. Submit a selected action.
6. Record the result in the audit log.

## Vulnerable behavior

The workflow response contains capability information such as:

```json
{
  "workflow_id": "WF-88291",
  "stage": "ROUTE_REVIEW",
  "capabilities": {
    "view_route": true,
    "request_reroute": false,
    "request_emergency_corridor": true
  }
}
```

The vulnerable implementation treats the capability object supplied with a later action request as authoritative.

The security boundary is therefore incorrectly placed in the browser/client workflow.

## Secure behavior

The server must derive authorization from trusted state:

- authenticated identity;
- role;
- clearance;
- mission ownership/assignment;
- mission risk;
- mission status;
- current business rules.

The client may request an action, but it must not decide whether the action is authorized.

## Learning outcome

The learner should be able to explain why a response-derived Boolean is not itself a security control, even when the response originally came from a trusted server.
