# CapaceteAR — Strategy 1: Micro-OLED + Magnification Optics

## 1. Project Overview

### 1.1 Goals

- Build a custom head-worn display capable of showing virtual monitors using micro-OLED panels
  behind compound magnification optics, one per eye.
- Achieve a glasses-like form factor suitable for extended workshop use.
- Drive the displays from an offboard PC or phone via a single USB-C tether (DisplayPort
  Alternate Mode + USB Power Delivery over the same cable).
- Validate the full display stack — panel brightness, FOV, eye relief, IPD adjustment, and
  wearability — before committing to Phase 2 spatial AR work.
- Keep the rear module hardware-forward-compatible with Phase 2 so the board does not need
  to be replaced when late-stage reprojection (LSR) firmware is added.

### 1.2 Constraints

- Builder profile: intermediate maker. Custom PCBs, firmware, and basic bench optics are
  within scope. Nanofab, precision optical grinding, and cleanroom processes are not.
- Budget envelope: $400–$2,000 for the display and optics subsystem across both eyes.
- Form factor: glasses-like. No visor or helmet geometry for Phase 1.
- Phase 1 scope is strictly head-locked monitors. No spatial tracking, no hand input,
  no wireless, no audio.
- All wearable elements are subject to review by the project's medical professional.

### 1.3 Non-Goals

- Spatial AR / world-locked holograms / 6DoF tracking (Phase 2).
- Diffractive or geometric waveguide optics.
- Wireless compute offload. Phase 1 is USB-C tethered only.
- Hand tracking, eye tracking, voice input.
- Outdoor use. Phase 1 targets controlled indoor lighting.
- Audio subsystem.
- Custom client software on the host. The glasses present as a standard external monitor.

---

## 2. System Architecture

### 2.1 High-Level Diagram

```
┌────────────────────────┐         ┌──────────────────────────┐         ┌──────────────┐
│   Front module         │◄────────│   Rear module            │◄────────│   Offboard   │
│   (glasses body)       │  MIPI   │   (counterweight, back   │  USB-C  │   PC / phone │
│                        │   DSI   │    of head)              │  DP+PD  │              │
│  • Micro-OLED ×2       │────────►│                          │────────►│              │
│  • Compound eyepiece ×2│  power  │  • Pi CM5 SoC            │  power  │  • Renders   │
│  • IMU (future)        │         │  • DP→MIPI bridge IC     │         │    video     │
│                        │         │  • USB PD input          │         │  • No special│
│                        │         │  • Power distribution    │         │    software  │
└────────────────────────┘         └──────────────────────────┘         └──────────────┘
```

### 2.2 Component Responsibilities

**Front module**
Mechanical housing for the two optical assemblies. Each eye holds one micro-OLED panel at
the back focal plane of one compound eyepiece, mounted in a 3D-printed barrel. The barrel
seats into an IPD-adjustable bridge on the frame. An IMU is mounted here but unused in
Phase 1 — it is wired to the rear module for Phase 2 bring-up.

**Rear module**
Acts as a display adapter in Phase 1. It receives a DisplayPort signal over USB-C from the
host, converts it via a bridge IC to two MIPI DSI streams (one per eye), and distributes
5V power to the panels and front module. The Pi CM5 is present but runs a minimal init
script only — it is not in the video path for Phase 1. In Phase 2 it will perform LSR.

**Offboard device**
Any host with USB-C DisplayPort Alternate Mode output. Treats the glasses as a standard
extended display. No drivers or custom software required. Handles all rendering.

---

## 3. Hardware

### 3.1 Back Unit (Compute + Connectivity)

#### Options Considered

**Raspberry Pi CM5**
Debian-based Linux, excellent MIPI DSI support, two native MIPI DSI outputs (one per eye
without a splitter), large community, USB-C can be configured as a device exposing a DP
sink. Cost: ~$55 for the module plus a custom or off-the-shelf carrier board.

**Rockchip RK3588 (e.g. Radxa CM5 or custom)**
Substantially more compute, built-in NPU suitable for future hand and pose tracking, multiple
MIPI CSI/DSI lanes. More complex bring-up, thinner community documentation for custom
carrier boards. Cost: $80–$150 on available SBCs. Better long-term fit for Phase 2 but
adds complexity to Phase 1.

**Bridge IC only (no SoC — e.g. Toshiba TC358779)**
A pure hardware path: DP in, MIPI DSI out, no Linux, no boot time. Lowest latency, cheapest
bill of materials (~$15 for the IC). Cannot run LSR or any firmware. Rear module would need
a full board respin for Phase 2.

#### Tradeoffs

The bridge-IC-only path is cheapest and simplest for Phase 1 but creates throwaway hardware
— the entire rear board must be redesigned for Phase 2 when LSR is needed. The Pi CM5 adds
~$55 and some PCB complexity but means the rear module survives into Phase 2 with only a
firmware update. RK3588 is the best Phase 2 platform but introduces unnecessary bringup
risk for a Phase 1 validation build.

#### Decision

**Raspberry Pi CM5.** Phase 1 firmware is a minimal Linux image that initialises the two
MIPI DSI outputs and exposes itself as a USB-C DP sink to the host. The Pi is idle in the
video path — the bridge IC handles the DP→MIPI conversion in hardware. Phase 2 adds an LSR
GLSL shader running on the Pi's VideoCore VII GPU without a hardware respin.

---

### 3.2 Display Modules

#### Options Considered

**Sony ECX339A**
0.5" diagonal, 1280×720, silicon OLED (micro-OLED), ~2000 nit peak brightness. MIPI DSI
interface with documented timing. Approx. $150–$250 per panel from industrial distributors
(Symmetry Electronics, Richardson RFPD, Sony B2B). Community precedent exists — used in
several open-source headset builds with published bringup notes.

**Kopin Lightning 2**
0.49", 1280×1024, slightly higher PPI. Designed specifically for headset use. $200–$400 per
panel, harder to source in quantities below 50 units. MIPI interface.

**eMagin WUXGA**
Professional grade, >3000 nit, used in military and medical headsets. $300–$600+ per panel,
effectively limited to B2B channels with NDA. Unjustified for a prototype.

**Olightek / Wisechip (Chinese industrial suppliers)**
$80–$150 per panel, lower documentation quality, variable binning. Usable as a budget
fallback if ECX339A supply is disrupted.

#### Tradeoffs

The ECX339A sits at the intersection of price, documentation, and community knowledge.
Its 2000 nit brightness handles indoor ambient light comfortably. The 1280×720 resolution
per eye is sufficient for reading text on a virtual monitor at normal viewing distances.
Kopin's higher resolution is a meaningful upgrade but the sourcing friction is high for
a first prototype. eMagin is overengineered for Phase 1.

#### Decision

**Sony ECX339A**, two units ($300–$500 total). Source via Symmetry Electronics or equivalent
industrial distributor. A custom or evaluaton MIPI DSI driver board for this panel will be
needed; reference schematics are available in the Sony application note package.

---

### 3.3 Optics

#### Options Considered

**Single aspherical lens per eye**
One catalogue lens (Thorlabs, Edmund Optics) per eye positioned at the back focal distance
from the panel. Simplest possible assembly. Cost: $20–$80 per lens. Produces significant
edge distortion and chromatic aberration across the FOV. Acceptable for initial bench
testing only, not for sustained wearable use.

**Repurposed compound eyepiece (telescope Plössl or wide-angle)**
A 2–4 element eyepiece repurposed as a magnifier. Available from astronomy suppliers
(GSO, Celestron, Agena AstroProducts) for $30–$100 per eyepiece. Standard cylindrical
barrel fits cleanly into a 3D-printed housing. Corrects the majority of spherical and
chromatic aberrations visible at the edges of the FOV. Achievable FOV: 45–60° depending
on focal length chosen and eye relief. This is the same optical approach used by Project
North Star.

**Custom 3-element cemented design**
Designed in lens design software (Zemax, or open-source openLLT/Oslo Light) to match
exactly the ECX339A panel dimensions, target eye relief (18–22 mm), and desired FOV.
Fabricated by an optical prototyping shop (Optimax, II-VI Precision Optics). Cost:
$400–$1500 per element in prototype quantities, 4–8 week lead time.

**Birdbath combiner (eyepiece + curved beam-splitter)**
Adds a partial-mirror element to fold the optical path forward, reducing the front-to-back
depth of the assembly by roughly 40%. Adds $100–$300 per side for the curved beam-splitter
and ~50% optical efficiency loss at the combiner surface. Alignment is significantly more
demanding.

#### Tradeoffs

For Phase 1 validation the goal is to confirm that the display concept is worth pursuing,
not to achieve final optical quality. A compound eyepiece provides image quality that
is good enough to use for hours and costs under $100 for both eyes. The alignment
tolerance is in the millimetre range — achievable with a 3D-printed barrel and
iterative test prints. The birdbath's form factor improvement is not worth the added
complexity until the optical design is locked. Custom optics are the correct end state
but belong in Phase 2 after the concept is validated.

#### Decision

**Repurposed Plössl or wide-angle eyepiece** per eye (e.g. GSO 25mm, ~$25–$40 each).
Print a custom PLA barrel that seats the ECX339A at the eyepiece's back focal plane.
Validate eye relief and IPD range before committing barrel dimensions. Custom optics
are explicitly planned for Phase 2.

---

### 3.4 Frame & Physical Structure

#### Options Considered

**FDM 3D-printed PLA/PETG**
Fast iteration, $5–$20 per full frame print, sufficient rigidity for desk and workshop
use. Not suitable as a finished wearable (mass, surface finish, skin contact properties)
but ideal for Phase 1 prototyping.

**SLA resin**
Higher dimensional accuracy, better surface finish. Important for the optical barrels where
a 0.5 mm error in the lens-to-panel distance directly affects focus. Cost: $10–$30 per
optical assembly print.

**Machined aluminium**
Best thermal management and structural rigidity. Cost: $200–$800 per piece in prototype
quantities, 1–3 week lead time. Appropriate for the rear module enclosure in a final
revision but unjustified for Phase 1.

**Carbon fibre sheet + 3D-printed connectors**
Lightweight and stiff. Assembly complexity is higher, modification requires cutting new
sheet. Better suited to Phase 2 when the design is more stable.

#### Tradeoffs

Iteration speed dominates Phase 1. The number of frame prints needed before landing on
correct IPD range, weight balance, and ergonomics is likely five to ten. FDM at $10 per
attempt is far preferable to SLA at $30 or machining at $500. The exception is the
optical barrels, where dimensional accuracy matters enough to justify SLA.

#### Decision

**FDM PLA** for the main frame, headband, and rear module housing. **SLA resin** for
the optical barrel assemblies (lens seat and panel mount per eye). Revisit material choice
for Phase 2 when the form factor is frozen.

---

### 3.5 Power System

#### Options Considered

**USB PD from tether cable (no onboard battery)**
The host PC provides 15W over USB-C (USB PD profile 2). Pi CM5 draws ~5W under light
load; two ECX339A panels draw ~0.5–0.8W each; bridge IC and ancillaries draw ~1W.
Total budget: ~8W nominal against a 15W supply — comfortable margin. No battery,
no BMS, no charging circuit required.

**Single LiPo cell (3.7V, 5000 mAh) with BMS**
Required for Phase 2 when the USB-C tether is removed for wireless operation.
Adds ~85–110g to rear module mass, requires a BMS IC, a USB PD input charger,
and a 5V/3.3V regulator chain. All of this is straightforward but adds 3–4 weeks
of PCB work not justified for Phase 1.

**Dual LiPo in series (7.4V)**
More energy density, but adds BMS complexity with no Phase 1 benefit.

#### Tradeoffs

Phase 1 is tethered. The tether cable already exists for video. Routing power over the
same cable eliminates the battery design entirely from the Phase 1 critical path. The
Pi CM5 and panel power requirements are well within USB PD 15W. Adding a battery now
would increase rear module mass and introduce a thermal and safety variable into a
wearable that a medical professional is reviewing — unnecessary risk for validation work.

#### Decision

**USB PD over the tether cable, no onboard battery for Phase 1.** A USB PD 15W sink
negotiation chip (e.g. FUSB302 or IP2721) on the rear module PCB negotiates the correct
voltage profile from the host. A 5V LDO and 3.3V rail power the Pi, panels, and bridge
IC. Plan LiPo + BMS for the Phase 2 rear module PCB respin.

---

## 4. Connectivity & Offload

### 4.1 WiFi Architecture

#### Options Considered

- **WiFi 6 (802.11ax, 2.4/5 GHz):** 2–5 ms round-trip latency under good conditions.
- **WiFi 6E (802.11ax, 6 GHz band):** marginally better latency and congestion in
  dense RF environments.
- **No WiFi — USB-C tether only.**

#### Tradeoffs

Wireless offload is the entire point of Phase 2 but is premature in Phase 1. The tether
provides DisplayPort Alternate Mode video at zero encoding latency and USB PD power
simultaneously over a single cable. Any wireless path requires adding an encoder on the
host side, a decoder and buffer on the device side, and late-stage reprojection to hide
the round-trip delay — none of which exist in Phase 1 firmware. Engineering WiFi before
LSR is operational would produce an unusable result.

#### Decision

**No WiFi hardware on the Phase 1 rear module PCB.** Reserve a footprint on the carrier
board for an M.2 E-key WiFi 6E module (Intel AX210 or equivalent) to be populated in
Phase 2. Do not route the antenna or write any wireless firmware in Phase 1.

---

### 4.2 Supported Client Devices

Any device with a USB-C port that supports DisplayPort Alternate Mode (DP alt) and
USB Power Delivery at 15W or higher:

- **Laptops:** Windows, macOS, and Linux with USB-C DP alt — universal since ~2018.
- **Android phones:** Samsung Galaxy S-series (S20+), Google Pixel 6 and newer, most
  USB4/Thunderbolt Android devices.
- **Not supported in Phase 1:** iPhone (no DP alt on any model as of writing), USB-A
  ports, older USB-C ports without DP alt mode (common on budget laptops).

---

### 4.3 Communication Protocol

#### Options Considered

**DisplayPort Alternate Mode over USB-C**
The USB-C cable carries a full DP 1.4 signal on the high-speed lanes and USB 2.0 on the
sideband. The host GPU treats the glasses as a standard external monitor. No encoder,
no codec, no streaming stack. The bridge IC on the rear PCB converts the DP signal to
two MIPI DSI streams. Zero additional latency over what the host display pipeline already
has.

**HDMI Alternate Mode over USB-C**
Similar concept. Slightly wider chipset support on the bridge IC side (TC358743 is a
common HDMI→MIPI DSI part with good documentation). Slightly less common on host devices
than DP alt mode.

**ALVR or virtual streaming (e.g. Moonlight, Parsec)**
Full streaming stacks with encoding + transmission + decoding. Adds 10–40 ms of latency
and requires custom software on the host. Appropriate for Phase 2 wireless but completely
unnecessary for a tethered Phase 1.

#### Tradeoffs

DisplayPort Alternate Mode is strictly better than any streaming protocol for a wired
connection: no compression artefacts, no additional latency, no host software to install,
no codec licensing. HDMI alt mode is nearly equivalent and is the fallback if DP alt
mode proves harder to implement on the chosen bridge IC. Streaming protocols are
reserved for Phase 2.

#### Decision

**DisplayPort Alternate Mode (DP 1.4) as primary.** Bridge IC: Toshiba TC358779XBG
(DP 1.4 → dual MIPI DSI, well-documented, available from Mouser). HDMI alt mode via
TC358743 as fallback if host device compatibility proves limiting during testing.

---

## 5. Software Stack

### 5.1 Operating System

#### Options Considered

**Raspberry Pi OS (64-bit, Debian Bookworm base)**
Native support for all Pi CM5 peripherals including MIPI DSI. Largest community base for
bringup. Pre-built Wayland image available. Fastest path to a booting system.

**Ubuntu 24.04 LTS**
More current package versions, better upstream Wayland compositor support, and the
correct base for Monado (OpenXR runtime, needed in Phase 2). Runs on Pi CM5 with
some additional device tree work.

**Buildroot / Yocto (custom minimal image)**
Fastest boot (under 3 seconds achievable), smallest image, no extraneous services.
Significant upfront build system investment. Appropriate for a production device,
not for a prototype where the software is changing weekly.

#### Tradeoffs

Pi OS boots to a working MIPI DSI output with less configuration than Ubuntu. The
additional effort to get Ubuntu running on Pi CM5 is not justified for Phase 1 when
the firmware is trivially simple. Migrating to Ubuntu in Phase 2 is straightforward
because the hardware stays the same.

#### Decision

**Raspberry Pi OS (64-bit, Bookworm).** A custom minimal image built from the Pi OS
base, removing the desktop environment and unused services to reduce boot time. The only
running services are the MIPI DSI display initialisation script, an IMU polling daemon
(data logged but unused in Phase 1), and SSH for remote access during development.

---

### 5.2 Rendering Pipeline

#### Options Considered

**Pure hardware passthrough — no device-side rendering**
The DP→MIPI bridge IC handles the video conversion entirely in silicon. The Pi CM5 is not
in the video path at all during Phase 1. The Pi runs only a boot-time init script that
sends the MIPI DSI power-on sequence and timing configuration to the panels.

**Pi CM5 running Wayland compositor in video path**
Pi receives the DP stream via USB-C, composites it (potentially adding UI chrome or
overlays), and outputs to the panels. Introduces one full frame of latency through the
compositor. Enables on-device manipulation of the video stream — necessary for LSR in
Phase 2 — but adds latency and complexity for no Phase 1 benefit.

**On-device LSR shader (Phase 2 target)**
Pi's VideoCore VII GPU samples the most recent IMU quaternion and applies a warp transform
to the incoming frame before display, hiding network and rendering latency. This is the
Phase 2 firmware deliverable on the same hardware.

#### Tradeoffs

In Phase 1 with a wired tether and no head tracking, there is nothing for a device-side
rendering pipeline to do. Putting a compositor in the path adds latency and a potential
source of failures. The bridge IC handles everything better in hardware. The Phase 2
LSR firmware will require Wayland/DRM integration on the Pi, but that work belongs in
Phase 2.

#### Decision

**Hardware passthrough only for Phase 1.** The Pi's role in Phase 1 is to boot, run
the panel initialisation sequences over MIPI, and stay out of the video path. The
bridge IC handles DP→MIPI conversion with sub-millisecond latency. Phase 2 firmware
adds a DRM/KMS-based compositor with an LSR warp shader on the VideoCore VII.

---

### 5.3 Client Applications (Laptop / Phone)

#### Options Considered

**Native OS monitor extension (no custom software)**
The host OS recognises the glasses as a USB-C DP monitor and offers to extend or mirror
the desktop onto them. Windows: plug-and-play via DisplayPort MST. macOS: recognised as
an external monitor in System Settings. Linux: configured via xrandr or Wayland output
management protocols. Any window can be dragged onto the glasses display.

**Custom virtual desktop client**
A dedicated application managing virtual workspace layout, font DPI scaling, blue-light
filtering for evening use, and potentially window anchoring. Better long-term UX. Not
required to validate the optical and mechanical design in Phase 1.

**SteamVR / Monado virtual display driver**
Presents the glasses as an OpenXR HMD to applications. Required for Phase 2 spatial
features. Overly complex for Phase 1 head-locked use.

#### Tradeoffs

The fastest way to learn whether the display concept works is to plug it in and move a
browser window onto it. Custom client software adds a development workstream that delays
that answer. Build the client as a separate parallel effort — it does not block Phase 1
hardware validation.

#### Decision

**Native OS monitor extension for Phase 1.** No custom client software. The host
treats the glasses as a 1280×720 (per eye, presented as a single 2560×720 stereo
display or two independent monitors depending on bridge IC configuration) external
monitor. A custom virtual desktop client is tracked as a non-blocking parallel effort.

---

## 6. Open Questions

- What is the precise back focal distance of the chosen Plössl eyepiece for the ECX339A
  panel size? Needs bench measurement — catalogue focal length is nominal; actual BFD
  varies by up to 10%.
- Can the Pi CM5 drive two independent MIPI DSI outputs simultaneously at 60 Hz?
  The CM5 has two DSI ports in the datasheet but simultaneous independent operation
  needs validation in the device tree configuration.
- What IPD adjustment range must the frame accommodate? Adult population range is
  roughly 55–75 mm. The barrel mount design must allow at least this range.
- Is the ECX339A panel bright enough under the magnification setup to feel comfortable
  in the workshop? Target: ≥200 nit at the eye after optics loss (~30–40% efficiency
  through the eyepiece). ECX339A at 2000 nit gives ~1200–1400 nit at the panel, so
  the margin is substantial — but validate empirically.
- What is the thermal behaviour of the ECX339A in an enclosed barrel housing over a
  2-hour session? Sony specifies maximum junction temperature limits; real-world
  thermal rise in an enclosed PLA housing is unknown.

---

## 7. Insights & Lessons Learned

*(To be filled during build)*

---

## 8. Resources & References

- Sony ECX339A product brief and application note (Sony Semiconductor Solutions)
- Raspberry Pi CM5 datasheet and IO board schematics (raspberrypi.com)
- Toshiba TC358779XBG product brief — DisplayPort 1.4 to MIPI DSI bridge (toshiba.semicon-storage.com)
- FUSB302 USB PD sink controller datasheet (onsemi.com)
- Edmund Optics / Thorlabs aspherical lens catalogues
- Rutten & van Venrooij, *Telescope Optics* — Plössl eyepiece design reference
- Project North Star v2 mechanical and optical design (Ultraleap GitHub)
- MIPI Alliance DSI specification v1.3 (mipi.org, free with registration)
- USB Implementers Forum, *USB Type-C Alternate Mode for DisplayPort* specification
