# Update Log

This is the running record of what we actually tested, changed and observed. It is separate from [plan.md](plan.md), which describes what we intend to do.

- **Plan:** [plan.md](plan.md)
- **Fork overview:** [README.md](README.md)

No firmware changes or camera writes have been made yet.

---

## How to use this log

Add a new entry for every meaningful action:

- documentation or tooling change;
- offline build or verification;
- camera connection;
- firmware preparation;
- camera install;
- defect reproduction;
- HDMI capture session;
- store/bridge change.

Use the latest entry as the source of truth for the current state. Do not rewrite history; add a correction entry if an earlier conclusion changes.

---

## Entry template

```markdown
## YYYY-MM-DD — Short title

**Phase:** Phase N — name
**Build/package:** name and SHA-256, or `none`
**Camera state:** model, firmware, internal revision, ROM/RAM policy, if known
**Hardware:** cable, card, battery, lens, HDMI capture
**Actions:**
1. ...
**Expected:**
- ...
**Observed:**
- ...
**Artifacts:**
- capture path / report path / hash
**Result:** pass / fail / partial / blocked
**Next:**
- ...
```

---

## Defect report template

Use this when the camera misbehaves. Attach the HDMI capture or a photo/video when possible.

```markdown
### Defect summary

**Build/package:**
**Package SHA-256:**
**Camera model:**
**Reported firmware:**
**Internal revision:**
**ROM policy:**
**RAM policy:**
**SD card:**
**Battery:**
**Lens:**

### Steps to reproduce

1.
2.
3.

### Expected behavior

### Observed behavior

### Timing

- Appears on: Import / No LUT / cell switch / startup / other
- Number of frames or duration:
- Still image affected: yes / no / unknown
- Preview affected: yes / no / unknown
- Video affected: yes / no / unknown
- Persists after normal off/on: yes / no / unknown

### Capture

**HDMI capture file:**
**Frame numbers:**
**Photo/video:**

### Camera messages

### Additional notes
```

---

## Current status snapshot

| Item | State |
|---|---|
| Fork documentation | Rewritten: README, plan, update log |
| Target camera | DMC-GM1 Japanese; official 1.1 installed; internal `0110`; English-language state to confirm |
| USB cable | Working stock proprietary cable connected |
| HDMI capture | Mini-HDMI + USB capture card ready |
| Windows USB driver | WinUSB bound to `04DA:2370` with Zadig; device status OK |
| Camera read | **Passed** read-only profile inspection: `gm1-official-0110`, key and guards verified |
| ROM/RAM policy | ROM `0`, RAM `0`; modified unsigned updates enabled |
| Official GM1 1.1 | Installed and confirmed |
| Stable research patch | Installed; language/update-access package active |
| Camera writes | Official 1.1 update, temporary policy write, stable research update, LUT update |
| Firmware builds | Upstream LUT build and native opacity UI build `8b954b` verified on camera |
| Native opacity UI | Verified: `-5..+5` both directions, set persists, re-entry preserved, stock safe; image path unchanged |
| Defect reproduction | Colored lines characterized as a one-time cosmetic profile-menu artifact; low urgency |
| Next milestone | Apply per-cell opacity in the cached table-domain path |
| LUT store | Design only |

---

## 2026-11-23 — Project kickoff and source review

**Phase:** Pre-Phase 0 — orientation
**Build/package:** none
**Camera state:** not connected; exact region/revision/policy unknown
**Hardware:** USB cable pending; mini-HDMI + USB capture card ready

**Actions:**

1. Read the upstream README, GM1 USB, reproduction, LUT, program-analysis, language, policy, factory, Wi-Fi and Image App documents.
2. Reviewed the updater pipeline in `apps/lumix_updater`:
   - model profiles and guards;
   - USB transport and framing;
   - inspection and policy-enable write;
   - firmware decode/delta/encode/container build;
   - language patch;
   - native-preservation checks.
3. Reviewed the GM1 LUT review sources and recipes.
4. Reviewed the old 0.2.4 GM1 opacity implementation in Git history.
5. Audited the Windows host environment.
6. Ran the offline test suite.

**Key findings:**

- The GM1 patch is a trust bypass plus a feature patch, not one exploit.
- The GM1 key is published and the profile checks a fixed key hash, so offline package preparation is possible.
- A stock policy-`1` camera still needs one temporary service-USB policy write before the first unsigned install.
- Once policy `0` is running, later custom installs are SD-only.
- The 0.2.4 opacity lag is explained by the old producer passing the opacity tick into every preview/capture request, which called `lut_strength_generate` and rebuilt the table per job/frame.
- The current full-strength build removed that per-frame work.
- The colored-lines cause is not proven. The strongest hypotheses are in-place table publication racing the ISP, style/table switch order, GM1 table mode/flags inherited from GM5, cache coherency, and frame-copy geometry.
- The GM1 `cam.cgi` owner is unknown; Wi-Fi LUT upload feasibility is unresolved.
- The upstream Windows test suite has one Windows-only test-portability failure for a crafted `App\other.txt` ZIP member.

**Observed environment:**

- Python 3.12.2, Tk 8.6.
- Visual Studio 18 Community with MSVC 14.51 x64; `cl.exe` not on `PATH`.
- `PyInstaller`, `libusb-package` and `certifi` not installed in the default environment.
- No `USB\VID_04DA*` device currently enumerated.

**Offline test result:**

- 144 tests run.
- 1 failure: `tests/test_lumix_updater_release.py::test_camera_binary_or_unsafe_member_is_refused`, only the `App\other.txt` case.
- 56 skipped due to absent private firmware/emulator inputs.

**Decisions:**

- Focus only on DMC-GM1 1.1/internal `0110`.
- Fix colored lines before opacity.
- Restore opacity first; defer native sliders.
- Use a cached table-domain preblend for opacity to avoid the 0.2.4 per-frame cost.
- Use HDMI capture for visual defect diagnosis.
- Node.js store has `store` and `bridge` modes; LAN first, community hosting later; SD path first, Wi-Fi endpoint later.

**Next:**

- Phase 0 offline readiness.
- Prepare HDMI capture baseline and test LUTs.
- Decode GM1 and build the parent-object comparison.
- Wait for the USB cable for Phase 1 read-only validation.

---

## 2026-11-23 — Fork documentation reorganized

**Phase:** Pre-Phase 0
**Build/package:** none
**Camera state:** not connected

**Actions:**

1. Replaced the upstream README with a fork-focused overview:
   - purpose, priorities and status;
   - test bench and human-in-the-loop method;
   - safety rules;
   - one-page explanation of the trust bypass and feature patch;
   - current LUT import behavior;
   - repository map;
   - offline quick start;
   - established vs owner-reported vs open claims;
   - upstream warnings and document index.
2. Added `plan.md` with phases, gates, risks, open questions and decision log.
3. Added this `update-log.md` with templates and the kickoff record.

**Result:** pass

**Next:**

- Complete Phase 0 and record each verification with its hashes.
- Record the first HDMI baseline capture.
- When the USB cable arrives, perform the read-only camera check before any write.

---

## 2026-11-24 — First camera connection and build identification

**Phase:** Phase 1 — USB read-only validation
**Build/package:** upstream updater `0.2.8` for self-test; no firmware package used
**Camera state:** DMC-GM1, Japanese body, English-language change, internal revision `0x0109`; ROM/RAM policy not usable because the reviewed address map does not match
**Hardware:** working stock proprietary USB cable, 20mm f/1.7 lens, mini-HDMI + USB capture card, Windows laptop

### Actions

1. Connected the camera in the previously unlocked service state. Windows initially reported:
   - `DMC-GM1`, `USB\VID_04DA&PID_2370`;
   - status `Error`, problem `CM_PROB_FAILED_INSTALL`;
   - no driver service.
2. Installed Zadig 2.9.788 and bound **WinUSB** to the `2370` service interface.
3. Rechecked the device:
   - status `OK`;
   - class `USBDevice`;
   - problem `CM_PROB_NONE`;
   - driver service `WinUSB`;
   - driver `oem166.inf`.
4. Verified the release archive:
   - `Lumix-Firmware-Updater-win32-x86_64.zip`;
   - bytes `17,397,885`;
   - SHA-256 `e2bba57f95f66363f53d759be8d4dd86ca86d981cabb477a4bff23df6b300214`.
5. Verified the bundled libusb DLL:
   - SHA-256 `698752cd7f4634c36a7e5dd18006ef27b839fd851db571b3991c750351d8bf55`.
6. Ran the frozen self-test:
   - version `0.2.8`;
   - codec, USB library and TLS roots loaded;
   - models `DMC-GM5`, `DMC-GM1`;
   - profile `gm1-official-0110` present;
   - `camera_contacted: false`.
7. Attempted the app's read-only inspection through the release transport. It stopped at the version guard.
8. Dumped nine read-only 1 KiB pages for identification:
   - `0x40000000`, `0x40000400`, `0x4036a000`, `0x4036b000`, `0x4057b800`, `0x405ab400`, `0x405c5800`, `0x7024ac00`, `0x70903800`.
   - Capture: `work/gm1-pages-20261124T064350Z/`.
9. Downloaded and verified the official GM1 1.1 update:
   - ZIP `GM1__V11.zip`, bytes `31,987,180`, SHA-256 `4dbff5b5cd05f288d96506290f8b001ecc55969cea63a61c9fa868cc3084c24b`;
   - BIN `GM1__V11.bin`, bytes `31,983,104`, SHA-256 `3e8b4aab23f79db160cd016af0e466011f047add07bb7441c642f6b70df30d90`.
10. Decoded the official 1.1 program with the published key and verified:
    - `program` SHA-256 `a96fb6da618b33207db0d357ac5e2513583aaa71dd01df44bed498e63992fabe`;
    - internal revision at `0x6A0` = `0x0110 / 0x0110`.
11. Staged the official BIN for the SD update under `work/sd-update-official-1.1/` with a matching `SHA256SUMS`.

### Observed

- The camera's version field at `0x40000400 + 0x2A0` is `0x0109 / 0x0109`, not `0x0110 / 0x0110`.
- The page at `0x405AB400` contains strings such as `TREAM`/`.MTS`, and the bytes at `+0x60` are `a3c4df7ba8c4df7b0000000000000000`, not the reviewed GM1 key.
- The policy candidate at `0x405C5800 + 0x340` is `0x028c`, not `0`/`1`.
- Therefore the app's profile guard correctly refused the camera. No write was made.

### Result

**Blocked as expected.** The camera is not on the reviewed 1.1/internal `0110` build. It is presumably still on firmware 1.0. The correct next step is the official Panasonic 1.1 SD update, not a bypass of the guard.

### Next

1. Confirm the camera's displayed firmware version.
2. Copy the verified `GM1__V11.bin` to the FAT32 card root and verify the copied hash.
3. Install the official 1.1 update through the camera's normal Playback update procedure.
4. Recheck the camera version and reconnect service USB.
5. Re-run the read-only inspection; expect internal `0110`, the reviewed key and a usable policy state.
6. Only then prepare the first custom package.

---

## 2026-11-24 — Official 1.1 update and successful read-only profile check

**Phase:** Phase 1 — USB read-only validation
**Build/package:** official `GM1__V11.bin` for the camera update; no research package
**Camera state:** DMC-GM1 Japanese, official 1.1, internal `0110`
**Hardware:** stock proprietary USB cable, 20mm f/1.7 lens, WinUSB `04DA:2370`

### Actions

1. Confirmed the camera was on body firmware 1.0 using the in-camera version display.
2. Installed the official Panasonic `GM1__V11.bin` from the verified SD package.
3. Confirmed the camera now displays firmware 1.1.
4. Reconnected service USB. Windows still reports `DMC-GM1`, `04DA:2370`, status OK, service `WinUSB`.
5. Ran the read-only profile inspection through the release transport.

### Result

**Read-only profile check passed.**

| Field | Value |
|---|---|
| Model | `DMC-GM1` |
| Firmware | `1.1` |
| Internal revision | `0110` |
| Profile | `gm1-official-0110` |
| ROM policy | `1` |
| RAM policy | `1` |
| Modified updates enabled | `false` |
| Encryption key | reviewed GM1 key (matches the published value; not repeated here) |
| Key SHA-256 | `555c3fdaafe1cda9d5eb941792bd9c3fac2153e1da17e1203c7c107f533344f5` |
| Profile guards | Verified |
| Camera written by this check | No |

Inspection report: `work/gm1-inspection-20261124T070727Z.json`.

### Offline stable package prepared for comparison

- Output: `work/offline-prepared/stable-20261124T070815Z/`
- Candidate `GM1__V11.bin`: 24,512,000 bytes, SHA-256 `6ea79b39b7fca66ec52328ca96f2990235853dd61dc8dedf2bd50953c2fafecf`
- Stock restore `GM1__V11.bin`: 24,512,000 bytes, SHA-256 `cad4ea2739407f3d59d919e87830fde4e1bad03cb3c26d87c53f7e734a45c007`
- Components: `program` 0x900000 at 0, `guires` 0xB00000 at 0x900000, `program2` 0x360000 at 0x1400000
- Built offline with the inspected key; no camera contact

The release app's own stable preparation should produce the same candidate hash. Compare before copying to the card.

### Card preparation

- Card: drive `D:`, 128 GB exFAT, camera-formatted with `DCIM`, `MISC`, `PRIVATE` and `TXT`.
- Existing official `GM1__V11.bin` backed up to `work/card-backup/GM1__V11-official-1.1.bin`; SHA-256 `3e8b4aab…0d90`.
- Stable candidate copied to `D:\GM1__V11.bin`; copied SHA-256 `6ea79b39b7fca66ec52328ca96f2990235853dd61dc8dedf2bd50953c2fafecf`.
- No camera write yet.

### App-prepared stable candidate independently verified

The app prepared its own package at:

`firmware/modded-gm1-v11/GM1-stable-20261124-152507/`

Verification results:

- candidate `GM1__V11.bin`: 24,512,000 bytes, SHA-256 `6ea79b39b7fca66ec52328ca96f2990235853dd61dc8dedf2bd50953c2fafecf`;
- candidate is byte-for-byte identical to `work/prepared/stable-20261124T071942Z/GM1__V11.bin`;
- stock restore: 24,512,000 bytes, SHA-256 `cad4ea2739407f3d59d919e87830fde4e1bad03cb3c26d87c53f7e734a45c007`;
- container entries: `program` 0x900000 at 0, `guires` 0xB00000 at 0x900000, `program2` 0x360000 at 0x1400000;
- `preparation.json`: camera `gm1-official-0110`, internal `0110`, ROM policy `1`, RAM policy `1`, `temporary_flag_needed: true`;
- `automatic_camera_flash: false`, `sd_card_written: false`.

The prepared candidate is approved for the stable-first install. The camera must still be checked in the app immediately before the policy write.

### HDMI limitation

The GM1 HDMI output works in **playback only**; it does not stream live preview. The colored-lines diagnosis must therefore use:

- a short camera-recorded video played back through HDMI;
- still playback for table/format checks;
- phone-filmed LCD preview for artifacts that appear only in live view.

The HDMI capture rig is still useful, but the original plan's "live preview capture" is not possible on this body.

### Notes

- The official update did not break the service USB identity or the WinUSB binding.
- The camera is still on stock policy `1`; the first unsigned research install will need the one-time service-USB RAM policy write.
- The language state after the update still needs to be confirmed in-camera.
- No custom firmware or policy write has been made.

### Next

1. Capture the HDMI baseline before any custom install.
2. Build and verify the stable language/update-access package and its stock restore offline.
3. Prepare the SD card and perform the stable-first install.
4. Verify policy `0` and normal camera behavior.
5. Then install the current LUT package via SD-only to reproduce the colored lines.

---

## 2026-11-24 — Stable research patch installed; policy zero achieved

**Phase:** Phase 1/2 transition — first unsigned install
**Build/package:** `firmware/modded-gm1-v11/GM1-stable-20261124-153531/GM1__V11.bin`
**Package SHA-256:** `6ea79b39b7fca66ec52328ca96f2990235853dd61dc8dedf2bd50953c2fafecf`
**Camera state before install:** DMC-GM1, 1.1/`0110`, ROM policy `1`, RAM policy `1`
**Card:** 128 GB exFAT, stable candidate in root, hash verified

### Actions

1. Ran **Enable this update**; the app reported “Update access verified. Keep the camera awake and powered on.”
2. Unplugged USB and pressed Playback. The camera initially reported “No memory card” instead of the update prompt, even though the card was readable without the `.bin`.
3. Re-seated the card, tried another card, and rechecked policy. The policy had returned to `1/1` after a power cycle.
4. Re-ran the enable and tried again. The actual blocker was the camera not being in the **Standard** profile.
5. Set the camera to Standard, unplugged USB, pressed Playback, confirmed the update, and kept power stable.
6. After the restart, reconnected service USB and ran the read-only inspection.

### Result

**Stable research patch installed successfully.**

| Field | Value |
|---|---|
| Model | `DMC-GM1` |
| Firmware | `1.1` |
| Internal revision | `0110` |
| Profile | `gm1-official-0110` |
| ROM policy | **`0`** |
| RAM policy | **`0`** |
| Modified updates enabled | **`true`** |
| Encryption key | reviewed GM1 key (matches the published value; not repeated here) |
| Key SHA-256 | `555c3fdaafe1cda9d5eb941792bd9c3fac2153e1da17e1203c7c107f533344f5` |
| Profile guards | Verified |

Inspection report: `work/gm1-inspection-20261124T075459Z.json`.

### Notes

- The “No memory card” symptom was **not** a policy-lifetime failure. The policy write worked; the camera needed to be in the Standard profile before Playback would present the update.
- The camera now accepts unsigned custom updates; the stock policy initializer is `0`.
- Future custom installs should be SD-only and should not need the temporary USB policy write.
- The stable package includes the 26-language catalog and update access, but no LUT cells.
- The 128 GB card is exFAT. The LUT importer still requires FAT32; a 16 GB FAT32 card is available for LUT files.
- No LUT package has been installed yet.

### Next

1. Confirm the camera language menu and English selection after the stable install.
2. Record a short native-style baseline via camera-recorded video and HDMI playback.
3. Prepare the current LUT package (`luts` choice) and its stock restore.
4. Install the LUT package SD-only.
5. Reproduce the colored lines using recorded-video playback and/or phone-filmed LCD preview.

---

## 2026-11-24 — LUT reproduction package prepared

**Phase:** Phase 2 — colored-lines reproduction
**Build/package:** current upstream GM1 LUT build
**Package SHA-256:** `4c34eaf367451313775ffed5688eb9c8a9f25dabb5ddeffb4dcaefba305f1fa2`
**Camera state:** stable research patch installed; ROM/RAM policy `0`; unsigned SD installs enabled
**Cards:** 128 GB exFAT (E:), 16 GB FAT32 (D:, label `TZ90`)

### Prepared LUT package

- Output: `work/offline-prepared/luts-20261124T075549Z/`
- Candidate: `23,987,712` bytes, SHA-256 `4c34eaf367451313775ffed5688eb9c8a9f25dabb5ddeffb4dcaefba305f1fa2`
- Components:
  - `program`: 8,912,896 bytes, `0dce652908a235fae60d1588b4b81642414dc732495fc9cfd89f5a9956b0984c`
  - `guires`: 11,534,336 bytes, `9fa9f03ca4a2cc665d32d56537ed03cabd9094e623e8a978177be5fad594e321`
  - `program2`: 3,538,944 bytes, `21b5cdae4fd4ee66892028a611f345a7eab985f56c4ac30f27cd39c643479cbc`
- Stock restore: `cad4ea2739407f3d59d919e87830fde4e1bad03cb3c26d87c53f7e734a45c007`
- Clear stock-restore copy: `firmware/modded-gm1-v11/STOCK-RESTORE/GM1__V11.bin`

This is the current upstream LUT build, intentionally installed to reproduce the colored-line defect. It is not the fix build.

### Card preparation

- E: 128 GB exFAT:
  - existing stable `GM1__V11.bin` backed up to `work/card-backup/E-GM1__V11-stable-before-lut.bin`, SHA-256 `6ea79b39…fecf`;
  - replaced with LUT candidate, verified SHA-256 `4c34eaf3…1fa2`;
  - card photos untouched.
- D: 16 GB FAT32, label `TZ90`:
  - already FAT32; **no format was needed**;
  - existing photos untouched;
  - added test LUTs in root:
    - `IDENT.CUB` — identity;
    - `SWAP.CUB` — red/blue swap;
    - `COLOR.CUB` — strong color transform.
- Both volumes flushed.

### Additional practical test LUTs

Added original film-inspired approximations to D: root. These are for evaluation, not official film emulations.

| File | Look |
|---|---|
| `FLAT.CUB` | low-contrast neutral baseline |
| `PORTRA.CUB` | soft warm portrait, cyan-green shadows, lifted blacks |
| `EKTAR.CUB` | punchy Kodak negative, cool shadows, strong reds |
| `FUJI400.CUB` | low-contrast Fuji negative, pastel highlights |
| `VELVIA.CUB` | high-contrast saturated slide, strong greens/blues |
| `RICOH.CUB` | Ricoh GR-style street contrast, deep blacks, punchy midtones |
| `TRIX.CUB` | high-contrast black and white |
| `CINE.CUB` | teal shadows / warm highlights |

Also added `LUTS.TXT` with the descriptions.

Base-profile note: the importer requires Standard, and the current GM1 LUT patch keeps native processing parameters neutral for a loaded LUT style. The LUT style carries its own contrast/saturation; a Natural `contrast -2 / saturation -2` setting is not dynamically composited by the current patch. Use `FLAT.CUB` or `PORTRA.CUB` when a softer base is wanted. If the camera behaves differently, record it.

### Next

1. Eject both cards.
2. Put the 128 GB card (E) into the camera.
3. Camera off, Standard profile, insert card, power on, press Playback.
4. Confirm the LUT firmware update and keep power stable.
5. After restart, verify LUT Library appears and policy remains `0`.
6. Put the 16 GB card (D) in the camera and import `IDENT.CUB` first.
7. Reproduce Import/No LUT with recorded-video playback and/or phone-filmed LCD preview.

---

## 2026-11-24 — Seven LUT sample images inspected

**Phase:** Phase 2 — colored-lines reproduction
**Build/package:** current upstream LUT build, package `4c34eaf3…1fa2`
**Camera state:** LUT firmware installed; ROM/RAM policy `0`
**Samples:** `samples/`

### Inputs

Seven camera JPEGs, all `4592 × 3448`, `Panasonic` / `DMC-GM1`, taken within one minute of each other:

- `Portra.JPG`
- `Ektar.JPG`
- `Fuji400.JPG`
- `Velvia.JPG`
- `Ricoh.JPG`
- `Trix.JPG`
- `Cine.JPG`

### Result

All seven LUTs applied and produced distinct, plausible looks:

- **Portra:** warm, soft, lifted shadows, controlled saturation.
- **Ektar:** punchier, cooler shadows, strong reds.
- **Fuji400:** lower contrast, pastel highlights, cyan-green shadows.
- **Velvia:** high contrast and saturation, vivid greens.
- **Ricoh:** deep blacks, punchy street-contrast midtones.
- **Trix:** high-contrast black and white.
- **Cine:** teal shadows with warm highlights.

**No colored vertical lines or stripes are visible in any of the seven still images.** The LUT tables, import path, conversion and steady-state rendering therefore look correct. The reported defect is consistent with a transient transition artifact, not a static table/format error.

### Next

1. Switch between a loaded LUT style and Standard/No LUT while watching the LCD.
2. Record a short video immediately after the switch and play it back through HDMI, or film the LCD in slow motion.
3. Capture the first frames after Import, No LUT and cell switch.
4. Report whether the lines appear only in live preview, in recorded video, or both.

---

## 2026-11-24 — Colored lines characterized as low-urgency cosmetic artifact

**Phase:** Phase 2/3 — colored-lines characterization
**Build/package:** current upstream LUT build, package `4c34eaf3…1fa2`
**Camera state:** LUT firmware installed; ROM/RAM policy `0`

### Observation

The owner reports the colored vertical lines are **transient** and appear **once, after the first LUT import**, when checking the LUTs in the **profile selection menu**.

They are **not** a functional defect:

- the LUTs import correctly;
- all seven sample images render correctly;
- normal shooting and profile selection continue to work;
- the artifact does not recur as a steady-state error.

The exact appearance, duration and whether it is LCD-only or also present in recorded video are not yet captured. The current evidence points to a one-time profile-menu preview/initialization transition, not the LUT table or conversion.

### Decision

- Keep the current upstream LUT build as the working baseline.
- Treat the colored lines as a **known cosmetic issue** with low urgency.
- Do not block opacity or LUT-store work on it.
- If it can be captured later without disrupting the main work, record it; otherwise document it as an unresolved transition artifact.

### Next

Proceed to **priority 2: restore GM1 opacity** using a cached table-domain preblend, then the Node.js LUT store.

---

## 2026-11-24 — Opacity recipe analysis and merge blocker

**Phase:** Phase 4 — restore opacity
**Build/package:** none; offline analysis only
**Camera state:** current upstream LUT build installed; policy `0`

### Analysis

Compared the 0.2.4 opacity recipe (`a5909a6`) with the current full-strength recipe:

- program: 1,481 old edits vs 1,337 current edits; 205 old-only offsets; 61 current-only offsets; 17 same-offset changes;
- guires: 179 old edits vs 142 current edits; 37 old-only offsets; one conflicting offset;
- program2: identical.

The old opacity code spans many regions:

- native slider/UI adapters, numeric/scale rendering and assets;
- cell-request/tick handling and `lut_strength_generate`;
- packed job/table generation;
- store/restore and producer/dispatcher paths;
- GUI resources for the slider/numbers.

A dry-run layering of the current recipe edits onto the old opacity target produced **71 program conflicts**; the reverse layering produced **59 program conflicts**, plus a GUI-resource conflict. The two builds are not separable by simple byte-range replacement.

### Blocker

The public repository does **not** include the private linker/build system that produced the installed code. A blind hybrid would be unsafe and would likely reintroduce the 0.2.4 stack/lifecycle issues or the per-frame opacity lag.

### Safe options

1. **Pre-baked strength variants (no firmware change).**
   - Generate 25/50/75% blends of the film LUTs with an identity table.
   - Load different strengths into different cells.
   - This gives practical opacity control immediately, without touching firmware.
2. **Controlled native-opacity experiment.**
   - Build the old 0.2.4 opacity target as a test package.
   - Install it with policy `0`, measure the menu/power-off lag, and confirm the native slider works on this body.
   - Then decide whether a cached-preblend patch is worth the reverse-engineering effort.
   - This reintroduces the known 0.2.4 risks and should be a deliberate test, not a default install.
3. **Full native opacity patch.**
   - Requires reconstructing or extending the private build/link step, or hand-assembling a cache-aware replacement and merging it with the current stack fixes.
   - Not safe to attempt as a blind byte merge.

### Workspace

- `work/opacity-analysis/old-gm1-luts.json`
- `work/opacity-analysis/official-components/`
- `work/opacity-analysis/old-components/`
- `work/opacity-analysis/new-components/`
- `work/opacity-analysis/hybrids/`
- `work/analyze_opacity.py`
- `work/merge_opacity.py`

No camera write was made during this analysis.

### Interim strength variants

Generated 21 pre-baked opacity variants from the film LUTs:

- 25%, 50% and 75% blends for `PORTRA`, `EKTAR`, `FUJI400`, `VELVIA`, `RICOH`, `TRIX` and `CINE`;
- names such as `PORT25.CUB`, `PORT50.CUB`, `PORT75.CUB`, `TRIX50.CUB`, etc.;
- output: `work/test-luts/strength/`;
- 16 GB card was not mounted when they were generated, so they are **not yet copied** to `D:`.

These provide practical opacity-like control by loading different strength cells. They are not the native slider.

### Cine strength variants copied to the camera card

The owner asked to evaluate the Cine look at three strengths first. Copied to the 16 GB FAT32 card:

| File | Strength | SHA-256 |
|---|---|---|
| `CINE25.CUB` | 25% | `57ff0fa12719e8efa2b2b79a3fc22fa65e5aad14aa4c6370c682e325542a8912` |
| `CINE50.CUB` | 50% | `129f6daac0b6dd7ad800f979d4c0bd8718cf6af45518eb55d330794ee1c6c663` |
| `CINE75.CUB` | 75% | `80851708d8e862686965d76eceb4b42bcf489b80ef3ded86b74b4824f472b61c` |
| `CINE.CUB` | 100% | existing full-strength grade |

`LUTS.TXT` on the card was updated with the strength list. The card was flushed.

Test plan: import the three variants into separate cells, select each from Photo Style/Quick Menu, use the same scene/exposure/white balance, and compare. The owner will confirm the useful strength before we design the native slider.

### Cine strength samples inspected

The owner supplied `Cine25.JPG`, `Cine50.JPG`, `Cine75.JPG` and `Cine100.JPG` in `samples/`.

| Sample | Result |
|---|---|
| 25% | subtle teal/warm shift; close to neutral |
| 50% | balanced cinematic look; useful midpoint |
| 75% | stronger contrast and teal shadows |
| 100% | full Cine grade |

The blend is monotonic and the 50% midpoint is usable. The static images are clean; no colored-line artifact appears in the stills.

### Toolchain milestone

Built a real MN103 toolchain under MSYS2:

- binutils 2.42 configured for `--target=mn10300-elf`;
- installed at `/usr/local/mn10300` (Windows path `C:\msys64\usr\local\mn10300`);
- working `mn10300-elf-as`, `ld`, `objcopy`, `objdump`;
- assembler syntax validated against the GM1 review-source instruction forms;
- fixed-address linking at `0x40830000` and binary extraction validated;
- a tiny probe was patched into a copy of the current `program` at file offset `0x830000` and disassembled back correctly.

Safe free space in the current LUT program:

- `0x8263ad..0x880000` is `0xFF` in both the official and current builds and is untouched by either recipe.
- The probe used base `0x40830000`; this is a viable module area for new code.

Current LUT adapter map (file offsets / addresses):

| Function | Address |
|---|---|
| `lut_gm1_fixed_loaded` | `0x40808000` |
| `lut_gm1_fixed_parameters` | `0x4080803d` |
| `lut_gm1_fixed_widgets` | `0x408080bd` |
| `lut_gm1_fixed_values` | `0x408081bc` |
| `native_row_state` | `0x4014dab2` |
| `native_parameter_mode` | `0x4020f96d` |
| `native_parameters_continue` | `0x400e20e3` |
| `native_style_mode` | `0x4020f93e` |
| `native_widgets_continue` | `0x400e6de4` |
| `native_values_continue` | `0x4020f6f7` |
| `lut_gm1_core_ui` | `0x40806116` |

Additional mapping:

- current cell adoption: `0x407f61d0` (`lut_gm1_packed_cell_adopt`);
- current record-check helper: `0x4079f0ca`;
- old opacity UI trampolines: `0x4079a048` → `0x407b0112`, `0x4079a07c` → `0x407f8086`;
- old opacity adapter block (0.2.4): `0x40800000..0x408002xx`;
- current no-opacity adapter block: `0x40808000..0x408082xx`;
- saved LUT banks: `0x80880000` and `0x808A0000`.

The old opacity UI block is a separate adapter layer, not the current job/producer code. The current cell-adoption hook at `0x407f61d0` is the best place for a one-time fixed-opacity table blend.

Next: dump the saved LUT bank over read-only USB, extract the camera's own neutral/identity table, then implement a fixed 50% blend at adoption in the free region.

### Bank dump and neutral table

The camera was reconnected in service mode and both saved LUT banks were dumped read-only:

- bank A `0x80880000`: `131072` bytes, SHA-256 `9d293a1a…cfac`;
- bank B `0x808A0000`: `131072` bytes, SHA-256 `da2a3f2b…580d`;
- capture: `work/lut-banks-20261124T104629Z/`.

The bank header is `GM5L`, version 2, ten cells, generation 22, mask `0xff`. Cell 7 is named `IDENT.CUB`; its 2,188-byte payload is the camera's exact identity table:

- payload SHA-256 `034a50ef2ea6369ff85142ebab7eb80c15987db11c2664b4bf5db15cfb73575b`;
- saved to `work/opacity-native/neutral-payload.bin`.

### Fixed-50% prototype

Implemented a fixed 50% blend at the current cell-adoption hook (`0x407f61d0`):

- new wrapper/module at `0x40830000` in the free region;
- adoption call at `0x407f625a` redirected to the wrapper;
- wrapper first calls the original record check at `0x4079f0ca`, then blends the 2,284-byte table in place with the neutral payload;
- Y bytes and CbCr words are averaged with rounding; descriptor pointers are unchanged;
- only two program ranges changed: the 4-byte call displacement at `0x7f625b` and the module at `0x830000..0x830900`.

Test package:

- `work/opacity-native/fixed50-package-20261124T105256Z/GM1__V11.bin`;
- bytes `23,987,712`, SHA-256 `9b947235a130fec6fcdad7fe000c71dd46b865087ef4755e48a3c5e344fce426`;
- program component `1ce5de27…8a7e`;
- guires/program2 current LUT targets;
- stock restore `cad4ea27…c007`;
- container entries and full component decode verified offline.

Expected behavior:

- the blend runs only when a LUT is adopted/imported;
- existing saved cells remain full-strength until re-imported or cleared;
- this is a fixed-50% test, not the native slider;
- no camera write yet.

### First fixed-50% test result

The first package was installed and the camera read back:

- installed call site at `0x407f625a` = `dd a6 9d 03 00 f0 14`, pointing to the wrapper;
- installed module at `0x40830000` is present;
- bank B generation 23, mask `0x1ff`;
- UI cell 7 = bank 6 `CINE.CUB` (full);
- UI cell 8 = bank 7 `IDENT.CUB`;
- UI cell 9 = bank 8 `CINE.CUB`.

Cell 9's payload hash is `76c11591c49a623e…`, **identical to the full Cine payload**, not a 50% blend. The installed patch is present, so this is either an import made before the patch or an import path that does not use the patched adoption hook.

Also found a prototype bug: the neutral payload has a padding byte after odd Y counts, and the first blend loop did not skip it. Corrected package built at `work/opacity-native/fixed50-package-20261124T111316Z/GM1__V11.bin`, SHA-256 `2a0b2e79356a5056252f8b40d42c42ee292b379b3ea1528dcea8485c0a28406a`.

Next diagnostic: with the current installed build, import `CINE.CUB` into the empty UI cell 10 and dump the bank. If cell 10 differs from full Cine, the adoption hook runs; then install the corrected package for a clean test. If cell 10 is still full Cine, trace the actual import path before another update.

### Cell 10 diagnostic

UI cell 10 was imported from `CINE.CUB`. Bank A generation 24, mask `0x3ff`; bank cell 9 is again `CINE.CUB` with payload hash `76c11591…`, identical to full Cine.

The adoption hook is therefore **not used by the import path**. The actual import parser is at `0x407bf000`, called from `0x407c01d4`; it parses/converts and then stores the table.

### Import-hook prototype

Moved the blend hook to the import parser call:

- wrapper at `0x40830000` calls the original parser `0x407bf000`;
- on success, it loads the table pointer from `(context+164)`;
- it runs the original record check, then the corrected 50% blend;
- if the table check fails, it skips the blend and preserves the original import result;
- the import call at `0x407c01d4` is redirected to the wrapper.

Only the 4-byte call displacement and the new module changed.

New package:

- `work/opacity-native/fixed50-package-20261124T112356Z/GM1__V11.bin`;
- SHA-256 `e410c520e5ae06ed79e261a1ac4bee624f62e12dc1b308e9c41217b8a34440f4`;
- program `32df3919…dfae`;
- stock restore `cad4ea27…c007`.

Next: install this package, re-import `CINE.CUB` into UI cell 10, dump the bank and compare with the expected 50% blend.

### Import-hook result — success

The import-hook package was installed and `CINE.CUB` was re-imported into UI cell 10.

Bank B generation 27, mask `0x2ff`:

- UI cell 7 = bank 6 `CINE.CUB`, full payload `76c11591…` preserved;
- UI cell 8 = bank 7 `IDENT.CUB`, payload `034a50ef…` preserved;
- UI cell 9 (bank 8) is now empty;
- UI cell 10 = bank 9 `CINE.CUB`, sequence 27.

UI cell 10 payload:

`2339a3910150ede5c824d32652cc3a9009c7a087282480d72d695ad8ad1aa4e4`

Expected 50% blend of the full Cine payload and the identity payload:

`2339a3910150ede5c824d32652cc3a9009c7a087282480d72d695ad8ad1aa4e4`

**Exact match, 0 differing bytes.** The table-domain blend, padding handling, import hook and persistence path are proven.

Existing full-strength cells were not modified.

### Visual control found a chroma bug

The owner supplied `samples/Cine-Proto-50.JPG`. It shows severe green/red/magenta color corruption, not a 50% Cine look.

Cause: the camera packs **Cb and Cr as two 6-bit components in one 16-bit word**. The first blend routine averaged the packed word, so a carry from Cb corrupted Cr. The bank comparison matched the Python model exactly because the model used the same packed averaging.

Corrected blend:

- Y bytes: average as before;
- Cb: `(Cb_a + Cb_b + 1) / 2` on the low 6 bits;
- Cr: `(Cr_a + Cr_b + 1) / 2` on the high 6 bits;
- repack `Cb | (Cr << 6)`.

Payload comparison:

- old cell 10 payload: `2339a391…a4e4`;
- corrected expected payload: `85b6f315745630694ce8bfd1bb120f3e493cc3929f0fd7d097c442a3f5a2c48e`;
- 414 bytes differ, almost entirely chroma.

Corrected package:

- `work/opacity-native/fixed50-package-20261124T114309Z/GM1__V11.bin`;
- SHA-256 `d8bcfe1f108a88109f5c68c0cf480a63f2707e2f7d3c7a9e04668fa88f22f0fe`;
- program `88d1a3c6…4080`;
- stock restore `cad4ea27…c007`.

Next: install the corrected package, re-import `CINE.CUB` into UI cell 10, and compare the stored payload to `85b6f315…c48e`. Then check the rendered image and, if needed, verify Cb/Cr ordering.

### Corrected blend verified on camera

The corrected package was installed and `CINE.CUB` re-imported into UI cell 10.

Bank B generation 29, mask `0x2ff`:

- existing cells 0–7 unchanged;
- UI cell 10 = bank 9 `CINE.CUB`, sequence 29;
- payload `85b6f315745630694ce8bfd1bb120f3e493cc3929f0fd7d097c442a3f5a2c48e`.

That is an **exact match, 0 differing bytes**, to the corrected expected 50% blend. The owner confirms the rendered image has no chroma corruption or inversion.

The fixed-50% opacity prototype is complete:

- import hook works;
- table-domain blend is correct;
- Y and packed Cb/Cr components are handled separately;
- bank persistence works;
- existing cells are preserved.

### Next: native slider

The fixed-50% prototype is complete. The remaining work is UI and persistence:

- use the native Contrast row as the opacity control;
- store a per-cell strength byte;
- keep the original imported table and generate the active blended table once on slider/cell change;
- redirect the per-frame producer to the active blended table;
- startup restore reads strength and regenerates once;
- map native get/set/decrease/increase/draw/backup/cancel adapters;
- no per-frame blend.

### Native slider design

The native slider will use the same table-domain blend, not a per-frame filter:

- **one active preblend cache** (2,284-byte canonical table);
- per-cell strength byte in the saved-bank metadata, defaulting old cells to 100%;
- native Contrast-row slider, ticks 0–10 → 0–100% in 10% steps;
- slider/cell change regenerates the cache once;
- the existing per-frame producer copies the cached table; no `lut_strength_generate` in the frame path;
- startup restore regenerates the active cache once;
- current stack-fix/job/producer code stays in place; we do not merge the old 0.2.4 job code.

The build step is the blocker: we need an MN103 assembler/linker (or a hand-assembled replacement) to place the new cache/UI routines and trampolines in the current build, then generate a new recipe and target hashes.

---

## 2026-11-24 - Native opacity slider, stage A

**Phase:** Phase 4 - restore opacity

### What was mapped

The 0.2.4 opacity build was decoded to recover its native-row adapter:

- selection predicate: style `0x70101252` in 7..16, mode `0x700fcec8 != 2`, and
  the per-style mask bit at LUT-core `+304`;
- per-cell strength record: 10 bytes at `0x70011524`, tag `0x4c53` at `+10`;
  invalid records initialise to 100%;
- row getter hook `0x4020f5f9`, row setter hook `0x4020f620`, step hooks
  `0x4010aa81` / `0x4010aafa`, backup `0x40109f2a`, cancel `0x40109fbe` /
  `0x40109f6b`;
- row value list at `0x708f47a8`; numeric option list for row 0 at `0x708f47d8`;
- row states: 1 hidden, 2 visible/non-adjustable, 4 visible/selectable.

### Change

A new module was assembled at `0x40840000` (1306 bytes) and the stock hooks
above were redirected to it. The current fixed-style parameter/widget adapters
are bypassed for loaded LUT styles:

- parameter rows 1..4 stay hidden; row 0 is forced to state 4 (selectable);
- widget `0x1604` is forced to state 4;
- the getter returns `round(strength / 10)` for row 0;
- the setter stores `tick * 10` for row 0 and leaves the native image
  parameter bytes neutral;
- step, backup and cancel operate on the record.

Only the nine hook sites and the new module changed; the rest of the program is
byte-identical to the current upstream LUT build.

### Scope limit

This stage is **UI only**. The per-frame table copy still uses the imported
cell table, so moving the slider changes the stored strength but not the image
yet. This isolates the menu/record risk from the image path.

### Package

- `work/opacity-native/slider-package-20261124T125529Z/GM1__V11.bin`
- package SHA-256 `492031dfefbd9506dce641afc98d92863d2a4f5d19f98dde3d849237a8e752a9`
- program SHA-256 `a5a2092bd547ad8936d8ce76a8d9270200745c27ec5f8ab936a486adab42a4d2`
- stock restore `cad4ea27...c007`

### Camera test plan

1. Select a loaded LUT cell.
2. Open the style adjustment page and look for a row (expected: the native
   Contrast row) with value 10.
3. Move it 10 -> 5 -> 0 and back; the displayed value must change.
4. The image must stay identical because the image path is unchanged.
5. Leave and re-enter the page; the value must persist in RAM.
6. Switch to another loaded LUT cell; each cell must keep its own value.
7. Power cycle and re-check to see whether `0x70011524` survives.

If the row does not appear or is not selectable, the next step is to inspect the
row/widget state convention rather than the blend.

---

## 2026-11-24 - Native opacity slider, stage A2 (editor draw path)

**Phase:** Phase 4 - restore opacity

### Camera result of stage A

The row appeared with the label **Contrast** and could be entered, but the
editor showed the native -5 / 0 / +5 contrast scale and the value did not move.

### Cause

The stage A setter stored only the opacity record. The native row editor draws
its value from the image-parameter array (`0x70101253`), which the LUT values
adapter keeps neutral, so the cursor never moved.

The 0.2.4 build did not write that array either. It substituted the displayed
value in two native drawing sites:

- `0x403244b1` (value used by the row editor) -> tick from the record;
- `0x40324731` (value-to-text map) -> a private map `0..10 -> 0x1b58..0x1b62`
  so the readout is 0..10 instead of -5..+5.

### Change

Added two read-only draw adapters to the module:

- `op_draw_value` (module `0x4084010a`): restores D1/A0/MDR/PSW, returns the
  tick for a loaded LUT row 0, otherwise the stock array byte, then continues
  at `0x403244b9`;
- `op_draw_label` (module `0x4084015c`): same selection, substitutes the
  private value map, then continues at `0x4032473f`; the stock path keeps the
  original `movbu (d1,a0),d0` at `0x40324737` and the native map.

The image-parameter array stays neutral, so the image path is still unchanged.

Two diagnostic counters were added: `0x70011532` counts setter hits and
`0x70011533` counts step hits. `work/dump_opacity_state.py` reads the record,
counters, parameter array and selected style.

### Package

- `work/opacity-native/slider-package-20261124T131924Z/GM1__V11.bin`
- package SHA-256 `287af20e20a2a19682f9ce3bd69372504d407dea28c81b371d3fe466bc20dfe5`
- program SHA-256 `cd35fc59a646a8b9a9b0965f47855d9c77c9d9e1eabc4b14424dd288ed4cea4b`
- stock restore `cad4ea27...c007`

### Camera test

1. Enter the Contrast row and use **left/right** (or the rear dial), not
   up/down, to change the value.
2. Report the number shown at entry and after each step.
3. Exit, re-enter and check the value.
4. Reconnect service USB; the state dump will show whether the record, the
   counters and the array moved.

---

## 2026-11-24 - Opacity slider: input and regression diagnostic

**Phase:** Phase 4 - restore opacity

### Camera findings

- The editor row shows value 10 (the draw adapter reads the record).
- No control changes the value. The setter and step counters stayed at zero, so
  the editor input never reaches `0x4020f620`, `0x4010aa81` or `0x4010aafa`.
- Going backward from the first LUT cell or forward from the last one exits the
  style menu to preview. The previous build did not do this.
- RAM dump: owner `0xb3000220`, mask `0x2ff`, selected style 11, record cells
  0..8 = 100 and cell 9 = 90. The record area therefore pre-dates this build.
- The page dispatcher tables were decoded: table `0x4037e6f0` maps event ids
  4/5, 15/16/17/18 and 31/32 to the value handlers, and 6/7 to row movement.

### Diagnostic build

Two risky pieces were removed:

- the custom value-to-text map hook (`0x40324731`) and its text ids;
- the full replacement of the parameter/widget adapters.  The shipped adapters
  now run unchanged except for three one-byte edits that keep row 0 and widget
  `0x1604` active (`0x4080806c`, `0x408080f2`, `0x408080ff`, value `01` -> `04`).

The record hooks, the step hooks and the value draw hook remain.

Three static event bitmasks were added so the next camera session identifies
the real control mapping:

- page 1 dispatcher `0x4010a637` -> mask `0x70011518`;
- page 2 dispatcher `0x4010a80c` -> mask `0x7001151c`;
- page 3 (value editor) dispatcher `0x4010a8e1` -> mask `0x70011520`.

Each dispatcher sets bit `(event & 31)` just before the indirect handler jump.
`work/dump_opacity_state.py` prints the three masks and the record.

### Package

- `work/opacity-native/slider-package-20261124T134838Z/GM1__V11.bin`
- package SHA-256 `2b25f6d32e27e22619b2e1bdc2638680e4f9b0f5c81a6da6d30c37c569ba70fd`
- program SHA-256 `8dc3bb346a635354afcce34c2c661ce19b4443cf3dac70d77396317804113f75`
- stock restore `cad4ea27...c007`

### Session

1. Install; the update reboot clears the masks.
2. Check whether the native profile wrap still exits to preview.
3. Open the Contrast row; press LEFT once, RIGHT once, then one rear-dial
   click each way; leave the menu open, then reconnect service USB.
4. Dump the masks: page 3 shows exactly which event ids the editor received.

---

## 2026-11-24 - Opacity slider: direct tick stepping

**Phase:** Phase 4 - restore opacity

### Diagnostic result

The event masks proved the editor receives the expected ids (page 3 mask ids
3,4,5,6,10,14,15), and the setter/step counters reached 82. So the input path
and our hooks were both running.

The record still did not move for the selected cell. The native option-list walk
(`0x4014d9b4` / `0x4014d972` over `0x708f47d8`) returned the same value each
time, so the setter kept storing 100. The full 11-entry list is present
(`0x708f47d8..0x708f4837`, values 0..10), so the walk itself is the problem for
this row.

### Change

`op_step` no longer uses the native option-list walk. It reads the stored tick
and applies clamped arithmetic:

- decrease: tick-1, clamped at 0;
- increase: tick+1, clamped at 10;
- then the existing setter stores `tick * 10`.

The stock profile wrap is already fixed: the shipped parameter/widget adapters
run with only the three one-byte edits.

### Package

- `work/opacity-native/slider-package-20261124T143303Z/GM1__V11.bin`
- package SHA-256 `0062731097cf1d7f6e4b8c4275f79d17e170fc0a19baf551aa0457d6615d9e06`
- program SHA-256 `608760a46d12170847379d73eac201af20e0a770fe71dfa6a1894d6d852283ed`
- stock restore `cad4ea27...c007`

### Expected camera result

- Left/right or the dial must move the readout between -5 and +5 (native
  contrast scale; tick 10 shows +5).
- The image path is still unchanged; only the stored per-cell strength moves.

---

## 2026-11-24 - Opacity slider: step works, sync the native value array

**Phase:** Phase 4 - restore opacity

### Diagnostic result

After the arithmetic step build:

- record cells 0, 4 and 9 all moved from 100 to 90;
- set and step counters reached 108;
- page 3 event mask still ids 3,4,5,6,10,14,15.

So the buttons, dial and touch all reach the editor and the step/store path now
works. The remaining mismatch is presentation only:

- the bar follows the record through the draw hook;
- the numeric readout still comes from the native parameter array
  (`0x70101253`), which the LUT path keeps neutral, so it shows +0.

### Change

The getter and setter now keep `0x70101253[0]` and the display mirror
`0x70101258` equal to the stored tick:

- `op_get` writes the tick into both before returning;
- `op_set` writes the tick into both after storing the record;
- the image path is unchanged; the LUT values adapter still supplies neutral
  processing parameters.

### Package

- `work/opacity-native/slider-package-20261124T154434Z/GM1__V11.bin`
- package SHA-256 `bc9a10f1634c327f68f38cbda5b6264e6b0002621e35203a83ea252856ac2ac4`
- program SHA-256 `1a8867dc94f5c2d53bca3e26e972a9d77014d9389636716301f5a09c259bcd5e`
- stock restore `cad4ea27...c007`

### Check on camera

- The numeric readout and bar must both move with left/right or the dial.
- The readout uses the native contrast scale, so 100% is +5 and minimum is -5.
- Watch whether the picture contrast changes. If it does, the array feeds the
  processing path and the display needs a different source.

---

## 2026-11-24 - Opacity slider: step/set trace build

**Phase:** Phase 4 - restore opacity

### Result of the array-sync build

- The bar and numeric readout both show +5 (raw 10).
- The step and set counters reached 159, so the input still reaches the hooks.
- The record stayed at cells 0/4/9 = 90 and the parameter array stayed
  `[5,5,5,5,0]`; the display mirror stayed 10.
- The 90 values survived two firmware updates, so the record area is part of a
  saved settings page rather than plain scratch RAM.

### Why trace bytes

The display value appears to come from the mirror (`0x70101258`) rather than the
draw hook, and the mirror stayed 10. The next build records the actual step and
set arguments in the same saved page:

- `0x70011510` step direction;
- `0x70011511` tick read before stepping;
- `0x70011512` tick after stepping;
- `0x70011513` step cell;
- `0x70011514` setter style;
- `0x70011515` setter parameter;
- `0x70011516` setter value;
- `0x70011517` setter cell;
- `0x7001150f` tick read back after the record store.

### Package

- `work/opacity-native/slider-package-20261124T160237Z/GM1__V11.bin`
- package SHA-256 `1af2f200f081f8c08791eda14eb575257d788ca2457fe7ed683dbee7c247ee6f`
- program SHA-256 `fa3e80c84a80b770041f43a8f21ab505e518ed58d8043838b9b9218adb987027`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Opacity record moved to the owned library tail

**Phase:** Phase 4 - restore opacity

### Findings

- The step computes `10 -> 9` and passes `9` to the setter, but the read-back
  from `0x70011524` is still `10`, and the raw store/read traces came back zero.
- The diagnostic bytes at `0x700114ea..0x70011523` are being cleared by the
  camera, and writing there also broke the native adjustment page: pressing
  down on a stock profile exited to preview.
- Conclusion: `0x700114xx..0x700115xx` is a live UI/settings buffer, not free
  scratch. The old 0.2.4 address was never hardware-validated on this body.

### New storage

A read-only pool scan found the owned LUT library object:

- `GML1` at `0xb3000420`, size 6144;
- reserved tail at `0xb3001740..0xb3001bff` (all zero).

The per-cell strength record now lives at `0xb3001740`:

- `+0..+9`: strengths, initialised to 100;
- `+10..+11`: tag `0x4c53`;
- the module finds the object at runtime by scanning the first 4 KiB of the
  pool10 alias for `GML1` with size 6144, so it follows reallocation.

All diagnostics and the native parameter-array writes were removed. The native
sliders should work again.

### Package

- `work/opacity-native/slider-package-20261124T163024Z/GM1__V11.bin`
- package SHA-256 `61dfdf71dc8195e5cc5589939c8e3af85031cf213208ffbc04dca98569732821`
- program SHA-256 `8977596325f68ecb7b0fa654f0e9666bdbe40e032a1b3b46580a1d0a86e80774`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Native slider crash bisect

**Phase:** Phase 4 - restore opacity

### Result after removing diagnostics

The stock-profile crash remained even with every diagnostic write removed.
The record is now in the owned library tail, so the crash is from a UI hook,
not from the old RAM range.

### Bisect build

This build removes every piece that touches the native adjustment page and
keeps only the four pass-through hooks:

- getter `0x4020f5f9`, setter `0x4020f620`, decrease `0x4010aa81`,
  increase `0x4010aafa`;
- no value-draw hook, no parameter/widget byte edits, no diagnostics.

For stock styles these four hooks replay the original code. If the stock
sliders still crash, one of the pass-throughs is wrong. If they work, the
crash is in the draw hook or the adapter byte edits.

The LUT adjustment row is intentionally absent in this build.

### Package

- `work/opacity-native/slider-package-20261124T164057Z/GM1__V11.bin`
- package SHA-256 `985961a11334a123a6db7caf311cba8b4416f1cd29e0f1cd8e82faba1688a569`
- program SHA-256 `0df00b2a73a585fdf39ca24a14213490b2d9a2e9da5e0a6cdc271a31d29a9009`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Native slider bisect result

**Phase:** Phase 4 - restore opacity

### Result

With only the four pass-through hooks (getter, setter, decrease, increase) the
stock sliders are fully usable again, so those hooks are safe. The crash came
from the value-draw hook or the row/widget byte edits.

### Fix attempt

- The draw hook now reads the UI selection (`0x70101252`) instead of the stale
  image-side style (`0x702250fa`). A stock page could previously receive a LUT
  tick (0..10) and draw it against a stock row with a narrower range.
- The row 0 and widget `0x1604` byte edits are restored, so the LUT row is
  back.

### Package

- `work/opacity-native/slider-package-20261124T165058Z/GM1__V11.bin`
- package SHA-256 `69dc99288d808b77f60f6c83cd3683a698776fb606733a0d382fece48f8354b3`
- program SHA-256 `a23fc3b3284612182e307d1cb7a23457107df22f1a56ff5a5bb15ac11b0646a7`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Bisect: row edits without the draw hook

**Phase:** Phase 4 - restore opacity

Re-adding both the draw hook (UI-style variant) and the row/widget byte edits
brought the stock-slider crash back. This build keeps the byte edits and drops
the draw hook, so the next result separates the two:

- if stock sliders work, the draw hook is the crash;
- if stock sliders still crash, the row/widget edits are the crash.

The LUT adjustment row should still appear. Its readout will not move because
the native array is not written and the draw hook is absent, but left/right
should update the record at `0xb3001740`, which `work/dump_opacity_state.py`
now prints.

### Package

- `work/opacity-native/slider-package-20261124T165930Z/GM1__V11.bin`
- package SHA-256 `f21a0f755b92caf5cc64a6009b88e08170961b49044cbef55fe1a5a89f866219`
- program SHA-256 `f7815122a4ffd04c7d4c3f9bf3a7972273666425c47bdc034fffef1f3d5a482c`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Display via the native parameter array

**Phase:** Phase 4 - restore opacity

### Bisect conclusion

- Four pass-through hooks: stock sliders work.
- Plus row 0 / widget `0x1604` byte edits: stock sliders still work.
- Plus the value-draw hook `0x403244b1`: stock sliders crash again.

So the row edits are safe and the draw hook is the crash. The owned record at
`b3001740` was verified: cell 0 read 90 after a decrease, with a valid tag.

### New display path

The draw hook is dropped. Instead the getter and setter write the tick into the
native parameter array `0x70101253[0]` and the display mirror `0x70101258`,
which is exactly where the native editor reads its value. The image path keeps
using the neutral processing adapter.

### Package

- `work/opacity-native/slider-package-20261124T171314Z/GM1__V11.bin`
- package SHA-256 `bc7704115f6c3f5bdf062cf0827095c572555024db518fcdcdb9fd831c1f6566`
- program SHA-256 `5acdc2f15d84f881d348c7457c49a00a568c6fffc984cb748e30feca7079a413`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Opacity display mirror written from the stored tick

**Phase:** Phase 4 - restore opacity

### Result of the array-display build

- Stock sliders stayed usable (no draw hook).
- The owned record moved: cells 0 and 4 read 90.
- The display stayed at `+5` and the parameter array stayed
  `[5,5,5,5,0]`, while the mirror `0x70101258` stayed 10.

So the number is driven by the mirror, the values adapter overwrites the
parameter array, and the previous setter read the record back through a path
that returned the pre-store tick.

### Change

`op_set` now keeps the incoming tick in A3 across `op_rec_set` and writes that
tick to both `0x70101253[0]` and the mirror `0x70101258`; there is no read-back.

### Package

- `work/opacity-native/slider-package-20261124T172509Z/GM1__V11.bin`
- package SHA-256 `9b934be6377d9dee00b291fb41962d37a3ec3cd3af33f355df14f7ca880d0125`
- program SHA-256 `5bff210f9b14a6996c825fa084fc04a3896ad0304bc3bcc6b4f2392548893942`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Opacity slider moves; re-entry resets the display

**Phase:** Phase 4 - restore opacity

### Result

The trace build (`2b18e99d...`) finally moved the display: pressing decrease
took the bar and number from `+5` to `+4`.

New defect: leaving the adjustment page to the profile/LUT list and entering it
again shows `+5` again.

### Likely cause

The setter store and the mirror write both work during the page. The revert is
on page rebuild:

- the native page transaction rebuilds/restores the parameter array and the
  display mirror, which are neutral (`5` / `10`) for a LUT style;
- the owned record is not consulted when the page is rebuilt, so the display
  starts from the neutral value.

Two possibilities remain, and the next read-only dump distinguishes them:

1. the record at `b3001740` still holds `90`, and only the display seed is
   missing;
2. the library object is re-created on page entry, so the record is reset to
   `100`.

### Next step

Reconnect the camera and dump:

- `b3001740..b300175f` (record and trace);
- a pool scan for `GML1` objects, to see whether the library moved.

If the record survives, add a page-entry seed: hook the parameter builder,
compute the tick from the record, and write `0x70101253[0]` and `0x70101258`.
If the record is reset, move the strength to a store tied to the style rather
than the library instance (native saved parameters or the bank cell header).

---

## 2026-11-24 - Seed the display on page entry

**Phase:** Phase 4 - restore opacity

### Dump result

The read-only dump after reproducing the re-entry reset proved:

- record `b3001740`: cell 4 = 90, tag valid;
- setter trace: 21 hits, last tick 9, immediate mirror read 9, immediate array
  read 9;
- pool: `GML1` still at `b3000420`, so the library did not move;
- after the page was left: array 5, mirror 10.

So the record and the setter are correct. Only the page rebuild resets the
display, and re-entry does not consult the record.

### Fix

Added a parameter-builder hook at `0x400e20db` (`op_parameters`):

- loaded LUT: call the shipped list builder (which keeps row 0 and widget
  `0x1604` active through the three byte edits), then compute the tick from
  the record and write `0x70101253[0]` and the mirror `0x70101258`;
- stock styles: replay the original builder unchanged.

### Package

- `work/opacity-native/slider-package-20261124T175538Z/GM1__V11.bin`
- package SHA-256 `67cc3581ff9061cc740a75151a19778392efd526a0c7ecfecb52d29fbd1ceab3`
- program SHA-256 `22cc59f7dd43f479c6cf6ae235e53555449662579561711d47226cf476804467`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Parameter wrapper repaired

**Phase:** Phase 4 - restore opacity

The page-entry seed build exposed two wrapper mistakes:

- it called only the backend helper, so the shipped adapter's row hiding never
  ran and LUT styles showed four adjustment rows;
- it let `op_lut_ptr` clear `a0` on the stock path, which the native builder
  needs.

Fix:

- after the helper, hide rows 1..4 and force row 0 to state 4;
- save `a0` in the wrapper's local frame and restore it on the stock path;
- keep the stock path otherwise byte-identical to the shipped adapter;
- seed `0x70101253[0]` and the mirror from the record.

### Package

- `work/opacity-native/slider-package-20261124T180544Z/GM1__V11.bin`
- package SHA-256 `f3bbbba1f15f6913f4bbd24b83b033468a41af5ae1ecc5dfd5f4151a396042ec`
- program SHA-256 `ae088d12332132e72687e9123cd306c9c7eedf2aa6c916b746fe9af842fa51a0`
- stock restore `cad4ea27...c007`

---

## 2026-11-24 - Seed after calling the shipped parameter adapter

**Phase:** Phase 4 - restore opacity

The previous wrapper still crashed stock profiles, so its own stock path was
abandoned. The new wrapper calls the shipped adapter `0x4080803d` with the
original 16-byte frame, saves its return registers, and only then seeds the
display from the record. Stock behavior is therefore exactly the shipped code.

Row 0 remains active because the shipped adapter's own row-0 value edit is in
place, and rows 1..4 stay hidden.

### Package

- `work/opacity-native/slider-package-20261124T181639Z/GM1__V11.bin`
- package SHA-256 `4783449d9684e02736bbce63975d1c6e565509bf1cb49224fab4e5c6c39e624b`
- program SHA-256 `9a19b3d3ea53d640e9b20351f1e92181da770561754efaf8872ba27523330b27`
- stock restore `cad4ea27...c007`

---

## 2026-11-25 - Revert to the known-good LUT build; stop blind flashing

**Phase:** Phase 4 - restore opacity

### Revert

The camera goes back to the upstream LUT package that had working stock
profiles and stock sliders:

- `work/offline-prepared/luts-20261124T075549Z/GM1__V11.bin`
- package SHA-256 `4c34eaf367451313775ffed5688eb9c8a9f25dabb5ddeffb4dcaefba305f1fa2`
- this keeps the ten LUT cells and the Library; it has no opacity UI.

Stock restore remains `cad4ea2739407f3d59d919e87830fde4e1bad03cb3c26d87c53f7e734a45c007`.

### What went wrong

The stock-profile crash correlates exactly with wrapping the parameter builder.
Even the last wrapper, which called the shipped adapter first, still ran the
seed check (`op_lut_ptr` -> `op_core_ui`) on stock styles. That call clobbers
`a1`, which the wrapper did not save. The caller's `a1` is live, so the stock
page crashed. Earlier wrappers had the same class of problem with `a0`.

This is an ABI mistake, not a menu-design problem: the shipped adapters depend
on argument registers that our wrappers clobbered.

### Offline plan before the next camera build

1. **Keep the proven-safe baseline**
   - four pass-through hooks: getter, setter, decrease, increase;
   - the three in-place row/widget byte edits;
   - no draw hook, no parameter/widget adapter replacement.
   This combination was camera-proven: stock profiles and sliders work, the LUT
   row appears, and a slider change stores the record (`cell 4 = 90`).

2. **Seed only on the LUT entry path**
   - hook the title entry `0x4010ab73`, which runs only when entering the
     adjustment page from the profile list;
   - save `d0,d1,d2,d3,a0,a1,mdr,psw`;
   - check style/mode/mask inline with no calls;
   - for a LUT, compute the tick and write the mirror; otherwise skip;
   - restore every register, then jump to the original title adapter.
   Stock pages never execute the seed.

3. **ABI audit before assembly**
   - for every hooked site, record the caller's live registers and the frame;
   - prove the wrapper saves/restores them all, including `a1`;
   - verify the stack is balanced on every path.

4. **Offline verification**
   - disassemble the patched program and diff only the intended bytes;
   - simulate the tick/record math;
   - confirm the seed path is unreachable for styles outside 7..16.

5. **One instrumented camera session**
   - add a getter/page-build counter in the owned library tail;
   - test entry, change, leave, re-enter in a single session;
   - dump the record, mirror and counters once.

No further camera writes until steps 1-4 pass.

---

## 2026-11-25 - Offline entry-seed design and ABI audit

**Phase:** Phase 4 - restore opacity

### Design

The camera-proven-safe base is kept:

- four pass-through hooks: getter `0x4020f5f9`, setter `0x4020f620`, decrease
  `0x4010aa81`, increase `0x4010aafa`;
- three in-place row/widget byte edits;
- no draw hook and no parameter/widget adapter replacement.

One new hook, on the LUT adjustment entry `0x4010ab73`:

- saves `d0-d3`, `a0`, `a1`, `other`, `mdr`, `psw` before any call;
- checks style 7..16, mode != 2, owner and mask bit inline, with no calls;
- for a loaded LUT, computes the tick from the owned record and writes
  `0x70101253[0]` and the mirror `0x70101258`;
- restores every register and jumps to the original title adapter
  `0x40808400`.

Stock styles therefore execute the original adapter with no seed writes and
with all registers restored. The previous crashes were ABI clobbers
(`a0`, later `a1`) in the wrapper; this one saves the full set.

The seed also records two counters in the owned library tail:
`b3001750` seed hits, `b3001751` last seeded tick, `b3001752` getter hits,
`b3001753` last getter tick.

### Offline verification

`work/opacity-native/slider/audit_slider.py` passes:

- only the declared hooks, byte edits and module differ from upstream;
- every hook resolves to its symbol;
- the title seed save order precedes the first call, which is `op_tick`;
- the restore order covers all saved registers and ends with the tail jump;
- the seed writes exactly the two display bytes.

### Package (built, not copied, not flashed)

- `work/opacity-native/slider-package-20261124T184237Z/GM1__V11.bin`
- package SHA-256 `225d81ccf252806fa9bc0ce89a4da3e883c0247b89da225a57a810690df7a832`
- program SHA-256 `6b66af8379456e83d3515e78cf1efd14c7f78f4591568c1cd90b8a795d34aa88`
- stock restore `cad4ea27...c007`

No camera write until the owner asks for the next test.

---

## 2026-11-25 - Getter ring build and candidate matrix

**Phase:** Phase 4 - restore opacity

### Candidate matrix (key deltas)

| Build | Record | Setter mirror | Draw | Seed | Stock | LUT |
|---|---|---|---|---|---|---|
| upstream | - | - | - | - | ok | n/a |
| F21A | library tail | yes | no | none | ok | static `+0` |
| 2b18 | library tail | yes (tick) | no | none | ok | `+5 -> +4`, reverts on re-entry |
| 69dc | library tail | yes | `0x403244b1` | none | crash | `+5/+0` |
| 4783 | library tail | yes | no | parameter wrapper | crash | `+5 -> +4`, reverts |
| this | library tail | yes (tick) | no | title entry | expected ok | seed + move |

Derived rules:

- stock crashes only appear with the draw hook or a builder wrapper, both of
  which clobbered caller registers;
- the move works only when the setter writes the mirror from the stored tick;
- the re-entry reset is a mirror write by one of the five page-event getter
  stores, returning a stale tick;
- the record and the input path are no longer unknowns.

The only remaining unknown is which style/cell the getter resolves on the page
entry/exit calls.

### Diagnostic

`op_get` now keeps the last four getter calls in the owned library tail, newest
first at `b3001760`: three bytes per entry `(style, cell, tick)`.

Together with the seed trace (`b3001750..53`), one session shows:

- whether the title seed ran and with what tick;
- which style/cell the page-entry getter used;
- whether the mirror ended at the getter's tick or was overwritten.

### Package

- `work/opacity-native/slider-package-20261125T051855Z/GM1__V11.bin`
- package SHA-256 `c4f19cfefdf5288b3aaea348cf09111fae8b68010187ae84dccce9d5345525a9`
- program SHA-256 `16b130b37e9853971e7347abb6b3b1277b0260a1d949392a24a186ae3ad27da5`
- stock restore `cad4ea27...c007`

---

## 2026-11-25 - Getter bound inversion found and fixed

**Phase:** Phase 4 - restore opacity
**Build/package:** `slider-package-20261125T053717Z`, package SHA-256
`8b954bbb1552bcb0376b6b2e0b1810434ba2e58a004887b7811e2734db592e29`
**Camera state:** DMC-GM1, official 1.1, internal `0110`, ROM/RAM policy `0`

### Defect

Candidate `c4f19c` moved `+5 -> +4` with buttons, dial and touch, but could
not move back `+4 -> +5`; selecting another LUT restored `+5`. The stored
record was `90` for the changed cell.

### Root cause

`op_rec_get` used:

```
cmp 10,d0
bcs .Lrg_default
```

In this codebase `bcs` is the proven unsigned *lower-than* condition; the
same condition is used correctly by `cmp 7,d1; bcs` to reject stock styles
below 7. Therefore every valid cell `0..9` branched to the 100% default
before the record was read. The setter wrote the record correctly, but every
later read returned `100`, so `op_tick` always returned `10`.

This one instruction explains the whole candidate pattern:

| Candidate | Getter behavior | Observed |
|---|---|---|
| F21A | never seeded | static `+0` |
| 2b18 | always `100` | `+5 -> +4`, no return, re-entry `+5` |
| 69dc | always `100` | draw hook crash; bar moved |
| 4783 | always `100` | wrapper crash; same revert |
| ring `c4f19c` | always `100` | `+5 -> +4`, no return; ring logs tick `10` |

The draw-hook and builder-wrapper crashes were separate ABI clobbers and are
not part of the remaining defect.

### Fix

- `op_rec_get` now uses `cmp 9,d0; bhi .Lrg_default`, so cells `0..9` read
  the record and only out-of-range cells default to 100.
- `check_slider.py` now fails if the inverted lower-than bound returns.
- Diagnostic counters separated so one dump is unambiguous:
  - `b3001750/51` seed hits/tick;
  - `b3001752/53` getter hits/tick;
  - `b3001754/55` setter hits/tick;
  - `b3001770..74` step hits/direction/before/after/cell;
  - `b3001775..77` record-get cell/init/raw byte;
  - `b3001760..6b` getter ring `(style, cell, tick)`.

### Verification

- build, placement, hook and ABI audit pass;
- package SHA-256 `8b954bbb...592e29`;
- program SHA-256 `4358340a...2ce0ff`;
- copied to the card as `D:\\GM1__V11.bin`, hash verified;
- previous candidate backed up as
  `work/card-backup/GM1__V11-before-getter-fix.bin`
  (SHA-256 `c4f19cfe...5525a9`), not yet flashed.

### Expected camera behavior

- stock profiles and stock sliders unchanged;
- fresh LUT records show `+5`; stored records show their own tick;
- both directions work across `0..10`;
- leaving and re-entering the adjustment page preserves the stored tick;
- the image is still unchanged until the cached preblend path lands.

---

## 2026-11-25 - Native opacity UI verified on camera

**Phase:** Phase 4 - restore opacity
**Build/package:** `slider-package-20261125T053717Z`, package SHA-256
`8b954bbb1552bcb0376b6b2e0b1810434ba2e58a004887b7811e2734db592e29`
**Camera state:** DMC-GM1, official 1.1, internal `0110`, ROM/RAM policy `0`
**Result:** pass

### Observed

- stock profiles and stock sliders: normal, no crash;
- LUT adjustment row: `-5..+5` in both directions with buttons, dial and
  touch;
- setting the value persists; leaving and re-entering the adjustment page
  does not revert to `+5`;
- image path is still unchanged, as intended for this stage.

### Dump

| Item | Value |
|---|---|
| library record `b3001740` | `[50, 100, 100, 100, 100, 100, 100, 100, 100, 100]` |
| library tag | `0x4c53` |
| seed hits / tick `b3001750/51` | `4 / 5` |
| getter hits / tick `b3001752/53` | `5 / 5` |
| setter hits / tick `b3001754/55` | `37 / 5` |
| step hits / dir / before / after / cell | `37 / 0 / 6 / 5 / 0` |
| record-get cell / init / raw byte | `0 / 0 / 60` |
| getter ring (newest) | `(style 7, cell 0, tick 5)` |
| parameter array / mirror | `[5, 5, 5, 5, 0] / 5` |
| current row / selected style | `0 / 7` |

### Interpretation

- The record-get raw byte is `60` because the final decrease read the record
  at tick 6 before the setter stored tick 5. The record now reads `50`.
- `init=0` means the record was already valid and was read, not defaulted to
  100.
- The step count equals the setter count (`37`), so every step reached the
  setter.
- The seed tick and getter tick are both `5`, matching the stored `50`, so
  page entry and re-entry use the record value.
- The inverted-bound fix is confirmed on camera.

### Next

- keep `8b954b` as the native UI baseline;
- apply the per-cell strength in the cached table-domain blend, with no
  per-frame regeneration;
- verify visually and by comparing the blended bank against the expected
  table.

---

## 2026-11-25 - Stage B mapping: bank, pool leases and live core

**Phase:** Phase 4 - cached image path
**Build/package:** none (read-only analysis)
**Camera state:** DMC-GM1, official 1.1, internal `0110`; service USB dropped
before the live-core read

### Bank format

- header `GM5LUT2`, version 2, ten cells, generation and mask;
- cell record stride `0x8b0`, payload `0x88c` (2,188 bytes), cell 0 payload
  at `+0x60`;
- bank A generation 22, mask `0xff`, seven named cells;
- the bank payloads are the original full-strength cell tables.

### Pool10

- `GML1` at `0xb3000420` (6,144 bytes) is library metadata (names, mask and the
  owned strength record at `0xb3001740`), not the table data;
- cell lease descriptors start at `0xb3002900`: `LIL1`, size `0x92c`
  (2,348 bytes), lease pointers `0xb3011800 + n*0xa00`;
- each bank payload appears in the lease area at `0xb3011860 + n*0xa04`,
  after a `0x60 + n*4` header. The leases are runtime copies of the saved
  cells.

### Live core

The validator at `0x40806009` checks a fixed 8 KiB object at `0x708f52f0`:

- core header, size 8,192;
- `GTL1` at `+0x20` (size 336, data area 288);
- `GML1` at `+0x180` (size 128);
- `GML1` at `+0x220` (size 6,144).

This is the live runtime library; the pool `GML1` is its backing copy. The
active-table pointer is inside `GTL1`/`GML1`, so one read-only dump of
`0x708f5000..0x708f7400` is needed to identify it.

### Next

1. Wake the camera and dump the live core.
2. Parse `GTL1` and map style `7..16` to cell and active table pointer.
3. Copy the original bank cell to the active table and blend with the stored
   strength, once per set/cell change/startup.
4. Build and audit offline; no card or flash yet.

---

## 2026-11-25 - Stage B: opacity applied to the active table

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T062028Z`, package SHA-256
`0d1a98ccb13248bcc36b6b148fc77102be30e8c5cab82c8e9b7d3e4a0498dd4c`
**Camera state:** DMC-GM1, official 1.1, internal `0110`, ROM/RAM policy `0`;
SD card not mounted at package time

### What the old 0.2.4 build did

The old build's cell-adoption function at `0x407f41f3` is the image-table
path:

- `0x407f4070` returns the active table (2,284 bytes) and its flat data
  buffer (2,188 bytes);
- `0x40792000` returns the source cell table;
- the current function copies the source table to the active table and
  rebases the eight bank descriptors (`0x407f42be..f2`);
- the old build then blended the active data with the embedded neutral
  table at `0x407a2000` and copied the result to the flat data buffer;
- the blend ran while the job state was 3 (busy), then the function
  published state 2 (ready).

The old blend routine takes `(destination, source, percent)` and blends
Y bytes plus the two packed 6-bit Cb/Cr components. At 50% its math is
`(a+b+1)/2`, exactly the proven fixed-50% result.

### This build

- module grew from 1,210 to 4,072 bytes at `0x40840000`;
- embedded camera-neutral payload (2,188 bytes, SHA-256
  `034a50ef...3575b`);
- new adoption hook `0x407f42f4 -> 0x4084060c`: runs after the copy and
  descriptor rebase, saves/restores all registers, reads the cell's record
  percentage, blends the active table from the original source, copies the
  result to the flat data buffer, then replays the two overwritten
  instructions and jumps to `0x407f42f9`;
- new `op_reblend_cached`: called at the end of `op_set` for the same cell,
  re-blends the cached active/source pointers at the new percentage and
  copies to the flat buffer;
- the frame path is unchanged; there is no per-frame blend.

Stage B cache/trace in the library tail:

| Address | Meaning |
|---|---|
| `b3001780` | active table pointer |
| `b3001784` | flat data pointer |
| `b3001788` | source table pointer |
| `b300178c` | cached cell |
| `b300178d` | last percentage |
| `b300178e` | adoption blends |
| `b300178f` | setter reblends |

### Offline gates

- patch, placement and hook encoding pass;
- adoption hook save/restore/replay pass;
- variable blend contains both divisions and the chroma repack;
- embedded neutral payload matches the known hash;
- cache addresses and the `op_set` reblend call are checked.

Package:

- `work/opacity-native/slider-package-20261125T062028Z/GM1__V11.bin`
- package `0d1a98cc...8dd4c`;
- program `80243c32...5c1dc`;
- stock restore `cad4ea27...c007`.

### Camera test

1. Install and reboot.
2. Stock profile and stock sliders: normal, no crash.
3. LUT cell: set opacity to 100 (full), 50, then 0 (neutral). The image
   must change at each step; 0 must remove the LUT look.
4. Move back and forth, leave and re-enter the page.
5. Reconnect USB for the dump.

Interpretation:

- `adoption blends > 0` and `setter reblends > 0`: both paths ran;
- if the image does not change while the counters move, the producer is
  reading a different copy than the active/flat buffers;
- if the counters stay zero, the adoption path or the setter call is not
  reached.

No card write or flash yet.

---

## 2026-11-25 - Portra samples expose native contrast contamination

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T063816Z`, package SHA-256
`2d24738680b46d58a2ac7c8284084e2e16c245e2bd30634e21181b6137416dff`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted

### Sample result

The three Portra samples show the opacity ordering clearly:

| Sample | Mean Y | Crop Y std | EXIF Contrast | Exposure |
|---|---|---|---|---|
| `Portra+5.JPG` | 126.9 | 63.1 | 2 (high) | 1/160 |
| `Portra+0.JPG` | 123.9 | 53.9 | 0 (normal) | 1/200 |
| `Portra-5.JPG` | 127.0 | 49.1 | 1 (low) | 1/200 |

The crop contrast falls monotonically, so the LUT opacity is applied. But
the EXIF `Contrast` tag changes exactly with the tick (`10 -> high`,
`5 -> normal`, `0 -> low`), and the `+5` shot is 1/3 stop brighter.

Cause: the UI still wrote the native contrast parameter array
`0x70101253[0]` in `op_get`, `op_set` and `op_title_seed`. That array feeds
the camera's native contrast processing, so the slider was changing both
opacity and native contrast. The samples are therefore not a pure opacity
test.

### Fix

- `op_get`, `op_set` and `op_title_seed` no longer write `0x70101253[0]`;
  they keep only the display mirror `0x70101258`.
- the shipped LUT values adapter keeps the native parameter array neutral
  (`5,5,5,5,0`);
- the old 0.2.4 display substitution is restored at `0x403244b1`
  (`jmp op_draw_value`), using the same safe trampoline as the old build:
  save `d0,d1,a0,mdr,psw`, substitute the record tick only for a loaded LUT
  style, otherwise use the stock `0x702250fb` map, restore, set `a1`, and
  continue at `0x403244b9`;
- the number display therefore moves without touching the native image
  parameter.

### Package

- `work/opacity-native/slider-package-20261125T063816Z/GM1__V11.bin`
- package `2d247386...16dff`;
- program `e2098d85...4192d`;
- stock restore `cad4ea27...c007`;
- previous Stage B build `0d1a98cc...8dd4c` backed up on the PC.

### Offline gates

- native array write removed from the title seed; the seed writes only the
  mirror;
- draw hook encoding and trampoline checked;
- adoption hook, variable blend, embedded neutral payload and setter
  reblend still pass;
- placement and stock-hook encoding pass.

### Camera test

1. Install and reboot.
2. Stock profile and stock sliders: normal, no crash.
3. LUT cell: the number/bar must still move `-5..+5`.
4. Image: `+5` full LUT, `0` mix, `-5` neutral. The native contrast should
   no longer swing with the tick; check the EXIF `Contrast` tag and exposure
   if possible. For a clean comparison, use manual exposure or exposure lock
   so the three shots have the same shutter speed.
5. Reconnect USB for the dump.

The blend remains one-shot per adoption/step, not per frame. The observed
adjustment delay is the table computation on each step; it can be reduced
later with the old lookup table if needed.

---

## 2026-11-25 - Ricoh samples confirm decoupling; number text hook fixed

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T065445Z`, package SHA-256
`a9e6ca975d18c7781230092b8fa365d1c8235d1a67bb879737274befca55ea80`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted

### Ricoh samples

All three Ricoh shots have identical exposure and neutral native contrast:

| Sample | Exposure | ISO | Contrast | Crop Y mean/std |
|---|---|---|---|---|
| `Ricoh+5.JPG` | 1/200 | 200 | 0 | 102.6 / 66.6 |
| `Ricoh+0.JPG` | 1/200 | 200 | 0 | 99.3 / 67.2 |
| `Ricoh-5.JPG` | 1/200 | 200 | 0 | 102.1 / 68.7 |

The native `Contrast` tag stays `0` and the exposure is identical, so the
Stage B2 fix decoupled opacity from the stock contrast. The crop contrast
is nearly constant; the LUT effect is color/tonal, not a native contrast
swing. The pixel-wise midpoint is not a clean test because the three shots
are not registered and the scene can move; the EXIF and crop statistics are
the reliable evidence.

### Why the number did not move

The bar follows the display mirror, but the numeric text comes from a
separate site at `0x40324731`. The dump showed:

- draw raw `0x702267d8 = 0` (row 0);
- selected style `11`;
- row-value array `0x702250fb = [5,5,5,5,0,0,11,2,...]`;
- parameter array `[5,5,5,5,0]` (neutral);
- mirror `0`.

The text site reads the neutral row-value array, so the number stayed at 5
even though the value/bar hook returned the tick.

### Fix

- new `op_draw_label` hook at `0x40324731` (6-byte site):
  - save `d0,d1,a0,a1,mdr,psw`;
  - for row 0 of a loaded LUT, read the tick and jump to the stock text
    lookup `0x4032473f` with the stock native text table `0x404604b2`;
  - otherwise set `a0 = 0x702250fb` and jump to the original stock path
    `0x40324737`;
- the number therefore uses the same `-5..+5` labels as the bar, not a
  private `0..10` map.

### Package

- `work/opacity-native/slider-package-20261125T065445Z/GM1__V11.bin`
- package `a9e6ca97...5ea80`;
- program `574379a3...e410f`;
- stock restore `cad4ea27...c007`;
- previous B2 build `2d247386...16dff` backed up on the PC.

### Camera test

1. Install and reboot.
2. Stock profiles and sliders: normal, no crash.
3. LUT cell: both the number and the bar must move `-5..+5`.
4. Image: `+5` full LUT, `0` mix, `-5` neutral. Native `Contrast` should
   stay `0`.
5. Reconnect USB for a dump if anything still disagrees.

The dump also showed the record reset to 100 and the Stage B counters at 0,
so power-cycle persistence is still pending; that is separate from this
display fix.

---

## 2026-11-25 - Fuji samples clean; stock draw path uses page style

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T070957Z`, package SHA-256
`4d24b64c26fea7e33535c70d6c1291fdf96bf77f58a5f3590a35642111d6ceb9`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted

### Fuji samples

| Sample | Exposure | Aperture | ISO | Contrast |
|---|---|---|---|---|
| `Fuji+5.JPG` | 1/200 | F2.2 | 200 | 0 |
| `Fuji+0.JPG` | 1/200 | F2.2 | 200 | 0 |
| `Fuji-5.JPG` | 1/200 | F2.5 | 200 | 0 |

The native contrast is neutral at all three; the `-5` shot is darker only
because the aperture stopped down to F2.5. The LUT opacity is decoupled from
the stock contrast.

The camera dump at the Fuji cell (style 9, cell 2):

- record cell 2 = 0, others 100;
- adoption blends 219, setter reblends 10;
- parameter array `[5,5,5,5,0]`, mirror 0;
- draw raw 0, page style 9, row-value array `[5,5,5,5,0,0,9,2,0,14,10,...]`.

The image path and the LUT display are working.

### Stock-profile regression

After the B3 text-map hook, stock profiles broke. The B2/B3 draw hooks read
the selected style from `0x70101252` (`STYLE`). That variable can still hold
the last LUT style while a stock profile page is open. The old 0.2.4 draw
trampolines used the page-scoped style byte `0x702250fa` instead, and were
stock-safe.

Change in B4:

- both `op_draw_value` and `op_draw_label` now read `0x702250fa`;
- the getter/setter/seed keep `0x70101252`, matching the old build's row
  selection;
- the stock paths still use `0x702250fb` and the original continuations.

### Package

- `work/opacity-native/slider-package-20261125T070957Z/GM1__V11.bin`
- package `4d24b64c...6ceb9`;
- program `79cfe76e...374de5`;
- stock restore `cad4ea27...c007`;
- B3 build `a9e6ca97...5ea80` backed up on the PC.

### Camera test

1. Install and reboot.
2. Stock profiles: select one, open its adjustment page, move its sliders.
   The page must be normal and the sliders must apply.
3. LUT cell: number and bar still move `-5..+5`; image still follows the
   opacity; native `Contrast` stays 0.
4. If stock still breaks, report whether it is a crash, a wrong number/bar,
   or a non-responsive slider, and leave a stock profile page open for a
   dump. The dump's `draw raw`/`draw style`/row-value array will show whether
   the draw hook is still taking the LUT path.

---

## 2026-11-25 - Stock switching restored; draw hooks removed

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T072316Z`, package SHA-256
`b098ebd73526d75f2f536b94060577733dc06cc387710933f04b56949fecac5a`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted

### Stock regression

B4 still could not switch to stock profiles at all. The two draw hooks
(`0x403244b1`, `0x40324731`) are on the stock drawing path. Even with the
page-style byte `0x702250fa`, they are not safe there.

B5 removes both draw hooks:

- `op_draw_value` and `op_draw_label` remain in the module as unused code;
- the stock drawing path is exactly the stock code again;
- the LUT bar still follows the display mirror `0x70101258`;
- the adoption/setter blend and the getter/setter/seed hooks remain;
- the numeric readout may stay at the native value; the bar and blend are
  the required behavior.

### Fuji samples

| Sample | Exposure | Aperture | ISO | Contrast |
|---|---|---|---|---|
| `Fuji+5.JPG` | 1/200 | F2.2 | 200 | 0 |
| `Fuji+0.JPG` | 1/200 | F2.2 | 200 | 0 |
| `Fuji-5.JPG` | 1/200 | F2.5 | 200 | 0 |

Native contrast is neutral at all three; the `-5` shot is darker only
because the aperture stopped down to F2.5. The blend endpoints and the
native-contrast decoupling are confirmed.

### Row label

The camera's guires resource contains no English UTF-16 `Opacity` string,
and the `Blend` hits are German aperture text (`Blenden`). A `Blend
Opacity` label therefore needs either:

- a custom UTF-16 string plus a label-render hook, or
- a spare text ID / text-table entry for the LUT row.

The row-label descriptor dump was not taken: the camera went to sleep before
the read. The next session should leave a LUT adjustment page open and
reconnect so the live row descriptor and text ID can be mapped.

### Package

- package `b098ebd7...cac5a`;
- program `89a58bd1...1baf4`;
- stock restore `cad4ea27...c007`;
- B4 build `4d24b64c...6ceb9` backed up on the PC.

### Camera test

1. Install and reboot.
2. **Stock profiles first**: switching must work and their sliders must
   operate.
3. LUT cell: the bar must move `-5..+5`; the image must follow the blend.
4. The number may stay at the native value; the label is still `Contrast`.
5. Reconnect on a LUT adjustment page for the row-label dump.

---

## 2026-11-25 - Bar-only draw hook restored

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T073525Z`, package SHA-256
`ab0b07763acf1c0789cdeb67aeefe8628a7fec13424cb062994a194375230fa1`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted,
camera not connected

### Change

B5 restored stock switching by removing both draw hooks, but the bar stopped
moving. B6 restores **only** the value draw hook at `0x403244b1`:

- page-scoped style byte `0x702250fa`, matching the old stock-safe
  trampoline;
- stock path keeps the original `0x702250fb` value map;
- the text-map hook at `0x40324731` remains off, so the number may stay
  native;
- the bar and the opacity blend are preserved.

### Package

- `work/opacity-native/slider-package-20261125T073525Z/GM1__V11.bin`
- package `ab0b0776...30fa1`;
- program `0c215a1f...e2256d`;
- stock restore `cad4ea27...c007`;
- B5 stock-safe build `b098ebd7...cac5a` backed up on the PC.

### Label mapping

The English parameter-name pointer table is at guires `0x112a80`:

- `0x112abc` -> `0x40b575f8` -> `"Contrast"`;
- adjacent entries are `"Sharpness"`, `"Saturation"`, `"Noise Reduction"`.

The row descriptor format from the state helper is eight bytes:

- `u16 id`, `u8 state`, `u8 flags`, `u32 label`.

The live row list is at owner `0x708f47a4` + 160 = `0x708f4844`. The camera
was not connected for the descriptor read, so the label field is not yet
identified. The parser is ready in `work/analyze_row_labels.py`.

### Test

1. Stock profiles first: switching and sliders.
2. LUT cell: bar `-5..+5`, blend at `+5/0/-5`; number optional.
3. If stock breaks with only the value hook, revert to B5 and use another
   bar source.
4. Reconnect with a LUT adjustment page open to dump the row label field.

---

## 2026-11-25 - Revert to B2 after stock-slider crash

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T063816Z`, package SHA-256
`2d24738680b46d58a2ac7c8284084e2e16c245e2bd30634e21181b6137416dff`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted

B6 (bar-only value hook using `0x702250fa`) made stock profiles selectable
but crashed the stock slider to preview with `Sleep cancel`. B5 had stock
working but no bar. The last candidate matching both stock and LUT
bar/blend is B2, the Ricoh-sample build:

- value draw hook at `0x403244b1` using `STYLE` (`0x70101252`);
- no text-map hook, no draw-label hook;
- native parameter array neutral;
- adoption and setter table blend;
- number stays native; bar and blend work.

Reverted the card to B2 and verified the hash. B6 is backed up on the PC.

The stock regressions are isolated to the later drawing changes:

- B3 added the text-map hook -> stock switching broke;
- B6 changed the value hook to `0x702250fa` -> stock slider crashed.

The B2 value hook uses `STYLE`, which is the version that matches the
requested working combination.

Label rename remains pending; it needs a LUT-page row-label dump and a
LUT-only custom text entry.

---

## 2026-11-25 - Matrix-based revert to Stage B original

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T062028Z`, package SHA-256
`0d1a98ccb13248bcc36b6b148fc77102be30e8c5cab82c8e9b7d3e4a0498dd4c`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted

### Candidate matrix

| Build | Native array | Value hook | Text hook | Stock switch | Stock sliders | LUT bar | LUT number | Contrast decoupled |
|---|---|---|---|---|---|---|---|---|
| UI `8b954b` | yes | no | no | ok | ok | yes | yes | n/a |
| Stage B `0d1a98cc` | yes | no | no | ok | ok | yes | yes | no |
| B2 `2d247386` | no | yes | no | ok | crash | yes | no | yes |
| B4 `4d24b64c` | no | yes | yes | ok | crash | yes | yes | yes |
| B5 `b098ebd7` | no | no | no | ok | ok | no | no | yes |
| B6 `ab0b0776` | no | yes | no | ok | crash | yes | no | yes |

The matrix shows the trade-off clearly:

- the bar follows either the native array write or the value draw hook;
- the value draw hook is present in every stock-slider crash
  (B2/B4/B6);
- the native array write restores the bar and number but also feeds the
  native contrast (Stage B original);
- B5 is the only stock-safe, contrast-decoupled build, but it has no bar;
- there is no build with stock sliders + LUT bar + decoupled contrast.

The requested combination (stock sliders + LUT bar/blend) is therefore
**Stage B original `0d1a98cc`**, accepting the known native-contrast
coupling. No new modification was made.

### Package

- `work/opacity-native/slider-package-20261125T062028Z/GM1__V11.bin`
- package `0d1a98cc...8dd4c`;
- program `4358340a...2ce0ff`;
- stock restore `cad4ea27...c007`;
- B2 build `2d247386...16dff` backed up on the PC.

### Test

1. Stock switching and stock sliders.
2. LUT cell: bar/number `-5..+5`, blend at `+5/0/-5`.
3. The native Contrast tag will move with the tick again; that is the
   known Stage B coupling, not a new regression.

A future LUT-only bar path could remove the coupling without the value hook,
but that is a separate design task, not a blind patch.

---

## 2026-11-25 - Matrix verification and next complete-candidate plan

**Phase:** Phase 4 - cached image path
**Build/package:** none (verification and design only)
**Camera state:** DMC-GM1, official 1.1, internal `0110`; no flash today

### Verified matrix

| Build | Native array | Value hook | Text hook | Stock switch | Stock sliders | LUT bar | LUT number | Contrast decoupled |
|---|---|---|---|---|---|---|---|---|
| UI `8b954b` | yes | no | no | ok | ok | yes | yes | n/a |
| Stage B `0d1a98cc` | yes | no | no | ok | ok | yes | yes | no |
| B2 `2d247386` | no | yes | no | ok | crash | yes | no | yes |
| B4 `4d24b64c` | no | yes | yes | ok | crash | yes | yes | yes |
| B5 `b098ebd7` | no | no | no | ok | ok | no | no | yes |
| B6 `ab0b0776` | no | yes | no | ok | crash | yes | no | yes |

The two closest builds are:

- **Stage B `0d1a98cc`**: stock sliders + LUT bar/number + blend, but the
  native Contrast is coupled;
- **B5 `b098ebd7`**: stock sliders + blend + decoupled Contrast, but no
  bar/number.

B2 and B6 behave the same: stock switching works, stock sliders crash,
LUT bar/blend works, number stays native.

### Root-cause correlation

The value draw hook is present in every stock-slider crash and absent in
both stock-safe builds. The likely mechanism is that the hook can take the
LUT path on a stock page because the style/mask state is still valid from
the LUT cell. The stock path then draws a LUT tick into the stock slider,
which crashes back to preview.

### Planned complete candidate

Base: **B5** (stock-safe, decoupled, no draw hooks, no native array writes).

Add one **strictly page-gated value draw hook**:

- a page flag in the library tail, e.g. `b3001794`;
- `op_title_seed` clears the flag on entry and sets it only when the LUT
  adjustment-page predicate passes;
- `op_get` / `op_set` also set it on the LUT success path and clear it on
  the stock pass-through paths;
- the draw hook checks the flag first:
  - flag 0: execute only the original `a1 = 0x702250fb` map and jump to
    `0x403244b9`, with no LUT call at all;
  - flag 1: save registers, read the record tick, substitute it for the
    bar, restore, continue;
- the native parameter array stays untouched, so the Contrast remains
  decoupled;
- number/label remain optional.

Diagnostic traces for the next session:

- `b3001794` page flag;
- `b3001795` draw-hook path (0 stock / 1 LUT);
- `b3001796` last draw `d0`;
- `b3001797` last page style;
- `b3001798` last tick.

### Verification gate

1. Stock profile: open the slider page. The flag must be 0 and the draw
   hook must take the stock path; the page must not crash.
2. LUT cell: flag 1, bar moves, blend still applies.
3. If stock still crashes with flag 0, the hook's mere presence is unsafe
   and the bar needs a non-hook source; the trace will show this before
   another attempt.

### Label

The English parameter-name pointer table is at guires `0x112a80`;
`0x112abc -> 0x40b575f8 -> "Contrast"`. The live row list is at
`0x708f47a4 + 160 = 0x708f4844`, eight bytes per row
(`u16 id`, `u8 state`, `u8 flags`, `u32 label`). The LUT-page row-label
dump will identify the label field so a LUT-only `Blend Opacity` entry can
be added without touching stock labels.

No firmware was built or flashed for this entry.

---

## 2026-11-25 - No WSL/VM; read-only draw-state test

**Phase:** Phase 4 - cached image path
**Build/package:** none (read-only test preparation)

### Environment check

- WSL is not installed; `wsl -l -v` reports no distribution.
- The MN10300 toolchain is the MSYS2 cross-binutils: assembler, linker,
  objdump, etc.
- There is no `mn10300-elf-run`, no QEMU MN10300 target, and no simulator in
  the workspace.

So the stock-page hypothesis is not VM-verified. The practical dynamic test
without a VM is a read-only dump on the failing stock page, compared with a
LUT page.

### Script

`work/dump_draw_state.py` reads, read-only:

- mode `0x700fcec8`;
- style `0x70101252`, parameter array `0x70101253`, mirror `0x70101258`;
- current row `0x70102930`;
- draw raw `0x702267d8`;
- page style `0x702250fa`, row values `0x702250fb`;
- owner `0x708f47a4` and the eight-byte row descriptors at owner + 160,
  including the `u32 label` field;
- the record and Stage B traces.

### Interpretation

- Stock page shows `draw raw = 0` and `page style` 7..16 with the LUT mask
  set: the value hook would take the LUT path; the page gate is needed.
- Stock page shows a stock style (0..6) or `draw raw != 0`: the hypothesis
  is wrong; the crash is in the hook's stock path/register handling, and a
  gate will not fix it.
- LUT page is the control; its row descriptor also identifies the label
  field for the `Blend Opacity` patch.

No build, copy, or flash was made.

---

## 2026-11-25 - LUT-page control draw dump

**Phase:** Phase 4 - cached image path
**Build/package:** B6 `ab0b0776...30fa1` (read-only dump)
**Camera state:** DMC-GM1, official 1.1, internal `0110`; LUT adjustment
page open

### Control values

| Item | Value |
|---|---|
| mode `0x700fcec8` | `0` |
| style `0x70101252` | `7` |
| parameter array | `[5,5,5,5,0]` |
| display mirror | `10` |
| current row | `0` |
| draw raw `0x702267d8` | `0` |
| page style `0x702250fa` | `7` |
| row values `0x702250fb` | `[5,5,5,5,0,10,7,2,0,14,8,...]` |

The control page satisfies the value hook's LUT precondition: `draw raw 0`,
page style 7 (loaded LUT), mode 0. This is the path that substitutes the
tick and moves the bar.

### Row descriptors

Owner `0x708f47a4 = 0xb3000220`; row list at owner + 160 = `0xb30002c0`.
The live descriptors are:

```
[00] id=0 state=4 flags=0x01 label=0
[01] id=1 state=1 flags=0x01 label=0
...
[05] terminator
```

The `u32 label` field is **zero** for every row. The row descriptor is a
state table only; the `Contrast` label is not stored there. The row-label
rename therefore needs the static parameter-name table/draw path, not a
row-descriptor edit.

The English parameter-name pointer table remains at guires `0x112a80`
(`0x112abc -> 0x40b575f8 -> "Contrast"`), but the code that indexes it has
not yet been located.

### Next

Reproduce the stock slider crash, reconnect, and run the same read-only
dump. The decisive values are `draw raw`, page style, mode, and style on
the failing stock page. If the page has already returned to preview, the
flag/path trace build is required.

---

## 2026-11-25 - Stock-page crash dump and row-state gate

**Phase:** Phase 4 - cached image path
**Build/package:** B7 `05305767...04c80` (prepared, card not mounted)
**Camera state:** DMC-GM1, official 1.1, internal `0110`; B6 flashed

### Control vs crash dump

| Item | LUT page (control) | Stock page after crash |
|---|---|---|
| mode | 0 | 0 |
| style | 7 | 7 |
| draw raw `0x702267d8` | 0 | 0 |
| page style `0x702250fa` | 7 | 7 |
| mirror | 10 | 0 |
| row states | row0=4, rows1..4=1 | rows0..4=4 |

The stock page satisfies the value hook's LUT precondition (`draw raw 0`,
page style 7, mode 0). The difference is the **row-state table**: the LUT
page hides rows 1..4 (state 1), while the stock page exposes all rows
(state 4). The post-crash dump therefore confirms the precondition and
identifies a page discriminator.

### B7 gate

`op_draw_value` now checks the live owner row 1 state before any LUT call:

- owner `0x708f47a4`, row 1 state at owner + 170;
- if state != 1, take the original `0x702250fb` map and jump to
  `0x403244b9`; no `op_lut_percent` call;
- if state == 1, run the existing LUT tick substitution;
- traces: `b3001790` row 1 state, `b3001791` path (0 stock / 1 LUT).

This is a data-backed gate from the live dump, not a style guess.

### Package

- `work/opacity-native/slider-package-20261125T095550Z/GM1__V11.bin`
- package `05305767...04c80`;
- program `b1b18602...dc448`;
- B6 `ab0b0776...30fa1` backed up on the PC.

The card reader showed no media at copy time, so B7 is not on the card yet.

### Test

1. Stock slider: must not crash; gate trace `b3001791` must be 0.
2. LUT page: bar moves; gate trace `b3001791` must be 1.
3. If stock still crashes with trace 0, the hook's stock path itself is
   unsafe and the bar needs a non-hook source.

The row descriptor label field is zero; the `Contrast` label is static, so
the `Blend Opacity` rename remains a separate label-table/draw task.

---

## 2026-11-25 - B7 stock fixed; gate trace blocks LUT bar

**Phase:** Phase 4 - cached image path
**Build/package:** B7 `05305767...04c80` flashed
**Camera state:** DMC-GM1, official 1.1, internal `0110`

### Result

- stock switching and stock sliders: working;
- LUT blend with press/dial/touchscreen: working;
- LUT bar and number: not changing.

### LUT-page dump

| Item | Value |
|---|---|
| style / page style | 7 / 7 |
| draw raw | 0 |
| mode | 0 |
| live row 1 state | 1 |
| record | `[0,0,100,...]` |
| mirror | 0 |
| setter / step hits | 20 / 31 |
| adoption blends / reblends | 245 / 20 |
| draw gate row 1 state `b3001790` | 0 |
| draw gate path `b3001791` | 0 |

The live row state is 1, but the draw hook recorded 0 and took the stock
path. That explains why the bar stays native while the blend still works.
The trace may be stale, or the owner/row base read by the hook is different
at draw time.

### Next

Press once on the LUT adjustment page and re-dump. If the gate trace still
shows 0, add an owner-pointer/draw counter trace in the value hook to
distinguish:

- owner pointer zero at draw time;
- owner pointer valid but row offset/state read wrong;
- draw hook not running after the page build.

No new flash is needed for the re-dump.

---

## 2026-11-25 - Absolute LIT1 row-state gate (B8)

**Phase:** Phase 4 - cached image path
**Build/package:** `slider-package-20261125T101606Z`, package SHA-256
`16f12181f04dd55202b415996e1ba3cef7d316fb3a3d330396259d8255d03d55`
**Camera state:** DMC-GM1, official 1.1, internal `0110`; card mounted

The B7 dump showed the live row 1 state is 1 on the LUT page, but the draw
hook recorded 0 and took the stock path. The owner-pointer read at draw time
is unreliable, so B8 reads the live LIT1 row-state byte directly:

- gate: `movbu (0xb30002ca)`; only state 1 takes the LUT tick path;
- any other state takes the original `0x702250fb` map with no LUT call;
- traces:
  - `b3001790` row 1 state seen by the hook;
  - `b3001791` path (0 stock / 1 LUT);
  - `b3001794` owner pointer seen by the hook;
  - `b3001798` draw-hook hit count.

The pool LIT1 object has stayed at `0xb3000220` (row 1 state at
`0xb30002ca`) in every live dump, and the record already uses the fixed
`0xb3001740` address.

### Package

- `work/opacity-native/slider-package-20261125T101606Z/GM1__V11.bin`
- package `16f12181...03d55`;
- program `dda8ec51...c58dae`;
- B7 `05305767...04c80` backed up on the PC.

### Test

1. Stock slider: must still work; gate sees state 4 -> stock path.
2. LUT page: set +4; the bar should now move. If not, dump:
   - `b3001790` should be 1;
   - `b3001791` should be 1;
   - `b3001794` shows whether the owner pointer was zero at draw;
   - `b3001798` shows that the draw hook ran.
3. If the bar moves, the complete stock + LUT bar + blend + decoupled
   contrast combination is achieved; the number and label remain optional.

---

## Firmware and package registry

Track every prepared build that reaches the camera. Never reuse a hash for a different build.

| Date | Build name | Purpose | Package SHA-256 | Camera result | Log entry |
|---|---|---|---|---|---|
| — | — | — | — | — | — |

---

## Test session index

| Date | Session | Camera state | Result | Capture |
|---|---|---|---|---|
| — | — | — | — | — |

---

## Corrections

If an earlier entry is wrong, add a correction here rather than editing history.

| Date | Corrected entry | Correction | Evidence |
|---|---|---|---|
| — | — | — | — |
