Decay Model — Bounded Specification  
Scope: Mathematical transformation rule applied to issued credit balances over discrete time steps.  
Version: 1.1 (Revised)

---

## 1) Decay Function Definition

### 1.1 Mathematical Form

Decay is defined as a discrete exponential transformation applied at each declared time step.

For an issued amount \( W_0 \) at time \( t_0 \), the balance at elapsed time \( t \) is:

\[
W(t) = W*0 \times (1 - r*{\text{effective}})^{\Delta n}
\]

Where:

- \( W(t) \): balance at time \( t \)
- \( W_0 \): issued amount at issuance time
- \( r\_{\text{effective}} \): effective decay rate per declared time step
- \( \Delta n \): number of elapsed discrete time steps since issuance

Constraint:

\[
0 \le r*{\text{effective}} \le r*{\text{max}}
\]

Where:

- \( r\_{\text{max}} \) is defined by the active parameter set
- Negative decay rates are invalid
- Growth behavior (i.e., negative decay) is not permitted unless explicitly defined in a separate parameter specification

This transformation defines time-indexed value adjustment only. It does not imply incentive shaping, behavioral targeting, redistribution intent, or social outcome optimization.

---

### 1.2 Function Signature

DECAY(
initial_amount: decimal,
elapsed_steps: integer,
effective_decay_rate: decimal
) -> decimal

---

### 1.3 Input Variables

- `initial_amount` ≥ 0
- `elapsed_steps` ≥ 0 (integer)
- `effective_decay_rate` satisfying:
  \[
  0 \le r*{\text{effective}} \le r*{\text{max}}
  \]

---

### 1.4 Time Variable Definition

- Time is discretized into fixed-length declared steps.
- A step unit is defined in Section 4.
- `elapsed_steps` is the integer count of completed steps between issuance timestamp and evaluation timestamp.

Fractional steps are not applied unless explicitly defined in deployment configuration.

---

### 1.5 Output Variable Definition

- Output is a non-negative decimal representing transformed balance at evaluation time.
- Output precision and rounding are governed by Section 2.

---

### 1.6 Discrete Application Rule

Decay is applied only after issuance and only at completed declared time-step boundaries.

For multiple entries:

- Each entry decays independently based on its issuance timestamp and associated decay profile.
- Aggregate balance is the sum of independently decayed entries.

No continuous transformation is implied.

---

## 2) Parameter Bounds

### 2.1 Base Decay Rate Bounds

Parameter sets MUST define:

- `r_min` ≥ 0.0000
- `r_max` ≤ 0.0500 per declared time step

Constraint:

\[
0 \le r*{\text{base}} \le r*{\text{max}}
\]

---

### 2.2 Effective Rate Bounds

If modifiers are enabled:

\[
r*{\text{effective}} = r*{\text{base}} \times m
\]

Where:

- \( m \in [m_{\text{min}}, m_{\text{max}}] \)
- \( m\_{\text{min}} \ge 0.0 \)
- \( m\_{\text{max}} \le 3.0 \)

Effective rate enforcement:

\[
r*{\text{effective}} = \min(\max(r*{\text{effective}}, 0), r\_{\text{max}})
\]

Any value outside bounds MUST cause validation failure.

---

### 2.3 Precision and Rounding

Parameter sets MUST define:

- Decimal precision (e.g., 4–8 decimal places)
- Rounding mode:
  - Half-up
  - Half-even
  - Truncate
  - Floor

Rounding is applied after exponentiation calculation.

---

### 2.4 Validation Enforcement

During validation:

- `effective_decay_rate` MUST be computed using declared parameter inputs only.
- Values outside declared bounds are invalid.
- Non-numeric inputs are invalid.

---

## 3) Modifier Constraints

### 3.1 Allowed Modifier Types

Modifiers MUST be explicitly defined in the associated parameter set.

Permitted structures include:

- Scalar multiplier
- Step-indexed multiplier
- Profile-based multiplier

No additional modifier sources are permitted outside the declared parameter set.

---

### 3.2 Constraint Enforcement

- Modifiers MUST be stored in the parameter set referenced by `parameter_set_id`.
- Decay calculation MUST use only modifier values active at issuance time unless forward-application rules apply.
- Runtime adjustment outside parameter sets is disallowed.

---

### 3.3 No Discretionary Adjustment

Decay rates and modifiers:

- MUST be defined prior to application.
- MUST be referenced by `decay_profile_id` or `parameter_set_id`.
- MUST not be altered per-entry outside rule-defined constraints.

---

## 4) Temporal Indexing Standard

### 4.1 Step Granularity

Deployment MUST declare one of:

- Day
- Week
- Month

Step unit MUST remain constant within a parameter set.

---

### 4.2 Elapsed Time Calculation

\[
\Delta n = \left\lfloor \frac{t*{\text{evaluation}} - t*{\text{issuance}}}{\text{step_duration}} \right\rfloor
\]

Where:

- All timestamps are UTC ISO-8601.
- Partial steps are ignored unless explicitly defined in deployment configuration.

---

### 4.3 Application Order

At snapshot boundary \( T_k \):

1. Identify entries with `timestamp ≤ T_k`
2. Compute `elapsed_steps` per entry
3. Apply decay transformation
4. Sum results for aggregate balance

Issuance precedes decay application.

---

### 4.4 Snapshot Interaction

Decay is evaluated only at declared snapshot time boundaries.

Entries accepted after a snapshot boundary are included at the next boundary.

No implicit retroactive recomputation is defined in this specification.

---

## 5) Stability Constraints

### 5.1 Numerical Stability Conditions

Decay remains numerically stable when:

- \( 0 \le r\_{\text{effective}} \le 1 \)
- Precision is sufficient to prevent numerical underflow within declared time horizon
- Exponentiation uses defined decimal or floating precision rules

---

### 5.2 Non-Negativity Constraint

Given:

\[
W*0 \ge 0
\quad \text{and} \quad
0 \le r*{\text{effective}} \le 1
\]

Then:

\[
W(t) \ge 0
\]

Negative balances are not produced by the decay function.

---

### 5.3 Asymptotic Behavior

As \( \Delta n \to \infty \):

\[
W(t) \to 0
\]

This reflects mathematical convergence of the exponential function under bounded decay.

No additional interpretation is implied.

---

### 5.4 Parameter Update Forward-Application Rule

If a new parameter set becomes active at time \( T_u \):

- Entries issued before \( T_u \) retain their original `decay_profile_id`.
- Entries issued at or after \( T_u \) reference the new parameter set.
- Retroactive application of new decay rates is not permitted unless explicitly defined in external deployment documentation.

---
