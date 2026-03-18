W.E.A.L. Ledger Protocol — Core (Economic Infrastructure Potent)
Version: 1.1 (Revised)
Scope: Bounded ledger specification for recording issuance events, balance state updates, and synchronization across a Local → Regional → Federated topology.

---

## 1) Ledger Object Model

### 1.1 Entry Object Schema (Append-Only Record)

Each ledger entry is an immutable record describing a single issuance event and its parameters at time of entry creation.

**Entry (JSON object)**

- `entry_id` (string, required): Unique identifier for the entry.
- `timestamp` (string, required): ISO-8601 UTC timestamp (`YYYY-MM-DDTHH:MM:SSZ`).
- `ledger_id` (string, required): Identifier of the originating ledger instance (Local).
- `subject_id` (string, required): Identifier of the credited account (pseudonymous permitted).
- `event_type` (string, required): Contribution/event classification (enumerated by the Issuance Inputs file).
- `inputs` (object, required): Canonical issuance inputs used to compute the issued amount.
- `issued_amount` (number, required): Result of the rule-defined issuance calculation.
- `decay_profile_id` (string, required): Reference to the decay parameter set in effect at issuance time.
- `parameter_set_id` (string, required): Reference to the full parameter set used (includes caps/bounds).
- `origin_ref` (object, optional): External reference metadata; structure defined by implementing institution.
- `integrity` (object, required):
  - `entry_hash` (string, required): Hash of the canonicalized entry payload (excluding `integrity.entry_hash`).
  - `prev_entry_hash` (string, optional): Hash pointer to the immediately previous accepted entry in this ledger instance (tamper-evidence only; no consensus implied).
  - `canonicalization` (string, required): Name/version of canonical JSON serialization rules.

The presence of `prev_entry_hash` supports tamper-evident ordering within a single ledger instance. It does not imply distributed consensus, tokenization, or blockchain operation.

---

### 1.2 Required Fields

An entry is structurally valid only if all required fields are present, type-correct, and within bounds defined in the associated parameter files.

Required:
`entry_id`, `timestamp`, `ledger_id`, `subject_id`, `event_type`, `inputs`, `issued_amount`, `decay_profile_id`, `parameter_set_id`, `integrity.entry_hash`, `integrity.canonicalization`.

---

### 1.3 Timestamping Standard

- All timestamps MUST be recorded in UTC using ISO-8601 with trailing `Z`.
- Local time inputs MUST be converted to UTC prior to entry creation.
- Timestamps are immutable after acceptance; corrections use the correction protocol (Section 2.5).

---

### 1.4 Unique Identifier Rules

- `entry_id` MUST be globally unique within the Federated topology.
- Acceptable generation methods:
  - ULID
  - UUIDv7
- `ledger_id` MUST be unique per ledger instance and stable across restarts.
- `subject_id` MUST be stable within a ledger.

---

### 1.5 State Update Logic (Balance Model)

Ledger state is derived from:

- The ordered set of accepted entries
- The decay model referenced by `decay_profile_id`
- The time-indexed update rules (Section 5)

Derived artifacts (not entries):

- `balance_state(subject_id, t)` computed from accepted entries and decay rules at time `t`.
- `snapshot(t)` representing computed balances at a specific time boundary.

Entries do not directly mutate stored balances; balances are computed from the append-only record plus decay rules.

---

## 2) Transaction Recording Standard

### 2.1 Append-Only Structure

- Ledgers MUST implement an append-only log of accepted entries.
- Deletion or in-place modification of accepted entries is disallowed.
- Ordering is defined by acceptance order.

---

### 2.2 Validation Preconditions

Prior to acceptance, an entry MUST pass:

- Schema validation (Section 1.1)
- Type validation
- Bounds validation:
  - `issued_amount` within issuance caps defined by the parameter set
  - Required `inputs` keys present and within bounds

---

### 2.3 Rejection Criteria

An entry MUST be rejected if:

- Required fields are missing
- Timestamp format is invalid or not UTC
- Duplicate `entry_id`
- Unknown `ledger_id`
- Unknown `decay_profile_id` or `parameter_set_id`
- `issued_amount` does not match the rule-defined issuance calculation for provided inputs
- Integrity hash mismatch

---

### 2.4 Error Handling State

Rejected entries are recorded in an operational log (separate from the append-only ledger of accepted entries) including:

- `attempt_id`
- `entry_id` (if present)
- `received_at` (UTC ISO-8601)
- `rejection_reason` (enumerated)
- `rejection_detail`
- `source_ref` (optional)

Rejected entries do not affect derived balances.

---

### 2.5 Correction Protocol

Corrections are implemented as additional entries, not edits.

If an accepted entry requires correction:

- `event_type = "CORRECTION"`
- `inputs.correction_of_entry_id` references the prior entry
- `issued_amount` computed by the same rule-defined issuance calculation

The original entry remains in the log.
State reconstruction applies both original and correction entries in acceptance order.

Silent overrides are disallowed.

---

## 3) Issuance Logic (Rule-Defined)

### 3.1 Rule-Defined Issuance Calculation

Issuance is computed by a rule-defined function:

`ISSUE(inputs, parameter_set)`

For identical inputs and parameter sets, the function MUST produce identical outputs.

---

### 3.2 Required Input Parameters

`inputs` MUST include:

- `coherence_input` (object)
- `impact_tier` (string or integer)
- `issuer_weight` (number)
- `federation_multiplier` (number)

No additional inputs may be used unless defined in the parameter set.

---

### 3.3 Reproducibility Requirement

The ledger MUST store all inputs required to reproduce `issued_amount`.

Floating-point handling MUST be specified in the parameter set:

- fixed-point decimal representation or
- explicit rounding mode and precision

---

### 3.4 No Discretionary Override

- `issued_amount` MUST be computed exclusively via the rule-defined issuance calculation.
- Manual edits or discretionary issuance are disallowed.
- Parameter changes apply prospectively via new parameter sets.

---

### 3.5 Bounded Issuance Constraint

Parameter sets MUST define:

- `max_issued_amount_per_entry`
- `max_issuer_weight`
- `max_federation_multiplier`

Acceptance MUST fail if caps are exceeded.

---

## 4) Synchronization Flow

### 4.1 Topology

- Local Ledger (LL)
- Regional Ledger (RL)
- Federated Ledger (FL)

This file defines synchronization as data exchange and reconciliation. It does not define distributed consensus.

All synchronization occurs within participating institutions operating under applicable legal and regulatory frameworks.

---

### 4.2 Synchronization Cadence

Cadence is configuration-defined and declared per deployment.

Supported flows:

- LL → RL: daily batch or streaming
- RL → FL: weekly batch or streaming
- FL → RL/LL: parameter distribution on change

No performance guarantees are implied.

---

### 4.3 Conflict Resolution (Rule-Based Only)

Conflicts include:

- Duplicate `entry_id` with mismatched payload
- Hash chain integrity breaks
- Missing parameter sets

Resolution:

- Identity conflicts marked `CONFLICTED`
- Integrity breaks marked `INTEGRITY_BREAK`
- Unknown parameter sets quarantined

---

### 4.4 State Reconciliation

Reconciliation includes:

- Import of accepted entries
- Integrity verification
- Reproduction of issuance calculation
- Provenance preservation

---

### 4.5 Node Role Boundary

Node roles define data flow only; governance authority remains external to the ledger protocol.

---

## 5) Time-Indexed State Updates

### 5.1 Time-Step Granularity

Discrete time steps:

- default: weekly
- allowable: day/week/month

---

### 5.2 Update Ordering Rules

At boundary `T_k`:

1. Apply accepted entries with `timestamp <= T_k`
2. Apply decay transformations
3. Produce `snapshot(T_k)`

Entries accepted after `T_k` are incorporated at the next snapshot unless the deployment explicitly defines controlled historical recomputation procedures outside the core ledger specification.

---

### 5.3 State Snapshot Logic

Snapshot includes:

- `snapshot_id`
- `snapshot_time`
- `parameter_set_versions`
- `balance_map`
- `snapshot_hash`

Snapshots are derived artifacts.

---

### 5.4 Historical Reconstruction Capability

Given:

- Append-only entry log
- Parameter sets
- Decay profiles
- Canonicalization rules

An implementation MUST reconstruct balances at declared time boundaries.

---

## Legal Neutrality Boundary

This specification defines a ledger recording and synchronization model only and does not establish legal tender status, banking authority, or monetary sovereignty.
