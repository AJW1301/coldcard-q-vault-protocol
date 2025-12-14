# Backup and Storage (Abstract Model)

This document defines **what artifacts exist**, **what each artifact enables**, and **where artifacts should be stored** under the zoning-based 2-of-3 multisig model. The goal is to maximize survivability while keeping day-to-day operation simple.

This is a **model-level** document. It does not prescribe specific products, media brands, or software.

---

## 1. Artifact Inventory

This model assumes the following categories of artifacts exist.

### 1.1 Signing Components

- **Signer devices**: three hardware signing devices (Key A, Key B, Key C)

### 1.2 Key Backups and Access Material

- **Seed phrase backups**: one seed phrase backup per key
- **Passphrase backup**: stored separately from digital recovery materials
- **Optional (convenience)**: an **encrypted passphrase file on removable media** (e.g., microSD), to avoid typing the passphrase in public signing locations (such as a bank). Store this separately from seed backups. *(Non-normative: some devices, such as COLDCARD, support this workflow.)*
- **Device access credentials**: PINs/passwords required to unlock each signing device

### 1.3 Wallet State and Metadata

This model treats the following as sensitive metadata:

- **Descriptor / wallet configuration**: enough information to reconstruct the multisig wallet in coordinator software. If each signer supports multisig registration and/or exporting the multisig wallet configuration, the descriptor/policy effectively exists in multiple places already (the coordinator wallet plus each registered signer). This only applies if the hardware wallet supports exporting or retaining the multisig wallet configuration.
  - **Backup approach:** keep **one additional encrypted backup** (e.g., in cloud storage). Store only the **pointer/link** to that encrypted backup in the Master Index.

- **Encrypted wallet backups**: do not enable spending by themselves. If encryption is sound and the decryption secret is not exposed, the contents should not leak privacy. Treat them as sensitive because loss of the decryption secret compromises privacy, and because non-content metadata (existence, filenames, timestamps, size) may still be observable.
  - **Encryption strength requirement:** the decryption secret (passphrase/key) should provide **at least the same bits of entropy as the seed phrase**; otherwise the encrypted backups become the weakest link.

- **Recovery instructions**: step-ordered guidance for recovery and inheritance

> *Non-normative note:* Hardware wallets known to support exporting or retaining multisig wallet configuration include COLDCARD Q and Blockstream Jade.

---

## 1.4 Documentation

- **Master Index**: a concise map of location codes and what each location contains, including pointers/links to encrypted backups (e.g., the encrypted descriptor backup)

---

## 2. What Each Artifact Enables

This section defines the blast radius if an artifact is lost or stolen.

- **Signer device (alone)**: should not enable spending without the passphrase
- **Seed phrase (alone)**: should not enable spending without the passphrase
- **Passphrase (alone)**: does not enable spending without a seed/signing key
- **Seed phrase + passphrase**: enables spending for that key
- **Two keys’ spending capability** (e.g., two seeds + passphrase): enables spending in a 2-of-3 wallet

- **Wallet descriptor / policy state**: enables wallet reconstruction and can leak privacy; should be treated as sensitive metadata

- **Encrypted wallet backups**: do not enable spending by themselves. If encryption is sound and the decryption secret is not exposed, the contents should not leak privacy. Treat them as sensitive because loss of the decryption secret compromises privacy, and because non-content metadata (existence, filenames, timestamps, size) may still be observable.
  - **Encryption strength requirement:** the decryption secret (passphrase/key) should provide **at least the same bits of entropy as the seed phrase**; otherwise the encrypted backups become the weakest link.

- **Documentation / Master Index**: enables recovery coordination and can leak privacy; should be protected accordingly

---

## 3. Storage Rules

The following rules are intended to prevent single-event compromise.

### 3.1 Non-Co-Location Rules (Must Not)

- Do not store **a seed phrase backup** and the **passphrase backup** in the same location
- Do not store **two seed phrase backups** in the same location
- Do not store **two functional signers** in the same location
- Do not store **enough material to satisfy 2-of-3 signing** in the same zone

### 3.2 Acceptable Co-Location

- It is acceptable to store a **seed phrase backup** with its corresponding **signer device**, because the passphrase gates spending. This reduces the number of required secure locations and keeps the system usable for normal users.
- Documentation may be stored with **encrypted** wallet state (descriptor / policy state).
- Device access credentials may be stored with documentation **if separated from seeds and the passphrase**.

---

## 4. Zone Mapping

Zones represent failure domains. Location codes are placeholders; exact distances and jurisdictions are user-defined.

### 4.1 Operating Zone (Zone A)

**ZA-1 (nearby location)**
- Signer device: Key A

**ZA-2 (nearby secure location)**
- Signer device: Key B

Operating Zone locations must be close enough to support routine signing without travel friction.

If you must sign in a public place (e.g., a bank), consider workflows that reduce passphrase exposure (for example, loading the passphrase from encrypted removable media instead of typing it).

### 4.2 Recovery Zone (Zone B)

The Recovery Zone is split into two locations.

**ZB-KEY (key-only location)**

- Signer device: Key C
- Seed phrase backup: Key C

**ZB-KNOWLEDGE (knowledge and recovery location)**

To keep Zone B simple, treat this location as a **Recovery Kit** with clear separation between *instructions* and *encrypted data*.

Physical:

- **Passphrase backup (separate)**: stored physically (not inside the Recovery Packet). Recommended: **stamp the passphrase onto steel washers** for durability.

M-DISC + Cloud (instructions only):

- **Recovery Packet (single file)** stored in **three copies**: **two M-DISCs** and **one encrypted cloud upload**.
- The Recovery Packet contains only:
  - Recovery instructions
  - Device access notes (e.g., where PINs/passwords are stored)
  - Descriptor export notes (what exists on signers/coordinator)
  - Cloud pointers/links and verification checksums

microSD (encrypted data files):

- **Encrypted wallet backups** stored as separate files (not embedded inside the Recovery Packet)
- (Optional) **encrypted passphrase file(s)** for Key C, if used
- Include a checksum file (hashes) for verification

Design goal: the Recovery Zone should enable recovery even if the Operating Zone is totally lost.

---

## 5. Backup Media and Durability

Seed backups:

- Stamp seed phrases onto **M8 304 stainless steel washers**, stored on an **M8 (or 5/16'') bolt with a nut**.

Recovery Packet (instructions only):

- Store the Recovery Packet as **2× M-DISC + 1× encrypted cloud copy**.

Encrypted wallet backups (data files):

- Store encrypted wallet backups as **separate files on microSD** (not embedded inside the Recovery Packet).
- Include a **checksum file** (e.g., `SHA256SUMS.txt`) to detect corruption or tampering.
- Copy the checksum file into the **Recovery Packet** as a redundant reference.

M-DISC integrity:

- Store a checksum for the Recovery Packet on each M-DISC (e.g., `RECOVERY_PACKET.sha256`).
- Keep the same checksum alongside the encrypted cloud copy.

---

## 6. Verification and Drills

Backups are only useful if they are correct.

- Perform periodic checks that:
  - all location codes still exist and remain accessible
  - devices can be unlocked
  - the passphrase backup matches the passphrase used by each signer
  - documentation remains readable and current

Drills should prioritize **verification without exposure**: confirm correctness without unnecessarily re-entering secrets on computers.
