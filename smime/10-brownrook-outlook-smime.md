# Brown Rook S/MIME for Outlook (MOP)

Identity:
- Email: paul.larocque@brownrook.com

Recommended crypto:
- RSA 3072 end-entity key for compatibility

Required extensions:
- basicConstraints: CA:FALSE
- keyUsage: digitalSignature, keyEncipherment (critical)
- extendedKeyUsage: emailProtection (critical)
- subjectAltName: email:<address>

macOS:
- Import .p12 into login keychain
- Verify cert appears under "My Certificates" with private key attached

Outlook:
- Ensure account "From" address matches SAN email
- Select signing cert under account security settings (if required)
