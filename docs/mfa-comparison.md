## MFA Factor Comparison

TOTP and email are susceptible to phishing and relay attacks because they're vulnerable to a proxy like Evilginx. The user approves or types the code without cryptographically verifying the origin of the login page.

Phishing-resistant factors (FIDO2, Okta Verify FastPass, passkeys) verify the origin domain cryptographically. A proxy on a different domain can't complete the challenge, making them suitable for production.

### Test Configuration

**Policy Rule:** Catch-all rule (Any request)

**Access:** Allowed with any 2 factor types

**Knowledge factor:** Password or Okta Verify FastPass

**Possession factor:** Okta Verify TOTP or Okta Verify FastPass

**Constraints:** 
- Require user interaction (responding to prompts or device activation)
- Re-authenticate on every sign-in to AWS

**Outcome:** User must authenticate with password + a second factor (TOTP or push notification)
