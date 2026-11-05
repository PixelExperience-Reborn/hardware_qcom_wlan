# PE-Reborn Android 17 — Qualcomm WLAN source modernization

**2026-11-05 | P2 row 153: `hardware/qcom/wlan` | official AOSP17 history merged, device build OPEN.**

## Provenance: real Git history, not a renamed Android14 branch

| Role | Commit / tree |
| --- | --- |
| Historical PE `seventeen` parent | `e11906fd983bad90a5cdb7601efbea81f7264c85` |
| Official AOSP `android-17.0.0_r1` commit | `b467db6eb6e31cc160a2600d0c8cb4e20c52f3fa` |
| Official source tree | `20b83951680bc43da6f697a354b271367ecb59c0` |
| Reviewed two-parent merge | `6e067e8856248f05154a4d27c979241cc7cac87f` |

Official source: https://android.googlesource.com/platform/hardware/qcom/wlan/+/refs/tags/android-17.0.0_r1

Source integration workflow: [GitHub Actions #42006107297 SUCCESS](https://github.com/PixelExperience-Reborn/hardware_qcom_wlan/actions/runs/42006107297). The merge run fetched Google's exact tag and verified both commit and tree SHAs. It preserved old PE ancestry and modern official AOSP17 ancestry.

## Actual conflict resolution

A standard `git merge` surfaced **one** conflict in `legacy/qcwcn/wifi_hal/common.cpp`:
- PE used relative `#include "wifi_hal.h"`.
- AOSP17 uses public `#include <hardware_legacy/wifi_hal.h>` matching its Soong header interface.
- Independently merged PE's `#include <linux-private/linux/fib_rules.h>` (historical `bbcfb6f705e3`) remains intact.

The CI resolved **only this reviewed exact hunk**; unexpected conflict paths or changed marker contents abort without a push. No global `-X theirs`, `-X ours`, force update or drop of original PE commits. The final `common.cpp` retains the private kernel header and follows the AOSP17 public Wi-Fi HAL include.

## Current source tree and platform contract

- Official AOSP17 source now contains modern root `Android.bp` with `wifihal_qcom_defaults`, `libwifi-hal-qcom` and Soong `qcom_wifi` options, plus `legacy`, `wcn6740` and `wcn7760` trees. Source presence does not imply the device uses wcn7760.
- AOSP17 platform reference manifest already checks out `platform/hardware/qcom/wlan` to `hardware/qcom/wlan`. Only an **additional isolated P2 source-graph profile** may select the merged `PixelExperience-Reborn/hardware_qcom_wlan` checkout at the **same** path, instead of a second duplicate WLAN project.
- The existing `LF52406/mondrian_android_device_xiaomi_sm8450-common` `lineage-24.0` reference sets `BOARD_WLAN_DEVICE := qcwcn`, `WIFI_DRIVER_BUILT := qca_cld3`, uses NL80211 and selects `libwifi-hal-qcom` plus `android.hardware.wifi-service`. It copies firmware configs for `qca6490` and `qca6750`.
- `BOARD_WLAN_CHIP` was **not defined in the inspected `BoardConfigCommon.mk`**, so the exact Soong chip variant and `libcld80211` module path must be resolved across the complete *independent* PE device product rather than guessed.
- Keep separate old `hardware/qcom-caf/wlan` archive **disabled**, not simultaneously included. It is not a second Wi-Fi HAL source owner.

## Required gates, NOT completed

1. Validate the source-pinned profile with official `repo init`, `repo manifest`, selected-project `repo sync` and resolved Git SHAs. Static manifest checks are not a full sync.
2. Test Android17 Soong/Make compilation of `libwifi-hal-qcom`, `libcld80211`, `lib_driver_cmd_qcwcn` / supplicant helpers and `android.hardware.wifi-service`, along with versioned stable interfaces.
3. Resolve `BOARD_WLAN_CHIP` and the exact vendor/firmware/kernel/UAPI source against QCA6490/6750, including `libnl`, PASN/PMF, GSCAN, NAN, RTT and link-layer stats paths.
4. Test `checkvintf`, SELinux enforcing, init permissions and all product module owners; do not introduce duplicate Soong namespace/module sources or obsolete PE hacks.
5. Device runtime: STA association, roaming, AP/tethering, STA+P2P/NAN concurrency, Wi-Fi Direct/WFD, WPA2/WPA3, IPv4/IPv6, suspend/resume, low-latency gaming and sustained throughput on physical mondrian.

**Scope:** successful source-level merge only. No complete AOSP17 platform build, GNSS/Bluetooth coex, device networking, VTS or hardware Wi-Fi test has been claimed. Original `seventeen` and current MistOS device trees were not changed.

**Next matrix row:** `hardware/ril` (after pinning and validating the WLAN review profile).
