# Spiral Warrior English Offline Local Revival Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.

**Goal:** Produce an English-language, offline/local playable package for Spiral Warrior using the international Android client as the player-facing build, supported by local tooling, local backend emulation, asset extraction, translation patches, and emulator packaging.

**Architecture:** This is not a source-code recompilation or native PC port at first. The practical architecture is an Android client running on PC via emulator/Waydroid/BlueStacks-compatible tooling, redirected to a local backend on `127.0.0.1` or a LAN hostname. The local backend will emulate the minimum game/server APIs required for login, server selection, player profile, progression, resources, and saves; English text will be supplied through the international client, backend responses, and targeted asset/script patches.

**Tech Stack:** Android APK/XAPK, Cocos Creator/cocos2d-js, `.jsc` reverse engineering/instrumentation, Python/FastAPI or Node.js/Express local backend, SQLite/JSON local saves, mitmproxy/HTTP Toolkit, adb/logcat, apktool/jadx, Frida where needed, emulator packaging.

---

## 0. Legal and Scope Notes

This plan is for preservation/research and a local offline proof-of-concept. Before distributing patched clients or assets to third parties, confirm rights/permissions. Avoid reusing live publisher services or monetization/payment endpoints. Strip or disable payment, tracking, ads, and third-party SDK calls where possible.

---

## 1. Current Context and Known Artifacts

Workspace:

```text
/path/to/spiral-warrior
```

Downloaded/verified artifacts:

```text
international_latest.xapk
cn_9game.apk
intl_xapk/com.oversea.spinarena.apk
intl_xapk/NewAssets.apk
intl_base/
intl_assets/
cn_apk/
strings/*.urls.txt
triage_report.md
docs/ARTIFACT_INVENTORY.md
CHECKSUMS.sha256
```

Known international package:

```text
com.oversea.spinarena
Version: 1.1.0.96
Version code: 153
```

Known Chinese/public package source:

```text
cn_9game.apk
likely package: com.dianhun.lxys.aligames
```

Engine:

```text
Cocos Creator / cocos2d-js
```

Important recovered endpoints:

```text
https://sdk-login-cocooversea.17m3.com/DHSDK/Action_UserLogin.aspx
https://sdk-package-update-config.17m3.com
https://ups-config.17m3.com
https://sdk-heartbeat.17m3.com/ttl
https://spinarena.firebaseio.com
http://cdnsgp-x3jl-release.17m3.com/oversea/remote-assets/project.manifest
http://cdnsgp-x3jl-release.17m3.com/oversea/remote-assets/version.manifest
http://lxys-area.17m3.com/login/1/?account=
http://lxys-audit-area.17m3.com:25801/auth/es?account=
http://lxys-test-area.17m3.com:15801/auth/es?account=
http://lxys1-client.17m3.com/x3-hd/remote-assets/project.manifest
http://lxys1-client.17m3.com/x3-hd/remote-assets/version.manifest
```

---

## 2. Target Deliverables

### 2.1 Milestone A — Research Lab

A reproducible project setup containing:

```text
tools/
  extract_apks.sh
  install_intl_xapk.sh
  capture_logcat.sh
  run_proxy.sh
  scan_endpoints.py
research/
  endpoint_inventory.md
  launch_logcat/*.log
  network_captures/*.mitm
```

Validation:

- APK/XAPK can be installed on a known emulator.
- Launch produces captured logcat.
- First network calls are identified.

### 2.2 Milestone B — Client Boot and English Baseline

A working emulator launch of the international client, with proof of:

- app starts without immediate crash;
- language state is English or forced to English;
- startup blockers identified;
- dead endpoints redirected or mocked.

### 2.3 Milestone C — Local Backend MVP

A local backend that can respond to the first required client calls:

```text
GET/POST /DHSDK/Action_UserLogin.aspx
GET /login/1/?account=...
GET /auth/es?account=...
GET /remote-assets/version.manifest
GET /remote-assets/project.manifest
```

Validation:

- client reaches login/server selection or next screen using local server responses.

### 2.4 Milestone D — Lobby Entry

The client can enter the lobby/main menu with a fake local account and English strings.

### 2.5 Milestone E — Offline Single-Player Loop

The local package supports a minimal playable loop:

- local profile;
- initial inventory/team;
- stage selection;
- battle entry;
- battle result/reward;
- save/load.

### 2.6 Milestone F — English Distribution Package

A user-friendly PC package:

```text
SpiralWarrior-English-Offline/
  README_START_HERE.md
  launcher.sh / launcher.bat
  emulator_setup/
  client/
  server/
  saves/
  docs/
```

---

## 3. Folder Layout to Maintain

Use this layout:

```text
/path/to/spiral-warrior/
  README.md
  CHECKSUMS.sha256
  international_latest.xapk
  cn_9game.apk
  intl_xapk/
  intl_base/
  intl_assets/
  cn_apk/
  docs/
    ARTIFACT_INVENTORY.md
    PROJECT_PLAN_ENGLISH.md
    LEGAL_SCOPE.md
    REVERSE_ENGINEERING_NOTES.md
    API_SPEC_DRAFT.md
    ENGLISH_LOCALIZATION_NOTES.md
  research/
    endpoint_inventory.md
    launch_logcat/
    network_captures/
    screenshots/
  tools/
    extract_apks.sh
    install_intl_xapk.sh
    install_cn_apk.sh
    capture_logcat.sh
    scan_endpoints.py
    patch_hosts.py
  server/
    pyproject.toml
    README.md
    app/
      main.py
      routes/
      models/
      data/
      saves/
    tests/
  patches/
    README.md
    apktool/
    strings/
    certs/
  emulator/
    README.md
    device_profiles.md
  dist/
```

---

## 4. Development Cycle Overview

Each implementation iteration must follow this cycle:

1. Pick one blocker/screen transition.
2. Capture the exact failing request/log/error.
3. Add or patch one response/path/config.
4. Re-run the client.
5. Record before/after evidence in `research/`.
6. Commit changes with a clear message.
7. Update `docs/API_SPEC_DRAFT.md` and `docs/REVERSE_ENGINEERING_NOTES.md`.

Do not build large speculative systems. Implement only what the client actually requests.

---

## 5. Phase 1 — Environment and Artifact Baseline

### Task 1: Create top-level README

**Objective:** Make the workspace self-explanatory.

**Files:**

- Create/modify: `README.md`

**Content outline:**

```markdown
# Spiral Warrior English Offline Revival

This local working folder contains the operator's own Android client files and tooling for building an English local/offline preservation prototype. The client files are never part of the public repository.

## Key Artifacts
- `international_latest.xapk` — international client, player-facing base.
- `cn_9game.apk` — Chinese build, richer reference data.
- `intl_base/`, `intl_assets/`, `cn_apk/` — extracted assets.

## Main Plan
See `docs/PROJECT_PLAN_ENGLISH.md` and `.hermes/plans/2026-07-22_010544-english-offline-local-revival.md`.
```

**Verification:**

Run:

```bash
readlink -f README.md && test -s README.md
```

Expected: file path printed, exit code 0.

### Task 2: Verify artifact checksums

**Objective:** Ensure original archives are preserved.

**Files:**

- Read: `CHECKSUMS.sha256`

**Command:**

```bash
sha256sum -c CHECKSUMS.sha256
```

**Expected:**

All listed files report `OK`.

### Task 3: Add extraction script

**Objective:** Make APK/XAPK extraction reproducible.

**Files:**

- Create: `tools/extract_apks.sh`

**Implementation:**

```bash
#!/usr/bin/env bash
set -euo pipefail
ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
cd "$ROOT"
rm -rf intl_xapk intl_base intl_assets cn_apk
mkdir -p intl_xapk intl_base intl_assets cn_apk
unzip -q international_latest.xapk -d intl_xapk
unzip -q intl_xapk/com.oversea.spinarena.apk -d intl_base
unzip -q intl_xapk/NewAssets.apk -d intl_assets
unzip -q cn_9game.apk -d cn_apk
echo "Extraction complete under $ROOT"
```

**Verification:**

```bash
chmod +x tools/extract_apks.sh
./tools/extract_apks.sh
test -f intl_base/AndroidManifest.xml
test -f intl_assets/AndroidManifest.xml
test -f cn_apk/AndroidManifest.xml
```

---

## 6. Phase 2 — Emulator Launch Lab

### Task 4: Document emulator target

**Objective:** Pick a reproducible test environment.

**Files:**

- Create: `emulator/device_profiles.md`

**Recommended baseline:**

```text
Android version: 9, 10, or 11 first
ABI: arm64-v8a if available, otherwise x86_64 with ARM translation
RAM: 4 GB+
Storage: 8 GB+
Network: proxy configurable
Root: optional but useful
```

Use one of:

- Android Studio Emulator with ARM translation;
- Genymotion;
- MuMu/LDPlayer/BlueStacks on Windows if needed;
- Waydroid on Linux if GPU/ARM compatibility works.

### Task 5: Create international install script

**Objective:** Install split XAPK correctly.

**Files:**

- Create: `tools/install_intl_xapk.sh`

**Implementation:**

```bash
#!/usr/bin/env bash
set -euo pipefail
ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
adb devices
adb install-multiple -r \
  "$ROOT/intl_xapk/com.oversea.spinarena.apk" \
  "$ROOT/intl_xapk/NewAssets.apk"
```

**Verification:**

```bash
chmod +x tools/install_intl_xapk.sh
adb devices -l
./tools/install_intl_xapk.sh
adb shell pm list packages | grep com.oversea.spinarena
```

### Task 6: Create logcat capture script

**Objective:** Capture launch failures and network hints.

**Files:**

- Create: `tools/capture_logcat.sh`

**Implementation:**

```bash
#!/usr/bin/env bash
set -euo pipefail
ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
mkdir -p "$ROOT/research/launch_logcat"
OUT="$ROOT/research/launch_logcat/$(date +%Y%m%d_%H%M%S)-spiralwarrior.log"
adb logcat -c
adb shell monkey -p com.oversea.spinarena 1 || true
adb logcat -v time | tee "$OUT"
```

**Usage:**

Run the script, wait until crash/stall, then stop with `Ctrl-C`.

**Verification:**

```bash
test -s research/launch_logcat/*.log
```

---

## 7. Phase 3 — Network and Endpoint Discovery

### Task 7: Build endpoint inventory from strings

**Objective:** Create a clean endpoint list from extracted files.

**Files:**

- Create: `tools/scan_endpoints.py`
- Create/update: `research/endpoint_inventory.md`

**Implementation:**

```python
#!/usr/bin/env python3
import os, re
ROOT = os.path.abspath(os.path.join(os.path.dirname(__file__), '..'))
TARGETS = ['intl_base', 'intl_assets', 'cn_apk']
URL_RE = re.compile(rb'https?://[A-Za-z0-9_./:?=&%#@+\-]+')
noise = [b'adobe.com', b'w3.org', b'android.com', b'github.com', b'apache.org']
found = {}
for target in TARGETS:
    base = os.path.join(ROOT, target)
    for dirpath, _, files in os.walk(base):
        for fn in files:
            path = os.path.join(dirpath, fn)
            try:
                data = open(path, 'rb').read()
            except Exception:
                continue
            for m in URL_RE.findall(data):
                if any(n in m for n in noise):
                    continue
                url = m.decode('utf-8', 'ignore')
                found.setdefault(url, set()).add(os.path.relpath(path, ROOT))

out = os.path.join(ROOT, 'research', 'endpoint_inventory.md')
os.makedirs(os.path.dirname(out), exist_ok=True)
with open(out, 'w', encoding='utf-8') as f:
    f.write('# Endpoint Inventory\n\n')
    for url in sorted(found):
        f.write(f'## `{url}`\n\n')
        for src in sorted(found[url]):
            f.write(f'- `{src}`\n')
        f.write('\n')
print(out)
```

**Verification:**

```bash
chmod +x tools/scan_endpoints.py
./tools/scan_endpoints.py
test -s research/endpoint_inventory.md
grep -E 'spinarena|lxys|17m3' research/endpoint_inventory.md
```

### Task 8: Configure proxy capture

**Objective:** Capture real runtime requests.

**Files:**

- Create: `tools/run_proxy.sh`
- Update: `research/network_captures/README.md`

**Implementation:**

```bash
#!/usr/bin/env bash
set -euo pipefail
ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
mkdir -p "$ROOT/research/network_captures"
mitmweb --listen-host 0.0.0.0 --listen-port 8080 --set block_global=false
```

**Verification:**

- Emulator proxy points to host IP:8080.
- Launch produces flows in mitmweb.
- Export flows to `research/network_captures/YYYYMMDD-first-launch.mitm`.

---

## 8. Phase 4 — Local Backend MVP

### Task 9: Initialize backend project

**Objective:** Create a simple local server with tests.

**Files:**

- Create: `server/pyproject.toml`
- Create: `server/app/main.py`
- Create: `server/tests/test_health.py`

**Recommended stack:** Python FastAPI, pytest, uvicorn.

**`server/pyproject.toml`:**

```toml
[project]
name = "spiralwarrior-local-server"
version = "0.1.0"
description = "Local offline backend emulator for Spiral Warrior preservation lab"
requires-python = ">=3.11"
dependencies = [
  "fastapi>=0.111",
  "uvicorn[standard]>=0.30",
  "pydantic>=2.7",
]

[project.optional-dependencies]
dev = ["pytest>=8", "httpx>=0.27"]

[tool.pytest.ini_options]
testpaths = ["tests"]
```

**`server/app/main.py`:**

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse, PlainTextResponse

app = FastAPI(title="Spiral Warrior Local Server")

@app.get("/health")
def health():
    return {"ok": True, "service": "spiralwarrior-local-server"}

@app.api_route("/{path:path}", methods=["GET", "POST"])
async def catch_all(path: str, request: Request):
    body = await request.body()
    print({
        "method": request.method,
        "path": "/" + path,
        "query": str(request.url.query),
        "headers": dict(request.headers),
        "body_len": len(body),
    })
    return JSONResponse({
        "ok": True,
        "local": True,
        "path": "/" + path,
        "message": "stub response - implement schema after capture",
    })
```

**`server/tests/test_health.py`:**

```python
from fastapi.testclient import TestClient
from app.main import app


def test_health():
    client = TestClient(app)
    r = client.get('/health')
    assert r.status_code == 200
    assert r.json()['ok'] is True
```

**Verification:**

```bash
cd server
python3 -m venv .venv
. .venv/bin/activate
pip install -e '.[dev]'
pytest -q
uvicorn app.main:app --host 0.0.0.0 --port 18080
curl http://127.0.0.1:18080/health
```

Expected: tests pass and health returns JSON.

### Task 10: Add manifest endpoints

**Objective:** Serve local resource manifests to satisfy update checks.

**Files:**

- Modify: `server/app/main.py`
- Create: `server/app/data/project.manifest`
- Create: `server/app/data/version.manifest`
- Create: `server/tests/test_manifests.py`

**Initial manifest shape:**

Use Cocos-style JSON. Adjust after observing exact client expectations.

```json
{
  "packageUrl": "http://127.0.0.1:18080/remote-assets/",
  "remoteManifestUrl": "http://127.0.0.1:18080/remote-assets/project.manifest",
  "remoteVersionUrl": "http://127.0.0.1:18080/remote-assets/version.manifest",
  "version": "1.1.0.96-local-en",
  "assets": {},
  "searchPaths": []
}
```

**Routes to support:**

```text
/oversea/remote-assets/project.manifest
/oversea/remote-assets/version.manifest
/x3-hd/remote-assets/project.manifest
/x3-hd/remote-assets/version.manifest
/remote-assets/project.manifest
/remote-assets/version.manifest
```

**Verification:**

```bash
pytest -q
curl http://127.0.0.1:18080/remote-assets/project.manifest
```

---

## 9. Phase 5 — APK Redirection and Patching

### Task 11: Try DNS/hosts redirection before binary patching

**Objective:** Avoid repacking APK until necessary.

**Method options:**

1. Emulator `/etc/hosts` if rooted.
2. Router/DNS override.
3. mitmproxy host remap.
4. Local Wi-Fi DNS server.

**Hosts to redirect:**

```text
sdk-login-cocooversea.17m3.com
sdk-package-update-config.17m3.com
ups-config.17m3.com
sdk-heartbeat.17m3.com
cdnsgp-x3jl-release.17m3.com
cdn-spinarena.17m3.com
lxys-area.17m3.com
lxys-audit-area.17m3.com
lxys-test-area.17m3.com
lxys1-client.17m3.com
```

**Verification:**

Inside emulator:

```bash
adb shell getprop | grep dns
adb shell ping -c 1 sdk-login-cocooversea.17m3.com
```

Expected: resolves to local dev machine or proxy.

### Task 12: Patch client strings only if DNS is insufficient

**Objective:** Replace hardcoded domains with local hostnames/IPs.

**Files:**

- Create: `patches/README.md`
- Create: `tools/patch_hosts.py`

**Important:** Direct string patching must preserve byte length unless rebuilding resources. Prefer a same-length dev hostname or apktool-level resource edits.

**Patch candidates:**

```text
intl_base/assets/dh_config.json
intl_base/assets/lebian/globalSettings.xml
intl_assets/assets/res/... manifest refs
cn_apk/assets/...
```

**Verification:**

- Repacked APK installs.
- App launches.
- Logcat shows requests to local backend/proxy.

---

## 10. Phase 6 — Login and Server Selection Emulation

### Task 13: Capture first login request

**Objective:** Know exact method, params, headers, and response expectation.

**Evidence to collect:**

```text
research/network_captures/login-request.md
research/launch_logcat/login-attempt.log
```

Record:

- URL/path;
- HTTP method;
- query/body;
- encryption/signature fields;
- SDK/device headers;
- client behavior after response.

### Task 14: Implement minimal login response

**Objective:** Return a response that the client accepts.

**Files:**

- Modify: `server/app/routes/login.py`
- Modify: `server/app/main.py`
- Create: `server/tests/test_login.py`

**Approach:**

Start with the simplest possible success JSON/XML/text and refine based on logcat. If response parsing fails, locate parser in `.jsc`/Java classes and match schema.

**Possible endpoint names:**

```text
/DHSDK/Action_UserLogin.aspx
/Wbsrv/Check_Login_DH.aspx
/login/1/
/auth/es
```

**Verification:**

- Client no longer fails at SDK login.
- Next request appears in logs.

### Task 15: Implement local server-list response

**Objective:** Let client select a local offline server.

**Expected English values:**

```text
Server ID: 1
Server Name: Offline English
Status: Smooth
Host: 127.0.0.1 or local backend host
```

**Verification:**

- Client displays/uses a server list.
- Next account/profile request appears.

---

## 11. Phase 7 — Profile, Save Data, and Lobby Entry

### Task 16: Build local save model

**Objective:** Define persistent local player data.

**Files:**

- Create: `server/app/models/save.py`
- Create: `server/app/saves/default_profile.json`
- Create: `server/tests/test_save_model.py`

**Initial data fields:**

```json
{
  "account_id": "local_player",
  "nickname": "Player",
  "language": "en",
  "level": 1,
  "currency": {
    "coins": 999999,
    "gems": 999999
  },
  "tutorial_state": "complete_or_start",
  "inventory": [],
  "tops": [],
  "characters": [],
  "stages_unlocked": [1]
}
```

**Verification:**

```bash
pytest -q server/tests/test_save_model.py
```

### Task 17: Emulate profile load

**Objective:** Return enough player/profile data to enter lobby.

**Process:**

1. Capture next failing endpoint after server selection.
2. Add route stub.
3. Return default profile.
4. Inspect client parse errors.
5. Add fields until client advances.

**Verification:**

- Client reaches main lobby or next deterministic screen.

---

## 12. Phase 8 — English Localization

### Task 18: Force English environment

**Objective:** Ensure Android/client chooses English resources where available.

**Methods:**

- Set emulator language to English.
- Inspect app locale selection in logs.
- Patch local backend to send `language=en`.
- Patch SDK/device headers if language is sent to server.

**Verification:**

- Visible client UI is English where existing English text exists.

### Task 19: Backend English strings

**Objective:** Make all server-provided text English.

**Files:**

- Create: `server/app/data/en/messages.json`
- Create: `server/app/data/en/server_list.json`
- Create: `server/app/data/en/notices.json`

**Examples:**

```json
{
  "server_name": "Offline English",
  "login_success": "Login successful",
  "maintenance": "Offline local mode",
  "welcome_mail_title": "Welcome to Spiral Warrior Offline",
  "welcome_mail_body": "This is a local preservation build. Progress is saved on this PC."
}
```

**Verification:**

- Any notices/mail/server names displayed by client are English.

### Task 20: Identify hardcoded Chinese UI text

**Objective:** Locate Chinese text inside Cocos JSON/JSC/assets.

**Files:**

- Create: `tools/extract_cjk_strings.py`
- Create: `docs/ENGLISH_LOCALIZATION_NOTES.md`

**Script approach:**

Scan JSON/JSC/plain assets for CJK ranges and output path + snippet + candidate translation.

**Verification:**

```bash
python tools/extract_cjk_strings.py > docs/ENGLISH_LOCALIZATION_NOTES.md
test -s docs/ENGLISH_LOCALIZATION_NOTES.md
```

### Task 21: Patch text assets incrementally

**Objective:** Translate only visible blocking text first.

**Priority order:**

1. Login/server selection.
2. Main lobby.
3. Tutorial prompts.
4. Stage/battle UI.
5. Inventory/item names.
6. Shop/gacha/event text.
7. Lower-priority flavor text.

**Verification:**

- Screenshot before/after in `research/screenshots/`.
- No layout-breaking overflow in common UI panels.

---

## 13. Phase 9 — Gameplay Loop Reconstruction

### Task 22: Stage/battle entry capture

**Objective:** Identify APIs needed to enter a single-player stage.

**Evidence:**

- route path;
- request body;
- required config IDs;
- battle seed/config;
- client-side vs server-side result handling.

### Task 23: Implement minimal battle start response

**Objective:** Let the client enter one playable battle/stage.

**Approach:**

- Use static configs from APK assets where possible.
- Return deterministic local seed.
- Disable multiplayer/ranking paths.

**Verification:**

- Client enters battle scene.

### Task 24: Implement battle result/save response

**Objective:** Persist progression after a battle.

**Data to save:**

- stage cleared;
- XP/level;
- rewards;
- inventory changes;
- timestamp.

**Verification:**

- Clear battle.
- Restart app/server.
- Progress remains.

---

## 14. Phase 10 — Packaging for Friends

### Task 25: Create local launcher

**Objective:** Start backend and provide emulator instructions.

**Files:**

- Create: `dist/launcher.sh`
- Create: `dist/README_START_HERE.md`

**Launcher responsibilities:**

1. Start local backend.
2. Show URL/port.
3. Optionally launch emulator if configured.
4. Print troubleshooting steps.

### Task 26: Build distribution folder

**Objective:** Produce a clean shareable archive.

**Folder:**

```text
dist/SpiralWarrior-English-Offline/
  README_START_HERE.md
  client/
  server/
  tools/
  saves/
  docs/
```

**Do not include:**

- unrelated research logs with private hostnames/tokens;
- unnecessary original third-party SDK docs;
- payment credentials;
- personal absolute paths.

**Verification:**

- Copy dist folder to a clean machine/user account.
- Follow README from scratch.
- Launch reaches same milestone.

---

## 15. Testing Strategy

### Static tests

```bash
sha256sum -c CHECKSUMS.sha256
python tools/scan_endpoints.py
pytest -q server/tests
```

### Runtime tests

1. Fresh emulator install.
2. Launch without backend: confirm expected failure.
3. Launch with backend: confirm improved state.
4. Capture screenshots at each milestone.
5. Restart app/server: confirm save persistence.

### Regression evidence

For each milestone, save:

```text
research/launch_logcat/YYYYMMDD-milestone-name.log
research/screenshots/YYYYMMDD-milestone-name.png
research/network_captures/YYYYMMDD-milestone-name.mitm
```

---

## 16. Risks and Mitigations

### Risk: Client crashes before network calls

Mitigation:

- Test older Android versions.
- Install split APK correctly.
- Check ABI/native library support.
- Inspect logcat for missing SDK/libs.
- Patch/remove dead SDK initialization if needed.

### Risk: HTTPS pinning or signatures

Mitigation:

- Prefer DNS redirection for HTTP endpoints.
- Use Frida/objection only in lab if necessary.
- Patch network security config if present.
- Replace hardcoded endpoints with HTTP local equivalents if possible.

### Risk: `.jsc` scripts are difficult to decompile

Mitigation:

- Instrument runtime/logcat first.
- Use Cocos JSB hooks.
- Search strings and component handler names in JSON prefabs.
- Patch by observation rather than fully decompiling where possible.

### Risk: Server-authoritative gameplay

Mitigation:

- Start with a minimal offline proof: login → lobby → one stage.
- Implement deterministic local responses.
- If battle simulation is server-side, scope to local demo/remake of battle result flow first.

### Risk: English text incomplete

Mitigation:

- Use international client as base.
- Put all backend text in English.
- Translate visible hardcoded text incrementally.
- Avoid translating huge unused text dumps until shown in-game.

---

## 17. Definition of Done

### Prototype Done

- International client installs and launches in emulator.
- Local backend handles first requests.
- User reaches login/server/lobby with English text.
- Evidence saved in `research/`.

### MVP Done

- User can start app from PC instructions.
- User can enter lobby with local profile.
- At least one single-player activity/stage is playable or demonstrably enters battle.
- Progress saves locally.
- All visible MVP text is English.

### Shareable Build Done

- Clean `dist/` package exists.
- Setup guide is written for non-developers.
- Fresh-machine test passes.
- Known limitations are documented.

---

## 18. Immediate Next Actions

1. Create `README.md` and copy this plan to `docs/PROJECT_PLAN_ENGLISH.md`.
2. Create executable scripts in `tools/`.
3. Install the international XAPK on an emulator.
4. Capture first launch logcat.
5. Build the FastAPI local backend scaffold.
6. Redirect one known endpoint to the local backend.
7. Iterate until the first screen advances.
