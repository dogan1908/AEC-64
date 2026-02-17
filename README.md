# AEC-64: A Self-Describing 64-Bit Binary Format for Atomic Electron Configurations

**Author:** Dogan Balban — Independent Researcher  
**Status:** Preprint (v1.0, 2026)  
**License:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)

---

## Overview

Electron configurations are traditionally expressed in textual notation —
human-readable, but inefficient for computational systems.

**AEC-64** is a compact, fixed-width 64-bit binary format that encodes the
complete electron configuration of any chemical element (Z = 1–118) in a
single machine word.

```
┌─────────────────┬────────────────────────────────────────────────────────┐
│  8-bit Header   │                  56-bit Payload                        │
│  Layout Index   │  7p │ 7s │ 6d │ 6p │ 6s │ 5f │ ... │ 2s │ 1s         │
└─────────────────┴────────────────────────────────────────────────────────┘
```

The **Layout Index** makes the format self-describing: no external schema
file or lookup table is needed to decode any AEC-64 word.

---

## Key Properties

| Property | Description |
|---|---|
| **Compact** | 64 bits — fits in a single CPU register |
| **Self-describing** | Layout Index encodes the decoding schema |
| **Lossless** | Full configuration recoverable exactly |
| **Exception-safe** | Encodes Aufbau exceptions (Cr, Cu, Au …) correctly |
| **Verifiable** | Atomic number Z derived as Σeᵢ — built-in consistency check |
| **Extensible** | 255 additional Layout Indices for ions, excited states, compressed payloads |
| **O(1) decoding** | 39 bit-shift + AND operations — no S-boxes, no key schedule |

---

## Why Not Just Store Z?

Storing only the atomic number Z (7 bits) and reconstructing the configuration
via the Madelung rule fails silently for **~17% of all elements**:

| Element | Z | Madelung predicts | Actual ground state |
|---|---|---|---|
| Chromium | 24 | [Ar] 3d⁴ 4s² | [Ar] **3d⁵ 4s¹** |
| Copper | 29 | [Ar] 3d⁹ 4s² | [Ar] **3d¹⁰ 4s¹** |
| Palladium | 46 | [Kr] 4d⁸ 5s² | [Kr] **4d¹⁰ 5s⁰** |
| Gold | 79 | [Xe] 4f¹⁴ 5d⁹ 6s² | [Xe] 4f¹⁴ **5d¹⁰ 6s¹** |

AEC-64 stores the experimentally verified configuration directly —
no reconstruction algorithm involved.

---

## Quick Example: Sodium (Na, Z = 11)

Configuration: `1s² 2s² 2p⁶ 3s¹`

```
Header (Layout 0):   00000000
Payload (56 bits):   000 00 0000 000 00 0000 0000 000 00 0000 0000 000 00 0000 000 01 110 10 10
                     7p  7s  6d  6p  6s  5f  5d  5p  5s  4f  4d  4p  4s  3d  3p  3s  2p  2s  1s
```

Verification: Σeᵢ = 1 + 2 + 6 + 2 = 11 = Z ✓

---

## Encoding / Decoding (Python)

```python
def encode_aec64(layout_index, subshell_values, bit_lengths):
    payload = ""
    for value, bits in zip(subshell_values, bit_lengths):
        if value < 0 or value >= (1 << bits):
            raise ValueError(f"Value {value} exceeds {bits}-bit capacity.")
        payload += format(value, f'0{bits}b')
    assert len(payload) == 56
    return format(layout_index, '08b') + payload

def decode_aec64(aec64_bits, layout_def):
    layout_index = int(aec64_bits[:8], 2)
    payload = aec64_bits[8:]
    counts, pos = [], 0
    for bits in layout_def:
        counts.append(int(payload[pos:pos+bits], 2))
        pos += bits
    return layout_index, counts, sum(counts)  # (H, [e_i], Z)
```

---

## Canonical Layout (Index 0)

Subshell order in payload (MSB → LSB):

```
7p(3) · 7s(2) · 6d(4) · 6p(3) · 6s(2) · 5f(4) · 5d(4) · 5p(3) · 5s(2)
· 4f(4) · 4d(4) · 4p(3) · 4s(2) · 3d(4) · 3p(3) · 3s(2) · 2p(3) · 2s(2) · 1s(2)
= 56 bits total
```

Number in parentheses = bit width. Configuration space:
**N_valid = 3⁷ · 7⁴ · 11⁴ · 15² ≈ 1.73 × 10¹⁰** syntactically valid payloads.

---

## Performance

AEC-64 decoding requires exactly **39 elementary operations** (bit shifts + AND masks)
and runs in **O(1)** — independent of input value.

| Operation | Latency | Complexity |
|---|---|---|
| **AEC-64 decode** | **~6.7 ns** | **O(1)** |
| AES-128 (hardware) | ~20 ns | O(r) |
| AES-128 (software) | ~200 ns | O(r) |
| DES (software) | ~250 ns | O(r) |

> Note: Benchmarks are analytical estimates for a 3 GHz single-threaded processor.
> AEC-64 is not a cryptographic format — the 64-bit width does not imply
> cryptographic complexity. Decoding is a fixed sequence of bit projections.

---

## Repository Contents

```
AEC64.pdf       — Full manuscript (18 pages)
AEC64.tex       — LaTeX source
README.md       — This file
```

---

## Cite This Work

If you use or reference AEC-64, please cite:

```bibtex
@misc{balban2026aec64,
  author    = {Balban, Dogan},
  title     = {{AEC-64}: A Self-Describing 64-Bit Binary Format
               for Atomic Electron Configurations},
  year      = {2026},
  note      = {Independent research preprint},
  url       = {https://github.com/dogan1908/AEC-64}
}
```

## Cite This Work

If you use or reference AEC-64, please cite:

```bibtex
@misc{balban2026aec64,
  author    = {Balban, Dogan},
  title     = {{AEC-64}: A Self-Describing 64-Bit Binary Format
               for Atomic Electron Configurations},
  year      = {2026},
  note      = {Independent research preprint},
  url       = {https://github.com/Dogan1908/AEC-64}
}
```

---

## License

This work is licensed under
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
You are free to share and adapt the material for any purpose, provided
appropriate credit is given.
