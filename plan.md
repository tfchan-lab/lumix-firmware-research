# Plan and Roadmap

**Project:** Lumix GM1 firmware working fork
**Target:** DMC-GM1, firmware 1.1 / internal `0110`
**Status:** Official 1.1 and stable research patch installed; current LUT build installed for testing. Opacity is the next implementation milestone.
**Last updated:** 2026-11-24

This file is the working plan. It is expected to change as camera evidence arrives. Record actual progress, defects and test results in [update-log.md](update-log.md).

---

## 1. Objective

Make the GM1 LUT firmware reliable and useful, then add a LUT distribution workflow.

In priority order:

1. **Characterize the transient colored vertical lines** after the first LUT import. They are currently a low-urgency cosmetic artifact, not a functional defect.
2. **Restore opacity** on GM1 without the severe 0.2.4 UI lag. This is the next implementation milestone.
3. **Add native sliders** later, only after the GM1 image-processing order is traced.
4. **Build a Node.js LUT store**, then a local bridge and, if the firmware allows it, Wi-Fi LUT upload.

### Non-goals for this fork

- Supporting GM5, GM1S, GX9 or other bodies.
- Removing recording limits. The upstream limit-removal patch is withdrawn after corrupted-video reports.
- Building an independent recovery loader or claiming one works.
- Shipping firmware, camera keys, private captures or calibration data.
- Replacing the native image processor with a software RGB filter.
- Unattended flashing or automatic camera recovery.

---

## 2. Current state

| Item | State |
|---|---|
| Working tree | Upstream archive at `e22ac72`, plus this fork's documentation and local working notes |
| Camera | DMC-GM1 Japanese, official 1.1/internal `0110`, English language |
| USB cable | Working stock proprietary cable connected |
| Windows USB driver | WinUSB bound to `04DA:2370`; device OK |
| HDMI capture | Mini-HDMI + USB capture card ready; GM1 HDMI is **playback-only** |
| Host | Windows x64, Python 3.12.2, Tk 8.6, MSVC 14.51 |
| Offline tests | 144 run, 1 Windows-only test-portability failure, 56 skipped |
| Official GM1 1.1 | Installed and confirmed |
| Stable research patch | Installed; 26-language catalog; ROM/RAM policy `0` |
| Current LUT build | Installed; ten cells, test LUTs working |
| Colored lines | Observed once after first import in profile menu; cosmetic, low urgency |
| Firmware changes | None in this tree yet; the installed builds are upstream/offline packages |
| Camera writes | Official 1.1, temporary policy, stable research update, LUT update |

**Current focus:** opacity. The camera is on the reviewed profile with unsigned updates enabled. The colored-line artifact is documented but does not block progress.

### Known Windows test issue

`tests/test_lumix_updater_release.py::test_camera_binary_or_unsafe_member_is_refused` fails only for the crafted name `App\other.txt`. Python's `zipfile.writestr` converts `\` to `/` on Windows, so the test cannot construct that member. The archive validator still rejects literal backslashes in real ZIPs. This is a test-portability issue, not a runtime blocker. Fix it when we next touch the test suite.

---

## 3. Test bench and responsibilities

### Host

- Windows x64 PC.
- Python 3.12.2 + Tk 8.6.
- Visual Studio 18 Community / MSVC 14.51 x64 (not on `PATH`; use `vcvars64.bat`).
- USB HDMI capture card; mini-HDMI cable.
- Panasonic proprietary USB cable pending.

### Camera

- DMC-GM1 only.
- Fully charged battery for any write.
- Camera-formatted FAT32 SD card.
- Known-good stock-restore package kept separately before any experiment.

### Human in the loop

The owner performs all physical camera actions and reports defects. The agent prepares builds, scripts, offline analyses and test plans. The owner must confirm:

- the camera is in the expected state before a write;
- the prepared package hash and filename;
- the observed screen/HDMI behavior;
- whether a defect survives a power cycle;
- when it is safe to proceed to the next step.

No firmware write happens without an explicit owner confirmation.

---

## 4. Defect and test reporting

Every camera test should produce a log entry in [update-log.md](update-log.md) with:

- date/time;
- build name and package SHA-256;
- camera model, reported firmware, internal revision;
- ROM/RAM policy state, if known;
- SD card and battery state;
- exact steps;
- expected result;
- observed result;
- HDMI capture file and frame range, or photo/video;
- whether the defect persists after off/on;
- any camera message;
- next action.

For visual defects, keep the original capture file and note the frame numbers. Do not edit the capture; crop/annotate copies instead.

---

## 5. Phase 0 — Offline readiness

**Goal:** Have every offline input, tool and test asset ready before the USB cable arrives.

### Tasks

1. **Verify the released updater**
   - Download `Lumix-Firmware-Updater-win32-x86_64.zip` from `updater-v0.2.8`.
   - Verify size `17,397,885` and SHA-256 `e2bba57f95f66363f53d759be8d4dd86ca86d981cabb477a4bff23df6b300214`.
   - Extract the whole folder; keep `_internal`.
   - Check the bundled libusb DLL against `698752cd…bf55`.
   - Run `--self-test-report` and `--gui-smoke-report`; confirm `camera_contacted: false`.
   - Validate the bundled patch catalogs/deltas against the source with `tools/lumix_updater_release_check.py`.

2. **Verify the official GM1 1.1 firmware**
   - Download `GM1__V11.zip` from the pinned Panasonic page.
   - Verify ZIP: `31,987,180` bytes, SHA-256 `4dbff5b5cd05f288d96506290f8b001ecc55969cea63a61c9fa868cc3084c24b`.
   - Extract only `GM1__V11.bin`.
   - Verify BIN: `31,983,104` bytes, SHA-256 `3e8b4aab23f79db160cd016af0e466011f047add07bb7441c642f6b70df30d90`.

3. **Build offline tools**
   - Use `vcvars64.bat`.
   - Build `apps/lumix_updater/codec_bridge.c` into `apps/lumix_updater/native/lumix-codec.dll`.
   - Build `tools/panasonic_aes5.c` into `work/panasonic-aes5.exe`.
   - Run both self-tests.
   - Install `certifi` and `libusb-package` for source runs.

4. **Decode the official firmware**
   - Decode all six GM1 components with the published key `350481b5…4622` and verify every component hash.
   - Reproduce `recipes/gm1-v11-luts.json` with `tools/lumix_lut_components.py`.
   - Verify the shipped `.lxdp` deltas reconstruct the catalog targets.

5. **Build a local offline package**
   - Use `apps.lumix_updater.firmware.build(...)` with the official image and the reviewed key.
   - Build stable, LUT and restore variants into `work/offline-prepared/`.
   - Parse each output independently and compare component hashes, marker, CRC and directory geometry.

6. **Prepare LUT test assets**
   - `IDENTITY.CUB`: neutral 17×17×17 or 33×33×33 identity table.
   - `COLOR.CUB`: strong but bounded color transform.
   - `SWAP.CUB`: channel swap or a distinctive transform that makes chroma errors obvious.
   - Keep all files small, ASCII-named and in a test card root.

7. **Set up HDMI capture**
   - Connect GM1 micro-HDMI to the USB capture card.
   - Record a stable native-style baseline at 60 fps.
   - Build a frame extraction and inspection workflow (`ffmpeg` + a local script).
   - Confirm the capture is progressive or deinterlace correctly.
   - Establish a repeatable action marker so the first frames after a switch can be located.

8. **Fix or document the Windows test failure** from section 2.

### Deliverables

- `work/` build log and hashes;
- decoded component directory with a report;
- reproduced LUT components;
- offline prepared packages;
- test LUT files;
- HDMI baseline capture and frame-extraction notes;
- a clean offline test run.

### Gate

- Every official/decoded/reproduced hash matches.
- The released app and source build both pass their offline checks.
- HDMI capture can reliably show a known on-screen change.
- No camera has been contacted or written.

---

## 6. Phase 1 — USB read-only validation

**Trigger:** Panasonic proprietary USB cable arrives.

**Goal:** Prove the camera identity, profile, key and policy without writing.

### Prerequisite: official 1.1 baseline

The first connected GM1 reports internal revision `0109`, which is outside the reviewed `0110` profile. The reviewed patch must not be forced onto it.

Before any custom package:

1. Confirm the camera's displayed firmware version.
2. Install the official Panasonic GM1 1.1 update from a verified SD card.
3. Verify the camera displays 1.1 and reconnects in the service state.
4. Re-run the read-only inspection and require internal `0110`, the reviewed key and a consistent policy state.

The official update is signed. It is separate from the unsigned research patch and does not need the service-USB policy write. It may reset language or settings and may change the service indication; record what happens.

### Tasks

1. Record Windows PnP identity:
   ```powershell
   Get-PnpDevice -PresentOnly | Where-Object {$_.InstanceId -like 'USB\VID_04DA*'} |
     Format-List Status,Class,FriendlyName,InstanceId,Problem
   ```
   Record whether the PID is `2370` (service), `2372` (PC/storage) or `2374` (PTP).
2. If `2370` is present, verify the driver. Bind WinUSB to the exact service interface with Zadig if needed. Do not touch the normal-mode device.
3. Run the released updater's self-test, then **Read camera**.
4. Record the inspection report:
   - model;
   - firmware/internal revision;
   - profile;
   - `rom_policy`;
   - `ram_policy`;
   - key recovered/guard verification.
5. Optionally capture the full 8-page inspection and key hash with a read-only script for local evidence.
6. Do not press **Enable this update**, do not write RAM, and do not insert a prepared update card.

If the camera only enumerates as `2372`/`2374`, stop. The repo has no documented GM1 service-entry procedure. Do not guess button combinations.

### Gate

- Exact `DMC-GM1` profile and internal `0110` confirmed; or a clear blocker is recorded.
- `rom_policy` known (`0` already unlocked, or `1` stock).
- No writes were sent.

### What the policy result means

- `rom_policy = 0`: the body already accepts unsigned updates; SD-only custom installs may be possible, and packages can be prepared offline with the published key.
- `rom_policy = 1`: the first custom install still needs the one-time service-USB policy write.

---

## 7. Phase 2 — Reproduce the colored lines

**Goal:** Characterize the defect before changing firmware.

### Capture limitation

The GM1 HDMI output works in **playback only**; it does not stream live preview. This changes the diagnosis:

- Record a short video immediately after Import/No LUT, then play it back through HDMI and capture the playback frames.
- If the artifact appears only in live preview and not in the recorded file, film the camera LCD with a phone, preferably at 60 fps or slow motion.
- A still-image playback capture is useful for table/format errors; video playback is needed for transition timing.
- Keep the original camera file and the HDMI recording; do not re-encode before analysis.

### Test matrix

Run each at least 10 times, from a clean boot where practical:

| Test | Purpose |
|---|---|
| Native style switch | Capture baseline; confirm no rig artifact |
| Import `IDENTITY.CUB` | Separate table content from switch timing |
| Import `COLOR.CUB` | Expose chroma/layout/mode problems |
| Import `SWAP.CUB` | Make channel-order errors obvious |
| No LUT from a loaded cell | Test the clear/restore path |
| Cell A → cell B | Test table replacement |
| Recorded video playback vs still playback | Separate table/format errors from frame timing |
| Phone-filmed LCD preview | Capture preview-only artifacts that HDMI cannot carry |
| First import after boot vs later | Detect pool/bank initialization |

### Measurements

- number of frames with lines and duration;
- line position: fixed stride, edge, full-frame, random;
- still clean or not;
- preview only, video only, or both;
- Import only, No LUT only, or both;
- behavior after a normal power cycle.

### Hypotheses to discriminate

1. In-place table publication races the image processor.
2. Style/table switch order is wrong.
3. GM1 table mode/flags differ from the GM5-derived `mode 7` / `flags 131`.
4. Cache coherency between the table writer and the ISP/DMA reader.
5. Frame-copy stride/geometry mismatch.
6. Style mask published before the table is ready.

### Offline work

- Decode GM1 and map the LUT constructor/apply path.
- Run the native GM1 style constructor in the emulator for native Standard, native Photo Style and the LUT-applied parent.
- Dump the 5,856-byte parent and compare `2700`, `2728`, `2730`, `2732`, `2736`, `4924`, `4928`.
- Compare the LUT frame-copy geometry with native preview.

### Gate

- The defect is reproducible on demand.
- A ranked root-cause hypothesis is supported by HDMI frames and/or offline parent comparison.
- We know whether stills are affected, which distinguishes table/format from frame timing.

---

## 8. Phase 3 — Fix the colored lines (deferred, low urgency)

**Goal:** Eliminate the artifact with the smallest reliable change.

### Observed result

The owner reports the colored lines are transient and appear **once after the first LUT import**, when checking the LUTs in the profile selection menu. The LUTs import and render correctly, and the artifact does not recur as a steady-state error. Treat it as a cosmetic profile-menu/preview transition issue, not a functional defect.

Do not block opacity or LUT-store work on it. If it is captured later, test one change per firmware build.

### Candidate A — Transition-safe publication

- Keep the previous table/style active until the new table is fully captured and validated.
- Copy into the active parent only at a safe point, or use a second table/parent slot and swap atomically.
- For No LUT, restore the stock table before the Standard style becomes visible.
- Gate on the native processing-idle/apply state if one can be identified.

### Candidate B — GM1-native table mode/flags

- Replace the inherited GM5 `mode 7` / `flags 131` / `+2730 = 0` values with the GM1-native values found by emulator comparison.
- Verify chroma order and bank layout.

### Candidate C — Cache/uncached placement

- Log the table address and view.
- If the writer uses cached `0x70` and the ISP reads through an uncached alias, add cache maintenance or allocate in the uncached pool.

### Candidate D — Frame-copy geometry

- Only if stills are clean but preview/video shows lines.
- Match the native preview width, height, stride and plane offsets.

### Test protocol

- One build per candidate.
- HDMI capture the full matrix from Phase 2.
- If a candidate does not fix it, record the negative result and revert rather than stacking changes.
- Keep the best known-good package.

### Gate

- 10× Import, 10× No LUT and 10× cell switch with zero colored frames.
- Preview, still, video, stop and Library/Playback all clean.
- Stock restore still works.

---

## 9. Phase 4 — Restore opacity (next)

**Goal:** Re-add the native opacity slider without the 0.2.4 lag. This is the current implementation milestone.

### Implementation constraint

The public repo has the exact 0.2.4 opacity byte recipe and review assembly, but not the private linker/build system. The old opacity build and the current stack-fixed build overlap heavily:

- 205 old-only program offsets, 61 current-only offsets, 17 changed at the same offset;
- 37 old-only GUI-resource offsets;
- dry-run byte merges produce 59–71 program conflicts.

A blind hybrid is not safe. The practical options are:

1. **Pre-baked strength variants** — 25/50/75% LUT blends loaded into different cells; no firmware change.
2. **Controlled old-build test** — install the 0.2.4 opacity target, measure lag and confirm the slider, then decide whether a cache patch is worth the effort.
3. **Full native patch** — reconstruct/extend the private build step or hand-assemble a cache-aware replacement; higher effort and risk.

Do not merge the recipes by raw byte ranges.

### Root cause to avoid repeating

The 0.2.4-era code passed the opacity tick into every preview/capture request. `job_capture` then called `lut_strength_generate` and rebuilt the table for that job. The producer runs per frame, so the table was regenerated per frame.

### Design

```text
slider tick or cell change
  → generate the strength-blended 2,284-byte table once
  → store it in one active preblend cache
preview/capture producer
  → copy the cached table
```

A single active cache is enough because only one LUT cell is selected at a time. This avoids allocating ten full tables and keeps the current per-frame copy cost.

### Tasks

1. Reuse the old native opacity UI:
   - native Contrast row;
   - ticks 0–10 → 0–100% in 10% steps;
   - numeric and scale painting;
   - parent backup/cancel.
2. Add one active preblend cache keyed by active cell, tick and generation.
3. Remove `lut_strength_generate` from the per-frame producer path.
4. Persist opacity per cell:
   - bump the bank format and use a reserved cell-header byte;
   - old cells default to 100%;
   - do not overwrite the original imported table.
5. Generate the preblended table once at startup restore and on cell/slider change.
6. Keep the other native rows hidden for this release.

### Build plan

The current build is a byte recipe; the private linker is unavailable. To produce a native slider we need:

1. **MN103 assembler/linker support**
   - build GNU binutils `as`/`ld` for `mn10300-elf`, or a minimal assembler for the instruction subset used;
   - keep the exact current-build addresses and call conventions.
2. **Patch layout**
   - find free space or reuse the existing LUT module holes/trampolines in the current build;
   - add the cache/UI routines without moving current stack-fixed code.
3. **Recipe generation**
   - start from the current LUT target components;
   - replace/add only the new routines and call sites;
   - regenerate bounded `before`/`after` edits and target hashes.
4. **Offline verification**
   - native tests for slider get/set, backup/cancel, cache invalidation, blend math, bank persistence and startup restore;
   - full container decode/signature/geometry checks.
5. **Camera test**
   - install SD-only with policy `0`;
   - verify 0/50/100%, latency, persistence and that existing imported cells survive.

### Open design choices

- **Slider granularity:** the old native row is 0–10 ticks (10% steps). Exact 25/75% requires a finer control or a 0–20/0–100 numeric table. The first implementation should use the proven 0–10 row; the pre-baked 25/75 variants remain the exact-strength reference.
- **Blend domain:** blend the camera's canonical table with the neutral table, matching `lut_strength_generate`, not the source RGB. The pre-baked samples validate the visual direction but are not the exact firmware path.
- **Cache lifetime:** one active cache with a generation/token guard; regenerate on cell or tick change; never expose a partially blended table to the producer.

### Gate

- 0/10/50/100% output is correct and monotonic.
- Opacity persists per cell across off/on.
- Cancel restores the previous value.
- Menu, mode switch and power-off latency are comparable to the no-opacity build.
- No colored lines return.

---

## 10. Phase 5 — Native sliders (deferred)

**Goal:** Expose native adjustment rows for LUT tabs only if they are safe.

### Tasks

1. Trace whether GM1 applies native parameter bytes before or after the LUT table.
2. If after: expose the supported rows and stop forcing neutral values.
3. If before: implement only table-domain transforms that make sense (contrast, saturation, color tone) and keep sharpness/NR hidden.
4. Keep opacity on the Contrast row or move it to a dedicated hidden row; do not silently duplicate a native control.
5. Extend the parent backup/cancel to cover opacity and native values together.

### Gate

- Each exposed slider has a visible, correct effect.
- LUT metadata and native preservation checks remain valid.
- No performance regression.
- Unsupported rows stay hidden rather than doing nothing.

---

## 11. Phase 6 — LUT store, stage 1 (Node.js + SD)

**Goal:** Ship a useful LUT catalog before the Wi-Fi firmware endpoint exists.

### Architecture

```text
Node.js store (HTTPS)
  → catalog, metadata, validated LUT files
  → browser UI
  → user downloads LUT to FAT32 card
  → camera imports from LUT Library
```

### Tasks

1. Node.js service with:
   - `GET /api/luts`;
   - `GET /api/luts/:id`;
   - `GET /api/luts/:id/file`;
   - static store UI.
2. Validation:
   - parse `.cube`/`.vlt`;
   - grid 2–33, finite values, monotonic domains;
   - size limits;
   - SHA-256 and content-addressed storage;
   - license/author/source metadata.
3. Store UI:
   - browse/search;
   - preview;
   - download;
   - installation instructions for FAT32/8.3 names.
4. Ops:
   - Docker image;
   - environment config;
   - SQLite/Postgres;
   - rate limits;
   - moderation/takedown fields;
   - no camera keys, serials, firmware or captures.
5. Local test:
   - run on the LAN;
   - verify a downloaded LUT imports on the camera via SD.

### Gate

- The store serves only validated LUTs.
- A downloaded LUT imports successfully through the existing SD workflow.
- No firmware changes are required.

---

## 12. Phase 7 — LUT store, stage 2 (bridge + firmware endpoint)

**Goal:** Push LUTs over Wi-Fi without the SD step, if the camera HTTP server is patchable.

### Make-or-break research

- Decode all six GM1 components.
- Search for `cam.cgi`, `accctrl`, `getinfo`, `camcmd`, `setsetting`, `HTTP/1.`, `Content-Length`, `SOAPAction`, `MediaServer`.
- Determine the owner:
  - `program`/`program2`: plausible;
  - `armcode`: significant extension and risk;
  - Wi-Fi module: no firmware endpoint; stay on the SD bridge.

### Firmware endpoint

```text
GET  /cam.cgi?mode=lut_begin&value=<size>&value2=<crc32>
GET  /cam.cgi?mode=lut_status&value=<session>
POST /cam.cgi?mode=lut_chunk&value=<session>&value2=<offset>
GET  /cam.cgi?mode=lut_commit&value=<session>
GET  /cam.cgi?mode=lut_abort&value=<session>
```

If POST is not feasible, use bounded GET chunks.

Requirements:

- `accctrl` session plus a per-upload nonce/token;
- total size, chunk size and chunk count limits;
- reuse the existing parser/converter/cell/bank path;
- keep the old bank until commit;
- abort, timeout, disconnect and power-loss safety;
- no arbitrary paths or addresses;
- progress/errors through Library status rows.

### Local bridge

- Node.js bridge on a PC/Raspberry Pi on the camera LAN.
- Serves a local UI at `http://localhost:PORT`.
- Performs `accctrl`, keeps the session alive with `getstate`.
- Downloads/validates from the store, chunks to the camera, reports progress.
- Binds to localhost; never exposes the camera to the internet.

### Memory

A 33³ CUBE can need up to ~512 KiB of source data. First network release may cap the grid or require preview idle. Streaming parsing is a later optimization.

### Gate

- 17³ VLT upload succeeds.
- 33³ CUBE upload succeeds or is explicitly capped.
- Bank commit and startup restore work.
- Abort and power loss leave the previous LUT intact.

---

## 13. Phase 8 — Community hosting

**Goal:** Run the store for others without exposing cameras or private data.

### Tasks

- Public HTTPS deployment, CDN/object storage for LUT files.
- Accounts, uploads, moderation, takedown and license handling.
- Content-addressed immutable files and signed manifests.
- Rate limits, size limits and abuse controls.
- Privacy: no camera serials, keys, captures or telemetry by default.
- Bridge installer/package for Windows/macOS/Linux.
- Documentation for AP-only vs Via Network camera setups.
- Health monitoring and backup.

### Gate

- A community user can browse, download and install via SD.
- A user with the bridge can push over Wi-Fi on a supported patched camera.
- No camera credentials or private data reach the server.
- The store can be redeployed from the repository and config.

---

## 14. Release engineering

Each firmware change needs:

- updated `recipes/gm1-v11-luts.json` with bounded, hash-checked edits;
- regenerated `.lxdp` deltas and catalog target hashes;
- preservation checks for native startup/SD regions;
- offline round-trip decode and directory verification;
- a new `GM1__V11.bin` target hash;
- a separate stock-restore file from the same starting profile;
- a new prepared folder with `SHA256SUMS`, `preparation.json` and `NEXT-STEPS.txt`;
- an [update-log.md](update-log.md) entry with build and camera results.

Do not edit the released 0.2.8 app in place. Use a new local build or a fork-specific package.

---

## 15. Risk register

| Risk | Impact | Mitigation |
|---|---|---|
| Colored-lines cause is not publication timing | Fix takes longer; may require table-format work | Diagnostic matrix and parent-object comparison before patching |
| Opacity reintroduces per-frame work | Severe lag returns | Cache preblended table; no generator in producer; measure latency |
| Native sliders alter LUT metadata | Wrong colors or unstable style | Trace ISP order; hide unsupported rows; table-domain fallback |
| `cam.cgi` is in the Wi-Fi module | Wi-Fi upload impossible via main firmware | SD store fallback; bridge can still write SD |
| Upload buffer too large | Allocation failure or instability | Cap grid, stream parse, require preview idle, fail cleanly |
| No independent recovery | Brick risk | Full battery, one change per build, stock restore, no boot-code experiments |
| Owner report is mistaken | False success | HDMI capture, repeated tests, power-cycle persistence, exact hashes |
| Windows-only test issue hides a real regression | False confidence | Fix test harness; keep cross-platform CI-style checks |

---

## 16. Open questions

1. Does the GM1 support the same `cam.cgi` API and “Via Network” client mode as GM5?
2. Where is `cam.cgi` implemented in the GM1 image?
3. Does GM1 apply native parameter bytes after the LUT table?
4. Is the opacity byte still present in the old saved-bank format, or do we need a new bank version?
5. Can a 512 KiB upload buffer coexist with preview/recording memory?
6. What is the exact GM1 service-entry procedure, if the camera is not already in the `2370` state?
7. Does the body have a prior service/language modification that already set policy `0`?

---

## 17. Decision log

| Date | Decision | Reason |
|---|---|---|
| 2026-11-23 | Focus only on DMC-GM1 1.1/0110 | Reviewed profile; avoids cross-model assumptions |
| 2026-11-23 | Fix colored lines before opacity | Same table/transition path; avoid confounding |
| 2026-11-23 | Opacity first, native sliders later | Lowest-risk path; opacity has a known working GM5 design |
| 2026-11-23 | Use cached table preblend for opacity | Removes the 0.2.4 per-frame regeneration cause |
| 2026-11-23 | Use HDMI capture for visual defects | Needed to distinguish transient publication from static format errors |
| 2026-11-23 | Node.js store in `store` + `bridge` modes | LAN tests now, community hosting later; avoids browser mixed-content issues |
| 2026-11-23 | SD store path first, Wi-Fi endpoint second | Works without firmware changes; derisks the `cam.cgi` unknown |

---

## 18. Immediate next actions

1. Rewrite and verify the fork documentation (this file, README, update log).
2. Complete Phase 0 offline setup and capture the first HDMI baseline.
3. Decode GM1 and build the parent-object comparison.
4. Prepare identity/color/swap test LUTs.
5. Once the USB cable arrives, perform the read-only profile check.
6. Start Phase 2 defect reproduction and record each run in [update-log.md](update-log.md).
