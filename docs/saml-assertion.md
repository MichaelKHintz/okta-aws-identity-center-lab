# SAML Assertion Annotation

Okta sends this SAML response to AWS's ACS URL during federation.

## Response Structure

```xml
<!-- Okta sends this SAML response to AWS's ACS URL -->
<saml2p:Response 
    Destination="https://us-west-2.sso.signin.aws/platform/saml/acs/..." 
    ID="id-356802842[REDACTED]"
    IssueInstant="2026-09-27T05:07:07.216Z"
    Version="2.0">
    
    <!-- Okta's entity ID - AWS checks this matches the uploaded metadata -->
    <saml2:Issuer>http://www.okta.com/exk1821[REDACTED]</saml2:Issuer>
    
    <!-- Digital signature using Okta's private key -->
    <ds:Signature>
        <ds:SignatureValue>[REDACTED]</ds:SignatureValue>
        <!-- AWS validates the signature using Okta's public cert from metadata -->
        <ds:X509Certificate>[REDACTED - Okta's cert]</ds:X509Certificate>
    </ds:Signature>
    
    <saml2:Assertion>
        <!-- The username AWS uses to match against provisioned users -->
        <saml2:NameID Format="...emailAddress">michael+testuser@</saml2:NameID>
        
        <saml2:SubjectConfirmation>
            <saml2:SubjectConfirmationData 
                NotOnOrAfter="2026-09-27T05:12:07.216Z"
                Recipient="https://us-west-2.sso.signin.aws/platform/saml/acs/..."
            />
        </saml2:SubjectConfirmation>
        
        <!-- Validity window - AWS checks current time is within this -->
        <saml2:Conditions 
            NotBefore="2026-09-27T05:02:07.216Z"
            NotOnOrAfter="2026-09-27T05:12:07.216Z">
            
            <!-- AWS checks this matches its own issuer URL - prevents replay to other services -->
            <saml2:Audience>https://us-west-2.signin.aws.amazon.com/platform/saml/d-92[REDACTED]</saml2:Audience>
        </saml2:Conditions>
    </saml2:Assertion>
</saml2p:Response>
```

## Key Fields AWS Validates

| Field | Purpose | Value |
|-------|---------|-------|
| **Issuer** | Confirms assertion is from Okta | `http://www.okta.com/exk1821roqi0Ha[REDACTED` |
| **Destination** | Ensures assertion is for this ACS URL | `https://us-west-2.sso.signin.aws/platform/saml/acs/...` |
| **Recipient** | Prevents misuse at other endpoints | Same as Destination |
| **Audience** | Prevents replay to other service providers | `https://us-west-2.signin.aws.amazon.com/platform/saml/d-92[REDACTED]` |
| **NameID** | Matches to provisioned user account | `michael+testuser@` |
| **NotBefore/NotOnOrAfter** | Validity window (10 minutes) | `2026-09-27T05:02:07Z` to `05:12:07Z` |
| **Signature** | Cryptographic proof of authenticity | Validated using Okta's X.509 certificate |