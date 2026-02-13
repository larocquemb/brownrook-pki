# Intermediate Certificate Authority Workspace

This directory contains the working environment for the BrownRook intermediate
Certificate Authority (CA). It is used to issue and track certificates signed
by the intermediate CA.

## Directory Purpose

The intermediate CA sits between the root CA and end-entity certificates:

Root CA → Intermediate CA → Leaf Certificates

This layered model limits risk by keeping the root CA offline while allowing
the intermediate CA to perform operational signing.

---

## Workspace Structure

```
intermediate/
├── certs/       # Public certificates (safe to share)
├── private/     # Private CA key (NEVER commit)
├── db/          # CA database state
└── newcerts/    # Archive of issued certificates
```

### newcerts/

The `newcerts/` folder is automatically managed by OpenSSL.

It stores copies of **every certificate issued by this CA**, named by serial
number:

```
newcerts/
1000.pem
1001.pem
...
```

Each file is a public X.509 certificate, not a private key.

Purpose:

- Provides a permanent audit trail of issued certificates
- Enables revocation tracking
- Supports CA bookkeeping and recovery

Think of it as a local certificate ledger.

These files are public artifacts, but they are considered **operational CA
output**, not repository source material. They are backed up securely but are
typically excluded from version control to avoid noise and accidental leakage
of operational metadata.

---

## Creating the Intermediate CA Workspace

From the PKI root directory:

```bash
mkdir -p intermediate/{certs,private,db,newcerts}
touch intermediate/db/index.txt
echo 1000 > intermediate/db/serial
```

This initializes the OpenSSL CA database:

- `index.txt` → certificate registry
- `serial` → next certificate serial number

---

## Security Model

Private material must never leave protected storage:

```
private/  → secret signing key
db/       → operational state
newcerts/ → issued certificate archive
```

Only public certificates in `certs/` are intended for distribution.

Formal safety rule:

Git contents ⊆ public artifacts  
Git contents ∩ private material = ∅

---

## Backup Strategy

The intermediate CA workspace represents signing authority and certificate
history. Regular encrypted backups are mandatory.

Loss of this directory means:

- Lost issuance history
- Revocation tracking failure
- Operational recovery difficulty

---

## Summary

The intermediate CA workspace provides:

- Certificate issuance tracking
- Audit trail of signed certificates
- Secure separation of public vs private artifacts
- Recovery support

It is an operational CA environment — not just a certificate store.