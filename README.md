# **PQSF — Post-Quantum Security Framework**

An Open Standard for Deterministic Post-Quantum Security Primitives for Internet Protocols, AI Systems and Digital Asset Custody

**Specification Version:** v1.0.0
**Status:** Implementation Ready. Protocol Review Requested.
**Author:** rosiea
**Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
**Date:** November 2025
**Licence:** Apache License 2.0 — Copyright 2025 rosiea


---

# **SUMMARY**

The Post-Quantum Security Framework (PQSF) defines deterministic, post-quantum-safe primitives for transport, authorisation, and decision-making across heterogeneous environments. It replaces trust in local clocks, operating-system state, coordinators, cloud services, and identity intermediaries with cryptographically verifiable constructs that behave identically across all compliant implementations.

PQSF’s goal is to give systems **user sovereignty**, **strong privacy guarantees**, and **cross-implementation consistency** without depending on DNS, NTP, PKI, or trusted hosts. It does this by explicitly defining time, consent, policy, runtime-state, and structure validity as first-class protocol elements.

## **Core Problem and Operating Model**

Traditional Internet security depends on assumptions that are often false: system clocks can be modified, OS state can be tampered with, UI consent can be forged, and identity services can leak or be unavailable. Replay, rollback, downgrade, and drift attacks remain possible because critical information is not verifiable in a deterministic way.

PQSF removes these assumptions. Every sensitive operation must pass a unified predicate set:

```
valid_tick
AND valid_consent
AND valid_policy
AND valid_runtime     (optional: PQVL)
AND valid_quorum      (child-spec)
AND valid_ledger
AND valid_structure
```

If any element fails, the system fails-closed.

Key design principles include deterministic behaviour, canonical encoding, post-quantum safety, explicit time binding, exporter-bound sessions, OS-agnostic logic, and stateless transport.

## **The Three Core Authorities**

### **1. EpochTick — Temporal Authority**

A signed, verifiable timestamp that replaces untrusted system time.
Ticks must be **fresh** (≤ 900 s), **monotonic**, and signed using **ML-DSA-65**.
All consent, policy, ledger, and runtime decisions are bound to the active tick and its pinned Epoch Clock profile.

### **2. ConsentProof — Intent Authority**

Explicit, cryptographically authenticated user intent.
Bound to:

* a specific action
* a session exporter_hash
* a fresh EpochTick
* an ML-DSA-65 signature
* a deterministic, canonical structure

This prevents replay across sessions or contexts.

### **3. Policy Enforcer — Policy Authority**

Deterministic evaluation of allowlists, denylists, thresholds, limits, time windows, and constraints.
Policies must be canonicalised and hashed with **SHAKE256-256**, ensuring they cannot be modified without detection.

## **Transport, Encoding, and Cryptography**

### **Encrypted-Before-Transport (EBT)**

A universal rule: all sensitive artefacts must be encrypted *before* entering any transport, including TLSE-EMP, STP, or cloud storage.
Hosts and intermediaries cannot access plaintext.

### **Transport Profiles**

* **TLSE-EMP** — deterministic TLS 1.3 profile enforcing exporter binding, replay resistance, and downgrade detection.
* **STP (Sovereign Transport Protocol)** — DNS-free, offline-capable transport with deterministic framing for sovereign and air-gapped environments.

### **Canonical Encoding**

All PQSF objects use deterministic **CBOR** or **JCS JSON**, eliminating ambiguity and ensuring consistent hashing, signatures, and structure-validation across implementations.

### **Post-Quantum Cryptography**

* Signatures: **ML-DSA-65**
* Hashing: **SHAKE256-256**
* Derivation: **cSHAKE256** with domain separation
* Optional encryption: **ML-KEM-1024**, AES-256-GCM, or XChaCha20-Poly1305

### **Deterministic Ledger**

An append-only Merkle ledger with freeze-on-error semantics, recording tick validation, consent events, policy rotations, runtime results, and profile changes.
Ensures integrity and strict monotonic history.

---

## **INDEX**

[ABSTRACT](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#abstract)

[PROBLEM STATEMENT](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#problem-statement)

[PURPOSE](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#purpose)

[WHAT THIS SPECIFICATION COVERS](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#what-this-specification-covers)

---

## **2. ARCHITECTURE**

[2. ARCHITECTURE OVERVIEW](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#2-architecture-overview-normative)

[2.1 Components](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#21-components)

[2.2 Minimal Compliance](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#22-minimal-compliance)

[2.3 Design Principles](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#23-design-principles)

[2.4 Device Identity](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#24-device-identity-in-pqsf-consumers-informative)

[2.5 Independence from Central Infrastructure](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#25-independence-from-central-infrastructure-informative)

---

## **3. CRYPTOGRAPHIC PRIMITIVES**

[3.1 Hash Functions](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#31-hash-functions)

[3.2 Key Derivation](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#32-key-derivation)

[3.3 Universal Secret Derivation](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#33-universal-secret-derivation-normative)

[3.4 Post-Quantum Signatures](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#34-post-quantum-signatures)

[3.5 Canonical Encoding](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#35-canonical-encoding)

---

## **4. TEMPORAL AUTHORITY — EPOCH TICK**

[4. Temporal Authority — EpochTick](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#4-temporal-authority--epoch-tick-normative)

[4.1 Tick Structure](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#41-tick-structure)

[4.2 Profile Structure & Lineage](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#42-profile-structure--parentchild-clock-selection)

[4.3 Freshness Rule](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#43-freshness-rule-tick-age-limit)

[4.4 Monotonicity](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#44-monotonicity)

[4.5 profile_ref Validation](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#45-profile_ref-validation)

[4.6 TickCache](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#46-tickcache)

---

## **5. INTENT AUTHORITY — CONSENTPROOF**

[5. Intent Authority — ConsentProof](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#5-intent-authority--consentproof-normative)

[5.1 Structure](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#51-structure)

[5.2 Canonicalisation](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#52-canonicalisation)

[5.3 Tick Binding](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#53-tick-binding)

[5.4 Exporter Binding](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#54-exporter-binding-session-binding)

[5.5 Role Binding](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#55-role-binding)

[5.6 Signature Requirements](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#56-signature-requirements)

---

## **6. POLICY AUTHORITY — POLICY ENFORCER**

[6. Policy Authority — Policy Enforcer](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#6-policy-authority--policy-enforcer-normative)

[6.1 Structure](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#61-structure)

[6.2 Canonicalisation](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#62-canonicalisation)

[6.3 Required Predicates](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#63-required-predicates)

[6.4 Tick-Dependent Rules](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#64-tick-dependent-rules)

[6.5 Allowlist / Denylist Enforcement](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#65-allowlist--denylist-enforcement)

[6.6 Threshold & Constraint Rules](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#66-threshold--constraint-rules)

[6.7 Signature Bundles](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#67-signature-bundles)

---

## **7. TRANSPORT SECURITY — TLSE-EMP & STP**

[7. Transport Security](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#7-transport-security--tlse-emp-and-stp-normative)

[7.1 TLSE-EMP](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#71-tlse-emp-deterministic-tls)

[7.2 Exporter Hash Derivation](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#72-exporter-hash-derivation-normative)

[7.3 Downgrade Resistance](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#73-downgrade-resistance)

[7.4 STP](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#74-stp--sovereign-transport-protocol)

[7.5 Transport Fail-Closed](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#75-transport-fail-closed)

[7.6 Encrypted-Before-Transport (EBT)](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#76-encrypted-before-transport-ebt-requirement-normative)

---

## **8. LEDGER**

[8. Ledger](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#8-ledger-normative)

[8.1 Ledger Entry Structure](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#81-ledger-entry-structure)

[8.2 Merkle Construction](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#82-merkle-construction)

[8.3 Monotonic Ordering](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#83-monotonic-ordering)

[8.4 Reconciliation](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#84-reconciliation)

[8.5 Required Events](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#85-required-ledger-events)

[8.6 Fail-Closed Ledger Behaviour](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#86-fail-closed-ledger-behaviour)

---

# **ANNEXES**

[ANNEX A — MVP Compliance Profile](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-a--mvp-compliance-profile-normative)

[ANNEX B — Full & Extended Compliance Profiles](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-b--full--extended-compliance-profiles-normative)

[ANNEX C — Workflow Examples](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-c--workflow-examples-informative)

[ANNEX D — Recovery Capsules](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-d--recovery-capsules-normative)

[ANNEX E — CDDL Definitions](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-e--cddl-definitions-informative)

[ANNEX F — Reference Test Vectors](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-f--reference-test-vectors-informative)

[ANNEX G — Secure Payments Module](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-g--secure-payments-module-optional-normative)

[ANNEX H — Cryptographic Erasure Proof](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-h--cryptographic-erasure-proof-optional-normative)

[ANNEX I — Browser Privacy & Consent Controls](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-i--browser-privacy--consent-controls-optional-normative)

[ANNEX J — Ephemeral Carts & Privacy-Preserving Ecommerce](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-j--ephemeral-carts--privacy-preserving-ecommerce-optional-normative)

[ANNEX K — Verified Identity & KYC Credentials](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-k--verified-identity--kyc-credentials-optional-normative)

[ANNEX L — Delegated Identity](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-l--delegated-identity-optional-normative)

[ANNEX M — Delegated Payments](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-m--delegated-payments-optional-normative)

[ANNEX N — Quantum-Safe Wallet-Backed Login](https://github.com/rosieRRRRR/pqsf?tab=readme-ov-file#annex-n--quantum-safe-wallet-backed-user-login-optional-normative)

---

# **ABSTRACT**

The Post-Quantum Security Framework (PQSF) defines deterministic security primitives that augment existing transport protocols with verifiable time, explicit cryptographic intent, canonical policy evaluation, and privacy-preserving execution semantics. PQSF introduces three core constructs—EpochTick, ConsentProof, and Policy Enforcer that binds authorisation to a signed temporal reference, an authenticated session, and deterministic policy logic. These constructs operate alongside a profile of TLS 1.3 (TLSE-EMP) and a sovereign, DNS-independent transport (STP), providing replay resistance, downgrade protection, and cross-implementation consistency without modifying existing TLS wire formats.

PQSF is designed for environments that require user sovereignty and strong privacy guarantees. It enables local verification of all security conditions without reliance on system clocks, centralised authorities, or third-party identity services. Canonical encoding and fail-closed semantics ensure that sensitive operations cannot occur under stale time, ambiguous structures, or unverified runtime conditions. PQSF also defines an optional runtime-validity predicate that may be satisfied by external attestation layers such as PQVL.

Optional annexes provide identity, delegated authority, delegated payments, and privacy modules that extend, but do not modify the core framework.

---

# **PROBLEM STATEMENT**

Existing Internet security models assume trust in local clocks, operating-system state, centralised time sources, UI-level consent mechanisms, and broad identity disclosures. These assumptions undermine both privacy and security: replay and rollback attacks become feasible, stale signatures remain valid, and authorisation decisions often depend on unverifiable local state. Transport protocols provide confidentiality and endpoint authentication, but they do not bind operations to explicit user intent, signed time, deterministic policies, or privacy-preserving identity semantics.

Current designs also assume centralised intermediaries or ambient system state for runtime trust. This limits user sovereignty: users cannot independently verify authorisation conditions, and cannot operate securely in constrained, offline, or partitioned environments. Privacy is further weakened by inconsistent encoding, metadata leakage, and ad-hoc handling of identity attributes or consent records.

PQSF defines deterministic locally verifiable primitives that bind time, intent, policy, and (optionally) runtime-validity into a single transport-compatible model. PQSF allows systems to operate without trusting local clocks, coordinators, or external identity providers, while supporting optional integration with external attestation layers such as PQVL when available.

---

# **PURPOSE**

PQSF defines a deterministic, post-quantum-secure foundation for authorisation and transport-bound decision making across heterogeneous systems. It enables privacy-preserving operation and user sovereignty through local verification of time, intent, policy, and structural integrity without dependency on central authorities or OS-level trust.

## PQSF Provides

* verifiable, privacy-preserving temporal authority (EpochTick)
* explicit, cryptographically authenticated intent (ConsentProof)
* deterministic and canonical policy enforcement (Policy Enforcer)
* exporter-bound session identity over TLSE-EMP and STP
* canonical deterministic encoding for all security-relevant artefacts
* deterministic cSHAKE256-based key-derivation and lifecycle rules
* an append-only, deterministic Merkle ledger for integrity-critical events
* a runtime-validity predicate that may be satisfied by PQVL
* mandatory Encrypted-Before-Transport semantics for all sensitive content
* sovereign, DNS-free STP transport with offline and Stealth Mode operation
* a Probe API for runtime, privacy, AI-alignment, and policy-related checks
* compliance tiers (MVP, FULL, EXTENDED) for interoperable deployments
* optional privacy-preserving identity and selective-disclosure credentials
* optional delegated identity and delegated-payment authority models
* optional KeyMail out-of-band confirmation for high-risk flows
* deterministic recovery capsule structures for controlled restoration
* optional wallet-backed login using tick-bound, consent-bound assertions
* privacy-preserving ecommerce flows (EphemeralSession and EphemeralCart)
* cloud-sovereignty rules ensuring all PQSF artefacts are encrypted locally before storage or transmission

These primitives support privacy-first deployments where data minimisation, local verification, and transport-bound semantics are required.

### Pseudocode (Informative) — High-Level PQSF Predicate Pipeline

```pseudo
// High-level PQSF decision: may this sensitive operation proceed?
function pqsf_may_proceed(ctx):
    // 1. Validate tick (EpochTick)
    (ctx.valid_tick, ctx.tick_error) =
        pqsf_validate_tick(ctx.tick, ctx.pinned_profile_ref, ctx.last_tick)

    if not ctx.valid_tick:
        return deny("E_TICK_INVALID", ctx.tick_error)

    // 2. Validate ConsentProof
    (ctx.valid_consent, ctx.consent_error) =
        pqsf_validate_consent(ctx.consent, ctx.session.exporter_hash, ctx.tick)

    if not ctx.valid_consent:
        return deny(ctx.consent_error)

    // 3. Validate Policy Enforcer
    (ctx.valid_policy, ctx.policy_error) =
        pqsf_evaluate_policy(ctx.policy, ctx.policy_context)

    if not ctx.valid_policy:
        return deny(ctx.policy_error)

    // 4. Optional runtime validity via PQVL
    if ctx.has_pqvl:
        (ctx.valid_runtime, ctx.runtime_error) =
            pqsf_check_runtime(ctx.attestation, ctx.tick, ctx.attestation_window)
        if not ctx.valid_runtime:
            return deny("E_RUNTIME_INVALID", ctx.runtime_error)
    else:
        ctx.valid_runtime = true

    // 5. Ledger integrity
    (ctx.valid_ledger, ctx.ledger_error) =
        pqsf_check_ledger(ctx.ledger_state, ctx.tick)

    if not ctx.valid_ledger:
        return deny("E_LEDGER_INVALID", ctx.ledger_error)

    // 6. Quorum (child-spec: PQHD, etc.)
    (ctx.valid_quorum, ctx.quorum_error) =
        pqsf_check_quorum_if_applicable(ctx)

    if not ctx.valid_quorum:
        return deny("E_POLICY_THRESHOLD_FAILED", ctx.quorum_error)

    // 7. Structural validity of all objects
    if not pqsf_valid_structure(ctx):
        return deny("E_TRANSPORT_CANONICAL_FAIL")

    // 8. All predicates satisfied
    return allow()
```

---

## **SCOPE**

PQSF covers:

* temporal authority and freshness semantics (EpochTick)
* explicit-intent binding, consent structures, and authorisation windows (ConsentProof)
* deterministic policy evaluation, canonical policy hashing, and mandatory predicates (Policy Enforcer)
* transport-layer session binding, exporter-hash semantics, and replay resistance (TLSE-EMP, STP)
* canonical deterministic encoding for all security-relevant objects (CBOR / JCS JSON)
* deterministic key-derivation and lifecycle rules based on cSHAKE256
* deterministic Merkle-ledger construction, reconciliation, and freeze-on-error semantics
* runtime-validity integration via the Probe API and PQVL
* Encrypted-Before-Transport requirements for all sensitive artefacts
* sovereign operation, including DNS-free STP, offline and Stealth-Mode support
* privacy-preserving operation, including metadata minimisation and local-first verification
* conformance profiles (MVP, FULL, EXTENDED) for interoperable deployments
* optional identity, selective-disclosure, delegated-authority, and delegated-payment modules
* optional wallet-backed, quantum-safe user authentication
* optional privacy-preserving ecommerce flows (EphemeralSession, EphemeralCart)
* cloud-sovereignty rules ensuring all PQSF artefacts are encrypted locally before storage or transmission

PQSF does **not** define:

* runtime measurement logic (see PQVL 1.1–1.2)
* AI alignment (see PQAI 1.1–1.2)
* tick issuance or profile lineage (see Epoch Clock 1.1–1.2 and 3.6.1)
* identity storage or application-specific data flows

PQSF allows—but does not require—external attestation systems such as PQVL to supply runtime-validity predicates.

### Pseudocode (Informative) — Validating a PQSF Context Object

```pseudo
// Generic sanity check over a "PQSF context" grouping all primitives
function pqsf_validate_context(ctx):
    if not pqsf_is_canonical(ctx.epoch_tick):
        return false

    if not pqsf_is_canonical(ctx.consent):
        return false

    if not pqsf_is_canonical(ctx.policy):
        return false

    if not pqsf_is_canonical(ctx.ledger_entry_template):
        return false

    // Further checks are spec-section-specific, but this
    // ensures at least structural correctness
    return true
```

---

# **WHAT THIS SPECIFICATION COVERS**

This specification defines:

1. **Temporal authority (EpochTick)** — structure, signature validation, freshness, monotonicity, and profile-ref rules enabling privacy- and sovereignty-preserving time validation.

2. **Intent authority (ConsentProof)** — canonical structure, tick-bounded validity, exporter-bound session semantics, and explicit, privacy-preserving user-intent enforcement.

3. **Policy authority (Policy Enforcer)** — deterministic policy evaluation, mandatory predicates, thresholds, limits, time windows, and canonical hashing for centralisation-free enforcement.

4. **Transport binding** — deterministic TLS (TLSE-EMP) and sovereign transport (STP), including exporter-hash binding, replay resistance, metadata minimisation, offline operation, and Stealth-Mode behaviour.

5. **Canonical encoding** — deterministic CBOR and JCS JSON encoding for all security-relevant objects, ensuring cross-implementation reproducibility and minimising metadata leakage.

6. **Deterministic key-derivation** — cSHAKE256-based derivation rules for identity, consent, transport, audit, delegation, and recovery keys, with no reliance on local clocks or entropy.

7. **Deterministic ledger** — append-only, Merkle-verified, monotonic event recording with reconciliation, freeze-on-error semantics, and sovereignty-preserving local operation.

8. **Encrypted-Before-Transport (EBT)** — a mandatory requirement that all sensitive artefacts be encrypted locally prior to transmission or storage across any transport, including cloud environments.

9. **Probe API integration** — a standardised interface for runtime, privacy, and AI-related probes, consumed by PQVL, PQAI, and other modules requiring deterministic validation inputs.

10. **Sovereign and offline operation** — rules enabling DNS-free STP, offline tick reuse, Stealth-Mode constraints, and deterministic behaviour under partitioned or air-gapped deployments.

11. **Compliance profiles** — MVP, FULL, and EXTENDED deployment profiles for constrained, offline, sovereign, or multi-device ecosystems.

12. **Optional identity and authority modules** — selective-disclosure identity credentials, delegated-identity, and delegated-payment flows, each using canonical tick-bound, consent-bound structures.

13. **Optional wallet-backed authentication** — deterministic, exporter-bound, quantum-safe login mechanisms using PQHD-derived identity keys.

14. **Optional privacy-preserving ecommerce flows** — EphemeralSession and EphemeralCart structures that support metadata-minimal merchant interactions.

External attestation systems such as PQVL may satisfy PQSF’s runtime-validity predicate but are not required.

---

## **1.4 Security Uplift (INFORMATIVE)**

PQSF introduces an authenticated temporal authority (Epoch Clock), ensuring operations occur under a fresh, verifiable time source rather than untrusted system time. This removes replay, rollback, forged expiry, and NTP dependency issues.

ConsentProof binds user intent to specific actions with tick-bound validity windows. Policy Enforcer enforces deterministic, tick-bound, policy-bound evaluation for sensitive operations.

PQSF integrates PQVL for runtime integrity verification (see PQVL 6), preventing operations under tampered execution environments. Deterministic encoding, exporter binding, and fail-closed semantics unify the system against replay, downgrade, and malleability attacks.

---

# **2. ARCHITECTURE OVERVIEW (NORMATIVE)**

PQSF defines the shared primitives used by higher-level protocols.

## **2.1 Components**

1. **Temporal Authority (Epoch Clock)**
2. **Intent Authority (ConsentProof)**
3. **Policy Authority (Policy Enforcer)**
4. **Transport Security (TLSE-EMP)**
5. **Sovereign Transport Protocol (STP)**
6. **Deterministic Derivation**
7. **Ledger**

### Pseudocode (Informative) — Architectural Data Flow

```pseudo
// Abstract data flow through PQSF components
function pqsf_process_operation(op_ctx):
    // Temporal authority
    (valid_tick, tick_err) =
        pqsf_validate_tick(op_ctx.tick, op_ctx.pinned_profile_ref, op_ctx.last_tick)
    if not valid_tick:
        return deny(tick_err)

    // Intent authority
    (valid_consent, consent_err) =
        pqsf_validate_consent(op_ctx.consent, op_ctx.session.exporter_hash, op_ctx.tick)
    if not valid_consent:
        return deny(consent_err)

    // Policy authority
    (valid_policy, policy_err) =
        pqsf_evaluate_policy(op_ctx.policy, op_ctx.policy_context)
    if not valid_policy:
        return deny(policy_err)

    // Transport
    if not pqsf_validate_transport(op_ctx.session):
        return deny("E_EXPORTER_MISMATCH")

    // Ledger
    pqsf_append_ledger_event(op_ctx.ledger, op_ctx.event_template, op_ctx.tick)

    return allow()
```

## **2.2 Minimal Compliance**

Three compliance levels:

* MVP
* FULL
* EXTENDED

Defined in Annex A.

## **2.3 Design Principles**

* determinism
* fail-closed semantics
* quantum safety
* canonical encoding
* explicit tick binding
* OS-agnostic behaviour
* stateless transport

## **2.4 Device Identity in PQSF Consumers (INFORMATIVE)**

Allows:

* PQVL-derived attested identity
* deterministic minimal device identity

Identity mismatch MUST cause fail-closed behaviour.

## **2.5 Independence from Central Infrastructure (INFORMATIVE)**

All PQSF continuity derives from exporter binding, deterministic encoding, and on-chain Epoch Clock profiles. No DNS, NTP, PKI, or cloud dependency.

---

# **3. CRYPTOGRAPHIC PRIMITIVES (NORMATIVE)**

## **3.1 Hash Functions**

PQSF MUST use SHAKE256-256 for:

* event hashing
* canonical object hashing
* Merkle nodes
* derivation contexts

### Pseudocode (Informative) — Canonical Object Hash

```pseudo
// Hash a PQSF object deterministically
function pqsf_hash_object(obj):
    bytes = pqsf_canonical_encode(obj)   // CBOR or JCS JSON
    return shake256_256(bytes)
```

## **3.2 Key Derivation**

PQSF uses **cSHAKE256** for deterministic derivation of:

* audit keys
* identity keys
* delegation keys
* consent keys
* transport keys
* recovery keys
* device-bound keys

### **3.3 Universal Secret Derivation (NORMATIVE)**

PQSF supports deterministic derivation of domain-scoped secrets:

```
Secret = cSHAKE256(root_seed,
                   domain="PQSF-Secret:<context>",
                   input=canonical(context_parameters))
```

Secrets MUST be:

* deterministic
* unlinkable
* canonical-input-derived

This enables PQHD or other consumers to behave as universal password/security managers — PQSF itself defines only the primitives.

### Pseudocode (Informative) — Universal Secret Derivation

```pseudo
// Derive a scoped secret from a root seed and context
function pqsf_derive_secret(root_seed, context_label, context_params):
    domain = "PQSF-Secret:" + context_label
    input_bytes = pqsf_canonical_encode(context_params)
    return cshake256(root_seed, domain, input_bytes)
```

## **3.4 Post-Quantum Signatures**

PQSF MUST use ML-DSA-65 for:

* consent proofs
* policy bundles
* ledger entries
* session-bound objects

### Pseudocode (Informative) — ML-DSA Sign / Verify Wrapper

```pseudo
function pqsf_sign_pq(sk_pq, obj):
    bytes = pqsf_canonical_encode(obj)
    sig = ml_dsa_65_sign(sk_pq, bytes)
    return sig

function pqsf_verify_pq(pk_pq, obj, sig):
    bytes = pqsf_canonical_encode(obj)
    return ml_dsa_65_verify(pk_pq, bytes, sig)
```

## **3.5 Canonical Encoding**

Deterministic CBOR or JCS JSON MUST be used.
Re-encoding between hashing and signing is forbidden.

### Pseudocode (Informative) — Canonical Encoding Selection

```pseudo
// Global configuration
const PQSF_ENCODING_MODE = "JCS_JSON"  // or "CBOR"

function pqsf_canonical_encode(obj):
    if PQSF_ENCODING_MODE == "JCS_JSON":
        return jcs_canonical_json_encode(obj)
    else:
        return deterministic_cbor_encode(obj)

function pqsf_is_canonical(bytes):
    decoded   = decode(bytes)             // chosen encoding
    reencoded = pqsf_canonical_encode(decoded)
    return (reencoded == bytes)
```

---

# **4. TEMPORAL AUTHORITY — EPOCH TICK (NORMATIVE)**

PQSF uses the Epoch Clock as the authoritative source of time across all ConsentProof, Policy Enforcer, ledger, and session-bound operations.
The Epoch Clock is defined on-chain via a **parent inscription (Epoch Clock v2)**, with all subsequent updates published as **child inscriptions**.

PQSF does not define Clock issuance; it defines how Clock profiles and ticks are **validated**.

---

## **4.1 Tick Structure**

All PQSF implementations MUST support the following canonical EpochTick structure:

```text
EpochTick = {
  "t": uint,
  "profile_ref": tstr,
  "alg": "ML-DSA-65",
  "sig": bstr
}
```

### Tick validation requirements:

Ticks MUST be:

* ML-DSA-65 signature verified
* encoded using deterministic CBOR or JCS JSON
* consistent with the currently pinned Epoch Clock profile
* fresh (≤900 seconds old by default)
* monotonic (strictly increasing `t`)

PQSF is pinned to the following Epoch Clock profile:

```
pinned_profile_ref = "ordinal:439d7ab1972803dd984bf7d5f05af6d9f369cf52197440e6dda1d9a2ef59b6ebi0"
```

All EpochTicks validated under PQSF MUST reference this profile_ref.
Ticks referencing any other profile MUST be rejected and MUST cause fail-closed behaviour.

### Pseudocode (Informative) — Inline Tick Validation

```pseudo
function pqsf_validate_tick(tick, pinned_profile_ref, last_tick):
    if tick.profile_ref != pinned_profile_ref:
        return (false, "E_TICK_PROFILE_MISMATCH")

    if tick.alg != "ML-DSA-65":
        return (false, "E_TICK_INVALID")

    if not pqsf_verify_pq(epoch_clock_pubkey_for_profile(tick.profile_ref), {
        t: tick.t,
        profile_ref: tick.profile_ref,
        alg: tick.alg
    }, tick.sig):
        return (false, "E_TICK_INVALID")

    if last_tick != null and tick.t <= last_tick.t:
        return (false, "E_TICK_ROLLBACK")

    if (pqsf_unix_now_from_clock() - tick.t) > 900:
        return (false, "E_TICK_STALE")

    return (true, null)
```

---

## **4.2 Profile Structure & Parent/Child Clock Selection**

The Epoch Clock v2 profile is published as a **parent inscription**.
Updates are published as **child inscriptions**, forming a deterministic lineage.

### Active profile selection rules:

1. Identify the parent Epoch Clock v2 inscription.
2. Enumerate all child inscriptions linked to the parent.
3. Select the **most recent valid child** inscription as the active profile.
4. If no children exist, the parent is the active profile.
5. Validate the canonical encoding and profile hash.
6. Pin the active profile for all subsequent tick validation.
7. If a child is malformed or invalid → fail-closed and fall back to the last known valid profile.

This model ensures that the Epoch Clock is externally controlled while PQSF locally verifies lineage.

### Pseudocode (Informative) — Active Profile Resolution

```pseudo
function pqsf_select_active_profile(parent_inscription, child_inscriptions):
    valid_children = []

    for child in child_inscriptions:
        if pqsf_validate_profile_child(parent_inscription, child):
            valid_children.append(child)

    if len(valid_children) == 0:
        return parent_inscription.profile_ref

    latest = select_child_with_max_height(valid_children)
    return latest.profile_ref
```

---

## **4.3 Freshness Rule (Tick Age Limit)**

Given:

```text
current_tick = latest_validated_tick()
```

A tick is valid iff:

```text
t_current - t_issued <= 900 seconds
```

Deployments MAY configure stricter bounds.

Expired ticks MUST cause:

* `valid_tick = false`
* all dependent operations (consent, policy, ledger) to fail-closed

---

## **4.4 Monotonicity**

PQSF enforces temporal strictness:

```
tick_new.t > tick_previous.t
```

If violated, PQSF MUST:

* reject the tick,
* halt dependent operations,
* require a fresh tick,
* revalidate the profile_ref.

---

## **4.5 profile_ref Validation**

Each EpochTick MUST reference the active Clock profile.

PQSF MUST verify:

* profile_ref corresponds to:

  * the parent inscription, or
  * the most recent valid child inscription
* canonical profile hash
* ML-DSA-65 signature on the profile
* lineage consistency

Ticks referencing unknown or invalid profiles MUST be rejected.

---

## **4.6 TickCache**

Implementations **MAY** cache validated ticks for up to 900 seconds.

TickCache MUST be invalidated if:

* the active profile changes
* a transport session restarts
* PQVL runtime attestation fails
* monotonicity is violated

TickCache MUST NOT bypass freshness or signature rules.

### Pseudocode (Informative) — TickCache Management

```pseudo
function pqsf_get_valid_tick(cache, pinned_profile_ref, now_unix):
    if cache.tick is not null:
        // Check if cached tick still valid
        (valid, err) = pqsf_validate_tick(cache.tick, pinned_profile_ref, cache.last_tick)
        if valid and (now_unix - cache.tick.t) <= 900:
            return cache.tick

    // Otherwise fetch a fresh tick from Epoch Clock
    tick = epoch_clock_fetch_latest()
    (valid, err) = pqsf_validate_tick(tick, pinned_profile_ref, cache.last_tick)
    if not valid:
        raise err

    cache.tick = tick
    cache.last_tick = tick
    return tick
```

---

# **5. INTENT AUTHORITY — CONSENTPROOF (NORMATIVE)**

ConsentProof binds **user intent** to **action**, **transport session**, and **temporal authority**, ensuring explicit, verifiable authorisation.

---

## **5.1 Structure**

```
ConsentProof = {
  "action": tstr,
  "intent_hash": bstr,
  "tick_issued": uint,
  "tick_expiry": uint,
  "exporter_hash": bstr,
  "role": tstr / null,
  "consent_id": tstr,
  "signature_pq": bstr
}
```

---

## **5.2 Canonicalisation**

ConsentProof MUST be canonically encoded using:

* deterministic CBOR **or**
* JCS JSON (RFC 8785)

Non-canonical forms MUST be rejected.

### Pseudocode (Informative) — Building a ConsentProof

```pseudo
function pqsf_build_consent(action, intent_obj, tick, exporter_hash, role_opt):
    intent_hash = pqsf_hash_object(intent_obj)

    consent = {
        action:        action,
        intent_hash:   intent_hash,
        tick_issued:   tick.t,
        tick_expiry:   tick.t + 900,   // or stricter
        exporter_hash: exporter_hash,
        role:          role_opt,
        consent_id:    generate_uuid()
    }

    consent_bytes = pqsf_canonical_encode(consent)
    consent.signature_pq = ml_dsa_65_sign(consent_sk, consent_bytes)
    return consent
```

---

## **5.3 Tick Binding**

Consent validity requires both:

```
tick_issued <= current_tick.t
```

and:

```
current_tick.t <= tick_expiry
```

Tick expiry defaults to **≤900 seconds**, unless stricter child-spec rules apply.

Violation → `E_CONSENT_EXPIRED` or `E_CONSENT_INVALID`.

---

## **5.4 Exporter Binding (Session Binding)**

ConsentProof MUST bind to the active transport session:

```text
ConsentProof.exporter_hash == session.exporter_hash
```

Mismatch →
`valid_consent = false` → `E_CONSENT_EXPORTER_MISMATCH`.

This ensures consent cannot be replayed across TLS/STP sessions.

---

## **5.5 Role Binding**

If used (e.g., PQHD, PQAI):

* role MUST be canonical
* role MUST match expected action semantics

PQSF does not define roles; it validates bindings only.

---

## **5.6 Signature Requirements**

ConsentProof MUST be signed with ML-DSA-65.

Signature failure MUST cause fail-closed behaviour.

### Pseudocode (Informative) — Consent Validation

```pseudo
function pqsf_validate_consent(consent, exporter_hash, current_tick):
    // Canonical and signature check
    bytes = pqsf_canonical_encode({
        action:        consent.action,
        intent_hash:   consent.intent_hash,
        tick_issued:   consent.tick_issued,
        tick_expiry:   consent.tick_expiry,
        exporter_hash: consent.exporter_hash,
        role:          consent.role,
        consent_id:    consent.consent_id
    })

    if not ml_dsa_65_verify(consent_pk, bytes, consent.signature_pq):
        return (false, "E_CONSENT_SIGNATURE_INVALID")

    // Exporter binding
    if consent.exporter_hash != exporter_hash:
        return (false, "E_CONSENT_EXPORTER_MISMATCH")

    // Tick window
    if consent.tick_issued > current_tick.t:
        return (false, "E_CONSENT_INVALID")

    if current_tick.t > consent.tick_expiry:
        return (false, "E_CONSENT_EXPIRED")

    return (true, null)
```

---

# **6. POLICY AUTHORITY — POLICY ENFORCER (NORMATIVE)**

The Policy Enforcer enforces policy in a deterministic, tick-bound, canonical manner.
It is the **policy enforcement layer** for wallets, AI systems, and sensitive automation.

---

## **6.1 Structure**

```
PolicyEnforcerPolicy = {
  "policy_id": tstr,
  "policy_hash": bstr,
  "allowlist": [* tstr],
  "denylist":  [* tstr],
  "thresholds": { * tstr => uint },
  "limits":     { * tstr => uint },
  "time_rules": {
      "min_delay": uint,
      "max_window": uint
  },
  "constraints": { * tstr => any }
}
```

---

## **6.2 Canonicalisation**

Policy MUST be canonicalised using deterministic CBOR/JCS.
The policy_hash MUST equal:

```
SHAKE256-256(canonical(policy))
```

Mismatch → `E_POLICY_HASH_MISMATCH`.

### Pseudocode (Informative) — Policy Hashing

```pseudo
function pqsf_compute_policy_hash(policy_without_hash):
    bytes = pqsf_canonical_encode(policy_without_hash)
    return shake256_256(bytes)

function pqsf_verify_policy_hash(policy):
    tmp = clone(policy)
    tmp.policy_hash = null
    expected = pqsf_compute_policy_hash(tmp)
    return (policy.policy_hash == expected)
```

---

## **6.3 Required Predicates**

PQSF defines the seven mandatory predicates:

```
valid_tick
AND valid_consent
AND valid_policy
AND valid_runtime       // PQVL
AND valid_quorum        // child-spec defined
AND valid_ledger
AND valid_structure     // schema/encoding validity
```

Child specs MAY add predicates but MUST NOT remove any.

### Pseudocode (Informative) — Combined Predicate Evaluation

```pseudo
function pqsf_evaluate_all_predicates(ctx):
    if not ctx.valid_tick:
        return false

    if not ctx.valid_consent:
        return false

    if not ctx.valid_policy:
        return false

    if ctx.requires_runtime and not ctx.valid_runtime:
        return false

    if ctx.requires_quorum and not ctx.valid_quorum:
        return false

    if not ctx.valid_ledger:
        return false

    if not pqsf_valid_structure(ctx):
        return false

    return true
```

---

## **6.4 Tick-Dependent Rules**

Policy Enforcer MUST reject if:

* minimum delay not satisfied
* window exceeded
* tick expired
* monotonicity violated

---

## **6.5 Allowlist / Denylist Enforcement**

If `allowlist` is **not** empty:

```
destination ∉ allowlist → valid_policy = false
```

If destination appears in `denylist`:

```
valid_policy = false
```

Destination semantics belong to child specifications.

---

## **6.6 Threshold & Constraint Rules**

Thresholds and limits are deployment- or child-spec-defined.
PQSF defines deterministic enforcement only.

Examples:

* spending thresholds (PQHD)
* API call limits
* content-type restrictions

Failure MUST yield `E_POLICY_THRESHOLD_FAILED` or `E_POLICY_CONSTRAINT_FAILED`.

---

## **6.7 Signature Bundles**

Policy bundles MUST be:

* canonical
* tick-bound
* signed with ML-DSA-65

Policy rotation MUST appear in the ledger.

### Pseudocode (Informative) — Policy Evaluation Function

```pseudo
function pqsf_evaluate_policy(policy, ctx):
    if not pqsf_verify_policy_hash(policy):
        return (false, "E_POLICY_HASH_MISMATCH")

    // allowlist / denylist
    if len(policy.allowlist) > 0 and ctx.destination not in policy.allowlist:
        return (false, "E_POLICY_CONSTRAINT_FAILED")

    if ctx.destination in policy.denylist:
        return (false, "E_POLICY_CONSTRAINT_FAILED")

    // time rules
    delta = ctx.current_tick.t - ctx.request_created_tick.t
    if delta < policy.time_rules.min_delay:
        return (false, "E_POLICY_CONSTRAINT_FAILED")

    if delta > policy.time_rules.max_window:
        return (false, "E_POLICY_CONSTRAINT_FAILED")

    // thresholds / limits
    if not thresholds_satisfied(policy.thresholds, ctx.metrics):
        return (false, "E_POLICY_THRESHOLD_FAILED")

    if not limits_satisfied(policy.limits, ctx.metrics):
        return (false, "E_POLICY_CONSTRAINT_FAILED")

    return (true, null)
```

---

# **7. TRANSPORT SECURITY — TLSE-EMP AND STP (NORMATIVE)**

PQSF defines two transport models that bind session identity, runtime state, consent, and tick validation into one deterministic structure:

1. **TLSE-EMP** — a deterministic profile of TLS 1.3
2. **STP** — a DNS-free, privacy-preserving transport for sovereign and offline-capable deployments

Both transports MUST provide:

* exporter-bound identity
* deterministic framing
* downgrade resistance
* replay resistance
* canonical session binding

---

## **7.1 TLSE-EMP (Deterministic TLS)**

TLSE-EMP MUST be supported by PQSF implementations.
It provides:

* hybrid post-quantum + classical key agreement
* deterministic transcript hashing
* exporter-bound session identity
* in-band replay protection
* downgrade attack prevention

PQSF does **not** modify TLS wire format; TLSE-EMP is a profile applied to existing TLS libraries.

---

## **7.2 Exporter Hash Derivation (NORMATIVE)**

PQSF uses the TLS 1.3 exporter defined in RFC 8446 to produce an exporter-bound session identity.

The exporter MUST be invoked as:

```
exporter_hash = TLS-Exporter("PQSF-EXPORTER", context, 32)
```

Where:

* label = `"PQSF-EXPORTER"`
* output length = **32 bytes**
* `context` = canonical CBOR or JCS JSON of:

```
{
  "pqsf_version": "1.0.0",
  "session_id": tstr
}
```

**Rules:**

* session_id MUST be identical on both endpoints
* canonical encoding MUST be identical across implementations
* exporter_hash MUST be embedded in all session-bound PQSF objects
* mismatch MUST cause fail-closed (`E_EXPORTER_MISMATCH`)

### Pseudocode (Informative) — Exporter Hash Helper

```pseudo
function pqsf_derive_exporter_hash(tls_session, session_id):
    ctx = {
        pqsf_version: "1.0.0",
        session_id:   session_id
    }
    ctx_bytes = pqsf_canonical_encode(ctx)
    return tls_exporter(tls_session, "PQSF-EXPORTER", ctx_bytes, 32)

function pqsf_check_exporter_binding(object_exporter_hash, session_exporter_hash):
    return (object_exporter_hash == session_exporter_hash)
```

---

## **7.3 Downgrade Resistance**

If negotiation results in classical-only cipher suites:

* PQSF MUST fail-closed **unless** the deployment explicitly allows transitional compatibility,
* and child specifications permit classical fallback.

This ensures PQSF-bound applications cannot silently downgrade.

---

## **7.4 STP — Sovereign Transport Protocol**

STP is a minimal, canonical, deterministic transport for offline/sovereign environments.

STP MUST support:

* operation without DNS
* exporter-bound replay protection
* deterministic framing rules
* minimal external dependency
* single-round-trip establishment

STP MUST be used in:

* privacy/stealth-granularity modes
* sovereign deployments
* offline-to-online transitions
* PQAI, PQHD, PQVL transport layers

**Normative STP examples** appear in **Annex C.10**.

---

## **7.5 Transport Fail-Closed**

Transport MUST fail-closed if:

* exporter mismatch
* tick invalid or stale
* canonicalisation fails
* downgrade detected
* replay detected

PQSF MUST NOT accept any session-bound object unless the transport session is fully validated.

---

# **7.6 Encrypted-Before-Transport (EBT) Requirement (NORMATIVE)**

The PQ Ecosystem follows a universal **Encrypted-Before-Transport (EBT)** rule:
all sensitive artefacts MUST be encrypted locally **before** they are transmitted over any transport channel, including TLSE-EMP, STP, offline transfer media, or cloud-storage endpoints. This rule ensures that hosts, intermediaries, mirrors, coordinators, cloud services, and network operators are cryptographically unable to inspect plaintext content.

EBT is a foundational privacy and sovereignty guarantee across the ecosystem.

## **7.6.1 Scope of EBT**

The following PQSF-bound artefacts MUST be encrypted locally before transmission:

1. **Secure AI Prompting payloads**
   (PQAI SafePrompt, prompt content, model-bound data)
   Encryption MUST use ML-KEM-1024 or AES-256-GCM/XChaCha20-Poly1305 under a key derived via PQSF deterministic derivation (§3.2).

2. **KeyMail messages**
   (Annex L in PQHD; §9 in PQSF)
   All KeyMailEnvelope content MUST be AEAD-encrypted prior to OOB transport.

3. **Recovery Capsules**
   (PQSF Annex D; PQHD §10)
   Capsule `encrypted_material` MUST be encrypted with ML-KEM-1024 using deterministic, canonical inputs. Capsules MUST never be transmitted in plaintext.

4. **Continuity Capsules / Ledger Bundles**
   (PQSF §8, Annex D)
   Exported ledger bundles MUST be encrypted locally before upload, cloud storage, or cross-device transport.

5. **Cloud-stored application artefacts**
   Any PQSF-compliant system using cloud transport or cloud storage MUST encrypt artefacts locally before uploading. Cloud hosts MUST NOT have access to any unencrypted PQSF, PQHD, PQAI, or PQVL state.

6. **Identity, KYC, and delegated credentials**
   (PQHD Annexes N–Q, PQAI §7)
   All credential objects MUST be encrypted before transport or remote storage.

7. **STP sovereign transport payloads**
   (PQSF §7.4)
   STP frames MUST NOT contain plaintext sensitive content. Payloads MUST be encrypted prior to STP encapsulation.

8. **Export/Import flows**
   (PQHD Secure Import, PQSF Annexes)
   All migration envelopes, authority objects, or exported secrets MUST be encrypted at the application boundary, before transmission.

## **7.6.2 Encryption Requirements**

All EBT encryption MUST satisfy:

* **Canonical input encoding**:
  Inputs to encryption MUST be canonicalised using deterministic CBOR or JCS JSON.

* **Post-quantum authenticated encryption**:
  Implementations MUST use one of:
  – **ML-KEM-1024** (for hybrid or KEM-sealed objects)
  – **AES-256-GCM** or **XChaCha20-Poly1305** (for symmetric AEAD)
  Keys MUST be derived using §3.2 (cSHAKE256 domain-separated deterministic derivation).

* **Exporter-bound semantics**:
  When EBT-protected items are sent during PQSF sessions, their encryption contexts MUST bind to the active exporter_hash to prevent cross-session replay or misuse.

* **Tick-bound validity**:
  EBT-protected items MUST embed or accompany a fresh EpochTick (§4).
  Stale ticks MUST render the decrypted content invalid.

### Pseudocode (Informative) — EBT Encryption Helper

```pseudo
function pqsf_encrypt_before_transport(label, plaintext_obj, ctx):
    // 1. Canonicalise inputs
    payload_bytes = pqsf_canonical_encode(plaintext_obj)

    // 2. Derive key via cSHAKE256
    key_context = {
        label:          label,
        exporter_hash:  ctx.exporter_hash,
        tick:           ctx.tick.t
    }
    key = pqsf_derive_secret(ctx.root_seed, "EBT-" + label, key_context)

    // 3. AEAD encrypt
    nonce = random_nonce()  // or deterministic per deployment rules
    aad   = pqsf_canonical_encode({ tick: ctx.tick, label: label })
    ciphertext, tag = aead_encrypt(key, nonce, payload_bytes, aad)

    return {
        ciphertext: ciphertext,
        nonce:      nonce,
        tag:        tag,
        tick:       ctx.tick
    }
```

## **7.6.3 Host Visibility Requirements**

Under EBT:

* Cloud hosts, coordinators, mirrors, transport intermediaries, and service operators MUST NOT be able to view plaintext PQSF artefacts.
* Only canonical ciphertext MUST appear in storage environments outside the user’s device.
* Hosts MAY observe encrypted binary blobs; they MUST NOT observe:
  – plaintext prompts,
  – credentials,
  – capsules,
  – PSBT-related data,
  – ledger bundles,
  – recovery material,
  – delegated identity/payment objects.

This ensures cloud-assisted workflows remain **sovereign**, **private**, and **independent of host trust**.

## **7.6.4 Error Handling (NORMATIVE)**

If an implementation attempts to transmit any EBT-scoped artefact in plaintext:

```
E_TRANSPORT_UNENCRYPTED
```

MUST be raised.

Any such event MUST:

* fail-closed the operation,
* freeze all dependent signing or AI actions,
* invalidate the current session,
* require a fresh tick,
* require PQVL re-attestation (for PQHD/PQAI consumers).

### Pseudocode (Informative) — EBT Enforcement

```pseudo
function pqsf_send_sensitive_artefact(obj, ctx):
    if obj.is_plaintext:
        return error("E_TRANSPORT_UNENCRYPTED")

    if not obj.is_ebt_encrypted:
        return error("E_TRANSPORT_UNENCRYPTED")

    // only now may transport send ciphertext
    ctx.session.send(obj.ciphertext)
    return ok()
```

## **7.6.5 Interaction With TLSE-EMP**

TLSE-EMP provides deterministic transport security (§7.1).
EBT is **additional** and MUST NOT be replaced by TLSE-EMP encryption.

* TLSE-EMP protects sessions.
* EBT protects artefacts **before** sessions exist.

Both protections MUST be applied.

## **7.6.6 Offline, Air-Gapped & Stealth Mode**

EBT MUST apply identically in:

* offline environments,
* QR/USB/serial transfer workflows,
* STP-only Stealth Mode.

In all cases:

* encrypted payloads MUST be used,
* plaintext handling MUST NOT occur at any transport boundary.

---

# **8. LEDGER (NORMATIVE)**

PQSF defines a deterministic append-only ledger for recording critical security events.
This ledger supports:

* policy integrity
* time-bound signing flows
* runtime verification
* consent events
* profile changes
* delegated identity/payment events
* erasure proofs

The ledger MUST be consistent across implementations.

---

## **8.1 Ledger Entry Structure**

```
LedgerEntry = {
  "event": tstr,
  "tick": uint,
  "payload": { * tstr => any },
  "signature_pq": bstr,
  "prev_hash": bstr
}
```

### Requirements:

* MUST be canonicalised
* MUST be hashed deterministically
* MUST chain to previous entry
* MUST be signed with ML-DSA-65
* MUST use SHAKE256-256

### Pseudocode (Informative) — Ledger Entry Builder

```pseudo
function pqsf_build_ledger_entry(event, tick, payload, prev_hash):
    entry = {
        event:        event,
        tick:         tick.t,
        payload:      payload,
        prev_hash:    prev_hash
    }
    entry_bytes = pqsf_canonical_encode(entry)
    entry.signature_pq = ml_dsa_65_sign(ledger_sk, entry_bytes)
    return entry
```

---

## **8.2 Merkle Construction**

PQSF uses a deterministic Merkle tree:

* leaf prefix = `0x00`
* internal node prefix = `0x01`
* hash function = SHAKE256-256

Leaf construction:

```
leaf = SHAKE256-256(0x00 || canonical(entry))
```

Node construction:

```
node = SHAKE256-256(0x01 || left || right)
```

The Merkle root MUST be identical across implementations for identical sequences.

### Pseudocode (Informative) — Merkle Root Calculation

```pseudo
function pqsf_merkle_root(entries):
    if len(entries) == 0:
        return zero32()

    leaves = []
    for entry in entries:
        bytes  = pqsf_canonical_encode(entry)
        leaf   = shake256_256(0x00 || bytes)
        leaves.append(leaf)

    nodes = leaves
    while len(nodes) > 1:
        next_level = []
        for i in range(0, len(nodes), 2):
            left = nodes[i]
            if i + 1 < len(nodes):
                right = nodes[i + 1]
            else:
                right = left  // duplicate last leaf if odd length
            node = shake256_256(0x01 || left || right)
            next_level.append(node)
        nodes = next_level

    return nodes[0]
```

---

## **8.3 Monotonic Ordering**

Ledger entries MUST be ordered strictly by tick:

```
tick_new > tick_previous
```

Violation MUST freeze the ledger until reconciliation.

---

## **8.4 Reconciliation**

Reconciliation MUST:

* accept the ledger with the highest tick
* verify `prev_hash` chain
* verify ML-DSA-65 signatures
* verify Merkle root
* reject divergent histories

If divergence cannot be reconciled automatically, the deployment MAY define canonicalisation rules outside PQSF.

### Pseudocode (Informative) — Ledger Append + Consistency

```pseudo
function pqsf_append_ledger_event(ledger, event, tick, payload):
    last_entry = ledger.last()
    if last_entry != null and tick.t <= last_entry.tick:
        return error("E_LEDGER_INVALID")

    prev_hash = last_entry.hash if last_entry != null else zero32()
    entry = pqsf_build_ledger_entry(event, tick, payload, prev_hash)

    ledger.entries.append(entry)
    ledger.root = pqsf_merkle_root(ledger.entries)
    return ok()
```

---

## **8.5 Required Ledger Events**

The following events MUST be supported:

* `consent_issued`
* `consent_expired`
* `policy_rotated`
* `runtime_attested`
* `runtime_drift_critical`
* `tick_validated`
* `profile_rotated`
* `transport_restarted`

Additional events are defined by annexes (payments, identity, erasure, etc.).

---

## **8.6 Fail-Closed Ledger Behaviour**

If any ledger entry is invalid:

* ledger MUST freeze
* system MUST fetch a fresh tick
* runtime MUST re-attest
* reconciliation MUST occur

Child specifications MUST NOT override this rule.

---

# **9. OUT-OF-BAND CHANNELS — KEYMAIL (OPTIONAL, NORMATIVE)**

KeyMail is an optional PQSF extension for high-risk, out-of-band (OOB) confirmations.
It is not required for wallets or PQHD but MAY be enabled by policy.

KeyMail provides:

* independent confirmation channel
* resistance against UI spoofing
* additional control for high-value or sensitive actions
* support for delegated authority flows

If KeyMail is disabled, PQSF implementations MAY omit all KeyMail flows.

---

## **9.1 KeyMail Overview (Optional)**

KeyMail MAY be enabled for:

* Secure Import
* delegated-key changes
* recovery activation
* high-value actions
* identity delegation
* payment delegation

Deployments choose whether to enable or disable KeyMail.

---

## **9.2 KeyMail Keypairs and Revocation**

KeyMail uses ML-DSA-65 signatures.
KeyMail keypairs MUST support:

* rotation under local implementation policy
* deterministic derivation (cSHAKE256)
* tick-bound activation
* revocation with DEAD-KEY semantics
* ledger logging
* immediate invalidation of pending KeyMail confirmations upon revocation

A revoked KeyMail key MUST NOT validate future confirmations.

---

## **9.3 Message Structure and Binding**

A KeyMail confirmation MUST include:

```
KeyMailMessage = {
  "km_id": tstr,
  "consent_ref": tstr,
  "action": tstr,
  "content_hash": bstr,
  "tick": EpochTick,
  "exporter_hash": bstr,
  "signature_pq": bstr
}
```

Requirements:

* MUST be AEAD-encrypted (ML-KEM-1024 or AES-256-GCM-SIV)
* MUST be canonicalised
* MUST bind to active exporter_hash
* MUST be tick-valid
* MUST be signed with ML-DSA-65
* MUST be logged for high-risk operations

### Pseudocode (Informative) — KeyMail Creation and Validation

```pseudo
function pqsf_build_keymail_message(consent_ref, action, content_obj, tick, exporter_hash):
    content_hash = pqsf_hash_object(content_obj)
    msg = {
        km_id:         generate_uuid(),
        consent_ref:   consent_ref,
        action:        action,
        content_hash:  content_hash,
        tick:          tick,
        exporter_hash: exporter_hash
    }
    msg_bytes = pqsf_canonical_encode(msg)
    msg.signature_pq = ml_dsa_65_sign(keymail_sk, msg_bytes)
    return msg

function pqsf_validate_keymail_message(msg, exporter_hash, current_tick):
    if msg.exporter_hash != exporter_hash:
        return (false, "E_EXPORTER_MISMATCH")

    (valid_tick, err) = pqsf_validate_tick(msg.tick, pinned_profile_ref, null)
    if not valid_tick:
        return (false, err)

    bytes = pqsf_canonical_encode({
        km_id:         msg.km_id,
        consent_ref:   msg.consent_ref,
        action:        msg.action,
        content_hash:  msg.content_hash,
        tick:          msg.tick,
        exporter_hash: msg.exporter_hash
    })
    if not ml_dsa_65_verify(keymail_pk, bytes, msg.signature_pq):
        return (false, "E_SIGNATURE_INVALID")

    return (true, null)
```

---

## **9.4 Security Rationale (Informative)**

KeyMail produces a **second independent confirmation path**.
An attacker must compromise both the main session and KeyMail to authorise sensitive actions.

It reduces risk from:

* phishing
* UI-swapping
* transport injection
* coordinator manipulation
* social-engineering attacks

Optional, but highly protective when enabled.

---

# **10. COMPLIANCE (NORMATIVE)**

Compliance profiles define minimum capabilities required for interoperability across PQSF implementations.
All PQSF-conformant systems MUST explicitly declare their compliance level.

---

## **10.1 MVP Compliance**

The **MVP** profile defines the smallest viable PQSF implementation.

MVP MUST implement:

* EpochTick validation (§4)
* ConsentProof (§5)
* exporter-bound TLSE-EMP (§7)
* deterministic canonical encoding (§3.5)
* SHAKE256-256 hashing
* cSHAKE256 deterministic key derivation (§3.2)
* minimal ledger (single-chain mode)

### MVP Ledger Requirements:

* append-only
* strictly monotonic tick
* ML-DSA-65 signatures on entries

### Events required under MVP:

* `tick_validated`
* `consent_issued`
* `consent_expired`
* `policy_rotated`
* `transport_restarted`

---

## **10.2 FULL Compliance**

FULL includes **all MVP requirements**, plus:

* full Policy Enforcer evaluation (§6)
* Sovereign Transport Protocol (STP) (§7.4)
* deterministic Merkle ledger (§8)
* ledger reconciliation rules
* policy bundles and tick-bound policy changes
* support for recovery capsule format (Annex D)

FULL is suitable for multi-device and multi-policy deployments.

---

## **10.3 EXTENDED Compliance**

EXTENDED includes all FULL requirements, and adds integration with:

* PQHD (wallet custody)
* PQAI (AI model integrity flows)
* PQVL (runtime verification)
* Epoch Clock v2 lineage rules (§4.2)

EXTENDED is recommended for:

* high-assurance systems
* regulated deployments
* sovereign/offline installations
* environments requiring runtime-state guarantees

---

## **10.4 Compliance Manifest**

A PQSF implementation SHOULD distribute a signed manifest:

```
{
  "implementation": tstr,
  "version": tstr,
  "compliance": "MVP" | "FULL" | "EXTENDED",
  "tick": uint,
  "signature_pq": bstr
}
```

Signature MUST be ML-DSA-65 and verifiable based on public key material distributed by the implementation.

### Pseudocode (Informative) — Compliance Manifest Issuance

```pseudo
function pqsf_build_compliance_manifest(impl_name, version, level, tick):
    manifest = {
        implementation: impl_name,
        version:        version,
        compliance:     level,
        tick:           tick.t
    }
    bytes = pqsf_canonical_encode(manifest)
    manifest.signature_pq = ml_dsa_65_sign(manifest_sk, bytes)
    return manifest
```

---

# **11. ERROR CODES (NORMATIVE)**

PQSF defines standardised error codes for consistent implementation behaviour.

---

## **11.1 Tick / Time Errors**

```
E_TICK_INVALID
E_TICK_STALE
E_TICK_ROLLBACK
E_TICK_PROFILE_MISMATCH
```

---

## **11.2 Consent Errors**

```
E_CONSENT_INVALID
E_CONSENT_EXPIRED
E_CONSENT_EXPORTER_MISMATCH
E_CONSENT_SIGNATURE_INVALID
```

---

## **11.3 Policy Errors**

```
E_POLICY_HASH_MISMATCH
E_POLICY_CONSTRAINT_FAILED
E_POLICY_THRESHOLD_FAILED
```

---

## **11.4 Transport Errors**

```
E_EXPORTER_MISMATCH
E_TRANSPORT_DOWNGRADE
E_TRANSPORT_REPLAY
E_TRANSPORT_CANONICAL_FAIL
E_TRANSPORT_UNENCRYPTED
```

---

## **11.5 Ledger Errors**

```
E_LEDGER_INVALID
E_LEDGER_DIVERGED
E_LEDGER_FROZEN
E_LEDGER_SIGNATURE_INVALID
```

---

## **11.6 Integration Errors**

Errors surfaced from child specs and integrated modules:

```
E_RUNTIME_INVALID            // PQVL
E_PROFILE_INVALID            // PQAI ModelProfile
E_FINGERPRINT_INVALID        // PQAI
E_DEVICE_INVALID             // PQHD with PQVL
```

### Pseudocode (Informative) — Central Error Mapper

```pseudo
function pqsf_error_from_flags(flags):
    if flags.tick_profile_mismatch:
        return "E_TICK_PROFILE_MISMATCH"
    if flags.tick_rollback:
        return "E_TICK_ROLLBACK"
    if flags.tick_stale:
        return "E_TICK_STALE"
    if flags.tick_invalid:
        return "E_TICK_INVALID"

    if flags.consent_sig_invalid:
        return "E_CONSENT_SIGNATURE_INVALID"
    if flags.consent_exporter_mismatch:
        return "E_CONSENT_EXPORTER_MISMATCH"
    if flags.consent_expired:
        return "E_CONSENT_EXPIRED"

    if flags.policy_hash_mismatch:
        return "E_POLICY_HASH_MISMATCH"
    if flags.policy_threshold_failed:
        return "E_POLICY_THRESHOLD_FAILED"
    if flags.policy_constraint_failed:
        return "E_POLICY_CONSTRAINT_FAILED"

    if flags.exporter_mismatch:
        return "E_EXPORTER_MISMATCH"
    if flags.transport_downgrade:
        return "E_TRANSPORT_DOWNGRADE"
    if flags.transport_replay:
        return "E_TRANSPORT_REPLAY"
    if flags.transport_noncanonical:
        return "E_TRANSPORT_CANONICAL_FAIL"
    if flags.transport_unencrypted:
        return "E_TRANSPORT_UNENCRYPTED"

    if flags.ledger_invalid:
        return "E_LEDGER_INVALID"
    if flags.ledger_diverged:
        return "E_LEDGER_DIVERGED"
    if flags.ledger_frozen:
        return "E_LEDGER_FROZEN"
    if flags.ledger_sig_invalid:
        return "E_LEDGER_SIGNATURE_INVALID"

    if flags.runtime_invalid:
        return "E_RUNTIME_INVALID"

    return "E_RUNTIME_INVALID"  // safe default
```

---

# **12. SECURITY CONSIDERATIONS (INFORMATIVE)**

## **12.1 Quantum Safety**

PQSF relies exclusively on PQ-safe primitives:

* ML-DSA-65 signatures
* ML-KEM (child-spec encryption)
* SHAKE256-256 hashing
* cSHAKE256 derivation

---

## **12.2 Determinism**

Canonical encoding eliminates:

* malleability
* cross-implementation ambiguity
* divergent hashing
* signature inconsistencies

Determinism ensures identical verification results across all compliant implementations.

---

## **12.3 Fail-Closed Model**

Any failure of:

* tick
* consent
* policy
* ledger
* runtime integrity

MUST cause dependent systems to fail-closed.

No silent fallback is permitted.

---

## **12.4 No Trust in OS or Hardware**

PQSF assumes the operating system and hardware environment are untrusted.

Runtime integrity is validated via PQVL (if used).

---

## **12.5 Ledger Safety**

Merkle ledger rules prevent tampering, rollback, and reordering of event history.

---

# **13. IMPLEMENTATION NOTES (INFORMATIVE)**

## **13.1 Canonical Encoding Strategies**

Recommended:

* JCS JSON for transport
* deterministic CBOR for storage

Both MUST adhere to §3.5.

---

## **13.2 Transport Layers**

TLSE-EMP SHOULD be built atop an existing PQ-capable TLS library.
STP SHOULD be used for sovereign/offline modes.

---

## **13.3 Tick Sources**

Systems MUST pin a single Epoch Clock profile.
Ticks MUST come from trusted mirror sources or local indexing of the on-chain inscription tree.

---

## **13.4 Multi-Device Environments**

PQSF is multi-device-safe if:

* exporter binding is enforced
* monotonic ticks maintained
* ledger reconciliation implemented

---

# **14. BACKWARDS COMPATIBILITY (INFORMATIVE)**

PQSF is designed for incremental adoption:

* no OS changes
* no hardware enclaves
* no blockchain consensus changes
* classical TLS fallback permitted only when explicitly allowed

PQSF deployments MAY function:

* fully offline
* sovereign or on-premise
* hybrid cloud environments
* consumer/mobile platforms

---


# **ANNEX A — MVP COMPLIANCE PROFILE (NORMATIVE)**

The MVP profile defines the absolute minimum requirements for a PQSF-compliant implementation.
Any implementation claiming MVP compliance MUST implement all elements of this annex.

---

## **A.1 Required Capabilities**

### **A.1.1 Temporal Authority**

MVP implementations MUST:

* validate EpochTicks according to Section 4
* enforce tick freshness (≤900 seconds)
* enforce monotonicity
* validate profile_ref against the pinned Epoch Clock profile

---

### **A.1.2 Intent Authority**

MVP implementations MUST:

* implement the ConsentProof structure (Section 5)
* canonicalise ConsentProof using deterministic CBOR or JCS JSON
* validate ML-DSA-65 signature
* enforce ConsentProof exporter_hash binding
* enforce tick_issued / tick_expiry bounds

---

### **A.1.3 Transport Requirements**

MVP implementations MUST:

* implement TLSE-EMP as defined in Section 7
* enforce exporter binding
* detect downgrade attempts
* enforce canonical framing rules

---

### **A.1.4 Deterministic Encoding**

MVP implementations MUST:

* support deterministic CBOR or JCS JSON for all PQSF objects
* reject non-canonical encodings

---

### **A.1.5 Key Lifecycle**

MVP implementations MUST:

* support cSHAKE256 deterministic derivation
* use domain separation strings for each key category

---

### **A.1.6 Ledger (Minimal Mode)**

MVP ledger MUST:

* support a minimal single-chain ledger (no reconciliation)
* append events strictly monotonically by tick
* sign ledger entries with ML-DSA-65

---

## **A.2 Minimal Event Requirements**

The following events MUST be supported:

* `tick_validated`
* `consent_issued`
* `consent_expired`
* `policy_rotated`
* `transport_restarted`

Child specifications MAY define additional events.

---

## **A.3 Interoperability Requirements**

MVP implementations MUST:

* accept canonical CBOR/JCS from other MVP-compatible implementations
* enforce identical tick rules
* treat exporter mismatch as fatal
* fail closed on any signature failure

---

# **ANNEX B — FULL & EXTENDED COMPLIANCE PROFILES (NORMATIVE)**

Full and Extended profiles define additional PQSF capabilities required for multi-device, multi-policy, or high-assurance environments.

---

## **B.1 FULL Compliance**

FULL implementations MUST satisfy **all MVP requirements** plus the following.

---

### **B.1.1 Policy Authority**

FULL implementations MUST:

* implement full Policy Enforcer semantics
* enforce allowlist/denylist
* enforce thresholds, limits, and min/max time windows
* compute and validate policy_hash

---

### **B.1.2 Ledger Requirements**

FULL implementations MUST:

* support deterministic Merkle ledger (Section 8)
* support reconciliation rules
* validate prev_hash chain
* freeze the ledger on divergence

---

### **B.1.3 Additional Transport Requirements**

FULL implementations MUST:

* support STP (Section 7.4)
* implement replay detection for STP frames
* validate canonical STP frames

---

### **B.1.4 Profile Updates**

FULL implementations MUST:

* support profile selection and validation rules from Section 4.2
* commit `policy_rotated` and `profile_rotated` events
* enforce tick monotonicity when recording these events

---

## **B.2 EXTENDED Compliance**

EXTENDED implementations MUST satisfy **all FULL** requirements, plus stable integration with the following child specifications:

---

### **B.2.1 PQHD — Wallet Custody (see PQHD 1.3 and 5–9)**

PQHD defines the deterministic custody model consumed by PQSF’s predicates.  
EXTENDED-profile PQSF implementations MUST support PQHD-level custody rules, including deterministic multisig, tick-bound signing windows, PSBT canonicalisation, policy enforcement, runtime-integrity gating, ledger anchoring, and threshold-based recovery workflows.

---

### **B.2.2 PQAI — AI Behaviour Verification (see PQAI 1.3–1.6 and 5–11)**

PQAI defines deterministic behavioural-verification rules for high-assurance AI operation.  
EXTENDED-profile PQSF implementations MUST consume PQAI alignment fingerprints, drift-classification results, SafePrompt validation, model-profile checks, and tick-fresh alignment enforcement to ensure that AI-mediated actions occur only under verified, deterministic behavioural conditions.

---

### **B.2.3 PQVL — Runtime Verification (see PQVL 5–7)**

PQVL provides deterministic runtime-integrity verification consumed by PQSF transport, consent, policy, and ledger predicates.  
EXTENDED-profile PQSF implementations MUST evaluate drift_state, mandatory probe sets, attestation freshness, canonical AttestationEnvelope structure, and exporter-bound transport semantics before considering any runtime-dependent predicate valid.

---

### **B.2.4 Epoch Clock v2 (see Epoch Clock 1.7–1.10)**

Epoch Clock v2 defines the canonical, Bitcoin-anchored temporal authority consumed by PQSF.
EXTENDED-profile PQSF implementations MUST validate ticks under the active profile lineage, enforce monotonicity and freshness, and apply deterministic, fail-closed tick handling across consent windows, policy windows, runtime verification, and ledger events.

---

## **B.3 Compliance Manifest Requirements**

```
ComplianceManifest = {
  implementation: tstr,
  version: tstr,
  compliance: "MVP" / "FULL" / "EXTENDED",
  tick: uint,
  signature_pq: bstr
}
```

Manifest MUST be signed using ML-DSA-65 under the implementation’s identity.

---

# **ANNEX C — WORKFLOW EXAMPLES (INFORMATIVE)**

This annex provides example flows to help developers integrate PQSF correctly.
These examples are non-normative.

---

## **C.1 Tick Validation Flow**

1. Fetch tick from Epoch Clock source (see Epoch Clock 4–5).
2. Validate ML-DSA signature.
3. Validate profile_ref.
4. Check monotonicity.
5. Check freshness (≤900 seconds).
6. Mark `valid_tick = true`.
7. Append `tick_validated` to ledger.

---

## **C.2 ConsentProof Validation Flow**

1. Canonicalise ConsentProof.

2. Verify ML-DSA signature.

3. Validate exporter_hash = session exporter_hash.

4. Ensure:

   ```
   tick_issued ≤ current_tick ≤ tick_expiry
   ```

5. Mark `valid_consent = true`.

---

## **C.3 Policy Enforcer Evaluation Flow**

1. Validate policy_hash.
2. Evaluate allowlist/denylist.
3. Evaluate thresholds.
4. Evaluate min_delay & max_window.
5. If all pass → `valid_policy = true`.

---

## **C.4 TLSE-EMP Handshake Flow**

1. Initiate PQ-hybrid TLS handshake.
2. Verify deterministic transcript.
3. Derive exporter_hash.
4. Bind exporter_hash to ConsentProof and other PQSF structures.
5. Begin secure session.

---

## **C.5 STP Sovereign Mode Flow**

1. Client sends STP-INIT frame.
2. Server responds with STP-ACCEPT.
3. Both sides derive exporter_hash.
4. PQSF-bound operations begin.
5. Replay attempts rejected.

---

## **C.6 Ledger Append Flow**

1. Construct new LedgerEntry.

2. Compute Merkle leaf hash:

   ```
   leaf = SHAKE256-256(0x00 || canonical(entry))
   ```

3. Extend Merkle tree.

4. Compute root.

5. Validate prev_hash chain.

6. Sign ledger entry.

7. Commit entry.

---

## **C.7 EpochTick Validation Pseudocode**

```pseudo
function validate_epoch_tick(tick, pinned_profile_ref, last_tick, now_unix):
    if tick.profile_ref != pinned_profile_ref:
        return (false, E_TICK_PROFILE_MISMATCH)

    if !verify_ml_dsa_65(tick):
        return (false, E_TICK_INVALID)

    if last_tick != null and tick.t <= last_tick.t:
        return (false, E_TICK_ROLLBACK)

    if (now_unix - tick.t) > 900:
        return (false, E_TICK_STALE)

    return (true, null)
```

---

## **C.8 ConsentProof Validation Pseudocode**

```pseudo
function validate_consent(consent, session_exporter_hash, current_tick):

    canonical_bytes = canonicalise(consent)

    if !verify_ml_dsa_65(canonical_bytes, consent.signature_pq):
        return (false, E_CONSENT_SIGNATURE_INVALID)

    if consent.exporter_hash != session_exporter_hash:
        return (false, E_CONSENT_EXPORTER_MISMATCH)

    if consent.tick_issued > current_tick.t:
        return (false, E_CONSENT_INVALID)

    if current_tick.t > consent.tick_expiry:
        return (false, E_CONSENT_EXPIRED)

    return (true, null)
```

---

## **C.9 Policy Enforcer Evaluation Pseudocode**

```pseudo
function evaluate_policy_enforcer(policy, context):

    canonical_policy = canonicalise(policy)
    expected_hash = SHAKE256_256(canonical_policy)

    if policy.policy_hash != expected_hash:
        return (false, E_POLICY_HASH_MISMATCH)

    if policy.allowlist is not empty:
        if context.destination not in policy.allowlist:
            return (false, E_POLICY_CONSTRAINT_FAILED)

    if context.destination in policy.denylist:
        return (false, E_POLICY_CONSTRAINT_FAILED)

    if (context.current_tick.t - context.created_tick.t) < policy.time_rules.min_delay:
        return (false, E_POLICY_CONSTRAINT_FAILED)

    if (context.current_tick.t - context.created_tick.t) > policy.time_rules.max_window:
        return (false, E_POLICY_CONSTRAINT_FAILED)

    if not thresholds_satisfied(policy.thresholds, context):
        return (false, E_POLICY_THRESHOLD_FAILED)

    if not limits_satisfied(policy.limits, context):
        return (false, E_POLICY_CONSTRAINT_FAILED)

    return (true, null)
```

---

## **C.10 STP Example Flows**

### **C.10.1 Basic STP Handshake**

**Client → Server:**

```text
STP-INIT {
  version: "1.0",
  session_id: "sess-1234",
  client_nonce: <random>,
  client_caps: ["pqsf", "epoch", "consent"]
}
```

**Server → Client:**

```text
STP-ACCEPT {
  version: "1.0",
  session_id: "sess-1234",
  server_nonce: <random>,
  server_caps: ["pqsf", "epoch"],
  exporter_hash: <derived as per §7.2>
}
```

### **C.10.2 Resumption**

Client:

```text
STP-RESUME {
  previous_session_id: "sess-1234",
  resume_token: <opaque>,
  client_nonce: <random>
}
```

Server either:

* resumes session and provides new exporter_hash, **or**
* rejects and demands new STP-INIT.

### **C.10.3 Replay Detection**

Replayed STP-INIT or STP-ACCEPT triggers:

```text
STP-ERROR {
  code: "E_TRANSPORT_REPLAY",
  session_id: "sess-1234"
}
```

PQSF MUST fail-closed.

---

# **ANNEX D — RECOVERY CAPSULES (NORMATIVE)**

Recovery capsules define deterministic, verifiable objects used for **disaster recovery**, **controlled restoration**, and **authorised rollback** under the constraints of PQSF security primitives.

PQSF itself does **not** define how these capsules are used at the application layer; PQHD and other child specifications specialise these structures.

---

## **D.1 Recovery Capsule Structure**

A RecoveryCapsule MUST have the following structure:

```text
RecoveryCapsule = {
  "capsule_id": tstr,
  "guardian_set": [* tstr],
  "threshold": uint,
  "encrypted_material": bstr,
  "capsule_hash": bstr,
  "creation_tick": uint,
  "valid_from_tick": uint,
  "recovery_delay": uint,
  "signature_pq": bstr
}
```

### Normative Requirements:

* `capsule_hash` MUST equal:

  ```text
  SHAKE256-256(canonical(capsule))
  ```

  where `canonical(capsule)` is computed over the capsule with `capsule_hash` and `signature_pq` omitted.

* Capsule MUST be ML-DSA-65 signed.

* Activation MUST NOT occur before `valid_from_tick`.

* Guardians MUST satisfy the threshold signature rules defined by the consuming specification.

* `encrypted_material` MUST be treated as opaque at the PQSF layer. It MAY contain:

  * backup secrets
  * encrypted account metadata
  * recovery key material
  * ledger anchor checkpoints

PQSF does **not** define the semantics of `encrypted_material`; consuming specs (for example PQHD) MUST define:

* the internal format of `encrypted_material`,
* the AEAD/KEM scheme used to protect it,
* who is allowed to decrypt it under which conditions.

---

## **D.2 Recovery Activation Flow**

A RecoveryCapsule is activated using the following flow:

1. Verify ML-DSA signature on the capsule.

2. Verify `capsule_hash` against the canonicalised capsule body.

3. Verify that guardian approvals meet the `threshold` requirement.

4. Validate that:

   ```text
   current_tick ≥ valid_from_tick
   ```

5. Validate that the required `recovery_delay` window has elapsed since `creation_tick` (as defined by the consuming spec).

6. Decrypt `encrypted_material` according to the child specification’s rules.

7. Commit a `recovery_activated` ledger entry that includes `capsule_id` and any relevant metadata.

PQSF requires that all recovery activation MUST be:

* tick-bound,
* canonical, and
* ledger-recorded.

PQSF does not define what systems do *after* recovery is activated. That is the responsibility of the consuming specification (for example, PQHD may restore a wallet state; an AI system might restore alignment profiles).

---

### **Pseudocode (Informative) — Recovery Capsule Activation**

```pseudo
function pqsf_activate_recovery_capsule(capsule, guardian_sigs, current_tick):
    // 1. Verify signature and hash

    // Build a hashable view of the capsule with capsule_hash + signature stripped
    tmp = clone(capsule)
    tmp.capsule_hash = null
    tmp.signature_pq = null

    expected_hash = pqsf_hash_object(tmp)
    if capsule.capsule_hash != expected_hash:
        return error("E_LEDGER_INVALID")

    // Verify ML-DSA-65 signature over canonical capsule body
    if not pqsf_verify_pq(recovery_pk, tmp, capsule.signature_pq):
        return error("E_SIGNATURE_INVALID")

    // 2. Guardian threshold (child spec defines guardian_set semantics)
    if not guardian_threshold_satisfied(
        capsule.guardian_set,
        guardian_sigs,
        capsule.threshold
    ):
        return error("E_POLICY_THRESHOLD_FAILED")

    // 3. Time checks (all tick values are EpochClock ticks, not local time)
    if current_tick.t < capsule.valid_from_tick:
        return error("E_POLICY_CONSTRAINT_FAILED")

    // Enforce a minimum delay from creation to activation
    if current_tick.t < capsule.creation_tick + capsule.recovery_delay:
        return error("E_POLICY_CONSTRAINT_FAILED")

    // 4. Decrypt encrypted_material (child spec defines exact scheme and contents)
    decrypted = decrypt_capsule_material(capsule.encrypted_material)

    // 5. Ledger record
    pqsf_append_ledger_event(
        global_ledger,
        "recovery_activated",
        current_tick,
        { capsule_id: capsule.capsule_id }
    )

    // 6. Hand decrypted payload back to consuming spec for domain-specific handling
    return decrypted
```

This pseudocode is **informative only**. It illustrates:

* how to canonicalise and hash the capsule body,
* how to validate ML-DSA-65 signatures,
* how to enforce guardian threshold and time-based constraints,
* how to record a `recovery_activated` ledger event.

Consuming specifications MUST define:

* the structure and meaning of `guardian_sigs`,
* the exact guardian threshold semantics,
* the AEAD/KEM mechanism used in `decrypt_capsule_material`,
* what is done with `decrypted` after activation.

---

# **ANNEX E — CDDL DEFINITIONS (INFORMATIVE)**

This annex provides machine-readable CDDL definitions for PQSF object types.
These definitions assist with automated validation, schema checking, and code generation.

Normative rules remain in the main specification.

---

## **E.1 EpochTick**

```cddl
EpochTick = {
  t: uint,
  profile_ref: tstr,
  alg: tstr,        ; MUST be "ML-DSA-65"
  sig: bstr
}
```

---

## **E.2 ConsentProof**

```cddl
ConsentProof = {
  action:        tstr,
  intent_hash:   bstr,
  tick_issued:   uint,
  tick_expiry:   uint,
  exporter_hash: bstr,
  role:          tstr / null,
  consent_id:    tstr,
  signature_pq:  bstr
}
```

---

## **E.3 PolicyEnforcerPolicy**

```cddl
PolicyEnforcerPolicy = {
  policy_id:   tstr,
  policy_hash: bstr,
  allowlist:   [* tstr],
  denylist:    [* tstr],
  thresholds:  { * tstr => uint },
  limits:      { * tstr => uint },
  time_rules: {
      min_delay:  uint,
      max_window: uint
  },
  constraints: { * tstr => any }
}
```

---

## **E.4 LedgerEntry**

```cddl
LedgerEntry = {
  event:         tstr,
  tick:          uint,
  payload:       { * tstr => any },
  signature_pq:  bstr,
  prev_hash:     bstr
}
```

All CDDL encodings MUST follow deterministic CBOR rules defined in RFC 8949 §4.2.

---

# **ANNEX F — REFERENCE TEST VECTORS (INFORMATIVE)**

This annex provides reference examples of canonical encoding, hashing, ledger entries, policy definitions, and other PQSF objects.
These vectors are **informative**, aiding implementation correctness.

---

# **F.1 ConsentProof — Canonical Encoding Example**

Diagnostic JSON:

```
{
  "action": "spend",
  "intent_hash": "0x112233...",
  "tick_issued": 1730000000,
  "tick_expiry": 1730000300,
  "exporter_hash": "0xaabbcc...",
  "role": "primary",
  "consent_id": "consent-001",
  "signature_pq": "<signature>"
}
```

Canonical deterministic CBOR (hex, abbreviated):

```
a8
  66 616374696f6e         ; "action"
  65 7370656e64           ; "spend"
  6b696e74656e745f68617368 43 112233...
  6b7469636b5f697373756564 1a 6728b580
  6b7469636b5f657870697279 1a 6728c3bc
  6d6578706f727465725f68617368 42 aabbcc...
  64 726f6c65 67 7072696d617279
  6a636f6e73656e745f6964 6b636f6e73656e742d303031
  6c7369676e61747572655f7071 58 40 <sig>
```

A compliant implementation MUST generate identical bytes for canonical CBOR.

---

# **F.2 cSHAKE256 Derivation Example**

Given:

* root_key:
  `0x9a7f3e2d1c8b6a5f4e3d2c1b0a9f8e7d6c5b4a3f2e1d0c9b8a7f6e5d4c3b2a1`

* domain: `"PQSF-Derive"`

* path (CBOR):
  `0x820102` (represents `[1, 2]`)

Derive:

```text
child_key = cSHAKE256(root_key, domain="PQSF-Derive", input=0x820102)
```

Expected output (truncated):

```
0x3f1a948b...
```

---

# **F.3 Merkle Ledger Example**

Given entries L1 and L2:

```
leaf1 = SHAKE256-256(0x00 || CBOR(L1))
leaf2 = SHAKE256-256(0x00 || CBOR(L2))
root  = SHAKE256-256(0x01 || leaf1 || leaf2)
```

Compliant implementations MUST produce identical outputs.

---

# **F.4 Tick Hash Example**

Canonical CBOR (abbreviated):

```
a4
  61 74 1a 6728b580
  6b70726f66696c655f726566 <profile_ref>
  63 616c67 6a4d4c2d4453412d3635
  63 736967 <sig>
```

Tick Hash:

```
SHAKE256-256(canonical_tick)
```

---

# **F.5 PolicyEnforcerPolicy — JSON Example**

```
{
  "policy_id": "policy-main-001",
  "policy_hash": "0xabc123...",
  "allowlist": ["merchant:001", "merchant:abc"],
  "denylist": [],
  "thresholds": { "daily_spend_minor": 1000000 },
  "limits": { "per_tx_minor": 250000 },
  "time_rules": { "min_delay": 60, "max_window": 3600 },
  "constraints": { "currency": "USD" }
}
```

`policy_hash` MUST match canonical SHAKE256-256 output.

---

# **F.6 LedgerEntry — JSON Example (`tick_validated`)**

```
{
  "event": "tick_validated",
  "tick": 1730000000,
  "payload": {
    "tick_hash": "0xdeadbeef...",
    "profile_ref": "epoch-clock-v2-mainnet"
  },
  "signature_pq": "<ml-dsa-signature>",
  "prev_hash": "0x00112233..."
}
```

---

# **F.7 PaymentIntent — JSON Example**

```
{
  "intent_id": "pi-1234",
  "merchant_id": "merchant-001",
  "amount_minor": 5000,
  "currency": "USD",
  "description": "Coffee",
  "mode": "CIB",
  "method": {
    "kind": "psp_element",
    "element_id": "elt-7890",
    "capability": ["card", "3ds"]
  },
  "tick": {
    "t": 1730000100,
    "profile_ref": "epoch-clock-v2-mainnet",
    "alg": "ML-DSA-65",
    "sig": "<tick-sig>"
  },
  "consent_hash": "0x445566...",
  "created_at": 1730000090,
  "expires_at": 1730000390,
  "policy_ref": "policy-main-001"
}
```

---

# **F.8 KYCCredential — JSON Example (Over 18)**

```
{
  "cred_id": "kyc-abc123",
  "issuer_id": "issuer:kyc-provider",
  "subject_id": "sub-7890",
  "attributes": {
    "over_18": true,
    "jurisdiction": "AU"
  },
  "issued_at": 1730000000,
  "expires_at": 1761536000,
  "tick": {
    "t": 1730000000,
    "profile_ref": "epoch-clock-v2-mainnet",
    "alg": "ML-DSA-65",
    "sig": "<tick>"
  },
  "issuer_sig_pq": "<issuer-ml-dsa-sig>"
}
```

---

# **F.9 LedgerEntry — `consent_issued`**

```
{
  "event": "consent_issued",
  "tick": 1730000100,
  "payload": {
    "consent_id": "consent-001",
    "action": "payment.authorise",
    "intent_hash": "0x445566..."
  },
  "signature_pq": "<ml-dsa-signature>",
  "prev_hash": "0xaaaabbbbcccc..."
}
```

---

# **F.10 LedgerEntry — `profile_rotated`**

```
{
  "event": "profile_rotated",
  "tick": 1731000000,
  "payload": {
    "old_profile": "epoch-clock-v2-mainnet",
    "new_profile": "epoch-clock-v2-mainnet-child-001"
  },
  "signature_pq": "<ml-dsa-signature>",
  "prev_hash": "0xbbbbccccdddd..."
}
```

---

# **F.11 LedgerEntry — `erasure_proof_issued`**

```
{
  "event": "erasure_proof_issued",
  "tick": 1730500000,
  "payload": {
    "erasure_id": "erase-1234",
    "subject_hash": "0x999988887777...",
    "erasure_mode": "third_party",
    "actor_id": "service:storage-1"
  },
  "signature_pq": "<ml-dsa-signature>",
  "prev_hash": "0xccccddddeeee..."
}
```

---

# **F.12 LedgerEntry — `runtime_drift_critical`**

```
{
  "event": "runtime_drift_critical",
  "tick": 1730600000,
  "payload": {
    "runtime_id": "rt-env-01",
    "drift_code": "MODEL_DRIFT_CRITICAL",
    "details": "hash_mismatch_inference_binary"
  },
  "signature_pq": "<ml-dsa-signature>",
  "prev_hash": "0xddddeeeeffff..."
}
```

---

# **ANNEX G — SECURE PAYMENTS MODULE (OPTIONAL, NORMATIVE)**

## **G.1 Purpose and Scope (INFORMATIVE)**

This optional module defines a post-quantum-secure payments flow leveraging PQSF primitives.
The module provides:

* PaymentIntent
* AuthorisationRequest / AuthorisationResult
* Receipt
* replay protection
* binding to EpochTick + ConsentProof + exporter_hash
* privacy and optional Travel Rule fields

This module is **transport-agnostic**, assuming PQSF Hybrid TLS (TLSE-EMP) and PQHD as the canonical wallet implementation.

---

## **G.2 Roles (INFORMATIVE)**

* **Payer** — End user initiating a payment.
* **PQHD Wallet** — Consumes PQSF primitives and signs payment consent.
* **Merchant Frontend** — Browser/native client under merchant control.
* **Payment Gateway / PSP** — Authorises and settles transactions.
* **Issuer / Acquirer** — Legacy card network participants.

PQSF governs the Wallet–Merchant–Gateway integration.

---

## **G.3 Security Objectives (NORMATIVE)**

### **Transport Security**

All payment messages MUST use PQSF Hybrid TLS 1.3 with exporter binding.

### **Authorisation Binding**

Every payment MUST bind:

1. PaymentIntent
2. ConsentProof (`action = payment.authorise`)
3. a fresh EpochTick
4. the session exporter_hash

### **Replay Resistance**

PSPs MUST enforce idempotency of AuthorisationRequest:

* idempotency_key MUST be unique
* stale ticks MUST be rejected

### **Data Protection**

* PAN/CVV MUST NOT be exposed to PQSF components.
* Merchants MUST NOT store raw PAN/CVV.

### **Auditability**

Receipts MUST prove:

* which intent was approved
* by which PSP key
* under which tick and exporter binding

### **Privacy**

Travel Rule data MUST be logically separable and optional.

---

## **G.4 Deployment Modes (NORMATIVE)**

### **G.4.1 Card-in-Wallet (CIW)**

* PQHD holds tokenised card or network tokens
* Merchant never sees PAN/CVV
* Highest security mode

### **G.4.2 Card-in-Browser (CIB)**

* PSP-provided browser element collects PAN/CVV
* Merchant sees only opaque element IDs
* PQSF protects all higher layers (consent, tick, session, replay)

Active mode MUST be declared via PaymentIntent.mode.

---

## **G.5 Data Structures (CDDL, NORMATIVE)**

Canonical encodings MUST be JCS JSON or deterministic CBOR.

### **Aliases**

```
b64u        = tstr
uuidv7      = tstr
iso4217     = tstr
unix_s      = uint
kid         = tstr
amount_str  = tstr
```

---

### **G.5.1 PaymentIntent**

```cddl
PaymentIntent = {
  intent_id:        uuidv7,
  merchant_id:      tstr,
  amount_minor:     uint,
  currency:         iso4217,
  description:      tstr,
  mode:             "CIW" / "CIB",
  method:           PaymentMethod,
  tick:             EpochTick,
  consent_hash:     b64u,
  created_at:       unix_s,
  expires_at:       unix_s,
  policy_ref:       tstr / null
}

PaymentMethod = CIW_Method / CIB_Method

CIW_Method = {
  kind:              "card_token",
  token_ref:         tstr,
  network:           tstr,
  last4:             tstr,
  exp_mmyy:          tstr,
  hardware_attested: bool
}

CIB_Method = {
  kind:        "psp_element",
  element_id:  tstr,
  capability:  [+ tstr]
}
```

**Normative rules:**

* `tick` MUST be fresh.
* `consent_hash` MUST be hash of canonical ConsentProof.
* `intent_id` MUST be unique.

---

### **G.5.2 Payment Consent Binding**

ConsentProof MUST use:

```
action = "payment.authorise"
```

and bind:

```cddl
PaymentConsentBinding = {
  intent_id:    uuidv7,
  amount_minor: uint,
  currency:     iso4217,
  merchant_id:  tstr,
  mode:         "CIW" / "CIB"
}
```

Tick MUST be valid for session exporter_hash.

---

### **G.5.3 AuthorisationRequest**

```cddl
AuthorisationRequest = {
  intent:           PaymentIntent,
  consent:          ConsentProof,
  idempotency_key:  uuidv7,
  ?compliance_data: TravelRuleData
}
```

Normative:

* idempotency_key must be unique per merchant+intent
* PSP MUST enforce idempotency

---

### **G.5.4 AuthorisationResult**

```cddl
AuthorisationResult = {
  intent_id:      uuidv7,
  status:         "approved" / "declined" / "challenge" / "error",
  auth_code:      tstr / null,
  psp_ref:        tstr / null,
  three_ds:       ThreeDSResult / null,
  signed_receipt: Receipt / null,
  ?error_code:    tstr,
  ?error_detail:  tstr
}

ThreeDSResult = {
  ds_trans_id: tstr,
  eci:         tstr,
  cavv:        b64u / null,
  result:      "frictionless" / "challenge" / "failed"
}
```

Normative:

* approved → signed_receipt MUST be present
* challenge → three_ds MUST be present

---

### **G.5.5 Receipt**

```cddl
Receipt = {
  receipt_id:      uuidv7,
  intent_id:       uuidv7,
  merchant_id:     tstr,
  amount_minor:    uint,
  currency:        iso4217,
  tick:            EpochTick,
  psp_ref:         tstr,
  capture_later:   bool,
  req_hash:        b64u,
  sig_pq_gateway:  b64u,
  sig_pub_kid:     kid
}
```

Normative rules:

* req_hash MUST be canonical hash of AuthorisationRequest
* sig_pq_gateway MUST be ML-DSA-65
* sig_pub_kid MUST reference PSP’s public key

---

### **G.5.6 TravelRuleData (Optional)**

```cddl
TravelRuleData = {
  originator:  tstr,
  beneficiary: tstr,
  transaction: {
    amount:         amount_str,
    currency:       iso4217,
    originator_ref: tstr
  }
}
```

Travel Rule data MUST NOT be stored longer than regulation requires.

---

### **G.5.7 WebhookEnvelope**

```cddl
WebhookEnvelope = {
  event_id:       uuidv7,
  issued_at:      unix_s,
  type:           tstr,
  payload:        b64u,
  sig_pq_gateway: b64u,
  sig_pub_kid:    kid
}
```

MUST reject:

* events older than 300 seconds
* duplicate event_id

---

## **G.6 Protocol Flow**

### **G.6.1 Intent Creation**

1. Merchant establishes TLSE-EMP.
2. Merchant creates PaymentIntent.
3. Wallet presents intent to user.

---

### **G.6.2 Consent and Wallet Signing**

1. Wallet validates exporter_hash.
2. User reviews summary.
3. Wallet signs ConsentProof.
4. Returns it to merchant or PSP.

---

### **G.6.3 Authorisation**

1. Merchant/wallet submits AuthorisationRequest.
2. PSP verifies:

   * exporter_hash
   * ConsentProof
   * Tick
   * idempotency_key
3. PSP performs network authorisation.

---

### **G.6.4 Result and Receipt**

1. PSP returns AuthorisationResult.
2. Approved → MUST include Receipt.
3. Merchant stores Receipt.
4. Wallet appends minimal ledger entry.

---

## **G.7 Error Handling**

Example mappings:

* `E_IDEMPOTENT` → 409
* `E_TICK_STALE` → 400
* `E_CONSENT_MISMATCH` → 400
* `E_PSP_DECLINE` → 200 ("declined")
* `E_SCA_REQUIRED` → 202 ("challenge")
* `E_TICK_UNAVAILABLE` → 503

Wallets MUST NOT display raw issuer codes.

---

## **G.8 Privacy and Compliance (INFORMATIVE)**

* CIW provides complete PAN isolation
* CIB isolates PAN using opaque PSP elements
* consent ledger SHOULD record minimal references
* Travel Rule data optional and minimally exposed

---

## **G.9 Implementation Profiles (INFORMATIVE)**

### **G.9.1 Minimal Payments Profile**

* CIB-only
* PaymentIntent, AuthorisationRequest, AuthorisationResult, Receipt

### **G.9.2 Full Payments Profile**

* CIW + CIB
* all G.5 structures
* full annex support

---

# **ANNEX H — CRYPTOGRAPHIC ERASURE PROOF (OPTIONAL, NORMATIVE)**

Cryptographic Erasure Proofs provide verifiable assurance that specific data has been permanently deleted.

They support:

* self-erasure
* third-party erasure
* zero-knowledge minimisation
* retention compliance

---

## **H.1 Purpose and Scope**

Supports verifiable deletion of:

* AI logs
* wallet metadata
* credentials
* profile data
* payment history subsets
* identity attributes

---

## **H.2 ErasureProof Structure**

```
ErasureProof = {
  "erasure_id":        tstr,
  "subject_hash":      bstr,
  "erasure_mode":      "self" | "third_party",
  "actor_id":          tstr,
  "tick":              EpochTick,
  "consent_ref":       tstr,
  "erased_at":         uint,
  "signature_pq":      bstr
}
```

Normative:

* subject_hash MUST be SHAKE256-256(canonical_subject_bytes)
* erasure MUST be authorised by ConsentProof
* tick MUST be fresh
* signature MUST be ML-DSA-65

---

## **H.3 Deletion Procedure**

1. Canonicalise target data
2. Hash subject_bytes
3. User issues ConsentProof (`delete.data`)
4. Actor deletes data
5. Actor constructs ErasureProof
6. Actor signs ErasureProof
7. Optionally log via ledger

---

## **H.4 Verification**

Verifier MUST:

1. canonicalise ErasureProof
2. verify ML-DSA-65 signature
3. validate tick
4. fetch ConsentProof
5. validate:

```
consent.action == "delete.data"
consent.intent_hash == subject_hash
```

6. confirm mode & actor match expectations

---

## **H.5 Optional Ledger Event**

```
event = "erasure_proof_issued"
```

Payload MUST include:

* erasure_id
* subject_hash
* erasure_mode
* actor_id

---

## **H.6 Security Properties (INFORMATIVE)**

* explicit intent-bound deletion
* privacy-preserving (hash only)
* independent verification (no trust in storage provider)
* tick-bound time correctness

---

# **ANNEX I — BROWSER PRIVACY & CONSENT CONTROLS (OPTIONAL, NORMATIVE)**

## **I.1 Purpose and Scope**

Defines per-category browser-level consent and privacy controls.
Allows users to grant or deny fine-grained permissions:

* AI training allowed / marketing prohibited
* analytics allowed / profiling prohibited
* retention controls
* session vs persistent scopes

---

## **I.2 PrivacyPolicy Structure**

```
PrivacyPolicy = {
  "policy_id":    tstr,
  "grants":       { * tstr => bool },
  "retention":    { * tstr => uint },
  "scope":        "session" | "persistent",
  "tick":         EpochTick,
  "consent_ref":  tstr,
  "signature_pq": bstr
}
```

Normative:

* grants MUST list explicit categories
* retention MAY be omitted
* scope MUST be declared
* signature MUST be ML-DSA-65

---

## **I.3 Consent Binding**

ConsentProof MUST have:

```
action = "privacy.set_policy"
intent_hash = SHAKE256-256(canonical(PrivacyPolicy))
```

---

## **I.4 Enforcement at Runtime**

When a page/script requests data:

1. Canonicalise requested category
2. Check grants
3. If denied → block
4. If retention expired → re-consent
5. Always fail-closed

---

## **I.5 Third-Party Access Control**

All 3P script requests MUST be subject to the same enforcement.

Unauthorized access MUST trigger:

```
privacy_violation_attempt
```

(optional ledger entry).

---

## **I.6 Optional Ledger Events**

* privacy_policy_applied
* privacy_policy_updated
* privacy_violation_attempt

---

## **I.7 Example (Informative)**

```
grants = {
  "ai_training": true,
  "marketing": false,
  "analytics": true,
  "personalisation": true
}

retention = {
  "ai_training": 604800,
  "analytics": 86400
}

scope = "persistent"
```

---

# **ANNEX J — EPHEMERAL CARTS & PRIVACY-PRESERVING ECOMMERCE (OPTIONAL, NORMATIVE)**

## **J.1 Purpose and Scope**

This annex defines a privacy-preserving ecommerce model where:

* users browse anonymously
* merchants see no cart until checkout
* carts are local-only and PQ-encrypted
* sessions use ephemeral PQ keys
* merchant visibility requires explicit consent
* tracking is impossible by design

This annex strengthens privacy without altering payment flows defined in Annex G.

---

## **J.2 EphemeralSession Structure**

```
EphemeralSession = {
  "session_id":        tstr,
  "session_pubkey":    bstr,
  "exporter_hash":     bstr,
  "created_at_tick":   EpochTick
}
```

Normative:

* session keypair MUST be ephemeral (per browsing session)
* exporter_hash MUST bind session to transport
* MUST NOT persist beyond session

---

## **J.3 EphemeralCart Structure**

```
EphemeralCart = {
  "cart_id":      tstr,
  "session_id":   tstr,
  "items":        [ CartItem ],
  "policy_ref":   tstr / null,
  "created_at":   uint,
  "expires_at":   uint,
  "hash":         bstr
}
```

Where:

```
CartItem = {
  "sku":  tstr,
  "qty":  uint,
  "price": uint,
  "meta": { * tstr => any }
}
```

Normative:

* MUST NOT be transmitted until checkout
* MUST be deterministically hashed:

```
cart.hash = SHAKE256-256(canonical(cart))
```

* MAY be encrypted locally via ML-KEM-1024

---

## **J.4 Merchant Visibility Rules**

Before checkout:

* merchant MUST NOT see the cart
* merchant MUST NOT receive identifiers
* merchant MUST NOT track items added/removed
* merchant MUST NOT infer user identity

Only after checkout-start consent MAY the cart be transmitted.

---

## **J.5 Checkout Commitment Flow**

1. User initiates checkout
2. Wallet signs ConsentProof:

```
action = "checkout.start"
intent_hash = cart.hash
```

3. Cart is transmitted only after consent
4. Merchant MUST verify:

```
cart.hash == consent.intent_hash
```

5. Payment then proceeds via Annex G.

---

## **J.6 Expiry & Auto-Zeroisation**

* session keypairs MUST be deleted after session
* carts MUST be deleted after expiration
* erasure MAY produce an ErasureProof (Annex H)

---

## **J.7 Optional Ledger Events**

* ephemeral_session_started
* checkout_started
* checkout_committed
* ephemeral_cart_erased

All MUST be signed and tick-bound.

---

## **J.8 Security Properties (Informative)**

* browsing unlinkability
* no persistent IDs
* no merchant visibility until user consents
* cryptographic binding of checkout to cart state
* deterministic enforcement rules
* privacy-first ecommerce architecture

---

# **ANNEX K — VERIFIED IDENTITY & KYC CREDENTIALS (OPTIONAL, NORMATIVE)**

## **K.1 Purpose and Scope**

This annex defines an optional, privacy-preserving identity credential model.
The goal is to allow:

* **third-party verified identity**
* **user-controlled selective disclosure**
* **minimal information release**
* **no correlation identifiers**
* **privacy across independent services**

PQSF treats identity as **attribute-based** rather than **document-based**, allowing proofs such as:

* “over 18”
* “resident in AU”
* “verified customer”

without revealing:

* legal name
* address
* phone number
* date of birth
* government identifiers

Full identity may be revealed **only if the user consents** or **lawfully required**.

---

## **K.2 KYCCredential Structure (NORMATIVE)**

```
KYCCredential = {
  "cred_id":       tstr,
  "issuer_id":     tstr,
  "subject_id":    tstr,
  "attributes":    { * tstr => any },
  "issued_at":     uint,
  "expires_at":    uint / null,
  "tick":          EpochTick,
  "issuer_sig_pq": bstr
}
```

### Normative rules:

* MUST be canonicalised before signature.
* `issuer_sig_pq` MUST be ML-DSA-65.
* `attributes` MUST contain only user-approved fields.
* PQSF does not define issuer trust policy — deployments do.

---

## **K.3 Selective Disclosure (NORMATIVE)**

Selective disclosure allows users to reveal only a subset of credential attributes.

```
KYCDisclosure = {
  "cred_id":      tstr,
  "disclosed":    { * tstr => any },
  "tick":         EpochTick,
  "consent_ref":  tstr,
  "signature_pq": bstr
}
```

### Normative rules:

* `disclosed` MUST be canonicalised.
* `signature_pq` MUST be ML-DSA-65.
* Verifier MUST confirm each disclosed field matches original credential.
* No undisclosed attributes MAY leak.

---

## **K.4 Verification (NORMATIVE)**

Verifiers MUST:

1. canonicalise KYCDisclosure
2. verify ML-DSA-65 signature
3. validate tick
4. validate ConsentProof
5. validate issuer signature (via KYCCredential)
6. confirm attributes are unchanged
7. apply issuer trust policy

If ANY step fails → reject.

---

## **K.5 Privacy Considerations (INFORMATIVE)**

PQSF treats identity as a **privacy-first mechanism**:

* users disclose only what is required
* verifiers MUST NOT receive unrelated attributes
* disclosures are tick-bound and explicit
* no correlation identifiers are exposed
* full legal identity MUST NOT be disclosed without explicit user consent
* full identity MAY be disclosed only in response to a lawful, authorised investigative request

This annex ensures KYC can exist *without surveillance* or unintended data spread.

---

# **ANNEX L — DELEGATED IDENTITY (OPTIONAL, NORMATIVE)**

## **L.1 Purpose and Scope**

Defines a deterministic, privacy-preserving framework for delegating identity or authority to:

* another device
* a family member
* a trusted helper
* a service agent
* an automation system

Delegation is:

* scoped
* consent-bound
* tick-bound
* revocable
* privacy-minimised

---

## **L.2 DelegatedIdentity Structure (NORMATIVE)**

```
DelegatedIdentity = {
  "delegation_id":  tstr,
  "subject_id":     tstr,
  "delegate_id":    tstr,
  "scopes":         [* tstr],
  "limits":         { * tstr => uint },
  "valid_from":     uint,
  "valid_until":    uint,
  "tick":           EpochTick,
  "consent_ref":    tstr,
  "signature_pq":   bstr
}
```

Normative Rules:

* MUST be authorised by ConsentProof (`action="identity.delegate"`).
* `scopes` MUST explicitly define permitted actions.
* `limits` MUST define per-scope restrictions.
* lifetime MUST be bounded by tick_expiry of authorising consent.
* signature MUST be ML-DSA-65.

---

## **L.3 Verification (NORMATIVE)**

Verifier MUST:

1. canonicalise DelegatedIdentity
2. verify signature
3. validate EpochTick
4. check ConsentProof
5. enforce scopes and limits

Fail-closed behaviour is mandatory.

---

## **L.4 Revocation (NORMATIVE)**

```
DelegationRevocation = {
  "delegation_id": tstr,
  "tick":          EpochTick,
  "signature_pq":  bstr
}
```

Revocation MUST:

* immediately invalidate delegation
* be logged to the PQSF ledger (`delegation_revoked`)

---

## **L.5 Privacy (INFORMATIVE)**

* delegate receives only scoped identity
* no sensitive personal data included by default
* no correlation identifiers beyond delegation_id
* revocable and time-limited

---

# **ANNEX M — DELEGATED PAYMENTS (OPTIONAL, NORMATIVE)**

## **M.1 Purpose and Scope**

This annex defines formal structures for delegating **payment authority** in a secure, tick-bound, deterministic way.

Supports:

* family spending limits
* employee delegated spending
* subscription automation
* merchant-specific delegation
* spending constraints

All delegated payments remain governed by Policy Enforcer and ConsentProof.

---

## **M.2 DelegatedPaymentCredential Structure (NORMATIVE)**

```
DelegatedPaymentCredential = {
  "delegation_id":     tstr,
  "payer_id":          tstr,
  "delegate_id":       tstr,
  "allowed_merchants": [* tstr] / null,
  "spend_limit_minor": uint,
  "currency":          iso4217,
  "per_tx_limit":      uint / null,
  "interval_limit":    uint / null,
  "valid_from":        uint,
  "valid_until":       uint,
  "tick":              EpochTick,
  "consent_ref":       tstr,
  "signature_pq":      bstr
}
```

Normative notes:

* MUST be signed with ML-DSA-65
* MUST be authorised via ConsentProof (`action="payment.delegate"`)
* MUST enforce monetary limits and scopes
* MUST be tick-bound
* MUST be canonical

---

## **M.3 Delegated Payment Transaction**

```
DelegatedPayment = {
  "delegation_id": tstr,
  "intent":        PaymentIntent,
  "delegate_tick": EpochTick,
  "signature_pq":  bstr
}
```

Normative:

* MUST derive from a valid DelegatedPaymentCredential
* MUST validate tick freshness
* MUST validate ML-DSA-65 signature

---

## **M.4 Verification Rules**

PSP, merchant, or wallet MUST:

1. verify DelegatedPaymentCredential signature
2. verify Delegate signature
3. validate Tick and ConsentProof
4. enforce `spend_limit_minor` cumulative
5. enforce `per_tx_limit`
6. enforce `interval_limit`
7. enforce merchant allowlist/denylist

Fail-closed required.

---

## **M.5 Ledger Events (OPTIONAL)**

Deployments MAY log:

* `delegated_payment_created`
* `delegated_payment_redeemed`
* `delegated_payment_revoked`
* `delegated_payment_violation`

All MUST be ML-DSA-65 signed and tick-ordered.

---

## **M.6 Revocation (NORMATIVE)**

```
DelegatedPaymentRevocation = {
  "delegation_id": tstr,
  "tick":          EpochTick,
  "signature_pq":  bstr
}
```

Revocation MUST immediately invalidate delegation and SHOULD be written to the ledger.

---

## **M.7 Privacy (INFORMATIVE)**

* delegates see only minimal payment metadata
* merchant identity is constrained by allowlist
* no persistent identifiers unless user opts in
* spending behaviour cannot be profiled across merchants

---

# **ANNEX N — Quantum-Safe Wallet-Backed User Login (OPTIONAL, NORMATIVE)**

## **N.1 Purpose and Scope**

This annex defines an optional, wallet-backed user login mechanism that uses PQSF primitives instead of passwords or ad hoc session identifiers. The goal is to allow applications to authenticate users by:

* treating the wallet as the primary user credential,
* binding login to a fresh EpochTick and exporter_hash,
* using ConsentProof to record explicit user intent,
* avoiding reusable passwords and shared secrets.

This annex does not modify any core PQSF semantics. It is an application pattern that combines PQSF, PQHD, and optional identity modules.

## **N.2 Roles (Informative)**

* **User**
  Person who controls a PQHD wallet and wishes to log in.

* **Wallet Client**
  PQHD or PQSF-aware application that can produce ConsentProof objects and export non-custodial authentication assertions.

* **Relying Service**
  Website, API, or application that wishes to authenticate a user using wallet-backed login instead of passwords.

* **Identity Provider (optional)**
  Entity that issues KYCCredential or other identity attributes under Annex K.

## **N.3 Data Structures (Normative)**

### **N.3.1 WalletLoginIntent**

A login begins with a WalletLoginIntent issued by the relying service:

```
WalletLoginIntent = {
  "intent_id":      tstr,
  "service_id":     tstr,
  "redirect_uri":   tstr / null,
  "scopes":         [* tstr],
  "nonce":          bstr,
  "tick":           EpochTick,
  "expires_at":     uint,
  "policy_ref":     tstr / null
}
```

Normative requirements:

1. `intent_id` MUST be unique per service.
2. `tick` MUST be a valid EpochTick under PQSF section 4.
3. `expires_at` MUST be derived from `tick.t` and MUST NOT exceed the normal consent window.
4. `nonce` MUST be unpredictable and MUST be unique per intent.
5. The intent MUST be encoded using deterministic CBOR or JCS JSON.

### **N.3.2 WalletLoginConsent**

The wallet binds user intent to the login via a ConsentProof:

```
ConsentProof = {
  "action":        "auth.login",
  "intent_hash":   bstr,
  "tick_issued":   uint,
  "tick_expiry":   uint,
  "exporter_hash": bstr,
  "role":          tstr / null,
  "consent_id":    tstr,
  "signature_pq":  bstr
}
```

Normative requirements:

1. `intent_hash` MUST equal `SHAKE256-256(canonical(WalletLoginIntent))`.
2. `action` MUST be `"auth.login"` for login flows.
3. `tick_issued` and `tick_expiry` MUST satisfy PQSF section 5.3.
4. `exporter_hash` MUST match the active transport session as in PQSF section 7.2.
5. The ConsentProof MUST be canonical and signed with ML-DSA-65.

### **N.3.3 WalletLoginAssertion**

The wallet produces a WalletLoginAssertion that the relying service can verify:

```
WalletLoginAssertion = {
  "assertion_id":   tstr,
  "intent_id":      tstr,
  "service_id":     tstr,
  "user_pub":       bstr,
  "tick":           EpochTick,
  "exporter_hash":  bstr,
  "consent_id":     tstr,
  "credential_ref": tstr / null,
  "signature_pq":   bstr
}
```

Normative requirements:

1. `user_pub` MUST be derived deterministically from a wallet key class that is dedicated to authentication and MUST NOT be a custody key.
2. `tick` MUST be a fresh EpochTick that satisfies PQSF section 4.3.
3. `exporter_hash` MUST match both the wallet session and the relying service session.
4. `credential_ref` MAY reference a KYCCredential or KYCDisclosure under Annex K but is optional.
5. `signature_pq` MUST be an ML-DSA-65 signature over the canonical WalletLoginAssertion.

## **N.4 Login Flow (Normative)**

### **N.4.1 Step 1 — Intent Creation**

The relying service:

1. Generates a WalletLoginIntent.
2. Canonicalises and signs or logs it locally.
3. Presents the intent to the user wallet via browser, QR, deep link, or STP.

The intent MUST be delivered over a PQSF compliant transport (TLSE-EMP or STP).

### **N.4.2 Step 2 — User Consent in Wallet**

The wallet:

1. Validates the EpochTick in the intent.
2. Validates `service_id`, `scopes`, and optional `policy_ref` against local policy.
3. Constructs a ConsentProof with `action = "auth.login"`.
4. Signs the ConsentProof using an ML-DSA-65 key from a non-custodial class dedicated to authentication.

If any validation fails, the wallet MUST refuse to proceed and MUST NOT generate a login assertion.

### **N.4.3 Step 3 — Assertion Creation**

If consent is valid, the wallet:

1. Derives an authentication public key `user_pub` deterministically from a PQHD or PQSF key class that is reserved for login identities.
2. Builds a WalletLoginAssertion that includes `intent_id`, `service_id`, `user_pub`, `tick`, `exporter_hash`, and `consent_id`.
3. Signs the assertion with ML-DSA-65.
4. Returns the assertion to the relying service over a PQSF transport.

### **N.4.4 Step 4 — Service-Side Validation**

The relying service MUST:

1. Canonicalise the WalletLoginAssertion.
2. Verify the ML-DSA-65 signature.
3. Validate the EpochTick freshness and monotonicity.
4. Reconstruct `intent_hash` and validate the corresponding ConsentProof.
5. Confirm that `exporter_hash` matches the active session.
6. Check that `intent_id` and `nonce` have not been used before.

If all checks succeed, the service MAY create a local user session bound to `user_pub` and MUST record a ledger entry using the PQSF ledger with an event such as:

```
{
  "event": "auth_login",
  "tick": <tick>,
  "payload": {
    "intent_id": "<id>",
    "service_id": "<service>",
    "user_pub": "<public-key-fingerprint>"
  }
}
```

If any check fails, the service MUST reject the login and MUST NOT create or resume a user session.

## **N.5 Session Binding and Logout (Normative)**

1. The relying service MUST bind its application session identifier to `user_pub` and `exporter_hash`.

2. Session resumption MUST require:

   * a fresh EpochTick,
   * a valid exporter_hash,
   * a stored binding to `user_pub`.

3. Logout operations SHOULD be recorded as ledger events with `event = "auth_logout"` but MAY be handled purely at the application layer.

4. Session cookies or tokens MUST NOT be treated as primary credentials and MUST be considered valid only in combination with the WalletLoginAssertion binding.

## **N.6 Security Properties (Informative)**

Wallet-backed login under this annex provides:

* **Passwordless authentication**
  No reusable password or shared secret is needed. The wallet acts as the authentication authority.

* **Replay resistance**
  EpochTick freshness, `nonce`, and exporter_hash prevent replay of login assertions.

* **Phishing resistance**
  ConsentProof binds the login to a specific service and session. A malicious site cannot reuse a login created for a different service_id or exporter_hash.

* **Upgrade path for identity**
  KYCCredential and KYCDisclosure under Annex K can be attached via `credential_ref` without exposing unnecessary identity attributes.

* **Separation from custody keys**
  Authentication keys are derived from dedicated non-custodial classes and never share material with PQHD custody keys.

---

## **Acknowledgements (Informative)**

This specification acknowledges the foundational contributions of:

Peter Shor, whose discovery of Shor’s algorithm motivates the transition to post-quantum security models.

Ralph Merkle, originator of Merkle tree constructions, which influence deterministic ledger structures.

Joan Daemen and Vincent Rijmen, for their work leading to the AES family and the design lineage that informed sponge-based hashing.

Guido Bertoni, Joan Daemen, Michaël Peeters, and Gilles Van Assche, inventors of Keccak, from which SHAKE256 and cSHAKE256 are derived.

Their work provides essential primitives underpinning PQSF’s deterministic, post-quantum-secure architecture.

---

If you find this work useful and want to support it, you can do so here:
bc1q380874ggwuavgldrsyqzzn9zmvvldkrs8aygkw
