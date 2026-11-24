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
| Firmware builds | Stable and current upstream LUT builds verified and installed |
| Defect reproduction | Colored lines characterized as a one-time cosmetic profile-menu artifact; low urgency |
| Next milestone | Restore opacity without the 0.2.4 lag |
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
