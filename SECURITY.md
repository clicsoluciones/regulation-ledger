# Security Policy

## Supported versions

| Version | Supported          |
|---------|--------------------|
| 0.1.x   | :white_check_mark: |
| < 0.1   | :x:                |

## Reporting a vulnerability

We take security seriously. If you discover a security vulnerability in `regulation-ledger`, please report it privately.

**Do not open a public issue for security vulnerabilities.**

Send an email to: yaquimartinez@gmail.com

Please include:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

You should receive a response within 72 hours. We will work with you to understand the issue and coordinate disclosure.

## Cryptographic considerations

`regulation-ledger` provides immutability through cryptographic hashes and signatures. The security of the system depends on:

1. **Hash function strength.** We use SHA-256, currently considered secure. Future migration to SHA-3 or BLAKE3 is planned.
2. **Private key protection.** The optional Ed25519 signature feature requires the user to manage their own keys. The library does not store private keys.
3. **Storage security.** The library does not implement at-rest encryption. If you store records in an encrypted volume (LUKS, encrypted database, etc.), the records inherit that protection.

We do not consider the following as vulnerabilities of `regulation-ledger`:
- Misconfiguration of the user's storage layer
- Compromise of the user's private keys
- Compromise of the host operating system
- Side-channel attacks on the user's hardware
