# VN081122-releases
Official firmware releases for VN081122 devices (pH thermometer)

Official firmware distribution repository for VN081122 devices.

Firmware binaries are provided as GitHub Release assets.

Stable releases are intended for normal use. Beta releases contain changes undergoing testing.

Firmware files are encrypted and signed for compatible VN081122 devices. Installation causes the device to restart.

This repository contains firmware distribution files and update metadata. Firmware source code is maintained separately.

## Choosing a channel

| Channel | Intended use |
| --- | --- |
| Stable | Qualified releases for normal operation |
| Beta | Prereleases for testing changes |

Stable firmware checks `stable/manifest.json`; beta firmware checks `beta/manifest.json`. Development builds check stable. The current web UI has no channel selector and performs no scheduled checks or automatic installations.

## Installing an update

1. Open the device web UI and expand the Firmware update card.
2. Click **Check for updates**. The device needs Internet access.
3. Review the available version, then click **Install update**.
4. Wait for installation and restart, reload the web UI, and check again to verify the installed release.

Installation interrupts measurements. Keep the device powered during installation.
Alternatively, download the encrypted `.bin` asset from a release and use **Upload and install**. A compatible build from another channel can be installed this way; the installed build determines subsequent channel checks.

`Current: stable #4 (aeb6dd5)` means channel stable, release sequence 4, and source commit beginning `aeb6dd5`. The values do not indicate whether installation used USB or OTA. Only a higher release sequence is offered as an update.
If no manifest exists for a channel, the device reports that no firmware has been published for that channel.

## Release files

- `VN081122_ota_encrypted-<commit>.bin`: signed firmware wrapped in the device's encrypted OTA transport format.
- `manifest.json`: product, target, channel, sequence, source commit, asset URL, encrypted size, and SHA-256.
- `stable/manifest.json` and `beta/manifest.json`: pointers to the currently advertised release for each channel, updated after the release assets are available.

SHA-256 checks transfer integrity; device signature verification establishes firmware authenticity. Published firmware assets are not overwritten.

## Maintainer commands

Run these from the private firmware checkout `C:\VN081122-FW`, with ESP-IDF, Node/npm, private keys, and authenticated GitHub CLI available. The publishing script is not included in this public distribution repository.
Choose unused sequences greater than all existing stable and beta releases; 5 and 6 below are examples.

```powershell
gh release list --repo littleboot/VN081122-releases --limit 100

# Build and publish beta.
.\helper_scripts\release.ps1 -Channel beta -Sequence 5
.\helper_scripts\release.ps1 -Channel beta -Sequence 5 -ReusePackage -Publish

# Build and publish stable after qualification.
.\helper_scripts\release.ps1 -Channel stable -Sequence 6
.\helper_scripts\release.ps1 -Channel stable -Sequence 6 -ReusePackage -Publish
```

Use the same `-ReusePackage -Publish` command to resume an interrupted publication. New published builds require committed source; reused packages retain their original build identity and hashes.

## Factory flashing and production

Public OTA assets are **not blank-device factory images** and must not be written directly with esptool or ESP-Prog.
A fresh board requires the matching signed bootloader, partition table, initial OTA data, and signed application from the private manufacturing build. ESP-Prog's UART programming interface is distinct from JTAG; Secure Boot/flash encryption provisioning can permanently disable JTAG.

The production process is: inspect and functionally test the assembled board, qualify the production security configuration, flash the complete manufacturing image on a fresh board, allow first-boot security provisioning, then verify security, assign device identity, calibrate, and record acceptance tests.
Stable/beta describes release channels, not flash encryption security mode. A stable build does not automatically switch flash encryption from development to release mode.

The detailed factory procedure, ESP-Prog commands, configuration checks, identity API example, and recovery distinctions are documented in `C:\VN081122-FW\helper_scripts\RELEASING.md` in the private source repository. Use that procedure with the exact manufacturing build; do not apply a fresh-board flashing command to an already encrypted device.
