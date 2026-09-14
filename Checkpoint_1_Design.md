# Checkpoint 1: Password Manager Design Document

## 1. System Architecture & Data Flow
The architecture of the password manager consists of three main components:
*   **User-Management Module**: Handles user authentication, session management, and master password verification[cite: 1].
*   **Encryption Module**: Responsible for cryptographic operations, utilizing Python's `cryptography` library to encrypt and decrypt vault entries[cite: 1].
*   **Storage Layer**: A local SQLite database that stores user accounts and encrypted vault data[cite: 1].

**Data Flow:**
1. The user inputs their master password via the interface.
2. The User-Management module derives a key using a Key Derivation Function (KDF) and verifies it.
3. The encrypted password blob is fetched from the Storage Layer.
4. The Encryption module decrypts the blob in memory using the derived key.
5. The plaintext is sent to the Interface and displayed.

## 2. Threat Model
*   **Master Password**:
    *   *Threat*: Brute-force and dictionary attacks.
    *   *Mitigation*: Use a memory-hard KDF like PBKDF2HMAC with a unique, randomly generated salt per user.
*   **Vault at Rest**:
    *   *Threat*: An attacker gains access to the SQLite database file and attempts to read stored credentials.
    *   *Mitigation*: All vault entries are encrypted using AES-256-GCM. The encryption key is derived entirely from the master password and is never stored on disk.
*   **Vault in Memory**:
    *   *Threat*: Memory dumping or cold-boot attacks exposing plaintext passwords or keys while the application is running.
    *   *Mitigation*: Minimize the lifetime of sensitive variables by explicitly overwriting or clearing variables holding plaintext keys immediately after decryption.
*   **Interface**:
    *   *Threat*: Man-in-the-Middle (MitM) attacks or XSS leading to credential interception.
    *   *Mitigation*: Enforce HTTPS/TLS for all traffic, use strict Content Security Policy (CSP), and implement CSRF tokens.

## 3. Initial Design Decision
*   **Vault Format**: A relational database (SQLite). The `Users` table stores `username`, `password_hash`, and `salt`. The `Vault` table stores `service_name` (plaintext) and `encrypted_password` (ciphertext + nonce/IV)[cite: 1].
*   **Cryptographic Scheme**: `PBKDF2HMAC` for key derivation and `AES-256` in `GCM` (Galois/Counter Mode) for authenticated encryption[cite: 1].
