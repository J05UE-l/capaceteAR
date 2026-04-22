# CapaceteAR — Strategy 2: Transparent OLED (TOLED)

## 1. Project Overview

### 1.1 Goals

- Build a custom head-worn display showing virtual monitors using a transparent OLED
  (TOLED) panel placed directly in the optical path of each eye — no magnification
  optics, no combiner, no waveguide.
- Achieve a glasses-like or visor-like form factor suitable for extended workshop use
  under controlled indoor lighting.
- Drive the displays from an offboard PC or phone via USB-C tether (DisplayPort Alternate
  Mode + USB Power Delivery over the same cable).
- Validate the concept rapidly — TOLED's primary advantage is that the optics problem
  is eliminated, making this the fastest path to a working prototype.
- Establish a clear understanding of TOLED's brightness and contrast limitations before
  deciding whether to continue this strategy or pivot to Strategy 1.

### 1.2 Constraints

- Builder profile: intermediate maker. Custom PCBs, firmware, and basic mechanical design
  are within scope. Nanofab, precision optical grinding, and cleanroom processes are not.
- Budget envelope: $600–$1,600 for the display subsystem across both eyes.
- Phase 1 scope is strictly head-locked monitors. No spatial tracking, no hand input,
  no wireless, no audio.
- Critical limitation of this strategy: TOLED panels have low contrast and wash out in
  bright ambient light. This strategy is only viable for indoor or controlled lighting.
- All wearable elements are subject to review by the project's medical professional.
- Panel sourcing is the primary risk in this strategy. TOLED panels in small form factors
  are not commodity components. Sourcing lead time may be 4–12 weeks.

### 1.3 Non-Goals

- Outdoor use. TOLED in ambient daylight is not practical.
- Spatial AR / world-locked holograms / 6DoF tracking (Phase 2).
- Diffractive or geometric waveguide optics.
- Wireless compute offload. Phase 1 is USB-C tethered only.
- Hand tracking, eye tracking, voice input.
- Audio subsystem.
- Custom client software on the host. The glasses present as a standard external monitor.

---

## 2. System Architecture

### 2.1 High-Level Diagram

```
┌─────────────────────────┐         ┌──────────────────────────┐         ┌──────────────┐
│   Front module          │◄────────│   Rear module            │◄────────│   Offboard   │
│   (glasses / visor)     │  MIPI   │   (counterweight, back   │  USB-C  │   PC / phone │
│                         │   DSI   │    of head)              │  DP+PD  │              │
│  • TOLED panel ×2       │  or eDP │                          │────────►│              │
│    (transparent)        │────────►│  • Pi CM5 SoC            │  power  │  • Renders   │
│  • No optics required   │  power  │  • DP→panel bridge IC    │         │    video     │
│  • IMU (future)         │         │  • USB PD input          │         │  • No special│
│                         │         │  • Power distribution    │         │    software  │
└─────────────────────────┘         └──────────────────────────┘         └──────────────┘
```

The architecture is nearly identical to Strategy 1. The difference is entirely in the
front module: the optical barrel and micro-OLED assembly is replaced by a TOLED panel
mounted directly in an open frame in front of each eye.

### 2.2 Component Responsibilities

**Front module**
A minimal frame holding one TOLED panel per eye at the correct distance from the eye
(approximately 15–25 mm, depending on panel size). No optics housing required. The
frame must block stray light from the panel edges and position the panel flat and
perpendicular to the optical axis.

**Rear module**
Identical role to Strategy 1. Receives DP over USB-C from the host, converts via bridge
IC to the panel interface (MIPI DSI or eDP depending on panel chosen), distributes power.
Pi CM5 runs minimal init firmware.

**Offboard device**
Any host with USB-C DisplayPort Alternate Mode. Treats the glasses as a standard external
monitor. No special software required.

---

## 3. Hardware

### 3.1 Back Unit (Compute + Connectivity)

#### Options Considered

**Raspberry Pi CM5**
Same rationale as Strategy 1. MIPI DSI outputs on the CM5 match the interface of most
small TOLED panels. Well-documented Linux support. ~$55 module + carrier board.

**Rockchip RK3588**
Stronger compute, NPU, multiple DSI outputs. Better Phase 2 platform but adds bring-up
complexity unnecessary for Phase 1.

**Bridge IC only (no SoC)**
Cheaper, lower latency, but no upgrade path for Phase 2 LSR without a full board respin.

#### Tradeoffs

Identical reasoning to Strategy 1. The Pi CM5 survives into Phase 2 with a firmware
update. The bridge-IC-only path creates throwaway hardware. The cost delta does not
justify the Phase 2 respin.

#### Decision

**Raspberry Pi CM5** for the same reasons as Strategy 1. Phase 1 firmware is a minimal
display initialisation script. Phase 2 adds the LSR shader on the same board without
new hardware. Note: TOLED panels may use eDP rather than MIPI DSI depending on panel
choice — the carrier board design must accommodate both interface options or commit to
one after panel sourcing is confirmed.

---

### 3.2 Display Modules

#### Options Considered

**Visionox VM series small TOLED**
Visionox produces TOLED panels in the 1–3 inch diagonal range for industrial and
wearable markets. Some models have been made available in small quantities through
Chinese electronics distributors (LCSC, Alibaba verified suppliers). Approx. $150–$300
per panel. Interface: MIPI DSI. Transparency: ~35–45%. This is the most tractable
small-format TOLED sourcing path as of writing.

**WiseChip / Univision Technology TOLED panels**
WiseChip (WOLED series) offers transparent passive-matrix OLED panels in small sizes
(0.96"–2.8"). Passive-matrix limits resolution compared to active-matrix. Suitable only
for simple overlays and text; not adequate for rendering a full monitor view. $40–$100
per panel from distributor stock.

**LG / Samsung large-format TOLED (cut-down or teardown)**
Large format transparent OLED panels (15"+ diagonal) are produced for the retail signage
market. These cannot be cut — OLED panels shatter and short if scribed. Using them as-is
means a very different form factor (closer to a transparent visor than slim glasses).
Cost: $800–$2,500 for a single large panel. Not appropriate for a slim Phase 1 build
unless visor geometry is acceptable.

**Teardown from commercial transparent OLED products**
Some prototype and concept devices (Samsung Galaxy lineup concepts, select museum
display hardware) have used small TOLED panels that can occasionally be sourced as
replacement parts or from electronics recyclers. Availability is unpredictable and
cannot be relied upon for a planned build.

#### Tradeoffs

TOLED sourcing is the central risk of this entire strategy. The component is not
a commodity. Small active-matrix TOLED panels in the correct size range (1–3 inch,
high enough resolution to render readable text, MIPI interface, available in units
of two) are rare. Visionox is the best-identified path but requires relationship-building
with a Chinese distributor and realistic lead times of 4–12 weeks with minimum order
uncertainties.

WiseChip passive-matrix panels are easy to source but cannot render a usable monitor
view — ruling them out for this use case.

The large-format LG/Samsung panels shift the build toward a visor form factor, which
contradicts the slim glasses goal but is otherwise buildable and would provide good
image quality (higher brightness than small panels, larger display area).

There is a design decision embedded in this choice: if small active-matrix TOLED cannot
be sourced, the strategy forces either a visor form factor or a pivot to Strategy 1.

#### Decision

**Pursue Visionox small-format active-matrix TOLED (1.5"–2.5" diagonal) as primary.**
Begin sourcing immediately — this is the critical path item. If Visionox panels cannot
be secured within the project's timeline, trigger a decision gate: either accept a
visor form factor using a 3"–5" TOLED panel from an industrial supplier, or pivot the
Phase 1 display technology to Strategy 1 (micro-OLED + optics). Do not wait more than
6 weeks on sourcing before activating the contingency.

---

### 3.3 Optics

#### Options Considered

**No optics (bare TOLED in frame)**
The defining characteristic of this strategy. The TOLED panel is placed directly in
front of the eye. The eye perceives the panel's emissive pixels superimposed on the
real world visible through the transparent substrate. No combiners, no lenses, no
waveguides. Alignment tolerance is in the millimetre range.

**Anti-reflective coating on TOLED substrate**
Not an additional optical element but a surface treatment that reduces ghosting from
ambient light reflecting off the panel's glass surface back toward the eye. Some TOLED
panels ship with AR coating; if not, an aftermarket AR film ($20–$60 per panel) can
reduce this effect meaningfully.

**Polarising film overlay**
Darkens the perceived transparency of the real-world view by ~30% but dramatically
improves display contrast by rejecting ambient light that would otherwise wash out the
emissive pixels. Useful for bright workshop environments at the cost of the see-through
experience feeling tinted. Cost: negligible ($5–$15 per panel as cut film).

**Single lens for image magnification (hybrid approach)**
Placing a magnifying lens behind the TOLED would increase the apparent size of the
displayed image. However this defeats the core simplicity advantage of the strategy —
if a lens is added, Strategy 1 (which has much better display quality) becomes
preferable. Treat this as a downgrade path indicator, not a design option.

#### Tradeoffs

The no-optics path is both the simplest and the biggest quality compromise in this
strategy. Without magnification, the perceived display size is physically limited by
the panel dimensions at the eye-relief distance. A 2-inch panel at 20 mm from the eye
subtends roughly 57° diagonal — decent, but with no ability to adjust apparent size.
The contrast problem is real: TOLED panels achieve approximately 30:1 to 100:1 contrast
ratio in ambient light vs the 500:1 to 1000:1 achievable with an opaque panel behind
optics. Text and fine detail are substantially harder to read in TOLED than in
Strategy 1 under identical ambient conditions.

The AR coating and polarising film options are complementary partial mitigations.
Neither solves the fundamental physics of a low-contrast transparent display but
both meaningfully improve the practical experience.

#### Decision

**No magnification optics. TOLED panel placed directly in frame.** Apply AR coating
(panel-native if available, aftermarket film if not) to reduce ghosting. Include a
detachable polarising film overlay for use in brighter parts of the workshop where
contrast becomes limiting. Accept the contrast limitation as a known characteristic
of this strategy and use it as a key evaluation axis during Phase 1 testing.

---

### 3.4 Frame & Physical Structure

#### Options Considered

**FDM 3D-printed PLA/PETG**
Fast iteration. The frame for TOLED is simpler than Strategy 1 — no deep optical barrel
needed, just a flat bezel holding the panel at the correct eye distance. Cost: $5–$15
per frame print.

**SLA resin**
Better surface finish and dimensional accuracy. Less important than in Strategy 1 since
the panel-to-eye distance tolerance is looser (no focal plane to hit precisely). Worth
using for the panel bezel to avoid flex affecting panel alignment.

**Flat acrylic / polycarbonate sheet**
The TOLED panel is inherently flat. A laser-cut acrylic frame is a natural fit — clean
edges, optical clarity if the frame overlaps the panel active area, low cost ($10–$30).
Can be combined with 3D-printed connectors for the bridge and rear headband attachment.

**Visor geometry (if large panel path is taken)**
If the small panel sourcing fails and a 3"–5" panel is used, the frame geometry shifts
to a curved visor sitting further from the face (~30–40 mm). This still works mechanically
but changes the ergonomics significantly.

#### Tradeoffs

TOLED's form factor advantage is that the front module is mechanically trivial —
a flat panel in a flat bezel. FDM iteration for the overall frame plus laser-cut acrylic
for the precise panel bezels gives both speed and accuracy. The visor path, if triggered,
requires a new frame design but is not a fundamental blocker.

#### Decision

**FDM PLA** for main frame, bridge, and rear module housing. **Laser-cut acrylic** for
the panel bezels and any portions of the frame that border the active display area.
Design the frame with a modular bezel so the same headband/rear assembly can accept
either the small-panel or large-panel (visor) front module without a full respin.

---

### 3.5 Power System

#### Options Considered

**USB PD from tether cable (no onboard battery)**
Same rationale as Strategy 1. TOLED panels typically draw less power than micro-OLEDs
at equivalent brightness output (~0.5–1W per panel). Total system budget similar:
Pi CM5 ~5W + two panels ~1–2W + bridge IC ~1W = ~7–8W against a 15W USB PD supply.

**Onboard LiPo (Phase 2)**
Required for wireless operation. Not justified for tethered Phase 1.

#### Tradeoffs

Identical to Strategy 1. The tether exists for video; power over the same cable
eliminates the battery design from the Phase 1 scope entirely.

#### Decision

**USB PD over the tether cable, no onboard battery for Phase 1.** FUSB302 or IP2721
sink negotiation IC on the rear module PCB. Plan LiPo + BMS for Phase 2 alongside
the wireless work.

---

## 4. Connectivity & Offload

### 4.1 WiFi Architecture

#### Options Considered

- WiFi 6 (802.11ax)
- WiFi 6E (6 GHz band)
- No WiFi — USB-C tether only

#### Tradeoffs

Identical to Strategy 1. Wireless offload is a Phase 2 concern. Phase 1 is wired.
LSR must be implemented before wireless latency can be tolerated.

#### Decision

**No WiFi for Phase 1.** Reserve an M.2 E-key footprint on the rear module PCB for
an Intel AX210 module to be populated in Phase 2. No antenna routing or wireless
firmware in Phase 1.

---

### 4.2 Supported Client Devices

Identical to Strategy 1. Any device supporting USB-C DisplayPort Alternate Mode
and USB PD at 15W:

- Laptops running Windows, macOS, or Linux with USB-C DP alt mode.
- Android phones: Samsung Galaxy S20+, Google Pixel 6+, USB4-capable Android devices.
- iPhone: not supported (no DP alt mode).

---

### 4.3 Communication Protocol

#### Options Considered

**DisplayPort Alternate Mode over USB-C**
Preferred. Host GPU drives the TOLED panels as a standard external monitor. Zero
encoding latency, no codec, no special software. Bridge IC converts DP→MIPI DSI
(or DP→eDP, depending on panel interface).

**eDP (embedded DisplayPort) direct**
Some TOLED panels use eDP rather than MIPI DSI. eDP is a point-to-point protocol
derived from DP, typically used for laptop internal displays. A DP→eDP bridge IC
(e.g. ANX7483, PS8461) handles the conversion. Functionally equivalent to the
MIPI DSI path from the host and software perspective.

**MIPI DSI direct**
Same as Strategy 1. Preferred if the chosen TOLED panel has a MIPI DSI interface.

**ALVR / virtual display streaming**
Phase 2 only. Not appropriate for tethered Phase 1.

#### Tradeoffs

The panel interface (MIPI DSI vs eDP) is unknown until the specific TOLED panel is
confirmed during sourcing. Both are well-supported by available bridge ICs. The
rear module PCB should be designed with solderable options for both, or the bridge IC
selection deferred until the panel is in hand. From the host and software perspective
the protocol is identical — DP alt mode over USB-C.

#### Decision

**DisplayPort Alternate Mode (DP 1.4) over USB-C.** Bridge IC selection deferred
until the TOLED panel interface is confirmed during sourcing. Primary candidates:
TC358779XBG for MIPI DSI, ANX7483 or PS8461 for eDP. Rear PCB should route both
options if the board spin happens before panel confirmation.

---

## 5. Software Stack

### 5.1 Operating System

#### Options Considered

Identical options to Strategy 1: Raspberry Pi OS, Ubuntu 24.04, Buildroot/Yocto.

#### Tradeoffs

Identical reasoning. Phase 1 firmware is minimal — TOLED panels, like micro-OLEDs,
require only an initialisation sequence at boot. The OS complexity difference between
strategies is negligible.

#### Decision

**Raspberry Pi OS (64-bit, Bookworm)** for Phase 1. Migrate to Ubuntu 24.04 for
Phase 2 when Monado/OpenXR integration begins. The one difference from Strategy 1:
the panel init sequence will differ because TOLED panels often require different
power rail sequencing than micro-OLEDs (TOLED cathode/anode voltages are typically
higher and must be ramped in a specific order to avoid panel damage). The init script
must be written from the panel datasheet, not assumed from Strategy 1.

---

### 5.2 Rendering Pipeline

#### Options Considered

**Pure hardware passthrough — no device-side rendering**
Bridge IC handles DP→panel interface in hardware. Pi CM5 is not in the video path.
Runs only panel init script at boot.

**Brightness compensation layer (TOLED-specific consideration)**
Because TOLED panels have significantly lower contrast than opaque displays, a
software step on the host side that boosts contrast, increases font weight, and
adjusts colour balance for TOLED's colour shift could meaningfully improve readability.
This is a host-side filter, not device-side rendering — it can run as a display ICC
profile or a simple GPU shader on the offboard machine.

**On-device LSR shader (Phase 2)**
Same as Strategy 1. LSR is added in Phase 2 firmware without hardware changes.

#### Tradeoffs

The pure passthrough path is correct for Phase 1. The TOLED brightness compensation
layer is worth implementing as a host-side ICC profile early — it costs nothing in
terms of latency and could make the difference between the display being usable and
frustrating during the evaluation phase. It does not belong in the device firmware.

#### Decision

**Hardware passthrough only for Phase 1.** Additionally, create a custom ICC display
profile for the TOLED output on the host machine that boosts contrast, compensates
for TOLED's colour temperature shift, and increases minimum font weight for legibility.
This is a 1–2 hour host configuration task with measurable usability impact.

---

### 5.3 Client Applications (Laptop / Phone)

#### Options Considered

Same options as Strategy 1: native OS monitor extension vs custom virtual desktop client
vs SteamVR/Monado.

#### Tradeoffs

Same reasoning as Strategy 1. Fastest validation path is native OS monitor extension.
TOLED has an additional consideration: because contrast is low, the default white
backgrounds of most desktop environments will look washed out. Testing with a dark-mode
desktop theme and high-contrast UI from day one is recommended.

#### Decision

**Native OS monitor extension for Phase 1.** Configure the host display output to the
TOLED panels with a dark desktop theme and the custom ICC profile described in section
5.2. No custom client software. A virtual desktop client with TOLED-specific UI
optimisation (forced dark mode, contrast enhancement, reduced motion) is tracked as
a non-blocking parallel effort.

---

## 6. Open Questions

- Can the target TOLED panel (Visionox or equivalent) be sourced in quantities of two
  within the project timeline? This is the critical path item and must be resolved in
  the first two weeks of the project. If not, activate the decision gate.
- What is the exact power rail sequencing and timing requirement for the chosen TOLED
  panel? TOLED panels are more sensitive to power-on sequence violations than standard
  OLEDs. This information must be extracted from the panel datasheet or manufacturer
  application note before the rear module PCB is finalised.
- At what ambient lux level does the TOLED display become unreadable? Testing needed
  in actual workshop conditions. A workshop under normal LED lighting is approximately
  500–1000 lux — borderline for TOLED. This is the primary pass/fail criterion for
  this strategy.
- Is the panel-to-eye distance of ~20 mm achievable in a glasses form factor given the
  panel's physical dimensions and connector placement? TOLED panels in the 1.5"–2.5"
  range may have edge-mounted flex connectors that push the panel further from the eye
  than desired.
- What is the pixel density at 20 mm eye relief for the chosen panel? Target for
  readable text is ≥40 PPD (pixels per degree). Calculate: PPD = (panel_horizontal_res /
  panel_horizontal_mm) × eye_relief_mm × tan(1°).

---

## 7. Insights & Lessons Learned

*(To be filled during build)*

---

## 8. Resources & References

- Visionox product portfolio (visionox.com) — transparent OLED panel specs
- WiseChip WOLED series datasheet (wisechip.com.tw) — passive matrix reference
- Raspberry Pi CM5 datasheet and IO board schematics (raspberrypi.com)
- Toshiba TC358779XBG product brief — DisplayPort 1.4 to MIPI DSI bridge
- ANX7483 / PS8461 product briefs — DP to eDP bridge ICs
- FUSB302 USB PD sink controller datasheet (onsemi.com)
- USB Implementers Forum, *USB Type-C Alternate Mode for DisplayPort* specification
- MIPI Alliance DSI specification v1.3 (mipi.org, free with registration)
- Covestro Bayfol HX photopolymer datasheet — for Phase 2 HOE reference
- Colucci & Velazquez, "Transparent display technologies" — overview of TOLED contrast
  physics and ambient light rejection strategies
