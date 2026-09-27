## Identity Source Control

Identity Center settings (identity source, SAML metadata, group assignments) can be modified by principals with the following permissions:

- iam:CreateSAMLProvider / iam:UpdateSAMLProvider / iam:DeleteSAMLProvider
- iam:CreateRole / iam:PutRolePolicy (for aws-reserved/sso.amazonaws.com/* roles)
- identitystore:* (to manage users and groups)
- organizations:* (to view account/org structure)

**Current Principals with These Permissions:**
- Root account (account ID: [REDACTED])
- [Any delegated admins if applicable]
- [Any other roles with AdministratorAccess]

**Risk:** Overly broad access to Identity Center could allow an admin to change the federation source or disable MFA without audit trail.

**Recommendation:** Limit Identity Center admin permissions to a specific role. Enable CloudTrail monitoring of identitystore and sso actions.

See `policies/identity-center-admin-policy.json` for the full IAM policy that governs Identity Center admin permissions.

## Admin Holders

The `allow_admin_s3_us_west2` permission set is currently assigned to:
- `aws-admins` group (0 users) (screenshots/)

No users currently hold admin privileges. The group exists for future assignment when needed.

### Positive Control: Admin Access Limited

**Observation:** The `allow_admin_s3_us_west2` admin permission set is defined but no users are currently assigned. This follows the principle of least privilege.

**Status:** Pass

### Finding 1: Root Account Can Bypass Federation

**Risk:** Root account password is enabled and can be used to sign in directly, completely bypassing Identity Center and SAML/MFA controls. Last used 2026-02-06.

**Recommendation:** Delete root account password. Use temporary credentials from Identity Center or STS for any required access.

---

### Finding 2: IAM User "[REDACTED]" Bypasses Federation

**Risk:** IAM user "[REDACTED]" has:
- Password enabled (direct console sign-in without SAML)
- No MFA enabled
- Active access key (can bypass federation via CLI/API)

This account completely circumvents Identity Center controls.

**Recommendation:** Either migrate "[REDACTED]" to Identity Center federation or disable password and access keys, require STS assume role via Identity Center.