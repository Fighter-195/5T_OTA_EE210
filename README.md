# Constant-g<sub>m</sub> 5T-OTA — 10 mS Target (EE210 Analog Electronics Course Project)

A fully differential **5-transistor Operational Transconductance Amplifier (OTA)** designed in a
**180 nm CMOS** process (GPDK180, Cadence Virtuoso), targeting a transconductance of
**$g_m = 10$ mS with less than 1 % variation over −20 °C to +100 °C**.

> **Author:** Parth Dhamija (Roll No. 240733) · Department of Electrical Engineering, IIT Kanpur
> **Course:** EE210 — Analog Electronics · **Course Project:** Constant-$g_m$ Bias (5T-OTA, 10 mS target)
> **Result:** target $g_m$ held to **< 0.1 % error** across the full temperature range.

---

## The problem

The transconductance of an OTA directly sets bandwidth and linearity in the filters, data
converters, and sensor front-ends it drives. In a conventionally (constant-current) biased OTA,
$g_m$ drifts strongly with temperature because carrier mobility $\mu$ falls as temperature rises —
so a naive design that hits 10 mS at room temperature drops well out of spec at 100 °C.

**Goal:** hold $g_m = 10$ mS to within 1 % from −20 °C to +100 °C, in a *fully differential*
topology (which additionally requires common-mode feedback to keep the high-impedance output
nodes from drifting out of saturation).

## The approach — in four phases

The design was built up iteratively; each phase in the report addresses one limitation of the
previous one.

| Phase | What was added | Why | Outcome |
|-------|----------------|-----|---------|
| **1 — Baseline 5T OTA** | Diode-connected PMOS loads, sized via the **$g_m/I_D$** methodology (moderate inversion, $g_m/I_D \approx 17$ V⁻¹, $I_{branch} \approx 606$ µA, $I_{tail} \approx 1.2$ mA) | Validate that 10 mS is reachable at nominal temperature | Hits 10 mS at 27 °C, but **$g_m$ drifts badly** with temperature |
| **2 — Constant-$g_m$ bias + ideal CMFB** | **Beta-multiplier** bias core (ties $g_m$ to a stable resistor $R_s$), active loads, ideal VCVS common-mode feedback | Cancel mobility-driven drift; a resistor is far more temperature-stable than a transistor | $g_m$ now **nearly flat** vs. temperature |
| **3 — Optimized bias scaling** | Beta-multiplier ratio **$K$ raised 4 → 16**, plus a bias-distribution / mirror network into the OTA | Larger, more stable $R_s$ while keeping 10 mS → lower sensitivity to resistor variation | Residual drift further reduced; OTA input pair ≈ flat at 10 mS |
| **4 — Full integration + real CMFB** | Ideal VCVS replaced by a transistor-level **DDA-based CMFB** (differential-difference amplifier, non-loading) | A physically realizable design that regulates output common-mode without resistively loading (and degrading) the output | **< 1 % $g_m$ variation**, stable operation, high output impedance |

### Key idea 1 — Beta-multiplier constant-g<sub>m</sub> bias

A self-biased loop (symmetric PMOS mirror + asymmetric NMOS pair of ratio $K$, with source
degeneration resistor $R_s$) forces:

$$g_m = \frac{2}{R_s}\left(1 - \frac{1}{\sqrt{K}}\right)$$

Because $g_m$ is pinned to $R_s$ — a resistor, which is far more temperature-stable than
transistor parameters — the mobility-driven drift is largely cancelled.

### Key idea 2 — Scaling K to trade for a bigger, more stable R<sub>s</sub>

The sensitivity of $g_m$ to $R_s$ is

$$\frac{dg_m}{dR_s} = -\frac{2}{R_s^{2}}\left(1 - \frac{1}{\sqrt{K}}\right),$$

i.e. it shrinks as $R_s$ grows. Raising **$K$ from 4 to 16** changes the relation from
$g_m = 1/R_s$ to $g_m = 1.5/R_s$, allowing a ~50 % larger (and thus more stable) $R_s$ while
still delivering 10 mS.

### Key idea 3 — DDA-based CMFB that doesn't load the output

A differential-difference amplifier senses the output common-mode at high-impedance gates,
compares it to $V_{ref}$ (through a replica source-follower for DC matching), amplifies the error,
and drives the PMOS load gates via a compensated loop (MOS cap $C_B$ + triode PMOS $M_{c9}$). This
holds the output common-mode fixed **without adding resistive loading**, preserving OTA gain.

## Result

The fully integrated transistor-level design holds the OTA input-pair transconductance at
**≈ 10.01 mS** (see Table I in the report) with **< 0.1 % error** across −20 °C → +100 °C, while
the CMFB keeps the output common-mode stable and the output impedance high. The report also
includes a **failure analysis** (process/geometric mismatch, voltage-headroom limits, and the
beta-multiplier's zero-current degenerate startup state / CMFB phase-margin concerns).

---

## Repository layout

```
5T_OTA_EE210/
├── README.md                              ← this file
├── report/
│   └── EE210_5T_OTA_gm_report.pdf         ← full IEEE-style project report (phases 1–4, tables, analysis)
├── design/
│   └── OTA_5T_210/                        ← Cadence Virtuoso design library (as submitted)
│       ├── cds.lib                        ← library definitions (references IITK GPDK180 PDK)
│       ├── cdsinfo.tag                    ← Cadence library tag
│       ├── data.dm                        ← OpenAccess data-management database
│       ├── schematic/                     ← the OTA schematic cellview (sch.oa)
│       └── state_submission/              ← saved ADE test-bench / simulation state (analyses, outputs, vars)
└── results/
    └── final_annotated_schematic.png      ← final integrated schematic with DC operating-point annotations (Fig. 8)
```

## Design summary (final integrated system)

| Block | Role |
|-------|------|
| **Constant-$g_m$ bias core** | $K = 16$ beta-multiplier; sets a temperature-stable $g_m$ reference from $R_s$ |
| **Bias distribution** | Current mirrors replicate/scale the reference into the OTA tail and branches |
| **5T OTA core** | NMOS input pair ($L = 540$ nm $= 3 \times L_{min}$, $m = 10$), PMOS active loads, NMOS tail |
| **DDA CMFB loop** | Senses & regulates output common-mode without resistive loading |

Selected nominal operating points (from report, Table I — OTA core):

| Device | $I_{DS}$ | $V_{GS}$ | $V_{DS}$ | $g_m$ | Region |
|--------|------|------|------|------|--------|
| NM0 / NM3 (input pair) | 602.1 µA | 603.9 mV | 609.1 mV | **10.01 mS** | sat |
| PM1 / PM2 (loads) | 602.1 µA | −640.9 mV | −894.8 mV | 6.35 mS | sat |
| NM2 (tail) | 1204.2 µA | 515.6 mV | 296.1 mV | 21.86 mS | sat |

---

## Opening the design

This is a **Cadence Virtuoso / Spectre** project built against IIT Kanpur's **GPDK180** PDK. It is
included as an archival snapshot of the submitted work.

- `cds.lib` references PDK paths (`/home/vlsi_IITK/GPDK/gpdk180_v3.3/...`) that exist on the IITK
  VLSI servers, so the schematic opens in that environment. To open elsewhere, edit `cds.lib` to
  point at a local `gpdk180` install and add this library via *Library Manager*.
- The saved ADE setup in `state_submission/` reproduces the temperature-sweep test bench used to
  characterize $g_m(T)$.
- Stale Cadence lock files (`*.cdslck`) from the submission machine have been removed.

For readers without a Cadence setup, the **report PDF** and the **annotated schematic** in
`results/` fully document the circuit, methodology, equations, and results.
