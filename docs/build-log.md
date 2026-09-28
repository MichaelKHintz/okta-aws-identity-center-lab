AWS was already set up from previous testing
Signed up for Okta Integrator Plan
Installed SAML-tracer browser extension
Forgot to save the .gitignore file so it didn't make it in the intial setup push, and so it was trying to push a sensitive file, but that was caught and remediated before committing. 
IDP setup failed in AWS (screenshots/initial-idp-integration-failed.png), initially suspected file name mismatch but root cause was improper XML indentation in the metadata file. Fixed and verified working.
Created a test user in Okta and created two groups (auditors and admins). Added test user to the auditors group. 
IAM Tile is now visible in the test user's Okta account. 
Attempted to login as the test user via the tile in Okta, but received a "something went wrong; Looks like this code isn't right. please try again" error (screenshots/failed-login-att-with-test-user.png)
Verified that Identity source reads as "External Identity Provider" in AWS IAM Identity Center (screenshots/external-idp-confirmation.png)

Phase 2 Start: Provisioning and Access
Turned on automatic provisioning in the IAM Identity Center and copied the SCIM endpoints and access token
Enabled the SCIM API integration functionality in Okta.
Pasted the AWS generated SCIM endoint for IPv4 url into the Base Url field in Okta. Chose this instead of Dual-Stack because IPv6 functionality is likely not needed for this integration.
Pasted the AWS generated access token into the API Token field in Okta. 
Tested the api credentials am seeing an "AWS IAM Identity Center was verified successfully!" message in Okta (screenshots/SCIM-api-test-success.png)
Saved the settings in Okta
Enabled the "Create Users", "Update User Attributes", & "Deactivate Users" functionality in Okta so that Okta would function as the single source of truth. 
Also unchecked 'Set password' in the "Create Users" section because federated users authenticate through Okta, not with passwords stored in Identity Center. (screenshots/provisioning-to-app-settings.png)
AWS Identity Center SCIM token will expire in 365 days. AWS Identity Center will notify 90 days before expiry
Reassigned the aws-auditors and aws-admins groups in Okta so that Okta would provision the users. 
Pushed the aws-auditors and aws-admins groups from Okta to AWS Identity Center
Validated the the groups and user made it into AWS IAM Identity Center and showed "SCIM" in "Created By" column (screenshots/user-pushed-to-aws-via-scim.png) (screenshots/groups-pushed-to-aws-via-scim.png). Had to Push both the groups and the users even though the users are listed in the group so that the individual user would have permissions. 
Created a new permission set for the aws-auditors group using pre-defined "ReadOnlyAccess". Opted for this instead of "SecurityAudit" because the "SecurityAudit" permissions would be better suited for automated software review of the config metadata. Session limit was set to 1 hour
Created an custom inline policy for admin users so that I could get vert granular about what they could do. Initially it was set so that admins were only allowed to perform actions on S3 objects for all resources as long as it was in the us-west-2 region (this is the home region used for the testing account).
Added the poliy sample to policies/admin-scoped-permission-set.json
Navigated to the "AWS Accounts" portion of the IAM Identity Center and assigned the "ReadOnlyAccess" permission set to the "aws-auditors" group and assigned the "allow_admin_s3_us_west2" permission set to the "aws-admins" group (screenshots/group-permission-set-assignment.png)
The AWS Okta tile is now working for the root user and login is allowed. When attempting to access the AWS access portal URL https://{portal_id}.awsapps.com/start (created in IAM Identity Center) directly in the browser, it has the Okta login module present and opens Okta instead of just logging directly into AWS
Test user only has "ReadOnlyAccess" option showing on the AWS access portal
Attempts to create an S3 bucket with this account failed with response "Failed to create bucket: To create a bucket, the s3:Createbucket permission is required" (screenshots/bucket-creation-failure.png)
Removed test user from the auditors group and moved them to admins group. Logged out and back into AWS through Okta and user is still showing both permission sets. Looked in Okta admin panel and confirmed that audit group is showing 0 users and admin group is showing 1 user. Logged out of test user Okta and back in to AWS and both permissions sets still showing. AWS Identity center is still showing the user in both groups. Forced a push of the aws-auditors group in Okta and now AWS Identity Center looks correct. Logged back into AWS access portal via Okta as test user and only the admins permission set is showing now
Logged in to the test user account using the admin permissions. Attempted to create a bucket in a region that was not allowed and the creation failed with the same error as before, which was expected (screenshots/admin-bucket-creation-failure-wrong-region.png).
Attempted to create a bucket in the allowed region and it was successful (screenshots/admin-bucket-creation-success.png).
Removed user from the aws-admins group in Okta. Validated that their tile no longer shows up and that user account shows as "disabled" in AWS IAM Identity Center. (screenshots/user-disabled-in-iam-identity-center.png)

Phase 3: Sign-on policy and MFA
Created a new authentication policy in Okta that forced MFA with at least 2 factors.
Added the policy to the AWS IAM Identity Center app in Okta. 
Had to force push the aws-admins group from Okta again becuase the user was still showing in IAM Identity center for some reason. Forced the push of that group and the user is no longer there. 
Added the test user to the aws-auditors group in Okta and validated that they now show up in AWS IAM Identity Center. 
The attempt to log into AWS Identity Center did not force MFA for the test user. 
First issue: The user did not have a second factor set up in the Okta env. Second issue: The Okta management console only had phone and Okta Verify App set up as allowed authenticators. Email was "added" as an authenticator, but was disabled in the default config. 
When attempting to enable email, received this error message To set Email Enrollment to Optional or Required, you must first enable the email authenticator for authentication and recovery. Go to Authenticators > Setup to change the current Recovery only setting. (screenshots/email-authenticator-setup-error.png)
Opened the email authenticator setup and selected the "Authentication and Recovery" radio button. (screenshots/email-authentication-and-recovery.png)
Enabled email as an optional authenticator and left the auto-enroll option selected so that any new accounts would be auto-enrolled using the account profile when possible. 
Push and Phone could also be used as authenticators, but would require some additional configuration so I opted to leave those alone for now
MFA still wasn't working when I tried to open the AWS app. Did some research and found that the AWS-Access policy now showed email, but the "Disallow specific authentication methods" radio button was selected and "Email" was listed as a disallowed item. (screenshots/mfa-email-disallowed.png)
Changed the radio button to "Allow any method that can be used to meet the requirement" instead (screenshots/mfa-any-method.png).
MFA still was not working and there did not appear to be a way to add it in the test users settings. After some research, I found a toggle in the "Settings --> Features" section of the Okta Admin dashboard and that had an "Enable optional email enrollment for Okta Identity Engine" toggle that needed to be turned on (screenshots/enable-email-enrollment.png)
After switching the toggle on, email was now present as one of the security methods in the test users Okta account and it was automatically set already.
MFA was still failing. Further researching showed a "Disable Force Authentication" checkbox being checked in the SAML 2.0 setting of the AWS app in the Okta admin portal (screenshots/saml-disable-force-auth.png)
Unchecking that box and saving still did not allow MFA to work properly when accessing the AWS app from Okta. 
After some further research, there is another section at the bottom of the authentication policy that is attached to the AWS app titled, "Prompt for authentication". That was set by default to "When it's been over a specified length of time since the user accessed any resource protected by the active Okta global session: Time since last sign in: 12 Hours" (screenshots/prompt-for-authentication-default.png). I'm going to try and change that to "Every time user signs in to resource".
Finally! Now the app is asking for MFA when attempting to open the AWS app and TOTP and Email are shown as options (screenshots/mfa-prompt.png). This final config for authentication policy for the app is shown in (screenshots/prompt-for-authentication-every-time.png)
Now that that is working, I'm going to turn off email as an authenticator as it's not as secure of a method to use. TOTP is slightly better, but not phishing resistant. Something like a FIDO2 key, Okta verify with FastPass, or passkeys would be the better phishing resistant options because they rely on the physical device to also match. 
Email was disallowed and deactivated from use in the Okta Admin Portal

Phase 4: Inspect and audit
Opened SAML-tracer in Firefox for a separate session from the default browser.
Found the SAML POST request in SAML-tracer (marked with the SAML tag)
Saved a screenshot of the SAML POST in (screenshots/saml-assertion.png) an annotated version has been added to docs/saml-assertion.md
Navigated to Cloudtrail with the test user account to look at login info. Changed the region to the home region for the account
Identified a "AssumeRoleWithSAML" event that correlated to the time of login (screenshots/cloudtrail-assume-role-with-saml-event.png)
Traced sign-in flow in CloudTrail. AssumeRoleWithSAML event shows the test user (testuser@) successfully assumed the AWSReservedSSO_ReadOnlyAccess role with session name matching the Okta username. This confirms the federation chain from Okta identity through SAML assertion to AWS role assumption (screenshots/cloudtrail-assume-role-with-saml-details.png)
Reviewed users with potential admin privileges and found there is a policy that could allow for admin privileges, but no current users in the group. Added the finding to the docs/audit-findings.md doc
Navigated to IAM --> Credential Reports to review current users and their permissions
Identified two users with credentials that could circumvent federation and noted those in the docs/audit-findings.md file

Post Lab Notes:
Went back and set up Okta Fastpass and was able to sign in to the Test User account with FastPass as an option. This provided a phish resistant option for users.
