# Git SSH Signing with YubiKey (FIDO2) (MOP)

Goal:
Sign commits using ED25519-SK (hardware-backed) so GitHub shows "Verified".

Key creation:
- ssh-keygen -t ed25519-sk -O verify-required -C "<email>"

Git config:
- gpg.format = ssh
- user.signingkey = <path to .pub>
- commit.gpgsign = true

Verification:
- git log --show-signature -1
- GitHub UI indicates "Verified"

Operational notes:
- Signing may prompt for touch/PIN depending on key and OS integration
