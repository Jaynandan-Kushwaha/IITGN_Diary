# POR Removal Justification – VSD Caravel SoC (SCL-180)

---

## Objective

This document provides a clear technical justification for **removing the on-chip Power-On Reset (POR)** logic from the VSD Caravel SoC when targeting the **SCL-180 PDK**.

The intent is to demonstrate that:
- The existing RTL does **not architecturally depend** on POR
- An **external reset pin is sufficient and safe**
- Retaining POR in SCL-180 adds **no functional value**

---

## Background

The original Caravel RTL includes a module named `dummy_por`, inherited from flows targeting **SKY130**.  
In SKY130, POR was necessary due to uncertain pad behavior during power-up.

However, **SCL-180 has fundamentally different pad characteristics**, making the same POR strategy unnecessary.

---

## Reset Pad Behavior in SCL-180

In the SCL-180 technology:

- Input pads become valid immediately once **VDD is applied**
- No internal enable or gating is required for pad operation
- Reset pins are **asynchronous and power-independent**
- Reset can be asserted safely during the power ramp
- Reset deassertion is clean once driven externally

This ensures that reset behavior is **deterministic from power-on**.

---

## Analysis of Existing POR Usage

### Where `dummy_por` Appears
- Instantiated only at the **top level**
- Generates signals such as:
  - `porb_h`
  - `porb_l`
  - `por_l`

### What These Signals Do
- They **fan out into reset trees**
- They reset flip-flops like any standard reset
- They do **not**:
  - Detect power stability
  - Implement analog POR behavior
  - Enforce delayed reset release

Functionally, these signals behave exactly like a **normal external reset**.

---

## RTL Dependency Audit

A detailed review of the RTL confirms:

- No logic waits for POR completion
- No counters or FSMs depend on POR timing
- No power-good checks exist
- No sequencing logic assumes internal POR behavior

All reset paths treat POR signals as **generic digital resets**.

This proves that the SoC **does not rely on POR semantics**.

---

## Why POR Was Required in SKY130 (But Not Here)

| Aspect | SKY130 | SCL-180 |
|------|-------|--------|
| Pad readiness after VDD | Uncertain | Immediate |
| Reset validity during ramp | Not guaranteed | Guaranteed |
| Internal enables | Required | Not required |
| Need for POR | Mandatory | Unnecessary |

In SKY130, POR masked pad uncertainty.  
In SCL-180, that uncertainty **does not exist**.

---

## Risks of Retaining POR in SCL-180

Keeping POR in this technology can introduce problems:

- Artificial reset delays not present in silicon
- Simulation vs silicon mismatch
- Additional blackboxes in synthesis and GLS
- Violation of clean, industry-standard reset design

POR becomes **technical debt**, not protection.

---

## Justification for POR Removal

Removing POR is justified because:

- External reset is electrically valid at power-up
- No RTL block depends on POR-specific behavior
- Reset requirements are fully met without POR
- Design becomes simpler and more portable
- Synthesis and GLS flows become cleaner

---

## Final Conclusion

For the **VSD Caravel SoC on SCL-180**:

- On-chip POR provides **no functional benefit**
- The design operates correctly with a **single external reset**
- POR removal aligns with **industry best practices**
- The SoC remains safe, deterministic, and robust

**POR can be safely removed without impacting functionality.**

---

