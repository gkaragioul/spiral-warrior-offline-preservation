# Spiral Warrior / 螺旋勇士 public-source triage

## Client builds examined (operator's local copies, not included here)

- International XAPK:
  - Local file: `international_latest.xapk`
  - Package: `com.oversea.spinarena`
  - Version: `1.1.0.96`
  - Version code: `153`
  - Size: `118,094,316` bytes
  - SHA-256: `2e11ee055dfc8b558d1f30e9203a88fb56597d7ca1b1dde3eb1af94feb53beff`
  - Split APKs:
    - `com.oversea.spinarena.apk` SHA-256 `e294af5e59f5058189cc067e9a9c19278330f1aa8cebf4f7a79bb2b865f1d89d`
    - `NewAssets.apk` SHA-256 `586a4724b1135c841dd1d6027c5271b2c0b16999dc87c83365e192d918754bfa`
- Chinese 9game APK:
  - Local file: `cn_9game.apk`
  - Package implied by filename: `com.dianhun.lxys.aligames`
  - Size: `390,314,526` bytes
  - SHA-256: `4e02fbc9adf4e31051140194b55b8004c1272c74d76293208d6b1809c77358d5`

## Engine / packaging

- Both public clients appear to be **Cocos Creator / cocos2d-js**, not Unity.
- The international XAPK contains a base APK plus an asset split. The asset split has thousands of Cocos `res/import` JSON assets and raw assets.
- The Chinese APK is much larger and contains substantially more shipped assets/configs, making it the better source for reconstructing game data.

## Important extracted endpoints

### International SDK / account endpoints

From `intl_base/assets/dh_config.json`:

- `https://sdk-package-update-config.17m3.com`
- `https://ups-config.17m3.com`
- `https://ups-sdk-log.17m3.com`
- `http://ups-sdk-error.17m3.com`
- `https://sdk-login-cocooversea.17m3.com/DHSDK/Action_UserLogin.aspx`
- `https://sdk-pay-coco4game.17m3.com`
- `https://service.e-soul.net`
- `https://upload.e-soul.net`
- heartbeat: `https://sdk-heartbeat.17m3.com/ttl`
- Firebase URL found in strings: `https://spinarena.firebaseio.com`

Live checks:

- `https://spinarena.firebaseio.com/.json` returns `401 Permission denied`.
- `https://ups-config.17m3.com/getconf` returns `{}`.
- Guessed package-update config URLs returned 404.

### Game/resource endpoints found in client assets

International:

- `http://cdnsgp-x3jl-release.17m3.com/oversea/remote-assets/project.manifest`
- `http://cdnsgp-x3jl-release.17m3.com/oversea/remote-assets/version.manifest`

Chinese/public build:

- `http://lxys-area.17m3.com/login/1/?account=`
- `http://lxys-audit-area.17m3.com:25801/auth/es?account=`
- `http://lxys-test-area.17m3.com:15801/auth/es?account=`
- `http://lxys1-client.17m3.com/x3-hd/remote-assets/project.manifest`
- `http://lxys1-client.17m3.com/x3-hd/remote-assets/version.manifest`
- `http://lxys1-client.17m3.com/x3-hd-test/remote-assets/project.manifest`
- `http://lxys1-client.17m3.com/x3-hd-test/remote-assets/version.manifest`
- `http://cdnsgp-x3jl-release.17m3.com/oversea/2`
- `http://cdn-spinarena.17m3.com/oversea/`

Live checks:

- `lxys-area.17m3.com` and `lxys1-client.17m3.com` currently fail DNS resolution.
- `cdnsgp-x3jl-release.17m3.com` currently fails DNS resolution.
- `lxys-test-area.17m3.com:15801` resolves but connection is refused.

## Initial feasibility conclusion

A private-server revival is plausible from the client builds alone because:

1. two client builds (international and Chinese) exist to compare;
2. the Chinese APK includes a large amount of shipped asset/config data;
3. game/resource/login endpoint names are recoverable from assets;
4. the client is Cocos JS, which is generally more approachable than IL2CPP for protocol reconstruction.

Main blockers:

1. core gameplay scripts are packaged as Cocos `.jsc` bytecode / compiled JS artifacts, so decompilation or runtime instrumentation is needed;
2. original production DNS/CDN endpoints are dead;
3. backend API response schemas still need to be reconstructed by decompiling scripts and/or running the client under proxy/logcat.

## Recommended next milestone

Build a local reconstruction lab:

1. Install both APK variants on an Android emulator.
2. Capture `adb logcat` and network traffic on first launch.
3. Decompile/disassemble the Cocos `.jsc` project files enough to identify login/server-list/request handlers.
4. Stand up a local HTTP mock for the discovered endpoints.
5. Patch DNS or APK host strings to redirect to the mock server.
6. Advance the client screen-by-screen, implementing only endpoints the client actually requests.
