# avnc — Architecture

## Overview

`avnc` is a minimal Android fork of the open-source [gujjwal00/avnc](https://github.com/gujjwal00/avnc) VNC client. The fork contains exactly **2 commits** of Praan-specific changes on top of upstream. Everything else — VNC protocol handling, rendering, gestures, networking — is inherited from upstream unchanged.

This document covers only the Praan delta. For upstream internals, refer to the [upstream repository](https://github.com/gujjwal00/avnc).

---

## System Context

```
+---------------------+        WiFi LAN (10.42.0.x)       +----------------------+
|  Android Tablet /   |  <-------- VNC (RFB / port 5900) --|  Praan "Backpack"    |
|  Phone              |                                     |  3D Mapping Device   |
|  (avnc app)         |                                     |  (VNC server)        |
+---------------------+                                     +----------------------+
```

The backpack device hosts its own WiFi hotspot. The Android device running avnc connects to that hotspot and VNC-connects to `10.42.0.1:5900`. The avnc app is a thin remote-control UI for the backpack.

---

## What Praan Changed (vs Upstream)

The Praan-specific changes are confined to 6 files. Everything else is upstream code.

### Commit 1 — `db42c30` ("praan branding with auto server add on install")

| File | Change |
|---|---|
| `HomeActivity.kt` | Added first-run guard that pre-creates a VNC server profile |
| `strings.xml` | `app_name`: `AVNC` → `Praan 3d mapping` |
| `home_drawer_header.xml` | Replaced upstream wordmark `ImageView` with Praan `app_logo.png` |
| `home_drawer.xml` | Removed "About" and "Report Bug" `<item>` entries |
| `app/src/main/res/mipmap-*/` | All launcher icons replaced with Praan-branded `.webp` icons |
| `keystore/password.txt` | `1234@Abcd` committed in plaintext |
| `app/release/app-release.apk` | Signed release APK (v2.5.3, versionCode 35) committed to repo |

### Commit 2 — `0d122f0` ("changes in server settings, app name change")

| File | Change |
|---|---|
| `HomeActivity.kt` | `gestureStyle` → `"touchscreen"`, `imageQuality` = `0`, `useRawEncoding` = `true`, `screenOrientation` = `"landscape"` |
| `strings.xml` | `app_name`: `Praan 3d mapping` → `Praan 3D Mapping` (capitalisation fix) |
| `BasicHomeTest.kt` | Tests for About/BugReport menu items commented out (items were removed) |

---

## First-Run Auto-Connect Logic

The key Praan behaviour lives entirely in `HomeActivity.kt`. On first launch:

```
App starts
    |
    v
HomeActivity.onCreate()
    |
    v
SharedPreferences.getBoolean("is_first_run", true)
    |
   true?
    |
    v
Build VncProfile:
  host     = "10.42.0.1"
  port     = 5900
  name     = "backpack"
  password = "123456"
  gestureStyle     = "touchscreen"
  imageQuality     = 0       (raw, uncompressed)
  useRawEncoding   = true
  screenOrientation = "landscape"
  fConnectOnAppStart = true
    |
    v
Insert profile into local DB (Room database, inherited from upstream)
    |
    v
SharedPreferences.put("is_first_run", false)
    |
    v
App proceeds — upstream auto-connect fires because fConnectOnAppStart = true
```

On all subsequent launches, the `is_first_run` guard is false, so the pre-fill block is skipped. The saved profile in the Room database is loaded by upstream code and the connection is established automatically.

---

## Inherited Upstream Architecture

The following is a brief summary of the upstream architecture that Praan's fork inherits without modification.

### Tech Stack

| Layer | Technology |
|---|---|
| Language | Kotlin |
| Platform | Android (native) |
| Build | Gradle |
| VNC Protocol | RFB over TLS (wolfSSL, git submodule) |
| Architecture | MVVM (ViewModel + LiveData) |
| Local Storage | Room (SQLite) — stores server profiles |
| Min SDK | Android 8.0 (API 26) |

### VNC Protocol

The app implements the RFB (Remote Framebuffer) protocol. wolfSSL is included as a git submodule to provide TLS transport. The upstream supports multiple VNC encodings (Raw, Tight, ZRLE, etc.); Praan forces **Raw encoding** (`useRawEncoding = true`) and **quality 0** for maximum performance on the local LAN connection to the backpack.

### Data Flow

```
VNC Server (backpack)
        |
    RFB Protocol (TCP 5900)
        |
        v
    VncViewModel
        |
    FramebufferView  <-- renders remote screen via Canvas / OpenGL
        |
    GestureHandler   <-- translates touch events → RFB pointer/key events → server
```

### Removed UI Elements

The upstream drawer includes "About" and "Report Bug" items. Both are removed in Praan's `home_drawer.xml`, which simplifies the UI for field use and prevents engineers from accidentally reporting bugs to the upstream project.

---

## Signing and Release

A signing keystore is committed at `keystore/` alongside `keystore/password.txt` (plaintext password: `1234@Abcd`). The signed release APK is committed at `app/release/app-release.apk` (v2.5.3, versionCode 35) for direct sideloading onto field devices.

> **Security note**: Committing keystores and passwords to version control is a security risk. If this app is ever distributed outside internal field use, the keystore should be rotated and moved to CI secrets.

---

## Upstream Maintenance

The fork is 256 commits behind upstream master (as of September 2025). To pull upstream improvements:

```bash
git remote add upstream https://github.com/gujjwal00/avnc.git
git fetch upstream
git merge upstream/master
# Expect conflicts only in:
#   app/src/main/java/com/gaurav/avnc/ui/home/HomeActivity.kt
#   app/src/main/res/values/strings.xml
```

All Praan-specific changes are isolated to the 6 files listed above, which keeps merge conflicts manageable.

---

## Files Modified by Praan

```
app/src/main/
├── java/com/gaurav/avnc/ui/home/
│   └── HomeActivity.kt          ← First-run auto-profile injection
├── res/
│   ├── values/
│   │   └── strings.xml          ← App name
│   ├── drawable/
│   │   └── app_logo.png         ← Praan logo (drawer header)
│   ├── layout/
│   │   └── home_drawer_header.xml  ← Drawer header (logo swap)
│   ├── menu/
│   │   └── home_drawer.xml      ← Drawer menu (About + BugReport removed)
│   └── mipmap-*/                ← Launcher icons (all variants)
keystore/                        ← Android signing keystore
app/release/app-release.apk      ← Pre-built signed APK
```
