[README.md](https://github.com/user-attachments/files/33743381/README.md)
# Primary Cementing Simulator (V5)

**File:** `Primary_Cementing_V5.html`
**Author:** S. Siddhu | Central FEU
**Type:** Single-file HTML / CSS / JavaScript (no external libraries, no network calls)
**Purpose:** Training and visualization tool for a lead/tail primary casing cement job with a two-plug (bottom wiper / solid top) system. Every fluid position and every hydrostatic pressure shown is computed from the cumulative volume pumped.

> This is a teaching and review aid, not a job design tool. See *Assumptions & Limitations*.

---

## 1. How to Run

1. Open `Primary_Cementing_V5.html` in any current browser.
2. Edit inputs in the left panel. Results update as you type.
3. Click steps **1 → 8**, or **▶ RUN FULL JOB** to play the whole sequence. Completed steps show ✓.
4. **↺ RESET** returns to standby (open hole full of mud, no casing).
5. **☀ Light view / ☾ Dark view** (top right of the schematic) switches color themes. Light view is recommended for projectors and printing. The choice is remembered in that browser.

Steps can be clicked in any order; the well state for each step is computed directly from the volumes, so jumping ahead gives the same picture as playing through.

---

## 2. Inputs

### Well Construction
| Field | Units | Default |
|---|---|---|
| Hole TD | ft | 8,000 |
| Bit Size / OH | in | 8.75 |
| Casing Shoe | ft | 7,980 |
| Float Collar | ft | 7,940 |
| Open-Hole Excess | % | 0 |

**Open-hole excess** increases the annular capacity: annular capacity = gauge annular capacity × (1 + excess/100). The whole annulus is treated as open hole. The panel shows the equivalent effective hole diameter, √(OD² + annular capacity × 1029.4), and the schematic draws the hole at that diameter.

### Casing
| Field | Units | Default |
|---|---|---|
| Casing Preset | — | 7" @ 26.00 lb/ft |
| Casing OD | in | 7.000 |
| Nominal Weight | lb/ft | 26.00 |
| ID Override | in | blank (formula used) |

**Casing ID is calculated from OD and weight:**

ID = √(OD² − W / 2.667)

Example: 4-3/4" 20.0# → √(22.5625 − 7.4991) = √15.0634 = **3.8812"**.

Choosing a preset fills OD and weight. Editing OD or weight directly switches the preset to *Custom*, so any combination can be entered. Each dropdown entry also shows its formula ID. If you enter an **ID Override** (for example, a published or measured ID), it replaces the formula value; leave it blank to use the formula.

**Note on accuracy of the formula:** the formula treats the entered weight as the weight of plain steel pipe. Nominal casing weight also includes the couplings, so the formula ID comes out slightly smaller than published API IDs:

| Casing | Formula ID | Published ID | Difference | Capacity effect |
|---|---|---|---|---|
| 5-1/2" 15.50# | 4.9435" | 4.950" | −0.007" | −0.26% |
| 7" 26.00# | 6.2651" | 6.276" | −0.011" | −0.35% |
| 9-5/8" 36.00# | 8.8962" | 8.921" | −0.025" | −0.56% |
| 13-3/8" 54.50# | 12.5879" | 12.615" | −0.027" | −0.43% |

For 7" 26# to 7,940 ft, this makes the displacement volume about 1.1 bbl smaller (302.8 vs 303.8 bbl). Use ID Override when the exact published ID matters.

**Preset list** (edit `CASING_SETS` in the script to add sizes): common API 5CT OD/nominal-weight combinations, grouped by OD, plus the 4-3/4" 20.0# example.

| OD | Nominal weights (lb/ft) |
|---|---|
| 4-1/2" | 9.50, 10.50, 11.60, 13.50, 15.10 |
| 4-3/4" | 20.00 |
| 5" | 11.50, 13.00, 15.00, 18.00 |
| 5-1/2" | 14.00, 15.50, 17.00, 20.00, 23.00 |
| 6-5/8" | 20.00, 24.00, 28.00, 32.00 |
| 7" | 17.00, 20.00, 23.00, 26.00, 29.00, 32.00, 35.00, 38.00 |
| 7-5/8" | 24.00, 26.40, 29.70, 33.70, 39.00 |
| 8-5/8" | 24.00, 28.00, 32.00, 36.00, 40.00, 44.00, 49.00 |
| 9-5/8" | 32.30, 36.00, 40.00, 43.50, 47.00, 53.50 |
| 10-3/4" | 32.75, 40.50, 45.50, 51.00, 55.50, 60.70, 65.70 |
| 11-3/4" | 42.00, 47.00, 54.00, 60.00 |
| 13-3/8" | 48.00, 54.50, 61.00, 68.00, 72.00 |
| 16" | 65.00, 75.00, 84.00 |
| 18-5/8" | 87.50 |
| 20" | 94.00, 106.50, 133.00 |

Verify any weight against the API 5CT tables or the mill sheet before relying on it for a specific well.

### Fluids
| Field | Units | Default |
|---|---|---|
| Mud density | ppg | 10.5 |
| Displacement density | ppg | 10.6 |
| Spacer volume / density | bbl / ppg | 50 / 10.5 |
| Lead volume / density | bbl / ppg | 250 / 12.5 |
| Tail volume / density | bbl / ppg | 60 / 15.8 |

### Pressure Assumptions
| Field | Units | Default | Use |
|---|---|---|---|
| Friction allowance | psi | 300 | Added to surface pump pressure only while pumping. Placeholder value; see 4.5 |
| Bump over FCP | psi | 500 | Pressure applied above final circulating pressure to bump the top plug |

All defaults live in the `FIELDS` array; that is the single source for both the form and the calculations.

---

## 3. Input Validation

**Errors (simulation is blocked and fields are outlined red):**
- Blank or non-numeric field
- Depths or bit size ≤ 0; densities outside 0–30 ppg; negative volumes or pressures
- Shoe deeper than TD
- Float collar at or below the shoe
- Casing OD ≥ bit size (casing does not fit the hole)
- Casing weight too heavy for the OD (formula gives no real ID)
- ID override ≤ 0 or ≥ OD

A value of 0 is accepted (for example, 0 bbl spacer).

**Warnings (simulation runs; shown on one line above the schematic, hover for detail). Input errors also appear there in red.**
- Lead, tail, or spacer circulated out at surface, with the volume
- Tail volume smaller than the shoe track (lead left in shoe track)
- Free-fall / U-tubing predicted: casing column outweighs annulus by more than the friction allowance, with the stage and the maximum imbalance
- Final annulus hydrostatic not greater than casing hydrostatic (floats carry no differential)

---

## 4. Calculation Method

### 4.1 Capacities
| Quantity | Formula |
|---|---|
| Casing ID (in) | √(OD² − W / 2.667), unless overridden |
| Casing capacity (bbl/ft) | ID² / 1029.4 |
| Annular capacity (bbl/ft) | (Bit² − OD²) / 1029.4 × (1 + excess/100) |
| Casing volume to shoe | Shoe × casing capacity |
| Annular volume, shoe to surface | Shoe × annular capacity |
| Shoe track volume | (Shoe − Collar) × casing capacity |
| Displacement volume | Collar × casing capacity |

### 4.2 Fluid tracking (plug flow)
The casing and annulus are treated as one continuous flow path measured in barrels:

- `s = 0` to casing volume → inside casing, surface down to shoe (depth = s ÷ casing capacity)
- casing volume to casing + annular volume → annulus, shoe back up to surface (depth = shoe − annular distance ÷ annular capacity)
- beyond that → returned at surface

Fluids are pumped in order: spacer, lead, tail, displacement. The well starts full of mud. A fluid parcel that entered the casing when the cumulative pumped volume was *u* sits at path position *s = V − u* after *V* barrels have been pumped. This gives the exact location of every interface at every moment, including:

- how far the spacer travels down the casing,
- when each fluid turns the shoe and starts up the annulus,
- how much of each fluid is pushed out at surface.

**Plugs:** the bottom plug rides the spacer/lead interface and stops at the float collar (diaphragm then ruptures). The top plug rides the tail/displacement interface and bumps when displacement volume reaches the collar. Plugs are treated as zero volume.

**Default case example:** 50 bbl of spacer occupies the top 1,311 ft of a 304 bbl casing, so it stays entirely inside the casing during Step 3. It reaches the shoe only after about 254 bbl of lead has been pumped behind it, and by the bump all 50 bbl has been circulated out at surface.

### 4.3 Pressures (all referenced to the shoe)
| Quantity | Method |
|---|---|
| Casing hydrostatic (Pin) | 0.052 × Σ (density × height) of fluids inside casing, surface to shoe |
| Annulus hydrostatic (Pout) | 0.052 × Σ (density × height) of fluids in annulus, shoe to surface |
| Hydrostatic Δ | Pout − Pin |
| Surface pump pressure while pumping | max(0, Δ + friction allowance) |
| Final circulating pressure (FCP) | max(0, final Δ + friction allowance) |
| Bump pressure | FCP + bump-over input |
| Float check | Surface pressure bled to 0; floats hold the final Δ |
| U-tube flowback if floats fail | Volume that would flow back out of the casing, with the annulus level dropping, until Pin = Pout (solved by bisection) |

When Δ + friction is negative while pumping, the casing column is heavier than the annulus. Surface pressure is shown as 0 and the state line reads **FREE-FALL / U-TUBING**.

### 4.4 Default-case results (7" 26#, formula ID 6.2651", 8.75" hole, 0% excess)
| Output | Value |
|---|---|
| Casing capacity | 0.03813 bbl/ft |
| Annular capacity | 0.02678 bbl/ft |
| Casing volume to shoe | 304.3 bbl |
| Annular volume | 213.7 bbl |
| Shoe track volume | 1.5 bbl |
| Displacement volume | 302.8 bbl |
| Tail TOC | 5,796 ft |
| Lead TOC | 0 ft (94.8 bbl lead + 50 bbl spacer out at surface) |
| Pin / Pout at shoe | 4,409 / 5,562 psi |
| Differential held by floats | 1,152 psi |
| FCP / Bump | 1,452 / 1,952 psi |
| Max free-fall imbalance | 1,077 psi (during tail pumping) |
| U-tube flowback if floats fail | 30.5 bbl |

With 40% excess the annular capacity becomes 0.03749 bbl/ft (effective hole 9.359"). The tail top rises to 6,420 ft, and only 9.3 bbl of lead goes out at surface.

### 4.5 Friction Allowance — Basis and Caveat

**Where the 300 psi default comes from.** It is not derived. V4 contained a hidden fixed "+300 psi" in its circulating-pressure formula; V5 exposed it as an editable input and kept 300 as the default. Treat it as a placeholder.

**Check of the order of magnitude** (7" 26# in 8-3/4" hole, 7,980 ft; Bingham-plastic laminar flow; illustrative rheology, not lab data):

| Fluid | Plastic viscosity (cP) | Yield point (lb/100 ft²) |
|---|---|---|
| Mud | 15 | 10 |
| Lead | 40 | 15 |
| Tail | 60 | 20 |
| Displacement | water-like | — |

| Pump rate | Casing interior (displacement) | Annulus, mud only | Annulus at end of job (lead + tail) |
|---|---|---|---|
| 2 bpm | ~2 psi | ~280 psi | ~520 psi |
| 3 bpm | ~4 psi | ~300 psi | ~590 psi |
| 5 bpm | ~10 psi | ~350 psi | ~740 psi |

Annular gradient used (psi/ft): PV × v / (1000 × (D_h − D_p)²) + YP / (200 × (D_h − D_p)), with v in ft/s and diameters in inches. Pipe flow checked for turbulence.

**What this shows:**
- Friction inside the casing is negligible. The large ID and low velocity give only a few psi.
- Almost all friction is in the annulus. A 7" casing in an 8-3/4" hole leaves only 0.875" radial clearance.
- In laminar flow the yield-point term, YP / (200 × gap), dominates and **does not decrease with pump rate**. Slowing the pumps reduces annular friction only modestly.
- 300 psi is in the right range for circulating mud in this geometry, but it **understates** friction once the annulus is full of cement at the bump.
- Larger clearances (for example 9-5/8" in 12-1/4") give a fraction of these values. No single fixed allowance is right across casing sizes.

**Effects on the model:**
- The static hydrostatic differential (Pout − Pin, 1,152 psi in the default case) does **not** depend on friction.
- Surface pump pressure while pumping, FCP, bump pressure, and the free-fall check all **do** depend on the allowance. Because friction is added only while pumping, the allowance shifts where free-fall is flagged.
- Planned improvement: replace the flat allowance with a calculated friction model using a pump-rate input and PV/YP for each fluid, applied to each fluid's tracked position in the casing and annulus.

---

## 5. Display

**Warning line (above schematic):** overflow, free-fall, and shoe-track warnings on one line; input errors in red.

**Fill-volume strip (top of schematic):** gauge volumes and the excess implied by the fluid inputs.

| Cell | Calculation |
|---|---|
| Casing fill (to shoe) | Shoe × casing capacity; sub-line shows displacement to collar |
| Annulus fill (gauge) | (Shoe − Target TOC) × gauge annular capacity (bit size, no excess) |
| Target TOC | User input, ft; 0 = surface. Must be shallower than the shoe |
| Cement to annulus | Lead + Tail − shoe track volume |
| Excess over gauge | (Cement to annulus ÷ Annulus fill − 1) × 100. Green = over gauge, red = short |
| vs. OH excess input | Cement to annulus − Annulus fill × (1 + OH excess/100), in bbl over (+) or short (−) |

Default case: 308.5 bbl of cement against a 213.7 bbl gauge annulus to surface gives **+44.4%** (94.8 bbl over). That is the 94.8 bbl of lead the model shows going out at surface with 0% OH excess.

**Main wellbore (left):** full depth at true vertical scale, with a depth axis. Hole and casing widths are drawn proportional to their diameters. Labels to the right give the depth range of each fluid inside the casing (csg) and in the annulus (ann), plus any volume circulated out at surface.

**Shoe Track Detail (inset):** a magnified view (×N shown in the title) from just above the float collar to TD, showing the float collar and check valve, landed/bumped plugs, shoe track contents, guide shoe, and rathole.

**Right column:**
- Fluid legend
- **Surface Pump Pressure** gauge
- **Hydrostatic Δ @ Shoe** gauge (reads negative when the casing side is heavier)
- **Total Volume Pumped** gauge
- Stage box: stage name, stage volume, FCP, bump pressure, friction allowance

Gauge ranges scale automatically to the job.

**Readout strip (bottom):** Pin, Pout, Δ, total pumped, and system state.

**Animation timing:** pumping steps run for times proportional to their volumes (the largest stage takes about 12 s), and timing is clock-based rather than tied to screen refresh rate.

---

## 6. Assumptions & Limitations

- Vertical well (MD = TVD).
- Excess is one uniform percentage applied to the full annulus; there is no caliper input or zone-by-zone washout.
- No previous casing string; the full annulus is open hole.
- Rathole below the shoe stays full of mud and does not take cement.
- Plug flow: no mixing, channeling, or contamination at interfaces.
- No slurry compressibility, pump efficiency, or surface-line volume.
- Friction is a single user-entered allowance, not calculated from rheology and rate (see 4.5).
- During free-fall, the model still assumes returns match pumped volume; it flags the condition but does not model the vacuum or the lag.
- Float check assumes floats hold; the "if floats fail" flowback is computed separately.

---

## 7. Changes from V4

**Bugs fixed**
- Lead/spacer overflow to surface is now reported with volumes instead of being silently clamped to 0 ft.
- Casing that does not fit the hole, collar at or below the shoe, and shoe below TD are now blocked with error messages.
- A 0 input is accepted. Defaults come from a single table, so the form and the calculations no longer disagree.
- All pressures use the shoe as the reference depth, so the readouts no longer jump between steps.
- The oversized mud fill that drew a bar into the gauge area is removed.
- The gauge alarm threshold no longer applies to the volume gauge.
- The element ID `lblHydAnn=` is renamed `lblHydAnn`.
- The "PROFILE GAP" label is removed.

**Model and display changes**
- Fluid positions in every step are computed from pumped volume. V4 used fixed fractions; for example, Step 3 always drew the spacer to 40% of the casing.
- Lead stays visible inside the casing during displacement.
- Gauges and readouts are computed at every frame. V4 used scripted increments.
- The Bump-over-FCP (default 500 psi) and friction allowance (default 300 psi) are now inputs. V4 had fixed hidden values of +1,650 psi and +300 psi.
- Added a free-fall/U-tubing check and the U-tube flowback-if-floats-fail volume.
- The canvas fills the full panel height and adds a depth axis and the Shoe Track Detail inset.
- Added RUN FULL JOB, step completion marks, and live input updates.
- Added open-hole % excess.
- Casing ID is now computed from OD and nominal weight (ID = √(OD² − W/2.667)), with an optional ID override.
- The preset list is expanded to 69 common API 5CT OD/weight combinations from 4-1/2" to 20", plus a Custom entry.
- Added Light/Dark themes with higher-contrast fluid colors and text, and larger canvas fonts.
- Compacted the left panel (one-line warnings with hover detail, two-column step buttons).
- Added the fill-volume strip with Target TOC and implied % excess.
- Layout fits the screen height; the left panel scrolls independently.
- Added a credit line.
