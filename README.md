# Lumix GM1 Firmware — Working Fork

**A practical fork of the upstream Lumix firmware research archive, focused on the DMC-GM1 LUT patch and the tooling around it.**

> **Status: planning and offline preparation.** No camera writes or firmware changes have been made yet. The Panasonic proprietary USB cable is still pending; the mini-HDMI cable and USB capture card are ready.

This fork starts from the upstream archive at commit `e22ac72` (the `updater-v0.2.8` documentation state). The upstream project is an unmaintained reference archive. This fork is our own working copy for experiments, fixes and new tooling. It does not upstream pull requests.

- [Plan and roadmap](plan.md)
- [Update log](update-log.md)
- [Upstream archive policy](CONTRIBUTING.md)

---

## Why this fork exists

The upstream archive documents a large amount of successful research, but it is organized as a chronological evidence archive. This fork adds a clear working entry point for one camera and one goal:

1. **Characterize the transient colored vertical lines** that appear after the first LUT import. They are currently considered a low-urgency cosmetic artifact, not a functional defect.
2. **Restore opacity** on GM1, using a cached table-domain blend so the old severe UI lag does not return. This is the next implementation milestone.
3. **Add native sliders later**, only after the GM1 image-processing order is traced.
4. **Build a LUT store**, with a Node.js web server and a local bridge that can push LUTs to the camera over Wi-Fi when the firmware supports it.

The detailed work breakdown, gates and risks are in [plan.md](plan.md). Daily notes and test results go in [update-log.md](update-log.md).

---

## Current priorities

| Priority | Work | Status | Exit gate |
|---|---|---|---|
| **1** | GM1 colored vertical lines after first LUT import | Characterized: one-time cosmetic profile-menu artifact; low urgency | Optional: capture LCD/video evidence; no functional blocker |
| **2** | Restore GM1 opacity without the 0.2.4 lag | **Next implementation milestone** | 0/50/100% correct, persists per cell, no menu/power-off lag, no colored lines |
| **3** | Native sliders for LUT tabs | Deferred until the GM1 ISP order is traced | Sliders affect the intended output and do not corrupt LUT metadata |
| **4** | Node.js LUT store | Offline design ready | Store catalog + validated downloads; later local bridge and firmware endpoint |
| **5** | GM1 Wi-Fi LUT upload | Research pending | `cam.cgi` owner identified; safe chunked upload; bank commit/restore |

The current GM1 LUT patch applies LUTs at **100%** and hides the native adjustment rows. That is the baseline we are improving.

---

## Test bench and working method

**Host**

- Windows x64 PC
- Python 3.12.2 + Tk 8.6
- Visual Studio 18 Community with MSVC 14.51 x64
- Mini-HDMI cable and USB HDMI capture card ready
- Panasonic proprietary USB cable **pending**

**Camera**

- DMC-GM1, firmware 1.1/internal `0110` expected
- Region, current policy state and installed patch are not yet read
- GM1S is **not** supported

**Human in the loop**

The camera owner runs all physical actions and reports defects. The agent prepares builds, scripts, offline checks and test plans. There is no unattended flashing and no automatic camera recovery.

When a defect is observed, record:

- the exact prepared package name and SHA-256;
- camera model and internal revision;
- ROM/RAM policy state, if known;
- exact steps before the defect;
- expected vs observed behavior;
- HDMI capture file and frame numbers, or a photo/video;
- whether the defect survives a normal off/on;
- battery, SD card and lens state.

A report template is kept in [update-log.md](update-log.md).

---

## Safety first

Flashing Panasonic cameras is risky. This archive has **no validated independent recovery procedure** for GM1 or GM5. A bad update can leave the camera unusable.

Rules for this fork:

- Read the camera first; make no writes until the read-only profile is reviewed.
- Change one variable per firmware build.
- Use a fully charged battery and stable power.
- Keep the known-good prepared folder and its separate `stock-restore` copy.
- Reset camera settings while patched firmware is still running before installing stock restore.
- Do not interrupt an update, and do not press other buttons during programming.
- Treat a progress bar or a restart as insufficient proof; verify the installed behavior afterward.
- Do not attempt the factory `GM1.AHX` NOR-writer path as a recovery experiment.

The upstream recording-limit removal is withdrawn after reports of corrupted video beyond 30 minutes. This fork preserves native recording limits.

---

## How the GM1 patch works, in one page

The patch is not a single “jailbreak.” It is two separate mechanisms.

### 1. Update trust bypass

| Layer | What happens |
|---|---|
| Official update container | `UPD` v2: header, directory, encrypted components, outer CRC32 |
| Cipher | Five-round AES-128-CBC, not standard ten-round AES; per-component IV |
| Integrity | SHA-256 per decoded component |
| Authentication | ECDSA P-256 signature over the directory |
| Camera key | GM1 key at flash `0x405AB460` (also published in the research docs) |
| Signing key | Not present in the camera and not recovered; the AES key cannot sign |
| Unsigned branch | A zero first marker byte can skip signature/component hashes **only when** runtime policy byte `0x70903814` is zero |
| Stock policy | ROM initializer `0x405C5B40` supplies `1`, so unsigned updates are rejected |
| Patched policy | The patch writes `0` into the ROM initializer, so later boots allow unsigned updates |
| First install | Still needs the temporary RAM policy set to zero through service USB; the package cannot authorize its own first install |
| Later installs | SD-only once the patched firmware is running with policy `0` |

### 2. Feature patch

The GM1 LUT, language and stock-restore packages prepared by this updater write only three components:

- `program` (main MN103 code, language patch, LUT code, policy byte);
- `guires` (GUI resources);
- `program2` (secondary program).

The other official components — settings, body calibration, `armcode`, `other` and `eep` — are excluded from these packages. The LUT `program` prefix stops at `0x880000`; the two saved-LUT sectors after it are preserved.

The language patch redirects the existing regional language lists to the firmware’s complete 26-language catalog. The LUT patch adds ten saved cells, a Setup → LUT Library menu, native Photo Style/Quick Menu selection, touch rows, SD import, the 9×9×9 conversion path and internal NOR persistence.

For the full technical record, see:

- [GM1 LUT patch and limitations](docs/gm1-luts.md)
- [GM1 USB readback](docs/gm1-usb-service.md)
- [GM1 program decoding and signatures](docs/gm1-program-analysis.md)
- [GM1 reproduction guide](docs/gm1-reproduction.md)
- [Complete language catalog](docs/lumix-all-languages.md)

---

## How LUT import works today

On the patched camera:

```text
Setup → LUT Library → select a cell → Load
      → picker lists compatible files in the SD card root
      → select file → Standard confirmation
```

- Use a camera-formatted **FAT32** card.
- Use short ASCII **8.3 filenames** in the card root, e.g. `SWAP.VLT`, `CYAN.CUB`.
- Supported formats: `.VLT` (17×17×17, 16-bit RGB) and `.CUBE` (2–33 points per axis, float/int RGB).
- Only the **Standard** base picture style is supported.
- Metadata must be neutral: grain off/L0, color noise off, sharpness/noise reduction 0/L50.
- The file is resampled to the camera’s **9×9×9 YCbCr table with 6-bit values**. This is a hardware table, not a software RGB pixel filter.
- Ten cells are saved in two internal NOR banks. Startup restores the saved banks; it does **not** scan the SD card at boot or on card insertion.
- GM1 currently applies LUTs at fixed 100% strength and has no adjustment rows. GM5 retains an opacity control.
- The exFAT importer is withdrawn. There is no separate EIR style or 65:24 feature in this package.

Current known GM1 limitations:

- brief colored vertical lines after Import/No LUT (priority 1);
- slightly slower than native Photo Styles;
- complete recording-format and long-session coverage not established;
- no independent recovery.

---

## Repository map

| Path | What it contains |
|---|---|
| [`apps/lumix_updater`](apps/lumix_updater) | Desktop updater source; patch catalogs, deltas, USB transport, codec bridge |
| [`apps/README.md`](apps/README.md) | Updater usage, build steps, release history |
| [`docs/`](docs) | Model guides, protocol findings, flashing and reproduction notes |
| [`evidence/`](evidence) | Summarized receipts and hashes; not raw firmware or captures |
| [`recipes/`](recipes) | Exact guarded LUT/language patch recipes |
| [`research/gm1-luts`](research/gm1-luts) | Selected assembly review sources for the GM1 LUT subsystems |
| [`research/gm5-luts`](research/gm5-luts) | GM5 LUT sources, including the working opacity/slider design |
| [`tools/`](tools) | Decoders, capture verifiers, package builders, emulators |
| [`tests/`](tests) | Offline tests; firmware/emulator tests skip when private inputs are absent |
| `work/`, `captures/`, `firmware/` | Local-only inputs and outputs; git-ignored |

Start with:

- [Plan and roadmap](plan.md) for what we are doing next;
- [Update log](update-log.md) for what has actually been tested;
- [GM1 LUT guide](docs/gm1-luts.md) for the current firmware behavior;
- [Updater app guide](apps/README.md) for preparing an SD update;
- [Reproducibility notes](docs/reproducibility.md) for evidence boundaries.

---

## Offline quick start (Windows)

These commands do not contact a camera.

```powershell
# Offline test suite. Firmware/emulator tests skip without private inputs.
python -B -m unittest discover -s tests -v

# Python dependencies for the updater from source.
python -m pip install -r apps\requirements-updater.txt
```

The updater from source also needs a native codec DLL built with the Visual Studio x64 tools. The exact commands, official firmware hashes and offline build checks are in [plan.md](plan.md).

The upstream `updater-v0.2.8` Windows app is the fastest path for a physical test. It is not built from this working tree and should be hash-checked before use. Release links and checksums are in [apps/README.md](apps/README.md). This fork has not published its own binaries yet.

---

## What is established, reported, and still open

| Claim | Status |
|---|---|
| GM1 service USB identity `04DA:2370`, `ff/a0/b0` | Measured |
| Complete GM1 main (9 MiB) and secondary (3.375 MiB) readback | Measured; hashes match official decoded components |
| GM1 five-round AES-128-CBC, key, component SHA-256, ECDSA directory signature | Independently verified offline |
| GM1 LUT patch installation and normal camera use | Owner-reported |
| GM1 colored lines after Import/No LUT | Owner-reported, unresolved |
| GM1 opacity with acceptable performance | 0.2.4 had severe lag; removed in 0.2.5 |
| Native adjustment rows for GM1 LUT tabs | Not implemented; ISP order not yet traced |
| GM1 `cam.cgi` owner and Wi-Fi upload feasibility | Not yet investigated |
| Independent GM1 recovery | Not validated |

Do not present owner reports as independent hardware verification. The evidence records distinguish static analysis, bounded emulation, physical readback and owner observation.

---

## Upstream warnings and archive policy

The following upstream warnings still apply:

- **All original project code was written by a chatbot.** Flashing Panasonic cameras is risky because recovery options are limited.
- **No independent GM1/GM5 recovery procedure is validated here.** A bad update can leave a camera unusable.
- **Recording-limit removal is withdrawn.** The owner reported corrupt SD video files after recording beyond 30 minutes. Newly prepared firmware preserves native recording limits.
- **GM1S is unsupported** and is rejected by the model guards.
- **Upstream accepts no contributions, pull requests, bug reports or support requests.** This is a personal fork; use your own copy for experiments.

Upstream release context and detailed limitations remain in the original docs, especially [apps/README.md](apps/README.md), [docs/gm1-luts.md](docs/gm1-luts.md) and [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Documentation index

**This fork**

- [Plan and roadmap](plan.md)
- [Update log](update-log.md)

**GM1**

- [LUT patch, limitations and reproduction](docs/gm1-luts.md)
- [USB service observation and bounded probe](docs/gm1-usb-service.md)
- [Program decoding, cipher and signature](docs/gm1-program-analysis.md)
- [Reproduction guide](docs/gm1-reproduction.md)

**Updater and releases**

- [Desktop updater guide](apps/README.md)
- [Language catalog implementation](docs/lumix-all-languages.md)
- [Reproducibility and evidence boundaries](docs/reproducibility.md)

**Protocol and other models**

- [Porting the investigation to another model](docs/porting-to-other-models.md)
- [GX9 service USB](docs/gx9-usb-service.md)
- [GM5 flashing](docs/gm5-flashing.md)
- [Factory programming and AHX paths](docs/factory-programming.md)

---

## Contributing to this fork

This is not an upstream-supported project. The working rule is simple:

1. Change one thing at a time.
2. Record the build hash, camera state and observed result in [update-log.md](update-log.md).
3. Keep private firmware, keys, captures and calibration data out of Git.
4. Do not weaken a hash guard, range check or native-preservation check to make an experiment pass.
5. Do not claim installation, recovery or performance from a progress bar or a single successful boot.

If a change cannot be verified, label it as unverified rather than promoting it to a feature.
