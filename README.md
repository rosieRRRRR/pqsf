# PQSF – Post-Quantum Security Framework

* **Specification Version:** 2.0.2
* **Status:** Public beta
* **Date:** 2026
* **Author:** rosiea
* **Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea

---

## Summary

PQSF provides the foundational protocol primitives for the PQ stack: canonical encoding, cryptographic suite indirection, deterministic object grammars, session binding, and encrypted-before-transport wrappers. It eliminates ambiguity at the protocol layer through deterministic CBOR and JCS Canonical JSON encoding, enabling byte-stable hashing and signing. CryptoSuiteProfile indirection allows algorithm transitions without specification rewrites. Exporter binding primitives enable session-scoped security without trusting networks. PQSF is deliberately minimal—it defines grammar, encoding, and validation rules only. All enforcement and authority semantics live in consuming specifications.

**Key Properties:** Canonical encoding discipline | Crypto-agility via profiles | Session binding primitives | EBT artefact protection | Zero enforcement semantics | Protocol-layer foundation

---

## Index

1. [Summary](#summary)
2. [Non-Normative Overview](#non-normative-overview--for-explanation-and-orientation-only)
3. [Scope and Protocol Boundary](#scope-and-protocol-boundary)
4. [Non-Goals and Authority Prohibition](#non-goals-and-authority-prohibition)
5. [Architecture Overview](#architecture-overview)
6. [Explicit Dependencies](#expliciT-dependencies)
7. [Canonical Encoding](#canonical-encoding)
   1. [PQSF-Native Canonical Encoding (Deterministic CBOR Only)](#pqsf-native-canonical-encoding-deterministic-cbor-only)
   2. [Strict CBOR Profile](#strict-cbor-profile)
   3. [Fuzzing Resistance via Canonical Encoding](#fuzzing-resistance-via-canonical-encoding)
   4. [Epoch Clock Canonical Encoding Exception (JCS Canonical JSON Only)](#epoch-clock-canonical-encoding-exception-jcs-canonical-json-only)
8. [CryptoSuiteProfile Indirection](#cryptosuiteprofile-indirection)
9. [Hashing Interface](#hashing-interface)
10. [Signature Interface](#signature-interface)
11. [Domain Separation](#domain-separation)
12. [Deterministic Derivation](#deterministic-derivation)
13. [Universal Secret Derivation](#universal-secret-derivation)
14. [Epoch Clock Integration Objects](#epoch-clock-integration-objects)
15. [Exporter Binding Primitive](#exporter-binding-primitive)
16. [ConsentProof Grammar](#consentproof-grammar)
17. [Policy Objects and Governance Metadata](#policy-objects-and-governance-metadata)
18. [LedgerEntry Grammar](#ledgerentry-grammar)
19. [Merkle Ledger Serialization](#merkle-ledger-serialization)
20. [RecoveryCapsule Grammar](#recoverycapsule-grammar)
21. [Encrypted-Before-Transport Wrapper](#encrypted-before-transport-wrapper)
22. [Supply Chain Artefact Types](#supply-chain-artefact-types)
23. [Supply Chain Ledger Anchoring](#supply-chain-ledger-anchoring)
24. [Reproducible Build Support](#reproducible-build-support)
25. [No Trust in Network Location](#no-trust-in-network-location)
26. [Sovereign Transport Protocol Grammar](#sovereign-transport-protocol-grammar-stp)
27. [Failure Semantics](#failure-semantics)
28. [Conformance](#conformance)
29. [Security Considerations](#security-considerations)
30. [Annexes](#annexes)
31. [Changelog](#changelog)

---

## Non-Normative Overview — For Explanation and Orientation Only

**This section is NOT part of the conformance surface.  
It is provided for explanatory and onboarding purposes only.**

### Plain Summary

PQSF defines the foundational protocol primitives used across the PQ
stack. It specifies canonical encoding rules, cryptographic suite
indirection, deterministic object grammars, and session binding
primitives. PQSF provides infrastructure only and grants no authority.

### What PQSF Is / Is Not

| PQSF IS | PQSF IS NOT |
|---------|-------------|
| A protocol grammar and ruleset | An enforcement engine |
| A canonical encoding specification | An authority source |
| A crypto agility framework | A concrete algorithm selector |
| A session binding primitive provider | A transport protocol |

### Canonical Flow (Single Line)

Artefact → Canonical Encoding → Profile-Governed Cryptography → Verification

### Why This Exists

PQSF exists to eliminate ambiguity at the protocol layer. Canonical
encoding prevents parser and representation attacks. CryptoSuiteProfile
indirection enables algorithm agility without rewriting specifications.
Exporter binding primitives allow session-scoped security without
trusting networks or intermediaries. PQSF is deliberately minimal and
foundational; all authority and enforcement semantics live elsewhere.

---

## 1. Scope and Protocol Boundary

PQSF defines deterministic, post-quantum protocol primitives only.

PQSF normatively defines:

* canonical encoding rules for protocol artefacts
* cryptographic suite indirection via pinned profile references
* deterministic hashing, signing, and derivation interfaces
* exporter-bound session binding primitives
* sovereign transport message grammars
* deterministic object grammars for protocol artefacts
* deterministic ledger entry format and Merkle serialization rules
* encrypted-before-transport artefact wrapping rules
* policy object structure and governance metadata

**Protocol Boundary:**
PQSF defines protocol grammar, canonical encoding, and structural validation requirements only.

**Enforcement Boundary:**
PQSF does not perform enforcement, gating, admission, refusal, escalation, lockout, freshness enforcement, monotonicity enforcement, or authority decisions. All such behaviour is defined exclusively by consuming specifications, including PQSEC and domain specifications.

**Non-Protocol Behaviour:**
Any implementation using PQSF primitives for enforcement, refusal, escalation, lockout, freshness enforcement, monotonicity enforcement, or authority decisions outside PQSEC is architecturally non conformant.

---

## 2. Non Goals and Authority Prohibition

PQSF does not define:

* predicate enforcement, refusal, escalation, lockout, backoff, or freeze semantics
* runtime integrity attestation semantics or drift classification semantics
* custody tiers, signing authority, quorum authority, or PSBT authority semantics
* AI behavioural gating, action class admission control, or model authority semantics
* Epoch Clock anchoring, issuance, mirror consensus enforcement, tick freshness enforcement, or tick reuse enforcement
* application business logic, UI behaviour, or user workflows

**Authority Prohibition:**
PQSF grants no authority, makes no decisions, and performs no enforcement. PQSF defines encodings, structures, and validation rules only. Authority derives exclusively from PQSEC enforcement of PQSF-defined artefacts.

---

## 3. Threat Model

PQSF assumes adversaries may:

* replay, reorder, or substitute protocol messages
* inject non canonical encodings
* exploit ambiguity in serialization
* attempt downgrade via alternate encodings or cryptographic identifiers
* attempt cross-session artefact reuse
* attempt partial object substitution inside larger payloads
* attempt metadata correlation through transport or storage intermediaries

PQSF does not assume trusted networks, trusted clocks, trusted runtimes, trusted coordinators, or trusted storage providers.

---

## 4. Trust Assumptions

PQSF operates under the following trust assumptions:

* cryptographic verification is performed locally by consumers
* canonical encoding eliminates ambiguity in hashing and signing
* exporter binding derives from authenticated transport session secrets
* time artefacts and profile lineage are external inputs
* enforcement, authority, and admission decisions are external to PQSF

---

## 5. Architecture Overview

PQSF defines a protocol substrate consisting of:

* **Canonical Encoding Layer**
  Deterministic CBOR strict profile for all PQSF-native signed, hashed, or compared objects.

* **External Canonical Artefact Exception**
  Epoch Clock profiles and ticks are externally canonicalized and MUST use JCS Canonical JSON exclusively. They are not PQSF-native CBOR objects.

* **Cryptographic Suite Indirection Layer**
  All cryptographic operations reference pinned CryptoSuiteProfile objects.

* **Protocol Object Layer**
  Canonical object grammars for PQSF-native protocol artefacts and wrappers.

* **Transport Binding Layer**
  Exporter hash derivation and binding fields for session-bound artefacts.

* **Sovereign Transport Grammar Layer**
  Deterministic STP message grammar.

* **Ledger Serialization Layer**
  Deterministic Merkle construction rules.

PQSF defines object structure and validation rules only. PQSF does not define operational behaviour.

---

## 5A. Explicit Dependencies

| Specification | Minimum Version | Purpose |
|---------------|-----------------|---------|
| Epoch Clock | ≥ 2.1.1 | JCS Canonical JSON artefact handling exception |

PQSF is foundational infrastructure. Most PQ specifications depend on PQSF; PQSF depends only on Epoch Clock for the canonical JSON encoding exception.

---

## 6. Conformance Keywords

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119.

---

## 7. Canonical Encoding

### 7.1 PQSF-Native Canonical Encoding (Deterministic CBOR Only)

1. All PQSF-native protocol objects that are hashed, signed, compared, or used in Authoritative decisions MUST be encoded using Deterministic CBOR.
2. The CBOR encoding MUST follow RFC 8949 deterministic encoding rules with the additional strict constraints defined in Section 7.2.
3. Re-encoding a decoded object MUST produce byte-identical output.
4. Non canonical encodings MUST be rejected.
5. JSON representations MAY be used only for unsigned display or transport wrappers and MUST NOT be hashed or signed.

### 7.2 Strict CBOR Profile

The following constraints are mandatory:

1. Integer values MUST use the shortest encoding form.
2. Floating-point values MUST NOT appear in any canonical artefact.

   **Rationale:**
   * IEEE 754 floating-point has multiple binary representations:
     - NaN has 2^52 distinct bit patterns
     - Positive zero (+0.0) and negative zero (−0.0) are distinct
     - Subnormal numbers and rounding behaviour vary by platform
   * These properties violate byte-identical re-encoding requirements.
   * Non-deterministic encoding breaks hashing, signing, and verification.

   **Permitted Alternatives:**
   * **Fixed-point integers:** value + scale (e.g., 234 / 10000 = 0.0234)
   * **Rational numbers:** numerator + denominator
   * **Integer units:** smallest unit representation (e.g., milliseconds instead of seconds)

   **Error Code:** `E_ENCODING_NONCANONICAL`

   **Exceptions:** None. Floating-point values are forbidden in all
   PQSF-native canonical CBOR artefacts, including informational fields.
3. Indefinite-length items MUST NOT be used.
4. Map keys MUST be unique. Duplicate keys MUST cause rejection.
5. Map key ordering MUST be bytewise lexicographic over the encoded key bytes.
6. Text and byte strings are distinct and MUST NOT be coerced.
7. CBOR tags MUST NOT be used unless explicitly permitted by a consuming specification profile.


## 7.3 Fuzzing Resistance via Canonical Encoding

### 7.3.1 Canonical Encoding as Fuzzing Defense

Deterministic canonical encoding is PQSF’s primary defense against
structural fuzzing attacks.

**Attack Model:**
Adversaries may supply semantically equivalent but byte-different
encodings to exploit:
- parser ambiguities
- hash or signature mismatches
- state-machine edge cases
- inconsistent error handling

**Defense Strategy:**
Canonical encoding collapses all semantic equivalents into a single
byte representation. Structural ambiguity is eliminated as an attack
surface.

---

### 7.3.2 Eliminated Encoding Fuzzing Vectors

**JSON Key Reordering**
```json
{"a": 1, "b": 2}
{"b": 2, "a": 1}
```

Mitigation: CBOR map keys MUST be sorted bytewise lexicographically.

---

**CBOR Length Encoding Variants**

```
18 42      // integer 66 (short form)
19 00 42   // integer 66 (long form)
```

Mitigation: Only shortest encoding form is permitted.

---

**Floating-Point Variants**

```
+0.0, -0.0, NaN, subnormals
```

Mitigation: Floating-point values are forbidden entirely.

---

**Indefinite-Length Encoding**

```
9F 01 02 03 FF   // indefinite-length array
83 01 02 03      // definite-length array
```

Mitigation: Indefinite-length items MUST be rejected.

---

**Unicode Normalization**

```
"café"  // NFC vs NFD representations
```

Mitigation: CBOR text strings are raw byte sequences; no protocol-level
normalization occurs.

---

### 7.3.3 Canonical Encoding Test Requirements

Implementations MUST pass the following tests:

```python
def test_reencoding_stability():
    original = canonical_encode(obj)
    decoded = canonical_decode(original)
    reencoded = canonical_encode(decoded)
    assert original == reencoded
```

```python
def test_noncanonical_rejection():
    noncanonical = bytes([0x19, 0x00, 0x42])
    with pytest.raises(EncodingError):
        canonical_decode(noncanonical)
```

```python
def test_cross_implementation_consistency():
    a = impl_a.encode(obj)
    b = impl_b.encode(obj)
    assert a == b
```

---

### 7.3.4 Conformance

PQSF conformant implementations MUST:

* ☐ Produce byte-identical encodings for identical semantic input
* ☐ Reject all non-canonical encodings
* ☐ Enforce shortest encoding form
* ☐ Reject indefinite-length items
* ☐ Reject floating-point values
* ☐ Enforce deterministic map key ordering
* ☐ Pass all canonical encoding test vectors


### 7.4 Epoch Clock Canonical Encoding Exception (JCS Canonical JSON Only)

1. Epoch Clock profiles and ticks MUST be processed, hashed, and verified in their JCS Canonical JSON byte form exclusively.
2. Epoch Clock artefacts are externally canonicalized.
3. Epoch Clock artefacts MUST NOT be re-encoded into CBOR or any other format for the purposes of hashing, signing, comparison, or signature verification.
4. Any PQSF-native object that needs to bind to an Epoch Clock profile or tick MUST bind using either:

   * the Epoch Clock artefact bytes directly, or
   * a hash computed over the Epoch Clock artefact bytes under a referenced hash-class CryptoSuiteProfile.
5. Transport carriage MAY embed Epoch Clock artefacts as opaque bytes. Parsing is permitted for routing or display only and MUST NOT alter the verified byte identity.

---

## 8. CryptoSuiteProfile Indirection

PQSF replaces all hardcoded cryptographic algorithm identifiers with profile references.

### 8.1 CryptoSuiteProfile Definition

```
CryptoSuiteProfile = {
  profile_id: tstr,
  suite_class: "signature" / "hash" / "kem" / "aead" / "derivation",
  family: tstr,
  parameters: { * tstr => any },
  domain_set_id: tstr,
  test_vector_hash: bstr,
  valid_from_tick: uint,
  valid_until_tick: uint / null,
  revoked: bool,
  signature: bstr
}
```

### 8.2 Profile Rules

1. All cryptographic operations MUST reference a CryptoSuiteProfile by profile_id.
2. Profiles MUST be signed and pinned by hash.
3. Unknown, expired, or revoked profiles MUST cause rejection.
4. Profile downgrade is forbidden. A regressed profile is one whose security properties, parameter strength, or effective validity window are strictly weaker than a previously accepted profile within the same profile lineage.
5. Profile validity evaluation, including tick interpretation, is performed by consuming specifications. PQSF treats profiles as inert data structures only.
6. Profile revocation state MUST be discoverable via authenticated profile distribution or authenticated revocation artefacts defined by consuming specifications.
7. Profile revocation MUST be honoured immediately for Authoritative operations.

### 8.3 Profile Distribution Architecture

#### 8.3.1 Distribution Mechanisms

Profile distribution provides the means for implementations to discover and update CryptoSuiteProfiles.

1. **Bootstrap profiles** MUST be hardcoded in implementations.
2. **Profile updates** MUST be signed by governance keys.
3. **Profile discovery** MAY use:
   * HTTPS endpoints with certificate pinning
   * Gossip protocols (with signature verification)
   * Local profile cache with update mechanism

#### 8.3.2 Profile Update Protocol

The following protocol enables secure profile updates:

1. Client queries known distribution endpoints for profile updates.
2. Distribution endpoint returns a ProfileUpdate artefact:

```
ProfileUpdate = {
  profile: CryptoSuiteProfile,
  governance_sigs: [* {
    signer_id: tstr,
    sig: bstr
  }],
  valid_from_tick: uint,
  update_priority: "CRITICAL" / "RECOMMENDED" / "OPTIONAL"
}
```

3. Client verifies M-of-N governance signatures over canonical ProfileUpdate payload.
4. Client validates profile compatibility (no downgrade per Section 8.2).
5. Client stages profile for activation at valid_from_tick.
6. At valid_from_tick, client activates new profile for all future operations.

#### 8.3.3 Profile Update Validation

Before accepting a ProfileUpdate:

1. Verify that governance_sigs contains at least M valid signatures.
2. Verify each signature against known governance public keys.
3. Verify that profile.valid_from_tick >= current_tick.
4. Verify that profile does not regress security properties.
5. Verify that profile.signature is valid (self-signature by profile authority).

### 8.4 Profile Revocation Handling

Profile revocation enables the invalidation of compromised or deprecated cryptographic profiles.

#### 8.4.1 ProfileRevocation Artefact

```
ProfileRevocation = {
  revoked_profile_id: tstr,
  revoked_at_tick: uint,
  reason: tstr,
  superseded_by_profile_id: tstr / null,
  governance_sigs: [* {
    signer_id: tstr,
    sig: bstr
  }]
}
```

#### 8.4.2 Revocation Requirements

1. Revocations MUST be signed by M-of-N governance keys.
2. Revocations MUST be checked before each Authoritative operation.
3. If superseded_by_profile_id is present, clients SHOULD automatically upgrade to the replacement profile.
4. Grace period: 48 hours from revoked_at_tick for Non-Authoritative operations.
5. Authoritative operations MUST refuse immediately upon revocation discovery.

#### 8.4.3 Revocation Discovery

Implementations MUST:

1. Check for revocations at startup.
2. Periodically poll revocation endpoints (recommended: every 3600 seconds).
3. Cache revocation list locally.
4. Accept revocation push notifications from distribution infrastructure where available.

### 8.5 Emergency Cryptographic Transition

Emergency transitions address catastrophic cryptographic failures.

#### 8.5.1 Emergency Profile Requirements

In case of algorithm compromise:

1. **Emergency Profile Creation:**
   * CAN violate non-downgrade rule if compromise is cryptographically proven
   * MUST be signed by emergency_quorum (all governance keys, no threshold)
   * MUST include compromise_evidence_hash referencing public disclosure

2. **Emergency Profile Structure:**

```
EmergencyProfile = {
  profile: CryptoSuiteProfile,
  replaces_profile_id: tstr,
  compromise_evidence_hash: bstr,
  emergency_justification: tstr,
  transition_deadline_tick: uint,
  governance_sigs: [* {
    signer_id: tstr,
    sig: bstr
  }]
}
```

#### 8.5.2 Transition Period

Emergency transitions follow a phased approach:

1. **Phase 1: Dual-Algorithm Mode (Days 0-7)**
   * Accept both old (compromised) and new algorithms
   * Log all uses of old algorithm
   * Warn users about pending deprecation

2. **Phase 2: Authoritative Restriction (Days 7-30)**
   * Reject old algorithm for Authoritative operations
   * Continue accepting for Non-Authoritative operations
   * Escalate user warnings

3. **Phase 3: Complete Deprecation (Day 30+)**
   * Reject old algorithm for all operations
   * Remove old algorithm support from codebase

#### 8.5.3 Rollback Protection

To prevent reactivation of compromised profiles:

1. Compromised profiles MUST be added to rollback_denylist immediately.
2. Profiles in rollback_denylist CANNOT be reactivated even with valid signatures.
3. rollback_denylist MUST be signed by emergency_quorum.
4. rollback_denylist additions are permanent and cannot be removed.

### 8.6 Profile Compatibility and Version Negotiation

#### 8.6.1 Compatibility Rules

When evaluating profile updates, implementations MUST apply these compatibility rules:

**Signature Algorithm Families:**
* ML-DSA-65 → ML-DSA-87: PERMITTED (stronger)
* ML-DSA-87 → ML-DSA-65: FORBIDDEN (downgrade)
* ML-DSA-* → ECDSA: FORBIDDEN (quantum-vulnerable)
* ECDSA → ML-DSA-*: PERMITTED (upgrade to PQ)

**Hash Algorithm Families:**
* SHAKE256-256 → SHAKE256-512: PERMITTED (stronger)
* SHAKE256-512 → SHAKE256-256: FORBIDDEN (downgrade)
* SHAKE256-* → SHA256: FORBIDDEN (quantum concerns)
* SHA256 → SHAKE256-*: PERMITTED (upgrade to PQ)

**KEM Algorithm Families:**
* ML-KEM-768 → ML-KEM-1024: PERMITTED (stronger)
* ML-KEM-1024 → ML-KEM-768: FORBIDDEN (downgrade)
* ECDH → ML-KEM-*: PERMITTED (upgrade to PQ)
* ML-KEM-* → ECDH: FORBIDDEN (quantum-vulnerable)

#### 8.6.2 Mixed-Version Operation

During transition periods:

1. Implementations MUST support reading both old and new profile formats.
2. Implementations MUST write using the newest acceptable profile.
3. Verification MUST accept any non-revoked profile within valid_from_tick/valid_until_tick window.
4. If multiple valid profiles exist, prefer the profile with the latest valid_from_tick.

---

### 8.6.3 Decentralized Governance Protocol

#### 8.6.3.1 Governance Model

CryptoSuiteProfile updates MUST be governed by a distributed threshold
signature scheme to prevent single-authority compromise.

```

GovernanceConfig = {
governance_id: tstr,
governance_members: [* GovernanceMember],
threshold_m: uint,
total_n: uint,
valid_from_tick: uint,
governance_policy_hash: bstr,
signature: bstr
}

GovernanceMember = {
member_id: tstr,
public_key_pq: bstr,
organization: tstr,
role: "proposer" / "approver" / "auditor",
added_at_tick: uint,
valid_until_tick: uint / null
}

```

**Requirements:**
1. `threshold_m` MUST be ≥ ⌈(total_n / 2)⌉ + 1.
2. Governance members MUST represent at least three independent organizations.
3. GovernanceConfig MUST be signed by the active governance threshold or bootstrap authority.
4. Governance metadata defines legitimacy only and carries no enforcement authority.

#### 8.6.3.2 Profile Update Proposal Workflow

Profile updates MUST follow a propose-approve-activate lifecycle.

```

ProfileUpdateProposal = {
proposal_id: tstr,
proposed_profile: CryptoSuiteProfile,
replaces_profile_id: tstr / null,
rationale: tstr,
proposer_id: tstr,
proposed_at_tick: uint,
approval_deadline_tick: uint,
approvals: [* GovernanceApproval],
proposal_hash: bstr,
signature: bstr
}

GovernanceApproval = {
member_id: tstr,
approved_at_tick: uint,
signature: bstr
}

```

Activation MUST NOT occur unless the required threshold approvals are
received before `approval_deadline_tick`.

#### 8.6.3.3 Authority Boundary

Governance artefacts define update legitimacy only.
All acceptance, refusal, revocation, and enforcement semantics are
defined exclusively by consuming specifications.

---

## 9. Hashing Interface

1. Hashing operations MUST reference a hash-class CryptoSuiteProfile.
2. For PQSF-native objects, hash inputs MUST be canonical CBOR bytes.
3. For Epoch Clock artefacts, hash inputs MUST be the JCS Canonical JSON bytes of the artefact.
4. Digest length and algorithm behaviour are defined exclusively by the referenced profile.
5. Implementations MUST NOT vary digest length or parameters outside the profile definition.

---

## 10. Signature Interface

1. Signature operations MUST reference a signature-class CryptoSuiteProfile.
2. For PQSF-native objects, signature input MUST be the canonical CBOR encoding of the payload with signature fields omitted.
3. Signature verification MUST reconstruct the canonical payload deterministically.
4. Classical signatures MAY appear only where explicitly permitted by a consuming specification and carry no authority semantics within PQSF.
5. Epoch Clock signature verification is external and is defined by the Epoch Clock specification. PQSF MUST NOT redefine Epoch Clock signature inputs.

---

## 11. Domain Separation

1. All cryptographic operations over PQSF-native objects MUST apply explicit domain separation.
2. Domain separation identifiers are defined by the domain_set_id referenced in the CryptoSuiteProfile.
3. Domain separation identifiers MUST NOT be altered by implementations.

Epoch Clock domain separation and signature inputs are defined by Epoch Clock and are not modified by PQSF.

---

## 12. Deterministic Derivation

1. Deterministic key and secret derivation MUST reference a derivation-class CryptoSuiteProfile.
2. Derivation inputs MUST be canonically encoded.
3. Derivation MUST be deterministic and reproducible across platforms.

---

## 13. Universal Secret Derivation

PQSF supports deterministic derivation of domain-scoped secrets for various cryptographic purposes.

### 13.1 Derivation Formula

```
Secret = cSHAKE256(
  root_seed,
  domain = "PQSF-Secret:<context>",
  input  = canonical(context_parameters)
)
```

### 13.2 Requirements

1. Secrets MUST be deterministic given identical inputs.
2. Secrets MUST be unlinkable across domains via domain separation.
3. context_parameters MUST be canonically encoded.
4. Root seed MUST have at least 256 bits of entropy.

### 13.3 Standard Context Strings

The following context strings are reserved for standard uses:

* `"PQSF-Secret:SessionKey"` - For session establishment and key agreement
* `"PQSF-Secret:RecoveryShare"` - For recovery material generation
* `"PQSF-Secret:EBTWrap"` - For EBT key derivation
* `"PQSF-Secret:SigningKey"` - For ephemeral signing key derivation
* `"PQSF-Secret:AuthToken"` - For authentication token generation

Implementations MAY define additional context strings using the `"PQSF-Secret:"` prefix.

---

## 14. Epoch Clock Integration Objects

PQSF does not redefine Epoch Clock profiles or ticks as PQSF-native objects.

### 14.1 EpochClockProfileBytes

```
EpochClockProfileBytes = {
  profile_ref: tstr,
  encoding: "JCS-JSON",
  profile_bytes: bstr
}
```

Requirements:

1. profile_bytes MUST be the exact JCS Canonical JSON bytes of the Epoch Clock profile.
2. profile_ref MUST match the profile_ref contained in the decoded JCS JSON.
3. Verification of the profile_bytes is defined by Epoch Clock and consuming enforcement specifications.
4. profile_bytes MUST NOT be re-encoded.

### 14.2 EpochClockTickBytes

```
EpochClockTickBytes = {
  profile_ref: tstr,
  encoding: "JCS-JSON",
  tick_bytes: bstr
}
```

Requirements:

1. tick_bytes MUST be the exact JCS Canonical JSON bytes of the Epoch Clock tick.
2. profile_ref MUST match the profile_ref contained in the decoded JCS JSON.
3. Verification of the tick_bytes is defined by Epoch Clock and consuming enforcement specifications.
4. tick_bytes MUST NOT be re-encoded.

### 14.3 Binding Rule

Any PQSF-native object that binds to time MUST bind to one of:

* the hash of tick_bytes under a referenced hash profile, or
* the tick_bytes directly as an opaque byte string.

---

## 15. Exporter Binding Primitive

1. Exporter binding MUST reference a derivation-class CryptoSuiteProfile.
2. Exporter context MUST be canonical CBOR encoding of:

```
{
  pqsf_version: "2.0.2",
  session_id: tstr,
  role_id: tstr,
  operation_class: "Authoritative" / "NonAuthoritative"
}
```

3. For Authoritative operations, exporter_hash MUST NOT be null.
4. Exporter mismatch MUST invalidate the artefact.

---

## 16. ConsentProof Grammar

```
ConsentProof = {
  action: tstr,
  intent_hash: bstr,
  issued_tick: uint,
  expiry_tick: uint,
  session_id: tstr,
  exporter_hash: bstr,
  consent_id: tstr,
  suite_profile: tstr,
  signature: bstr
}
```

Requirements:

1. ConsentProof MUST be canonical CBOR.
2. signature MUST be computed over canonical CBOR payload with signature omitted.
3. Producing specifications MUST ensure that intent payloads are unique per operation attempt.

ConsentProof carries no authority. Enforcement semantics are external.

---

## 17. Policy Objects and Governance Metadata

### 17.1 PolicyObject

A PolicyObject is a canonical, inert policy declaration consumed by enforcement systems.

PQSF defines the structure and canonical validation rules for PolicyObject. PQSF does not define enforcement semantics.

A PolicyObject MUST contain the following fields:

```
PolicyObject = {
  policy_id: tstr,
  policy_body: { * tstr => any },
  policy_hash: bstr
}
```

**Field Requirements:**

* **policy_id**  
  A tstr identifier. The namespace and interpretation of policy_id are defined by consuming specifications. Policy identifiers SHOULD use a prefix namespace.

* **policy_body**  
  A deterministic map defining policy parameters. The internal schema of policy_body is defined by the policy_id namespace. PQSF defines only canonical encoding and hashing rules.

* **policy_hash**  
  A bstr equal to the hash of the canonical CBOR encoding of policy_body under the active CryptoSuiteProfile.

PolicyObject carries no authority and does not imply enforcement. A PolicyObject is valid only when canonical encoding and policy_hash validation succeed.

---

### 17.2 PolicyBundle

A PolicyBundle is a signed wrapper that associates a PolicyObject with governance metadata and optional session binding.

```
PolicyBundle = {
  bundle_id: tstr,
  policy: PolicyObject,
  issued_tick: uint,
  exporter_hash: bstr / null,
  governance: {
    version: uint,
    min_version: uint,
    signers: [* tstr],
    threshold: uint,
    rollback_denylist: [* bstr],
    rotation_lock_ticks: uint
  },
  suite_profile: tstr,
  signature: bstr
}
```

**Field Requirements:**

* **bundle_id**  
  A unique tstr identifier for this specific bundle instance.

* **policy**  
  A PolicyObject as defined in Section 17.1.

* **issued_tick**  
  A uint representing the issuing tick. Interpretation and freshness enforcement are external to PQSF.

* **exporter_hash**  
  A bstr or null. When present, the bundle is bound to a specific session.

* **governance**  
  A deterministic map containing governance metadata. If present, it MUST include:

  * **version** - Current policy version number (monotonically increasing)
  * **min_version** - Minimum acceptable version (cannot roll back below this)
  * **signers** - List of authorized signer identifiers or public key references
  * **threshold** - M-of-N threshold for policy updates (M signatures required)
  * **rollback_denylist** - List of policy_hash values that cannot be reactivated
  * **rotation_lock_ticks** - Minimum tick interval between policy updates

* **suite_profile**  
  A tstr referencing the CryptoSuiteProfile used for signing.

* **signature**  
  A bstr computed over the canonical CBOR encoding of the bundle with the signature field omitted.

**Validation Requirements:**

PQSF defines structure and validation only. Governance monotonicity, rollback prevention, and refusal semantics are defined exclusively by consuming specifications.

The `signers` field identifies public key identifiers or profile references as defined by the consuming specification.

Policy evaluation semantics are external.

---

## 18. LedgerEntry Grammar

```
LedgerEntry = {
  event: tstr,
  tick: uint,
  payload: { * tstr => any },
  prev_hash: bstr,
  suite_profile: tstr,
  signature: bstr
}
```

**Requirements:**

1. LedgerEntry MUST be canonical CBOR.
2. signature MUST be computed over the canonical CBOR encoding of the entry with signature field omitted.
3. prev_hash MUST reference the hash of the previous canonical LedgerEntry.
4. For the genesis entry, prev_hash MUST be a fixed null value or zero hash as defined by the ledger specification.

---

## 19. Merkle Ledger Serialization

PQSF defines deterministic Merkle tree construction for ledger integrity verification.

### 19.1 Leaf Hash Construction

```
leaf_hash = SHAKE256-256(0x00 || canonical(ledger_entry))
```

Where:
* `0x00` is a single-byte domain separator for leaf nodes
* `canonical(ledger_entry)` is the canonical CBOR encoding of the LedgerEntry
* `||` denotes byte concatenation

### 19.2 Internal Node Hash Construction

```
node_hash = SHAKE256-256(0x01 || left_hash || right_hash)
```

Where:
* `0x01` is a single-byte domain separator for internal nodes
* `left_hash` and `right_hash` are the child node hashes
* `||` denotes byte concatenation

### 19.3 Odd Number of Nodes

If a level has an odd number of nodes:

1. The final node MAY be duplicated for pairing at that level.
2. This rule MUST be consistent within a deployment.
3. Alternative: The final node MAY be promoted to the next level without pairing.
4. Whichever approach is chosen MUST be documented and consistent.

### 19.4 Root Computation

The Merkle root is computed by:

1. Constructing leaf hashes for all ledger entries.
2. Pairing adjacent leaf hashes and computing internal node hashes.
3. Recursively pairing and hashing until a single root hash remains.
4. The final root hash is the Merkle root.

---

## 20. RecoveryCapsule Grammar

```
RecoveryCapsule = {
  capsule_id: tstr,
  guardian_set: [* tstr],
  threshold: uint,
  encrypted_material: bstr,
  creation_tick: uint,
  valid_from_tick: uint,
  recovery_delay: uint,
  suite_profile: tstr,
  signature: bstr
}
```

**Requirements:**

1. RecoveryCapsule MUST be canonical CBOR.
2. signature MUST be computed over the canonical CBOR encoding of the capsule with signature field omitted.
3. Interpretation of encrypted_material is defined by the consuming specification.
4. recovery_delay defines the minimum tick interval between creation_tick and recovery activation.

---

## 21. Encrypted-Before-Transport Wrapper

PQSF defines encrypted artefact wrapping for secure transport and storage.

### 21.1 EBT Requirements

1. Sensitive artefacts transported or stored under PQSF MUST be encrypted before entering any transport or storage boundary outside the originating security domain.
2. EBT encryption MUST satisfy:
   * Canonical input encoding for plaintext and associated data
   * An authenticated encryption scheme selected from the supported list

### 21.2 EBT Key Derivation

EBT encryption keys MUST be derived deterministically:

```
ebt_key = cSHAKE256(
  root_seed,
  domain = "PQSF-EBT:<label>",
  input  = canonical(context_parameters)
)
```

Where:
* `root_seed` is a high-entropy secret (minimum 256 bits)
* `<label>` identifies the specific EBT use case
* `context_parameters` are canonically encoded contextual inputs

### 21.3 Supported AEAD Schemes

PQSF recognizes the following authenticated encryption schemes for EBT:

**Required (Post-Quantum):**
* ML-KEM-1024 + AES-256-GCM
  * Use ML-KEM-1024 to encapsulate a symmetric key
  * Use encapsulated key with AES-256-GCM for authenticated encryption

**Optional (Classical Compatibility):**
* AES-256-GCM (with pre-shared or derived keys)
* XChaCha20-Poly1305

Implementations claiming post-quantum compliance MUST support ML-KEM-1024 + AES-256-GCM.

### 21.4 EBT Nonce Requirements

EBT nonces MUST satisfy:

1. Generated from a cryptographically secure random number generator (CSPRNG).
2. Never reused with the same key.
3. Length requirements:
   * **96 bits (12 bytes)** for AES-256-GCM
   * **192 bits (24 bytes)** for XChaCha20-Poly1305
   * **Scheme-specific** for ML-KEM-based constructions

### 21.5 EBTEnvelope Structure

```
EBTEnvelope = {
  label: tstr,
  tick: uint / null,
  session_id: tstr / null,
  exporter_hash: bstr / null,
  suite_profile: tstr,
  nonce: bstr,
  aad: bstr,
  ciphertext: bstr,
  tag: bstr
}
```

**Field Requirements:**

* **label** - Human-readable identifier for the envelope type
* **tick** - Optional timestamp for freshness tracking
* **session_id** - Optional session binding
* **exporter_hash** - Optional session exporter binding
* **suite_profile** - References the CryptoSuiteProfile defining the AEAD scheme
* **nonce** - The nonce/IV used for encryption (never reused)
* **aad** - Additional authenticated data (canonical encoding of metadata)
* **ciphertext** - Encrypted payload
* **tag** - Authentication tag from the AEAD scheme

**Validation Requirements:**

1. EBTEnvelope MUST be canonical CBOR.
2. aad MUST be the canonical encoding of associated data fields.
3. ciphertext MUST be encryption of the canonical plaintext bytes.
4. tag MUST be the AEAD authentication tag.
5. Where exporter binding is used, EBTEnvelope exporter_hash MUST match session exporter_hash.

PQSF defines EBT structure and wrapping requirements only. Operational refusal behaviour is external.

---

## 22. Supply Chain Artefact Types

PQSF defines deterministic artefact schemas for supply-chain integrity.

### 22.1 Runtime Signing Key (RSK)

Every deployable component (server, service, container, binary) MUST maintain a Runtime Signing Key.

```
RuntimeSignature = {
  component_id: tstr,
  build_hash: bstr,
  config_hash: bstr,
  environment_hash: bstr,
  issued_tick: uint,
  expiry_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

**Requirements:**
1. RuntimeSignature MUST be generated at deployment/startup
2. signature MUST be verifiable by clients before trust establishment
3. build_hash MUST be SHAKE256-256 of canonical build artefact
4. Prevents: poisoned deployments, swapped binaries, silent config changes

### 22.2 Publish Key (PUK) Artefacts

Static assets, installers, containers, and media MUST be signed with Publish Keys.

```
PublishSignature = {
  artefact_id: tstr,
  content_hash: bstr,
  artefact_type: "container" / "binary" / "archive" / "media",
  metadata: { * tstr => any },
  issued_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

**Requirements:**
1. Clients MUST verify PUK signatures even from CDN/mirrors
2. content_hash MUST be SHAKE256-256 of raw artefact bytes
3. Prevents: CDN injection, mirror poisoning, dependency substitution

### 22.3 Operation-Scoped Keys (OSK)

Privileged supply-chain actions (build, deploy, publish) MUST use one-time OSKs.

```
OperationKey = {
  operation_id: tstr,
  operation_type: "build" / "deploy" / "publish" / "sign",
  nonce: bstr,
  issued_tick: uint,
  expiry_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

**Requirements:**
1. OSKs MUST NOT be reused
2. Replay MUST be rejected
3. expiry_tick MUST enforce time-bounded validity
4. Prevents: stolen CI credentials, replayed deploy actions

### 22.4 Delegated Keys (DK) for CI/CD

Build systems receive short-lived Delegation Certificates.

```
DelegationCertificate = {
  delegate_id: tstr,
  scope: [* tstr],  // e.g., ["build", "test"]
  issued_tick: uint,
  expiry_tick: uint,
  revocable: bool,
  suite_profile: tstr,
  signature: bstr
}
```

**Requirements:**
1. DKs MUST be narrowly scoped
2. DKs MUST be short-lived (default: 3600 seconds)
3. Revocation MUST propagate immediately
4. No long-lived API keys permitted

### 22.5 Audit Key (ARK) Artefacts

Long-lived Audit Keys sign compliance artefacts.

```
AuditSignature = {
  audit_id: tstr,
  event_type: "build_summary" / "deployment_snapshot" / "policy_attestation",
  payload_hash: bstr,
  issued_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

**Requirements:**
1. ARK signatures MUST be append-only
2. ARK fingerprints MAY be published publicly
3. Used for: WORM-style audit trails, regulator compliance

---

## 23. Supply Chain Ledger Anchoring

### 23.1 Cross-Registrar Anchoring

Supply-chain events (build, publish, revoke) MUST be:
1. Recorded in registrar logs
2. Anchored to deterministic ledger (as defined in Section 11)
3. Cross-signed by peer registrars (where applicable)

### 23.2 Supply Chain Event Structure

```
SupplyChainEvent = {
  event_id: tstr,
  event_type: "build" / "deploy" / "publish" / "revoke",
  artefact_ref: tstr,
  actor_id: tstr,
  tick: uint,
  ledger_entry_hash: bstr,
  signature: bstr
}
```

**Requirements:**
1. Events MUST be tamper-evident
2. Monotonic tick ordering MUST be enforced
3. prev_hash chain MUST validate

### 23.3 Revocation as Supply-Chain Control

Any compromised key (DK, OSK, RSK, PUK) MUST be revocable via:
1. Revocation artefact with governance signature
2. Immediate propagation to clients
3. Artefact marking as untrusted

Wallets/clients MUST:
1. Block affected artefacts immediately
2. Mark runtimes as untrusted (Red ✗ state)

---

## 24. Reproducible Build Support

### 24.1 Build Attestation Structure

```
BuildAttestation = {
  build_id: tstr,
  source_hash: bstr,
  build_env_hash: bstr,
  output_hash: bstr,
  reproducibility_claim: bool,
  verification_keys: [* bstr],  // For third-party verification
  issued_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

### 24.2 Reproducibility Requirements

1. Builds claiming `reproducibility_claim: true` MUST:
   - Be deterministic across platforms
   - Produce identical output_hash for identical inputs
   - Publish verification instructions

2. Third-party verifiers MAY:
   - Re-build from source
   - Compare output_hash
   - Sign independent verification artefacts

3. Divergence MUST:
   - Trigger security review
   - Be logged in audit trail
   - Prevent deployment until resolved

---

## 25. No Trust in Network Location

PQSF supply-chain security MUST NOT rely on:
1. IP allowlists
2. Shared secrets
3. Implicit trust in CI environments
4. DNS or hostname verification

All trust MUST be:
1. Cryptographic
2. Scoped
3. Time-bound (via Epoch Clock ticks)
4. Auditable

---

## 26. Security Considerations (Supply Chain Additions)

### Threats Addressed

Supply-chain extensions address:

- **Dependency poisoning:** PUK + RSK + signed manifests
- **CI credential theft:** OSKs + short TTL
- **Insider deploy sabotage:** Scoped DKs + audit trail
- **CDN injection:** PUK verification at client
- **Build/runtime mismatch:** RSK startup attestation
- **Silent config changes:** Signed config manifests
- **Log tampering:** Ledger anchoring prevents history rewriting

### Threat Model Updates

Supply-chain adversaries MAY:
- Compromise build infrastructure
- Inject malicious dependencies
- Poison CDN/mirror content
- Steal long-lived credentials
- Tamper with deployment configs

Supply-chain defenses DO NOT protect against:
- Zero-day vulnerabilities in dependencies (detection only)
- Malicious but correctly-signed artefacts (requires governance)
- Compromised root signing keys (requires revocation + recovery)

---


## 27. Sovereign Transport Protocol Grammar (STP)

PQSF defines a deterministic, minimal message grammar for STP framing.

### 27.1 STP Message Requirements

STP messages MUST include:

* protocol version identifier
* session identifier
* nonce material for session establishment or resumption
* capability advertisement as a bounded list of tokens
* exporter_hash field when a session exporter binding is produced

### 27.2 STP Message Types

STP defines the following message types:

#### 27.2.1 STP-INIT

```
STP-INIT = {
  version: tstr,
  session_id: tstr,
  client_nonce: bstr,
  client_caps: [* tstr]
}
```

**Purpose:** Client initiates a new session.

**Requirements:**
* version MUST be "1.0" for this specification version
* session_id MUST be unique per session attempt
* client_nonce MUST be generated from CSPRNG (minimum 128 bits)
* client_caps lists client-supported capabilities

#### 27.2.2 STP-ACCEPT

```
STP-ACCEPT = {
  version: tstr,
  session_id: tstr,
  server_nonce: bstr,
  server_caps: [* tstr],
  exporter_hash: bstr
}
```

**Purpose:** Server accepts session and provides exporter binding.

**Requirements:**
* version MUST match STP-INIT version
* session_id MUST match STP-INIT session_id
* server_nonce MUST be generated from CSPRNG (minimum 128 bits)
* server_caps lists server-supported capabilities (subset of client_caps)
* exporter_hash is derived from the established transport session

#### 27.2.3 STP-RESUME

```
STP-RESUME = {
  version: tstr,
  previous_session_id: tstr,
  resume_token: bstr,
  client_nonce: bstr
}
```

**Purpose:** Client attempts to resume a previous session.

**Requirements:**
* previous_session_id references an earlier STP-ACCEPT session
* resume_token is an opaque token provided by server in previous session
* client_nonce MUST be fresh (never reused from previous session)

#### 27.2.4 STP-ERROR

```
STP-ERROR = {
  version: tstr,
  code: tstr,
  session_id: tstr,
  message: tstr / null
}
```

**Purpose:** Either party signals an error condition.

**Requirements:**
* code is a structured error code (defined by consuming specifications)
* session_id references the affected session
* message is an optional human-readable description

### 27.3 STP Canonical Encoding

STP messages MUST be canonical CBOR as defined in Section 7.

### 27.4 STP Capability Negotiation

1. Client advertises capabilities in client_caps.
2. Server MUST respond with server_caps containing only capabilities it supports.
3. The effective capability set is the intersection of client_caps and server_caps.
4. If intersection is empty or insufficient, server SHOULD respond with STP-ERROR.

---

## 28. Failure Semantics

1. Validation of any PQSF-native object MUST be atomic. Either all validation steps succeed and the object is accepted, or any single validation failure causes rejection with no partial acceptance.
2. Any PQSF-native object failing structural validation, canonical encoding validation, profile validation, hash validation, or signature verification MUST be rejected.
3. Any Epoch Clock artefact whose verified bytes do not match the supplied JCS Canonical JSON bytes MUST be rejected by consuming enforcement specifications.
4. PQSF defines no retry, escalation, or authority behaviour.

---

## 29. Conformance

An implementation is PQSF-conformant if it:

* enforces strict deterministic CBOR encoding for PQSF-native objects
* enforces the Epoch Clock JCS Canonical JSON exception and forbids Epoch Clock re-encoding
* enforces CryptoSuiteProfile indirection and pinning
* enforces profile distribution, revocation, and emergency transition rules
* enforces exporter binding rules
* enforces deterministic hashing, signing, and derivation
* enforces schema validation
* rejects malformed or non canonical objects
* supports EBT encryption with required AEAD schemes
* implements STP message grammar correctly

---

## 30. Explicit Dependencies

PQSF depends on:

* RFC 8949 Deterministic CBOR
* RFC 8785 JCS Canonical JSON (for Epoch Clock artefacts only)
* TLS 1.3 exporter mechanism (RFC 8446)
* ML-DSA-65 for post-quantum signatures
* SHAKE256-256 for hashing
* ML-KEM-1024 for post-quantum key encapsulation (EBT)
* cSHAKE256 for key derivation
* externally defined CryptoSuiteProfile signing keys
* external enforcement specifications, including PQSEC
* external time specification, including Epoch Clock

PQSF defines no enforcement authority.

---


## 31. Security Considerations

### Threats Addressed

PQSF addresses the following threats within its defined scope:

- **Encoding ambiguity and parser differentials:**  
  Deterministic canonical encoding eliminates semantically equivalent but
  byte-distinct representations that can be exploited across
  implementations.

- **Hash and signature mismatch attacks:**  
  Byte-identical canonical encoding ensures that hashing and signing
  inputs are stable and reproducible across platforms.

- **Algorithm downgrade and agility failures:**  
  CryptoSuiteProfile indirection prevents hardcoded algorithm reliance
  and enables governed transitions without specification rewrites.

- **Session confusion and cross-session replay:**  
  Exporter binding primitives allow consuming specifications to bind
  artefacts to authenticated session state.

---

### Threats NOT Addressed (Out of Scope)

PQSF does NOT protect against:

- **Enforcement errors or policy mistakes:**  
  PQSF provides primitives only. Enforcement correctness is delegated to
  consuming specifications (e.g., PQSEC).

- **Transport protocol flaws:**  
  PQSF does not define or secure transport protocols themselves.

- **Key compromise or runtime compromise:**  
  Key management and runtime integrity are outside PQSF’s scope.

- **Denial-of-service attacks:**  
  PQSF prioritizes correctness and determinism over availability.

---

### Authority Boundary

PQSF grants no authority and performs no enforcement.

It defines protocol rules and primitives only. Any authority or decision
derived from PQSF artefacts is the responsibility of consuming
specifications.

---

### Fail-Closed Semantics

PQSF itself does not permit or deny operations.

However, consuming systems MUST:
- reject non-canonical encodings
- reject unverifiable signatures or hashes
- reject artefacts that do not conform to declared profiles

Failure to meet PQSF rules MUST be treated as a hard failure by
consumers.

---

### Fuzzing and Structural Attack Resistance

Canonical encoding requirements eliminate structural fuzzing attack
classes that rely on ambiguous or inconsistent representations.

The following attack vectors are structurally mitigated:

- **Parser differentials:**  
  Semantically identical inputs MUST canonicalize to a single byte
  representation or be rejected. Multiple interpretations are forbidden.

- **Hash and signature mismatch attacks:**  
  All hashing and signing operations are performed over canonical bytes
  only. Re-encoding between parse and verification is forbidden.

- **Encoding variance attacks:**  
  Alternate CBOR encodings (long-form integers, indefinite-length items,
  unsorted maps) MUST be rejected deterministically.

- **Floating-point representation attacks:**  
  Floating-point values are prohibited entirely, eliminating NaN,
  signed-zero, and subnormal representation variance.

- **Unicode normalization ambiguity:**  
  Canonical encodings operate on byte sequences. No implicit Unicode
  normalization is permitted at the protocol layer.

As a result, fuzzing inputs collapse to one of two outcomes:
- a single canonical byte representation, or
- deterministic rejection.

No degraded, heuristic, or best-effort parsing is permitted for
PQSF-governed artefacts.

---

## 32. Conformance Checklist

An implementation is PQSF conformant if it satisfies all REQUIRED items
below and documents any OPTIONAL features it claims to support.

### Required (MUST)

☐ Enforces deterministic canonical encoding for all PQSF-governed artefacts  
☐ Produces byte-identical encodings for semantically identical input  
☐ Rejects all non-canonical encodings deterministically  
☐ Rejects floating-point values in all canonical artefacts  
☐ Enforces shortest encoding form for all CBOR primitives  
☐ Enforces deterministic map key ordering (bytewise lexicographic)  
☐ Validates schema compliance before hashing or signing  
☐ Computes hashes and signatures over canonical bytes only  
☐ Enforces CryptoSuiteProfile pinning and validation  
☐ Provides exporter binding primitives suitable for session binding  

### Conditional (MUST if applicable)

☐ If CBOR is used, rejects indefinite-length items  
☐ If JSON is permitted, enforces the specified canonical JSON profile  
☐ If exporter binding is used, produces stable exporter material per session  
☐ If profile governance metadata is present, enforces version and rollback rules  

### Recommended (SHOULD)

☐ Uses independent encoder/decoder implementations for cross-checking  
☐ Fuzz-tests decoders with malformed and non-canonical inputs  
☐ Logs canonicalization failures with deterministic error codes  
☐ Monitors profile usage and deprecation status  
☐ Maintains a registry of accepted and revoked profiles  

### Optional (MAY)

☐ Provides tooling to canonicalize and inspect artefacts  
☐ Provides reference test vectors for encoders and decoders  
☐ Supports multiple profile registries (e.g., offline mirrors)  

### Testing

☐ Demonstrates re-encoding stability (decode → encode → identical bytes)  
☐ Demonstrates rejection of non-canonical CBOR forms  
☐ Demonstrates rejection of floating-point encodings  
☐ Demonstrates cross-implementation encoding equivalence  
☐ Demonstrates correct exporter material derivation (where applicable)  

### Documentation

☐ Documents canonical encoding rules and exceptions  
☐ Documents supported CryptoSuiteProfiles and minimum versions  
☐ Documents exporter binding interfaces and expectations  
☐ Provides a conformance statement with version numbers  

---

## 33. Annexes (Non Normative)

### Annex A – Canonical CBOR Profile Summary

The PQSF strict CBOR profile enforces:

* Deterministic encoding per RFC 8949 §4.2
* Shortest integer form
* No floating point values
* No indefinite-length items
* Unique map keys
* Lexicographic key ordering
* No CBOR tags unless explicitly profiled by consuming specifications

---

### Annex B – Example CryptoSuiteProfile

```json
{
  "profile_id": "pqsf:crypto:hash:shake256-v1",
  "suite_class": "hash",
  "family": "SHAKE",
  "parameters": {
    "output_bits": 256
  },
  "domain_set_id": "pqsf-domain-v2",
  "test_vector_hash": "01ab...",
  "valid_from_tick": 0,
  "valid_until_tick": null,
  "revoked": false,
  "signature": "02cd..."
}
```

---

### Annex C – Reference Pseudocode (PQSF-Native)

```python
def canonical_hash_cbor(obj, profile):
    """Hash a PQSF-native object."""
    bytes = canonical_cbor_encode(obj)
    return hash_with_profile(profile, bytes)

def sign_cbor(obj, profile, sk):
    """Sign a PQSF-native object."""
    payload = canonical_cbor_encode(obj)
    return sign_with_profile(profile, sk, payload)

def verify_cbor(obj, profile, pk, sig):
    """Verify signature on a PQSF-native object."""
    payload = canonical_cbor_encode(obj)
    return verify_with_profile(profile, pk, payload, sig)
```

---

### Annex D – Reference Pseudocode (Epoch Clock Binding)

```python
def epochclock_hash_tick_bytes(tick_json_jcs_bytes, hash_profile):
    """Hash Epoch Clock tick bytes (already in JCS JSON form)."""
    return hash_with_profile(hash_profile, tick_json_jcs_bytes)

def bind_time_by_tick_hash(pqsf_obj, tick_hash):
    """Bind a PQSF object to time via Epoch Clock tick hash."""
    pqsf_obj["epoch_tick_hash"] = tick_hash
    return pqsf_obj
```

---

### Annex E – Replay Safety Checklist

For replay-safe artefact design:

* ☐ intent_hash bound
* ☐ session_id bound
* ☐ exporter_hash bound (for Authoritative)
* ☐ issued_tick and expiry_tick present
* ☐ single-use enforced by consumer where required
* ☐ decision_id or operation_id unique per attempt
* ☐ nonce present for challenge-response patterns

---

### Annex F – Privacy Policy Namespace

Privacy policies MAY be represented as PolicyObject instances using the `privacy:` namespace.

**Recommended policy_id values:**

* `privacy:retention:v1` - Data retention policies
* `privacy:logging:v1` - Logging and audit policies
* `privacy:disclosure:v1` - Data disclosure policies
* `privacy:telemetry:v1` - Telemetry and metrics policies
* `privacy:bundle:v1` - Bundled privacy policy suite

**Recommended artefact class identifiers within privacy policy bodies:**

* `consent` - Consent artefacts
* `policy` - Policy objects themselves
* `attestation` - Attestation envelopes
* `ledger` - Ledger entries
* `session` - Session data
* `model` - AI model artefacts
* `prompt` - User prompts and AI outputs
* `export` - Data exports

This annex is informative only. Consuming specifications define which identifiers are required or meaningful.

---

### Annex G – STP Session Lifecycle Example

```
Client                                  Server
  |                                       |
  |--- STP-INIT -----------------------> |
  |    {session_id: "abc", nonce: ...}   |
  |                                       |
  | <-- STP-ACCEPT ----------------------|
  |     {session_id: "abc",              |
  |      exporter_hash: ...}             |
  |                                       |
  |--- Encrypted Application Data -----> |
  | <-- Encrypted Application Data ------|
  |                                       |
  |--- STP-RESUME (later) --------------> |
  |    {previous_session_id: "abc"}      |
  |                                       |
  | <-- STP-ACCEPT or STP-ERROR ---------|
```

---

## Annex H — KeyMail (Informative)

### H.1 Scope

KeyMail defines secure, non-authoritative messaging artefacts for use
within the PQ stack. KeyMail provides confidentiality and integrity for
message exchange only and carries no execution, authority, or consent
semantics.

KeyMail artefacts:

* MUST NOT trigger execution
* MUST NOT propagate authority
* MUST NOT be consumed by PQSEC as action predicates
* MUST NOT be interpreted as consent, authorization, enforcement
  outcomes, or execution capability

KeyMail is explicitly out of scope for enforcement, admission, refusal,
or escalation logic.

---

### H.2 KeyMailEnvelope

```

KeyMailEnvelope = {
message_id: tstr,
sender_identity_ref: tstr,
recipient_identity_ref: tstr,
ciphertext: bstr,
issued_tick: uint,
expiry_tick: uint,
suite_profile: tstr
}

```

---

### H.3 Rules

1. KeyMailEnvelope MUST be canonically encoded under PQSF canonical
   encoding rules.
2. KeyMailEnvelope.ciphertext MUST be produced using authenticated
   encryption under the referenced suite_profile.
3. issued_tick and expiry_tick MUST be present and MUST be enforced by
   consuming systems. Expired KeyMailEnvelope artefacts MUST be rejected.
4. KeyMailEnvelope carries no authority and MUST NOT be treated as an
   execution, permission, enforcement, or predicate signal by any
   consuming specification, including PQSEC.

---

### Annex I – EBT Encryption Flow Example

```python
def ebt_encrypt(plaintext: bytes, label: str, root_seed: bytes, context: dict):
    """
    Encrypt plaintext using EBT pattern.
    """
    # Derive EBT key
    ebt_key = cSHAKE256(
        root_seed,
        domain=f"PQSF-EBT:{label}",
        input=canonical_encode(context)
    )
    
    # Generate nonce
    nonce = os.urandom(12)  # 96 bits for AES-GCM
    
    # Prepare AAD
    aad = canonical_encode({
        "label": label,
        "context": context
    })
    
    # Encrypt
    ciphertext, tag = aes_256_gcm_encrypt(
        key=ebt_key,
        nonce=nonce,
        plaintext=plaintext,
        aad=aad
    )
    
    # Return EBTEnvelope
    return {
        "label": label,
        "suite_profile": "pqsf:aead:aes-256-gcm:v1",
        "nonce": nonce,
        "aad": aad,
        "ciphertext": ciphertext,
        "tag": tag
    }
```

---

### Annex J – Exporter Hash Derivation Example

```python
def derive_exporter_hash(tls_session, session_id: str, role_id: str, operation_class: str):
    """
    Derive exporter hash from TLS session.
    """
    context = canonical_cbor_encode({
        "pqsf_version": "2.0.2",
        "session_id": session_id,
        "role_id": role_id,
        "operation_class": operation_class
    })
    
    # TLS 1.3 exporter
    exporter_hash = tls_exporter(
        session=tls_session,
        label="PQSF-EXPORTER",
        context=context,
        length=32  # 256 bits
    )
    
    return exporter_hash
```

---

### Annex K – Merkle Tree Construction Example

```python
def build_merkle_tree(ledger_entries: list) -> bytes:
    """
    Build Merkle tree from ledger entries and return root hash.
    """
    from hashlib import shake_256
    
    def shake256_256(data: bytes) -> bytes:
        return shake_256(data).digest(32)
    
    def leaf_hash(entry: dict) -> bytes:
        canonical_bytes = canonical_cbor_encode(entry)
        return shake256_256(b'\x00' + canonical_bytes)
    
    def node_hash(left: bytes, right: bytes) -> bytes:
        return shake256_256(b'\x01' + left + right)
    
    # Build leaf layer
    current_level = [leaf_hash(entry) for entry in ledger_entries]
    
    # Build tree bottom-up
    while len(current_level) > 1:
        next_level = []
        
        # Process pairs
        for i in range(0, len(current_level), 2):
            left = current_level[i]
            
            # Handle odd number of nodes (duplicate last)
            if i + 1 < len(current_level):
                right = current_level[i + 1]
            else:
                right = left  # Duplicate last node
            
            next_level.append(node_hash(left, right))
        
        current_level = next_level
    
    return current_level[0]  # Root hash
```

---

### Annex L – PolicyBundle Validation Example

```python
def validate_policy_bundle(bundle: dict, governance_pubkeys: dict, current_tick: int):
    """
    Validate a PolicyBundle structure and signatures.
    Returns True if valid, False otherwise.
    """
    # 1. Validate structure
    required_fields = ["bundle_id", "policy", "issued_tick", "governance", "suite_profile", "signature"]
    if not all(field in bundle for field in required_fields):
        return False
    
    # 2. Validate PolicyObject
    policy = bundle["policy"]
    if not validate_policy_object(policy):
        return False
    
    # 3. Check tick freshness (if required by policy)
    if bundle["issued_tick"] > current_tick:
        return False  # Future-dated
    
    # 4. Validate governance signatures
    governance = bundle["governance"]
    threshold = governance["threshold"]
    signers = governance["signers"]
    
    # Bundle must be signed by threshold signers
    valid_sigs = 0
    for signer_id in signers:
        if signer_id in governance_pubkeys:
            # Reconstruct canonical payload
            payload = canonical_cbor_encode({
                "bundle_id": bundle["bundle_id"],
                "policy": bundle["policy"],
                "issued_tick": bundle["issued_tick"],
                "exporter_hash": bundle.get("exporter_hash"),
                "governance": bundle["governance"],
                "suite_profile": bundle["suite_profile"]
            })
            
            # Verify signature (implementation-specific)
            if verify_signature(
                pubkey=governance_pubkeys[signer_id],
                payload=payload,
                signature=bundle["signature"],
                profile=bundle["suite_profile"]
            ):
                valid_sigs += 1
    
    return valid_sigs >= threshold

def validate_policy_object(policy: dict) -> bool:
    """
    Validate PolicyObject structure and hash.
    """
    if not all(field in policy for field in ["policy_id", "policy_body", "policy_hash"]):
        return False
    
    # Recompute policy_hash
    body_bytes = canonical_cbor_encode(policy["policy_body"])
    computed_hash = shake256_256(body_bytes)
    
    return computed_hash == policy["policy_hash"]
```

---

### Annex M – Profile Update Acceptance Logic

```python
def should_accept_profile_update(
    current_profile: dict,
    update: dict,
    governance_pubkeys: dict,
    current_tick: int
) -> tuple[bool, str]:
    """
    Determine if a ProfileUpdate should be accepted.
    Returns (accept: bool, reason: str).
    """
    new_profile = update["profile"]
    
    # 1. Verify governance signatures
    required_sigs = len(governance_pubkeys) // 2 + 1  # M-of-N majority
    valid_sigs = 0
    
    for sig_entry in update["governance_sigs"]:
        signer_id = sig_entry["signer_id"]
        sig = sig_entry["sig"]
        
        if signer_id in governance_pubkeys:
            payload = canonical_cbor_encode(new_profile)
            if verify_signature(governance_pubkeys[signer_id], payload, sig):
                valid_sigs += 1
    
    if valid_sigs < required_sigs:
        return False, f"Insufficient signatures: {valid_sigs}/{required_sigs}"
    
    # 2. Check activation time
    if new_profile["valid_from_tick"] > current_tick:
        return False, "Update is future-dated, not yet valid"
    
    # 3. Check for downgrade
    if is_downgrade(current_profile, new_profile):
        return False, "Profile update is a security downgrade"
    
    # 4. Check revocation status
    if new_profile.get("revoked", False):
        return False, "New profile is already revoked"
    
    return True, "Profile update accepted"

def is_downgrade(old_profile: dict, new_profile: dict) -> bool:
    """
    Check if new_profile is a security downgrade from old_profile.
    """
    # Check suite class consistency
    if old_profile["suite_class"] != new_profile["suite_class"]:
        return True  # Cannot change suite class
    
    # Algorithm family checks
    old_family = old_profile["family"]
    new_family = new_profile["family"]
    
    # PQ to classical is downgrade
    pq_families = ["ML-DSA", "ML-KEM", "SHAKE"]
    classical_families = ["ECDSA", "RSA", "SHA"]
    
    if old_family in pq_families and new_family in classical_families:
        return True
    
    # Check parameter strength (implementation-specific)
    # For example, ML-DSA-87 -> ML-DSA-65 is downgrade
    if old_family == new_family:
        old_strength = get_security_strength(old_profile)
        new_strength = get_security_strength(new_profile)
        if new_strength < old_strength:
            return True
    
    return False
```

---

### Annex N – Emergency Transition State Machine

```
Normal Operation
       |
       | (algorithm compromise detected)
       v
Emergency Profile Issued
       |
       | (governance signatures verified)
       v
Phase 1: Dual-Algorithm Mode (Days 0-7)
       | - Accept both old and new algorithms
       | - Log all uses of old algorithm
       | - Warn users
       |
       | (Day 7 reached)
       v
Phase 2: Authoritative Restriction (Days 7-30)
       | - Reject old algorithm for Authoritative ops
       | - Accept for Non-Authoritative ops only
       | - Escalate warnings
       |
       | (Day 30 reached)
       v
Phase 3: Complete Deprecation
       | - Reject old algorithm for all operations
       | - Remove from codebase
       |
       v
New Normal Operation
```

---

### Annex O – STP Capability Negotiation Example

```python
def negotiate_capabilities(client_caps: list[str], server_caps: list[str]) -> dict:
    """
    Negotiate session capabilities between client and server.
    """
    # Required capabilities that must be present
    required = ["pqsf-v2", "cbor-canonical", "ml-dsa-65"]
    
    # Compute intersection
    agreed_caps = set(client_caps) & set(server_caps)
    
    # Check required capabilities
    if not all(req in agreed_caps for req in required):
        return {
            "success": False,
            "error": "E_CAPABILITY_MISMATCH",
            "missing": [req for req in required if req not in agreed_caps]
        }
    
    # Optional capabilities
    optional = {
        "compression": ["gzip", "zstd"],
        "extensions": ["batch-ops", "streaming"]
    }
    
    negotiated = {
        "success": True,
        "required": required,
        "optional": {}
    }
    
    # Negotiate optional features
    for feature, options in optional.items():
        for opt in options:
            cap_name = f"{feature}:{opt}"
            if cap_name in agreed_caps:
                if feature not in negotiated["optional"]:
                    negotiated["optional"][feature] = []
                negotiated["optional"][feature].append(opt)
    
    return negotiated
```

---

### Annex P – ConsentProof Construction Example

```python
def create_consent_proof(
    action: str,
    intent: dict,
    session_id: str,
    exporter_hash: bytes,
    issuer_sk,
    current_tick: int,
    validity_duration: int = 900  # 15 minutes
) -> dict:
    """
    Create a ConsentProof artefact.
    """
    import os
    from hashlib import shake_256
    
    # Compute intent hash
    intent_bytes = canonical_cbor_encode(intent)
    intent_hash = shake_256(intent_bytes).digest(32)
    
    # Generate unique consent_id
    consent_id = os.urandom(16).hex()
    
    # Construct ConsentProof payload
    consent_proof = {
        "action": action,
        "intent_hash": intent_hash,
        "issued_tick": current_tick,
        "expiry_tick": current_tick + validity_duration,
        "session_id": session_id,
        "exporter_hash": exporter_hash,
        "consent_id": consent_id,
        "suite_profile": "pqsf:sig:ml-dsa-65:v1"
    }
    
    # Sign the payload
    payload_bytes = canonical_cbor_encode(consent_proof)
    signature = ml_dsa_65_sign(issuer_sk, payload_bytes)
    
    consent_proof["signature"] = signature
    
    return consent_proof
```

---

### Annex Q – LedgerEntry Chain Validation

```python
def validate_ledger_chain(entries: list[dict]) -> tuple[bool, str]:
    """
    Validate a chain of ledger entries.
    Returns (valid: bool, error_message: str).
    """
    if not entries:
        return False, "Empty ledger"
    
    # Genesis entry validation
    genesis = entries[0]
    if genesis["prev_hash"] != b'\x00' * 32:  # Assuming zero hash for genesis
        return False, "Invalid genesis prev_hash"
    
    # Validate chain continuity
    for i in range(1, len(entries)):
        current = entries[i]
        previous = entries[i - 1]
        
        # Compute hash of previous entry
        prev_payload = canonical_cbor_encode({
            "event": previous["event"],
            "tick": previous["tick"],
            "payload": previous["payload"],
            "prev_hash": previous["prev_hash"]
        })
        computed_prev_hash = shake256_256(prev_payload)
        
        # Check prev_hash reference
        if current["prev_hash"] != computed_prev_hash:
            return False, f"Chain break at entry {i}: prev_hash mismatch"
        
        # Check monotonicity
        if current["tick"] < previous["tick"]:
            return False, f"Non-monotonic tick at entry {i}"
        
        # Verify signature
        if not verify_ledger_signature(current):
            return False, f"Invalid signature at entry {i}"
    
    return True, "Ledger chain valid"

def verify_ledger_signature(entry: dict) -> bool:
    """
    Verify signature on a ledger entry.
    """
    payload = canonical_cbor_encode({
        "event": entry["event"],
        "tick": entry["tick"],
        "payload": entry["payload"],
        "prev_hash": entry["prev_hash"]
    })
    
    return verify_signature(
        pubkey=get_ledger_pubkey(entry["suite_profile"]),
        payload=payload,
        signature=entry["signature"],
        profile=entry["suite_profile"]
    )
```

---

### Annex R – RecoveryCapsule Activation Logic

```python
def can_activate_recovery(
    capsule: dict,
    current_tick: int,
    guardian_approvals: list[str]
) -> tuple[bool, str]:
    """
    Check if a RecoveryCapsule can be activated.
    """
    # 1. Check valid_from_tick
    if current_tick < capsule["valid_from_tick"]:
        return False, "Recovery period has not started"
    
    # 2. Check recovery_delay
    activation_tick = capsule["creation_tick"] + capsule["recovery_delay"]
    if current_tick < activation_tick:
        remaining = activation_tick - current_tick
        return False, f"Recovery delay not elapsed (remaining: {remaining} ticks)"
    
    # 3. Check guardian threshold
    required_guardians = capsule["threshold"]
    guardian_set = set(capsule["guardian_set"])
    
    # Verify approvals are from valid guardians
    valid_approvals = [g for g in guardian_approvals if g in guardian_set]
    
    if len(valid_approvals) < required_guardians:
        return False, f"Insufficient guardian approvals: {len(valid_approvals)}/{required_guardians}"
    
    return True, "Recovery can proceed"
```

---

### Annex S – Test Vector Checklist

For PQSF conformance testing, implementations MUST provide test vectors demonstrating:

**Canonical Encoding:**
* ☐ CBOR encoding round-trip (encode → decode → encode produces identical bytes)
* ☐ Rejection of non-canonical CBOR (wrong integer encoding, unsorted maps, etc.)
* ☐ JCS JSON handling for Epoch Clock artefacts

**Cryptographic Operations:**
* ☐ Hash reproducibility (same input → same hash across implementations)
* ☐ Signature verification (valid signatures accepted, invalid rejected)
* ☐ Exporter hash derivation (consistent across sessions)

**Profile Management:**
* ☐ Profile update acceptance (valid updates accepted)
* ☐ Downgrade rejection (security downgrades rejected)
* ☐ Revocation enforcement (revoked profiles rejected)
* ☐ Emergency transition (phased deprecation works correctly)

**Object Validation:**
* ☐ PolicyBundle validation (structure, hash, signatures)
* ☐ ConsentProof validation (structure, binding, freshness)
* ☐ LedgerEntry chain validation (continuity, monotonicity)

**Merkle Tree Construction:**
* ☐ Root hash reproducibility (same entries → same root)
* ☐ Odd-node handling (consistent across implementations)

**EBT Encryption:**
* ☐ Key derivation determinism (same inputs → same key)
* ☐ Nonce uniqueness enforcement
* ☐ AEAD correctness (encrypt/decrypt round-trip)

**STP Protocol:**
* ☐ Message structure validation
* ☐ Capability negotiation correctness
* ☐ Session resumption handling

---

### Annex T – Performance Optimization Guidelines

While PQSF does not define performance requirements, the following optimizations are recommended:

**Caching Strategies:**
1. **Profile Cache:** Cache validated CryptoSuiteProfiles by profile_id
2. **Hash Cache:** Cache frequently computed hashes (e.g., policy_hash)
3. **Signature Verification:** Cache verified signatures within freshness window

**Parallelization Opportunities:**
1. **Independent Signature Verification:** Verify multiple signatures in parallel
2. **Merkle Tree Construction:** Build subtrees in parallel
3. **Profile Updates:** Check multiple governance signatures concurrently

**Lazy Evaluation:**
1. **Profile Discovery:** Only fetch profiles when needed
2. **Revocation Checks:** Cache revocation list, update periodically
3. **Capability Negotiation:** Defer optional feature setup until used

**Memory Management:**
1. **Canonical Encoding:** Reuse encoding buffers where safe
2. **Merkle Trees:** Stream large ledgers rather than loading entirely
3. **EBT Envelopes:** Encrypt/decrypt in streaming mode for large payloads

---

### Annex U – Security Considerations Summary

**Critical Security Properties:**

1. **Canonical Encoding is Non-Negotiable**
   - Non-canonical encodings create hash ambiguity
   - Implementations MUST reject any non-canonical input

2. **Profile Downgrade Prevention**
   - Never accept weaker cryptography
   - Emergency transitions are the ONLY exception

3. **Exporter Binding for Authoritative Operations**
   - Session binding prevents cross-session replay
   - Null exporter_hash for Authoritative ops is a vulnerability

4. **Signature Verification Order**
   - Verify structure BEFORE cryptographic operations
   - Avoids expensive operations on malformed inputs

5. **Time Binding via Epoch Clock**
   - Never trust system clocks
   - Always consume externally signed ticks

6. **Ledger Continuity**
   - prev_hash chain prevents insertion/deletion
   - Monotonic ticks prevent reordering

**Common Implementation Mistakes:**

* Re-encoding Epoch Clock artefacts (breaks signature verification)
* Accepting unsigned or self-signed profiles
* Allowing null exporter_hash for Authoritative operations
* Trusting system time instead of Epoch Clock
* Not checking revocation before profile use
* Reusing nonces in EBT encryption

---

# Annex V — KeyMail: Secure Messaging Artefacts (Normative)

This annex defines secure, non-authoritative messaging artefacts for the PQ ecosystem.

---

## V.1 Scope and Authority Boundary

KeyMail defines canonically encoded, encrypted message artefacts for secure communication between PQ-aware endpoints.

**KeyMail provides:**
- End-to-end encrypted message envelopes
- Sender and recipient identity binding
- Time-bounded message validity
- Optional session binding for conversational contexts
- Thread continuity for multi-message exchanges

**Authority Boundary:**

KeyMail grants no authority.

- Receiving a KeyMailEnvelope does NOT imply consent
- Receiving a KeyMailEnvelope does NOT trigger execution
- Receiving a KeyMailEnvelope does NOT propagate custody authority
- KeyMail artefacts MUST NOT be interpreted as ConsentProof, delegation, or permission

KeyMail is a transport and storage format only. All enforcement, admission, and authority decisions are external to KeyMail and defined by PQSEC.

---

## V.2 KeyMailEnvelope Structure

KeyMailEnvelope = {
  message_id: tstr,
  thread_id: tstr / null,
  sender_identity_ref: tstr,
  recipient_identity_ref: tstr,
  issued_tick: uint,
  expiry_tick: uint,
  session_id: tstr / null,
  exporter_hash: bstr / null,
  content_type: tstr,
  encapsulated_key: bstr / null,
  ciphertext: bstr,
  nonce: bstr,
  tag: bstr,
  suite_profile: tstr,
  sender_signature: bstr
}

---

## V.3 Field Semantics

**message_id**
Unique identifier for this message. MUST be unique across all messages from this sender.

**thread_id**
Optional identifier for message threading. If present, MUST be consistent across all messages in a thread. Absence indicates a standalone message.

**sender_identity_ref**
Reference to the sender's identity artefact (ModelIdentity for AI, custody identity for human operators, or service identity). Format is deployment-defined but MUST be resolvable to a verification key.

**recipient_identity_ref**
Reference to the recipient's identity artefact. Used for key agreement and routing. Format is deployment-defined.

**issued_tick**
Epoch Clock tick at which the message was created.

**expiry_tick**
Epoch Clock tick after which the message SHOULD be considered stale. Consuming systems MAY refuse messages past expiry. Enforcement is external.

**session_id**
Optional session identifier for binding messages to a specific conversational session.

**exporter_hash**
Optional TLS exporter hash for session binding. When present, MUST match the active session's exporter hash for the message to be accepted in session-bound contexts.

**content_type**
MIME type or PQ-defined type identifier for the decrypted payload. Common values:
- `text/plain` — Plain text message
- `application/cbor` — Structured CBOR payload
- `application/json` — JSON payload
- `pqsf:artefact` — Embedded PQSF artefact (ConsentProof, PolicyBundle, etc.)

**encapsulated_key**
ML-KEM encapsulated key ciphertext for the recipient. MUST be present when using ephemeral key agreement. MAY be null when using pre-shared keys or when the encapsulated key is transported separately. When present, MUST be the output of ML-KEM-1024.Encapsulate() or equivalent KEM operation.

**ciphertext**
Encrypted message payload. MUST be produced by an authenticated encryption scheme defined by suite_profile.

**nonce**
Cryptographic nonce used for encryption. MUST satisfy the nonce requirements of the AEAD scheme in suite_profile. MUST NOT be reused with the same key.

**tag**
Authentication tag from the AEAD scheme.

**suite_profile**
CryptoSuiteProfile reference defining:
- Key agreement scheme (e.g., ML-KEM-1024, X25519)
- AEAD scheme (e.g., AES-256-GCM, XChaCha20-Poly1305)
- Signature scheme for sender_signature

**sender_signature**
Signature over the canonical CBOR encoding of the envelope with sender_signature field omitted. Provides sender authentication and non-repudiation.

---

## V.4 Canonical Encoding Requirements

1. KeyMailEnvelope MUST be encoded using PQSF Deterministic CBOR.
2. sender_signature MUST be computed over the canonical CBOR encoding with the sender_signature field omitted.
3. Re-encoding a decoded KeyMailEnvelope MUST produce byte-identical output.
4. Non-canonical encodings MUST be rejected.

---

## V.5 Key Agreement and Encryption

### V.5.1 Ephemeral Key Agreement

For post-quantum key agreement:

```
(shared_secret, encapsulated_key) = ML-KEM-1024.Encapsulate(recipient_public_key)
```

The encapsulated_key MUST be transmitted alongside the KeyMailEnvelope or embedded in deployment-specific key transport.

### V.5.2 Message Key Derivation

```
message_key = cSHAKE256(
  shared_secret,
  domain = "PQSF-KEYMAIL:message-key",
  input  = canonical({
    "message_id": message_id,
    "sender_identity_ref": sender_identity_ref,
    "recipient_identity_ref": recipient_identity_ref,
    "issued_tick": issued_tick
  })
)
```

### V.5.3 Encryption

```
(ciphertext, tag) = AEAD.Encrypt(
  key   = message_key,
  nonce = nonce,
  aad   = canonical({
    "message_id": message_id,
    "sender_identity_ref": sender_identity_ref,
    "recipient_identity_ref": recipient_identity_ref,
    "content_type": content_type
  }),
  plaintext = payload
)
```

---

## V.6 Validation Requirements

A KeyMailEnvelope is valid if and only if:

1. **Structural validity**: All required fields present and correctly typed
2. **Canonical encoding**: Envelope is valid Deterministic CBOR
3. **Signature verification**: sender_signature verifies under the sender's public key referenced by sender_identity_ref
4. **Time validity**: issued_tick ≤ current_tick < expiry_tick (when freshness is enforced)
5. **Session binding**: If session_id or exporter_hash is present, it matches the expected session context
6. **Decryption success**: ciphertext decrypts successfully with valid tag

Validation failure MUST NOT be silent. Implementations SHOULD report specific failure reasons for debugging.

---

## V.7 Thread Continuity

For multi-message exchanges:

1. All messages in a thread MUST share the same thread_id.
2. thread_id SHOULD be generated by the thread initiator and reused by all participants.
3. Thread membership is determined by thread_id only; sender/recipient pairs MAY vary within a thread.
4. Thread ordering is determined by issued_tick. Ties are broken by message_id lexicographic order.

---

## V.8 Error Codes

KeyMail defines the following error codes for validation failures:

| Error Code | Meaning |
|------------|---------|
| E_KEYMAIL_STRUCTURE_INVALID | Required fields missing or malformed |
| E_KEYMAIL_ENCODING_NONCANONICAL | Non-canonical CBOR encoding |
| E_KEYMAIL_SIGNATURE_INVALID | Sender signature verification failed |
| E_KEYMAIL_EXPIRED | Current tick ≥ expiry_tick |
| E_KEYMAIL_FUTURE | issued_tick > current_tick |
| E_KEYMAIL_SESSION_MISMATCH | Session binding mismatch |
| E_KEYMAIL_DECRYPTION_FAILED | AEAD decryption or tag verification failed |
| E_KEYMAIL_SENDER_UNKNOWN | sender_identity_ref cannot be resolved |
| E_KEYMAIL_RECIPIENT_MISMATCH | Message not addressed to this recipient |

---

## V.9 Security Considerations

### V.9.1 Forward Secrecy

KeyMail with ephemeral ML-KEM key agreement provides forward secrecy: compromise of long-term keys does not compromise past messages encrypted with ephemeral shared secrets.

For deployments requiring forward secrecy, implementations MUST:
- Use ephemeral key encapsulation per message or per session
- Securely delete ephemeral key material after use

### V.9.2 Replay Protection

KeyMail envelopes are self-describing and time-bounded, but replay protection is NOT inherent.

Consuming systems MUST:
- Track seen message_id values within a deployment scope
- Reject duplicate message_id submissions
- Consider expiry_tick when determining replay window

### V.9.3 Metadata Protection

KeyMail encrypts message content but does NOT encrypt:
- sender_identity_ref
- recipient_identity_ref
- issued_tick / expiry_tick
- thread_id
- content_type

Deployments with metadata confidentiality requirements MUST use additional transport-layer encryption (e.g., TLS, onion routing) or wrap KeyMailEnvelope in an outer EBTEnvelope.

### V.9.4 Non-Authority Reinforcement

Implementations MUST NOT:
- Interpret KeyMailEnvelope receipt as consent
- Trigger Authoritative operations based on message content without separate ConsentProof
- Propagate custody or signing authority through KeyMail
- Treat sender_signature as equivalent to ConsentProof signature

---

## V.10 Reference Implementation

```python
from dataclasses import dataclass
from typing import Optional
import os

@dataclass(frozen=True)
class KeyMailEnvelope:
    message_id: str
    thread_id: Optional[str]
    sender_identity_ref: str
    recipient_identity_ref: str
    issued_tick: int
    expiry_tick: int
    session_id: Optional[str]
    exporter_hash: Optional[bytes]
    content_type: str
    ciphertext: bytes
    nonce: bytes
    tag: bytes
    suite_profile: str
    sender_signature: bytes


def create_keymail(
    sender_identity_ref: str,
    recipient_identity_ref: str,
    payload: bytes,
    content_type: str,
    issued_tick: int,
    expiry_tick: int,
    sender_signing_key,
    recipient_public_key,
    suite_profile: str,
    thread_id: Optional[str] = None,
    session_id: Optional[str] = None,
    exporter_hash: Optional[bytes] = None
) -> KeyMailEnvelope:
    """
    Create a KeyMailEnvelope.
    """
    import uuid
    
    message_id = str(uuid.uuid4())
    
    # Key agreement (ML-KEM-1024)
    shared_secret, encapsulated_key = ml_kem_1024_encapsulate(recipient_public_key)
    
    # Derive message key
    key_context = canonical_cbor_encode({
        "message_id": message_id,
        "sender_identity_ref": sender_identity_ref,
        "recipient_identity_ref": recipient_identity_ref,
        "issued_tick": issued_tick
    })
    message_key = cshake256(
        shared_secret,
        domain=b"PQSF-KEYMAIL:message-key",
        input=key_context,
        output_length=32
    )
    
    # Generate nonce
    nonce = os.urandom(12)  # 96 bits for AES-256-GCM
    
    # Prepare AAD
    aad = canonical_cbor_encode({
        "message_id": message_id,
        "sender_identity_ref": sender_identity_ref,
        "recipient_identity_ref": recipient_identity_ref,
        "content_type": content_type
    })
    
    # Encrypt
    ciphertext, tag = aes_256_gcm_encrypt(
        key=message_key,
        nonce=nonce,
        plaintext=payload,
        aad=aad
    )
    
    # Build envelope without signature
    envelope_data = {
        "message_id": message_id,
        "thread_id": thread_id,
        "sender_identity_ref": sender_identity_ref,
        "recipient_identity_ref": recipient_identity_ref,
        "issued_tick": issued_tick,
        "expiry_tick": expiry_tick,
        "session_id": session_id,
        "exporter_hash": exporter_hash,
        "content_type": content_type,
        "ciphertext": ciphertext,
        "nonce": nonce,
        "tag": tag,
        "suite_profile": suite_profile
    }
    
    # Sign
    signing_payload = canonical_cbor_encode(envelope_data)
    sender_signature = sign(sender_signing_key, signing_payload, suite_profile)
    
    return KeyMailEnvelope(
        message_id=message_id,
        thread_id=thread_id,
        sender_identity_ref=sender_identity_ref,
        recipient_identity_ref=recipient_identity_ref,
        issued_tick=issued_tick,
        expiry_tick=expiry_tick,
        session_id=session_id,
        exporter_hash=exporter_hash,
        content_type=content_type,
        ciphertext=ciphertext,
        nonce=nonce,
        tag=tag,
        suite_profile=suite_profile,
        sender_signature=sender_signature
    )


def validate_keymail(
    envelope: KeyMailEnvelope,
    current_tick: int,
    sender_public_key,
    expected_session_id: Optional[str] = None,
    expected_exporter_hash: Optional[bytes] = None,
    seen_message_ids: Optional[set] = None
) -> tuple[bool, Optional[str]]:
    """
    Validate a KeyMailEnvelope.
    Returns (valid, error_code).
    """
    # Check replay
    if seen_message_ids is not None:
        if envelope.message_id in seen_message_ids:
            return False, "E_KEYMAIL_REPLAY"
    
    # Check time bounds
    if envelope.issued_tick > current_tick:
        return False, "E_KEYMAIL_FUTURE"
    
    if current_tick >= envelope.expiry_tick:
        return False, "E_KEYMAIL_EXPIRED"
    
    # Check session binding
    if expected_session_id is not None and envelope.session_id != expected_session_id:
        return False, "E_KEYMAIL_SESSION_MISMATCH"
    
    if expected_exporter_hash is not None and envelope.exporter_hash != expected_exporter_hash:
        return False, "E_KEYMAIL_SESSION_MISMATCH"
    
    # Verify signature
    envelope_data = {
        "message_id": envelope.message_id,
        "thread_id": envelope.thread_id,
        "sender_identity_ref": envelope.sender_identity_ref,
        "recipient_identity_ref": envelope.recipient_identity_ref,
        "issued_tick": envelope.issued_tick,
        "expiry_tick": envelope.expiry_tick,
        "session_id": envelope.session_id,
        "exporter_hash": envelope.exporter_hash,
        "content_type": envelope.content_type,
        "ciphertext": envelope.ciphertext,
        "nonce": envelope.nonce,
        "tag": envelope.tag,
        "suite_profile": envelope.suite_profile
    }
    signing_payload = canonical_cbor_encode(envelope_data)
    
    if not verify_signature(sender_public_key, signing_payload, envelope.sender_signature, envelope.suite_profile):
        return False, "E_KEYMAIL_SIGNATURE_INVALID"
    
    # Track message_id
    if seen_message_ids is not None:
        seen_message_ids.add(envelope.message_id)
    
    return True, None


def decrypt_keymail(
    envelope: KeyMailEnvelope,
    recipient_private_key,
    encapsulated_key: bytes
) -> tuple[Optional[bytes], Optional[str]]:
    """
    Decrypt a KeyMailEnvelope payload.
    Returns (plaintext, error_code).
    """
    # Decapsulate shared secret
    shared_secret = ml_kem_1024_decapsulate(recipient_private_key, encapsulated_key)
    
    # Derive message key
    key_context = canonical_cbor_encode({
        "message_id": envelope.message_id,
        "sender_identity_ref": envelope.sender_identity_ref,
        "recipient_identity_ref": envelope.recipient_identity_ref,
        "issued_tick": envelope.issued_tick
    })
    message_key = cshake256(
        shared_secret,
        domain=b"PQSF-KEYMAIL:message-key",
        input=key_context,
        output_length=32
    )
    
    # Prepare AAD
    aad = canonical_cbor_encode({
        "message_id": envelope.message_id,
        "sender_identity_ref": envelope.sender_identity_ref,
        "recipient_identity_ref": envelope.recipient_identity_ref,
        "content_type": envelope.content_type
    })
    
    # Decrypt
    try:
        plaintext = aes_256_gcm_decrypt(
            key=message_key,
            nonce=envelope.nonce,
            ciphertext=envelope.ciphertext,
            tag=envelope.tag,
            aad=aad
        )
        return plaintext, None
    except Exception:
        return None, "E_KEYMAIL_DECRYPTION_FAILED"
```

---

## V.11 Test Vector Requirements

Implementations MUST provide test vectors demonstrating:

- ☐ KeyMailEnvelope round-trip (encode → decode → encode produces identical bytes)
- ☐ Signature verification success with valid sender key
- ☐ Signature verification failure with wrong sender key
- ☐ Decryption success with correct recipient key
- ☐ Decryption failure with wrong recipient key
- ☐ Rejection of expired messages (current_tick ≥ expiry_tick)
- ☐ Rejection of future-dated messages (issued_tick > current_tick)
- ☐ Session binding enforcement when exporter_hash is present
- ☐ Replay detection via message_id tracking

---

## V.12 Conformance

An implementation is KeyMail-conformant if it:

1. Produces only canonical CBOR KeyMailEnvelope artefacts
2. Validates all required fields before processing
3. Verifies sender_signature before trusting message content
4. Enforces time bounds when freshness is required
5. Does NOT interpret KeyMailEnvelope as authority, consent, or execution capability
6. Implements replay protection via message_id tracking
7. Supports at least one post-quantum key agreement scheme (ML-KEM-1024)

---

*KeyMail provides secure message transport only. All authority and enforcement semantics are defined by PQSEC.*

---

Changelog
Version 2.0.2 (Current)
Encoding Determinism: Standardized the use of JCS (JSON Canonicalization Scheme) and deterministic CBOR for byte-stable hashing and signing.

Crypto-Agility: Enhanced the CryptoSuiteProfile mechanism to allow for seamless algorithm transitions (e.g., NIST Round 4 candidates) without requiring specification rewrites.

Session Binding: Refined the Exporter Binding primitives to enable cryptographic session-scoped security that remains independent of transport-layer trust.

EBT Implementation: Standardized the Encrypted-Before-Transport (EBT) wrapper for sensitive cryptographic and identity artefacts.

---

## **ACKNOWLEDGEMENTS (INFORMATIVE)**

This specification builds on decades of work in cryptography, secure protocol design, distributed systems, and formal verification.

The author acknowledges the foundational contributions of the following individuals and communities, whose work made the concepts formalised in **PQSF** possible:

* **Ralph Merkle** — for Merkle trees and tamper-evident state commitment, foundational to deterministic ledgers and integrity verification.
* **Whitfield Diffie** and **Martin Hellman** — for public-key cryptography and key exchange, enabling modern secure protocols.
* **Ron Rivest**, **Adi Shamir**, and **Leonard Adleman** — for early public-key systems that shaped adversarial threat models later challenged by quantum computation.
* **Peter Shor** — for demonstrating the impact of quantum computation on classical cryptographic assumptions.
* **Daniel J. Bernstein** — for cryptographic engineering, constant-time design, misuse resistance, and adversarial robustness.
* **Hugo Krawczyk** — for HKDF, protocol composition, and modern key-derivation and binding primitives.
* **Kahn Academy of Secure Protocol Design / CFRG contributors** — for advancing modern protocol analysis, canonical encoding discipline, and cryptographic agility.
* **IETF CBOR and COSE working groups** — for deterministic encoding, structured data representation, and secure object signing foundations.
* **IETF TLS working group** — for exporter mechanisms and session-bound security primitives.
* **The NIST Post-Quantum Cryptography Project** — for standardising post-quantum cryptographic primitives suitable for long-lived, high-assurance systems.
* **Researchers and contributors to post-quantum KEM and signature schemes** — for practical, analyzable replacements for quantum-vulnerable primitives.
* **Bitcoin protocol and security researchers** — for adversarial thinking around replay resistance, deterministic encoding, and failure modes in hostile networks.

Acknowledgement is also due to open-source contributors, cryptographic reviewers, and independent researchers whose adversarial review culture continues to surface edge cases, refine threat models, and improve protocol correctness. Any remaining errors or omissions are the responsibility of the author.

---

If you find this work useful and wish to support continued development and public availability of the PQSF specification, donations are welcome:

**Bitcoin:**
`bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw`
