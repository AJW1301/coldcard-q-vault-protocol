# A Decoupled Method for Multisig Spending and Shamir Recovery

This document describes a **customizable single-user custody model** that combines:

- **Multisig** for day-to-day spending control
- **SLIP-39 (Shamir)** for distributed backup and recovery
- **BIP85** for deterministic provisioning of multiple signer keys

When combined, these methods let you choose:

- the number of **active signing keys** (multisig), and
- the number of **backup shares** (Shamir),

without coupling the two into a single threshold.

---

## 1. Problem statement

Bitcoin self-custody increasingly faces two physical-world risks: **coercion** (forced signing) and **confiscation** within a user’s residential jurisdiction. Multisignature wallets are a common mitigation because they can require multiple keys to authorize spending.

However, for a single user, the goals of **coercion resistance** and **backup redundancy** are often coupled in standard multisig practice. Increasing the number of signers (N) can improve survivability under loss, but it also increases operational complexity and may require placing additional signing keys under third-party custody.

This model decouples these concerns:

- Use a **multisig policy (m-of-n)** for spending.
- Use a **Shamir policy (t-of-k)** for backup and recovery.

Example: operate day-to-day with a **2-of-2** signing policy for coercion resistance, while using a **3-of-5** Shamir backup policy for recoverability.

---

## 2. System Topology

This section defines the structure of the model: what secrets exist, how they are derived, and what policies control spending vs recovery.

### 2.1 Parameters

- **Signing policy (multisig):** *m-of-n*
- **Recovery policy (Shamir):** *t-of-k*
- **Signer index set:** *I* (use simple indices such as `1, 2, 3, ...`)

### 2.2 Secrets and artifacts

- **Root secret (master seed):** `S`
- **Shamir shares:** `sh_1..sh_k`, created from `S` under *t-of-k*
- **Derived signer seeds:** `s_i = BIP85(S, i)` for each `i ∈ I`
- **Passphrase:** a single user-generated passphrase `p` (Diceware via physical dice) applied to **all** signers
- **Signer keys:** `K_i = f(s_i, p)` (seed + passphrase → signer key)

### 2.3 Workflows

**Signer key derivation (brief):** For each signer index `i`, a signer seed `s_i` is derived from the master seed `S` via BIP85. The signer key `K_i` is then obtained by applying the shared passphrase `p` to `s_i` using the wallet’s standard *seed + passphrase* derivation.

**Provisioning**

1. Generate `S` on a secure device.
2. For each signer index `i ∈ I`, derive `s_i` via BIP85 and initialize signer `K_i` using the shared passphrase `p`.
3. Create the multisig wallet with spending policy *m-of-n* over the signer set `{K_i}`.
4. Split `S` into Shamir shares `sh_1..sh_k` under policy *t-of-k*.
5. **After verifying all shares were created correctly, destroy `S` (the master seed) from the provisioning device.**

**Spending**

- The coordinator constructs PSBTs and tracks wallet state.
- A transaction is spendable only when **m** distinct signers approve and sign the same PSBT.

**Recovery**

- Collect any **t** of **k** Shamir shares to reconstruct `S`, then re-derive signer seeds `s_i` for the required indices and re-initialize the needed signers.

### 2.4 System topology figure

![System topology](docs/figures/figure-system-topology.png)
*Figure 2.4. System topology.* The master seed `S` is split into Shamir shares for recovery and deterministically derives signer seeds via BIP85. Multisig defines the spending threshold (*m-of-n*), while Shamir defines the recovery threshold (*t-of-k*).

---

## 3. Security Properties

This model has two independent control thresholds:

- **Recovery threshold (Shamir):** *t-of-k* shares to reconstruct the master seed `S`.
- **Spending threshold (multisig):** *m-of-n* signer keys to authorize a spend.

Security is governed by the **easier** of these two compromise paths (assuming the attacker also knows the shared passphrase `p`).

### 3.1 Shamir threshold baseline

Because `S` can re-provision signer material, a **1-of-1** backup of `S` creates a single catastrophic weakness: compromise of that one backup is sufficient to reconstruct `S`.

**Recommendation:** start from **2-of-3** Shamir (or stronger). Choose *t* and *k* so that reaching *t* backups is at least as hard as reaching *m* signer keys.

### 3.2 Compromise paths

Assuming the attacker knows `p`, funds are at risk if they can obtain **either**:

- **t-of-k Shamir shares** → reconstruct `S` → re-derive signer seeds via BIP85 → recreate enough signer keys, **or**
- **m-of-n signer keys** → satisfy the wallet’s multisig spending policy directly.

**Example:** with a 2-of-2 wallet, stealing one signer is insufficient. But if `S` is stored as a **1-of-1** backup, stealing that single backup may be easier than stealing two signers—making the backup the weak point.

### 3.3 What is not a security control

- **BIP85 indices are not encryption.** They are deterministic selectors. If `S` is compromised, indices can be tried and matched.

---

## 4. Storage Layout Design Principles

This section provides principles for placing devices, backups, and recovery artifacts across locations. The core abstraction is **zoning**: grouping locations into failure domains that align with your coercion and confiscation threat model.

### 4.1 Zones

#### Zone A — Primary signing zone

Zone A is your **current residential jurisdiction**. It contains **two or more signer locations** within the same jurisdiction, chosen so that:

- signing can be completed within a reasonable time window, and
- no single location compromise yields enough material to spend.

Zone A may also hold **some** Shamir shares, provided that no single Zone A location becomes a catastrophic recovery point.

#### Zone B — Primary knowledge zone

Zone B is your **primary knowledge and recovery zone**. Zone B should have **two or more locations** and is intended to hold recovery-critical knowledge and artifacts, including:

- device passwords / PINs
- the shared passphrase `p`
- recovery instructions

Users choose how **independent** Zone B should be from Zone A (distance, jurisdictional separation, and trust boundary) based on their personal threat model.

### 4.2 Layout design constraints

Treat the user’s presence and memory (“the brain”) as a security-relevant component. The layout is designed to satisfy two constraints:

**Constraint A — No single location + user present (local coercion resistance)**

Even if the user is physically present and can recall all needed information, it must not be possible to spend from a **single location**. Spending should require access to **two or more distinct locations** to assemble the multisig signing threshold.

This constraint targets **local coercion** where the adversary can control **one location** (and potentially the user at that location).

**Constraint B — No single zone without the user (jurisdictional confiscation resistance)**

If the user is not present, material contained within **Zone A alone** or **Zone B alone** must be insufficient to spend. An adversary should need information from **both zones** to recover enough capability to spend.

This constraint primarily targets **confiscation/seizure risk within the user’s current residential jurisdiction**.

### 4.3 Limitation: jurisdiction-level coercion

Assume a high-capability adversary can detain the user and access **all locations within Zone A**. Under this threat, “two locations to spend” inside the same zone does not help: the adversary can obtain whatever is needed across the zone, including user-held information.

Constraint A does not address jurisdiction-level coercion. Resisting it generally requires that spending cannot be completed using only materials located inside the adversary-controlled jurisdiction (i.e., at least one critical element must be outside Zone A).

### 4.4 Storage rules for critical information

For critical information, follow a **3-2-1 rule**:

- **3 copies** total
- stored on **2 different media**
- with **1 copy off-site**

**Documents (recovery instructions, descriptor backup, wallet backup metadata):**

- Maintain **three copies** in total:
  - **two physical copies** stored in **two distinct locations**, and
  - **one encrypted off-site copy** in cloud storage.

**Passphrase `p`:**

- Treat the user’s **brain** as one “copy” (memorized passphrase).
- Maintain **two additional physical copies** stored in **two distinct locations**.
- The passphrase must **not** be stored in any digital form, including cloud storage.

### 4.5 Example layout (non-normative)

This example illustrates one concrete layout that satisfies **Constraint A** and **Constraint B**.

- **Multisig signing policy:** 2-of-2
- **Shamir recovery policy:** 2-of-3
- **Signers:** two COLDCARD Q devices (example hardware wallet)
- **Passphrase:** one shared Diceware passphrase `p` used across both signers
- **Device backups:** each COLDCARD Q is co-located with its own microSD backup

**Zone A — Primary signing zone (same jurisdiction)**

- **ZA-1 (Location A1):**
  - Signer `K1` on COLDCARD Q
  - microSD backup co-located with device
  - Shamir share `sh1`

- **ZA-2 (Location A2):**
  - Signer `K2` on COLDCARD Q
  - microSD backup co-located with device
  - Shamir share `sh2`

**Zone B — Primary knowledge zone (two locations)**

- **ZB-1 (Location B1):**
  - Physical copy of passphrase `p`
  - “Recovery packet” (documents): recovery instructions, descriptor backup/export, encrypted wallet backup metadata, and integrity checksums

- **ZB-2 (Location B2):**
  - Second physical copy of passphrase `p`
  - Second physical copy of the recovery packet
  - Shamir share `sh3` (note: Zone B still has < *t* shares)

**Cloud (off-site, encrypted):**

- One encrypted copy of the recovery packet (documents only; **no passphrase**)

**Why this satisfies the constraints**

- **Constraint A:** even with the passphrase in mind, spending requires visiting **two Zone A locations** to use both signers.
- **Constraint B:** without the user, **Zone A alone** lacks the passphrase, and **Zone B alone** lacks signers; both zones are required to recover spending capability.

---

## Closing note

This document describes a method to **decouple the signing policy** (multisig *m-of-n*) from **backup redundancy** (Shamir *t-of-k*).

By allowing a larger *k* for backups, each individual share becomes less critical to overall survivability, which can reduce the storage burden placed on any single share.

The method also supports a **2-of-2** multisig configuration, allowing a single user to keep custody fully personal while still requiring multiple devices to spend.

## Further reading

- **Wallet preparation:** `docs/wallet-preparation.md`
- **Storage layout worksheet / AI prompt:** `docs/layout-design-prompt.md`
- **Signing procedure:** `docs/signing-procedure.md`
- **Recovery procedure:** `docs/recovery.md`
