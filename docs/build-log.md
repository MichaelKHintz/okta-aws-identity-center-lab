AWS was already set up from previous testing
Signed up for Okta Integrator Plan
Installed SAML-tracer browser extension
Forgot to save the .gitignore file so it didn't make it in the intial setup push, and so it was trying to push a sensitive file, but that was caught and remediated before committing. 
IDP setup failed in AWS (screenshots/initial-idp-integration-failed.png), initially suspected file name mismatch but root cause was improper XML indentation in the metadata file. Fixed and verified working.
Created a test user in Okta and created two groups (auditors and admins). Added test user to the auditors group. 
IAM Tile is now visible in the test user's Okta account. 
Attempted to login as the test user via the tile in Okta, but received a "something went wrong; Looks like this code isn't right. please try again" error (screenshots/failed-login-att-with-test-user.png)
Verified that Identity source reads as "External Identity Provider" in AWS IAM Identity Center (screenshots/external-idp-confirmation.png)
