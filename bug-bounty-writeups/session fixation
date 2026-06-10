[Vulnerability Report] Session Fixation: Old Session Remain Valid After New Login on id.EXAMPLE.com

An issue with session management was identified within the target platform's Single Sign-On (SSO) authentication system (id.example.com). The server fails to properly invalidate historical state or session tokens generated during the authentication flow. Specifically, if a successful login response containing session-establishing tokens (Set-Cookie) is generated but intercepted and dropped before completion, that state remains permanently valid on the server side.

This state remains active even after subsequent valid logins, explicit logouts, or subsequent failed authentication attempts. This allows an attacker who has intercepted an unconsumed or historical session state to achieve persistent, unauthorized account access, bypassing the explicit logout actions of the user.
Vulnerability Details

    Vulnerability Type: Broken Authentication / Improper Session Handling

    OWASP Top 10: A01:2021-Broken Access Control / A07:2021-Identification and Authentication Failures

    Affected Endpoint: [https://id.example.com](https://id.example.com) / [https://accounts.example.com/websso/signin](https://accounts.example.com/websso/signin)

Problem Description

When a user authenticates successfully, the backend issues an authentication state (including Set-Cookie directives). If this response is intercepted and dropped (never completed by the client browser), the server does not track it as an abandoned or dead state.

Furthermore, the server-side session management does not enforce a strict concurrent session limit or tie the state to a single active session lifecycle. As a result, performing a manual logout or generating newer, valid sessions does not destroy the previously generated tokens. Multiple unique session states can coexist simultaneously for a single user account.

Steps to Reproduce

1. Navigate to the target login interface:
   https://accounts.example.com/websso/signin?client_id=[REDACTED_UUID]&response_type=code&redirect_uri=https://www.example.com/api/login&state=signin:/my-account
2. Enter valid credentials, configure your intercepting proxy (e.g., Burp Suite) to intercept responses, and submit the form.
3. Capture the successful HTTP response containing the `Set-Cookie` header(s) that establish the session. 
4. Copy and document this complete HTTP response (along with the specific `Set-Cookie` values) into a local file for later use.
5. **Drop the request** in Burp Suite so it never completes rendering in the browser.
6. Login normally again with the same correct credentials. This completes the flow and generates a newly updated, active session cookie in the browser.
7. Click **Logout** within the web application to explicitly terminate the active session.
8. Attempt to log in again, but intentionally provide a **correct email and a wrong password**. Intercept this failed request/response flow.
9. Using Burp Suite Repeater, modify the failed authentication response by replacing it entirely with the old, unused successful HTTP login response captured in **Step 4**.
10. Forward the modified response to the browser. Observe that the old session token is accepted by the application backend, granting full, authenticated access to the account.

> **Note on Multi-Token Persistence:** This process can be repeated multiple times. If three separate login attempts are made and intercepted, all three distinct `Set-Cookie` tokens will remain simultaneously valid and usable, rather than being revoked server-side upon logout.

---

## Impact

### 1. Persistent Unauthorized Access & Session Fixation
If an attacker intercepts or extracts an unused session token (via network sniffing on unsecured connections, localized proxy access, or malicious malware/extensions), they can retain an indefinite backdoor into the victim's account.

### 2. Complete Bypass of User Intent (Logout Invalidation)
When a user clicks "Logout," they expect all active sessions and tokens tying their identity to that browser state to be destroyed on the server. Because the server fails to invalidate these parallel states, the user is left with a false sense of security while their account remains exposed via the older tokens.

### 3. Increased Attack Surface
By allowing an arbitrary number of concurrent, historical session tokens to remain live simultaneously instead of enforcing a strict lifecycle closeout, the application significantly increases the window of opportunity for token reuse and session hijacking.

---

## Remediation Recommendations

* **Implement State Invalidation on Logout:** Ensure that an explicit logout action destroys *all* active or pending session tokens associated with that specific user account ID on the server database, not just the one currently used to send the logout request.
* **Tie State to Completion Lifecycle:** If an authentication token or state is issued but not explicitly consumed within a short time frame (e.g., 2–5 minutes), it should automatically expire server-side.
* **Enforce Strict Session Concurrency:** Implement a server-side check that limits the maximum number of concurrent active sessions allowed per user account, immediately revoking the oldest tokens when new ones are authorized.
