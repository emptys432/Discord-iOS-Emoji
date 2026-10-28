# iOS Emoji for any Discord build

An LSPosed module that replaces the emoji images bundled inside Discord with the
iOS ones. It is not tied to the `com.discord` package: it applies to any Discord
build you enable it for in LSPosed — the official app, mods, repacks, and clones
with custom package names.

The module has no launcher screen, no settings, and requests no permissions. Scope
is chosen entirely in LSPosed; there are no config files and no per-launch scanning.

## Download

APK: [Releases](../../releases/latest) — `iOS-Emoji-Discord-v4.3-anyapp.apk` (~30 MB).

## Install

1. Uninstall any older iOS Emoji module (different package and signing key).
2. Install the APK.
3. LSPosed → Modules → **iOS Emoji for any Discord build** → enable it, then tick
   every Discord build you want.
4. Force-stop the ticked apps, or reboot.

## Verify

LSPosed → Logs should show, for each enabled app:

```
iOS Emoji Assets: ready in <process name>; 3996 images
iOS Emoji Assets: replaced emoji-1f600.png
```

## How it works

Discord keeps its emoji as PNG files inside the APK (`assets/emoji-<codepoints>.png`),
so replacing the system emoji font has no effect on them. This module ships 3996 iOS
emoji PNGs plus the matching Discord asset names. At runtime it hooks
`AssetManager.open` and `openFd` and, only when the requested name matches one of
those known files, returns the replacement image. Everything else — other assets,
custom server emoji, stickers — passes through untouched. If a lookup fails, the
original image is returned and nothing breaks.

Replacement happens only in processes where LSPosed loaded the module, and only for
exact known names, so unrelated applications are unaffected.

## Compatibility

- LSPosed, Android 11+ (minSdk 30).
- Tested with the official Discord app and a repacked clone.
- Any Discord build that ships `assets/emoji-*.png` should work; builds with a
  different emoji pipeline keep their own emoji.
- If Android rejects the install because of a signing mismatch, uninstall the
  existing module, install this APK, and enable its scope in LSPosed again.

## Build

The prebuilt APK in Releases is the supported artifact. Rebuilding needs more than
this repository contains:

- JDK 17+, Android SDK platform and build-tools 36 (`aapt2`, `d8`, `apksigner`,
  `zipalign`) via `ANDROID_HOME`
- Xposed API 82 jar at `build/dependencies/xposed-api-82.jar`:
  `https://api.xposed.info/de/robv/android/xposed/api/82/api-82.jar`
- the iOS emoji font from the upstream project, `system/fonts/NotoColorEmoji.ttf`
  (~35 MB). The 3996 PNGs are extracted from that font at build time using HarfBuzz;
  they are not stored in this repository.
- `version` and `versionCode` in `module.prop`

```sh
python3 -m venv build/python
build/python/bin/python -m pip install -r tools/requirements-discord.txt
build/python/bin/python tools/build_discord.py
```

The APK is written to `build/`. The first build creates a signing key and a random
password under `.signing/`; back both up, they are required for future updates.
`build/discord/coverage.json` records how many images were replaced.

## Background

dtingley11's [KernelSU-iOS-Emoji](https://github.com/dtingley11/KernelSU-iOS-Emoji)
is a KernelSU module that replaces the system emoji font. Discord ignores that,
because its emoji are PNG files inside the APK, so that project also ships a small
LSPosed application for Discord. This repository is a standalone continuation of
that application only — the system font module is not included here.

Changes compared to the original application:

- the hardcoded `com.discord` check was removed, so the scope comes from LSPosed and
  clones with any package name work;
- initialization is idempotent, because LSPosed can invoke `handleLoadPackage` twice
  per process, which previously installed the hooks twice and duplicated log lines;
- own package id `app.iemoji.anyapp`, so both versions can be installed side by side
  (enable only one of them per app);
- asset filtering is unchanged: only the 3996 known `assets/emoji-*.png` are replaced.

## Credits

- Original module, code structure and the iOS emoji assets: **dtingley11**
- Package-name support, idempotent init, packaging: **emptys432**

Emoji artwork originates from Apple's iOS and is redistributed as part of the
original project's assets.

## Personal build

No maintenance or support is planned. For the system font module and general
issues, use the upstream project.
