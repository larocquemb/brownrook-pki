# Issue Server TLS Certificate (MOP)

## Goal

Issue an internal server TLS certificate signed by the BrownRook Intermediate CA while preserving the separation between CA signing keys and application private keys.

## Security rules

- The Intermediate CA private key remains on the YubiKey and is used only through its PKCS#11 URI.
- Generate a **new private key for every server certificate**.
- Never reuse an existing application private key for a newly issued certificate.
- **Do not reuse, copy, or export the Argo CD private key.** Argo CD's existing key is outside the scope of this procedure.
- Never store server private keys, PINs, passwords, PKCS#12 files, or CA database state in this repository.
- Commit only public certificates, CSRs, templates, or documentation when there is an operational reason to retain them in Git.

## Pre-requisites

- The BrownRook Intermediate CA certificate is available.
- The YubiKey containing the Intermediate CA private key is connected and accessible.
- `OPENSSL_MODULES` points to the OpenSSL 3 provider modules.
- `OPENSSL_CONF` loads the PKCS#11 provider.
- `INT_CA_KEY_URI` identifies the Intermediate CA key on the YubiKey using a PKCS#11 URI.
- The Intermediate CA OpenSSL configuration and CA database are available outside Git.
- The requested DNS names have been reviewed and approved.

Before issuance, verify that the CA key is reachable through PKCS#11 without exporting it.

## Inputs

Define the certificate identity before generating any key material.

Example:

```bash
SERVER_NAME=argocd.internal.example
SERVER_KEY=argocd.internal.example.key
SERVER_CSR=argocd.internal.example.csr
SERVER_CERT=argocd.internal.example.crt
```

Replace the example DNS name with the actual approved server name. If multiple DNS names are required, enumerate all of them in the Subject Alternative Name extension.

## Procedure

### 1. Generate a new server private key

Generate a fresh key specifically for this certificate. The key must be generated in the destination system's protected storage or in a secure working directory from which it will be installed immediately.

RSA 3072 example:

```bash
openssl genpkey \
  -algorithm RSA \
  -pkeyopt rsa_keygen_bits:3072 \
  -out "$SERVER_KEY"

chmod 600 "$SERVER_KEY"
```

Do not substitute the Argo CD private key or any other previously issued application's key.

### 2. Create the certificate signing request

Create a CSR using the new server key.

```bash
openssl req -new \
  -key "$SERVER_KEY" \
  -subj "/CN=${SERVER_NAME}" \
  -out "$SERVER_CSR"
```

The Common Name is retained for operator readability. Client validation must rely on the Subject Alternative Name extension.

### 3. Prepare server certificate extensions

Create a temporary extension file outside Git, for example `server-ext.cnf`:

```ini
[server_cert]
basicConstraints = critical,CA:FALSE
keyUsage = critical,digitalSignature,keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = argocd.internal.example
```

For additional approved names, add sequential entries:

```ini
DNS.2 = argocd.example
DNS.3 = argocd.service.internal
```

Do not issue certificates containing unreviewed wildcard or unrelated DNS names.

### 4. Review the CSR before signing

Inspect the CSR and confirm that the subject and public key belong to the intended server certificate request.

```bash
openssl req -in "$SERVER_CSR" -noout -text
```

Confirm:

- the Common Name is expected;
- the public key is newly generated for this request;
- no private key material is present in the CSR;
- the requested SAN list in the extension file is correct.

### 5. Sign with the YubiKey-backed Intermediate CA

Use the Intermediate CA through the configured PKCS#11 provider. The CA private key must remain on the YubiKey.

```bash
openssl ca \
  -config <intermediate-ca.cnf> \
  -extensions server_cert \
  -extfile server-ext.cnf \
  -in "$SERVER_CSR" \
  -out "$SERVER_CERT"
```

The Intermediate CA configuration must reference the YubiKey-backed key through `INT_CA_KEY_URI` / PKCS#11. Do not export the Intermediate CA key to make this command work.

### 6. Verify the issued certificate

Inspect the certificate:

```bash
openssl x509 -in "$SERVER_CERT" -noout -text
```

Confirm at minimum:

- `CA:FALSE`;
- Extended Key Usage contains `TLS Web Server Authentication` / `serverAuth`;
- Subject Alternative Name contains exactly the approved DNS names;
- issuer is the BrownRook Intermediate CA;
- validity period is appropriate.

Verify the chain:

```bash
openssl verify \
  -CAfile <root-ca.crt> \
  -untrusted <intermediate-ca.crt> \
  "$SERVER_CERT"
```

Verify that the certificate matches the newly generated private key without exposing the key:

```bash
openssl pkey -in "$SERVER_KEY" -pubout -outform DER | openssl sha256
openssl x509 -in "$SERVER_CERT" -pubkey -noout \
  | openssl pkey -pubin -outform DER \
  | openssl sha256
```

The two SHA-256 values must match.

### 7. Install the certificate and new server key

Install only:

- the newly issued server certificate;
- its newly generated server private key;
- the required public CA chain.

Apply destination-specific file ownership and permissions.

For Argo CD or any other Kubernetes workload, create/update the destination secret from the **newly generated server key and issued certificate**. Do not retrieve, decode, export, or repurpose an existing Argo CD private key as part of this process.

### 8. Clean up temporary material

After successful installation and validation:

- securely remove temporary extension files if they contain operational details that do not need retention;
- move the CSR/public certificate to the approved records location if required;
- retain the CA database and issuance record according to the Intermediate CA backup procedure;
- ensure the server private key remains only in its approved protected location.

## Post-issuance validation

From a client that trusts the BrownRook Root CA, validate the live TLS endpoint:

```bash
openssl s_client \
  -connect <server>:443 \
  -servername <dns-name> \
  -verify_return_error
```

Confirm that the served certificate is the newly issued certificate, the hostname matches a SAN, and the chain terminates at the expected BrownRook Root CA.

## Failure / abort conditions

Stop the procedure if any of the following occurs:

- the YubiKey-backed Intermediate CA key cannot be accessed through PKCS#11;
- the signing process requests exporting or copying the CA private key;
- the only available application key is an existing Argo CD or other workload private key;
- the CSR identity or SAN list does not match the approved request;
- the issued certificate has incorrect key usage, EKU, SANs, issuer, or validity;
- certificate/private-key verification does not match.

Correct the cause and generate a new request if necessary. Do not weaken the key-separation controls to complete issuance.
