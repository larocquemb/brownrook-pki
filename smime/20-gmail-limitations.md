# Gmail S/MIME Limitations (MOP)

Consumer Gmail web interface (gmail.com):
- Cannot use local Keychain / YubiKey private keys
- Decryption in browser not supported for arbitrary certificates

Workarounds:
- Use Apple Mail / Outlook Desktop / Thunderbird for S/MIME
- If using Google Workspace, evaluate hosted S/MIME features (admin-managed)

Key point:
- Recipients only need your public cert to encrypt to you
- Never share or upload your private key (.p12) publicly
