# Rate Limit Bypass via Interspersed Authentication Success

## Executive Summary
The application's rate-limiting and account lockout mechanisms are vulnerable to a logic bypass. The system enforces a lockout after 7 failed login attempts; however, this failure counter is automatically reset upon a single successful authentication event from the same client. An attacker can exploit this by alternating failed brute-force attempts on a target account with a single successful login on a controlled account, effectively neutralizing the security restriction and allowing indefinite credential guessing.

* **Vulnerability Type:** Rate Limit / Account Lockout Bypass (CWE-307)
* **Severity:** Medium
* **Target:** `example.com` / Login API Endpoint

---

## Vulnerability Description
The application implements a rate limiter tracked by the client's IP address or session context. If a user inputs incorrect credentials 7 consecutive times, the server flags the tracking identifier and enforces a temporary lockout period. 

The underlying flaw exists in the application's authentication state machine: when a successful login occurs, the server globally clears the failure counter associated with that client's tracking identifier, rather than scoping the success or failure strictly to the individual target account. By checking in as an authenticated user on an account they own, an attacker can reset their client failure score back to zero.

---

## Proof of Concept (PoC)

### 1. Verification of the Defensive Baseline
1. Attempt to log in to a target account (`victim@target.com`) using invalid passwords 7 consecutive times.
2. On the 8th attempt, observe that the server returns a `429 Too Many Requests` or a custom lockout status message containing a cooling-off countdown timer.

### 2. Bypassing the Counter
To circumvent this control, an automation workflow (such as a custom Python utility or Burp Suite Intruder) can be configured to use a specific sequential pattern.

| Sequence Number | Target Account | Credential State | Expected Application Behavior |
| :--- | :--- | :--- | :--- |
| **Attempts 1–7** | `victim@target.com` | Invalid Guess | Failure counter increments ($1 \rightarrow 7$) |
| **Attempt 8** | `attacker@target.com` | **Valid Password** | Counter resets to $0$ on successful login |
| **Attempts 9–15** | `victim@target.com` | Invalid Guess | Counter increments again ($1 \rightarrow 7$); no lockout occurs |
| **Attempt 16** | `attacker@target.com` | **Valid Password** | Counter resets to $0$ again |

By executing this alternating sequence, the `429` lockout threshold is never tripped, allowing an attacker to run an exhaustive dictionary attack against `victim@target.com` indefinitely.

---

## Impact
This logic flaw completely invalidates the application's primary defense against automated credential guessing. It grants an attacker the ability to conduct high-velocity brute-force or credential-stuffing attacks against any account on the platform, significantly increasing the risk of Account Takeover (ATO) and subsequent data exposure.

---

## Remediation
To mitigate this flaw, the application's authentication and rate-limiting logic should be refactored to implement the following controls:

* **Account-Based Tracking:** Track authentication failures based on a combination of the source identifier (IP/Session) **AND** the target username/email. A successful login on `Account B` must never clear the failure counter for `Account A`.
* **Distinct Lockout States:** Implement a dual-layer rate limiting strategy:
  1. An IP-based block for raw request volume (e.g., maximum 100 total login attempts per minute regardless of success).
  2. A strict target-account lockout that triggers after 5-7 failed attempts on that specific username, independent of where the request originated.
* **Token Invalidation:** Ensure that when an account authentication occurs, the previous session token context used during the guessing phase is entirely rotated or destroyed.
