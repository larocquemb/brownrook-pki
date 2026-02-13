# No-Secrets Policy (MOP)

This repository must never contain:
- Private keys (*.key, *.pem)
- PKCS#12 bundles (*.p12, *.pfx)
- PINs, passwords, recovery codes
- CA database directories (index.txt, serial, newcerts/)

Allowed:
- Public certificates (optional, if explicitly intended)
- High-level procedures and commands
- Non-sensitive identifiers (avoid detailed physical storage locations)
