# Reset Pad Analysis — SCL-180 vs SKY130

## Objective
This document explains the behavior of reset pads in the **SCL-180 PDK** and clarifies why an **on-chip Power-On Reset (POR)** circuit is **not required**, in contrast to designs based on the **SKY130 PDK**.

---

## Reset Pad Characteristics in SCL-180

In the **SCL-180 technology**, reset pads are designed to be electrically reliable from the very beginning of the power-up sequence:

- Input pads are directly powered by **VDD**
- No internal enable or gating logic is needed
- Reset paths are **not dependent on POR logic**
- Reset signals are **asynchronous**
- Reset pins are functional immediately during power ramp-up

This ensures that reset can be safely asserted even while the supply voltage is still stabilizing.

---

## Reset Source vs Reset Quality

From a digital logic perspective, flip-flops only care about **signal quality**, not the reset source:

**Flip-flops require:**
- A clean reset assertion
- A clean reset de-assertion

**Flip-flops do NOT require:**
- Reset to originate from an internal POR
- Reset to be generated on-chip
- Knowledge of how reset was created

As long as timing and signal integrity are satisfied, the reset origin (external pin or POR) is irrelevant.

---

## Why SKY130 Needed an On-Chip POR

In **SKY130**, reset reliability during power-up was not guaranteed:

- Pad behavior during early power stages was uncertain
- Internal enables could be in undefined states
- Pull-ups and pull-downs were not immediately reliable
- Reset pins were unsafe during initial power ramp

Because of this, an **on-chip POR** was necessary to:
- Isolate the design from unstable pad behavior
- Delay reset release until power was fully valid
- Prevent undefined internal states

---

## Why SCL-180 Does NOT Need a POR

SCL-180 eliminates the issues seen in SKY130:

- Pads become electrically valid as soon as VDD is present
- No dependency on internal enable signals
- Reset can be driven directly from the pad at power-on

As a result, an internal POR does **not** improve safety or reliability in this technology.

---

## Technology Comparison

| Feature | SKY130 | SCL-180 |
|------|------|------|
| Pad readiness after VDD | Delayed | Immediate |
| Reset reliability | POR-dependent | Pad-driven |
| Requirement for POR | Mandatory | Not required |

---

## Conclusion

For **SCL-180**, an **external reset pin alone is sufficient**.

Including an on-chip POR in this technology:
- Provides no functional advantage
- Increases the risk of simulation vs silicon mismatch
- Goes against standard industry design practices

Therefore, POR logic should be avoided in SCL-180-based designs unless explicitly required by system-level constraints.
