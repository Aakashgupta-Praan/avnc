# avnc — Praan 3D Mapping

A Praan-customised fork of [gujjwal00/avnc](https://github.com/gujjwal00/avnc), an open-source VNC client for Android. Rebranded and pre-configured to connect to Praan's portable "backpack" 3D mapping device automatically on first install.

Upstream: `gujjwal00/avnc` (this fork is 2 commits ahead, 256 commits behind upstream master as of September 2025)

## What This Does

Praan field engineers carry a portable device (referred to as "backpack") that runs a VNC server. This Android app is installed on a tablet or phone to display and interact with the backpack's interface over a local WiFi hotspot, enabling 3D room mapping during site surveys.

On first install, the app automatically:
1. Creates a saved VNC server profile pointing to the backpack (`10.42.0.1:5900`)
2. Connects to it immediately on launch
3. Sets display to landscape, raw encoding, lowest compression for performance on a local network

## Praan Customisations (vs upstream)

| Change | Detail |
|---|---|
| App name | `AVNC` → `Praan 3D Mapping` |
| App icon | Replaced upstream icon with Praan branding |
| Drawer header | Replaced upstream wordmark with Praan logo (`app_logo.png`) |
| Auto server profile | Pre-fills VNC server on first run: host `10.42.0.1`, port `5900`, name `backpack`, password `123456` |
| Auto-connect | `fConnectOnAppStart = true` — connects immediately on app open |
| Gesture style | `touchpad` → `touchscreen` |
| Image quality | Set to `0` (lowest, optimised for LAN performance) |
| Encoding | Raw encoding enabled (`useRawEncoding = true`) |
| Screen orientation | Forced landscape |
| Menu items removed | "About" and "Report Bug" entries removed from drawer |
| Compiled APK | `app/release/app-release.apk` (v2.5.3, versionCode 35) committed to repo |

## Pre-configured Server Profile

```
Host:     10.42.0.1        (backpack WiFi hotspot IP)
Port:     5900             (standard VNC)
Name:     backpack
Password: 123456
Gesture:  touchscreen
Quality:  0 (raw, uncompressed — best for local LAN)
Orient:   landscape
Auto-connect: on app start
```

This profile is only created on first run (guarded by a `SharedPreferences` `is_first_run` flag). Subsequent launches will use the saved profile.

## Tech Stack

Inherited from upstream `gujjwal00/avnc`:

| Layer | Technology |
|---|---|
| Language | Kotlin |
| Platform | Android (native) |
| Build | Gradle |
| VNC protocol | RFB over TLS (wolfSSL) |
| Architecture | MVVM (ViewModel + LiveData) |
| Min SDK | Android 8.0 (API 26) |

## Building

```bash
# Clone with submodules (wolfSSL is a git submodule)
git clone --recurse-submodules https://github.com/Praan-Climate-Technologies/avnc

# Build release APK
./gradlew assembleRelease
```

The release APK is also committed directly at `app/release/app-release.apk` for direct sideloading.

## Keystore

A signing keystore is committed in the `keystore/` directory. The password is in `keystore/password.txt`.

> Note: Committing keystores and passwords to version control is a security risk. If this app is ever distributed publicly, the keystore and credentials should be rotated and moved to CI secrets.

## Upstream Maintenance

This fork is significantly behind upstream (256 commits). To pull upstream improvements:

```bash
git remote add upstream https://github.com/gujjwal00/avnc.git
git fetch upstream
git merge upstream/master
# Resolve conflicts in HomeActivity.kt and strings.xml
```

The Praan-specific changes are confined to:
- `app/src/main/java/com/gaurav/avnc/ui/home/HomeActivity.kt`
- `app/src/main/res/values/strings.xml`
- `app/src/main/res/drawable/app_logo.png`
- `app/src/main/res/mipmap-*/` (icons)
- `app/src/main/res/layout/home_drawer_header.xml`
- `app/src/main/res/menu/home_drawer.xml`

## License

GPL-3.0-or-later (inherited from upstream). See `COPYING.txt`.
