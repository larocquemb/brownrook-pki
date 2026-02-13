# Export PKCS#12 (.p12) for macOS/Outlook (MOP)

Goal:
Bundle end-entity cert + private key + chain into a .p12 for import into Keychain.

Pre-req:
- End-entity private key and cert exist and match
- Intermediate and Root certs exist (public)

Steps:
1) Confirm key matches cert:
   openssl x509 -in <cert>.crt -noout -modulus | openssl sha256
   openssl rsa  -in <key>.key  -noout -modulus | openssl sha256
   (hashes must match)

2) Build chain bundle (intermediate + root):
   cat <intermediate>.crt <root>.crt > chain.pem

3) Export:
   openssl pkcs12 -export \
     -inkey <key>.key \
     -in <cert>.crt \
     -certfile chain.pem \
     -name "<friendly name>" \
     -out <output>.p12

4) Verify contents:
   openssl pkcs12 -in <output>.p12 -info -noout

Security:
- Do not commit .p12 files
- Store .p12 only in secure local storage
