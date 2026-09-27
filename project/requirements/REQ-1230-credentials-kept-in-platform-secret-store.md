---
id: REQ-1230
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1230

Credentials MUST be kept in the platform secret store: Keychain on macOS,
Credential Manager on Windows, and Secret Service (`libsecret`) on Linux.
meowg1k persists nothing itself and asks the operating system to, because OAuth
forces something to persist and the principle forbids meowg1k being the thing
that holds the secret.

(from https://github.com/retran/meowg1k/issues/166, high)
