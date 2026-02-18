# PQSF -- Post-Quantum Security Framework

* **Specification Version:** 2.0.3
* **Status:** Implementation Ready
* **Date:** 2026
* **Author:** rosiea
* **Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
* **Licence:** Apache License 2.0 — Copyright 2026 rosiea
* **PQ Ecosystem:** CORE — The PQ Ecosystem is a post-quantum security framework built on deterministic enforcement, fail-closed semantics, and refusal-driven authority. Bitcoin is the reference deployment. It is not the scope.

---

## Summary

PQSF defines the security framework and canonical data grammar for the PQ ecosystem.

It specifies deterministic encoding, cryptographic suite indirection, hashing and signing inputs, session binding primitives, and artefact grammars so that identical inputs always produce identical bytes and verifiable security properties.

PQSF does not make security decisions and does not enforce policy. All authorisation and enforcement decisions based on PQSF artefacts are evaluated exclusively by PQSEC.

PQSF is designed for local execution within the Holder Execution Boundary to ensure that canonical encoding, hashing, and cryptographic binding occur before any data is exposed to a transport layer or external intermediary.

---

## Index

* [Summary](#summary)
* [Non-Normative Overview (Informative)](#non-normative-overview--for-explanation-and-orientation-only)

1. [Scope and Protocol Boundary](#1-scope-and-protocol-boundary)
2. [Non-Goals and Authority Prohibition](#2-non-goals-and-authority-prohibition)
3. [Threat Model](#3-threat-model)
4. [Trust Assumptions](#4-trust-assumptions)
5. [Architecture Overview](#5-architecture-overview)

   * [5A. Explicit Dependencies](#5a-explicit-dependencies)
6. [Conformance Keywords](#6-conformance-keywords)
7. [Canonical Encoding](#7-canonical-encoding)

   * [7.1 PQSF-Native Canonical Encoding (Deterministic CBOR Only)](#71-pqsf-native-canonical-encoding-deterministic-cbor-only)
   * [7.2 Strict CBOR Profile](#72-strict-cbor-profile)
   * [7.3 Fuzzing Resistance via Canonical Encoding](#73-fuzzing-resistance-via-canonical-encoding)
   * [7.4 Epoch Clock Canonical Encoding Exception (JCS Canonical JSON Only)](#74-epoch-clock-canonical-encoding-exception-jcs-canonical-json-only)
   * [7.5 Evidence Precision and Data Minimisation Discipline](#75-evidence-precision-and-data-minimisation-discipline-normative)
8. [CryptoSuiteProfile Indirection](#8-cryptosuiteprofile-indirection)

   * [8.7 Cryptographic Sunset Discipline](#87-cryptographic-sunset-discipline-normative)
9. [Hashing Interface](#9-hashing-interface)
10. [Signature Interface](#10-signature-interface)
11. [Domain Separation](#11-domain-separation)
12. [Deterministic Derivation](#12-deterministic-derivation)
13. [Universal Secret Derivation](#13-universal-secret-derivation)
14. [Epoch Clock Integration Objects](#14-epoch-clock-integration-objects)
15. [Exporter Binding Primitive](#15-exporter-binding-primitive)
16. [ConsentProof Grammar](#16-consentproof-grammar)
16A. [ConsentRevocation](#16a-consentrevocation-normative)
17. [Policy Objects and Governance Metadata](#17-policy-objects-and-governance-metadata)

* [17.1 PolicyObject](#171-policyobject)
* [17.2 PolicyBundle](#172-policybundle)
* [17A Receipt Export Policy](#17a-receipt-export-policy-normative)
* [17B Privacy-Preserving Fleet Telemetry Envelope](#17b-privacy-preserving-fleet-telemetry-envelope-normative)

18. [LedgerEntry Grammar](#18-ledgerentry-grammar)
19. [Merkle Ledger Serialization](#19-merkle-ledger-serialization)
20. [RecoveryCapsule Grammar](#20-recoverycapsule-grammar)
21. [Encrypted-Before-Transport Wrapper](#21-encrypted-before-transport-wrapper)
21X. [BufferedMeasurementEnvelope](#21x-bufferedmeasurementenvelope-normative)
22. [Supply Chain Artefact Types](#22-supply-chain-artefact-types)
22X. [CaptureReceipt and MediaCommitment](#22x-capturereceipt-and-mediacommitment-normative)
22Y. [BlobRef and FileCommitment](#22y-blobref-and-filecommitment-normative)
23. [Supply Chain Ledger Anchoring](#23-supply-chain-ledger-anchoring)
24. [Reproducible Build Support](#24-reproducible-build-support)
25. [No Trust in Network Location](#25-no-trust-in-network-location)
26. [Supply Chain Security Considerations](#26-supply-chain-security-considerations)
27. [Sovereign Transport Protocol Grammar (STP)](#27-sovereign-transport-protocol-grammar-stp)

   * [27.1 STP Message Requirements](#271-stp-message-requirements)
   * [27.2 STP Message Types](#272-stp-message-types)
     * [27.2.1 STP-INIT](#2721-stp-init)
     * [27.2.2 STP-ACCEPT](#2722-stp-accept)
     * [27.2.3 STP-RESUME](#2723-stp-resume)
     * [27.2.4 STP-ERROR](#2724-stp-error)
     * [27.2.5 STP-DATA](#2725-stp-data-normative)
     * [27.2.6 STP-CLOSE](#2726-stp-close-normative)
   * [27.3 STP Canonical Encoding](#273-stp-canonical-encoding-normative)
   * [27.4 STP Capability Negotiation](#274-stp-capability-negotiation)
   * [27.5 STP Handshake State Machine](#275-stp-handshake-state-machine-normative)
     * [27.5.1 Session States](#2751-session-states)
     * [27.5.2 Valid Transitions](#2752-valid-transitions)
     * [27.5.3 Timeout Requirements](#2753-timeout-requirements-normative)
     * [27.5.4 Resumption and Error Recovery Rules](#2754-resumption-and-error-recovery-rules-normative)
     * [27.5.5 Key Material Lifecycle](#2755-key-material-lifecycle-normative)
   * [27.6 STP-to-TLS Binding](#276-stp-to-tls-binding-normative)
     * [27.6.1 Transport Requirements](#2761-transport-requirements)
     * [27.6.2 Operational Classification](#2762-operational-classification)
     * [27.6.3 Exporter Binding](#2763-exporter-binding)
   * [27.6A STP-Layer PQ KEM Fallback](#276a-stp-layer-pq-kem-fallback-for-authoritative-sessions-normative)
   * [27.6B STP Integrity and KDF Primitives](#276b-stp-integrity-and-kdf-primitives-normative)
   * [27.7 STP Credential Lifecycle](#277-stp-credential-lifecycle-normative)
     * [27.7.1 Credential Derivation](#2771-credential-derivation)
     * [27.7.2 Domain Binding](#2772-domain-binding)
     * [27.7.3 Storage and Non-Extractability](#2773-storage-and-non-extractability)
     * [27.7.4 Credential Rotation (Atomic)](#2774-credential-rotation-atomic)
     * [27.7.5 Credential Revocation](#2775-credential-revocation)
     * [27.7.6 Legacy Credential Migration (Normative)](#2776-legacy-credential-migration-normative)
   * [27.8 STP Web Discovery (Informative)](#278-stp-web-discovery-informative)

28. [Failure Semantics](#28-failure-semantics)
29. [Conformance](#29-conformance)
30. [Security Considerations](#30-security-considerations)
31. [Conformance Verification Procedures](#31-conformance-verification-procedures)
    - [31.1 Canonical Encoding Verification](#311-canonical-encoding-verification-normative)
32. [Annex Additivity Invariant](#32-annex-additivity-invariant)
32A. [Schema Version Governance](#32a-schema-version-governance-normative)
32B. [Evidence Classification Vocabulary](#32b-evidence-classification-vocabulary-normative)
33. [Deferred Interoperability Negotiation (Informative)](#33-deferred-interoperability-negotiation-informative)
34. [Annexes](#34-annexes)

---

## Annexes

* [Annex A -- Canonical CBOR Profile Summary](#annex-a--canonical-cbor-profile-summary)
* [Annex B -- Example CryptoSuiteProfile](#annex-b--example-cryptosuiteprofile)
* [Annex C -- Reference Pseudocode (PQSF-Native)](#annex-c--reference-pseudocode-pqsf-native)
* [Annex D -- Reference Pseudocode (Epoch Clock Binding)](#annex-d--reference-pseudocode-epoch-clock-binding)
* [Annex E -- Replay Safety Checklist](#annex-e--replay-safety-checklist)
* [Annex F -- Privacy Policy Namespace](#annex-f--privacy-policy-namespace)
* [Annex G -- STP Session Lifecycle Examples (Expanded)](#annex-g--stp-session-lifecycle-examples-expanded)
* [Annex H -- KeyMail Overview (Informative)](#annex-h--keymail-overview-informative)
* [Annex I -- EBT Encryption Flow Example](#annex-i--ebt-encryption-flow-example)
* [Annex J -- Exporter Hash Derivation Example](#annex-j--exporter-hash-derivation-example)
* [Annex K -- Merkle Tree Construction Example](#annex-k--merkle-tree-construction-example)
* [Annex L -- PolicyBundle Validation Example](#annex-l--policybundle-validation-example)
* [Annex M -- Profile Update Acceptance Logic](#annex-m--profile-update-acceptance-logic)
* [Annex N -- Emergency Transition State Machine](#annex-n--emergency-transition-state-machine)
* [Annex O -- STP Capability Negotiation Example](#annex-o--stp-capability-negotiation-example)
* [Annex P -- ConsentProof Construction Example](#annex-p--consentproof-construction-example)
* [Annex Q -- LedgerEntry Chain Validation](#annex-q--ledgerentry-chain-validation)
* [Annex R -- RecoveryCapsule Activation Logic](#annex-r--recoverycapsule-activation-logic)
* [Annex S -- Test Vector Checklist](#annex-s--test-vector-checklist)
* [Annex T -- Performance Optimization Guidelines](#annex-t--performance-optimization-guidelines)
* [Annex U -- Security Considerations Summary](#annex-u--security-considerations-summary)
* [Annex V -- KeyMail: Secure Messaging Artefacts (Normative)](#annex-v--keymail-secure-messaging-artefacts-normative)
* [Annex W -- ReceiptEnvelope (Normative)](#annex-w--receiptenvelope-normative)
* [Annex X -- Message, Transcript, Session Scope, and Consent Primitives (Normative)](#annex-x--message-transcript-session-scope-and-consent-primitives-normative)
* [Annex Y -- Backup and Recovery Re-Authorisation (Optional, Normative)](#annex-y--backup-and-recovery-re-authorisation-optional-normative)
* [Annex Z -- Update and Policy Profile Integrity (Optional, Normative)](#annex-z--update-and-policy-profile-integrity-optional-normative)
* [Annex AA -- Adapter Integrity and Binding (Optional, Normative)](#annex-aa--adapter-integrity-and-binding-optional-normative)
* [Annex AB -- Human Identity Binding (Optional, Normative)](#annex-ab--human-identity-binding-optional-normative)
* [Annex AC -- Evidence Producer Governance (Normative)](#annex-ac--evidence-producer-governance-normative)
* [Annex SM -- StalenessStateMachine (Normative)](#annex-sm--stalenessstatemachine-normative)
* [Annex ED -- Operation-Scoped Emission Discipline (Normative)](#annex-ed--operation-scoped-emission-discipline-normative)
* [Annex AD -- Determinism Verification Harness (Normative)](#annex-ad--determinism-verification-harness-normative)
* [Annex AE -- STP KEM Fallback Derivation Example (Informative)](#annex-ae--stp-kem-fallback-derivation-example-informative)
* [Annex AF -- ReplayStore Interface (Informative)](#annex-af--replaystore-interface-informative)
* [Annex AG -- PQ Stack Starter Pack and Conformance Harness (Normative)](#annex-ag--pq-stack-starter-pack-and-conformance-harness-normative)
* [Annex AH -- Bootstrap in Hostile or Isolated Networks (Normative)](#annex-ah--bootstrap-in-hostile-or-isolated-networks-normative)
* [Annex AI -- Gateway Adapter Pattern for Non-PQ Services (Normative)](#annex-ai--gateway-adapter-pattern-for-non-pq-services-normative)

---

[Changelog](#changelog)

---

## Non-Normative Overview -- For Explanation and Orientation Only

This section is NOT part of the conformance surface.  
It is provided for explanatory and onboarding purposes only.

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

Protocol Boundary:
PQSF defines protocol grammar, canonical encoding, and structural validation requirements only.

Enforcement Boundary:
PQSF does not perform enforcement, gating, admission, refusal, escalation, lockout, freshness enforcement, monotonicity enforcement, or authority decisions. All such behaviour is defined exclusively by consuming specifications, including PQSEC and domain specifications.

Non-Protocol Behaviour:
Any implementation using PQSF primitives for enforcement, refusal, escalation, lockout, freshness enforcement, monotonicity enforcement, or authority decisions outside PQSEC is architecturally non conformant.

### 1.X Holder Execution Boundary Requirement (Normative)

Conformant implementations MUST perform all canonical encoding, hashing, signing, signature verification, and cryptographic profile binding within the holder's execution boundary.

Cryptographic artefact generation and validation MUST NOT be delegated to untrusted middleware, remote services, cloud infrastructure, or transport-layer intermediaries.

If canonical encoding or cryptographic binding occurs outside the holder execution boundary, the implementation is non-conformant.

This requirement exists to prevent middlebox mutation, encoding substitution, downgrade attacks, and signature-preimage manipulation prior to binding.

PQSF artefacts are considered authoritative only if their canonical bytes were generated and verified locally under the holder execution boundary.

Normative Annex Notice:
Annex AC (Evidence Producer Governance) is normative and defines requirements that directly affect enforcement in PQSEC. Implementations claiming PQSF conformance MUST implement Annex AC when consuming evidence producer artefacts.

---

## 2. Non-Goals and Authority Prohibition

PQSF does not define:

* predicate enforcement, refusal, escalation, lockout, backoff, or freeze semantics
* runtime integrity attestation semantics or drift classification semantics
* custody tiers, signing authority, quorum authority, or PSBT authority semantics
* AI behavioural gating, action class admission control, or model authority semantics
* Epoch Clock anchoring, issuance, mirror consensus enforcement, tick freshness enforcement, or tick reuse enforcement
* application business logic, UI behaviour, or user workflows

Authority Prohibition:
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

* Canonical Encoding Layer
  Deterministic CBOR strict profile for all PQSF-native signed, hashed, or compared objects.

* External Canonical Artefact Exception
  Epoch Clock profiles and ticks are externally canonicalized and MUST use JCS Canonical JSON exclusively. They are not PQSF-native CBOR objects.

* Cryptographic Suite Indirection Layer
  All cryptographic operations reference pinned CryptoSuiteProfile objects.

* Protocol Object Layer
  Canonical object grammars for PQSF-native protocol artefacts and wrappers.

* Transport Binding Layer
  Exporter hash derivation and binding fields for session-bound artefacts.

* Sovereign Transport Grammar Layer
  Deterministic STP message grammar.

* Ledger Serialization Layer
  Deterministic Merkle construction rules.

PQSF defines object structure and validation rules only. PQSF does not define operational behaviour.

---

## 5A. Explicit Dependencies

| Specification | Minimum Version | Purpose |
|---------------|-----------------|---------|
| Epoch Clock | >= 2.1.0 | JCS Canonical JSON artefact handling exception |

PQSF is foundational infrastructure. Most PQ specifications depend on PQSF; PQSF depends only on Epoch Clock for the canonical JSON encoding exception.

---

## 6. Conformance Keywords

The key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL are to be interpreted as described in RFC 2119.

---

## 7. Canonical Encoding

### 7.1 PQSF-Native Canonical Encoding (Deterministic CBOR Only)

1. All PQSF-native protocol objects that are hashed, signed, compared, or used in Authoritative operations MUST be encoded using Deterministic CBOR.
2. The CBOR encoding MUST follow RFC 8949 deterministic encoding rules with the additional strict constraints defined in Section 7.2.
3. Re-encoding a decoded object MUST produce byte-identical output.
4. Non canonical encodings MUST be rejected.
5. JSON representations MAY be used only for unsigned display or transport wrappers and MUST NOT be hashed or signed.

### 7.2 Strict CBOR Profile

The following constraints are mandatory:

1. Integer values MUST use the shortest encoding form.
2. Floating-point values MUST NOT appear in any canonical artefact.

   Rationale:
   * IEEE 754 floating-point has multiple binary representations:
     - NaN has 2^52 distinct bit patterns
     - Positive zero (+0.0) and negative zero (−0.0) are distinct
     - Subnormal numbers and rounding behaviour vary by platform
   * These properties violate byte-identical re-encoding requirements.
   * Non-deterministic encoding breaks hashing, signing, and verification.

   Permitted Alternatives:
   * Fixed-point integers: value + scale (e.g., 234 / 10000 = 0.0234)
   * Rational numbers: numerator + denominator
   * Integer units: smallest unit representation (e.g., milliseconds instead of seconds)

   Error Code: `E_ENCODING_NONCANONICAL`

   Exceptions: None. Floating-point values are forbidden in all
   PQSF-native canonical CBOR artefacts, including informational fields.

   Decoder Requirement: Decoders MUST reject CBOR additional information values 25, 26, and 27 (IEEE 754 half, single, double precision) under major type 7.
3. Indefinite-length items MUST NOT be used.
4. Map keys MUST be unique. Duplicate keys MUST cause rejection.
5. Map key ordering MUST be bytewise lexicographic over the encoded key bytes.
6. Text and byte strings are distinct and MUST NOT be coerced.
7. CBOR tags MUST NOT be used unless explicitly permitted by a consuming specification profile.


### 7.3 Fuzzing Resistance via Canonical Encoding

### 7.3.1 Canonical Encoding as Fuzzing Defense

Deterministic canonical encoding is PQSF’s primary defense against
structural fuzzing attacks.

Attack Model:
Adversaries may supply semantically equivalent but byte-different
encodings to exploit:
- parser ambiguities
- hash or signature mismatches
- state-machine edge cases
- inconsistent error handling

Defense Strategy:
Canonical encoding collapses all semantic equivalents into a single
byte representation. Structural ambiguity is eliminated as an attack
surface.

---

### 7.3.2 Eliminated Encoding Fuzzing Vectors

JSON Key Reordering
```json
{"a": 1, "b": 2}
{"b": 2, "a": 1}
```

Mitigation: CBOR map keys MUST be sorted bytewise lexicographically.

---

CBOR Length Encoding Variants

```
18 42      // integer 66 (short form)
19 00 42   // integer 66 (long form)
```

Mitigation: Only shortest encoding form is permitted.

---

Floating-Point Variants

```
+0.0, -0.0, NaN, subnormals
```

Mitigation: Floating-point values are forbidden entirely.

---

Indefinite-Length Encoding

```
9F 01 02 03 FF   // indefinite-length array
83 01 02 03      // definite-length array
```

Mitigation: Indefinite-length items MUST be rejected.

---

Unicode Normalization

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

### 7.5 Evidence Precision and Data Minimisation Discipline (Normative)

PQSF governs canonical encoding and structural determinism. It does not define what evidence producers collect. However, artefact construction MUST observe precision discipline when encoding evidence that contains location, biometric, telemetry, behavioural, or other potentially sensitive measurements.

**Principle: Minimum Necessary Precision**

Evidence producers SHOULD emit the minimum granularity required for the consuming predicate to evaluate deterministically.

Examples:

- If a predicate evaluates jurisdiction at the country level, latitude and longitude coordinates with sub-meter precision MUST NOT be emitted.
- If a predicate evaluates geofence membership within a 500m radius, precision beyond that radius resolution SHOULD NOT be encoded.
- If classification requires only categorical state (e.g., "IN_REGION"), raw coordinate emission SHOULD NOT occur.

This discipline applies to:

- Location artefacts
- Environmental telemetry
- Biometric signals
- Behavioural metadata
- Device-derived spatial coordinates

When a consuming specification defines acceptable precision tiers, producers MUST conform to those tiers.

**Authority Boundary**

PQSF does not enforce collection discipline and does not evaluate precision compliance. Precision discipline is the responsibility of the evidence-producing specification or implementation.

Violation of precision minimisation is not a structural invalidation event unless explicitly required by an active policy in a consuming specification.

This clause establishes a normative design constraint on artefact construction without expanding enforcement semantics.

Precision discipline MUST NOT alter the canonical encoding rules defined in §7.1 and §7.2.

---

## 8. CryptoSuiteProfile Indirection

PQSF replaces all hardcoded cryptographic algorithm identifiers with profile references.

### 8.1 CryptoSuiteProfile Definition

```
CryptoSuiteProfile = {
  profile_id: tstr,
  suite_class: "signature" / "hash" / "kem" / "aead" / "derivation",
  family: tstr,
  parameters: opaque_payload,
  parameters_schema: tstr,
  domain_set_id: tstr,
  test_vector_hash: bstr,
  valid_from_tick: uint,
  valid_until_tick: uint / null,
  revoked: bool,
  signature: bstr
}
```

The `parameters` field contains canonical CBOR bytes whose schema is identified by the combination of `suite_class` and `family`. The `parameters_schema` field is a tstr identifier for the parameter schema (e.g., `"ml-dsa-65.params.v1"`). Consuming specifications define the parameter schemas for each suite class and family combination.

### 8.2 Profile Rules

1. All cryptographic operations MUST reference a CryptoSuiteProfile by profile_id.
2. Profiles MUST be signed and pinned by hash.
3. Unknown, expired, or revoked profiles MUST cause rejection.
4. Profile downgrade is forbidden. A regressed profile is one whose security properties, parameter strength, or effective validity window are strictly weaker than a previously accepted profile within the same profile lineage.
5. Profile validity evaluation, including tick interpretation, is performed by consuming specifications. PQSF treats profiles as inert data structures only.
6. Profile revocation state MUST be discoverable via authenticated profile distribution or authenticated revocation artefacts defined by consuming specifications.
7. Profile revocation MUST be honoured immediately for Authoritative operations.

Note: Other specifications may define their own suite_profile namespaces (for example, Epoch Clock `pqec.*`) for artefacts that are not PQSF-native CBOR objects. Such namespaces are external to PQSF and are validated according to their originating specification.

### 8.3 Profile Distribution Architecture

#### 8.3.1 Distribution Mechanisms

Profile distribution provides the means for implementations to discover and update CryptoSuiteProfiles.

1. Bootstrap profiles MUST be hardcoded in implementations.
2. Profile updates MUST be signed by governance keys.
3. Profile discovery MAY use:
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

3. Client verifies M-of-N governance signatures over the canonical CBOR encoding of the ProfileUpdate with the `governance_sigs` field omitted.
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

Governance signatures MUST be computed over the canonical CBOR encoding of the ProfileRevocation with the `governance_sigs` field omitted.

#### 8.4.2 Revocation Requirements

1. Revocations MUST be signed by M-of-N governance keys.
2. Revocations MUST be checked before each Authoritative operation.
3. If superseded_by_profile_id is present, clients SHOULD automatically upgrade to the replacement profile.
4. Grace period: Non-Authoritative operations MAY continue for a grace window after `revoked_at_tick`. The grace window length is determined by the Epoch Clock profile's tick interval and SHOULD NOT exceed the equivalent of 48 wall-clock hours. Implementations MUST document the grace window in ticks.
5. Authoritative operations MUST refuse immediately upon revocation discovery.

#### 8.4.3 Revocation Discovery

Implementations MUST:

1. Check for revocations at startup.
2. Periodically poll revocation endpoints (RECOMMENDED: every 3600 seconds).
3. Cache revocation list locally.
4. Accept revocation push notifications from distribution infrastructure where available.

### 8.5 Emergency Cryptographic Transition

Emergency transitions address catastrophic cryptographic failures.

#### 8.5.1 Emergency Profile Requirements

In case of algorithm compromise:

1. Emergency Profile Creation:
   * CAN violate non-downgrade rule if compromise is cryptographically proven
   * MUST be signed by emergency_quorum (all governance keys, no threshold)
   * MUST include compromise_evidence_hash referencing public disclosure

2. Emergency Profile Structure:

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

Governance signatures MUST be computed over the canonical CBOR encoding of the EmergencyProfile with the `governance_sigs` field omitted.

#### 8.5.2 Transition Period

Emergency transitions follow a phased approach. Phase boundaries MUST be expressed in Epoch Clock ticks. The wall-clock equivalents below are informative guidance only, based on the Epoch Clock profile's tick interval.

1. Phase 1: Dual-Algorithm Mode (RECOMMENDED: equivalent of 7 days in ticks)
   * Accept both old (compromised) and new algorithms
   * Log all uses of old algorithm
   * Warn users about pending deprecation

2. Phase 2: Authoritative Restriction (RECOMMENDED: equivalent of days 7-30 in ticks)
   * Reject old algorithm for Authoritative operations
   * Continue accepting for Non-Authoritative operations
   * Escalate user warnings

3. Phase 3: Complete Deprecation (RECOMMENDED: equivalent of day 30+ in ticks)
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

Signature Algorithm Families:
* ML-DSA-65 → ML-DSA-87: PERMITTED (stronger)
* ML-DSA-87 → ML-DSA-65: FORBIDDEN (downgrade)
* ML-DSA-* → ECDSA: FORBIDDEN (quantum-vulnerable)
* ECDSA → ML-DSA-*: PERMITTED (upgrade to PQ)

Hash Algorithm Families:
* SHAKE256-256 → SHAKE256-512: PERMITTED (stronger)
* SHAKE256-512 → SHAKE256-256: FORBIDDEN (downgrade)
* SHAKE256-* → SHA256: FORBIDDEN (quantum concerns)
* SHA256 → SHAKE256-*: PERMITTED (upgrade to PQ)

KEM Algorithm Families:
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

#### 8.6.3 Decentralized Governance Protocol

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

Requirements:
1. `threshold_m` MUST be ≥ ⌈(total_n / 2)⌉ + 1.
2. Governance members MUST represent at least three independent organizations.
3. GovernanceConfig MUST be signed by the active governance threshold or bootstrap authority. The signature MUST be computed over the canonical CBOR encoding of GovernanceConfig with the `signature` field omitted.
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

The ProfileUpdateProposal `signature` MUST be computed over the canonical CBOR encoding of the proposal with the `signature` and `approvals` fields omitted. Each GovernanceApproval `signature` MUST be computed over the canonical CBOR encoding of the approval with its `signature` field omitted.

Activation MUST NOT occur unless the required threshold approvals are
received before `approval_deadline_tick`.

#### 8.6.3.3 Authority Boundary

Governance artefacts define update legitimacy only.
All acceptance, refusal, revocation, and enforcement semantics are
defined exclusively by consuming specifications.

---

### 8.7 Cryptographic Sunset Discipline (Normative)

#### 8.7.1 Purpose

This section defines governance invariants for retiring cryptographic algorithms from active CryptoSuiteProfiles. It ensures that algorithm transitions are monotonic, bounded, and deterministic, preventing indefinite dual-use windows, silent rollbacks, or ambiguous profile states.

#### 8.7.2 Sunset States

Every CryptoSuiteProfile algorithm identifier exists in exactly one of the following states at any given time:

```
SunsetState = "ACTIVE" / "SUNSET_PENDING" / "SUNSET_FINAL"
```

**ACTIVE:** The algorithm is fully supported. All operations using this algorithm proceed normally.

**SUNSET_PENDING:** The algorithm has been marked for retirement. A dual-acceptance window is open. During this window, artefacts signed or encrypted under the pending algorithm MUST be accepted alongside artefacts under the replacement algorithm. The dual-acceptance window is bounded by `sunset_deadline_tick`.

**SUNSET_FINAL:** The algorithm has been retired. Artefacts using this algorithm MUST be refused. There is no re-activation path.

#### 8.7.3 State Transitions

The only permitted state transitions are:

```
ACTIVE → SUNSET_PENDING → SUNSET_FINAL
```

1. Transitions MUST be monotonic. Reverse transitions (SUNSET_PENDING → ACTIVE, SUNSET_FINAL → SUNSET_PENDING or ACTIVE) are prohibited.
2. Each transition MUST be recorded in the CryptoSuiteProfile governance history as a signed governance event.
3. The ACTIVE → SUNSET_PENDING transition MUST include a `sunset_deadline_tick` defining the latest tick at which the dual-acceptance window closes.
4. If `current_verified_tick > sunset_deadline_tick` and the profile is still in SUNSET_PENDING, implementations MUST treat it as SUNSET_FINAL automatically.

#### 8.7.4 Dual-Acceptance Window

During SUNSET_PENDING:

1. Both the retiring algorithm and the replacement algorithm MUST be accepted for signature verification and decryption.
2. New signatures and encryptions SHOULD use the replacement algorithm. Policy MAY require this as MUST.
3. The dual-acceptance window MUST NOT exceed the configured `max_dual_window_ticks`. If no explicit maximum is configured, the default is `4320` ticks (approximately 30 days at 10-minute intervals).
4. Implementations MUST track which algorithm was used for each artefact during the dual-acceptance window for audit purposes.

#### 8.7.5 Rollback Prohibition

1. An algorithm that has entered SUNSET_PENDING MUST NOT return to ACTIVE.
2. An algorithm that has entered SUNSET_FINAL MUST NOT return to SUNSET_PENDING or ACTIVE.
3. Any attempt to reverse a sunset state transition MUST be treated as a governance violation and MUST be refused.
4. Emergency re-activation of a retired algorithm requires a new CryptoSuiteProfile version with a new profile_id. The retired algorithm under the old profile_id remains in SUNSET_FINAL permanently.

#### 8.7.6 Offline Deployment Rules

For deployments that may be offline when a sunset deadline passes:

1. The implementation MUST evaluate sunset state using the last verified tick at the time of the operation.
2. If the last verified tick is beyond `sunset_deadline_tick`, the algorithm MUST be treated as SUNSET_FINAL regardless of whether the implementation received an explicit state transition event.
3. Reconnection after an offline period MUST NOT re-open a closed dual-acceptance window.

#### 8.7.7 Authority Boundary

This section defines governance invariants only. It does not define enforcement behaviour, refusal semantics, or lockout implications. Enforcement behaviour is defined by consuming specifications (principally PQSEC).

Refusal code: `E_PROFILE_SUNSET_FINAL` (registered in PQSEC Annex AE.49)

---

## 9. Hashing Interface

1. Hashing operations MUST reference a hash-class CryptoSuiteProfile.
2. For PQSF-native objects, hash inputs MUST be canonical CBOR bytes.
3. For Epoch Clock artefacts, hash inputs MUST be the JCS Canonical JSON bytes of the artefact.
4. Digest length and algorithm behaviour are defined exclusively by the referenced profile.
5. Implementations MUST NOT vary digest length or parameters outside the profile definition.

### 9.1 Hash Function Strategy (Normative)

The PQ specification stack uses hash functions in three tiers. Each tier has exactly one canonical function.

Tier 1 -- Bitcoin Script Compatibility: SHA-256

SHA-256 is used ONLY where Bitcoin consensus requires it. This includes `OP_SHA256`-gated script paths, Bitcoin transaction hashing, and PSBT transaction identifier computation. SHA-256 MUST NOT be used for canonical hashing of PQ artefacts outside Bitcoin Script contexts.

Tier 2 -- Canonical Hashing (Non-Script): SHAKE256-256 (32-byte output)

SHAKE256-256 is the canonical hash function for all PQ artefact hashing outside Bitcoin Script. This includes content hashes, bundle hashes, artefact hashes, intent hashes, fingerprint hashes, tick body hashes, and Merkle tree leaf hashing. All SHAKE256 invocations MUST produce exactly 32 bytes (256 bits) of output.

Tier 3 -- Domain-Separated Key Derivation: cSHAKE256 with L=256

cSHAKE256 (NIST SP 800-185 §6.2) is used for PQSF key derivation requiring domain separation,
unless a consuming specification defines an explicit SHAKE256-256 domain-labelling construction
for its own derivations (for example PQHD §10.0--§10.2). Parameter binding: `cSHAKE256(X=input, L=256, N="", S=domain_string)` producing 32 bytes. The function-name string N is empty; the customization string S carries the domain label. This applies to PQSF secret derivation and PQSF EBT key derivation.

Note: PQHD key derivation is defined by PQHD itself. PQHD v1.2.0 uses SHAKE256-256
with explicit domain-labelled inputs and a 0x00 separator (see PQHD §10.0--§10.2).

Special Case -- SEAL Transport Key Derivation: HKDF-SHA256

SEAL's AEAD key derivation from ML-KEM shared secrets uses HKDF-SHA256 (RFC 5869). Parameters: `HKDF-SHA256(salt=nil, IKM=shared_secret, info="seal/aead", L=32)`. This is execution-layer transport security, not custody-layer key management. HKDF-SHA256 is the most widely implemented and tested KDF for hybrid KEM constructions.

### 9.1A Canonical Algorithm Identifier Case (Normative)

All cryptographic algorithm identifiers expressed as `tstr` values (e.g., `hash_alg`, `suite_profile` family identifiers, receipt body `hash_alg` fields) MUST use lowercase ASCII characters.

Examples:

- `shake256-256`
- `shake256-512`
- `sha256`
- `ml-dsa-65`
- `ml-kem-1024`

Identifiers are case-sensitive. Mixed-case or uppercase variants (e.g., `SHAKE256-256`) MUST be rejected.

Refusal code: `E_HASH_ALG_UNSUPPORTED`

### 9.1B Epoch Clock Hash Identifier Alias (Normative)

For Epoch Clock artefacts only (JCS Canonical JSON bytes processed under the Epoch Clock exception in §7.4), implementations MUST accept the hash identifier string `"shake-256-256"` as an alias of `"shake256-256"`.

This alias applies only to identifier-string comparison and registry matching. The underlying hash function is SHAKE256 with 256-bit output (32 bytes) in both cases.

Outside the Epoch Clock artefact context, `"shake-256-256"` MUST be treated as unsupported unless explicitly permitted by policy.

Forward rule: all newly introduced Epoch Clock identifier strings MUST use lowercase ASCII per §9.1A. §9.1B exists solely for backward compatibility with the inscribed v2 profile identifier `"shake-256-256"`.

### 9.2 Canonical Typed Values (Normative)

To ensure deterministic canonical encoding, the following shared types are defined for use across all PQ specifications:

```
; Canonical typed value -- deterministic under RFC 8949 §4.2
typed_value = uint / int / tstr / bstr / bool / null

; Typed map -- all values must be typed_value
typed_map = { * tstr => typed_value }

; Opaque canonical payload -- bytes that are themselves canonical CBOR
; Consuming specification defines the schema via context
opaque_payload = bstr
```

Fields previously typed as `{ * tstr => any }` MUST use one of:
1. `typed_map` -- when values are simple primitives (configuration, metadata)
2. `opaque_payload` -- when content varies by context and is defined by consuming specifications

The `opaque_payload` pattern means the bytes inside are themselves canonical CBOR, but the enclosing structure treats them as opaque bstr. Hashing and signing operate on the outer structure's canonical encoding, which is deterministic because bstr has exactly one canonical encoding.

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

Tier 2 content-addressing hashes (e.g., bundle_hash, content_hash, Merkle leaf/node hashes) are exempt from textual domain labels unless explicitly specified by the producing specification; domain separation for Tier 2 is provided by schema-defined input structure and/or single-byte separators (e.g., Merkle 0x00/0x01).

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
  X = root_seed || canonical(context_parameters),
  L = 256,
  N = "",
  S = "PQSF-Secret:<context>"
)
```

Where X is the input string (concatenation of root_seed and canonical context parameters), L=256 is the output length in bits (producing 32 bytes), N is the empty function-name string, and S is the customization string carrying the domain label. See §9.1 Tier 3 for the canonical parameter binding.

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
* `"PQSF-Secret:ProviderCredential"` - For model provider credential derivation (PQ Gateway)

Implementations MAY define additional context strings using the `"PQSF-Secret:"` prefix.

Note: PQHD defines its own custody-layer derivation construction using SHAKE256-256 with explicit domain-labelled inputs and a 0x00 separator (PQHD §10.0--§10.2). PQHD derivation is distinct from PQSF cSHAKE256 derivation and MUST NOT be conflated.

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
  exporter_binding_id: "pqsf.exporter_context.v1",
  session_id: bstr(16),
  role_id: tstr,
  operation_class: "Authoritative" / "NonAuthoritative"
}
```

The `exporter_binding_id` is a stable binding scheme identifier. It MUST only change when the exporter context structure itself changes, not when the PQSF specification version changes. This prevents spec version bumps from invalidating active sessions.

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
  session_id: bstr(16),
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

## 16A. ConsentRevocation (Normative)

A ConsentRevocation is a cryptographically verifiable artefact that invalidates a previously issued ConsentProof prior to its expiry.

```
ConsentRevocation = {
  revocation_id: bstr(16),
  consent_id: tstr,
  revoked_at_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Rules:

1. `revocation_id` MUST be unique per revocation artefact. Implementations MUST generate `revocation_id` from a CSPRNG. Consuming enforcement specifications MUST deduplicate by `revocation_id` and define retention/GC semantics for the deduplication store.
2. `consent_id` MUST reference the `consent_id` field of a ConsentProof.
3. `revoked_at_tick` MUST be a verified Epoch Clock tick.
4. The signature MUST verify under the same authority that issued the referenced ConsentProof.
5. ConsentRevocation artefacts MUST be canonically encoded per PQSF rules.

PQSF defines structural and signature validity for ConsentRevocation. Whether a ConsentRevocation applies to a given enforcement evaluation is determined by PQSEC (see PQSEC §20A).

ConsentRevocation grants no authority and conveys no permission. It exists solely to withdraw previously issued consent.

---

## 17. Policy Objects and Governance Metadata

### 17.1 PolicyObject

A PolicyObject is a canonical, inert policy declaration consumed by enforcement systems.

PQSF defines the structure and canonical validation rules for PolicyObject. PQSF does not define enforcement semantics.

A PolicyObject MUST contain the following fields:

```
PolicyObject = {
  policy_id: tstr,
  policy_body: opaque_payload,
  policy_schema: tstr,
  policy_hash: bstr
}
```

Field Requirements:

* policy_id  
  A tstr identifier. The namespace and interpretation of policy_id are defined by consuming specifications. Policy identifiers SHOULD use a prefix namespace.

* policy_body  
  An opaque_payload (bstr) containing canonical CBOR bytes defining policy parameters. The internal schema of policy_body is identified by `policy_schema`. PQSF defines only canonical encoding and hashing rules.

* policy_schema  
  A tstr identifier for the schema used to interpret policy_body. The namespace and interpretation are defined by consuming specifications.

* policy_hash  
  A bstr equal to the hash of policy_body bytes under the active CryptoSuiteProfile.

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

Field Requirements:

* bundle_id  
  A unique tstr identifier for this specific bundle instance.

* policy  
  A PolicyObject as defined in Section 17.1.

* issued_tick  
  A uint representing the issuing tick. Interpretation and freshness enforcement are external to PQSF.

* exporter_hash  
  A bstr or null. When present, the bundle is bound to a specific session.

* governance  
  A deterministic map containing governance metadata. If present, it MUST include:

  * version - Current policy version number (monotonically increasing)
  * min_version - Minimum acceptable version (cannot roll back below this)
  * signers - List of authorized signer identifiers or public key references
  * threshold - M-of-N threshold for policy updates (M signatures required)
  * rollback_denylist - List of policy_hash values that cannot be reactivated
  * rotation_lock_ticks - Minimum tick interval between policy updates

* suite_profile  
  A tstr referencing the CryptoSuiteProfile used for signing.

* signature  
  A bstr computed over the canonical CBOR encoding of the bundle with the signature field omitted.

Validation Requirements:

PQSF defines structure and validation only. Governance monotonicity, rollback prevention, and refusal semantics are defined exclusively by consuming specifications.

The `signers` field identifies public key identifiers or profile references as defined by the consuming specification.

Policy evaluation semantics are external.

---

## 17A. Receipt Export Policy (Normative)

### 17A.1 Scope

This section defines a policy namespace for controlling storage, export, and redaction of ReceiptEnvelope objects (see Annex W).

Receipt export controls are privacy controls only. They MUST NOT be used as authority signals.

### 17A.2 Policy Namespace

Receipt export policies MUST use PolicyObject identifiers in the `receipt_export:` namespace.

Example:

```
policy_id = "receipt_export:v1"
```

### 17A.3 ReceiptExportPolicy Object

```
ReceiptExportPolicy = {
  v: uint,                                 ; MUST be 1
  default_mode: "LOCAL_ONLY" / "EXPORT_ALLOWED",
  allow_types: [* tstr] / null,            ; allowlist of receipt types; null = none
  deny_types:  [* tstr] / null,            ; denylist; takes precedence over allow_types
  redaction: {
    mode: "HASH_ONLY" / "REDACT_FIELDS" / "NONE",
    redact_fields: [* tstr] / null,        ; e.g. ["miner_id","endpoint_url","device_id"]
    hash_fields: [* tstr] / null           ; fields replaced with hash(field_bytes)
  },
  retention: {
    max_age_ticks: uint,
    max_count: uint
  },
  export: {
    require_user_approval: bool,
    require_policy_approval: bool          ; for enterprise deployments
  }
}
```

All maps and arrays MUST be deterministic CBOR under PQSF canonical encoding rules.

### 17A.4 Default Rules

1. `default_mode` MUST default to `LOCAL_ONLY`.
2. If `default_mode=LOCAL_ONLY`, no receipts may be exported unless explicitly allowed by `allow_types`.
3. If `deny_types` contains a type, it MUST NOT be exported even if present in `allow_types`.
4. Redaction MUST be applied before export.

### 17A.5 Redaction Requirements

If `redaction.mode != "NONE"`:

1. Redaction MUST produce deterministic output for identical inputs.
2. Redaction MUST NOT change receipt type, `v`, or signature-verification semantics for local storage.
3. Exported receipts MUST be considered non-authoritative evidence copies and MUST NOT be fed back into PQSEC as enforcement inputs.

If `redaction.mode="HASH_ONLY"`:

The exporter MUST produce an export record containing only:

* `type`
* `v`
* `body_hash`
* `signature_hash`
* `issued_tick`
* `expiry_tick` (if present)

### 17A.6 Export Prohibition for Correlation-Sensitive Fields

The following receipt fields MUST be treated as correlation-sensitive and MUST be redacted or hashed unless the policy explicitly sets `redaction.mode="NONE"`:

* network endpoint identifiers (e.g., SEAL `miner_id`, endpoint URLs)
* device identifiers
* stable actor identifiers
* timing diagnostics not required for verification

### 17A.7 Authority Boundary

Receipt export policy enforcement MUST NOT:

* permit operations
* deny operations
* alter PQSEC predicate evaluation

Receipt export policy is privacy control only.

---

## 17B. Privacy-Preserving Fleet Telemetry Envelope (Normative)

### 17B.1 Purpose

This section defines a constant-shape, correlation-minimized telemetry envelope for fleet health monitoring. It enables aggregate operational visibility without creating per-device fingerprinting surfaces.

### 17B.2 FleetTelemetryEnvelope Structure

```
FleetTelemetryEnvelope = {
  v:                    uint,           ; MUST be 1
  window_start_tick:    uint,           ; start of measurement window
  window_end_tick:      uint,           ; end of measurement window
  event_counts:         { * tstr => uint },  ; event type → count
  fault_hashes:         [* bstr(32)],   ; deterministic fault hashes
  build_hash:           bstr(32) / null, ; deployment-scoped firmware family identifier
  device_class:         tstr / null,    ; optional device classification
  suite_profile:        tstr,
  signature:            bstr
}
```

All fields MUST be canonically encoded under PQSF deterministic CBOR rules. Signature MUST be computed over the canonical CBOR encoding of the envelope with the `signature` field omitted.

### 17B.3 Structural Anonymity Rules

1. No stable device identifier is permitted in the envelope body. Fields such as `device_id`, `serial_number`, or `mac_address` MUST NOT appear.
2. Timing MUST be windowed to ticks. Sub-tick timing (milliseconds, microseconds, nanoseconds) MUST NOT appear in any field.
3. `fault_hashes` MUST derive from a fixed deterministic vocabulary of fault types. Each fault hash is computed as `SHAKE256-256(canonical("PQSF-FAULT:" || fault_type_string))`. Implementations MUST NOT include stack traces, memory addresses, or other device-specific diagnostic data in fault hashes.
4. `build_hash`, when present, MUST represent a deployment-scoped firmware family identifier shared across multiple devices within a device class.
5. `build_hash` MUST NOT encode per-device uniqueness. If a firmware build uniquely identifies a single device, `build_hash` MUST be null.
6. Emission MUST be operation-scoped, not periodic background telemetry. FleetTelemetryEnvelopes MUST be emitted only in response to explicit operational events (e.g. maintenance cycle completion, fault threshold breach), not on a fixed timer.
7. Export of FleetTelemetryEnvelopes is governed by ReceiptExportPolicy (Section 17A). Correlation-sensitive field rules (Section 17A.6) apply.

### 17B.4 TelemetryBudget

When present, TelemetryBudget constrains the volume and frequency of telemetry emissions per device class.

```
TelemetryBudget = {
  v:                          uint,         ; MUST be 1
  window_ticks:               uint,         ; budget measurement window in ticks
  max_events_per_window:      uint,         ; maximum event count per window
  max_fault_hashes_per_window: uint,        ; maximum distinct fault hashes per window
  max_bytes_per_envelope:     uint,         ; maximum serialised envelope size in bytes
  min_emit_interval_ticks:    uint,         ; minimum interval between emissions from same source
  drop_policy:                tstr,         ; "refuse" / "clamp"
  device_class:               tstr / null,  ; when present, budget applies per-class
  suite_profile:              tstr,
  signature:                  bstr
}
```

#### 17B.4.1 Budget Enforcement

1. If TelemetryBudget is present, it MUST be enforced. There is no "no enforcement" option. Absence of TelemetryBudget is the mechanism for opting out of budgets.
2. When `drop_policy` is `"refuse"`: an envelope that would exceed any budget constraint MUST be rejected entirely. No partial emission is permitted.
3. When `drop_policy` is `"clamp"`: an envelope that would exceed budget constraints MUST be truncated to fit within the budget. Truncation MUST remove the lowest-priority events first (implementation-defined priority ordering). The envelope MUST indicate truncation occurred via a `truncated: true` field.
4. `drop_policy` MUST be `"refuse"` or `"clamp"`. No other values are permitted.
5. `min_emit_interval_ticks` MUST be enforced per emission source. Emissions attempted before the interval has elapsed MUST be suppressed.

### 17B.5 Event Count Shape Invariant

The `event_counts` map MUST include a fixed set of event type keys for each device class, regardless of whether events of that type occurred in the measurement window. Absent event types MUST be represented as zero counts, not omitted keys.

This ensures constant-shape envelopes that do not leak information through structural variation.

### 17B.6 Authority Boundary

FleetTelemetryEnvelope is an operational visibility artefact only. It does not grant authority, modify enforcement semantics, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC.

---

## 18. LedgerEntry Grammar

```
LedgerEntry = {
  event: tstr,
  tick: uint,
  payload: opaque_payload,
  prev_hash: bstr,
  suite_profile: tstr,
  signature: bstr
}
```

Requirements:

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

When a tree level has an odd number of nodes, the last node MUST be promoted to the next level without hashing or duplication. It is NOT duplicated.

This avoids the duplication vulnerability present in Bitcoin's Merkle tree construction (CVE-2012-2459) while maintaining a deterministic, interoperable construction. All conformant implementations MUST produce identical Merkle trees for the same set of leaf entries.

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

Requirements:

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
  X = root_seed || canonical(context_parameters),
  L = 256,
  N = "",
  S = "PQSF-EBT:<label>"
)
```

Where:
* `X` is the input string: root_seed (minimum 256 bits of entropy) concatenated with canonical CBOR encoding of context_parameters
* `L = 256` produces 32 bytes of key material
* `N = ""` (empty function-name string)
* `S` is the customization string with `<label>` identifying the specific EBT use case
* See §9.1 Tier 3 for the canonical cSHAKE256 parameter binding

The `context_parameters` are use-case specific and MAY differ from the fields used in the EBT_AAD (§21.5). Implementations SHOULD avoid using identical bytes for both key derivation context and AAD unless this coupling is intentional.

### 21.3 Supported AEAD Schemes

PQSF recognizes the following authenticated encryption schemes for EBT:

Required (Post-Quantum):
* ML-KEM-1024 + AES-256-GCM
  * Use ML-KEM-1024 to encapsulate a symmetric key
  * Use encapsulated key with AES-256-GCM for authenticated encryption

Optional (Classical Compatibility):
* AES-256-GCM (with pre-shared or derived keys)
* XChaCha20-Poly1305

Implementations claiming post-quantum compliance MUST support ML-KEM-1024 + AES-256-GCM.

### 21.4 EBT Nonce Requirements

EBT nonces MUST satisfy:

1. Generated from a cryptographically secure random number generator (CSPRNG).
2. Never reused with the same key.
3. Length requirements:
   * 96 bits (12 bytes) for AES-256-GCM
   * 192 bits (24 bytes) for XChaCha20-Poly1305
   * Scheme-specific for ML-KEM-based constructions

### 21.5 EBTEnvelope Structure

```
EBTEnvelope = {
  label: tstr,
  tick: uint / null,
  session_id: bstr(16) / null,
  exporter_hash: bstr / null,
  suite_profile: tstr,
  nonce: bstr,
  aad: bstr,
  ciphertext: bstr,
  tag: bstr
}
```

Field Requirements:

* label - Human-readable identifier for the envelope type
* tick - Optional timestamp for freshness tracking
* session_id - Optional session binding
* exporter_hash - Optional session exporter binding
* suite_profile - References the CryptoSuiteProfile defining the AEAD scheme
* nonce - The nonce/IV used for encryption (never reused)
* aad - Additional authenticated data. MUST be the canonical CBOR encoding (RFC 8949 §4.2) of the following map with keys in lexicographic byte order:
  ```
  EBT_AAD = {
    "label": tstr,
    "session_id": bstr(16) / null,
    "suite_profile": tstr,
    "tick": uint / null
  }
  ```
  AAD binds the encryption context to the envelope metadata. The AAD map MUST include exactly the fields shown. Null-valued optional fields MUST be included as CBOR null (not omitted).
* ciphertext - Encrypted payload
* tag - Authentication tag from the AEAD scheme

Validation Requirements:

1. EBTEnvelope MUST be canonical CBOR.
2. aad MUST be the canonical encoding of associated data fields.
3. ciphertext MUST be encryption of the canonical plaintext bytes.
4. tag MUST be the AEAD authentication tag.
5. Where exporter binding is used, EBTEnvelope exporter_hash MUST match session exporter_hash.

PQSF defines EBT structure and wrapping requirements only. Operational refusal behaviour is external.

---

## 21X. BufferedMeasurementEnvelope (Normative)

### 21X.1 Purpose

This section defines a deterministic store-and-forward measurement batching structure for offline sensors, trackers, and constrained devices that cannot emit evidence in real-time. It ensures that buffered measurements are tick-bounded, ordered, and export-governed, and that absence of connectivity does not cause partial emission or data loss.

### 21X.2 MeasurementRecord

```
MeasurementRecord = {
  v:                      uint,           ; MUST be 1
  observed_tick:          uint,           ; Epoch Clock tick at observation time
  evidence_descriptor:    EvidenceDescriptor / null,  ; embedded descriptor
  evidence_descriptor_ref: bstr(32) / null, ; hash reference to descriptor in EvidenceBundle
  measurement_type:       tstr,           ; measurement classification string
  value:                  bstr,           ; canonical CBOR-encoded measurement payload
  producer_id:            tstr / null,    ; evidence producer identifier
  sequence:               uint            ; monotonic sequence number within batch
}
```

1. Exactly one of `evidence_descriptor` or `evidence_descriptor_ref` MUST be present. If `evidence_descriptor` is present, `evidence_descriptor_ref` MUST be null, and vice versa.
2. When `evidence_descriptor_ref` is present, it MUST be the SHAKE256-256 hash of the canonical CBOR encoding of the referenced `EvidenceDescriptor` object, and the referenced descriptor MUST be resolvable within the same `BufferedMeasurementEnvelope` or an associated `EvidenceBundle`.
3. `observed_tick` MUST be the Epoch Clock tick that was current (or last verified) at the time the measurement was taken.
4. `value` MUST be canonical CBOR. The internal structure of the measurement payload is defined by the consuming specification and identified by `measurement_type`.
5. `sequence` MUST be monotonically increasing within a single batch.

### 21X.3 BufferedMeasurementEnvelope

```
BufferedMeasurementEnvelope = {
  v:                    uint,           ; MUST be 1
  batch_id:             bstr(16),       ; CSPRNG-generated batch identifier
  records:              [+ MeasurementRecord],  ; ordered measurement records
  window_start_tick:    uint,           ; earliest observed_tick in batch
  window_end_tick:      uint,           ; latest observed_tick in batch
  record_count:         uint,           ; MUST equal length of records array
  evidence_bundle:      [* EvidenceDescriptor] / null,  ; optional shared descriptors
  export_policy_ref:    tstr / null,    ; reference to governing ReceiptExportPolicy
  suite_profile:        tstr,
  signature:            bstr
}
```

Signature MUST be computed over the canonical CBOR encoding of the envelope with the `signature` field omitted.

### 21X.4 Ordering and Integrity Rules

1. Records within the `records` array MUST be ordered by `observed_tick` ascending. Where multiple records share the same `observed_tick`, ordering MUST be by `sequence` ascending.
2. `window_start_tick` MUST equal the `observed_tick` of the first record. `window_end_tick` MUST equal the `observed_tick` of the last record.
3. `record_count` MUST equal the actual number of records in the `records` array. Mismatches MUST cause rejection at the structural validation layer.
4. No gaps in `sequence` values are permitted within a single batch. The sequence MUST start at 0 and increment by 1 for each record.

### 21X.5 Batching Rules

1. Batches MUST be tick-bounded. All records in a single batch MUST have `observed_tick` values within the range [`window_start_tick`, `window_end_tick`].
2. Absence of connectivity MUST NOT cause partial emission. If a batch cannot be transmitted completely, the entire batch MUST be retained locally until transmission is possible.
3. Partial batches (incomplete record sets for a measurement window) MUST NOT be emitted. A batch is complete when all measurements for the bounded window have been recorded or the window has closed.
4. `batch_id` MUST be CSPRNG-generated and MUST NOT encode deterministic device attributes, timestamps, or sequence information.

### 21X.6 Export Governance

Export of BufferedMeasurementEnvelopes is governed by ReceiptExportPolicy (Section 17A) when `export_policy_ref` is present. Correlation-sensitive field rules (Section 17A.6) apply to all fields in the exported envelope.

When `export_policy_ref` is null, the envelope is governed by the default ReceiptExportPolicy of the active PolicyBundle.

### 21X.7 Authority Boundary

BufferedMeasurementEnvelope is a data container only. It does not grant authority, modify enforcement semantics, or create new enforcement predicates. All enforcement decisions based on buffered measurements remain exclusively within PQSEC.

---

## 21B. Operational Boundary Requirements for Remote Storage (Normative)

This subsection defines operational boundary requirements for deployments that use the EBT wrapper defined in §21 for remote storage or cloud persistence.

The structural definition of EBT (including `EBTEnvelope`, key derivation, AEAD usage, and nonce discipline) remains exclusively defined in §21. This subsection does not redefine those structures.

### 21B.1 Holder Execution Boundary Enforcement

All encryption of canonical artefacts under the EBT wrapper MUST occur within the Holder Execution Boundary.

Encryption performed at a storage-provider edge, within remote middleware, within a cloud-managed Trusted Execution Environment not contained within the Holder Execution Boundary, or via Server-Side Encryption (SSE) after plaintext transmission is non-conformant.

Plaintext canonical artefacts that are hashed, signed, compared, or bound to enforcement MUST NOT cross the Holder Execution Boundary prior to encryption.

### 21B.2 Required Sequence of Operations

For any artefact persisted outside the Holder Execution Boundary, conformant implementations MUST perform the following sequence strictly within the Holder Execution Boundary:

1. Canonical encoding per §7 (Deterministic CBOR or JCS Canonical JSON where explicitly defined).
2. Hashing and signature binding under the active CryptoSuiteProfile.
3. EBT encryption of the fully bound artefact.
4. Emission of only the opaque ciphertext envelope outside the Holder Execution Boundary.

Reordering these steps is non-conformant.

### 21B.3 Metadata Containment

Remote storage providers MUST NOT have visibility into:

* Plaintext filenames.
* Directory hierarchies.
* Canonical artefact structure.
* Policy metadata.
* Enforcement metadata.
* Consent or session binding data.

Where object identifiers or storage metadata are required, they MUST NOT encode semantic information derived from plaintext content.

To a storage provider, an EBT-protected volume MUST appear as an authenticated sequence of opaque ciphertext objects.

### 21B.4 Authority Boundary

This subsection defines confidentiality and boundary requirements only.

It does not grant authority, modify predicate semantics, alter EnforcementOutcome production, or introduce new enforcement logic.

All enforcement decisions remain exclusively within PQSEC.

---

## 22. Supply Chain Artefact Types

PQSF defines deterministic artefact schemas for supply-chain integrity.

**Signing Input (Normative):** For any artefact in this section containing a `signature` field, the signature MUST be computed over the canonical CBOR encoding of the artefact with the `signature` field omitted. Verification MUST reconstruct the same canonical bytes. This rule applies to RuntimeSignature (§22.1), PublishSignature (§22.2), OperationKey (§22.3), DelegationCertificate (§22.4), and AuditSignature (§22.5).

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

Requirements:
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
  metadata: typed_map,
  issued_tick: uint,
  suite_profile: tstr,
  signature: bstr
}
```

Requirements:
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

Requirements:
1. OperationKey is single-use. The `operation_id` field is the replay key.
2. Consuming enforcement specifications (e.g. PQSEC) MUST reject reuse of the same `operation_id` within the policy-defined retention window and MUST define durable storage, retention/GC, and state-loss behaviour for replay detection.
3. Consuming enforcement specifications MUST treat `expiry_tick` as a hard validity boundary. Operations referencing an expired OperationKey MUST be refused.
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

Requirements:
1. DKs MUST be narrowly scoped
2. DKs MUST be short-lived. RECOMMENDED default: equivalent of 3600 seconds (1 hour) as determined by the active Epoch Clock profile's tick interval. TTL MUST be expressed in ticks via `expiry_tick`.
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

Requirements:
1. ARK signatures MUST be append-only
2. ARK fingerprints MAY be published publicly
3. Used for: WORM-style audit trails, regulator compliance

---

## 22X. CaptureReceipt and MediaCommitment (Normative)

### 22X.1 Purpose

This section defines tamper-evident capture commitments for media-producing devices. It enables cryptographic proof that a specific media artefact was captured at a specific tick by a specific device, without leaking device-identifying metadata.

### 22X.2 MediaCommitment

```
MediaCommitment = {
  v:                    uint,           ; MUST be 1
  content_hash:         bstr(32),       ; SHAKE256-256 hash of raw media content
  capture_tick:         uint,           ; Epoch Clock tick at capture time
  device_binding:       bstr(32),       ; hash of device attestation context (not device_id)
  media_type:           tstr,           ; MIME type or media classification
  resolution_hint:      tstr / null,    ; informative resolution/format hint
  suite_profile:        tstr,
  signature:            bstr
}
```

1. `content_hash` MUST be computed over the raw, unprocessed media bytes before any compression, transcoding, or metadata stripping.
2. `device_binding` MUST be a hash derived from the device's attestation context (e.g. runtime attestation, secure boot state). It MUST NOT be a stable device identifier, serial number, or hardware fingerprint.
3. `capture_tick` MUST be the Epoch Clock tick that was current (or last verified) at the time of capture.
4. Signature MUST be computed over the canonical CBOR encoding with the `signature` field omitted.

### 22X.3 CaptureReceipt

```
CaptureReceipt = {
  v:                    uint,           ; MUST be 1
  commitment_hash:      bstr(32),       ; SHAKE256-256 hash of canonical MediaCommitment
  capture_context:      tstr,           ; capture context classification
  session_id:           bstr(16) / null, ; optional session binding
  issued_tick:          uint,
  expiry_tick:          uint,
  suite_profile:        tstr,
  signature:            bstr
}
```

1. `commitment_hash` binds the receipt to a specific MediaCommitment without embedding the media content or device binding directly.
2. `capture_context` is a classification string indicating the circumstances of capture (e.g. `"user_initiated"`, `"automated_monitoring"`, `"regulatory_compliance"`). The vocabulary is deployment-defined.
3. CaptureReceipt MUST NOT contain raw media content, device identifiers, or geolocation data.
4. Signature MUST be computed over the canonical CBOR encoding with the `signature` field omitted.

### 22X.4 MediaDeleteReceipt

```
MediaDeleteReceipt = {
  v:                    uint,           ; MUST be 1
  commitment_hash:      bstr(32),       ; hash of the MediaCommitment being deleted
  destroyed_commitment: bstr(32),       ; hash proving destruction of media content
  deletion_tick:        uint,           ; tick at which deletion was performed
  deletion_method:      tstr,           ; "secure_erase" / "cryptographic_shred" / "verified_overwrite"
  suite_profile:        tstr,
  signature:            bstr
}
```

1. `destroyed_commitment` MUST be computed over evidence of content destruction. The exact evidence format is deployment-defined but MUST be deterministic and verifiable.
2. `deletion_method` MUST be one of the specified values. Unknown deletion methods MUST be rejected.
3. MediaDeleteReceipt governs media artefact lifecycle only.
4. Signature MUST be computed over the canonical CBOR encoding with the `signature` field omitted.

### 22X.5 Validation Requirements

1. All artefacts in this section MUST be canonically encoded under PQSF deterministic CBOR rules.
2. All hashes MUST use SHAKE256-256 (32 bytes) per PQSF §9 Tier 2.
3. All tick references MUST be valid Epoch Clock ticks.
4. Expired artefacts (past `expiry_tick`) MUST be rejected by consuming specifications.

### 22X.6 PQPS Interaction Clause

If media governed by MediaCommitment is also PQPS-managed state, the following precedence rules apply:

1. PQPS deletion semantics take precedence for state retention. MediaDeleteReceipt governs artefact lifecycle only and MUST NOT supersede PQPS deletion guarantees.
2. Absence of a PQPS `pqps.delete_receipt` MUST prevent claims of state-level deletion even if MediaDeleteReceipt exists.
3. A renderer MUST NOT display "deleted" for PQPS-managed state unless a `pqps.delete_receipt` is validated, regardless of MediaDeleteReceipt presence. This links to PQHR 4.9 rendering obligations. This clause is scoped to PQPS-managed state only and does not govern non-PQPS media deletion semantics.

### 22X.7 Authority Boundary

CaptureReceipt, MediaCommitment, and MediaDeleteReceipt are evidence artefacts only. They do not grant authority, modify enforcement semantics, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC.

---

## 22Y. BlobRef and FileCommitment (Normative)

This section defines canonical identification and commitment of arbitrary binary content within the PQ ecosystem. It does not define storage or transport mechanisms. It defines how bytes are referenced and bound deterministically.

### 22Y.1 BlobRef

A BlobRef is a deterministic reference to a byte sequence.

Canonical CBOR structure:

```
BlobRef = {
  hash: bstr(32),
  size: uint,
  media_type: tstr,
  name: tstr / null
}
```

Rules:

1. `hash` MUST be SHAKE256-256 computed over the exact raw byte sequence.
2. No newline normalization, Unicode normalization, transcoding, compression, or re-encoding is permitted unless explicitly defined by the consuming artefact type.
3. `name` and `media_type` are informative only and MUST NOT influence hash computation.
4. A BlobRef does not grant authority. It identifies content.
5. Identical byte sequences MUST produce identical BlobRef values.

### 22Y.2 FileCommitment

A FileCommitment binds a BlobRef to time and context.

Canonical CBOR structure:

```
FileCommitment = {
  schema: "pqsf.file_commitment.v1",
  blob: BlobRef,
  issued_tick: uint,
  context: tstr / null
}
```

Rules:

1. `issued_tick` MUST reference a valid Epoch Clock tick.
2. The commitment hash is computed over canonical CBOR bytes of the entire structure.
3. FileCommitment is evidence only. It does not imply storage.
4. FileCommitment MAY be included inside ReceiptEnvelope or other artefacts as a committed reference to external bytes.

FileCommitment ensures that "which exact bytes" is always deterministically bound to time and context.

### 22Y.3 Deterministic Bundle (Informative Pattern)

A PQ Bundle is a deterministic container for multiple artefacts and blobs. This pattern prevents archive metadata randomness from breaking reproducibility and signatures.

Bundle rules:

1. Bundle entries MUST be sorted lexicographically by path.
2. Each entry MUST contain a path (tstr) and a blob (BlobRef).
3. The bundle hash is SHAKE256-256 over canonical CBOR encoding of the sorted entry list.
4. Bundle metadata MUST NOT include timestamps, host identifiers, permission bits, or other environment-specific fields.

This pattern is informative and does not modify enforcement semantics.

### 22Y.4 Authority Boundary

BlobRef, FileCommitment, and Deterministic Bundle are evidence artefacts only. They do not grant authority, modify enforcement semantics, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC.

---

## 23. Supply Chain Ledger Anchoring

### 23.1 Cross-Registrar Anchoring

Supply-chain events (build, publish, revoke) MUST be:
1. Recorded in registrar logs
2. Anchored to deterministic ledger (as defined in Section 19)
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

Requirements:
1. Events MUST be tamper-evident
2. Monotonic tick ordering MUST be enforced
3. prev_hash chain MUST validate
4. Signature MUST be computed over the canonical CBOR encoding of the event with the `signature` field omitted

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

Signature MUST be computed over the canonical CBOR encoding of the attestation with the `signature` field omitted.

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

## 26. Supply Chain Security Considerations

### Threats Addressed

Supply-chain extensions address:

- Dependency poisoning: PUK + RSK + signed manifests
- CI credential theft: OSKs + short TTL
- Insider deploy sabotage: Scoped DKs + audit trail
- CDN injection: PUK verification at client
- Build/runtime mismatch: RSK startup attestation
- Silent config changes: Signed config manifests
- Log tampering: Ledger anchoring prevents history rewriting

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

```cbor
STP-INIT = {
  version: tstr,
  session_id: bstr(16),
  client_nonce: bstr,
  client_caps: [* tstr],
  kem_ciphertext: bstr / null     ; present when §27.6A fallback KEM is required
}
```

Purpose: Client initiates a new session.

Requirements:
* version MUST be "1.0" for this STP grammar version. STP versioning is independent of the PQSF document version; the STP `version` field tracks the STP grammar version only.
* session_id MUST be 16 bytes generated from CSPRNG and MUST be unique per session attempt.
* client_nonce MUST be generated from CSPRNG (minimum 16 bytes).
* client_caps lists client-supported capabilities.
* kem_ciphertext MUST be present (non-null) if and only if §27.6A fallback KEM is required for Authoritative eligibility. When not required, this field MUST be null.

#### 27.2.2 STP-ACCEPT

```cbor
STP-ACCEPT = {
  version: tstr,
  session_id: bstr(16),
  server_nonce: bstr,
  server_caps: [* tstr],
  exporter_hash: bstr(32),
  kem_confirm: bstr(32) / null    ; present when §27.6A fallback KEM is used
}
```

Purpose: Server accepts session and provides exporter binding.

Requirements:
* version MUST match STP-INIT version.
* session_id MUST match STP-INIT session_id.
* server_nonce MUST be generated from CSPRNG (minimum 16 bytes).
* server_caps lists server-supported capabilities (subset of client_caps).
* exporter_hash MUST be 32 bytes derived from the established transport session per §27.6.3.
* kem_confirm MUST be present (non-null) if and only if `kem_ciphertext` was present in the corresponding STP-INIT. When present, `kem_confirm = SHAKE256-256(shared_secret)` per §27.6A.

#### 27.2.3 STP-RESUME

```cbor
STP-RESUME = {
  version: tstr,
  previous_session_id: bstr(16),
  resume_token: bstr,
  client_nonce: bstr
}
```

Purpose: Client attempts to resume a previous session.

Requirements:
* previous_session_id references an earlier STP-ACCEPT session.
* resume_token is an opaque token provided by server in previous session.
* client_nonce MUST be fresh (never reused from previous session), minimum 16 bytes from CSPRNG.

#### 27.2.4 STP-ERROR

```cbor
STP-ERROR = {
  version: tstr,
  code: tstr,
  session_id: bstr(16),
  message: tstr / null
}
```

Purpose: Either party signals an error condition.

Requirements:
* code is a structured error code (defined by consuming specifications)
* session_id references the affected session
* message is an optional human-readable description

#### 27.2.5 STP-DATA (Normative)

```cbor
STP-DATA = {
  v: uint,                 ; MUST be 1
  session_id: bstr(16),
  sequence: uint,
  payload_type: tstr,
  payload: bstr,
  body_hash: bstr(32),
  mac: bstr
}
```

**Purpose:** Framed application data within an established STP session.

**Requirements:**

1. `session_id` MUST reference an active session in state `ESTABLISHED`.
2. `sequence` MUST be a monotonically increasing unsigned integer, starting at 0 after session establishment.
3. `sequence` MUST NOT repeat within a session. Implementations MUST reject any `STP-DATA` where `sequence ≤ highest_previously_received_sequence` for that `session_id`.
4. `payload_type` MUST be a registered PQSF artefact type identifier (e.g., `"ConsentProof"`, `"ReceiptEnvelope"`, `"EBTEnvelope"`, `"PolicyBundle"`, `"EpochTick"`).
5. `payload` MUST be:

   * canonical CBOR bytes as defined in §7, or
   * an EBT-wrapped payload whose inner body is canonical CBOR (see §21).
6. `body_hash` MUST equal `SHAKE256-256(payload)` (32 bytes).
7. `mac` MUST be computed using HMAC-SHA256 with `integrity_key` per §27.6B, over the canonical encoding of:

   `("STP-DATA" ‖ session_id ‖ sequence ‖ payload_type ‖ body_hash)`

8. Implementations MUST reject invalid `mac` before processing `payload` (per §27.6B).
9. If `payload_type` is not recognised by the receiver, the receiver MUST respond with `STP-ERROR` code `E_UNKNOWN_PAYLOAD_TYPE` and MUST NOT process the payload.

**Rationale:** `body_hash` aligns with PQSF hash discipline and allows independent artefact integrity verification; `mac` binds framing to the session and enforces sequence monotonicity.

#### 27.2.6 STP-CLOSE (Normative)

```cbor
STP-CLOSE = {
  v: uint,                 ; MUST be 1
  session_id: bstr(16),
  reason: tstr,
  final_sequence: uint,
  close_hash: bstr(32),
  mac: bstr
}
```

**Purpose:** Orderly session termination with key destruction mandate.

**Requirements:**

1. `session_id` MUST reference an active session.

2. `reason` MUST be one of: `"NORMAL"`, `"ERROR"`, `"TIMEOUT"`, `"POLICY_CHANGE"`, `"KEY_ROTATION"`.

3. `final_sequence` MUST equal the highest `sequence` number sent or received in this session by the sender.

4. `close_hash` MUST equal:

   `SHAKE256-256("STP-CLOSE" ‖ session_id ‖ final_sequence ‖ reason)`.

5. `mac` MUST be computed using HMAC-SHA256 with `integrity_key` per §27.6B, over the canonical encoding of:

   `("STP-CLOSE" ‖ close_hash)`

6. Upon sending `STP-CLOSE`, the sender MUST destroy all session key material within the same Epoch Tick boundary when ticks are available.

7. Upon receiving a valid `STP-CLOSE`, the receiver MUST destroy all session key material within the same Epoch Tick boundary when ticks are available.

8. After `STP-CLOSE`, any further `STP-DATA` on the same `session_id` MUST be rejected.

9. If key destruction cannot be confirmed, the implementation MUST:

   * emit a `ReceiptEnvelope` event indicating key-destruction failure (if receipts are enabled),
   * MUST NOT reuse the `session_id` or any resumption token derived from it,
   * MUST treat the session as `ERROR`-terminated for resumption eligibility (see §27.5.4).

### 27.3 STP Canonical Encoding (Normative)

All STP messages MUST be canonical CBOR as defined in §7.

All STP message types (`STP-INIT`, `STP-ACCEPT`, `STP-RESUME`, `STP-ERROR`, `STP-DATA`, `STP-CLOSE`) MUST be serialised using identical canonical encoding rules. Implementations MUST reject any STP message that does not round-trip to identical bytes under decode→encode re-encoding.

Canonical encoding MUST be applied before computing:
- `body_hash` (STP-DATA)
- `mac` (STP-DATA, STP-CLOSE)
- any signature fields contained within payload artefacts.

### 27.4 STP Capability Negotiation

1. Client advertises capabilities in client_caps.
2. Server MUST respond with server_caps containing only capabilities it supports.
3. The effective capability set is the intersection of client_caps and server_caps.
4. If intersection is empty or insufficient, server SHOULD respond with STP-ERROR.

### 27.5 STP Handshake State Machine (Normative)

STP sessions follow a deterministic state machine. Implementations MUST enforce these transitions. Any transition not listed below is INVALID and MUST cause session termination with `STP-ERROR` and immediate key destruction.

#### 27.5.1 Session States

| State | Description |
|---|---|
| IDLE | No active session. Client may send INIT or RESUME. |
| INIT_SENT | INIT sent; awaiting ACCEPT or ERROR. |
| RESUME_SENT | RESUME sent; awaiting ACCEPT or ERROR. |
| ESTABLISHED | ACCEPT received; DATA and CLOSE permitted. |
| CLOSING | CLOSE sent; awaiting peer CLOSE or forced close timeout. |
| CLOSED | Session terminated; keys destroyed. Terminal state. |
| ERROR | Unrecoverable error; keys destroyed. Terminal state. |

#### 27.5.2 Valid Transitions

```
IDLE -> INIT_SENT           (send INIT)
IDLE -> RESUME_SENT         (send RESUME)

INIT_SENT -> ESTABLISHED    (recv ACCEPT)
INIT_SENT -> ERROR          (recv ERROR or timeout)

RESUME_SENT -> ESTABLISHED  (recv ACCEPT)
RESUME_SENT -> ERROR        (recv ERROR or timeout)

ESTABLISHED -> ESTABLISHED  (send/recv DATA)
ESTABLISHED -> CLOSING      (send CLOSE)
ESTABLISHED -> CLOSED       (recv CLOSE)
ESTABLISHED -> ERROR        (send/recv ERROR)

CLOSING -> CLOSED           (recv CLOSE)
CLOSING -> CLOSED           (forced timeout)
```

Any transition not listed above is INVALID and MUST terminate the session with `STP-ERROR` and immediate key destruction.

#### 27.5.3 Timeout Requirements (Normative)

Timeouts are required to eliminate implementer ambiguity. All STP implementations MUST enforce timeout bounds.

| Transition | Default Timeout | Negotiable | Hard Bound |
|---|---:|---|---:|
| INIT_SENT → ERROR | 30 seconds | YES, via capability negotiation | MUST NOT exceed 120s |
| RESUME_SENT → ERROR | 15 seconds | YES, via capability negotiation | MUST NOT exceed 60s |
| ESTABLISHED → CLOSED (idle) | 3600 seconds | YES, via capability negotiation | MUST NOT exceed 86400s |
| CLOSING → CLOSED (forced) | 10 seconds | NO | Fixed |

Timeout measurement:

* Implementations MUST enforce timeouts using Epoch Clock ticks where available.
* If Epoch Clock is unavailable, monotonic local time MAY be used.
* Implementations MUST NOT use wall-clock time for timeout measurement.

#### 27.5.4 Resumption and Error Recovery Rules (Normative)

1. From `ERROR` state, the only permitted action is to return to `IDLE` and begin a new session with a fresh `session_id`.
2. `STP-RESUME` MUST NOT reference any session that entered `ERROR` state. Implementations MUST track error-terminated `session_id` values and reject resume attempts against them with `STP-ERROR` code `E_RESUME_FORBIDDEN`.
3. If three consecutive session attempts (INIT or RESUME) from the same counterparty result in `ERROR` within a single Epoch Clock tick, implementations SHOULD apply backoff discipline and MAY refuse further attempts for the remainder of that tick.

#### 27.5.5 Key Material Lifecycle (Normative)

* Session key material MUST be derived during the `*_SENT → ESTABLISHED` transition.
* Session key material MUST NOT exist in `IDLE`, `ERROR`, or `CLOSED` states.
* Transition to `CLOSED` or `ERROR` MUST trigger immediate key material destruction.
* Key material destruction SHOULD be logged via `ReceiptEnvelope` when receipts are enabled.

### 27.6 STP-to-TLS Binding (Normative)

STP operates as an overlay protocol within a TLS 1.3 (or later) protected channel.

#### 27.6.1 Transport Requirements

STP messages MUST be transmitted over a TLS 1.3 (RFC 8446) or later connection.

Implementations MUST compute and verify a TLS exporter binding for every STP session.

#### 27.6.2 Operational Classification

For **Non-Authoritative** operations (as classified by PQSEC):

* The underlying TLS connection MUST use at minimum a hybrid key exchange that includes one post-quantum KEM (e.g., X25519+ML-KEM-768) where available.
* If hybrid PQ key exchange is not available from the TLS layer, the STP session MUST be restricted to Non-Authoritative operations only.

For **Authoritative** operations:

* The underlying TLS connection SHOULD use post-quantum key exchange where available from the TLS layer.
* Where the TLS layer does not provide post-quantum key exchange, implementations MUST perform an additional ML-KEM-1024 key agreement at the STP layer during session establishment as defined in §27.6A.

#### 27.6.3 Exporter Binding

The `exporter_hash` in `STP-ACCEPT` MUST be derived using the TLS 1.3 exporter mechanism (RFC 8446 §7.5) with label:

`"PQSF-EXPORTER"`

Both parties MUST independently compute the exporter output and verify that `exporter_hash` matches. If the values do not match, the session MUST be terminated with `STP-ERROR` code `E_CHANNEL_BINDING_MISMATCH` and all session key material MUST be destroyed.

### 27.6A STP-Layer PQ KEM Fallback for Authoritative Sessions (Normative)

When TLS does not provide PQ key exchange and the session will carry Authoritative operations, STP MUST perform an ML-KEM-1024 key exchange at the STP layer during session establishment.

#### Fields

* `STP-INIT` MUST include an additional field: `kem_ciphertext: bstr` containing the ML-KEM-1024 encapsulation ciphertext.
* `STP-ACCEPT` MUST include an additional field: `kem_confirm: bstr(32)` where `kem_confirm = SHAKE256-256(shared_secret)`.

The initiator MUST verify `kem_confirm` before treating the session as eligible for Authoritative operations. If verification fails, the session MUST be terminated with `STP-ERROR` code `E_PQ_KEM_CONFIRM_FAILED`.

#### Key Derivation

HKDF in this section MUST be HKDF-SHA256 (RFC 5869) per §27.6B.

Let:

* `E = tls_exporter("PQSF-EXPORTER", context, 32)` -- the TLS exporter output. `E` MUST be exactly 32 bytes.
* `SS = stp_kem_shared_secret` -- the ML-KEM-1024 decapsulated shared secret. `SS` MUST be the raw shared secret bytes as output by the ML-KEM decapsulation API.

When STP-layer KEM is performed (Authoritative operations without PQ TLS):

```
PRK = HKDF-Extract(salt=E, ikm=SS)
session_key   = HKDF-Expand(PRK, info="PQSF-STP-SESSION"   ‖ session_id ‖ role_id, L=32)
integrity_key = HKDF-Expand(PRK, info="PQSF-STP-INTEGRITY" ‖ session_id ‖ role_id, L=32)
```

When PQ TLS is available (no STP-layer KEM needed):

```
PRK = HKDF-Extract(salt=0x00*32, ikm=E)
session_key   = HKDF-Expand(PRK, info="PQSF-STP-SESSION"   ‖ session_id ‖ role_id, L=32)
integrity_key = HKDF-Expand(PRK, info="PQSF-STP-INTEGRITY" ‖ session_id ‖ role_id, L=32)
```

#### Rules

1. Authoritative operations MUST NOT be processed unless either:

   * TLS provides PQ key exchange, or
   * STP-layer KEM fallback has been performed and verified successfully.

2. Failure to complete the KEM fallback (including `kem_confirm` verification failure) MUST produce `STP-ERROR` code `E_PQ_CHANNEL_REQUIRED` and MUST destroy all partial key material.

3. The `integrity_key` is the key used to compute `mac` fields in `STP-DATA` and `STP-CLOSE` per §27.6B.

4. The `session_key` is the key used for any application-layer encryption within the STP session (e.g., additional EBT wrapping of payload content).

### 27.6B STP Integrity and KDF Primitives (Normative)

STP uses the following cryptographic primitives for key derivation and message authentication:

1. **HKDF:** HKDF-SHA256 (RFC 5869). All HKDF operations in §27.6A MUST use HKDF-SHA256.
2. **MAC:** HMAC-SHA256 (RFC 2104). All `mac` fields in STP-DATA (§27.2.5) and STP-CLOSE (§27.2.6) MUST be computed using HMAC-SHA256.

MAC computation:

```
mac = HMAC-SHA256(key=integrity_key, data=stp_mac_preimage)
```

Where `stp_mac_preimage` is the canonical CBOR encoding of the preimage defined in the relevant section (§27.2.5 requirement 7 for STP-DATA, §27.2.6 requirement 5 for STP-CLOSE).

The `mac` output MUST be exactly 32 bytes.

Input constraints:

* `E` (TLS exporter output) MUST be exactly 32 bytes (TLS exporter output length parameter = 32).
* `SS` (ML-KEM shared secret, when §27.6A is used) MUST be the raw ML-KEM-1024 shared secret bytes as output by the ML-KEM decapsulation API.
* `integrity_key` and `session_key` MUST each be exactly 32 bytes.

Implementations MUST reject STP-DATA and STP-CLOSE frames with invalid `mac` before processing any payload or executing any state transition.

### 27.7 STP Credential Lifecycle (Normative)

STP sessions may require persistent credentials for authentication, session resumption, and domain binding. This section defines the credential lifecycle for STP credential material derived from the `PQSF-Secret:AuthToken` derivation context (§13).

#### 27.7.1 Credential Derivation

STP credentials MUST be derived using the Universal Secret Derivation mechanism (§13) with the following parameters:

```
credential = PQSF-Derive(
  context = "PQSF-Secret:AuthToken",
  domain  = domain_identifier,
  role    = role_identifier,
  tick    = issuance_tick,
  entropy = CSPRNG(256)
)
```

* `domain_identifier` MUST be a unique, stable identifier for the service or endpoint (e.g., an STP server's public key hash).
* `role_identifier` MUST match the BPC role under which the credential is used.
* `issuance_tick` MUST be the Epoch Clock tick at time of derivation.
* `entropy` MUST be fresh 256-bit material from a CSPRNG.

#### 27.7.2 Domain Binding

Credentials MUST be bound to exactly one domain. A credential derived for domain A MUST NOT be usable with domain B.

Domain binding MUST be enforced cryptographically by including `domain_identifier` in the derivation inputs (§27.7.1). The credential itself MUST NOT contain the domain in plaintext -- binding is implicit in the derivation.

#### 27.7.3 Storage and Non-Extractability

Where PQPS is available, credentials MUST be stored under PQPS sovereignty.

Where PQPS is unavailable, credentials MUST be:

* Encrypted at rest using EBT (§21) with a key derived from the holder's custody key material.
* Not stored in plaintext under any circumstances.

Device-binding requirement:

* Credentials MUST be non-extractable for use on a different device or enclave boundary by default.
* Export of credentials or equivalent authentication material MUST require explicit holder consent via ConsentProof.
* Credential export MUST be treated as an Authoritative operation subject to PQSEC enforcement.

If non-extractability cannot be enforced by the implementation, the implementation MUST disclose this limitation and MUST NOT claim high-assurance conformance.

#### 27.7.4 Credential Rotation (Atomic)

Credentials MUST be rotated:

* When the underlying CryptoSuiteProfile is updated.
* When the BPC role associated with the credential changes.
* At intervals not exceeding the `credential_max_lifetime` declared during STP capability negotiation (default: 2,592,000 seconds / 30 days).
* Immediately upon suspicion of compromise.

Rotation MUST follow an atomic sequence:

1. Derive `new_credential` with fresh entropy and current tick (§27.7.1).
2. Authenticate to the domain using `old_credential`.
3. Register `new_credential` with the domain.
4. Confirm `new_credential` via a challenge-response using the new credential.
5. Destroy `old_credential`.
6. Emit a rotation `ReceiptEnvelope` event (if receipts are enabled).

Failure semantics (mandatory):

* If step 4 fails, the implementation MUST destroy `new_credential`, retain `old_credential` as active, and emit a rotation-failure event (if receipts are enabled).
* The implementation MUST NOT leave both credentials active simultaneously.
* The implementation MUST NOT leave zero credentials active (unless revocation is intended).

#### 27.7.5 Credential Revocation

A credential MUST be revocable by:

* The holder (via ConsentProof asserting revocation).
* The domain (via server-side revocation, communicated as `STP-ERROR` code `E_CREDENTIAL_REVOKED`).
* Policy expiry (`credential_max_lifetime` exceeded without rotation).

Upon revocation:

* The credential MUST be destroyed locally.
* The domain MUST reject further authentication attempts using the revoked credential.
* Resumption tokens derived from the revoked credential's sessions MUST be invalid.
* A `RevocationEvidence` artefact SHOULD be generated where receipts are enabled.

### 27.7.6 Legacy Credential Migration (Normative)

This section defines a migration procedure for legacy credentials (API keys, OAuth refresh tokens, service passwords) into PQ-governed domain-scoped credentials for gateway adapters.

Migration is an Authoritative operation when it affects execution capability.

#### 27.7.6.1 Required Steps

1. Import legacy credential into holder boundary
2. Derive domain-scoped replacement credential (PQSF §13)
3. Register replacement with service (service-specific)
4. Attempt revocation of legacy credential (service-specific)
5. Emit migration receipt

No "atomic rollback" guarantee is made. Services differ. Operations requiring the migrated credential MUST fail closed unless a COMPLETE or PARTIAL migration receipt exists.

#### 27.7.6.2 Migration Receipt

**ReceiptEnvelope.type:** `"pqsf.credential_migration"`

**Body (deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version (MUST be 1) |
| `migration_id` | bstr (16 bytes) | Yes | Unique migration identifier |
| `service_id` | tstr | Yes | Target service domain |
| `credential_scope` | tstr | Yes | Scope of migrated credential |
| `legacy_credential_commitment` | bstr (32 bytes) | Yes | SHAKE256-256 of legacy credential (audit binding only) |
| `replacement_credential_commitment` | bstr (32 bytes) | Yes | SHAKE256-256 of PQ domain-bound replacement |
| `status` | tstr | Yes | `"COMPLETE"` / `"PARTIAL"` / `"FAILED"` |
| `legacy_revoked` | bool | Yes | Whether legacy credential was revoked at service |
| `issued_tick` | uint | Yes | Epoch Clock tick at migration |
| `signature` | bstr | Yes | Signature over canonical body with `signature` omitted |

#### 27.7.6.3 Migration Rules

1. Raw credential material MUST NOT appear in any artefact, receipt, log, or export.
2. Domain-scoped replacement MUST be derived per PQSF §13.
3. `PARTIAL` is permitted only when revocation is unsupported or unverifiable.
4. Legacy credentials MUST be stored encrypted (EBT, PQSF §21) if temporary storage is required.
5. Legacy credential material MUST be destroyed immediately after migration completion. "Migration completion" means: a COMPLETE or PARTIAL `pqsf.credential_migration` receipt has been emitted for the `migration_id`.
6. Both credentials MUST NOT be simultaneously active longer than the migration sequence window.

### 27.8 STP Web Discovery (Informative)

> **Note:** This subsection is informative. Web integration mechanisms are expected to evolve as PQ TLS standardisation matures.

For web-based STP deployment, the following discovery mechanisms are RECOMMENDED:

1. **HTTP header advertisement.**
   A server supporting STP MAY advertise this via an HTTP response header:
   ```
   PQ-STP: version="1.0"; endpoint="/pqsf/stp"; caps="pqsf-v2,cbor-canonical,ml-dsa-65"
   ```

2. **Well-known URI.**
   A server MAY publish STP configuration at:
   ```
   /.well-known/pqsf-stp.json
   ```
   containing endpoint URI, supported capabilities, and the server's PQ public key hash for credential binding.

3. **DNS TXT record.**
   A domain MAY publish STP support via:
   ```
   _pqsf-stp.example.com TXT "v=1.0; endpoint=https://example.com/pqsf/stp"
   ```

Discovery MUST NOT be treated as a substitute for session-level authentication. Exporter binding (§27.6) and credential authentication (§27.7) remain mandatory regardless of discovery mechanism.

---

## 28. Failure Semantics

1. Validation of any PQSF-native object MUST be atomic. Either all validation steps succeed and the object is accepted, or any single validation failure causes rejection with no partial acceptance.
2. Any PQSF-native object failing structural validation, canonical encoding validation, profile validation, hash validation, or signature verification MUST be rejected.
3. Any Epoch Clock artefact whose verified bytes do not match the supplied JCS Canonical JSON bytes MUST be rejected by consuming enforcement specifications.
4. PQSF defines no retry, escalation, or authority behaviour.

**Refusal Code Surface Rule (Normative):**
Any refusal code emitted outside the PQSF component boundary (including over APIs, in receipts, or in cross-specification documents) MUST be a PQSEC Annex AE code. PQSF-internal debug reasons MUST NOT be surfaced as interoperable refusal codes.

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
* implements STP message grammar correctly, including STP-DATA and STP-CLOSE framing
* enforces STP state machine transitions (§27.5) and rejects invalid transitions
* enforces STP timeout bounds (§27.5.3) and key material lifecycle (§27.5.5)
* enforces STP exporter binding (§27.6) and channel binding verification (§27.6.3)
* enforces STP-layer PQ KEM fallback (§27.6A) when Authoritative operations require PQ channel security and TLS does not provide it
* enforces STP credential domain binding, non-extractability, atomic rotation, and revocation (§27.7) where credentials are used
* validates EvidenceProducerProfile and EvidenceTypeConstraints artefacts per Annex AC
* satisfies Annex AD determinism verification requirements for all implemented artefact types



## 30. Security Considerations

### Threats Addressed

PQSF addresses the following threats within its defined scope:

- Encoding ambiguity and parser differentials:  
  Deterministic canonical encoding eliminates semantically equivalent but
  byte-distinct representations that can be exploited across
  implementations.

- Hash and signature mismatch attacks:  
  Byte-identical canonical encoding ensures that hashing and signing
  inputs are stable and reproducible across platforms.

- Algorithm downgrade and agility failures:  
  CryptoSuiteProfile indirection prevents hardcoded algorithm reliance
  and enables governed transitions without specification rewrites.

- Session confusion and cross-session replay:  
  Exporter binding primitives allow consuming specifications to bind
  artefacts to authenticated session state.

---

### Threats NOT Addressed (Out of Scope)

PQSF does NOT protect against:

- Enforcement errors or policy mistakes:  
  PQSF provides primitives only. Enforcement correctness is delegated to
  consuming specifications (e.g., PQSEC).

- Transport protocol flaws:  
  PQSF does not define or secure transport protocols themselves.

- Key compromise or runtime compromise:  
  Key management and runtime integrity are outside PQSF’s scope.

- Denial-of-service attacks:  
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

- Parser differentials:  
  Semantically identical inputs MUST canonicalize to a single byte
  representation or be rejected. Multiple interpretations are forbidden.

- Hash and signature mismatch attacks:  
  All hashing and signing operations are performed over canonical bytes
  only. Re-encoding between parse and verification is forbidden.

- Encoding variance attacks:  
  Alternate CBOR encodings (long-form integers, indefinite-length items,
  unsorted maps) MUST be rejected deterministically.

- Floating-point representation attacks:  
  Floating-point values are prohibited entirely, eliminating NaN,
  signed-zero, and subnormal representation variance.

- Unicode normalization ambiguity:  
  Canonical encodings operate on byte sequences. No implicit Unicode
  normalization is permitted at the protocol layer.

As a result, fuzzing inputs collapse to one of two outcomes:
- a single canonical byte representation, or
- deterministic rejection.

No degraded, heuristic, or best-effort parsing is permitted for
PQSF-governed artefacts.

---

## 31. Conformance Verification Procedures

Conformance to PQSF is demonstrated through:

- Canonical encoding determinism (identical inputs produce identical bytes).
- Correct hash and signature construction over canonical bytes only.
- Proper receipt and envelope validation per Annex W.
- Correct STP session semantics including state machine transitions, MAC verification, key destruction, and exporter binding.
- CryptoSuiteProfile pinning and validation.
- Schema version governance per Section 32A.

These requirements are normative and defined in the body of this specification.

Conformance is behavioural and verifiable, not declarative. No additional checklist is required. Implementations SHOULD use deterministic test vectors and cross-implementation encoding equivalence tests to validate conformance.  

### 31.1 Canonical Encoding Verification (Normative)

To ensure interoperability and prevent divergence due to encoding non-determinism, conformant implementations MUST successfully process the PQSF Canonical CBOR Test Vector Suite.

1. **Validation Requirement:** Implementations MUST reject any CBOR-encoded artefact that deviates from the strict profile defined in §7.2, including non-minimal integer encodings, indefinite-length items, duplicate keys, or non-canonical map key ordering.

2. **Hashing Invariant:** Implementations MUST demonstrate that identical logical data structures, when passed through the local deterministic encoding wrapper, produce byte-identical canonical encodings and identical derived hashes (including `intent_hash` and `decision_id`) matching the reference vectors.

3. **Canonical Byte Discipline:** Hashes MUST be computed over the canonical byte sequence, not over decoded structures or intermediate representations.

4. **Regression Assurance:** High-assurance implementations SHOULD incorporate the Canonical CBOR Test Vector Suite into automated regression testing to detect encoder drift introduced by library updates or platform differences.

Failure to satisfy these requirements renders the implementation non-conformant.

---

## 32. Annex Additivity Invariant (Normative)

Annexes in the PQ ecosystem are **additive extensions**.

An annex MAY introduce:
- new artefact types,
- new OPTIONAL fields on existing artefacts (explicitly marked OPTIONAL),
- new predicate classes evaluated by PQSEC,
- new error codes (registered where applicable).

An annex MUST NOT:
- reinterpret the semantics of any existing base field,
- change base hash or signature inputs for an existing artefact version,
- change base predicate evaluation semantics,
- modify refusal meanings or recovery semantics defined in PQSEC.

If an extension requires any non-additive change, it MUST:
- increment the affected artefact schema version(s), and
- specify the new behaviour as base (not as an annex overlay).

### 32.1 Parsing vs Semantic Handling

PQSF canonical encoding rules (including strict map key handling) remain authoritative.

If an implementation encounters an unknown field that violates PQSF structural or canonical rules, it MUST reject the artefact at the parsing layer.

Where an annex introduces OPTIONAL fields that are structurally valid under PQSF but semantically unsupported by an implementation, the implementation MAY ignore those fields at the semantic evaluation layer, provided that:

- ignoring the field does not reinterpret base semantics, and
- no base predicate depends on the presence or interpretation of that field.

Implementations MUST NOT relax PQSF canonical encoding requirements in order to accommodate annex extensions.

---

## 32A. Schema Version Governance (Normative)

### 32A.1 Purpose

This section defines deterministic schema version handling for all PQSF-governed artefacts. It prevents malicious downgrade via schema version ambiguity and ensures that version progression within a session is monotonic.

### 32A.2 Version Field Requirements

Every PQSF-governed artefact that includes a version field (`v: uint`) MUST comply with the following rules:

1. The `v` field is a non-negative integer representing the schema version of the artefact.
2. Schema version semantics are defined by the specification that owns the artefact type.
3. Implementations MUST validate `v` against deployment-configured supported version bounds before processing any other field.

### 32A.3 Additive vs Non-Additive Changes

**Additive changes** (adding optional fields, extending enums with new values) MUST NOT increment `v`. Implementations that encounter unknown optional fields on an artefact with a supported `v` value MAY ignore those fields at the semantic layer per Section 32.1.

**Non-additive changes** (reinterpreting field semantics, removing fields, changing required field types, altering signature input) MUST increment `v`. Implementations that encounter an artefact with an unsupported `v` value MUST reject it entirely.

### 32A.4 Deployment Version Bounds

Deployments MUST configure supported version bounds per artefact type:

```
VersionBounds = {
  artefact_type:  tstr,
  min_version:    uint,
  max_version:    uint
}
```

1. `min_version` is the oldest schema version accepted. Artefacts with `v < min_version` MUST be rejected with `E_SCHEMA_VERSION_UNSUPPORTED`.
2. `max_version` is the newest schema version accepted. Artefacts with `v > max_version` MUST be rejected with `E_SCHEMA_VERSION_UNSUPPORTED`.
3. `min_version` MUST be less than or equal to `max_version`.

### 32A.5 Session Floor Ratcheting

Within a single session (as defined by PQSF session primitives):

1. For each artefact type, the highest `v` value successfully processed establishes the session floor for that artefact type.
2. Subsequent artefacts of the same type with `v` below the session floor MUST be rejected with `E_SCHEMA_DOWNGRADE_ATTEMPT`.
3. The session floor is monotonically increasing and MUST NOT be reset within a session.
4. The session floor resets when a new session is established.

### 32A.6 Mixed-Version Refusal

PQSF does not define version negotiation. If a peer presents artefacts at a version outside the deployment's supported bounds, the artefact MUST be refused. There is no fallback, negotiation, or best-effort processing.

### 32A.7 Authority Boundary

Schema version governance is structural validation only. It does not grant authority, modify enforcement semantics, or create new enforcement predicates. All enforcement decisions remain exclusively within PQSEC.

Refusal codes: `E_SCHEMA_VERSION_UNSUPPORTED`, `E_SCHEMA_DOWNGRADE_ATTEMPT` (registered in PQSEC Annex AE.47)

---

## 32B. Evidence Classification Vocabulary (Normative)

### 32B.1 Purpose

This section defines a provenance taxonomy for evidence artefacts used across the PQ ecosystem. It enables consuming specifications to reason about evidence quality and provenance without granting any authority based on classification alone.

### 32B.2 Evidence Classes

The following evidence classes are defined:

| Class | Meaning |
|-------|---------|
| `hardware_attested` | Evidence produced by a hardware security element with tamper-resistant key storage |
| `secure_boot_bound` | Evidence bound to a verified secure boot chain |
| `software_measured` | Evidence produced by software measurement (e.g. runtime attestation, binary hash) |
| `model_inferred` | Evidence produced by a machine learning model or statistical inference system |
| `user_asserted` | Evidence asserted directly by a human user without independent verification |
| `external_asserted` | Evidence asserted by an external system or third party |
| `platform_bridged` | Evidence translated from platform-native attestation APIs via a governed adapter (PQAA) |

Classification is descriptive only and MUST NOT grant authority. A `hardware_attested` classification does not make evidence more authoritative than `software_measured` for enforcement purposes. Policy determines which evidence classes are required or acceptable for each operation class.

### 32B.3 EvidenceDescriptor

```
EvidenceDescriptor = {
  v:                      uint,           ; MUST be 1
  evidence_type:          tstr,           ; evidence type identifier (specification-defined)
  evidence_class:         tstr / null,    ; one of the classes defined in 32B.2, or null
  producer_id:            tstr / null,    ; evidence producer identifier
  producer_build_hash:    bstr(32) / null, ; producer build hash per 22A governance
  issued_tick:            uint,
  expiry_tick:            uint,
  suite_profile:          tstr
}
```

1. `evidence_type` identifies the specific evidence artefact type. The vocabulary is defined by the producing specification (e.g. PQAI, Neural Lock, PQEA).
2. `evidence_class` is one of the values defined in Section 32B.2 or null if classification is unknown or not applicable.
3. `producer_id` identifies the evidence producer. When null, the producer is anonymous. See PQSEC 22B for independence evaluation rules when `producer_id` is null.
4. `producer_build_hash` references the producer's build hash per PQSF Annex AC / PQSEC 22A governance. When null, build hash is unavailable.
5. All tick fields are Epoch Clock ticks.
6. EvidenceDescriptor does not carry a signature. It is embedded within or referenced by signed artefacts.

### 32B.4 Requirement Level

EvidenceDescriptor is SHOULD by default. Producing specifications SHOULD include EvidenceDescriptor with evidence artefacts. Policy MAY escalate this to MUST for specific operation classes.

### 32B.5 Predicate Mapping When Required by Policy

When active policy requires EvidenceDescriptor and it is absent:

1. EvidenceDescriptor absence evaluates as UNAVAILABLE at the predicate level when the descriptor is required by policy and the evidence artefact is otherwise present but lacks the required descriptor field or structure.
2. If the evidence artefact itself is absent, the predicate evaluates UNAVAILABLE under general evidence absence rules and MUST NOT carry error_code `E_EVIDENCE_DESCRIPTOR_REQUIRED`.
3. For Authoritative operations, any required predicate result of UNAVAILABLE results in PQSEC producing DENY.
4. When the specific cause of a required predicate UNAVAILABLE is "policy requires EvidenceDescriptor and it is absent", the EnforcementOutcome MUST carry error_code `E_EVIDENCE_DESCRIPTOR_REQUIRED`.
5. PQSF defines the vocabulary and descriptor structure. PQSEC defines enforcement mapping and selects error_code attribution.

### 32B.6 Authority Boundary

Evidence classification is descriptive only. It does not grant authority, modify enforcement semantics, or create new enforcement predicates. Classification does not affect evidence validity. All enforcement decisions remain exclusively within PQSEC.

Refusal code: `E_EVIDENCE_DESCRIPTOR_REQUIRED` (registered in PQSEC Annex AE.48)

---

## 33. Deferred Interoperability Negotiation (Informative)

This ecosystem intentionally does not define capability negotiation or profile registries at this stage.

If and when multiple independent implementations require formalised interoperability governance, it will be specified in a dedicated specification rather than retrofitted into existing core documents.

---

## 34. Annexes

### Annex A -- Canonical CBOR Profile Summary

The PQSF strict CBOR profile enforces:

* Deterministic encoding per RFC 8949 §4.2
* Shortest integer form
* No floating point values
* No indefinite-length items
* Unique map keys
* Lexicographic key ordering
* No CBOR tags unless explicitly profiled by consuming specifications

---

### Annex B -- Example CryptoSuiteProfile

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

### Annex C -- Reference Pseudocode (PQSF-Native)

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

### Annex D -- Reference Pseudocode (Epoch Clock Binding)

```python
def epochclock_hash_tick_bytes(tick_json_jcs_bytes, hash_profile):
    """Hash Epoch Clock tick bytes (already in JCS JSON form)."""
    return hash_with_profile(hash_profile, tick_json_jcs_bytes)

def bind_time_by_tick_hash(pqsf_obj, tick_hash):
    """Bind a PQSF object to time via Epoch Clock tick hash."""
    pqsf_obj["epoch_clock_hash"] = tick_hash
    return pqsf_obj
```

---

### Annex E -- Replay Safety Checklist

For replay-safe artefact design:

* ☐ intent_hash bound
* ☐ session_id bound
* ☐ exporter_hash bound (for Authoritative)
* ☐ issued_tick and expiry_tick present
* ☐ single-use enforced by consumer where required
* ☐ decision_id or operation_id unique per attempt
* ☐ nonce present for challenge-response patterns

---

### Annex F -- Privacy Policy Namespace

Privacy policies MAY be represented as PolicyObject instances using the `privacy:` namespace.

Recommended policy_id values:

* `privacy:retention:v1` - Data retention policies
* `privacy:logging:v1` - Logging and audit policies
* `privacy:disclosure:v1` - Data disclosure policies
* `privacy:telemetry:v1` - Telemetry and metrics policies
* `privacy:bundle:v1` - Bundled privacy policy suite

Recommended artefact class identifiers within privacy policy bodies:

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

### Annex G -- STP Session Lifecycle Examples (Expanded)

#### G.1 Happy Path -- New Session with DATA and CLOSE

```
Client                                    Server
  |                                         |
  |--- STP-INIT --------------------------> |
  |    {version: "1.0",                     |
  |     session_id: "abc",                  |
  |     client_nonce: ...,                  |
  |     client_caps: [...]}                 |
  |                                         |
  | <-- STP-ACCEPT -------------------------|
  |     {version: "1.0",                    |
  |      session_id: "abc",                 |
  |      server_nonce: ...,                 |
  |      server_caps: [...],                |
  |      exporter_hash: ...}                |
  |                                         |
  |    [State: ESTABLISHED]                 |
  |                                         |
  |--- STP-DATA (seq=0) -----------------> |
  |    {payload_type: "ConsentProof",       |
  |     body_hash: ..., mac: ...}           |
  |                                         |
  | <-- STP-DATA (seq=0) ------------------|
  |     {payload_type: "ReceiptEnvelope",   |
  |      body_hash: ..., mac: ...}          |
  |                                         |
  |--- STP-DATA (seq=1) -----------------> |
  |--- STP-DATA (seq=2) -----------------> |
  |                                         |
  |--- STP-CLOSE -------------------------> |
  |    {reason: "NORMAL",                   |
  |     final_sequence: 2,                  |
  |     close_hash: ..., mac: ...}          |
  |                                         |
  | <-- STP-CLOSE -------------------------|
  |    {reason: "NORMAL",                   |
  |     final_sequence: 0,                  |
  |     close_hash: ..., mac: ...}          |
  |                                         |
  |    [State: CLOSED]                      |
  |    [Key material destroyed]             |
```

#### G.2 Session Resumption

```
Client                                    Server
  |                                         |
  |--- STP-RESUME ------------------------> |
  |    {previous_session_id: "abc",         |
  |     resume_token: ...,                  |
  |     client_nonce: ...}                  |
  |                                         |
  | <-- STP-ACCEPT -------------------------| (new session_id)
  |     {session_id: "def",                 |
  |      exporter_hash: ...}                |
  |                                         |
  |    [State: ESTABLISHED]                 |
```

#### G.3 Error Recovery

```
Client                                    Server
  |                                         |
  |--- STP-INIT --------------------------> |
  |                                         |
  | <-- STP-ERROR -------------------------|
  |     {code: "E_CAPABILITY_MISMATCH"}     |
  |                                         |
  |    [State: ERROR]                       |
  |    [Key material destroyed]             |
  |    [Return to IDLE]                     |
  |    [New session_id required]            |
```

#### G.4 Idle Timeout

```
Client                                    Server
  |                                         |
  |    [ESTABLISHED, no activity]           |
  |    [3600s elapsed / tick timeout]       |
  |                                         |
  |    [Implicit CLOSE: reason=TIMEOUT]     |
  |    [State: CLOSED]                      |
  |    [Key material destroyed]             |
```

#### G.5 Authoritative Session with STP-Layer KEM Fallback

When TLS does not provide PQ key exchange and Authoritative operations are required:

```
Client                                    Server
  |                                         |
  |--- STP-INIT --------------------------> |
  |    {version: "1.0",                     |
  |     session_id: "xyz",                  |
  |     client_nonce: ...,                  |
  |     client_caps: [...],                 |
  |     kem_ciphertext: ...}                |  <-- ML-KEM-1024 encapsulation
  |                                         |
  | <-- STP-ACCEPT -------------------------|
  |     {version: "1.0",                    |
  |      session_id: "xyz",                 |
  |      server_nonce: ...,                 |
  |      server_caps: [...],                |
  |      exporter_hash: ...,                |
  |      kem_confirm: ...}                  |  <-- SHAKE256-256(shared_secret)
  |                                         |
  |    [Client verifies kem_confirm]        |
  |    [PRK = HKDF-Extract(E, SS)]          |
  |    [session_key, integrity_key derived] |
  |    [State: ESTABLISHED]                 |
  |    [Authoritative operations permitted] |
  |                                         |
  |--- STP-DATA (seq=0, Authoritative) ---> |
  |    {mac computed with integrity_key}    |
```

---

### Annex H -- KeyMail Overview (Informative)

This annex contains explanatory summaries only. If any divergence exists between this annex and Annex V, Annex V prevails.

This annex is explanatory only. The normative KeyMail specification, including all canonical structures, cryptographic requirements, and validation rules, is defined exclusively in Annex V.

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

### H.2 KeyMailEnvelope (Informative Summary Only)

The authoritative KeyMailEnvelope schema is defined in Annex V.

The structure previously shown here was intentionally simplified for explanatory purposes and MUST NOT be used as a normative or implementation schema.

Implementations MUST use the full canonical schema defined in Annex V.

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

### Annex I -- EBT Encryption Flow Example

```python
def ebt_encrypt(plaintext: bytes, label: str, root_seed: bytes,
                session_id: bytes | None, suite_profile: str,
                tick: int | None):
    """
    Encrypt plaintext using EBT pattern.
    AAD fields match §21.5 EBT_AAD exactly.
    """
    # Derive EBT key
    context_params = canonical_cbor_encode({
        "label": label,
        "session_id": session_id,
        "suite_profile": suite_profile,
        "tick": tick
    })
    ebt_key = cSHAKE256(
        X=root_seed + context_params,
        L=256,
        N="",
        S=f"PQSF-EBT:{label}"
    )
    
    # Generate nonce
    nonce = os.urandom(12)  # 96 bits for AES-GCM
    
    # Prepare AAD per §21.5 EBT_AAD (deterministic CBOR, keys in
    # lexicographic byte order, null-valued optional fields included)
    aad = canonical_cbor_encode({
        "label": label,
        "session_id": session_id,      # bstr(16) or CBOR null
        "suite_profile": suite_profile,
        "tick": tick                    # uint or CBOR null
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
        "tick": tick,
        "session_id": session_id,
        "exporter_hash": None,
        "suite_profile": suite_profile,
        "nonce": nonce,
        "aad": aad,
        "ciphertext": ciphertext,
        "tag": tag
    }
```

---

### Annex J -- Exporter Hash Derivation Example

```python
def derive_exporter_hash(tls_session, session_id: bytes, role_id: str, operation_class: str):
    """
    Derive exporter hash from TLS session.
    session_id is bstr(16) -- 16 bytes from CSPRNG.
    """
    context = canonical_cbor_encode({
        "exporter_binding_id": "pqsf.exporter_context.v1",
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

### Annex K -- Merkle Tree Construction Example

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
        for i in range(0, len(current_level) - 1, 2):
            left = current_level[i]
            right = current_level[i + 1]
            next_level.append(node_hash(left, right))
        
        # Odd node: promote without hashing or duplication (§19.3)
        if len(current_level) % 2 == 1:
            next_level.append(current_level[-1])
        
        current_level = next_level
    
    return current_level[0]  # Root hash
```

---

### Annex L -- PolicyBundle Validation Example

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

### Annex M -- Profile Update Acceptance Logic

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

### Annex N -- Emergency Transition State Machine

```
Normal Operation
       |
       | (algorithm compromise detected)
       v
Emergency Profile Issued
       |
       | (governance signatures verified)
       v
Phase 1: Dual-Algorithm Mode (tick-based; see §8.5.2)
       | - Accept both old and new algorithms
       | - Log all uses of old algorithm
       | - Warn users
       |
       | (Phase 1 tick boundary reached)
       v
Phase 2: Authoritative Restriction (tick-based; see §8.5.2)
       | - Reject old algorithm for Authoritative ops
       | - Accept for Non-Authoritative ops only
       | - Escalate warnings
       |
       | (Phase 2 tick boundary reached)
       v
Phase 3: Complete Deprecation
       | - Reject old algorithm for all operations
       | - Remove from codebase
       |
       v
New Normal Operation
```

---

### Annex O -- STP Capability Negotiation Example

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

### Annex P -- ConsentProof Construction Example

```python
def create_consent_proof(
    action: str,
    intent: dict,
    session_id: bytes,
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

### Annex Q -- LedgerEntry Chain Validation

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

### Annex R -- RecoveryCapsule Activation Logic

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

### Annex S -- Test Vector Checklist

For PQSF conformance testing, implementations MUST provide test vectors demonstrating:

Canonical Encoding:
* ☐ CBOR encoding round-trip (encode → decode → encode produces identical bytes)
* ☐ Rejection of non-canonical CBOR (wrong integer encoding, unsorted maps, etc.)
* ☐ JCS JSON handling for Epoch Clock artefacts

Cryptographic Operations:
* ☐ Hash reproducibility (same input → same hash across implementations)
* ☐ Signature verification (valid signatures accepted, invalid rejected)
* ☐ Exporter hash derivation (consistent across sessions)

Profile Management:
* ☐ Profile update acceptance (valid updates accepted)
* ☐ Downgrade rejection (security downgrades rejected)
* ☐ Revocation enforcement (revoked profiles rejected)
* ☐ Emergency transition (phased deprecation works correctly)

Object Validation:
* ☐ PolicyBundle validation (structure, hash, signatures)
* ☐ ConsentProof validation (structure, binding, freshness)
* ☐ LedgerEntry chain validation (continuity, monotonicity)

Merkle Tree Construction:
* ☐ Root hash reproducibility (same entries → same root)
* ☐ Odd-node handling (consistent across implementations)

EBT Encryption:
* ☐ Key derivation determinism (same inputs → same key)
* ☐ Nonce uniqueness enforcement
* ☐ AEAD correctness (encrypt/decrypt round-trip)

STP Protocol:
* ☐ Message structure validation
* ☐ Capability negotiation correctness
* ☐ Session resumption handling

---

### Annex T -- Performance Optimization Guidelines

While PQSF does not define performance requirements, the following optimizations are recommended:

Caching Strategies:
1. Profile Cache: Cache validated CryptoSuiteProfiles by profile_id
2. Hash Cache: Cache frequently computed hashes (e.g., policy_hash)
3. Signature Verification: Cache verified signatures within freshness window

Parallelization Opportunities:
1. Independent Signature Verification: Verify multiple signatures in parallel
2. Merkle Tree Construction: Build subtrees in parallel
3. Profile Updates: Check multiple governance signatures concurrently

Lazy Evaluation:
1. Profile Discovery: Only fetch profiles when needed
2. Revocation Checks: Cache revocation list, update periodically
3. Capability Negotiation: Defer optional feature setup until used

Memory Management:
1. Canonical Encoding: Reuse encoding buffers where safe
2. Merkle Trees: Stream large ledgers rather than loading entirely
3. EBT Envelopes: Encrypt/decrypt in streaming mode for large payloads

---

### Annex U -- Security Considerations Summary

Critical Security Properties:

1. Canonical Encoding is Non-Negotiable
   - Non-canonical encodings create hash ambiguity
   - Implementations MUST reject any non-canonical input

2. Profile Downgrade Prevention
   - Never accept weaker cryptography
   - Emergency transitions are the ONLY exception

3. Exporter Binding for Authoritative Operations
   - Session binding prevents cross-session replay
   - Null exporter_hash for Authoritative ops is a vulnerability

4. Signature Verification Order
   - Verify structure BEFORE cryptographic operations
   - Avoids expensive operations on malformed inputs

5. Time Binding via Epoch Clock
   - Never trust system clocks
   - Always consume externally signed ticks

6. Ledger Continuity
   - prev_hash chain prevents insertion/deletion
   - Monotonic ticks prevent reordering

Common Implementation Mistakes:

* Re-encoding Epoch Clock artefacts (breaks signature verification)
* Accepting unsigned or self-signed profiles
* Allowing null exporter_hash for Authoritative operations
* Trusting system time instead of Epoch Clock
* Not checking revocation before profile use
* Reusing nonces in EBT encryption

---

### Annex V -- KeyMail: Secure Messaging Artefacts (Normative)

This annex defines secure, non-authoritative messaging artefacts for the PQ ecosystem.

---

#### V.1 Scope and Authority Boundary

KeyMail defines canonically encoded, encrypted message artefacts for secure communication between PQ-aware endpoints.

KeyMail provides:
- End-to-end encrypted message envelopes
- Sender and recipient identity binding
- Time-bounded message validity
- Optional session binding for conversational contexts
- Thread continuity for multi-message exchanges

Authority Boundary:

KeyMail grants no authority.

- Receiving a KeyMailEnvelope does NOT imply consent
- Receiving a KeyMailEnvelope does NOT trigger execution
- Receiving a KeyMailEnvelope does NOT propagate custody authority
- KeyMail artefacts MUST NOT be interpreted as ConsentProof, delegation, or permission

KeyMail is a transport and storage format only. All enforcement, admission, and authority decisions are external to KeyMail and defined by PQSEC.

---

#### V.2 KeyMailEnvelope Structure

KeyMailEnvelope = {
  message_id: tstr,
  thread_id: tstr / null,
  sender_identity_ref: tstr,
  recipient_identity_ref: tstr,
  issued_tick: uint,
  expiry_tick: uint,
  session_id: bstr(16) / null,
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

#### V.3 Field Semantics

message_id
Unique identifier for this message. MUST be unique across all messages from this sender.

thread_id
Optional identifier for message threading. If present, MUST be consistent across all messages in a thread. Absence indicates a standalone message.

sender_identity_ref
Reference to the sender's identity artefact (ModelIdentity for AI, custody identity for human operators, or service identity). Format is deployment-defined but MUST be resolvable to a verification key.

recipient_identity_ref
Reference to the recipient's identity artefact. Used for key agreement and routing. Format is deployment-defined.

issued_tick
Epoch Clock tick at which the message was created.

expiry_tick
Epoch Clock tick after which the message SHOULD be considered stale. Consuming systems MAY refuse messages past expiry. Enforcement is external.

session_id
Optional session identifier for binding messages to a specific conversational session.

exporter_hash
Optional TLS exporter hash for session binding. When present, MUST match the active session's exporter hash for the message to be accepted in session-bound contexts.

content_type
MIME type or PQ-defined type identifier for the decrypted payload. Common values:
- `text/plain` -- Plain text message
- `application/cbor` -- Structured CBOR payload
- `application/json` -- JSON payload
- `pqsf:artefact` -- Embedded PQSF artefact (ConsentProof, PolicyBundle, etc.)

encapsulated_key
ML-KEM encapsulated key ciphertext for the recipient. MUST be present when using ephemeral key agreement. MAY be null when using pre-shared keys or when the encapsulated key is transported separately. When present, MUST be the output of ML-KEM-1024.Encapsulate() or equivalent KEM operation.

ciphertext
Encrypted message payload. MUST be produced by an authenticated encryption scheme defined by suite_profile.

nonce
Cryptographic nonce used for encryption. MUST satisfy the nonce requirements of the AEAD scheme in suite_profile. MUST NOT be reused with the same key.

tag
Authentication tag from the AEAD scheme.

suite_profile
CryptoSuiteProfile reference defining:
- Key agreement scheme (e.g., ML-KEM-1024, X25519)
- AEAD scheme (e.g., AES-256-GCM, XChaCha20-Poly1305)
- Signature scheme for sender_signature

sender_signature
Signature over the canonical CBOR encoding of the envelope with sender_signature field omitted. Provides sender authentication and non-repudiation.

---

#### V.4 Canonical Encoding Requirements

1. KeyMailEnvelope MUST be encoded using PQSF Deterministic CBOR.
2. sender_signature MUST be computed over the canonical CBOR encoding with the sender_signature field omitted.
3. Re-encoding a decoded KeyMailEnvelope MUST produce byte-identical output.
4. Non-canonical encodings MUST be rejected.

---

#### V.5 Key Agreement and Encryption

##### V.5.1 Ephemeral Key Agreement

For post-quantum key agreement:

```
(shared_secret, encapsulated_key) = ML-KEM-1024.Encapsulate(recipient_public_key)
```

The encapsulated_key MUST be transmitted alongside the KeyMailEnvelope or embedded in deployment-specific key transport.

##### V.5.2 Message Key Derivation

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

##### V.5.3 Encryption

KeyMail_AAD (Normative):

```
KEYMAIL_AAD = {
  "content_type": tstr,
  "message_id": tstr,
  "recipient_identity_ref": tstr,
  "sender_identity_ref": tstr
}
```

AAD MUST be the canonical CBOR encoding of exactly this map with keys in lexicographic byte order. All four fields are REQUIRED. No null handling applies -- all fields MUST be present and non-null.

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

#### V.6 Validation Requirements

A KeyMailEnvelope is valid if and only if:

1. Structural validity: All required fields present and correctly typed
2. Canonical encoding: Envelope is valid Deterministic CBOR
3. Signature verification: sender_signature verifies under the sender's public key referenced by sender_identity_ref
4. Time validity: issued_tick ≤ current_tick < expiry_tick (when freshness is enforced)
5. Session binding: If session_id or exporter_hash is present, it matches the expected session context
6. Decryption success: ciphertext decrypts successfully with valid tag

Validation failure MUST NOT be silent. Implementations SHOULD report specific failure reasons for debugging.

---

#### V.7 Thread Continuity

For multi-message exchanges:

1. All messages in a thread MUST share the same thread_id.
2. thread_id SHOULD be generated by the thread initiator and reused by all participants.
3. Thread membership is determined by thread_id only; sender/recipient pairs MAY vary within a thread.
4. Thread ordering is determined by issued_tick. Ties are broken by message_id lexicographic order.

---

#### V.8 Error Codes

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

#### V.9 Security Considerations

##### V.9.1 Forward Secrecy

KeyMail with ephemeral ML-KEM key agreement provides forward secrecy: compromise of long-term keys does not compromise past messages encrypted with ephemeral shared secrets.

For deployments requiring forward secrecy, implementations MUST:
- Use ephemeral key encapsulation per message or per session
- Securely delete ephemeral key material after use

##### V.9.2 Replay Protection

KeyMail envelopes are self-describing and time-bounded, but replay protection is NOT inherent.

Consuming systems MUST:
- Track seen message_id values within a deployment scope
- Reject duplicate message_id submissions
- Consider expiry_tick when determining replay window

##### V.9.3 Metadata Protection

KeyMail encrypts message content but does NOT encrypt:
- sender_identity_ref
- recipient_identity_ref
- issued_tick / expiry_tick
- thread_id
- content_type

Deployments with metadata confidentiality requirements MUST use additional transport-layer encryption (e.g., TLS, onion routing) or wrap KeyMailEnvelope in an outer EBTEnvelope.

##### V.9.4 Non-Authority Reinforcement

Implementations MUST NOT:
- Interpret KeyMailEnvelope receipt as consent
- Trigger Authoritative operations based on message content without separate ConsentProof
- Propagate custody or signing authority through KeyMail
- Treat sender_signature as equivalent to ConsentProof signature

---

#### V.10 Reference Implementation

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
    session_id: Optional[bytes]
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
    session_id: Optional[bytes] = None,
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
    expected_session_id: Optional[bytes] = None,
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

#### V.11 Test Vector Requirements

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

#### V.12 Conformance

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

### Annex W -- ReceiptEnvelope (Normative)

#### W.1 Purpose

ReceiptEnvelope is the canonical container for all evidence, audit, and domain receipts across the ecosystem. All receipts defined by PQSEC, PQAI, PQHD, ZEB, SEAL, and BPC MUST use this container format.

#### W.2 Structure (Normative)

ReceiptEnvelope MUST be a deterministic CBOR map with the following fields:

Required fields:

| Field | Type | Description |
|-------|------|-------------|
| `type` | tstr | Namespaced receipt type identifier (e.g. `pqsec.decision_receipt`, `pqsf.message`, `pqai.tool_profile`) |
| `v` | uint | Receipt schema version, starting at 1 |
| `issued_tick` | uint | Epoch Clock tick at which this receipt was created |
| `epoch_clock_hash` | bstr (32 bytes) | SHAKE256-256 hash of the Epoch Clock artefact used for time binding |
| `subject` | map | Subject identifier (see W.2.1) |
| `suite_profile` | tstr | Cryptographic suite profile identifier (e.g. `pqsf.suite.mldsa65_shake256-256`) |
| `body` | map | Receipt-type-specific payload (deterministic CBOR) |
| `signature` | bstr | Signature over canonical receipt bytes with signature field omitted |

Optional fields:

| Field | Type | Description |
|-------|------|-------------|
| `policy_ref` | bstr (32 bytes) | Hash reference to applicable policy profile |
| `evidence_refs` | array of bstr | Array of hashes referencing other receipts or artefacts |
| `result` | tstr | Outcome indicator (receipt-type-specific) |
| `issuer` | bstr | Issuer principal identifier |
| `expires_at` | uint | Expiry tick (if applicable) |

##### W.2.1 Subject Structure

The `subject` field MUST be a deterministic CBOR map containing:

| Field | Type | Description |
|-------|------|-------------|
| `subject_type` | tstr | Subject type identifier (e.g. `principal`, `session`, `transaction`, `adapter`) |
| `subject_ref` | bstr | Subject reference (hash, identifier, or principal ID) |

#### W.3 Encoding Rules (Normative)

1. ReceiptEnvelope MUST be encoded as deterministic CBOR per PQSF canonical encoding rules.
2. Map keys MUST be sorted lexicographically by key bytes.
3. Integer values MUST use shortest encoding.
4. No duplicate keys are permitted.
5. No indefinite-length items are permitted.
6. Signature MUST be computed over the canonical CBOR bytes of the ReceiptEnvelope with the `signature` field omitted.
7. The signing algorithm MUST match the `suite_profile` declaration.

#### W.4 Authority Boundary (Normative)

1. ReceiptEnvelope is descriptive only. It records that something occurred or was evaluated.
2. ReceiptEnvelope MUST NOT grant authority. Possession of a receipt does not imply permission.
3. ReceiptEnvelope MUST NOT replace enforcement. Receipts are outputs of enforcement, not inputs that bypass it.
4. Receipt presence MUST NOT imply permission. A receipt proving evaluation occurred does not mean the action was allowed.

#### W.5 Validation Rules (Normative)

A ReceiptEnvelope MUST be considered invalid if any of the following hold:

1. Any required field is missing
2. `type` is not a recognised receipt type
3. `v` is not a supported schema version for the receipt type
4. `issued_tick` is in the future relative to current Epoch Clock state
5. `epoch_clock_hash` does not match a known Epoch Clock artefact
6. `suite_profile` is not a supported cryptographic suite
7. `signature` does not verify under the declared suite and issuer
8. `body` does not conform to the schema for the declared `type`
9. `expires_at` is present and is in the past at evaluation time

#### W.6 Receipt Type Namespacing (Normative)

Receipt types MUST be namespaced using dot notation:

| Prefix | Specification |
|--------|---------------|
| `pqsf.*` | PQSF-defined receipts (message, transcript, session, consent, adapter_profile) |
| `pqsec.*` | PQSEC-defined receipts (decision, predicate, audit) |
| `pqai.*` | PQAI-defined receipts (tool profile, drift evidence) |
| `pqhd.*` | PQHD-defined receipts (custody-specific) |
| `zeb.*` | ZEB-defined receipts (observation) |
| `seal.*` | SEAL-defined receipts (submission evidence) |
| `bpc.*` | BPC-defined receipts (pre-contract fulfilment) |
| `neural_lock.*` | Neural Lock-defined receipts (operator state attestation) |
| `pqgw.*` | PQ Gateway-defined receipts (inference, usage, enrollment, provider, policy, payment) |
| `pqaa.*` | PQAA-defined receipts (attestation_bundle) |

**Reserved namespaces (DRAFT):** Some specifications may declare receipt namespaces while in DRAFT status (e.g. `pqea.*`). Reserved namespaces MAY appear in artefacts, but MUST be treated as invalid unless explicitly permitted by policy.

Policy that permits reserved namespaces MUST:
1. name the namespace prefix explicitly (e.g. `pqea.*`)
2. pin the producing specification version and hash
3. define required validation rules for the receipt bodies

Unknown namespaces MUST be treated as invalid unless explicitly permitted by policy.

#### W.7 Canonical Hash Functions (Normative)

Unless explicitly overridden by a declared `suite_profile`, receipt-internal hashes MUST use the hash algorithm defined by the applicable suite_profile. Where no suite_profile override exists, canonical Tier-2 hashing (SHAKE256-256, per Section 9.1) applies.

A specification MAY define additional hash functions only if ALL of the following hold:

1. The hash function is declared by name as an ASCII `tstr` identifier.
2. The bytes that are hashed are defined canonically (deterministic CBOR or explicit byte string).
3. The hash function selection is bound to a `suite_profile` or explicitly declared in the receipt body where used.
4. Implementations MUST reject unknown `hash_alg` identifiers unless explicitly permitted by policy.

If a receipt body contains a `hash_alg` field, the receipt MUST be rejected if:
- `hash_alg` is unknown or unsupported, or
- the computed hash does not match the value carried.

Refusal code: `E_HASH_ALG_UNSUPPORTED`

#### W.8 Receipt Body Base Discipline (Normative)

ReceiptEnvelope already carries common, mandatory fields (`v`, `issued_tick`, `epoch_clock_hash`, `suite_profile`, `signature`).

To prevent schema drift and duplicated time-binding patterns across the ecosystem:

1. Receipt bodies MUST NOT duplicate ReceiptEnvelope fields:
   - `issued_tick`
   - `epoch_clock_hash`
   - `suite_profile`
   - `signature`
   - `v` (ReceiptEnvelope version)

   **Exception:** Manifest-style receipt bodies (such as GatewayManifestBody in Annex AI.2 and its product-layer specialisations) MAY include body-scoped `v` (body schema version) and body-scoped `issued_tick` when these carry semantically distinct meaning from the envelope fields. When present, body-scoped `v` indicates the body schema version (not the envelope version), and body-scoped `issued_tick` indicates a body-specific issuance time that may differ from the envelope timestamp. Manifest-style bodies that include `signature` MUST document whether this is the body's own signature or a reference to an inner signed artefact. Implementations MUST NOT assume body-scoped and envelope-scoped values are interchangeable without explicit specification guidance.

2. If a receipt body includes any field that is semantically equivalent to an envelope field (for backwards compatibility), the value MUST exactly match the envelope field. Any mismatch MUST invalidate the receipt.

3. Receipt bodies that require explicit expiry semantics MUST use `expires_at` in the ReceiptEnvelope. Receipt bodies MUST NOT invent alternative expiry fields unless the producing spec is Bitcoin-consensus constrained (e.g. block height), in which case the alternative field MUST be clearly labelled as consensus-derived (e.g. `expires_at_height`).

Refusal code: `E_RECEIPT_BODY_DUPLICATES_ENVELOPE`
Refusal code: `E_RECEIPT_FIELD_MISMATCH`

#### W.9 `pqsf.merkle_checkpoint` (Normative)

This receipt type defines a canonical, ecosystem-wide compaction boundary for append-only receipt stores.

A `pqsf.merkle_checkpoint` allows an implementation to prune or archive individual receipts while preserving verifiable integrity across the removed range.

**ReceiptEnvelope.type:** `"pqsf.merkle_checkpoint"`

**Subject:**
- `subject_type`: `"receipt_store"`
- `subject_ref`: `bstr` identifying the local store or deployment store ID (implementation-defined)

**Body:**

| Field | Type | Description |
|------|------|-------------|
| `checkpoint_v` | uint | Body schema version (start at 1) |
| `range_start_tick` | uint | Start tick (inclusive) for committed range |
| `range_end_tick` | uint | End tick (inclusive) for committed range |
| `range_count` | uint | Number of receipts committed by this checkpoint |
| `root_alg` | tstr | MUST be `"shake256-256_merkle_v1"` |
| `merkle_root` | bstr(32) | Merkle root over `receipt_hashes` |
| `prev_checkpoint_hash` | bstr(32) / null | Hash of prior checkpoint receipt, or null if first |
| `store_policy_hash` | bstr(32) | Hash of retention/compaction policy in effect |

**Merkle construction (Normative):**
1. Each committed receipt MUST be hashed as `receipt_hash = SHAKE256-256(canonical_receipt_envelope_bytes)`.
2. Leaves MUST be ordered lexicographically by `receipt_hash` bytes.
3. Internal nodes MUST be `SHAKE256-256(0x01 || left || right)`.
4. Leaves MUST be `SHAKE256-256(0x00 || receipt_hash)`.
5. The Merkle root MUST be `merkle_root`.

**Integrity rule:**
- A store that prunes receipts MUST retain all `pqsf.merkle_checkpoint` receipts covering the pruned ranges.

**Authority boundary:**
- A checkpoint receipt is integrity evidence only. It MUST NOT be treated as permission or as a replacement for missing receipt content.

Refusal code: `E_CHECKPOINT_INVALID`

---

### Annex X -- Message, Transcript, Session Scope, and Consent Primitives (Normative)

#### X.1 Receipt Types Defined in This Annex

This annex defines the following ReceiptEnvelope `type` values:

| Type | Purpose |
|------|---------|
| `pqsf.message` | Agent/human communication with authority classification |
| `pqsf.transcript_commitment` | Cryptographic commitment to conversation transcript |
| `pqsf.session_scope` | Session boundary and participant declaration |
| `pqsf.human_confirm` | Human confirmation consent evidence |
| `pqsf.human_approve` | Human approval consent evidence (stronger than confirm) |
| `pqsf.delegation` | Delegation evidence distinct from human consent |

#### X.2 `pqsf.message`

ReceiptEnvelope.type: `"pqsf.message"`
ReceiptEnvelope.body: `MessageBody`

##### X.2.1 MessageClass Values

MessageClass defines the authority level of a message:

| Value | Authority Level | Description |
|-------|-----------------|-------------|
| `COORDINATION` | None | Inter-agent coordination, never admissible for authority |
| `PROPOSAL` | Context only | May be recorded but MUST NOT unlock authority |
| `REQUEST_AUTHORITY` | Request | May be evaluated; allows if predicates pass |
| `EXECUTE` | Execution | Admissible only if schema-valid, session-bound, and transcript-bound |

##### X.2.2 MessageBody Structure

MessageBody (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Message schema version |
| `class` | tstr | Yes | MessageClass value |
| `sid` | bstr (16 bytes) | Yes | Session identifier |
| `ctr` | uint | Yes | Monotonic per-session counter |
| `schema` | tstr | Conditional | Canonical schema identifier (REQUIRED for REQUEST_AUTHORITY, EXECUTE) |
| `payload` | bstr | Conditional | Payload bytes |
| `payload_hash` | bstr (32 bytes) | Conditional | SHAKE256-256(payload) (REQUIRED for REQUEST_AUTHORITY, EXECUTE) |
| `ts` | uint | No | Sender timestamp (informative only) |
| `pid` | bstr (32 bytes) | Conditional | Participant principal identifier (REQUIRED for PROPOSAL, REQUEST_AUTHORITY, EXECUTE) |
| `refs` | array of bstr | No | Hashes of referenced receipts or artefacts |

##### X.2.3 Message Rules (Normative)

1. `ctr` MUST be present and MUST increase monotonically within a given `sid`. The first message in a session MUST have `ctr = 0`. Duplicate or non-monotonic `ctr` values MUST be treated as invalid.

1a. `ctr` persistence: For a given `sid`, the sender MUST persist `ctr` across process restarts for the lifetime of the session scope. `ctr` MUST NOT reset to 0 unless a new `pqsf.session_scope` is established with a new `sid`. Non-persistent or reset counters within an active session MUST be treated as invalid message sequencing. On state loss (crash, reinstall, or unrecoverable counter state), the sender MUST NOT resume the existing session. A new `pqsf.session_scope` MUST be established with a new `sid`.

Refusal code: `E_MESSAGE_COUNTER_INVALID`

2. If `class ∈ {REQUEST_AUTHORITY, EXECUTE}`:
   - `schema` MUST be present and MUST reference a registered schema identifier.
   - `payload` MUST be present.
   - `payload_hash` MUST be present and MUST equal `SHAKE256-256(payload)`.
   - `pid` MUST be present.

3. If `class = PROPOSAL`:
   - `pid` MUST be present.
   - `schema` MAY be present.
   - `payload_hash` SHOULD be present if `payload` is present.

4. If `class = COORDINATION`:
   - `schema` MUST be absent.
   - `payload_hash` MUST be absent.
   - `pid` MAY be absent.

5. If `class` is missing from a received message, consuming specifications MUST treat the message as `REQUEST_AUTHORITY` (fail-safe default).

6. ReceiptEnvelope signatures, if present, MUST cover the ReceiptEnvelope bytes exactly as defined by Annex W.

#### X.3 CanonicalTranscript and `pqsf.transcript_commitment`

##### X.3.1 CanonicalTranscript Construction (Normative)

CanonicalTranscript is a deterministic byte sequence constructed from a set of `pqsf.message` receipts within a session.

Inclusion criteria:
- Include only `pqsf.message` receipts where `class ∈ {PROPOSAL, REQUEST_AUTHORITY, EXECUTE}`.
- Do NOT include `COORDINATION` messages.

Ordering:
- Sort included receipts by `(sid, ctr)` ascending.
- All receipts MUST share the same `sid`.
- If any included receipt is missing `ctr`, has duplicated `ctr`, or violates monotonicity for its `sid`, transcript construction MUST fail.

Encoding:
- CanonicalTranscript bytes MUST be the deterministic CBOR encoding of an array containing the full ReceiptEnvelope bytes for each included message, in sorted order.

```
CanonicalTranscriptBytes = CBOR_ENCODE([
  ReceiptEnvelope_0,
  ReceiptEnvelope_1,
  ...
  ReceiptEnvelope_N
])
```

##### X.3.2 `pqsf.transcript_commitment`

ReceiptEnvelope.type: `"pqsf.transcript_commitment"`
ReceiptEnvelope.body: `TranscriptCommitmentBody`

TranscriptCommitmentBody (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `sid` | bstr (16 bytes) | Yes | Session identifier |
| `hash_alg` | tstr | Yes | Hash algorithm identifier (e.g. `shake256-256`) |
| `t_hash` | bstr (32 bytes) | Yes | SHAKE256-256(CanonicalTranscriptBytes) |
| `scope` | tstr | Yes | `"SESSION"` or `"ACTION"` |
| `action_id` | bstr (16 bytes) | Conditional | REQUIRED if `scope="ACTION"`, MUST be absent otherwise |
| `expires_at` | uint | Yes | Expiry tick |
| `nonce` | bstr (32 bytes) | No | Optional nonce for replay protection |

##### X.3.3 Transcript Commitment Rules (Normative)

1. `t_hash` MUST be computed over CanonicalTranscript bytes as defined in X.3.1.

2. If `scope="ACTION"`, the CanonicalTranscript MUST include at least one `pqsf.message` receipt with `class="EXECUTE"` that binds `action_id` either:
   - by including `action_id` in `refs`, or
   - inside the schema-defined payload referenced by `schema`.

3. Transcript commitments MUST NOT include transcript plaintext. Only the hash is stored.

4. A transcript commitment MUST be rejected if `expires_at` is in the past at evaluation time.

5. Default freshness window: `expires_at = issued_tick + 300` (5 minutes). Policy MAY specify different windows.

6. For irreversible actions, `scope` MUST be `"ACTION"`. `scope="SESSION"` MUST NOT be used for irreversible actions.

#### X.4 `pqsf.session_scope`

ReceiptEnvelope.type: `"pqsf.session_scope"`
ReceiptEnvelope.body: `SessionScopeBody`

##### X.4.0 SessionScopeBody Structure

SessionScopeBody (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `sid` | bstr (16 bytes) | Yes | Session identifier |
| `mode` | tstr | Yes | `"SINGLE_AGENT"` or `"MULTI_AGENT"` |
| `supervision` | tstr | Yes | `"NONE"`, `"HUMAN_CONFIRM"`, or `"HUMAN_APPROVE"` |
| `participants` | array of Participant | Yes | List of session participants |
| `exporter_hash` | bstr (32 bytes) | Yes | Hash of secure transport exporter binding |
| `role_policy` | map | No | Role-specific policy overrides |
| `expires_at` | uint | Yes | Session expiry tick |

Participant (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pid` | bstr (32 bytes) | Yes | Participant principal identifier |
| `role` | tstr | Yes | `"AGENT"`, `"HUMAN"`, `"SUPERVISOR"`, or `"OBSERVER"` |
| `capabilities_ref` | bstr (32 bytes) | Conditional | Hash reference to `pqai.tool_profile` (REQUIRED for AGENT in MULTI_AGENT mode) |

##### X.4.1 Session Scope Minting Authority (Normative)

`pqsf.session_scope` receipts MUST be minted by the wallet authority process only.

Supervisor identities MAY provide consent evidence, but MUST NOT mint session scopes.

Session scope acceptance MUST verify:
- ReceiptEnvelope signature verifies under wallet authority key
- `exporter_hash` matches the evaluator's current transport exporter binding
- `sid` matches the evaluator's expected session identifier

Refusal code: `E_SESSION_SCOPE_ISSUER_INVALID`

##### X.4.2 Session Scope Expiry (Normative)

`expires_at` MUST be present for all session scopes.

Default TTL values:

| Mode | RECOMMENDED Default TTL | Maximum TTL |
|------|------------------------|-------------|
| `SINGLE_AGENT` | Equivalent of 30 minutes as determined by the Epoch Clock profile tick interval | Policy-defined |
| `MULTI_AGENT` | Equivalent of 10 minutes as determined by the Epoch Clock profile tick interval | Policy-defined |

TTL MUST be expressed in Epoch Clock ticks. The wall-clock equivalents above are informative guidance only. Implementations MUST convert to ticks using the active Epoch Clock profile's tick interval at session establishment time.

Expired session scopes MUST be rejected immediately.

Refusal code: `E_SESSION_SCOPE_EXPIRED`

##### X.4.3 Role Policy Defaults (Normative)

| Role | May emit REQUEST_AUTHORITY | May emit EXECUTE | May emit COORDINATION |
|------|---------------------------|------------------|----------------------|
| `AGENT` | Yes | Yes | Yes |
| `HUMAN` | Yes | Yes | Yes |
| `SUPERVISOR` | Yes | Yes | Yes |
| `OBSERVER` | No | No | Yes |

Additional constraints:
- If `supervision != NONE`, at least one participant with `role="SUPERVISOR"` MUST appear in `participants`.
- `OBSERVER` MUST NOT emit any `pqsf.message` with `class ∈ {PROPOSAL, REQUEST_AUTHORITY, EXECUTE}`.

Refusal code: `E_ROLE_POLICY_VIOLATION`

##### X.4.4 `capabilities_ref` Requirement (Normative)

In `MULTI_AGENT` mode:
- All participants with `role="AGENT"` MUST include `capabilities_ref` referencing a valid `pqai.tool_profile` receipt.
- The referenced tool profile MUST be valid and not expired at session scope evaluation time.

Refusal code: `E_AGENT_CAPABILITIES_REQUIRED`

##### X.4.5 Session Binding Anti-Fixation (Normative)

A `pqsf.session_scope` receipt MUST NOT be accepted if any of the following hold:

1. The `sid` does not match the evaluator's active session identifier.
2. The `exporter_hash` does not match the evaluator's current transport exporter binding.
3. The evaluator's `pid` is not listed in `participants` with a role permitted to participate.

This prevents session fixation attacks where an adversary:
- Establishes a session and transfers it to a victim
- Injects a stale or foreign session token into an active context

Detection of session fixation MUST fail closed.

Refusal code: `E_SESSION_FIXATION_DETECTED`

##### X.4.6 Session Scope General Rules (Normative)

1. If policy requires explicit session scope, any `pqsf.message` with `class ∈ {REQUEST_AUTHORITY, EXECUTE}` MUST be evaluated under a valid `pqsf.session_scope` for the same `sid`.

2. If `mode="SINGLE_AGENT"`:
   - Only one `pid` with `role="AGENT"` MAY be present in `participants`.
   - No other agent principal MAY introduce authority-bearing messages for that `sid`.

3. If `mode="MULTI_AGENT"`:
   - All authority-bearing `pqsf.message` receipts MUST include `pid`.
   - That `pid` MUST appear in `participants` with a role permitted to issue authority-bearing messages.

4. If `expires_at` is present and in the past at evaluation time, the session scope MUST be treated as invalid.

#### X.5 Consent Receipts: `pqsf.human_confirm` and `pqsf.human_approve`

These receipts represent human supervision evidence. They are evidence-only and MUST be evaluated by PQSEC under current conditions.

##### X.5.1 `pqsf.human_confirm`

ReceiptEnvelope.type: `"pqsf.human_confirm"`
ReceiptEnvelope.body: `ConsentBody`

`human_confirm` represents acknowledgement that the human has seen and confirmed an action. It is the weaker of the two consent levels.

##### X.5.2 `pqsf.human_approve`

ReceiptEnvelope.type: `"pqsf.human_approve"`
ReceiptEnvelope.body: `ConsentBody`

`human_approve` represents explicit approval with full understanding of consequences. It is required for high-risk or irreversible actions.

##### X.5.3 ConsentBody Structure

ConsentBody (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `sid` | bstr (16 bytes) | Yes | Session identifier |
| `action_id` | bstr (16 bytes) | Yes | Unique action identifier |
| `op_class` | tstr | Yes | Operation class identifier (from canonical registry) |
| `expires_at` | uint | Yes | Consent expiry tick |
| `nonce` | bstr (32 bytes) | Conditional | Nonce value (if policy requires nonce binding) |
| `nonce_hash` | bstr (32 bytes) | Conditional | SHAKE256-256(nonce) -- REQUIRED if policy requires nonce binding |
| `subject` | bstr (32 bytes) | No | Principal identifier being authorised |
| `pid` | bstr (32 bytes) | Recommended | Human principal identifier who gave consent |

##### X.5.4 Consent Rules (Normative)

1. Consent evidence used for irreversible operations MUST bind to `sid`, `action_id`, `op_class`, and `expires_at`. All four fields are mandatory for irreversible actions.

2. Consent evidence MUST be rejected if any of the following hold:
   - `sid` mismatches the evaluated session binding `sid`
   - `action_id` mismatches the evaluated action identifier
   - `expires_at` is in the past at evaluation time
   - `op_class` mismatches the evaluated operation class

3. If policy requires nonce binding, consent evidence MUST include `nonce_hash`, and the execution request MUST provide a value that, when hashed, matches `nonce_hash`.

4. Absence of required binding fields MUST fail closed.

5. Consent receipts are single-use. The same consent receipt MUST NOT be used to authorise multiple distinct `action_id` values.

6. Consent MUST NOT use `scope="SESSION"` for irreversible actions. Each irreversible action requires its own consent bound to a specific `action_id`.

#### X.6 Delegation Receipt: `pqsf.delegation`

ReceiptEnvelope.type: `"pqsf.delegation"`
ReceiptEnvelope.body: `DelegationBody`

Delegation receipts are separate artefacts (NOT `pqsf.human_confirm` or `pqsf.human_approve`) and represent reusable authority within defined scope and limits.

##### X.6.1 DelegationBody Structure

DelegationBody (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `delegation_id` | bstr (16 bytes) | Yes | Unique delegation identifier |
| `delegator_pid` | bstr (32 bytes) | Yes | Principal granting delegation |
| `delegatee_pid` | bstr (32 bytes) | Yes | Principal receiving delegation |
| `scope` | array of tstr | Yes | Array of delegated `op_class` values |
| `limits` | map | Yes | Constraints (see X.6.2) |
| `valid_from` | uint | Yes | Delegation start tick |
| `valid_until` | uint | Yes | Delegation expiry tick |
| `redelegation_permitted` | bool | No | Whether delegatee may further delegate (default: false) |

##### X.6.2 Delegation Limits Structure

Limits (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `max_amount_sats` | uint | No | Maximum transaction amount in satoshis |
| `max_uses` | uint | No | Maximum number of uses |
| `recipient_allowlist` | array of bstr | No | Hashes of permitted recipient identifiers |
| `tool_allowlist` | array of tstr | No | Permitted tool_id values |

##### X.6.3 Delegation Rules (Normative)

1. Delegation is reusable only within its explicit scope and limits.

2. Delegation MUST NOT satisfy human consent requirements where policy requires consent. Delegation and consent are separate authority domains.

3. A delegatee MUST NOT further delegate unless `redelegation_permitted = true` in the delegation receipt.

4. Delegation MUST be rejected if:
   - `valid_from` is in the future at evaluation time
   - `valid_until` is in the past at evaluation time
   - Requested `op_class` is not in `scope`
   - Any limit constraint is exceeded
   - `delegator_pid` does not have authority to delegate the requested operation

5. Delegation chain depth MUST NOT exceed policy maximum (default: 3).

6. Delegation cycles (A delegates to B delegates to A) MUST be detected and rejected.

Refusal code: `E_DELEGATION_INVALID`

#### X.7 Participant Identity Derivation (Normative)

`pid` (participant identifier) MUST be derived deterministically from the participant's public key:

```
pid = SHAKE256-256(pubkey_bytes)
```

Where `pubkey_bytes` is the canonical compressed public key (33 bytes for secp256k1, or the canonical form for other curves) used to sign the ReceiptEnvelope.

Binding requirement:
`pid` MUST be bound to the ReceiptEnvelope issuer identity by verifying the receipt signature with the corresponding public key.

Key rotation:
Key rotation within a session is prohibited. If a new key is required, a new session MUST be established with a new `sid`.

#### X.8 Message `pid` Requirement (Normative)

| MessageClass | `pid` Required |
|--------------|----------------|
| `COORDINATION` | No (MAY be absent) |
| `PROPOSAL` | Yes |
| `REQUEST_AUTHORITY` | Yes |
| `EXECUTE` | Yes |

---

### Annex Y -- Backup and Recovery Re-Authorisation (Optional, Normative)

#### Y.1 Purpose

This annex defines backup and recovery authority semantics for PQSF-conformant systems.

Core principle: Recovery MUST restore state but MUST NOT implicitly restore authority.

Any recovery event MUST result in a zero-authority state until current policy predicates are satisfied.

#### Y.2 Backup Metadata Requirements (Normative)

Any backup artefact intended for later recovery MUST include the following metadata, cryptographically bound to the backup contents:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `descriptor_set_hash` | bstr (32 bytes) | Yes | SHAKE256-256 of the complete active descriptor set at backup time |
| `policy_version_hash` | bstr (32 bytes) | Yes | SHAKE256-256 of the PQSEC policy profile in force at backup time |
| `wallet_mode_flags` | map | Yes | Mode flags including `quantum_aware_enabled` (bool) |
| `backup_epoch` | uint | Yes | Epoch Clock tick or block height when backup was created |
| `backup_id` | bstr (16 bytes) | No | Unique identifier for this backup instance |

This metadata is contextual only and MUST NOT confer authority. Possession of backup metadata does not grant permission to execute actions.

#### Y.3 Recovery Integrity Check (Normative)

Upon recovery, before enabling any signing or execution capability, the system MUST:

1. Recompute the current:
   - `descriptor_set_hash` from the recovered descriptor set
   - `policy_version_hash` from the current policy profile

2. Compare recovered metadata against current values.

Outcomes:

| Condition | State | Action |
|-----------|-------|--------|
| All hashes match | Normal | Proceed to re-authorisation |
| Any hash differs | `RECOVERED_UNQUALIFIED` | Block signing, require reconciliation |

In `RECOVERED_UNQUALIFIED` state:
- Signing MUST be disabled
- All authority-bearing actions MUST fail closed
- System MUST display clear indication of unqualified state

#### Y.4 Post-Recovery Authority Reset (Normative)

After ANY recovery, regardless of metadata match:

1. All prior session state MUST be discarded.
2. No `pqsf.session_scope` from before recovery MUST be considered valid.
3. No transcript commitments from before recovery MUST be considered valid.
4. No consent artefacts from before recovery MUST be considered valid.
5. No tool profiles from before recovery MUST be implicitly trusted.

Recovery MUST be treated as a cold start for authority. The system resumes with zero authority.

#### Y.5 Re-Authorisation Requirement (Normative)

To regain execution capability after recovery, the system MUST require:

- Establishment of a new session (`sid`)
- Satisfaction of current PQSEC policy predicates
- Re-issuance of any required consent evidence
- Re-validation of recipient allowlists and exposure controls

Recovered material MUST NOT bypass:
- MessageClass enforcement
- Transcript binding requirements
- Supervision requirements
- Quantum-aware restrictions

#### Y.6 Staleness and Downgrade Handling (Normative)

If recovery metadata indicates mismatch or potential downgrade:

| Condition | Required Action |
|-----------|-----------------|
| Descriptor mismatch | Block signing until explicitly reconciled by user |
| Policy mismatch | Require explicit user acknowledgement |
| Quantum-aware downgrade attempt | MUST be prohibited, no override permitted |
| Missing exposure history | Default to stricter policy |

No staleness condition MAY silently permit execution.

#### Y.7 Backup Identity and Audit Semantics (Normative)

If `backup_id` is present:

1. The system SHOULD record recovery events keyed by `backup_id` in the local audit log.
2. Multiple recoveries from the same `backup_id` MAY be treated as elevated risk by policy.
3. PQSEC MAY apply additional supervision or approval requirements based on:
   - Recovery count from same `backup_id`
   - Time since backup creation
   - Policy-defined risk thresholds

Absence of `backup_id` MUST NOT weaken fail-closed semantics.

#### Y.8 Refusal Codes

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_RECOVERY_REQUIRES_REAUTHORISATION` | Authority-bearing action attempted before re-authorisation complete | Yes |
| `E_RECOVERY_POLICY_MISMATCH` | Recovered policy hash differs from current | No |
| `E_RECOVERY_DESCRIPTOR_MISMATCH` | Recovered descriptor hash differs from current | No |

#### Y.9 Conformance

A system claiming conformance to this annex MUST:

1. Bind backups to descriptor and policy metadata per Y.2.
2. Enforce recovery integrity checks per Y.3.
3. Reset authority state per Y.4.
4. Require re-authorisation per Y.5.
5. Handle staleness per Y.6.
6. Fail closed on mismatch or missing evidence.

---

### Annex Z -- Update and Policy Profile Integrity (Optional, Normative)

#### Z.1 Purpose

This annex defines integrity and downgrade protections for:
- Wallet software updates
- Policy profile updates
- Capability profile updates (including tool profile bundles)
- Descriptor set updates

A PQSF-conformant system MUST NOT accept an update unless it is authenticated, policy-compatible, and non-downgrading under configured constraints.

#### Z.2 Update Package Requirements (Normative)

An update MUST be distributed as an Update Package containing:

- `update_manifest` (deterministic CBOR map)
- `payload` (bytes)
- `signature` over the `update_manifest` bytes

##### Z.2.1 Update Manifest Structure

update_manifest (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Manifest schema version |
| `package_type` | tstr | Yes | One of: `WALLET_BINARY`, `POLICY_PROFILE`, `TOOL_PROFILE_BUNDLE`, `DESCRIPTOR_SET` |
| `package_id` | bstr (16 bytes) | Yes | Unique identifier for this package |
| `payload_hash` | bstr (32 bytes) | Yes | SHAKE256-256(payload) |
| `build_version` | uint | Yes | Monotonically increasing version number |
| `policy_version_hash` | bstr (32 bytes) | Conditional | REQUIRED for `WALLET_BINARY` and `POLICY_PROFILE` |
| `quantum_aware_required` | bool | Conditional | REQUIRED for `WALLET_BINARY` |
| `min_wallet_build_version` | uint | No | Minimum wallet version required for this update |
| `issued_at` | uint | Yes | Epoch Clock tick when issued |
| `expires_at` | uint | No | Expiry tick (if applicable) |

The signature MUST be verifiable using pinned update keys as defined in Z.3.

#### Z.3 Update Key Pinning (Normative)

The wallet MUST maintain a local allowlist of update signing keys.

Requirements:
- An Update Package MUST be rejected unless its signature verifies under at least one pinned update key.
- Pinned update keys MUST be scoped by `package_type`.
- A key pinned for `POLICY_PROFILE` MUST NOT be accepted for `WALLET_BINARY` unless explicitly pinned for both.
- Compromise of one key type MUST NOT imply trust in other key types.

Key storage:
- Update keys MUST be stored in a tamper-evident manner.
- Key rotation requires explicit user action and SHOULD require elevated approval.

#### Z.4 Rollback and Downgrade Protection (Normative)

The wallet MUST persist a local monotonic version record for each `package_type`:

```
last_accepted_build_version[package_type] : uint
```

Rejection rules:

An Update Package MUST be rejected if any of the following hold:

| Condition | Refusal Code |
|-----------|--------------|
| `build_version < last_accepted_build_version[package_type]` | `E_UPDATE_DOWNGRADE_PROHIBITED` |
| `build_version == last_accepted_build_version[package_type]` AND `payload_hash` differs | `E_UPDATE_EQUIVOCATION_DETECTED` |
| `expires_at` is present and is in the past | `E_UPDATE_EXPIRED` |

#### Z.5 Quantum-Aware Lock (Normative)

If the wallet has ever operated with quantum-aware mode enabled, the wallet MUST enforce a quantum-aware lock:

```
quantum_aware_lock = true // Permanent after first enablement
```

Lock behavior:

When `quantum_aware_lock = true`, the wallet MUST reject any update that would:
- Disable quantum-aware mode
- Set `quantum_aware_required = false` for a `WALLET_BINARY` update
- Revert address reuse prohibitions
- Revert exposure controls required by quantum-aware mode

Refusal code: `E_QUANTUM_AWARE_DOWNGRADE_PROHIBITED`

#### Z.6 Policy Profile Update Semantics (Normative)

A `POLICY_PROFILE` update MUST include:
- `policy_profile_bytes` as the payload
- `policy_version_hash = SHAKE256-256(policy_profile_bytes)` in the manifest

Minimum wallet version check:

If `min_wallet_build_version` is present:
- The wallet MUST reject the policy profile update if:
  `wallet_build_version < min_wallet_build_version`
- Refusal code: `E_POLICY_REQUIRES_NEWER_WALLET_BUILD`

Update procedure:

The wallet MUST:
1. Verify signature under pinned policy issuer key
2. Verify `payload_hash == SHAKE256-256(payload)`
3. Verify monotonic `build_version`
4. Enforce `min_wallet_build_version` if present
5. Install profile only if non-downgrading under Z.5

After installing a new policy profile:
- All active sessions MUST be invalidated
- Any authority-bearing actions MUST require re-authorisation under the new policy

#### Z.7 Authority Reset on Update (Normative)

After accepting any update of type `WALLET_BINARY` or `POLICY_PROFILE`, the wallet MUST:

1. Discard all prior session state
2. Discard any pending approvals
3. Require new session establishment (new `sid`)
4. Require re-authorisation for any authority-bearing actions

This prevents "update mid-session" attacks and approval reuse across trust boundaries.

#### Z.8 Audit (Local, Normative)

The wallet MUST record update events locally, including:

| Field | Description |
|-------|-------------|
| `package_type` | Type of update |
| `package_id` | Update identifier |
| `build_version` | Version installed |
| `payload_hash` | Hash of payload |
| `policy_version_hash` | Policy hash (when applicable) |
| `issued_at` | When update was issued |
| `installed_at` | When update was installed |

These records MUST NOT be transmitted by default. Transmission requires explicit user consent.

#### Z.9 Refusal Codes

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_UPDATE_SIGNATURE_INVALID` | Signature does not verify | No |
| `E_UPDATE_KEY_NOT_PINNED` | Signing key not in allowlist | No |
| `E_UPDATE_PAYLOAD_HASH_MISMATCH` | payload_hash != SHAKE256-256(payload) | No |
| `E_UPDATE_DOWNGRADE_PROHIBITED` | build_version < last_accepted | No |
| `E_UPDATE_EQUIVOCATION_DETECTED` | Same version, different payload | No |
| `E_UPDATE_EXPIRED` | expires_at in the past | No |
| `E_POLICY_DOWNGRADE_PROHIBITED` | Policy would weaken constraints | No |
| `E_QUANTUM_AWARE_DOWNGRADE_PROHIBITED` | Would disable quantum-aware mode | No |
| `E_POLICY_REQUIRES_NEWER_WALLET_BUILD` | Wallet too old for policy | Yes (after wallet update) |

#### Z.10 Conformance

A system claiming conformance to this annex MUST:

1. Require signed update manifests per Z.2
2. Enforce update key pinning per Z.3
3. Enforce rollback protection per Z.4
4. Enforce quantum-aware lock per Z.5
5. Handle policy updates per Z.6
6. Reset authority after update per Z.7
7. Maintain audit records per Z.8
8. Fail closed on any violation

---

### Annex AA -- Adapter Integrity and Binding (Optional, Normative)

#### AA.1 Purpose

This annex defines integrity, identity, and policy binding requirements for tool adapters.

A tool adapter is any executable component that implements one or more tool operations (e.g., `http_post`, `read_file`, `sign_psbt`, `broadcast_tx`).

When this annex is adopted by an enforcement specification, that specification MUST NOT permit execution of a tool operation unless it is mediated by a verified, policy-bound adapter identity.

#### AA.2 Adapter Identity (Normative)

Each adapter MUST have a stable identity derived from its executable form:

```
adapter_id = SHAKE256-256(adapter_binary_bytes)
```

Or for containerised adapters:
```
adapter_id = SHAKE256-256(container_image_bytes)
```

Requirements:
- The derivation method MUST be deterministic.
- The derivation MUST NOT depend on runtime state.
- The same binary/image MUST always produce the same `adapter_id`.

#### AA.3 Adapter Manifest (Normative)

Each adapter MUST be distributed with an Adapter Manifest, cryptographically bound to the adapter binary.

Adapter Manifest (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Manifest schema version |
| `adapter_id` | bstr (32 bytes) | Yes | SHAKE256-256 of adapter binary |
| `adapter_version` | uint | Yes | Monotonically increasing version |
| `tool_ids` | array of tstr | Yes | List of tool identifiers implemented |
| `issuer` | bstr (32 bytes) | Yes | Adapter signing authority identifier |
| `issued_at` | uint | Yes | Epoch Clock tick when issued |
| `expires_at` | uint | No | Expiry tick (if applicable) |

The Adapter Manifest MUST be signed by a pinned adapter issuer key.

#### AA.4 Adapter Profile Receipt

A verified adapter MUST be represented inside the system as a ReceiptEnvelope:

ReceiptEnvelope.type: `"pqsf.adapter_profile"`

Body (deterministic CBOR map):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version |
| `adapter_id` | bstr (32 bytes) | Yes | Adapter identity hash |
| `adapter_version` | uint | Yes | Version number |
| `tool_ids` | array of tstr | Yes | Implemented tools |
| `issuer` | bstr (32 bytes) | Yes | Issuer identifier |
| `issued_at` | uint | Yes | Issue tick |
| `expires_at` | uint | No | Expiry tick |

The receipt signature MUST verify under a pinned adapter issuer key.

#### AA.5 Adapter Key Pinning (Normative)

The system MUST maintain a local allowlist of adapter issuer keys.

Requirements:
- An adapter profile receipt MUST be rejected unless its signature verifies under a pinned adapter issuer key.
- Adapter issuer keys MUST be scoped independently from:
  - Wallet update keys
  - Policy profile keys
- Compromise of one key domain MUST NOT imply trust in the others.

#### AA.6 Tool Invocation Binding (Normative)

Before executing any tool operation, the adopting enforcement specification MUST evaluate the predicate:

```
valid_adapter_profile(adapter_profile_receipt_hash)
```

The predicate MUST PASS if and only if ALL of the following hold:

1. A valid `pqsf.adapter_profile` receipt is present in the evaluation context
2. `adapter_id` in the receipt matches the executing adapter's computed identity
3. The requested `tool_id` is included in the receipt's `tool_ids` array
4. The receipt is not expired (`expires_at` not in the past, or absent)
5. The receipt signature verifies under a pinned adapter issuer key
6. `adapter_version >= last_accepted_adapter_version[adapter_id]`

If the predicate fails, tool execution MUST fail closed.

#### AA.7 Adapter Downgrade Protection (Normative)

The system MUST persist, per `adapter_id`:

```
last_accepted_adapter_version[adapter_id] : uint
```

Rejection rules:

An adapter profile receipt MUST be rejected if:

| Condition | Refusal Code |
|-----------|--------------|
| `adapter_version < last_accepted_adapter_version[adapter_id]` | `E_ADAPTER_VERSION_DOWNGRADE` |
| `adapter_version == last_accepted_adapter_version[adapter_id]` AND `adapter_id` differs (equivocation) | `E_ADAPTER_PROFILE_INVALID` |

#### AA.8 Adapter--Policy Compatibility (Normative)

If a tool operation requires supervision or special handling under the active policy profile:
- The adapter profile MUST NOT claim broader capability than policy permits.
- Policy MUST take precedence over adapter claims.

An adapter profile MUST NOT weaken policy constraints. If conflict exists, the stricter constraint applies.

#### AA.9 Authority Reset on Adapter Change (Normative)

When a new adapter profile is accepted (new version or new adapter):

1. All sessions using the prior adapter version MUST be invalidated.
2. Any pending approvals involving the adapter MUST be discarded.
3. New authority MUST be re-established under current policy.

Adapter changes MUST be treated as a trust boundary change.

#### AA.10 Refusal Codes

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_ADAPTER_PROFILE_MISSING` | No adapter profile in evaluation context | Yes |
| `E_ADAPTER_PROFILE_INVALID` | Profile present but fails validation | No |
| `E_ADAPTER_ISSUER_NOT_PINNED` | Issuer key not in allowlist | No |
| `E_ADAPTER_VERSION_DOWNGRADE` | Version less than last accepted | No |
| `E_ADAPTER_TOOL_MISMATCH` | Requested tool not in tool_ids | No |

#### AA.11 Conformance

A system claiming conformance to this annex MUST:

1. Derive deterministic `adapter_id` values per AA.2
2. Require signed adapter manifests per AA.3
3. Represent adapters as `pqsf.adapter_profile` receipts per AA.4
4. Enforce adapter key pinning per AA.5
5. Enforce tool invocation binding per AA.6
6. Enforce downgrade protection per AA.7
7. Respect policy precedence per AA.8
8. Reset authority on adapter change per AA.9
9. Fail closed on any violation

---

### Annex AB -- Human Identity Binding (Optional, Normative)

#### AB.1 Purpose

This annex binds human consent to a verifiable identity, device, and session, preventing consent spoofing and device substitution attacks.

#### AB.2 Human Identity Binding Object

Human identity binding is represented as a deterministic CBOR map:

```
human_identity_binding = {
  "pid": bstr(32), // Human principal identifier
  "device_id": bstr(32), // Trusted device identifier
  "exporter_hash": bstr(32), // Session transport exporter binding
  "issued_tick": uint // When binding was created
}
```

| Field | Type | Description |
|-------|------|-------------|
| `pid` | bstr (32 bytes) | SHAKE256-256(human_pubkey_bytes) |
| `device_id` | bstr (32 bytes) | Identifier of the trusted device (derived from device key or attestation) |
| `exporter_hash` | bstr (32 bytes) | Hash of the secure transport exporter for session binding |
| `issued_tick` | uint | Epoch Clock tick when this binding was created |

#### AB.3 Binding Rules (Normative)

Human consent MUST be accepted only if ALL of the following hold:

1. `pid` matches the session subject (the human who established the session)
2. `device_id` matches a trusted device in the wallet's device allowlist
3. `exporter_hash` matches the current session's transport exporter binding
4. The binding is fresh under policy (not expired, within allowed time window)

#### AB.4 Enforcement Predicate

When this annex is adopted by an enforcement specification, that specification MUST evaluate:

```
valid_human_identity(pid, device_id, exporter_hash, issued_tick)
```

PASS if and only if:
- `pid` is in the session's `participants` with `role ∈ {HUMAN, SUPERVISOR}`
- `device_id` is in the trusted device list
- `exporter_hash` matches the session's exporter binding
- `issued_tick` is within the freshness window defined by policy

#### AB.5 Refusal Code

| Code | Condition | Retryable |
|------|-----------|-----------|
| `E_HUMAN_IDENTITY_INVALID` | Any binding rule fails | No |

#### AB.6 Conformance

Human authority MUST be device-bound and session-bound.

A system claiming conformance to this annex MUST:
1. Require human identity binding for consent receipts
2. Validate all binding fields per AB.3
3. Fail closed on any mismatch

---

### Annex AC -- Evidence Producer Governance (Normative)

#### AC.1 Purpose

This annex defines the canonical grammar for evidence producer governance artefacts. Evidence producers are components that emit artefacts consumed by PQSEC predicates -- including Neural Lock classifiers, PQAI runtimes, runtime attestation probes, and any future producer integrated into the enforcement surface.

PQSEC §22A defines the mechanism for verifying evidence producer authenticity. This annex provides the structured grammar that §22A consumes.

Without a formal governance object, evidence producer allowlists are implied configuration blobs with no auditable lifecycle. This annex makes producer governance explicit, composable, and deterministically verifiable.

#### AC.2 Design Principles

1. Evidence producer governance is a long-lived governance artefact -- it changes when producers are onboarded, revoked, or their authority scope changes.
2. Evidence freshness constraints are short-lived operational parameters -- they change when deployments tune freshness budgets or reuse policies.
3. These two concerns have different lifecycles, different signing authorities, and different audit surfaces. They MUST be separate artefacts.
4. Both artefacts are descriptive and structural. Neither grants authority. Enforcement is exclusively performed by PQSEC.

#### AC.3 EvidenceProducerProfile (Governance Artefact)

EvidenceProducerProfile defines a governed evidence producer and bounds its influence within the enforcement surface.

```
EvidenceProducerProfile = {
  producer_id:              tstr,
  producer_kind:            tstr,
  allowed_predicates:       [+ tstr],
  allowed_operation_classes: [+ tstr],
  build_allowlist:          [+ bstr],
  governance: {
    threshold:              uint,
    signers:                [+ tstr]
  },
  valid_from_tick:          uint,
  expiry_tick:              uint,
  revoked:                  bool,
  evidence_type_constraints_ref: bstr / null,
  suite_profile:            tstr,
  signature:                bstr
}
```

##### AC.3.1 Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `producer_id` | tstr | Unique identifier for this evidence producer |
| `producer_kind` | tstr | Producer type. Values: `"neural_lock"`, `"pqai"`, `"pqvl_probe"`, `"supply_chain"`, `"custom"` |
| `allowed_predicates` | array of tstr | Predicate names this producer's evidence may influence. Evidence from this producer MUST be ignored for predicates not in this list. |
| `allowed_operation_classes` | array of tstr | Operation classes (`"Authoritative"`, `"NonAuthoritative"`) for which this producer's evidence is admissible. |
| `build_allowlist` | array of bstr | SHAKE256-256 hashes of permitted build artefacts or manifest hashes for this producer. |
| `governance` | map | Threshold signature requirements for this profile. |
| `governance.threshold` | uint | Minimum number of valid governance signatures required. |
| `governance.signers` | array of tstr | Identifiers of governance key holders authorised to sign this profile. |
| `valid_from_tick` | uint | Epoch Clock tick from which this profile is active. |
| `expiry_tick` | uint | Epoch Clock tick at which this profile expires. |
| `revoked` | bool | If true, this profile is revoked and MUST NOT be used. |
| `evidence_type_constraints_ref` | bstr or null | SHAKE256-256 hash of the associated EvidenceTypeConstraints artefact, if any. Null if no constraints are bound. |
| `suite_profile` | tstr | CryptoSuiteProfile identifier for signature verification. |
| `signature` | bstr | Signature over the canonical CBOR bytes of this artefact with the `signature` field omitted. |

##### AC.3.2 Validation Rules

1. EvidenceProducerProfile MUST be canonical CBOR per PQSF §7.
2. All required fields MUST be present.
3. `signature` MUST verify under the declared `suite_profile` and governance keys.
4. `governance.threshold` MUST be satisfied -- at least `threshold` governance signers MUST have signed.
5. `valid_from_tick` MUST NOT be in the future relative to the current verified Epoch Clock tick at the time of activation.
6. `expiry_tick` MUST be strictly greater than `valid_from_tick`.
7. If `revoked` is true, the profile MUST NOT be used for any evaluation.
8. `build_allowlist` MUST contain at least one entry. Empty allowlists MUST be rejected.
9. `allowed_predicates` MUST contain at least one entry.
10. `allowed_operation_classes` MUST contain at least one entry.
11. `producer_kind` MUST be a recognised value or explicitly permitted by policy.

##### AC.3.3 Governance Lifecycle

1. Onboarding: A new EvidenceProducerProfile MUST be issued with governance signatures meeting the threshold requirement. The profile MUST be active (past `valid_from_tick`, not past `expiry_tick`, not revoked) before the producer's evidence is admissible.
2. Rotation: A new profile MUST be issued before the current profile expires. The new profile MUST NOT reduce `allowed_operation_classes` scope without governance approval at the level of the highest operation class being removed.
3. Revocation: Setting `revoked = true` MUST require the same governance threshold as the original profile. Revocation takes effect immediately for all operation classes.
4. Build Update: Adding a new build hash to `build_allowlist` requires a new signed EvidenceProducerProfile. Build hashes MUST NOT be added by mutation -- a new profile version MUST be issued.

##### AC.3.4 Predicate Scope Enforcement

When PQSEC evaluates a predicate using evidence from a producer:

1. PQSEC MUST verify that the predicate being evaluated is listed in the producer's `allowed_predicates`.
2. PQSEC MUST verify that the current operation class is listed in the producer's `allowed_operation_classes`.
3. If either check fails, the evidence MUST be treated as absent for that predicate (evaluating as UNAVAILABLE under §8A semantics).

This prevents silent authority expansion: a producer cannot influence predicates it was not explicitly authorised to affect.

#### AC.4 EvidenceTypeConstraints (Policy Sub-Object)

EvidenceTypeConstraints defines freshness, reuse, and staleness parameters for evidence types emitted by a producer. It is referenced by hash from EvidenceProducerProfile and consumed by PQSEC for freshness enforcement.

```
EvidenceTypeConstraints = {
  constraints_id:     tstr,
  producer_ref:       tstr,
  evidence_types: [+ {
    evidence_type:    tstr,
    max_age_ticks:    uint,
    reuse_allowed:    bool,
    reuse_scope:      "none" / "operation" / "session",
    op_class_overrides: [* {
      operation_class: tstr,
      max_age_ticks:   uint / null,
      reuse_allowed:   bool / null,
      reuse_scope:     ("none" / "operation" / "session") / null
    }] / null
  }],
  issued_tick:        uint,
  suite_profile:      tstr,
  signature:          bstr
}
```

##### AC.4.1 Field Definitions

| Field | Type | Description |
|-------|------|-------------|
| `constraints_id` | tstr | Unique identifier for this constraints artefact |
| `producer_ref` | tstr | `producer_id` of the EvidenceProducerProfile this constrains |
| `evidence_types` | array of maps | Per-evidence-type freshness and reuse parameters |
| `evidence_types[].evidence_type` | tstr | Evidence type identifier (e.g., `"neural_lock.operator_attestation"`, `"pqai.drift_evidence"`) |
| `evidence_types[].max_age_ticks` | uint | Maximum age in Epoch Clock ticks before evidence is considered stale |
| `evidence_types[].reuse_allowed` | bool | Whether evidence may be reused across evaluations |
| `evidence_types[].reuse_scope` | tstr | If reuse is allowed, the scope: `"none"` (no reuse), `"operation"` (within a single operation evaluation), or `"session"` (within a single session) |
| `evidence_types[].op_class_overrides` | array or null | Optional per-operation-class overrides for any of the above parameters. Null fields inherit the parent value. |
| `issued_tick` | uint | Epoch Clock tick at which these constraints were issued |
| `suite_profile` | tstr | CryptoSuiteProfile identifier for signature verification |
| `signature` | bstr | Signature over canonical CBOR bytes with `signature` field omitted |

##### AC.4.2 Validation Rules

1. EvidenceTypeConstraints MUST be canonical CBOR per PQSF §7.
2. `producer_ref` MUST reference an active, non-revoked EvidenceProducerProfile.
3. The SHAKE256-256 hash of the canonical bytes of this artefact MUST match the `evidence_type_constraints_ref` in the referencing EvidenceProducerProfile.
4. `max_age_ticks` MUST be greater than zero.
5. If `reuse_allowed` is false, `reuse_scope` MUST be `"none"`.
6. `op_class_overrides` entries MUST reference valid operation classes.
7. `signature` MUST verify under the declared `suite_profile`.

##### AC.4.3 Freshness Budget Consumption

When PQSEC evaluates evidence against EvidenceTypeConstraints:

1. PQSEC MUST compute the age of the evidence: `current_verified_tick - evidence.issued_tick`.
2. If age exceeds `max_age_ticks` (or the applicable `op_class_overrides.max_age_ticks`), the evidence MUST be treated as stale and the predicate MUST evaluate as UNAVAILABLE.
3. If `reuse_allowed` is false and the evidence has been presented in a prior evaluation for the same scope, the evidence MUST be treated as replayed and the predicate MUST evaluate as FALSE.
4. PQSEC MUST record freshness budget consumption events in audit receipts with the following event classes: `FRESH`, `NEAR_EXPIRY`, `REUSED`, `STALE_REJECTED`. No exact time deltas or fine-grained counts shall be recorded to prevent observability side-channels.

#### AC.5 ReceiptEnvelope Integration

EvidenceProducerProfile and EvidenceTypeConstraints are governance and policy artefacts. They are NOT ReceiptEnvelope types. They are consumed directly by PQSEC as configuration inputs, not as evidence in the ReceiptEnvelope container.

However, changes to producer governance MUST be recorded as audit events using the existing ReceiptEnvelope `pqsec.decision_receipt` type with a domain-specific `refusal_code` if the change triggers enforcement consequences.

#### AC.6 Authority Boundary

EvidenceProducerProfile and EvidenceTypeConstraints are structural governance artefacts. They:

1. Grant no authority.
2. Do not modify enforcement semantics.
3. Do not override predicate evaluation.
4. Define the admissibility and freshness boundaries within which evidence is considered for PQSEC evaluation.

All enforcement outcomes remain exclusively produced by PQSEC.

#### AC.7 Conformance

An implementation claiming conformance to this annex MUST:

1. Support EvidenceProducerProfile as a first-class governance artefact with the schema defined in AC.3.
2. Support EvidenceTypeConstraints as a policy sub-object with the schema defined in AC.4.
3. Validate all fields per AC.3.2 and AC.4.2.
4. Enforce predicate scope (AC.3.4) -- reject evidence for predicates not in `allowed_predicates`.
5. Enforce operation class scope (AC.3.4) -- reject evidence for operation classes not in `allowed_operation_classes`.
6. Enforce freshness budgets (AC.4.3) when EvidenceTypeConstraints are bound.
7. Record freshness budget consumption as event classes, not fine-grained timing data.
8. Reject profiles with empty `build_allowlist`, `allowed_predicates`, or `allowed_operation_classes`.
9. Fail closed on any validation failure of either artefact.

---

### Annex SM -- StalenessStateMachine (Normative)

This annex defines a shared graded staleness primitive used by consumers to map tick distance to an operational state.

```
StalenessState = "FRESH" | "STALE_WARN" | "STALE_LOCK"
```

```
StalenessStateMachine = {
  last_good_tick:      uint,
  warn_window_ticks:   uint,
  lock_window_ticks:   uint
}
```

Rules:

1. `warn_window_ticks` MUST be > 0.
2. `lock_window_ticks` MUST be >= `warn_window_ticks`.
3. Compute `delta = current_tick - last_good_tick`. If `delta < 0`, treat as LOCK.
4. If `delta <= warn_window_ticks`, state is FRESH.
5. If `warn_window_ticks < delta <= lock_window_ticks`, state is STALE_WARN.
6. If `delta > lock_window_ticks`, state is STALE_LOCK.

Consumers MUST fail closed on STALE_LOCK.

Refusal code: `E_STALENESS_LOCK`

---

### Annex ED -- Operation-Scoped Emission Discipline (Normative)

Evidence producers in the PQ ecosystem MUST obey operation-scoped emission discipline.

Rules:

1. Evidence producers MUST emit evidence only within the scope of an explicit operation attempt.
2. Evidence producers MUST NOT emit background telemetry, periodic heartbeats, or unsolicited synchronisation that is observable outside the holder enforcement boundary.
3. Evidence producers MUST NOT allow the presence or absence of evidence to change outward timing characteristics.
4. If an evidence producer cannot produce evidence within an operation scope, it MUST emit `UNAVAILABLE` semantics as defined by the consuming specification (typically PQSEC ternary model).

This annex is consumed by PQSEC, PQAI, Neural Lock, and PQPS.

Refusal code: `E_EMISSION_DISCIPLINE_VIOLATION`

---

### Annex AD -- Determinism Verification Harness (Normative)

#### AD.1 Purpose

This ecosystem makes strong determinism claims. Conformance requires testable determinism obligations.

This Annex defines mandatory test categories and required properties for:
- canonical encoding,
- hashing and signature preimages,
- intent hashes and commitment hashes,
- STP framing correctness,
- EnforcementOutcome equivalence inputs (where consumed by PQSEC).

#### AD.2 Required Test Classes

Implementations claiming conformance with PQSF MUST implement:

1. **Re-encoding Stability Tests**

   For any canonical artefact `x`:

   `encode(decode(encode(x)))` MUST equal `encode(x)` byte-for-byte.

   This MUST hold for every artefact type the implementation produces or consumes.

2. **Cross-Implementation Byte Equivalence Tests**

   Given published test vectors, independent implementations MUST produce identical bytes for:
   - canonical CBOR encoding of the same logical artefact,
   - hash outputs given the same canonical bytes,
   - signature preimage bytes given the same artefact fields.

3. **Mutation Sensitivity Tests**

   Single-field mutation of any canonical artefact MUST:
   - change the relevant commitment hash or intent hash, and
   - cause signature verification failure where the artefact is signed.

   Implementations MUST demonstrate this for at least: one field per artefact type, the `issued_tick` field (where present), and the `suite_profile` field (where present).

4. **STP Framing Correctness Tests**

   Implementations claiming STP conformance MUST demonstrate:
   - sequence monotonicity is enforced (out-of-order or duplicate `sequence` values are rejected),
   - MAC verification occurs before payload processing (invalid `mac` never triggers payload parsing),
   - CLOSE destroys session keys and forbids `session_id` reuse,
   - all valid state transitions are exercised,
   - all invalid state transitions produce `STP-ERROR` and key destruction.

#### AD.3 Minimum Determinism Properties

For each canonical artefact type implemented, conformance MUST demonstrate:

- **P1: Encode is deterministic.** The same logical artefact always produces the same bytes.
- **P2: Hash input bytes are deterministic.** The hash preimage for a given artefact is always the same bytes.
- **P3: Signature preimage bytes are deterministic.** The bytes signed for a given artefact are always the same.
- **P4: Equality checks use canonical bytes only.** No semantic normalisation beyond what the specification permits is applied during comparison.

#### AD.4 Equivalence Class Partitions (Recommended)

Implementations SHOULD partition test inputs for each predicate and artefact type into at minimum:

- **Valid nominal:** expected success cases.
- **Valid boundary:** edge of acceptance (e.g., tick exactly at expiry, sequence at maximum representable value).
- **Invalid structural:** malformed input (e.g., non-canonical CBOR, missing required fields).
- **Invalid semantic:** well-formed but policy-violating (e.g., expired tick, wrong suite_profile).
- **Expired temporal:** valid structure but stale tick.
- **Replayed:** valid structure but duplicate nonce, sequence, or decision_id.

#### AD.5 Publication Requirement (Recommended)

Implementations SHOULD publish:

- a conformance test runner that can be executed independently,
- a minimal set of cross-implementation vectors for canonical encoding,
- a "known answer test" corpus for canonical artefact encoding and hashing.

---

### Annex AE -- STP KEM Fallback Derivation Example (Informative)

```python
import hashlib
import hmac

def derive_stp_session_keys_with_kem_fallback(
    tls_exporter_output: bytes,
    kem_shared_secret: bytes,
    session_id: bytes,
    role_id: bytes
) -> dict:
    """
    Derive STP session keys when STP-layer KEM fallback is used
    (Authoritative operations without PQ TLS).
    
    Per §27.6A:
    PRK = HKDF-Extract(salt=E, ikm=SS)
    session_key   = HKDF-Expand(PRK, info="PQSF-STP-SESSION"   ‖ session_id ‖ role_id, L=32)
    integrity_key = HKDF-Expand(PRK, info="PQSF-STP-INTEGRITY" ‖ session_id ‖ role_id, L=32)
    """
    
    # HKDF-Extract: PRK = HMAC-SHA256(salt=E, ikm=SS)
    prk = hmac.new(
        key=tls_exporter_output,      # salt = E
        msg=kem_shared_secret,        # ikm = SS
        digestmod=hashlib.sha256
    ).digest()
    
    # HKDF-Expand for session_key
    session_info = b"PQSF-STP-SESSION" + session_id + role_id
    session_key = hmac.new(
        key=prk,
        msg=session_info + b"\x01",
        digestmod=hashlib.sha256
    ).digest()
    
    # HKDF-Expand for integrity_key
    integrity_info = b"PQSF-STP-INTEGRITY" + session_id + role_id
    integrity_key = hmac.new(
        key=prk,
        msg=integrity_info + b"\x01",
        digestmod=hashlib.sha256
    ).digest()
    
    return {
        "session_key": session_key,
        "integrity_key": integrity_key
    }


def derive_stp_session_keys_with_pq_tls(
    tls_exporter_output: bytes,
    session_id: bytes,
    role_id: bytes
) -> dict:
    """
    Derive STP session keys when PQ TLS is available (no STP-layer KEM).
    
    Per §27.6A:
    PRK = HKDF-Extract(salt=0x00*32, ikm=E)
    session_key   = HKDF-Expand(PRK, info="PQSF-STP-SESSION"   ‖ session_id ‖ role_id, L=32)
    integrity_key = HKDF-Expand(PRK, info="PQSF-STP-INTEGRITY" ‖ session_id ‖ role_id, L=32)
    """
    
    # HKDF-Extract: PRK = HMAC-SHA256(salt=0x00*32, ikm=E)
    prk = hmac.new(
        key=b"\x00" * 32,                # salt = 0x00*32
        msg=tls_exporter_output,          # ikm = E
        digestmod=hashlib.sha256
    ).digest()
    
    # HKDF-Expand for session_key
    session_info = b"PQSF-STP-SESSION" + session_id + role_id
    session_key = hmac.new(
        key=prk,
        msg=session_info + b"\x01",
        digestmod=hashlib.sha256
    ).digest()
    
    # HKDF-Expand for integrity_key
    integrity_info = b"PQSF-STP-INTEGRITY" + session_id + role_id
    integrity_key = hmac.new(
        key=prk,
        msg=integrity_info + b"\x01",
        digestmod=hashlib.sha256
    ).digest()
    
    return {
        "session_key": session_key,
        "integrity_key": integrity_key
    }


def verify_kem_confirm(
    kem_shared_secret: bytes,
    received_kem_confirm: bytes
) -> bool:
    """
    Verify kem_confirm field in STP-ACCEPT (§27.6A).
    kem_confirm = SHAKE256-256(shared_secret)
    """
    from hashlib import shake_256
    expected = shake_256(kem_shared_secret).digest(32)
    return hmac.compare_digest(expected, received_kem_confirm)
```

---

### Annex AF -- ReplayStore Interface (Informative)

This annex defines a small reference interface for durable replay guards used across the PQ ecosystem (`decision_id`, `revocation_id`, `request_id`, `outcome_id`, `signal_id`, etc.).

```
ReplayStore = {
  namespace: tstr,                    // e.g. "pqsec.decision_id", "pqps.request_id", "bpc.outcome_id"
  put_once(id: bstr, expiry_tick: uint) -> bool,
  contains(id: bstr) -> bool,
  prune(current_tick: uint) -> uint
}
```

Recommended Behaviour:

1. `put_once` MUST be atomic: returns `false` if `id` is already present, `true` if newly inserted.
2. Storage MUST persist across restarts.
3. Loss of replay store integrity SHOULD cause fail-closed behaviour for Authoritative operations until integrity is restored.
4. `prune` MAY remove entries only when `expiry_tick < current_tick`.

This annex is advisory only. Consuming specifications remain authoritative for refusal semantics and retention requirements.

---

### Annex AG -- PQ Stack Starter Pack and Conformance Harness (Normative)

#### AG.1 Purpose

This appendix defines the structure and content requirements for the PQ Stack Starter Pack: a self-contained reference package that enables implementers to bootstrap conformant deployments.

The starter pack is tooling discipline only and introduces no protocol authority.

#### AG.2 Required Directory Structure

```
pq-starter-pack/
├── examples/           ; Complete artefact examples for all types
├── fixtures/           ; Deterministic test fixtures
├── harness/            ; Conformance test harness
├── compatibility/      ; Cross-version compatibility test cases
└── reference/          ; Reference implementation fragments
```

#### AG.3 Mandatory Examples

The `examples/` directory MUST include at least one complete, valid, canonical CBOR-encoded example for each artefact type defined in the active PQSF version, including:

1. CryptoSuiteProfile
2. ReceiptEnvelope (at least one per receipt type namespace)
3. PolicyBundle with at least one PolicyObject
4. ConsentProof
5. EBTEnvelope
6. LedgerEntry and MerkleRoot
7. FleetTelemetryEnvelope (Section 17B)
8. BufferedMeasurementEnvelope (Section 21X)
9. MediaCommitment, CaptureReceipt, and MediaDeleteReceipt (Section 22X)
10. EvidenceDescriptor (Section 32B)

Each example MUST include both the canonical CBOR bytes (as hex) and the decoded field values for human inspection.

#### AG.4 Fixture Requirements

The `fixtures/` directory MUST include:

1. Deterministic test vectors for all hash operations (Tier 1, Tier 2, Tier 3).
2. Deterministic test vectors for signature verification under at least one CryptoSuiteProfile.
3. At least one canonicalization round-trip test per artefact type (encode → decode → re-encode → byte-identical).
4. At least one rejection test per structural validation rule (non-canonical input → rejection).

#### AG.5 Harness Requirements

The `harness/` directory MUST include:

1. A test runner specification (input format, expected output format, pass/fail criteria).
2. Coverage mapping: every normative MUST in PQSF Sections 7--32B MUST have at least one corresponding test case.
3. Test cases for schema version governance (32A): supported version acceptance, unsupported rejection, downgrade refusal.

#### AG.6 Compatibility Requirements

The `compatibility/` directory MUST include:

1. Test vectors from at least the prior minor version of PQSF.
2. Forward-compatibility tests: artefacts with unknown optional fields at the current version MUST parse without error.
3. Backward-compatibility tests: artefacts at the minimum supported version MUST parse correctly.

#### AG.7 Conformance Obligation

1. Implementations claiming the embedded-minimal enforcement profile (PQSEC Annex AU.X) MAY omit the starter pack.
2. Implementations claiming standard or higher enforcement profiles MUST provide a starter pack conforming to this appendix.
3. Absence of a required starter pack invalidates claims of full ecosystem conformance.

---

### Annex AH -- Bootstrap in Hostile or Isolated Networks (Normative)

#### AH.1 Purpose

This appendix defines cold-start requirements for deployments where no network time sources are trusted and system clocks are non-authoritative. It complements PQSEC BOOTSTRAP mode (Section 29.1 EvaluationState) by defining the PQSF-layer tick bootstrapping mechanism.

#### AH.2 Pre-Provisioning Requirements

1. Epoch Clock profile pinning MUST be pre-provisioned before deployment. The pinned profile MUST include the canonical profile reference, public key(s) for tick signature verification, and profile version.
2. The pinned profile MUST be stored in tamper-resistant or tamper-evident storage.
3. System clocks, NTP, and network time sources MUST NOT be used as authoritative time during bootstrap.

#### AH.3 First Tick Acceptance

The first valid Epoch Clock tick MAY be obtained via:

1. Local mirror (pre-provisioned or physically transferred).
2. Offline transfer (signed tick bundle on removable media).
3. Preloaded tick bundle (included in deployment package, with tick signature verification on first boot).

In all cases:

1. The tick MUST verify against the pinned profile.
2. The tick MUST satisfy structural validation per PQSF Section 14.
3. After the first valid tick is accepted, BOOTSTRAP mode exits and normal tick refresh rules apply.

#### AH.4 Operational Constraints During Bootstrap

1. All Authoritative operations MUST be refused until the first valid tick is accepted.
2. Non-Authoritative operations MAY proceed during bootstrap only if explicitly permitted by the pre-provisioned policy.
3. No artefact with an `expiry_tick` or `issued_tick` field may be validated until a reference tick is established.

#### AH.5 Post-Bootstrap Behaviour

After the first valid tick is accepted:

1. Normal tick freshness and monotonicity rules apply (per PQSEC Section 18).
2. BOOTSTRAP mode MUST NOT be re-entered except after factory reset (per PQSEC Section 18.1).
3. The bootstrap tick establishes the initial monotonicity floor for all subsequent tick validation.

#### AH.6 Authority Boundary

This appendix defines tick bootstrapping mechanics only. It does not define enforcement behaviour during bootstrap (that is PQSEC's responsibility) and does not create new authority surfaces.

---

### Annex AI -- Gateway Adapter Pattern for Non-PQ Services (Normative)

This annex defines a normative bridging pattern for invoking external services that do not implement PQ artefacts or STP natively (e.g. Slack API, GitHub API, Gmail API, payment processor APIs).

A Gateway Adapter is not an enforcement authority. It MUST NOT evaluate predicates and MUST NOT override PQSEC decisions.

#### AI.1 Admission and Governance

1. Gateway Adapters MUST be admitted via PQSEC extension/adapter admission discipline (PQSEC Annex AX).
2. Gateway invocations MUST be governed by the active Tool Capability Profile (PQAI §27.2).
3. Gateway operations MUST fail closed if required governance artefacts (capability profile, delegation scope, session scope) are absent or invalid.
4. Admission MUST verify canonical encoding, signature validity, and binary_hash.

#### AI.2 GatewayManifest Receipt (Normative)

Gateway adapters MUST be admitted via a signed ReceiptEnvelope whose body is the canonical GatewayManifest.

**ReceiptEnvelope.type:** `"pqsf.gateway_manifest"`

**GatewayManifestBody (deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version (MUST be 1) |
| `gateway_id` | tstr | Yes | Unique gateway identifier |
| `target_domain` | tstr | Yes | Domain of the non-PQ service |
| `supported_operations` | [* tstr] | Yes | Operations this gateway can bridge |
| `credential_derivation_context` | tstr | Yes | PQSF §13 derivation context string |
| `binary_hash` | bstr (32 bytes) | Yes | SHAKE256-256 hash of gateway adapter binary |
| `issued_tick` | uint | Yes | Epoch Clock tick at issuance |
| `expiry_tick` | uint / null | No | Optional expiry tick |
| `suite_profile` | tstr | Yes | CryptoSuiteProfile reference |
| `signature` | bstr | Yes | Signature over canonical body with `signature` omitted |

**Rules:**

1. The manifest receipt is evidence only. It grants no authority.
2. Admission MUST verify:
   - Deterministic CBOR encoding
   - Signature validity under suite_profile
   - binary_hash matches installed gateway binary
3. Operations not in `supported_operations` MUST be refused by PQSEC. Default AE mapping: `E_POLICY_CONSTRAINT_FAILED`.

#### AI.3 Domain-Scoped Credential Derivation (Normative)

Gateway Adapters MUST derive service credentials using PQSF §13 Universal Secret Derivation.

Derivation context map MUST include:

```
{
  "derivation_purpose": "gateway_credential",
  "service_id": <tstr>,
  "credential_scope": <tstr>,
  "agent_binding": <bstr(32)>,
  "issued_tick": <uint>
}
```

Derived credential material MUST NOT be emitted in receipts, logs, or exports. Only commitments MAY be recorded.

#### AI.4 Gateway Call Receipt (Normative)

Every gateway invocation MUST emit a ReceiptEnvelope.

**ReceiptEnvelope.type:** `"pqsf.gateway_call"`

**GatewayCallBody (deterministic CBOR map):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `v` | uint | Yes | Schema version (MUST be 1) |
| `agent_binding` | bstr (32 bytes) | Yes | SHAKE256-256 of pinned ModelIdentity |
| `tool_id` | tstr | Yes | Tool identifier |
| `service_id` | tstr | Yes | Target service |
| `action` | tstr | Yes | Operation performed |
| `request_commitment` | bstr (32 bytes) | Yes | SHAKE256-256 of canonical request |
| `response_commitment` | bstr (32 bytes) / null | Yes | SHAKE256-256 of canonical response |
| `status` | tstr | Yes | `"SUCCESS"` / `"FAILED"` |
| `pqsec_decision_id` | bstr (16 bytes) / null | No | EnforcementOutcome reference |
| `issued_tick` | uint | Yes | Tick at response capture |
| `signature` | bstr | Yes | Signature over canonical body with `signature` omitted |

**Rules:**

1. `pqsec_decision_id` MUST be present if and only if the call executed under ALLOW.
2. If `pqsec_decision_id` is present, it MUST correspond to a currently valid EnforcementOutcome for the same `agent_binding` and `tool_id`.
3. `status=FAILED` is execution-layer failure, not a PQSEC refusal.
4. Raw credentials MUST NOT appear in any field.
5. The gateway MUST NOT retry automatically.

#### AI.5 Authority Boundary

This annex defines a bridging pattern. It does not grant authority, modify enforcement semantics, or create new predicate types. All enforcement remains exclusively within PQSEC.

#### AI.6 PQ Gateway Integration (Informative)

PQ Gateway (PQGW) defines a product-layer specialisation of this gateway adapter pattern for model inference APIs. PQ Gateway Provider Adapters (PQGW §P2) extend the GatewayManifest with provider-specific fields (provider_class, supported_model_ids, privacy_classification, response_format) and translate between proprietary inference APIs and canonical PQ artefacts. All PQSF Annex AI governance requirements (admission, credential derivation, fail-closed semantics, GatewayManifest receipts) apply to PQ Gateway Provider Adapters without modification.

---

## Changelog

### Version 2.0.3

* Added `"PQSF-Secret:ProviderCredential"` to §13.3 Standard Context Strings for PQ Gateway model provider credential derivation.
* Added `pqgw.*` to §W.6 Receipt Type Namespacing table for PQ Gateway product-layer receipts.
* Added §AI.6 (Informative) documenting PQ Gateway as a product-layer specialisation of the Annex AI Gateway Adapter Pattern.
* Added **Section 8.7 -- Cryptographic Sunset Discipline**: defines monotonic sunset states (ACTIVE, SUNSET_PENDING, SUNSET_FINAL), bounded dual-acceptance windows, rollback prohibition, and offline deployment rules for algorithm retirement. Introduces `E_PROFILE_SUNSET_FINAL` refusal code.
* Added **Section 17B -- Privacy-Preserving Fleet Telemetry Envelope**: defines constant-shape, correlation-minimized telemetry with `FleetTelemetryEnvelope` and `TelemetryBudget`. Enforces structural anonymity (no stable device identifiers, tick-windowed timing, operation-scoped emission).
* Added **Section 21X -- BufferedMeasurementEnvelope**: defines deterministic store-and-forward measurement batching with `MeasurementRecord`, tick-bounded batches, ordering rules, and export governance. Ensures connectivity absence does not cause partial emission.
* Added **Section 22X -- CaptureReceipt and MediaCommitment**: defines tamper-evident capture commitments with `MediaCommitment`, `CaptureReceipt`, and `MediaDeleteReceipt`. Includes PQPS interaction clause establishing deletion semantics precedence.
* Added **Section 32A -- Schema Version Governance**: defines deterministic schema version handling with deployment version bounds, session floor ratcheting, and mixed-version refusal. Introduces `E_SCHEMA_VERSION_UNSUPPORTED` and `E_SCHEMA_DOWNGRADE_ATTEMPT` refusal codes.
* Added **Section 32B -- Evidence Classification Vocabulary**: defines provenance taxonomy (hardware_attested, secure_boot_bound, software_measured, model_inferred, user_asserted, external_asserted) and `EvidenceDescriptor` artefact. Classification is descriptive only. Introduces `E_EVIDENCE_DESCRIPTOR_REQUIRED` refusal code.
* Added **Annex AG -- PQ Stack Starter Pack and Conformance Harness**: defines starter pack directory structure, mandatory examples, fixture requirements, harness coverage, and conformance obligation.
* Added **Annex AH -- Bootstrap in Hostile or Isolated Networks**: defines cold-start tick provisioning requirements complementary to PQSEC BOOTSTRAP mode.
* Updated **Conformance and Verification** (Section 31, formerly Conformance Checklist) with entries for all new sections.
* Extended STP with normative STP-DATA and STP-CLOSE message types (§27.2.5, §27.2.6), providing framed application data with monotonic sequencing, body hashing, session-bound MAC, and orderly session termination with key destruction mandate.
* Harmonised `session_id` type across all STP message types, exporter binding context (§15), ConsentProof (§16), EBTEnvelope (§21), and KeyMail (Annex V) to `bstr(16)`. All STP session identifiers are now fixed-size 16-byte CSPRNG output, eliminating encoding ambiguity between text and binary representations.
* Added `kem_ciphertext` field to STP-INIT and `kem_confirm` field to STP-ACCEPT for inline §27.6A fallback KEM negotiation.
* Expanded STP canonical encoding requirements (§27.3) to explicitly cover body_hash, mac, and payload signature computation ordering.
* Added normative STP Handshake State Machine (§27.5) with 7 defined states, valid transition set, timeout requirements with hard bounds, error recovery rules, and key material lifecycle.
* Added normative STP-to-TLS Binding (§27.6) specifying exporter binding derivation, channel binding verification, and Authoritative vs Non-Authoritative operational classification.
* Added normative STP-Layer PQ KEM Fallback (§27.6A) for Authoritative sessions when TLS does not provide post-quantum key exchange, including ML-KEM-1024 exchange during session establishment, kem_confirm verification, and dual-path key derivation.
* Added normative STP Integrity and KDF Primitives (§27.6B) explicitly specifying HKDF-SHA256 for key derivation and HMAC-SHA256 for MAC computation, with byte-precise input and output length requirements.
* Added normative STP Credential Lifecycle (§27.7) covering credential derivation, domain binding, storage and non-extractability, atomic rotation with explicit failure semantics, and revocation.
* Added informative STP Web Discovery (§27.8) with HTTP header, well-known URI, and DNS TXT mechanisms.
* Expanded Annex G with 5 lifecycle diagrams including DATA/CLOSE flows, error recovery, idle timeout, and KEM fallback path.
* Added normative Annex AD (Determinism Verification Harness) defining required test classes, minimum determinism properties, equivalence class partitions, and publication requirements.
* Added informative Annex AE (STP KEM Fallback Derivation Example) with reference pseudocode for both KEM fallback and PQ TLS key derivation paths.
* Updated Conformance (Section 29) and Conformance and Verification (Section 31, formerly Conformance Checklist) to include STP state machine, framing, binding, credential, and determinism verification requirements.
* Added **Section 31.1 -- Canonical Encoding Verification**: normative requirement that conformant implementations process the PQSF Canonical CBOR Test Vector Suite, demonstrate byte-identical encoding and hash equivalence, compute hashes over canonical bytes only, and incorporate regression testing for encoder drift.
* Clarified PQSF’s role as a protocol-layer foundation only. PQSF defines grammar, canonical encoding, and cryptographic indirection. All enforcement and authority semantics remain external.
* Standardised deterministic CBOR for all PQSF-native artefacts and formalised the Epoch Clock JCS Canonical JSON exception with strict no-re-encoding rules.
* Expanded CryptoSuiteProfile governance, including signed distribution, revocation, downgrade prevention, and emergency cryptographic transition handling.
* Refined exporter binding primitives for deterministic, session-scoped security independent of network trust.
* Tightened ConsentProof, PolicyObject, and PolicyBundle definitions as inert, non-authoritative structures with canonical validation only.
* Introduced and consolidated supply-chain artefact grammars, reproducible build support, and ledger anchoring.
* Finalised Encrypted-Before-Transport (EBT) wrapper structure and key-derivation rules.
* Finalised Sovereign Transport Protocol (STP) grammar and capability negotiation.
* Cleaned annex boundaries, clearly fencing normative, optional, and informative material.

---

If you find this work useful and wish to support continued development, donations are welcome:

**Bitcoin:**
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw
