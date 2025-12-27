# Reset Pad Analysis – SCL-180 vs SKY130

## Purpose of This Document

This document analyzes the reset pad characteristics of the **SCL-180 PDK** and explains why an **on-chip Power-On Reset (POR)** circuit is **not required**, in contrast to designs based on **SKY130** technology.

---

## Reset Pad Characteristics in SCL-180

In the **SCL-180** process, reset pads exhibit stable and predictable behavior immediately after power is applied:

- Input pads are directly powered by **VDD**
- No internal enable signal is needed to activate pad functionality
- There is no POR-gated or masked input path
- Reset pins are **asynchronous**
- Reset pins are operational immediately at power-up

This ensures that the reset signal can be safely asserted **during the power ramp itself**, without waiting for additional internal circuitry.

---

## Clean Reset Requirement vs Reset Origin

From a digital design perspective, flip-flops only require:

- A clean reset assertion
- A clean reset deassertion

They are **independent of**:
- The source of the reset signal
- Whether reset is generated internally (POR) or externally (pad)

As long as reset timing constraints are met, the **origin of the reset is irrelevant** to correct digital operation.

---

## Why SKY130 Needed an On-Chip POR

In the **SKY130** PDK, pad behavior during power-up was not inherently guaranteed:

- Pad enable states could be undefined
- Internal pull-ups and pull-downs were unreliable during early power-on
- Reset pads were unsafe during initial voltage ramp

Because of this uncertainty, an on-chip POR was necessary to:
- Mask unstable pad behavior
- Hold reset until power became stable
- Prevent incorrect logic initialization

---

## Why SCL-180 Does NOT Require POR

The **SCL-180** technology eliminates these concerns:

- Pads become electrically valid immediately after VDD is applied
- No dependency on internal enable logic
- Reset can be asserted directly at power-on

As a result, adding a POR circuit provides **no additional functional safety** in this process.

---

## Technology Comparison Summary

| Feature                          | SKY130        | SCL-180        |
|----------------------------------|---------------|---------------|
| Pad readiness after VDD          | Delayed       | Immediate     |
| Reset reliability                | POR-dependent | Pad-driven    |
| Requirement for internal POR     | Mandatory     | Not required  |

---

## Conclusion

For **SCL-180**, an external reset pin alone is **architecturally sufficient**.

Including an on-chip POR in this technology:
- Offers no functional advantage
- Can introduce simulation-to-silicon mismatch
- Goes against established industry best practices

Therefore, POR insertion is unnecessary and should be avoided in SCL-180-based designs.

