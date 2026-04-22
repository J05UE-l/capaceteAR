# CapaceteAR — Phase 1 Display Strategy Comparison

---

## 1. Quick Reference

| | Strategy 1 | Strategy 2 | Strategy 3 |
|---|---|---|---|
| **Technology** | Micro-OLED + compound eyepiece | Transparent OLED (TOLED) | Micro-projector + compound eyepiece |
| **Cost (two eyes)** | $600–$800 | $600–$1,600 | $700–$1,100 (DLP2010) / $1,100–$1,600 (DLP3010) |
| **Resolution per eye** | 1280×720 | 720–1080p (panel dependent) | 854×480 (DLP2010) / 1280×720 (DLP3010) |
| **Optical elements in path** | 2 (panel + eyepiece) | 1 (panel only) | 3–4 (projector + mirror + diffuser + eyepiece) |
| **Sourcing risk** | Low | **Critical** | Very low |
| **Build complexity** | Medium | Low | **High** |
| **Phase 2 upgrade path** | Camera-passthrough AR | Dead end | ✅ See-through AR (element swap) |
| **Recommended for Phase 1** | ✅ Yes | ⚠️ Conditional | ⚠️ Conditional |

---

## 2. Optical Paths

### Strategy 1 — Micro-OLED + Compound Eyepiece

```
  ┌─────────────────┐     ┌─────────────────────┐
  │   Micro-OLED    │     │   Compound eyepiece  │
  │   panel         │────►│   (Plössl / wide-    │────►  👁  Eye
  │   (self-emit.)  │     │    angle, $30–$100)  │
  └─────────────────┘     └─────────────────────┘
   sits at back focal          magnifies image,
   plane of eyepiece           corrects aberrations
   (precise placement,
    mm tolerance)

  Total elements in path: 2
  Light efficiency: ~65–70% through eyepiece
```

The panel is the light source. It sits fixed at the eyepiece's back focal plane.
Everything in the image path is directly in front of the eye.

---

### Strategy 2 — Transparent OLED

```
                   ┌──────────────────────────┐
  Real world ─────►│   TOLED panel            │────►  👁  Eye
                   │   (transparent substrate  │
                   │    with emissive pixels)  │
                   └──────────────────────────┘
                    transmits ~35–45% of
                    ambient light through;
                    emits image pixels on top

  Total elements in path: 1
  No optics required
  Real-world light always passing through panel
```

The simplest possible optical path. The display is also the window.
No focal distance, no magnification — what you see is the panel at face distance.

---

### Strategy 3 — Micro-Projector + Compound Eyepiece

```
  ┌──────────┐    fold    ┌──────────┐    ┌──────────────────┐
  │  DLP /   │   mirror   │ Diffuser │    │  Compound        │
  │  LCOS    │────╲  ╱───►│ (at back │───►│  eyepiece        │────►  👁  Eye
  │  engine  │     ╲╱     │  focal   │    │  (Plössl /       │
  └──────────┘            │  plane)  │    │   wide-angle)    │
   offset above/          └──────────┘    └──────────────────┘
   beside eye              catches and        magnifies
                           diffuses beam

  Total elements in path: 3–4
  Light efficiency: ~30–40% (diffuser loss ~50%, eyepiece ~70%)

  ─ ─ ─ ─ ─ ─ ─  Phase 2 upgrade (see-through AR)  ─ ─ ─ ─ ─ ─ ─

  ┌──────────┐    fold    ┌──────────────┐    ┌──────────────────┐
  │  DLP /   │   mirror   │  Partial     │    │  Compound        │
  │  LCOS    │────╲  ╱───►│  mirror      │───►│  eyepiece        │────►  👁  Eye
  │  engine  │     ╲╱     │  combiner    │    │                  │
  └──────────┘    (same)  │  (replaces   │    └──────────────────┘
                          │   diffuser)  │
                          └──────┬───────┘
                                 │ real world
                                 ▼ passes through
                            transparent substrate

  Phase 2 change: swap diffuser for combiner. All other hardware unchanged.
```

The projector sits off-axis. A fold mirror bends the beam toward the eyepiece.
A diffuser at the focal plane catches the projected image for the eyepiece to magnify.

---

## 3. Cost Breakdown

### Per-eye component costs

| Component | Strategy 1 | Strategy 2 | Strategy 3 (DLP2010) | Strategy 3 (DLP3010) |
|---|---|---|---|---|
| Display source | $150–$250 (Sony ECX339A) | $300–$800 (TOLED panel) | $200–$250 (DLP2010 eval kit) | $350–$450 (DLP3010 eval kit) |
| Optics | $30–$100 (eyepiece) | — (none) | $30–$100 (eyepiece) | $30–$100 (eyepiece) |
| Diffuser / combiner | — | — | $40–$80 (engineered diffuser) | $40–$80 |
| Fold mirror | — | — | $10–$30 | $10–$30 |
| Driver / bridge IC | $15–$30 | $15–$30 | — (built into eval kit) | — (built into eval kit) |
| Housing (SLA barrel) | $10–$20 | $5–$10 | $10–$20 | $10–$20 |
| **Per-eye total** | **$205–$400** | **$320–$840** | **$290–$480** | **$440–$680** |

### Full two-eye system + rear module

| Item | Strategy 1 | Strategy 2 | Strategy 3 (DLP2010) | Strategy 3 (DLP3010) |
|---|---|---|---|---|
| Front module (×2 eyes) | $410–$800 | $640–$1,680 | $580–$960 | $880–$1,360 |
| Rear module (shared) | $150–$300 | $150–$300 | $150–$300 | $150–$300 |
| Frame + assembly | $30–$80 | $30–$80 | $50–$120 | $50–$120 |
| **Total prototype** | **$590–$1,180** | **$820–$2,060** | **$780–$1,380** | **$1,080–$1,780** |

> **Note:** Strategy 2's range is wide because the TOLED panel cost is highly uncertain
> until sourcing is confirmed. The low end assumes a successful low-cost procurement;
> the high end reflects premium industrial panel pricing.

---

## 4. Image Quality

### Display metrics

| Metric | Strategy 1 | Strategy 2 | Strategy 3 (DLP2010) | Strategy 3 (DLP3010) |
|---|---|---|---|---|
| **Resolution (per eye)** | 1280×720 | Panel-dependent (720–1080) | 854×480 | 1280×720 |
| **Panel brightness** | ~2000 nit | ~300–600 nit | 30–50 ANSI lm | 50–100 ANSI lm |
| **Brightness at eye (est.)** | ~1200–1400 nit | ~100–250 nit | ~10–17 lm | ~17–35 lm |
| **Native contrast ratio** | ~500:1–1000:1 | ~30:1–100:1 (ambient) | ~1000:1 | ~1000:1 |
| **Pixels per degree @ 50° FOV** | ~26 PPD | Variable | ~17 PPD ⚠️ | ~26 PPD |
| **Color gamut** | ~100% DCI-P3 | ~80–90% sRGB | ~100% sRGB (LED) | ~100% sRGB |
| **Burn-in risk** | Yes (OLED) | Yes (OLED) | None (DMD) | None (DMD) |

> **PPD threshold for comfortable text reading: ~20 PPD.**
> Strategy 3 with DLP2010 falls below this. Strategy 1 and Strategy 3 with DLP3010 clear it.

### Real-world readability in workshop conditions (~500 lux ambient)

| Condition | Strategy 1 | Strategy 2 | Strategy 3 |
|---|---|---|---|
| Dense text (code, docs) | ✅ Excellent | ⚠️ Marginal (low contrast) | ⚠️ Acceptable (DLP3010) / Poor (DLP2010) |
| Large UI elements | ✅ Excellent | ✅ Acceptable | ✅ Good |
| Video playback | ✅ Excellent | ⚠️ Washed out | ✅ Good |
| Dark mode UI | ✅ Excellent | ✅ Best case for TOLED | ✅ Good |
| Outdoor / high ambient | ✅ Good | ❌ Unusable | ✅ Good |

---

## 5. Sourcing & Supply Chain

### Risk assessment

| Factor | Strategy 1 | Strategy 2 | Strategy 3 |
|---|---|---|---|
| **Primary component** | Sony ECX339A | TOLED panel (Visionox or equiv.) | TI DLPDLCR2010EVM |
| **Distributor type** | Industrial (Symmetry, Richardson) | Specialty / Chinese industrial | Major (Mouser, Digi-Key) |
| **In stock today?** | Usually, 1–2 week lead | ❌ Unknown, 4–12 week lead | ✅ Yes |
| **Minimum order** | 1 unit | Unknown (potentially 10–50 units) | 1 unit |
| **Replacement availability** | Medium (industrial channel) | ❌ Low (model discontinued risk) | ✅ High (TI ecosystem) |
| **Documentation quality** | Application note available | Minimal for small panels | ✅ Thorough (TI design guide) |
| **Community precedent** | Medium (DIY headset builds) | Almost none | Medium (DLP maker projects) |

### Sourcing risk summary

```
Strategy 1   ██████████░░░░░░░░░░  Low risk
Strategy 2   ████████████████████  CRITICAL — project-blocking risk
Strategy 3   ███░░░░░░░░░░░░░░░░░  Very low risk
```

---

## 6. Build Complexity

### Assembly steps per eye (front module only)

**Strategy 1**
1. Source and verify Sony ECX339A panel
2. Print SLA optical barrel
3. Seat eyepiece in barrel at correct back-focal-distance
4. Mount panel at focal plane of eyepiece
5. Wire MIPI DSI flex cable to driver board
6. Validate focus on bench

**Strategy 2**
1. Source and verify TOLED panel
2. Print FDM bezel frame
3. Mount TOLED panel in bezel at correct eye-relief distance
4. Wire panel to driver board
5. Validate display on bench

**Strategy 3**
1. Source and verify DLP eval kit
2. Design and print SLA optical barrel (diffuser + eyepiece + mirror mount)
3. Mount fold mirror at correct angle
4. Position diffuser at eyepiece back focal plane
5. Align DLP projection beam onto diffuser centre (requires bench alignment with live image)
6. Tune DLPC3430 keystone correction over I2C
7. Mount eyepiece at correct distance from diffuser
8. Validate image quality on bench
9. Check thermal profile under 30-minute sustained operation

### Alignment tolerance comparison

| | Strategy 1 | Strategy 2 | Strategy 3 |
|---|---|---|---|
| **Critical dimension** | Panel-to-eyepiece distance | Panel-to-eye distance | Projector aim, diffuser position, eyepiece distance |
| **Tolerance** | ±1–2 mm | ±3–5 mm | ±0.5–1 mm (diffuser placement) |
| **Iterative prints needed (est.)** | 3–5 | 2–3 | 5–8 |
| **Bench equipment needed** | Ruler + eye | Ruler + eye | Alignment target + live image |

### Complexity rating

```
Strategy 1   ████████░░░░░░░░░░░░  Medium
Strategy 2   █████░░░░░░░░░░░░░░░  Low (if panel sourced)
Strategy 3   █████████████████░░░  High
```

---

## 7. Form Factor & Ergonomics

| Factor | Strategy 1 | Strategy 2 | Strategy 3 |
|---|---|---|---|
| **Front depth (nose to front)** | ~40–55 mm (eyepiece barrel) | ~8–15 mm (panel + bezel) | ~20–30 mm (eyepiece only, projector offset) |
| **Front module mass (est. per eye)** | ~25–40 g | ~15–25 g | ~60–90 g (eval kit) / ~20–35 g (future custom) |
| **Mass distribution** | Centered on nose bridge | Centered on nose bridge | Offset to temple (better balance) |
| **Thermal output (front module)** | Low (OLED ~0.5W/eye) | Low (TOLED ~0.5W/eye) | **Medium-high (DLP LEDs ~2–3W/eye)** |
| **Ventilation required?** | No | No | Yes — housing must breathe |
| **Slim glasses feel** | ❌ Barrel protrudes | ✅ Closest to glasses | ⚠️ Bulk offset to temple |

> Strategy 3's thermal output is its most significant ergonomic risk. At ~2–3W of heat
> per eye in an enclosed housing positioned near the face, thermal management is a real
> design constraint, not an afterthought.

---

## 8. Power Consumption

| Consumer | Strategy 1 | Strategy 2 | Strategy 3 |
|---|---|---|---|
| Display (per eye) | ~0.5–0.8W | ~0.5–1.0W | ~2.5–3.5W (LED illumination) |
| Display (both eyes) | ~1–1.6W | ~1–2W | **~5–7W** |
| Pi CM5 | ~5W | ~5W | ~5W |
| Bridge / driver ICs | ~1W | ~1W | — (built into eval kit) |
| **Total system** | **~7–8W** | **~7–9W** | **~10–13W** |
| **USB PD profile needed** | 15W (5V/3A) | 15W (5V/3A) | **27W (9V/3A)** |

---

## 9. Phase 2 Upgrade Path

This is the most important long-term differentiator between the three strategies.

### What Phase 2 requires per strategy

| | Strategy 1 | Strategy 2 | Strategy 3 |
|---|---|---|---|
| **Phase 2 AR mode** | Camera-passthrough | ❌ No viable path | ✅ See-through (native) |
| **Display hardware change** | Full replacement (new display tech) | Full replacement | ❌ None — element swap only |
| **Optics change** | Partial (add cameras + combiner) | Full redesign | Add combiner ($100–$300/eye) |
| **Firmware change** | LSR shader (same) | LSR shader (same) | LSR shader (same) |
| **Rear module change** | None | None | None |
| **Phase 2 effort estimate** | Major (new display subsystem) | Not viable | Minor (optics swap + combiner alignment) |

### Phase 2 optical upgrade (Strategy 3 only)

```
  PHASE 1                              PHASE 2

  [Projector] → [mirror] → [diffuser] → [eyepiece] → 👁
                              ↑
                         REMOVE THIS
                              |
                              ▼
  [Projector] → [mirror] → [combiner] → [eyepiece] → 👁
                                ↑
                           Real world
                           passes through
                           glass substrate

  Hardware changed: 1 element per eye
  Everything else: identical
```

### Phase 2 readiness scores

```
See-through AR readiness:
  Strategy 1   ████░░░░░░░░░░░░░░░░  Poor  (needs new display tech)
  Strategy 2   ░░░░░░░░░░░░░░░░░░░░  None  (TOLED dead end for spatial AR)
  Strategy 3   ████████████████████  Best  (designed for this upgrade)

Camera-passthrough AR readiness:
  Strategy 1   ████████████████░░░░  Good  (just add outward cameras + LSR)
  Strategy 2   ████████░░░░░░░░░░░░  Poor  (contrast problems compound in AR use)
  Strategy 3   ████████████████░░░░  Good  (same as Strategy 1 for this path)
```

---

## 10. Risk Register

| Risk | Strategy | Severity | Likelihood | Mitigation |
|---|---|---|---|---|
| Panel cannot be sourced in small quantities | S2 | 🔴 Critical | High | Decision gate at week 2; pivot to S1 if unresolved |
| Panel discontinued before Phase 2 | S2 | 🔴 Critical | Medium | No mitigation — inherent to specialty TOLED market |
| DLP2010 resolution inadequate for text | S3 | 🟡 Major | Medium | Pre-validate PPD on bench before housing; upgrade to DLP3010 |
| DLP eval kit overheats in enclosed housing | S3 | 🟡 Major | Medium | Bench thermal test before finalising housing; add ventilation slots |
| Eyepiece barrel misalignment (focus off) | S1, S3 | 🟡 Major | Low | SLA printing + iterative bench focus test |
| TOLED contrast inadequate in workshop lighting | S2 | 🟡 Major | High | Accept as known limitation; validate before committing |
| Fold mirror angle introduces keystone distortion | S3 | 🟠 Minor | Medium | DLPC3430 hardware keystone correction via I2C |
| Sony ECX339A stock disruption | S1 | 🟠 Minor | Low | Kopin Lightning as drop-in fallback |
| MIPI DSI bringup difficulty on Pi CM5 | S1 | 🟠 Minor | Low | Reference code available from Project North Star community |
| USB PD 27W not available on host device | S3 | 🟠 Minor | Low | Validate with target laptop before PCB design |

---

## 11. Weighted Scoring Matrix

Criteria weights reflect importance for the stated Phase 1 goal (virtual monitors, workshop use)
and Phase 2 viability.

| Criterion | Weight | S1 Score | S2 Score | S3 (DLP2010) | S3 (DLP3010) |
|---|---|---|---|---|---|
| Image quality (contrast + resolution) | 25% | 9 | 4 | 6 | 8 |
| Sourcing reliability | 20% | 8 | 3 | 9 | 9 |
| Build complexity (inverse — simpler = better) | 15% | 8 | 9 | 5 | 5 |
| Cost efficiency | 15% | 9 | 6 | 8 | 6 |
| Phase 2 compatibility | 15% | 6 | 2 | 10 | 10 |
| Form factor | 10% | 6 | 9 | 7 | 7 |
| **Weighted total** | | **7.90** | **5.05** | **7.35** | **7.65** |

```
Weighted score (out of 10):

Strategy 1          ████████████████████████████████████████  7.90
Strategy 3 DLP3010  ██████████████████████████████████████░░  7.65
Strategy 3 DLP2010  █████████████████████████████████████░░░  7.35
Strategy 2          █████████████████████████░░░░░░░░░░░░░░░  5.05
```

> **Caveat:** These weights assume see-through AR is the Phase 2 goal. If Phase 2 is
> camera-passthrough AR (opaque displays + outward cameras, Vision Pro style), raise
> Strategy 1's Phase 2 score to 9 and lower Strategy 3's to 7. This would push Strategy 1
> decisively ahead of Strategy 3 at all DLP configurations.

---

## 12. Decision Guide

```
START
  │
  ▼
What is your Phase 2 AR mode?
  │
  ├─► Camera-passthrough (Vision Pro style — outward cameras, opaque displays)
  │     │
  │     ▼
  │   Is text readability the top priority?
  │     ├─► YES ──► STRATEGY 1 ✅
  │     │           (best PPD, best contrast, lowest risk)
  │     │
  │     └─► Is build simplicity more important than image quality?
  │               ├─► YES ──► STRATEGY 2 ⚠️
  │               │           (only if TOLED sourcing is confirmed)
  │               └─► NO  ──► STRATEGY 1 ✅
  │
  └─► See-through AR (HoloLens style — you look through the glass)
        │
        ▼
      Is the Phase 1 budget above $1,100 for two eyes?
        │
        ├─► YES ──► STRATEGY 3 (DLP3010) ✅
        │           (best Phase 2 upgrade path + adequate Phase 1 resolution)
        │
        └─► NO
              │
              ▼
            Can you accept ~17 PPD text sharpness in Phase 1?
              ├─► YES ──► STRATEGY 3 (DLP2010) ✅
              │           (cheapest path to see-through Phase 2)
              └─► NO  ──► STRATEGY 1 ✅ + plan display respin at Phase 2
                          (best Phase 1 experience, acknowledge Phase 2 cost)
```

---

## 13. Head-to-Head Summary

### Strategy 1 wins when:
- Optimal text readability in Phase 1 is non-negotiable
- Phase 2 will be camera-passthrough (not see-through)
- Fastest path to a working prototype is the priority
- Budget is under $1,000 for two eyes

### Strategy 2 wins when:
- The absolute thinnest possible front module is required before Phase 2 funding
- Panel sourcing has already been confirmed with a real supplier
- The build is primarily for demonstration rather than daily use

### Strategy 3 wins when:
- Phase 2 is definitely see-through AR
- Form factor matters more than front-module depth
- Long-term durability (no OLED burn-in) is valued
- Budget allows for the DLP3010 to close the resolution gap with Strategy 1

---

## 14. Final Recommendation

**If you have not yet decided on Phase 2 display mode:** build Strategy 1.
It is the best Phase 1 experience by a meaningful margin, it de-risks the project at
the point where you are learning the most, and it does not permanently foreclose any
Phase 2 path — camera-passthrough AR on top of Strategy 1's hardware is a well-trodden
road. The cost of switching display technology at Phase 2 is real but known.

**If you have committed to see-through Phase 2:** build Strategy 3 with the DLP3010.
Accept the higher cost as an investment in the upgrade path. The Phase 2 transition
for Strategy 3 is an afternoon of optics work. The Phase 2 transition for Strategy 1
(if see-through is the goal) is a display technology decision plus a new front module.

**Strategy 2 is not recommended** as the primary build path for this project. Its sourcing
risk is project-blocking, its contrast performance is inadequate for a daily-use virtual
monitor, and it has no viable see-through Phase 2 path. It may serve as a thin
demonstration unit once one of the other strategies is already working.
