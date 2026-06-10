# Authentication Bypass: Insecure OTP Lifecycle and Lack of Code Invalidation
Executive Summary

A critical logic flaw exists within the platform’s multi-factor authentication (MFA) system. The backend fails to invalidate previously issued One-Time Passwords (OTPs) when a new OTP is requested for the same user account. The server validates incoming codes based solely on a time-to-live (TTL) window (e.g., 10 minutes) and a single "used" flag, rather than enforcing a "latest-code-only" policy. This allows multiple historical OTPs to remain active simultaneously, significantly widening the window of opportunity for account takeover.
Vulnerability Details

* **Vulnerability Type:** Business Logic Error / Broken Authentication

* **OWASP Top 10:** A07:2021-Identification and Authentication Failures

* **Affected Feature:** OTP-based Authentication Flow

---

## Problem Description

When a user triggers multiple OTP requests (e.g., via "Resend" or initiating fresh login flows), the system fails to revoke the validity of older, unused codes. As long as a code is within the 10-minute validity window and has not been marked as "used," it will be accepted by the server.

This state management failure creates a "skeleton key" effect where multiple non-sequential codes remain valid for the same authentication session, disregarding the temporal progression of the user's login attempts.

---

## Steps to Reproduce

1. Navigate to the login page and authenticate with valid user credentials.

2. At the OTP verification prompt, capture the first received code (OTP #1). Do not use it.

3. Trigger a "Resend OTP" action to generate OTP #2.

4. Initiate a completely new login flow (re-enter credentials) to generate a third code (OTP #3).

5. Open an intercepting proxy or a browser session and submit OTP #1.

6. Result: The server accepts OTP #1 as valid, successfully authenticating the user, despite it being chronologically superseded by two subsequent codes.

---

## Business Impact

Account Hijacking Window: This flaw extends the window of exploitation for an attacker. If an attacker triggers an OTP request for a target account, they can retain access to that "winning" code for the full duration of the TTL (e.g., 10 minutes), even if the legitimate user interacts with the system or requests newer codes.

Brute-Force Amplification: The lack of strict invalidation allows for multiple valid codes to exist simultaneously. In scenarios where a user or an attacker requests multiple codes, the attack surface expands—an attacker no longer needs to guess a single valid code, but rather any one of the several valid codes currently stored in the backend state.

---

## Demonstrated Impact

Testing confirms that the application treats all generated OTPs for a single login session as equivalent valid tokens. The system does not invalidate older tokens upon the issuance of a newer one, nor does it invalidate tokens upon the initiation of a new login flow. This was verified by successfully using an "orphaned" OTP (the first of three) to gain access to the application dashboard.
Remediation Recommendations

Latest-Code-Only Policy: Modify the backend authentication logic to enforce that only the most recently generated OTP is considered valid for a specific user session or login attempt.

Immediate Invalidation: When a new OTP is requested, the server must explicitly set the status of all previous unused OTPs for that user to "expired" or "invalidated" in the database.

Contextual Binding: Ensure OTPs are strictly bound to the specific login request session ID. If a new session starts, existing unconsumed codes from previous sessions should be wiped immediately.
