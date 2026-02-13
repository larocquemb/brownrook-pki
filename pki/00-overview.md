# PKI Overview (MOP)

Scope:
- Private Root CA (offline)
- Intermediate CA (YubiKey-backed)
- End-entity certs for: S/MIME, internal TLS, document signing (non-regulatory)

Security rules:
- Never store private keys in this repo
- Never store PKCS#12 (.p12/.pfx) in this repo
- PINs/passwords are never written down

Artifacts live outside Git:
- CA databases (index.txt, serial, newcerts/)
- Private keys and tokens
