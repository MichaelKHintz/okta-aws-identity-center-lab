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