# GX80/GX85 Cinelike hack + LUT webstore research

**Status:** research baseline, 2026-11-26
**Scope:** DMC-GX80/GX85 Wi-Fi CGI hack; how it can support a LUT store
**Related:** [LUT store plan](../plan.md) phases 6–8, [GM1 LUT release](gm1-lut-opacity-b9.md)

---

## 1. Bottom line

The GX80/GX85 hack is an **undocumented HTTP control API** trick. It does not
modify firmware and it does **not** add in-camera LUT import. It asks the
camera to select hidden Photo Style values (`cinelike_d`, `cinelike_v`) that
the normal menu does not expose.

That means a LUT webstore cannot “push a LUT into a GX80” through this hack.
The realistic product is:

1. a **profile-aware LUT catalog** (Cinelike V, Cinelike D, Standard, etc.);
2. a **local camera bridge** that can set/restore the hidden profile over
   Wi-Fi;
3. a clear workflow: configure the camera, shoot, then apply the purchased
   LUT in post;
4. for GM1 (patched LUT import), keep the existing SD import flow.

A public HTTPS store page cannot safely call `http://192.168.54.1/cam.cgi`
directly. The camera has no CORS headers, and an HTTPS page cannot mix in
plain HTTP requests. The bridge is therefore not optional for a reliable
store experience.

---

## 2. What the hack actually does

The original BTM_Pix method (May 2017):

1. Turn on the camera Wi-Fi in “Remote Shooting & View” mode.
2. Connect the phone/laptop to the camera SSID.
3. Open a local HTML file with four links.
4. `HANDSHAKE` → camera approves the client.
5. `CONNECT` → camera enters remote-control mode (“Under Remote Control”).
6. `DEPLOY` → camera selects the hidden Cinelike D style.
7. `RESTORE` → camera returns to the Standard style.

The original `Deploy Cinelike D.html` is only 925 bytes. Its links are:

```text
http://192.168.54.1/cam.cgi?mode=accctrl&type=req_acc&value=4D454930-0100-1000-8001-020A000270B8&value2=D2304
http://192.168.54.1/cam.cgi?mode=camcmd&value=recmode
http://192.168.54.1/cam.cgi?mode=setsetting&type=colormode&value=cinelike_d
http://192.168.54.1/cam.cgi?mode=setsetting&type=colormode&value=standard
```

The community HTML client also lists `cinelike_v` as a valid `colormode`
value, so the same mechanism should enable it on models that accept it:

```text
/cam.cgi?mode=setsetting&type=colormode&value=cinelike_v
```

### Observable behavior

- The camera replies with a plain/XML `ok` result when the command succeeds.
- The Photo Style indicator on the camera disappears/blanks while the hidden
  style is active.
- The camera menu’s Photo Style page becomes blank/unselectable until a
  restore is sent.
- The hidden style persists across power cycles.
- Saving the configured state to C1/C2/C3 gives a practical way to switch
  between “Cinelike” and normal styles without repeating the browser steps.
- On the GX80, the body does not remember every live setting after sleep;
  C1/C2/C3 recall is the workaround.

### Model/firmware caveat

The original test was a GX80 on firmware 1.0. GX85, later GX80 firmware and
other Panasonic bodies may accept different values or require different
pairing steps. Treat this as a **test matrix**, not a universal API.

---

## 3. Protocol details

| Step | Request | Notes |
|---|---|---|
| Pair | `GET /cam.cgi?mode=accctrl&type=req_acc&value=<client-id>&value2=<client-name>` | Camera may prompt/approve. Response is typically `ok,<model>-<id>,remote,encrypted`. |
| Remote mode | `GET /cam.cgi?mode=camcmd&value=recmode` | Starts the remote-control session. |
| Capability | `GET /cam.cgi?mode=getinfo&type=capability` | May reveal supported `colormode` values; do not assume it lists hidden ones. |
| Current state | `GET /cam.cgi?mode=getstate` | Also usable as a keep-alive. |
| Current style | `GET /cam.cgi?mode=getsetting&type=colormode` | Useful for verification/restore state. |
| Set style | `GET /cam.cgi?mode=setsetting&type=colormode&value=cinelike_d` | Hidden style; `cinelike_v` is the other candidate. |
| Restore | `GET /cam.cgi?mode=setsetting&type=colormode&value=standard` | Restores normal Photo Style selection. |
| UPnP helper | `GET http://192.168.54.1:60606/<camera-id>/Server0/ddd` | Some models need this to finish pairing. The `<camera-id>` is model/instance-specific. |

The client ID is not settled. The published examples differ per phone/client:

- `4D454930-0100-1000-8001-020A000270B8` / `D2304`
- `4D454930-0100-1000-8001-020A0005A9CF` / `DC9602 Reader`
- `4D454930-0100-1000-8001-02FA000430C6` / `MI 5`
- `4D454930-0100-1000-8000-F02765BACACE` / `DMC-CM1`

The `4D454930` prefix is ASCII `MEI0`, so this is likely a client-app identity
format rather than a camera serial. It is unknown whether the camera accepts
an arbitrary UUID or expects an approved client. **This is the first protocol
experiment.**

The camera’s HTTP server may return plain text for `accctrl` and XML for
other commands. A robust client must parse both and must check the
application-level `result`, not just HTTP 200.

---

## 4. Browser and network constraints

| Approach | Works? | Problem |
|---|---|---|
| Public HTTPS store calls `http://192.168.54.1/cam.cgi` with `fetch` | No | Mixed content + CORS; camera does not send `Access-Control-Allow-Origin`. |
| Public HTTPS store navigates/opens the CGI URL | Partly | Top-level navigation can send the GET, but the store cannot read the response or confirm success. |
| Local HTML file with `<a href>` links | Yes | Original hack; fire-and-forget, no response parsing. |
| Local bridge on `http://localhost:PORT` | Yes | Best option: server-side HTTP client, response parsing, state machine, no CORS. |
| Native mobile app | Yes | Best UX, but more packaging/distribution work. |
| PWA served from camera AP | No | Camera does not host custom pages. |

The camera AP normally has no internet. The user must browse/download LUTs
**before** joining the camera Wi-Fi, or the bridge must cache the catalog and
LUT files locally. The store UI should be offline-capable while connected to
the camera.

Security requirements for the bridge:

- bind to `127.0.0.1` only;
- never expose a generic “proxy any URL” endpoint;
- require an explicit user click for pair/deploy/restore;
- keep camera identity, serials and captures local;
- enforce timeouts and a restore-on-exit path;
- never run the hack automatically from a public page.

---

## 5. Recommended architecture

```text
Public HTTPS store
  ├─ LUT catalog + previews + licenses
  ├─ LUT files (hash-addressed)
  └─ profile compatibility metadata
            │
            ▼
Local bridge (Node.js / Electron / Tauri)
  ├─ serves UI at http://localhost:PORT
  ├─ downloads/caches LUTs from the store
  ├─ camera client → http://192.168.54.1/cam.cgi
  ├─ GX80/GX85: pair → recmode → set colormode → restore
  ├─ GM1: write LUT files to SD / future firmware endpoint
  └─ state machine + logs + restore
```

This matches the existing plan’s “store + bridge” decision and avoids the
browser mixed-content problem.

### Two product tracks

**Track A — GX80/GX85 “Cinelike kit”**

- store tag: “Works with Cinelike V” / “Works with Cinelike D”;
- bridge button: “Enable Cinelike V” / “Restore Standard”;
- LUT bundle includes a shooting recipe (profile, contrast, saturation,
  white balance, exposure);
- LUTs are for Resolve/Premiere, not in-camera import.

**Track B — GM1 LUT import**

- store tag: “GM1 LUT import”;
- download validated `17³`/`33³` `.CUB` files;
- write to FAT32 SD with 8.3-safe names;
- camera imports through LUT Library;
- future: patched firmware endpoint for Wi-Fi upload.

The store should not blur these tracks. A GX80 user buying a “LUT” expects a
file, not a hidden camera profile; a GM1 user expects an importable file.

---

## 6. MVP scope

### Store

- browse/search by look, camera/profile compatibility, author, license;
- LUT detail page with before/after preview and hash;
- download validated `.CUB`/`.VLT`;
- “Camera recipe” panel for GX80/GX85 Cinelike V/D;
- no accounts/payments in the first local prototype; use local fixtures.

### Bridge

- `GET /api/camera/status` → disconnected / reachable / paired / remote /
  style set;
- `POST /api/camera/pair` → `accctrl`;
- `POST /api/camera/remote` → `recmode`;
- `POST /api/camera/style` with `{value: "cinelike_v"}`;
- `POST /api/camera/restore`;
- `GET /api/camera/capability` (read-only);
- explicit “Restore Standard” on exit/error;
- structured logs with request/response summaries, no secrets.

### Explicitly deferred

- in-camera LUT upload over Wi-Fi (needs a patched firmware endpoint);
- V-Log unlock (not established on GX80/GX85);
- monetizing the hidden profile itself;
- auto-pairing or background camera control.

---

## 7. Test plan

### Protocol

1. Connect a GX80/GX85 to the camera AP; run a local script against
   `192.168.54.1`.
2. Try the published client ID first; then a generated UUID with the `MEI0`
   prefix; record whether approval is required.
3. Test with and without the `:60606` UPnP request.
4. Run `getinfo&type=capability` and `getsetting&type=colormode` before/after.
5. Test `cinelike_d`, `cinelike_v`, and `standard`.
6. Record exact response bodies, HTTP headers, timing and any camera prompt.
7. Power-cycle and test C1/C2/C3 persistence.
8. Test restore after a failed/disconnected session.

### Browser/bridge

1. Confirm a public HTTPS page cannot read `cam.cgi` (CORS/mixed content).
2. Confirm `window.open` can send the command but cannot read the result.
3. Confirm a localhost bridge can read and parse the result.
4. Test offline store browsing while connected to the camera AP.
5. Test bridge bind address, CSRF token, and restore-on-exit.

### Safety

- one command per user action;
- no raw URL proxy;
- timeouts and retry limits;
- clear warning that Photo Styles may be blank until restore;
- no camera writes while recording/updating firmware.

---

## 8. Open questions

1. Does the camera accept an arbitrary client UUID, or must it be an approved
   Image App identity?
2. Is the `:60606` UPnP request required on GX80/GX85, or only on some models?
3. Does `cinelike_v` work on GX80/GX85, and on which firmware versions?
4. Does `capability` list hidden styles, or only the normal menu?
5. Does `colormode` survive a power cycle on its own, or only through C1/C2?
6. Can a laptop stay on the camera AP while serving a localhost UI? (Yes in
   principle; test DHCP/routes.)
7. What is the legal/ToS position of automating a hidden Panasonic feature?
8. Should the store ever sell a “Cinelike V unlock,” or only LUTs and recipes?
   Recommendation: **do not sell the unlock**; provide it as a free,
   reversible helper and sell the LUT/recipe content.

---

## 9. Sources

- EOSHD, “Smartphone hack enables extra features on Panasonic cameras”,
  3 June 2017:
  <https://www.eoshd.com/news/smartphone-hack-enables-extra-features-panasonic-cameras/>
- EOSHD forum, “Would You Perhaps Be Interested In A Different GX80/85
  Colour Profile???”, BTM_Pix, May 2017:
  <https://www.eoshd.com/comments/topic/24995-would-you-perhaps-be-interested-in-a-different-gx8085-colour-profile/>
- Daejeon Chronicles, “Turn Your GX80 Into a Monster!”, 27 May 2017:
  <https://daejeonchronicles.com/2017/05/27/turn-your-gx80-into-a-monster/>
- `auberginehill/panasonic-html-image-app`, `requests.html` and README:
  <https://github.com/auberginehill/panasonic-html-image-app>
- `cleverfox/lumixproto`, GX80 Wi-Fi protocol notes:
  <https://github.com/cleverfox/lumixproto>
- Existing store plan: [plan.md](../plan.md) phases 6–8.
