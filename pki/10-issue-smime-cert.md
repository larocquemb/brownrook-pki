# Issue S/MIME Certificate (MOP)

Goal:
Issue an end-entity S/MIME cert signed by the Intermediate CA.

Pre-req:
- OPENSSL_MODULES points to OpenSSL 3 provider modules
- OPENSSL_CONF loads pkcs11 provider
- INT_CA_KEY_URI set to the Intermediate CA key (pkcs11 URI)

Steps:
1) Generate end-entity key (RSA 3072 recommended for Outlook compatibility)
2) Create CSR with subject + emailAddress
3) Prepare extfile with:
   - basicConstraints = CA:FALSE
   - keyUsage = digitalSignature, keyEncipherment
   - extendedKeyUsage = emailProtection
   - subjectAltName = email:<address>
4) Issue with:
   openssl ca -config <intermediate-ca.cnf> -extensions v3_user -extfile <ext.cnf> -in <csr> -out <crt>
5) Verify:
   - SAN email matches
   - keyUsage/EKU correct
   - cert/key match (modulus hash)

Notes:
- For multiple identities (brownrook vs gmail), use separate keypairs and extfiles.
