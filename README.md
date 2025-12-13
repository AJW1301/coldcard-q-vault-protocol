
[multisig-zoning-model-README.md](https://github.com/user-attachments/files/24146533/multisig-zoning-model-README.md)
# Multi-Sig Governance & Resilience Plan

## 1. Purpose and Threat Model

This document describes a **zoning-based 2-of-3 multisig architecture** for individual Bitcoin holders. The goal is to remain secure, recoverable, and operational under realistic adverse conditions.

The design explicitly addresses three common threat classes:

1. **Coercion** – being pressured to sign using assets immediately accessible.
2. **Confiscation by authority** – loss of access to assets within a jurisdiction or failure domain.
3. **Loss** – theft, fire, destruction, or permanent misplacement of devices or backups.

The system is structured so that **no single threat event, location failure, or zone failure results in loss of funds**.

---

## 2. Zoning Model

The core abstraction is **zoning**. A zone represents a **failure domain**, not a specific geography. Distances, jurisdictions, and independence levels are configurable based on the owner’s personal threat model.

The system uses two zones:

### Operating Zone (Zone A)

- The primary signing zone for routine operations
- Holds **two signing keys** (Key A and Key B)
- Keys are distributed across **two distinct locations** that are close enough to allow normal transaction signing
- No single location in the Operating Zone holds enough material to sign

### Recovery Zone (Zone B)

- The survivability and recovery zone
- Exists to guarantee recovery if the Operating Zone is lost
- Composed of **two separate locations**:
  - **Recovery–Key Location:** holds a single signing key (Key C)
  - **Recovery–Knowledge Location:** holds all recovery material

Recovery material includes:

- Encrypted backups of Key A and Key B
- Passwords and user-generated passphrases
- Encrypted wallet descriptors and configuration data
- Documentation and the Master Index (map of all key locations)

The fundamental invariant of the system is:

> **Loss of any single zone does not imply loss of signing capability.**

---

## 3. Security and Design Principles

These principles guide implementation choices. They are defaults, not hard requirements.

### Simplicity and Uniformity

- Signers are implemented using equivalent hardware devices
- Credentials (passphrases and encryption passwords) are standardized
- The objective is to reduce user error and long-term lockout risk

### User-Introduced Entropy

- All critical passphrases and encryption passwords are **user-generated**, not device-generated
- Passphrases are generated using **physical randomness** (e.g., dice rolls) mapped to a **Diceware word list**
- This ensures secrets are:
  - Independent of hardware vendors and wallet standards
  - Auditable and reproducible by the user
  - Resistant to compromised RNGs or biased entropy sources

User-introduced entropy is treated as a **core security property**, not an optional enhancement.

### Cryptography over Obscurity

- All sensitive information is encrypted
- No reliance on hiding, mislabeling, or secrecy of storage media
- Clear labeling is acceptable because security derives from encryption and separation

### Controlled Knowledge Concentration

- All passwords, passphrases, and documentation are intentionally concentrated in the Recovery Zone
- Loss of the Recovery Zone results in **privacy loss**, but not loss of funds

### Memory Usage Rules

- Seeds are never memorized
- Memory is used only for redundancy (e.g., encryption passwords)
- No signing capability depends solely on human memory

### Privacy as a Security Property

- Wallet descriptors and metadata are treated as sensitive
- No plaintext descriptors, XPUBs, or configuration QR codes are stored

---

## 4. Key Inventory and Topology

*Locations are coded. Zone A is the Operating Zone. Zone B is the Recovery Zone.*

| Zone | Key Alias | Signer Type     | Location Code | Role                                        |
| ---- | --------- | --------------- | ------------- | ------------------------------------------- |
| A    | Key A     | Hardware signer | ZA-HOME       | Signing key (location 1)                    |
| A    | Key B     | Hardware signer | ZA-OFFSITE    | Signing key (location 2, secure)            |
| B    | Key C     | Hardware signer | ZB-HOME       | Recovery signing key                        |
| B    | A+B       | Encrypted media | ZB-OFFSITE    | Encrypted backups, passwords, documentation |

---

## 5. Operational Policy

- Routine spending occurs within the Operating Zone
- Signing requires access to two distinct Operating Zone locations
- Recovery Zone keys are not used for day-to-day operations

---

## 6. Maintenance Policy

- **Schedule:** Quarterly
- **Scope:** Signer health checks, software updates, recovery review, memory verification

---

## 7. Recovery Policy

This repository documents structure only; it never stores secrets.

If the Operating Zone is lost:

- Use Key C and recovery material from the Recovery Zone to reconstruct a second signing key
- Resume 2-of-3 signing entirely from the Recovery Zone

If the Recovery Zone is lost:

- Funds remain spendable using the Operating Zone
- Privacy and recovery redundancy are reduced

---

## 8. Final Note

This system is designed to balance **survivability and convenience**. It prioritizes privacy, recoverability, and resistance to catastrophic loss while remaining practical for routine use within the Operating Zone.
