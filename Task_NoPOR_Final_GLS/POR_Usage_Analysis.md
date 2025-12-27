# POR Usage Analysis – VSD Caravel SoC (SCL-180)

## Purpose of This Document

This document evaluates the role of the on-chip **Power-On Reset (POR)** in the
current VSD Caravel SoC RTL when targeting the **SCL-180** technology.

The intent of this analysis is to clearly understand:
- Where the `dummy_por` block is instantiated
- How POR-related signals are used inside the SoC
- Whether any logic *functionally depends* on POR behavior
- If the RTL assumes any POR-specific timing characteristics

This helps determine whether POR is truly required or can be safely removed.

---

## Location of `dummy_por` Instantiation

The `dummy_por` module is instantiated at the **top-level** design file
(`vsdcaravel.v`).

It generates three reset-related signals:
- `porb_h`
- `porb_l`
- `por_l`

These signals are routed into:
- Global SoC reset paths
- Housekeeping block resets
- Reset distribution logic

No other part of the design instantiates or generates POR signals.

---

## Interpretation of POR Signals

### `porb_h`
- Active-low reset signal
- Feeds reset inputs of:
  - CPU core
  - Peripherals
  - Interconnect and SRAM logic
- Behaves exactly like a standard asynchronous reset

There is no functional difference between `porb_h` and an external reset pin.

---

### `porb_l`
- Used primarily by housekeeping logic
- Derived directly from `porb_h`
- Does not introduce independent reset timing or sequencing

In practice, it is simply a renamed version of the same reset source.

---

### `por_l`
- Auxiliary control signal
- Exists for POR-style sequencing
- Does **not** directly reset flip-flops

Its presence does not influence functional correctness of the SoC.

---

## Meaning of “Reset Tree Fan-Out”

A **reset tree** refers to a reset signal being distributed to a large number
of flip-flops across the chip.

Key findings:
- `porb_h` and `porb_l` are only **sources** for reset trees
- No logic modifies, filters, or delays these signals
- No power-detection or analog POR behavior exists

Therefore, these signals act as **plain digital resets**, nothing more.

---

## Housekeeping Logic Evaluation

The housekeeping block:
- Treats reset as a normal digital input
- Does not wait for POR completion
- Does not monitor power stability
- Does not expect delayed reset deassertion

Any reference to POR signals is nominal and not functionally meaningful.

---

## RTL Review: No POR Timing Dependency

A detailed inspection of the RTL shows:
- No counters triggered by POR
- No logic waiting for POR to settle
- No power-good checks
- No reset release delays tied to POR

Across the entire design, reset is used as a **generic asynchronous reset**.

This confirms that the SoC does **not** rely on POR-specific behavior.

---

## Final Conclusion

- `dummy_por` does not implement real POR functionality
- All POR signals only serve as reset sources
- No block assumes POR-related timing or sequencing
- Functional behavior remains unchanged without POR

The VSD Caravel SoC is fully compatible with a **single external reset pin**
when implemented in **SCL-180** technology.
