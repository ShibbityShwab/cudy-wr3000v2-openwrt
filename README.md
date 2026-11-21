# cudy-wr3000v2-openwrt

> **Bench-verified 2-CPU / board-feature baseline on one unit, with a verified automated
> install path and a recorded DTS-regression lesson; NOT a public or flashable release.**
> The 6.18 target image built from this tree boots on the Cudy WR3000 v2.0 and was verified on
> **2026-11-21**: 2 CPUs, wired LAN through the documented CPU-copy mirror workaround, SSH, a
> persistent UBIFS overlay and a 15-minute soak with 0 failures and reboot persistence
> (canonical image `974e6ca6`), then the stock board_id=3 features - six writable
> `oem:green:*` LEDs, `event0` gpio-keys-polled (reset/WPS) and `watchdog0` via
> sysfs (image `7f270b8c`; all six LEDs and both RESET/WPS presses physically verified) - and the
quiet image **`7421bb7a`**, the known-good hardware
> baseline, which was reproduced **byte-for-byte** after the conservative comment-only DTS
> cleanup. The **install path is verified end-to-end** with the published bench stock-telnet
> installer ([tools/](tools/)): it wrote the inactive pair only, left the active pair
> unchanged, verified the selector and booted our 2-CPU image. **Regression record:** the first
> active-DTS cleanup (image `6f19c6d6`) did **not** boot - MTD partition registration stalled
> and the U-Boot watchdog reset the unit - so it was reverted; the conservative cleanup
> restores the hardware-verified devicetree (compiled DTB byte-identical to the known-good
> one). Host checks are necessary but not sufficient: **every active-DTS change must be
> boot-tested on the unit.** The **release line** is **53 target patches** (64 L0 patch files
> 001-060 + 062-065, minus 11 bench-only SMP diagnostics). **W22 native forwarding is now
> release-integrated**: patches **059** (PFE NAPT/flow-engine init) and **060** (PIE native
> exception-class delivery) are in the line, the canonical cmdline uses `luofu_switch.naptinit=1`
> with no CPU-copy mirror tokens, and the mirror patches 054-057 remain as the documented
> fallback. The W22 canonical image **`50023446`** boots with native CPU delivery on the bench
> (NAPT init verified in dmesg, 2 CPUs, ping/SSH, reboot determinism); M1a was first proven with
> image `5800c9ab`. **W22 is release-line only - the L2 upstream review form stays unchanged at
> 26 patches**; M1b (per-port/LAN-to-LAN), the fallback-image rebuild and the soak are pending
> bench steps. DTS-0001 is the current board-DTS entry (`886f3ad0...`, 694 lines), which now
adopts all **five boot-tested devicetree deltas** - TWD level-high, MAC-nvmem removal, switch
MMIO-first `reg`/`reg-names`, `cudy,wr3000-v2` first compatible and the final flashinfo
compatible pair (`"hisilicon,luofu-flashinfo", "hsan,flashinfo_reserved"`) - on top of the
conservative cleanup (image **`6e9c54d0`**; DTS 681 lines `381b2e8c...`; DTB `62574b11...` /
pad `8d79794e...`). **All documented upstream blockers are now closed (zero remain)**; the retained
review material under [docs/incoming/upstream-blockers/](docs/incoming/upstream-blockers/) is
maintainer-facing RFC documentation (driver/YAML items, reserved-memory binding), not blocker
tasks. The review-form patch 0021 was reordered MMIO-first and the form re-ran clean (26/26,
checkpatch 0 errors / 58 warnings); release-line manifest sha256 `1e00b568...`. A
**documentation-only upstream submission package** (artifact checklist,
cover-email draft, explicit no-Tested-by policy;
[`upstream/series-6.18/submission/`](upstream/series-6.18/submission/)) is included for
owner review - **nothing has been sent anywhere**. Thermal is
> **unsupported** (temperature register reads 0, vendor DT disables it) and Wi-Fi is
> **unsupported on 6.18** (vendor-binary driver only). **One Ethernet jack (eth2)** works via the
> CPU-copy workaround; the other jacks need the attended source-port experiment
> ([docs/MULTI-PORT-PLAN.md](docs/MULTI-PORT-PLAN.md)) or the native L2/VLAN bridge. The source-port experiment has now been run with the bench-only
probe image **`a91ebcf0`** and patch 066 (retained under
[docs/incoming/multi-port-probe/](docs/incoming/multi-port-probe/)): it was **negative** - the
CPU-copy completions carry no source-port/vport identifier and are field-for-field identical
across jacks, so multi-port LAN/WAN over CPU-copy is not feasible and the native L2/VLAN bridge
([docs/LSW-SPEC.md](docs/LSW-SPEC.md)) is the required path. The **E0-a** mirror-off
native-delivery experiment was also negative: with `cpu_mirror=0` the unit boots 2 CPUs and
native-path completions arrive on port 11, but every frame is `status=37 err=1` and IP is dead
(the stock CPU-punt/IFC/tagact classification is missing), so the **CPU-copy mirror remains
required**; the read-only E0-b stock-vs-ours captures (9,488 words per side, 0 read failures)
now identify the missing stock-only classifier/punt state and scope the planned native-forwarding
M1a write test (no blind replay). The plan, E0-b results and read-only/print-only instruments are
retained under [docs/incoming/lsw-forwarding/](docs/incoming/lsw-forwarding/), and the
bench-verified **install/recovery lessons** (env-bootflag/selector coupling, pair-b-only rule,
mixed-pair diagnosis, physical recovery) are in
[docs/INSTALL-RECOVERY-LESSONS.md](docs/INSTALL-RECOVERY-LESSONS.md). The owner-only **legacy
history-purge runbook** is prepared and dry-run validated in the notebook docs (not published
here); `PUBLISH-READINESS.md` item 1 remains unchecked - no history rewrite, push or cache
purge has run. Milestones:
> [release status](target-skeleton/RELEASE-NOTES.md) ·
> [release line](upstream/series-6.18/05-release-line.md) ·
> [SMP root cause](docs/SMP-RE.md) ·
> [install/regression record](target-skeleton/RELEASE-NOTES.md) ·
> [multi-port plan](docs/MULTI-PORT-PLAN.md) ·
> [feature inventory](target-skeleton/docs/incoming/feature-inventory/SUMMARY.md) ·
> [Wi-Fi recon](target-skeleton/docs/incoming/wifi-recon/SUMMARY.md).

An upstream-style **OpenWrt / Linux bring-up for the Cudy WR3000 v2.0** - a HiSilicon
**Hi5671Y "luofu"** (board R116) router that ships a vendor OpenWrt 22.03.6 / Linux 5.10.201
image. The goal is a mainline OpenWrt target, device tree and driver set for this board.

This curated repository collects the parts of the research that are **safe and useful to
publish**: the Linux 6.18 release line (53 target patches) and its upstream review form (26
patches: 19 merge-intent plus seven fail-closed data-path drafts marked do-not-merge), an
OpenWrt target skeleton and board DTS with the loader hand-off nodes, an image/install recipe
(with the standard squashfs + `rootfs_data` overlay UBI layout), reverse-engineering
specifications, measured read-only silicon inventory, the retained read-only device feature
inventory (plus its follow-up update) and Wi-Fi feasibility reports, the bench stock-telnet
install tooling, the multi-port Ethernet plan, the DTS cleanup/regression record, and an honest
status report.
It is a curated snapshot of the lab notebook
[**cudy-wr3000v2**](https://github.com/ShibbityShwab/cudy-wr3000v2) with fresh git history; the
security findings from the same research live in the separate
[**cudy-wr3000v2-cve**](https://github.com/ShibbityShwab/cudy-wr3000v2-cve) repository.

**Not in this repository:** vendor firmware images, vendor kernel modules (`.ko`/`.o`),
extracted vendor DTBs, Wi-Fi firmware blobs, calibration data, NAND/rootfs dumps, per-unit
identifiers (MAC/serial/PIN/efuse values), credentials or keys. See [NOTICE.md](NOTICE.md) and
[docs/PUBLISHING.md](docs/PUBLISHING.md) for the publication policy and the provenance of the
reverse-engineered code.

---

## Status at a glance (2026-11-21)

| Area | Host-validated (evidence in this repository) | Hardware status |
| --- | --- | --- |
| Target/subtarget metadata | `make defconfig` + `make target/linux/prepare` RC=0 on `openwrt main` `27663a0` (linux-6.18.60); generated `CONFIG_TARGET_hisilicon_luofu_DEVICE_cudy_wr3000-v2=y` | the target tree built the kernel and machine DTB used by both attended sessions (U-Boot `Image Name: Linux-6.18.60`; boot-log machine model `Cudy WR3000 v2.0 (HiSilicon luofu R116)`) |
| Release line (53 patches) | **53/53** include patches apply in manifest order to pristine linux-6.18.60 with `patch(1)` (149 file events, 3 offset hunks, **0 fuzz, 0 rejects**) and **strict `git apply` is clean**; the L0 stack (64 files, 001-060 + 062-065) applies 64/64. Validated in this repository's layout with `upstream/series-6.18/tools/check-release-line.sh` | the release line now carries **W22 native forwarding** (patch 059 NAPT/flow-engine init + patch 060 PIE native exception-class delivery); manifest sha256 **`619eb15a...`**. The bench images `974e6ca6` / `7f270b8c` / `7421bb7a` were built from the pre-W22 L0 stack plus the loader hand-off DTS fix; `6e9c54d0` is the current DTS-purpose baseline and `50023446` the canonical W22 image (native CPU delivery, no mirror tokens). DTS-0001 is unchanged (sha256 `886f3ad0...`, 694 lines; DTB `62574b11...` / pad `8d79794e...`); zero upstream blockers remain; **W22 is release-line only, not in the L2 review form**. |
| Upstream review form (001-026) | 26/26 apply in order with `patch(1)` and `git am` (identical trees; rerun after the in-series binding patch 0021 MMIO-first reorder, matching the boot-tested DTS); checkpatch 0 errors / 58 warnings / 6 strict CHECKs; raw logs in [validation](upstream/series-6.18/validation/) | the review form is not a merge list and has no upstream review; patches 020-026 are group-4 RFC drafts |
| Kernel compile | full linux-6.18.60 compile of the 001-026 series RC=0 under `CONFIG_WERROR=y`, 0 errors / 0 compiler warnings (37 pre-existing `modpost` warnings only); zImage 3,282,528 B (`08b76747...`); load/entry `0x80608000` unchanged (`_text = stext = 0xc0608000`) | the bench images load that kernel at `0x80608000`; the compile evidence is host-only |
| Board DTS + loader hand-off | the DTS is the **conservative comment-only cleanup plus all five boot-tested deltas** (TWD level-high, MAC-nvmem removal, switch MMIO-first `reg`/`reg-names`, `cudy,wr3000-v2` first compatible, flashinfo compatible pair) - **681 lines**, sha256 `381b2e8c...`; the conservative step alone leaves every active node name, unit address, property order/value, `reg` order, status, label, phandle and alias unchanged from the hardware-verified 1,329-line pre-clean DTS). It compiles to DTB **12,678 B sha256 `62574b11...`** (padded `Build/luofu-pad-dtb` to 12,680 B sha256 `8d79794e...`; the conservative step alone was byte-identical to the pre-clean `49fe5c47...`) and carries `/aliases/console`, `/reserved-memory/flashinfo@80600000` (`reg = <0x80600000 0x4000>`, `atag-offset = <0x1000>`) and `enable-method = "hisilicon,luofu-smp"` | the deltas are boot-tested on images `f6685b80`/`c3ef2a50`, and the final flashinfo compatible pair on **`6e9c54d0`** (stock loader `bootfip` staging PASS with `"hisilicon,luofu-flashinfo"` first; 2 CPUs, `GIC-0 29 Level twd`, cmdline `ubi.mtd=rootfsb` + project marker, no errors) - the loader hand-off/SMP/TWD/MAC/switch/compatible semantics are the hardware-verified ones; the **superseded active-DTS cleanup (image `6f19c6d6`) did not boot** (MTD stall, U-Boot watchdog reset) - see the regression record |
| SMP (patch 018 + DTS contract) | platsmp driver compiles; vendor release sequence decoded and documented; the fix is DTS-only and the diagnostics are excluded from the release line | **2 CPUs verified** on the canonical image (CPU1 mpidr `80000001`, `Brought up 1 node, 2 CPUs`, /proc/cpuinfo 2, soak + reboot) |
| TWD timer (Cortex-A9 local timer; upstream blocker 6) | DTS uses the standard encoding `<GIC_PPI 13 (GIC_CPU_MASK_SIMPLE(2) | IRQ_TYPE_LEVEL_HIGH)>` (0x304) with the `arm-gic.h`/`irq.h` includes; the measured read-only GICD_ICFGR1 PPI13 field is 0b11 and the GIC-400 PPI config is SoC-fixed, so the DT flag does not reprogram hardware | **boot-tested PASS on image `f6685b80`**: `/proc/interrupts` reports `GIC-0 29 Level twd`, both CPUs tick ~100 Hz, ping 20/20, no Oops/RCU stalls; blocker 6 is closed and the DTS/DTB/DTS-0001 were refreshed |
| Final devicetree deltas adopted (**zero blockers remain**; 2026-11-21) | MAC nvmem cells removed (userspace/vendor MAC provisioning), switch `reg`/`reg-names` MMIO-first order, `cudy,wr3000-v2`-first compatible + OpenWrt `SUPPORTED_DEVICES`, and the final flashinfo compatible pair `"hisilicon,luofu-flashinfo", "hsan,flashinfo_reserved"` (submodule commits `6c2cc2f`, `71b2167`; in-series binding patch 0021 reordered MMIO-first, review form rerun 26/26) | **boot-tested PASS**: MAC/switch/compatible deltas on image `c3ef2a50` (2 CPUs, `GIC-0 29 Level twd`, board_name `cudy,wr3000-v2`, ping 15/15); **flashinfo pair on image `6e9c54d0`** (stock loader `bootfip` staging PASS, `/proc/device-tree/reserved-memory/flashinfo@80600000/compatible` = new pair, 2 CPUs, cmdline `ubi.mtd=rootfsb`); **all documented upstream blockers are closed** - remaining review items are normal RFC content |
| Board features (LEDs / keys / watchdog) | stock board_id=3 overlay nodes in the board DTS (gpio-keys-polled reset GPIO22 / WPS GPIO23 active-low 20 ms; six `oem:green:*` LEDs GPIO24/25/3/2/0/1; target config `LEDS_GPIO` / `INPUT(+EVDEV/KEYBOARD)` / `KEYBOARD_GPIO_POLLED` / `WATCHDOG_SYSFS`, `INPUT_HISI_POWERKEY` off) | **verified on image `7f270b8c`**: six writable `/sys/class/leds/oem:green:*` devices, `/dev/input/event0` = gpio-keys-polled, `watchdog0` sysfs (identity HiSilicon luofu watchdog, timeout 30 s, not armed by us). **Physical checks (2026-11-21):** the user visually confirmed all six `oem:green:*` LEDs lit when driven to brightness 1 (set back to off); a RESET press was captured on `/dev/input/event0` as EV_KEY `KEY_RESTART` (0x198) press+release; both the RESET (`KEY_RESTART` 0x198) and WPS (`KEY_WPS_BUTTON` 0x211) presses were captured as EV_KEY press+release pairs |
| Wired LAN (native W22; mirror fallback) | the W22 path is release-integrated (patches 059/060; canonical cmdline `luofu_switch.naptinit=1`, no mirror/accepterr/cpucopy tokens) and host-validated (53/53) | **native eth2 delivery works without the mirror**: image `50023446` boots with the driver NAPT/flow-engine init (dmesg "W22: NAPT/flow engine initialized and verified"), 2 CPUs, ping 200/200 0% loss, SSH, reboot determinism PASS. M1a was first proven with image `5800c9ab` (userspace NAPT-init replay, native port=11 completions, status-77 class). The **CPU-copy mirror patches 054-057 remain in the line only as the documented fallback** (`naptinit=0` + mirror tokens), and the probe image `a91ebcf0` records the earlier negative source-port experiment. **Multi-port (per-port native delivery, LAN-to-LAN) is the next bench step (M1b) and is not claimed**; the fallback-image rebuild and the soak are pending too. Historical/planning docs: [docs/MULTI-PORT-PLAN.md](docs/MULTI-PORT-PLAN.md), [docs/incoming/lsw-forwarding/](docs/incoming/lsw-forwarding/) |
| Persistent overlay | two-volume squashfs + `rootfs_data` UBI layout in the image recipe; semantic UBI-volume verifier | `/dev/ubi0_1 on /overlay type ubifs (rw)`, files survive a reboot (canonical image) |
| Storage / FMC SPI-NAND | FMC driver + binding compile; contract documented | SPI-NAND (ESMT 128 MiB, 17 partitions) and UBI rootfs mounted on every boot |
| Install path (automated, bench-verified) | `tools/stock-telnet-install.sh` (fail-closed: `--inspect` read-only; `--apply` needs `--attended --recovery-armed`, writes only the **inactive** pair, verifies uploaded hashes, read-back slot hash, active-pair-unchanged and the selector) plus `tools/telnet-exec.py` | **verified end-to-end 2026-11-21** (last run: flashinfo-test image `6e9c54d0`): pair-b only, env-`bootflag`/`selector` agreement enforced, read-backs + semantic UBI verify + selector last, active pair unchanged, rebooted into our 6.18 image with 2 CPUs and the expected cmdline. Requires the vendor stock firmware and the notebook's bench-link/recovery helpers; **not a general install or recovery guide**; bench lessons: [docs/INSTALL-RECOVERY-LESSONS.md](docs/INSTALL-RECOVERY-LESSONS.md) |
| Upstream submission package (documentation only) | `upstream/series-6.18/submission/`: status/artifact checklist, one-page RFC cover-email draft, explicit no-Tested-by policy and a file manifest (`MANIFEST.sha256`, regenerated for this curated tree and verified `sha256sum -c`) | **nothing has been sent anywhere**; no account/list subscription/email transport; the package records **ZERO remaining blockers** (all five devicetree deltas and the flashinfo loader hand-off are boot-tested) plus the out-of-scope list |
| A/B install path | `platform_do_upgrade` runs against mocked MTD/sysfs/ubiformat: first-install **144/144**, A/B **100/100** (with the real built container); factory-MAC **103/103**; MAC-nvmem **77/77** (rewritten for the adopted 6.18 userspace MAC path after the nvmem-cell removal) | the fail-closed sequence was executed **manually** on 2026-11-19 and repeatedly with the automated stock-telnet installer on 2026-11-21 (pair-b only, env-bootflag/selector agreement enforced, read-backs, semantic UBI verify, selector last, active pair verified unchanged; handover fix `6e69aa3` after the mixed-pair diagnosis). `platform_do_upgrade` itself has never run on a device and the loader env re-stamp remains open; bench lessons: [docs/INSTALL-RECOVERY-LESSONS.md](docs/INSTALL-RECOVERY-LESSONS.md) |
| FIP trailer generator | reproduces both recorded vendor containers byte-for-byte; fail-closed on every decoded loader check; a novel generated trailer has never been booted | a FIP kernel slice assembled with this tooling booted from the inactive slot (2026-11-19); per-slot A/B trailer handling still [U] |
| Silicon inventory (2026-11-18) | first attended **read-only** inventory: 600/600 planned register/table reads OK (476 LSW + 108 PIE + 13 CRG + 3 SYSCTRL), 0 failed batches | no write, no reboot: one boot, read-only; no traffic/behaviour claim |
| CRG / watchdog / thermal / pinctrl / PCIe / restart (004-014, 019) | compile + binding checks pass; drivers are built-in in the target config | the bench boots registered FMC/SPI-NAND, UBI rootfs, restart and watchdog drivers (`watchdog0` sysfs verified, timeout 30 s, not armed by us); **thermal is unsupported** - the temperature register reads 0, the vendor DT disables the sensor and the zone returns `-ENODATA`; pinctrl/PCIe were not exercised; clock rates remain read-only |
| MDIO + PHY (007, 008) | complete driver ports, compile clean | bus/PHY never enumerated |
| Factory-MAC provisioning | userspace bdinfo derivation + validation (103/103 mocked); kernel-side read-only nvmem path (patch 026, host-only) | no MAC read from a real unit through the netdev path; the guarded LAN/WAN map is host-only. **MAC plan corrected:** eth0/eth1/eth2 sharing the LAN base MAC and eth3 = base+1 is the vendor rule (`board.d`/`99_fixwan`), not a defect - the earlier "duplicate MAC" blocker is withdrawn |
| Wi-Fi (Hi5622V100) | reverse-engineering/feasibility record only - the vendor stack is a closed cfg80211 full-MAC driver (not mac80211), no source and no mainline/OpenWrt 6.18 driver exists ([recon](target-skeleton/docs/incoming/wifi-recon/SUMMARY.md)) | vendor binary stack; **Wi-Fi is declared unsupported on the 6.18 target (option C)**; the vendor 5.10 kernel + modules remains the only Wi-Fi-capable system; GPL section 3 source request sent 2026-10-01, **no reply held** |
| **On-device validation** | - | **three bench phases**: 2026-11-19 first userspace boot (patches 001-017, console only); 2026-11-21 canonical `974e6ca6` (2 CPUs, bench LAN, SSH, ubifs overlay, 15-min soak 0 failures, reboot persistent); 2026-11-21 board-feature image `7f270b8c` (2 CPUs, six LEDs, `event0`, `watchdog0`, ping 0% loss) and quiet image **`7421bb7a`** (sha256 `7421bb7a...`; known-good baseline, periodic dmesg spam removed, byte-identical rebuild after the conservative DTS cleanup); automated stock-telnet install verified (2 CPUs after install); the superseded active-DTS image `6f19c6d6` did not boot (regression record); TWD level-high delta boot-tested on image `f6685b80` (2 CPUs, `GIC-0 29 Level twd`, ping 20/20); final MAC/switch/compatible deltas boot-tested on image `c3ef2a50` (2 CPUs, board_name `cudy,wr3000-v2`, ping 15/15); **flashinfo compatible-first boot-tested on image `6e9c54d0`** (stock loader staging PASS, new compatible pair in the DT, cmdline/rootfsb, 2 CPUs); E0-a mirror-off experiment negative (native completions error-flagged, IP dead - mirror workaround required); **E0-b read-only stock-vs-ours captures** (9,488 words each, 0 read failures; 30 candidates / 72 counter rows) led to the IFC->NAPT decode and the M1a proof; **W22 native forwarding landed** - canonical image `50023446` (2 CPUs, dmesg NAPT/flow engine initialized and verified, no mirror tokens, ping 200/200, SSH, reboot determinism PASS); M1a proof image `5800c9ab`; **M1b/fallback-image/soak pending** |

The one-line summary: the **release line is real, host-validated and bench-verified on one
unit**, but this is research engineering, not a firmware release: no install path has run on a
device, no native switch forwarding exists, no Wi-Fi driver exists, and nothing has been
upstream-reviewed.

---

## Bench-verified baselines (2026-11-21)

Image **974e6ca6** (`6,226,222 B`,
sha256 `974e6ca63f6e86418761bc0f135e438f07d0e6ef909c61bb0339f00c5596c508`): patches
001-061 + the loader hand-off DTS fix + canonical command line + DEVMEM/devmem/pair-recorder,
no probe tokens. Installed manually from the stock system and verified on the unit [V]:

- **2 CPUs**: `smp: Brought up 1 node, 2 CPUs`, CPU1 mpidr `80000001`,
  `3987.86 BogoMIPS`, `/proc/cpuinfo` = 2, online 0-1, SCU CONFIG bit 5 set;
- **network**: `br-lan` `192.168.1.1/24` up, host ping 20/20 0% loss (repeated
  0% loss runs), SSH login;
- **persistent overlay**: `/dev/ubi0_1 on /overlay type ubifs (rw)`, files survive a
  reboot;
- **stability**: 15-minute continuous-ping soak, 0 failures; no Oops/panic/BUG/WARNING (the
  only `panic` text is the literal `panic=0` cmdline);
- **persistent across reboot**: fresh boot, rootfsb, pair b recorded, 2 CPUs again.

### Board-feature and quiet baselines (2026-11-21)

The same 2-CPU/network/SSH/overlay line was extended with the stock **board_id=3** overlay
features and verified on the unit:

- **image `7f270b8c` (board features)** - six `oem:green:*` LEDs writable through
  `/sys/class/leds/*/brightness` (active-low map: status=GPIO24, wan=25, lan=3,
  internet=2, wifi=0, wifi5g=1); `event0` = `gpio-keys-polled` (reset GPIO22 /
  WPS GPIO23, active-low, 20 ms, `KEY_RESTART`/`KEY_WPS_BUTTON`); `watchdog0`
  via sysfs (identity HiSilicon luofu watchdog, timeout 30 s, state active - not armed by
  us); 2 CPUs online 0-1, ping 0% loss. **Physical checks (2026-11-21):** all six
  `oem:green:*` LEDs visually confirmed lit (then off), and both RESET and WPS presses
  captured on `event0` as EV_KEY press+release pairs (`KEY_RESTART` 0x198, `KEY_WPS_BUTTON` 0x211). [V]
- **image `7421bb7a` (quiet)** - patches 054/056 quieted from `dev_warn`/`dev_info`
  to `dev_dbg` and the heartbeat interval raised 5 s -> 60 s; measured at ~52 s uptime the
  quiet image has **471** dmesg lines with W17=0, W19=3 (boot only), HB=1 versus **876** lines
  with W17=162, W19=165, HB=61 on the previous image. Remaining pie-trace lines are boot-capped
  diagnostics; a clean boot log can now be captured for the remaining [U] items. [V]
- **thermal is unsupported on this board** - the tvsensor temperature register at
  `0x14900500+0x24` reads `0x00000000` (block otherwise zero except `+0x10=3`), the
  vendor formula turns raw 0 into -368000 mC (<= `THERMAL_TEMP_INVALID`) and the thermal
  core returns `-ENODATA`; the vendor DT disables the sensor and no enable/trim sequence is
  decoded. The driver/zone stays bound but reports no data; thermal telemetry is **not a
  release claim**. [U]/[X]

**Base-DTB padding is a build requirement (pad-dtb).** The new LED/keys DTS compiles to a
**13,041 B** base DTB whose FDT totalsize is not 8-byte aligned. The stock U-Boot `bootfip`
reads the overlay-container gate word as a 32-bit load at `base_dtb + totalsize`; the
unaligned load data-aborted (r1=`0x83200040`), reset the CPU and the loader fell back to
the stock pair. The image recipe therefore runs **`Build/luofu-pad-dtb`**
([target-skeleton/tools/pad-dtb.py](target-skeleton/tools/pad-dtb.py)) before `append-dtb`:
it pads to an 8-byte boundary, folds the padding into the FDT totalsize and asserts
`len(dtb) == totalsize` and `totalsize % 8 == 0`. Measured: 13,041 B -> **13,048 B**,
after which the image boots. Full measurement: [docs/TRAILER-FORMAT.md](docs/TRAILER-FORMAT.md)
section 11.

### Conservative DTS cleanup, known-good baseline and the regression record (2026-11-21)

- **Known-good hardware baseline: image `7421bb7a`** (sha256
  `7421bb7ace41e2e03613a0314874fabfcd16aeef60dbdd713ebc6c2b735cd4b3`): board features +
  quiet dmesg on the 2-CPU/SMP/network line; partition scan 0.93-1.16 s, `luofu-reboot`
  2.28 s, `luofu-wdt` 2.30 s, UBI attach 2.68 s, init 3.01 s.
- **Conservative board-DTS cleanup (this sync).** Only comments were removed: the file is now
  **691 lines**, and every active node name, unit address, property order/value, `reg` order,
  status, label, phandle, child name and alias is unchanged from the hardware-verified
  1,329-line pre-clean DTS. Proof: it compiles to the **byte-identical known-good DTB**
  (13,041 B sha256 `49fe5c47...`; padded by `Build/luofu-pad-dtb` to 13,048 B sha256
  `402e4f71...`, contract preserved), and the rebuilt image reproduces `7421bb7a`
  **byte-for-byte**. DTS-0001 is regenerated from it (sha256 `072d0b8b...`, 704 lines).
  [V]
- **Regression record - the superseded active-DTS cleanup did NOT boot.** The first cleanup
  (submodule commit `538377c`, image `6f19c6d6`, DTB 11,702 B) changed active
  devicetree content: the kernel came up with SMP but **MTD partition registration stalled**
  (17 prints stretched from ~0.9 s to ~24.55 s), `luofu-reboot`/`luofu-wdt` never
  probed, and the unit was reset by the U-Boot watchdog (~24.55 s) and fell back to stock pair
  A. It was reverted. Evidence log
  `2026-11-21-cleaned-dts-regression.log` sha256 `15dc3188...` (private lab
  evidence). [V]
- **Rule:** host-side checks (patch apply, dtc, contract values, DTS-0001 identity, pad-dtb)
  are **necessary but not sufficient** - every active-DTS change must be boot-tested on the
  unit. The 14 candidate active deltas (renames/removals/model/compatible) are listed in
  [07-dts-upstream-bindings.md](upstream/series-6.18/07-dts-upstream-bindings.md) section 3;
  none rides along with the prose cleanup.

### The SMP fix is DTS-only (release-critical)

The stock U-Boot `bootfip` hand-off staging is gated by two devicetree nodes that the
stock base DTB carries. They are now in the board DTS
([dts/luofu-r116.dts](target-skeleton/dts/luofu-r116.dts)):

    aliases { serial0 = &uart1; serial1 = &uart0; console = &uart1; };

    reserved-memory {
        #address-cells = <1>; #size-cells = <1>; ranges;
        flashinfo@80600000 {
            compatible = "hsan,flashinfo_reserved";
            reg = <0x80600000 0x4000>;
            atag-offset = <0x1000>;
        };
    };

Without them U-Boot printed `alias console not found`, skipped staging, and CPU1 never
polled the kernel pen - every kernel release variant was left with no consumer. The kernel
release implementation (patch 018) was correct all along; no SMP kernel code change is needed
for the fix. The contract ships as the explicit DTS-series entry **DTS-0001**
([upstream/series-6.18/dts-series/](upstream/series-6.18/dts-series/); documented in
[06-dts-series-dependency.md](upstream/series-6.18/06-dts-series-dependency.md)) and must be
posted with kernel patch 0018. Full root-cause record:
[docs/SMP-RE.md](docs/SMP-RE.md) sections 21-24.

**Retraction.** The 2026-11-20 report of a silent 2-CPU hang after ~23 minutes was a host
bench-link outage (USB-Ethernet carrier/IP loss), not a kernel hang: UART showed the box
alive with no Oops/panic/BUG/WARNING, and the same canonical image completed the verification
matrix and the 15-minute soak with 0 failures after the link was restored. There is no SMP
stability bug. The `maxcpus=1` and lockdep images built for the false hang are
diagnostics only.

### TWD timer interrupt encoding adopted (upstream blocker 6 closed, 2026-11-21)

The board DTS now uses the **standard Cortex-A9 TWD encoding** for the local timer:

    interrupts = <GIC_PPI 13 (GIC_CPU_MASK_SIMPLE(2) | IRQ_TYPE_LEVEL_HIGH)>;   /* 0x304 */

with the `arm-gic.h` / `irq.h` includes (previously the measured vendor form
`<1 13 0x301>`, edge-rising). Measured hardware fact [V]: the GIC-400 PPI configuration is
read-only/SoC-fixed and the measured GICD_ICFGR1 field for PPI13/ID29 is `0b11`, so the DT flag
does not reprogram hardware; the standard LEVEL_HIGH form is what upstream platforms use.
**Boot test PASS on image `f6685b80`**: 2 CPUs, `/proc/interrupts` reports
`GIC-0 29 Level twd`, both CPUs tick at ~100 Hz, ping 20/20 0% loss, no Oops/BUG/panic/RCU
stall (the previously verified baseline `7421bb7a` remains recorded). The TWD adoption
refreshed the current DTS (sha256 `13427d8e...`, 700 lines), **DTS-0001**
(`e9c9ed58...`, 713 lines), the DTB (`4fde76df...` / pad `4b50de73...`) and the upstream
validation records; at that point **five upstream blockers remained** - all are now closed (zero
remain, see below). Every further active-DTS delta still needs its own boot-test.

### Final devicetree deltas, zero remaining blockers, and the E0-b result (2026-11-21)

Five active-devicetree deltas were boot-tested on the unit and adopted:

- **MAC-nvmem removal** - the `fixed-layout`/`mac-base` bdinfo cells are removed; 6.18 uses the
  userspace/vendor MAC provisioning path instead;
- **switch MMIO-first `reg`/`reg-names`** - the 15 MMIO windows first, the two DDR regions last;
  the in-series binding (source patch and review-form patch 0021) was reordered to match and the
  review form was rerun clean (26/26 `patch(1)` + `git am`, checkpatch 0 errors / 58 warnings);
- **`cudy,wr3000-v2` first compatible + OpenWrt `SUPPORTED_DEVICES`** - `board_name` and sysupgrade
  metadata match the upstream compatible order;
- **TWD level-high** - see the section above (image `f6685b80`);
- **flashinfo compatible pair** - `compatible = "hisilicon,luofu-flashinfo", "hsan,flashinfo_reserved"`;
  the documented upstream string is first and the vendor string is the fallback.

**Boot-test PASS on image `c3ef2a50`** (MAC/switch/compatible deltas; sha256
`c3ef2a507e6e0fc3d8991139b9668eeccae8f59995db4d17eaa5e71f68ea5879`, installed with the verified telnet
installer on pair b, selector `0x11`): 2 CPUs, `/proc/interrupts` `GIC-0 29 Level twd`;
`/tmp/sysinfo/board_name = cudy,wr3000-v2`; `/proc/device-tree/compatible` starts with
`cudy,wr3000-v2`; eth0-eth2 MAC `...:73`, eth3 `...:74`; br-lan up, ping 15/15 0% loss; no
Oops/BUG/panic/RCU stall. [V]

**Last-blocker boot-test PASS on image `6e9c54d0`** (flashinfo compatible-first): the stock loader
accepted the new first compatible (U-Boot `bootfip` staging lines with
`alias console=/serial@1010f000`, `atag_addr=0x80601000`; `bootflag b bootreg 11`), the image
booted 6.18.60 with 2 CPUs and `GIC-0 29 Level twd`, `/proc/cmdline` contains `ubi.mtd=rootfsb` plus
the full `luofu_switch` marker, and `/proc/device-tree/reserved-memory/flashinfo@80600000/compatible`
reads `hisilicon,luofu-flashinfo hsan,flashinfo_reserved`. **This closes the last documented upstream
blocker: zero blockers remain.** The drafted reserved-memory binding and the driver/YAML review
items are now maintainer-facing RFC material, not blocker tasks. [V]

Current artifacts: DTS **681 lines** sha256 `381b2e8c...`; **DTS-0001** `886f3ad0...` (694 lines);
DTB **12,678 B sha256 `62574b11...`** (pad **12,680 B sha256 `8d79794e...`**); release-line
manifest sha256 `1e00b568...`. The DTS-purpose hardware baseline is **`6e9c54d0`** (previous
`c3ef2a50`; earlier `f6685b80` and `7421bb7a`). Any further active-DTS change still needs its
own boot-test; host checks cannot substitute for it.

**E0-a native-delivery result (2026-11-21).** With the CPU mirror disabled (`cpu_mirror=0`) the
unit still boots 2 CPUs and native-path completions arrive on port 11, but **every frame is
`status=37 err=1`** and the IP path is dead. The `status=37` class is an exception/punt marker, not
a corrupt frame: the exception path works, but only ARP-class traffic is punted - the missing
piece is unicast-to-CPU delivery. The **CPU-copy mirror workaround therefore remains required**,
and native forwarding is a separate, planned project. [V]

**E0-b stock-vs-ours runtime capture (2026-11-21, read-only).** Both sides were captured with
`capture-runtime-state.sh --profile punt` (9,488 words each, 0 read failures) on a consistent
runtime: ours = image `6e9c54d0`; stock = vendor 5.10.201 on pair a, reached by env-bootflag +
selector handover, then returned. The diff (30 candidates, 72 counter rows) found the stock-only
state our init lacks: **tagact entry 1** at `0x124d4010` (w1=`0x0000c000`) as the strongest
CPU-punt/classification candidate, **`pfe_glb+0x2c` high16 `0x1000`** as an exactly justified
write, plus EQM queue config and extensive NAPT/L2/PIE differences. NAPT and L2 stay refused by
name (DDR-pointer/contract hazards); the planned M1a step is a reviewed, fail-closed write of
only the justified words with the mirror kept for management, then a host ping looking for a
native 98-byte completion - never a blind replay. Evidence is recorded in the retained plan
(private capture tarballs are not published). [V]

### Install and recovery lessons (bench-verified 2026-11-21)

The stock telnet installer was verified end-to-end again on the flashinfo-test image `6e9c54d0`:
pair-b only, env-bootflag/selector agreement enforced, read-backs + semantic UBI verify +
selector-last, and the booted cmdline required to contain `ubi.mtd=rootfsb` + the project marker.
Lessons recorded: **set the U-Boot env `bootflag` together with the selector cell** (the loader reads
the kernel slot named by the env; a cell-only handover after a RESET-hold recovery can pair our
kernel with the stock rootfs); **our image is pair-b-only** (baked `ubi.mtd=rootfsb`); the broker
channel (CVE-2026-71961) only works on a consistent vendor pair (vendor
`/proc/sys/dev/flash_uuid`), and the mixed our-kernel/stock-rootfs state disables the vendor wizard
and cmagent; **recovery is physical** RESET-hold + TFTP stock image. Full record:
[docs/INSTALL-RECOVERY-LESSONS.md](docs/INSTALL-RECOVERY-LESSONS.md). [V]

### W22 native forwarding (release line, bench-verified 2026-11-21)

The native data path is now in the release line itself (host-validated + bench-verified):

- **Patch 059 - PFE NAPT/flow-engine init** (`059-net-lsw-napt-init.patch`, sha256
  `cfd7debc...`): the driver initializes the PFE NAPT/flow engine (the vendor
  `hi_pfe_napt_init` sequence with the stock-captured final values) behind
  `luofu_switch.naptinit=1` (default on). It verifies itself and logs `W22: NAPT/flow engine
  initialized and verified`; unicast traffic is punted to the CPU instead of being dropped in an
  uninitialized NAPT.
- **Patch 060 - PIE native exception-class delivery** (`060-net-pie-exception-delivery.patch`,
  sha256 `8c748061...`): delivers the native CPU-delivery completion class by default and
  retires the experimental `luofu_pie.accepterr` gate.
- **Canonical cmdline**: `luofu_switch.naptinit=1` and **no** `cpu_mirror` / `mirror_dst` /
  `accepterr` / `cpucopy_*` tokens. The CPU-copy mirror patches **054-057 remain in the line only
  as the documented fallback** rebuild (`naptinit=0` + mirror tokens).
- **Bench evidence:** canonical image **`50023446`** (sha256 `500234462bdc...`) boots 6.18.60,
  2 CPUs, `GIC-0 29 Level twd`, no Oops/BUG/panic; dmesg shows the W22 NAPT init verified;
  host ping 200/200 0% loss and SSH work with no mirror tokens; reboot determinism PASS. M1a was
  first proven with proof image **`5800c9ab`** (userspace NAPT-init replay, mirror off: native
  port=11 deliveries in the status-77 class, ping 200/200, SSH). [V]

**Scope and pending steps.** W22 is **release-line only - it is not in the L2 upstream review form**
(the review form is unchanged at 26 patches). Multi-port (per-port native delivery, LAN-to-LAN) is
the next bench experiment (**M1b**, plan in the retained forwarding docs); the fallback-image
rebuild and the soak are pending as well. Nothing multi-port is claimed. [I]

### Canonical boot configuration

Built-in cmdline (`CONFIG_CMDLINE_FORCE`, `luofu/config-6.18`):

    ubi.mtd=rootfsb ubi.block=0,0 root=/dev/ubiblock0_0 rootfstype=squashfs
    rootwait console=ttyS0,115200 earlycon noinitrd cma=0 maxcpus=2 nr_cpus=2
    additional_cpus=1 luofu_switch.arm=1 luofu_switch.egress=1
    luofu_switch.qidmap=1 luofu_switch.eps=1 luofu_switch.qminit=1 luofu_switch.pfe=1
    luofu_switch.naptinit=1 luofu_switch.ifc=1 luofu_switch.ext=1
    luofu_pie.arm=1 luofu_pie.rxcfg=1 luofu_gmac.macregs=1 oops=panic panic=0
    # W22 fallback image only: naptinit=0 plus
    # luofu_switch.cpu_mirror=1 mirror_dst=4 luofu_pie.cpucopy_deliver=1 cpucopy_port=11

Since the board-feature pass the canonical attended module gates and `oops=panic panic=0`
are **baked into `CONFIG_CMDLINE`** (the `974e6ca6` baseline predates that and passed the
same gates as attended parameters; the `7f270b8c`/`7421bb7a` baselines embed them). The config also
sets `CONFIG_DEVMEM=y` / `CONFIG_STRICT_DEVMEM=n` and the board-feature symbols
(`LEDS_GPIO`, `INPUT`/`INPUT_EVDEV`/`INPUT_KEYBOARD`, `KEYBOARD_GPIO_POLLED`,
`WATCHDOG_SYSFS`; `INPUT_HISI_POWERKEY` off). No `luofu_smp_*` tokens are used.
The diagnostic readout gates (`rxdiag`, `rxstroke`, `rxtrace`, `dump`, `scan`,
`pagedump`) are optional and not canonical.

---

## The attended bench work

### 2026-11-19: first boot to userspace (console only, patches 001-017)

**Result: SUCCESS - first full boot of our own OS on this hardware.** An attended bench
session (recovery door armed; console captured read-only on UART; no host-to-router serial
input) booted the inactive pair written by the manual install path: stock U-Boot failover ->
`Image Name: Linux-6.18.60` -> our kernel banner -> UBI `mtd14` (`rootfsb`) ->
`squashfs` root read-only -> `/sbin/init` -> procd -> kmodloader -> tmpfs
overlay fallback -> userspace console. The image was built from patches **001-017 only**
(UP, 1 CPU, no network, no persistent overlay) and was the debug-CRC build; a clean rebuild
(`e893c388...`) was never re-flashed.

**The five fixes that turned the first kernel banner into userspace** (all in the target tree
now):

1. **Built-in command line** - the stock loader supplies **no** kernel command line for our
   uImage (empty cmdline -> `VFS: Cannot open root device` -> panic); a
   `/chosen/bootargs` property made U-Boot data-abort before hand-off. The working fix
   is `CONFIG_CMDLINE` + `CONFIG_CMDLINE_FORCE` in `luofu/config-6.18`.
2. **4 KiB squashfs padding** - `ubiblock` rounds a static UBI volume down to a
   512-byte boundary, so an unpadded squashfs loses its tail. The image recipe pads to 4096 B
   before `ubinize`.
3. **`CONFIG_XZ_DEC_ARM=y`** - the OpenWrt squashfs uses XZ with the ARM BCJ filter;
   without the arm decoder every compressed file block failed although the raw FMC/NAND reads
   were byte-perfect.
4. **Semantic UBI-volume verification** - `ubiformat` rewrites the EC-header
   `image_seq` (and its CRC) on every PEB, so a raw full-partition hash can never match;
   the verifier is [target-skeleton/tools/ubi-volume-check.py](target-skeleton/tools/ubi-volume-check.py).
5. **Single-write selector** - the SYSENV `boot_reg` sysfs attribute must be written
   once (`echo -n "0x11" > boot_reg`); a tty-split `printf` wrote partial values.

### 2026-11-20 to 2026-11-21: connectivity then SMP

- **Working data path + SSH (2026-11-20).** Image `39424b94` (001-057) passed ping at
  0% loss and SSH with a persistent UBIFS overlay: the working path is the attended
  **CPU-copy workaround** (static mirror dst=4 + port-0 CPU-copy delivery to the mirror source
  netdev). This is the first fully working LAN on our own kernel/rootfs; native CFE L2/VLAN
  forwarding was later closed as a separate project (it is loader/DDR-carveout state, not
  MMIO-replayable - see [docs/LSW-SPEC.md](docs/LSW-SPEC.md)).
- **Canonical 001-058 image (2026-11-20).** `af777742`: full W3-W20 chain, ping 30/30
  0% loss, SSH, persistent overlay, heartbeat stable.
- **Reboot fallback fixed.** A reboot from an early working image fell back to the vendor
  slot A because our userspace did not maintain the loader selector; the boot-pair recorder
  init script now re-records the running pair, and the canonical image reboots with 2 CPUs
  and pair b recorded.
- **SMP investigation to DTS root cause (2026-11-20/21).** Probes 059-061 showed CPU1 never
  entered any kernel-side stub; the loader-side cause was decoded and fixed by adding the two
  stock DTB nodes. First 2-CPU image `d61c09e6`; canonical post-fix image
  `974e6ca6` passed the soak and reboot checks (the false hang is retracted above).

### 2026-11-21: board features, the DTS padding bug and the quiet image

- **Stock board_id=3 features (image `7f270b8c`).** The same single-source DTS gained the
  vendor overlay's gpio-keys-polled (reset GPIO22 / WPS GPIO23, active-low, 20 ms) and six
  `oem:green:*` gpio-leds (GPIO24/25/3/2/0/1); the target config enabled
  `LEDS_GPIO`, the input stack and `WATCHDOG_SYSFS`. On hardware this gives six writable
  LEDs, `event0` and `watchdog0` sysfs, with 2 CPUs and ping 0% loss. [V]
- **Un-aligned base-DTB bug found and fixed.** The new nodes pushed the compiled base DTB to
  13,041 B with an unaligned FDT totalsize; the stock U-Boot `bootfip` gate-word load
  data-aborted and the loader fell back to the stock pair. The fix is the image-build step
  `Build/luofu-pad-dtb` (`tools/pad-dtb.py`), which pads the DTB to 13,048 B / an 8-byte
  totalsize and folds the padding into the FDT total; the image then boots. [V]
- **Quiet image `7421bb7a`.** Patches 054/056 were quieted to `dev_dbg` and the
  heartbeat interval raised 5 s -> 60 s; the periodic W17/W19/HB dmesg spam dropped from
  876 to 471 lines at ~52 s uptime (W17 162 -> 0, W19 165 -> 3 boot-only, HB 61 -> 1) so a
  clean boot log can be captured for the remaining [U] items. [V]
- **Thermal root-caused as unsupported.** The tvsensor temp register at
  `0x14900500+0x24` reads 0, the vendor formula maps raw 0 to an invalid temperature and
  the thermal core returns `-ENODATA`; the vendor DT disables the sensor and no enable/trim
  sequence is decoded. Decision: record thermal as unsupported, not a bring-up target. [U]/[X]
- **MAC plan withdrawn as a blocker.** eth0/eth1/eth2 sharing the LAN base MAC and eth3 =
  base+1 is the vendor rule (`board.d` / `99_fixwan`), not a defect; the feature
  inventory's duplicate-MAC concern is withdrawn (curated note in
  [the inventory](target-skeleton/docs/incoming/feature-inventory/SUMMARY.md)). [V]
- **Upstream refresh.** DTS-0001 was regenerated for the new board nodes (sha256
  `8ee4b8e71...`; 1,329-line board DTS), 054/056 hashes were refreshed (manifest sha256
  `4da8b960...`) and the validator now exercises the pad-dtb step; the review form stays
  26 patches with 0 errors / 58 warnings. [V]

### Where the bench work is recorded

The console/soak/reboot logs are private lab evidence under the notebook's gitignored
`build/` tree (for example `2026-11-21-soak-2cpu.log`,
`2026-11-21-final-reboot-2cpu.log`); they are **not** published. The prose record is in
[target-skeleton/RELEASE-NOTES.md](target-skeleton/RELEASE-NOTES.md),
[docs/SKELETON-SMOKE.md](docs/SKELETON-SMOKE.md) and
[target-skeleton/docs/port-notes/smp-decode/SUMMARY.md](target-skeleton/docs/port-notes/smp-decode/SUMMARY.md).

---

## Images: canonical baseline and earlier builds

| image | stack | size | status |
| --- | --- | --- | --- |
| **`974e6ca6`** (pre-W22 canonical) | L0 pre-W22 (001-058 + 062-065) + loader hand-off DTS fix + pre-W22 canonical cmdline | 6,226,222 B | **bench-verified 2-CPU baseline** on 2026-11-21 (network via the bench workaround, SSH, overlay, soak, reboot) |
| **`7f270b8c`** (board features) | L0 + DTS fix + stock board_id=3 LED/keys nodes + pad-dtb + board config | - | bench-verified: 2 CPUs, six LEDs, `event0`, `watchdog0`, ping 0% loss (2026-11-21) |
| **`7421bb7a`** (known-good, current) | `7f270b8c` line with W17/W19 `dev_dbg` and heartbeat 5 s -> 60 s | sha256 `7421bb7a...` | **bench-verified known-good baseline** (2026-11-21): same features, periodic dmesg spam removed; reproduced byte-for-byte by the conservative comment-only DTS rebuild |
| **`6e9c54d0`** (DTS-purpose, current hardware baseline) | `7421bb7a` userspace/kernel + all five boot-tested DTS deltas (TWD level-high, MAC-nvmem removal, switch MMIO-first, cudy-first, flashinfo compatible pair; DTB `62574b11...` / pad `8d79794e...`) | - | **boot-tested** (2026-11-21): stock loader `bootfip` staging PASS with the new first compatible; 2 CPUs, `GIC-0 29 Level twd`, cmdline `ubi.mtd=rootfsb` + project marker, zero blockers remain; current DTS-purpose baseline |
| **`50023446`** (canonical W22) | release line 53 patches incl. W22 059/060 + canonical `naptinit=1` cmdline (no mirror tokens) | sha256 `50023446...` | **bench-verified canonical W22 image** (2026-11-21): 2 CPUs, dmesg `W22: NAPT/flow engine initialized and verified`, native eth2 CPU delivery, ping 200/200 0% loss, SSH, reboot determinism PASS; mirror fallback image documented |
| **`5800c9ab`** (M1a proof) | adopted release tree + cmdline `cpu_mirror=0 luofu_napt_init=1` + probe-only NAPT-init replay | sha256 `5800c9ab...` | **M1a PROVEN** (2026-11-21): native port=11 deliveries in the status-77 class, ping 200/200, SSH; the userspace proof that led to the W22 kernel patches |
| **`c3ef2a50`** (DTS-purpose, previous baseline) | `7421bb7a` userspace/kernel + final boot-tested DTS deltas (TWD level-high, MAC-nvmem removal, switch MMIO-first, cudy-first; DTB `a0af2899...` / pad `01b74561...`) | sha256 `c3ef2a50...` | **boot-tested** (2026-11-21): 2 CPUs, `GIC-0 29 Level twd`, `board_name=cudy,wr3000-v2`, MACs ...:73/...:74, br-lan up + ping 15/15 0% loss, no errors; superseded for DTS purposes by `6e9c54d0` |
| **`f6685b80`** (DTS-purpose, earlier TWD baseline) | `7421bb7a` userspace/kernel + boot-tested TWD level-high DTS delta (DTB `4fde76df...` / pad `4b50de73...`) | - | **boot-tested** (2026-11-21): 2 CPUs, `/proc/interrupts` `GIC-0 29 Level twd`, ~100 Hz counts on both CPUs, ping 20/20, no errors; superseded for DTS purposes by `c3ef2a50` (7421bb7a remains the earlier verified baseline) |
| `6f19c6d6` (reverted) | aggressive active-DTS cleanup | DTB 11,702 B | **did not boot** - MTD partition registration stalled and the U-Boot watchdog reset the unit, stock fallback; reverted, see the regression record |
| `af777742` | 001-058 canonical stack | - | bench-verified LAN + SSH + overlay (2026-11-20), before the SMP fix |
| `d61c09e6` | first DTS-fix 2-CPU image | - | first 2 CPUs; superseded by `974e6ca6` |
| `39424b94` | 001-057 | - | first working LAN + SSH (2026-11-20) |
| `26dde6fe...` | 001-026 full-series host rebuild | 6,205,742 B | **host-only, never flashed or booted** (superseded) |
| `15dae772...` | 001-023 overlay-volume rebuild | 6,205,742 B | host-only, never flashed or booted (superseded) |
| `e893c388...` | clean 001-017 rebuild after the first boot | 6,185,262 B | host-only, never re-flashed (superseded) |
| `51baf2ac...` | 001-017 debug-CRC build | - | the image that booted on 2026-11-19 (history) |

The `26dde6fe` / `15dae772` builds belong to the earlier host-only generation (the
repository's previous curated snapshot); they were never flashed and are kept in the history of
this page only for provenance. The previous verified bench baseline is **`7421bb7a`** (board features + quiet dmesg on the
`974e6ca6` 2-CPU/SMP/network line), reproduced byte-for-byte by the conservative
comment-only DTS rebuild; the current **DTS-purpose hardware baseline is `6e9c54d0`**, which
adopts all five boot-tested devicetree deltas, and the **canonical W22 data-path image is
`50023446`** (native CPU delivery, no mirror tokens; M1a proof image `5800c9ab`);
`c3ef2a50` was the previous DTS-purpose baseline, `f6685b80` the earlier TWD baseline and
`7421bb7a` the original verified baseline.

> **The install path is verified on the bench, but this is not a general install guide.**
> The stock-mediated install was executed manually on 2026-11-19 and again automatically on
> 2026-11-21 with [`tools/stock-telnet-install.sh`](tools/stock-telnet-install.sh) (inactive
> pair only, active pair verified unchanged, selector verified, 2-CPU image booted). The tool
> still requires the vendor stock firmware and the notebook's bench-link/recovery helpers, and
> `platform_do_upgrade` itself has never run on a device; the loader environment re-stamp
> and per-slot A/B trailer handling remain open. A **new/active DTS change** must be boot-tested
> before being trusted (see the regression record). Nothing in this repository is a recovery
> procedure; do not flash it.

---

## The Linux 6.18 kernel work: release line (53) and review form (26)

The target tree carries **64 plain patch files** (`001-060` + `062-065`, the L0
stack), and the repository publishes two artefacts built from the same sources:

1. **The release line** - 53 include patches (incl. W22 patches **059** NAPT/flow-engine init and **060** PIE native exception-class delivery), classified in
   [`upstream/series-6.18/release-line/manifest.tsv`](upstream/series-6.18/release-line/manifest.tsv)
   with one row per patch (include/exclude, class A/B/C/D, sha256, purpose). 11 bench-only SMP
   diagnostics are excluded: `028`, `029`, `033`, `034`,
   `035`, `037`, `039`, `062`, `063`, `064`,
   `065`. They stay in the tree so the canonical image is reproducible, but a release
   or upstream series must not carry them as the SMP fix. The release line is validated
   host-only by
   [`tools/check-release-line.sh`](upstream/series-6.18/tools/check-release-line.sh)
   (53/53 `patch(1)` RC=0, strict `git apply` clean, loader hand-off DTS contract
   compiled and checked; raw log in
   [`validation/release-line/`](upstream/series-6.18/validation/release-line/)). The
   overview is [05-release-line.md](upstream/series-6.18/05-release-line.md). The 2026-11-21
   board-feature pass refreshed the 054/056 hashes (`dev_warn`/`dev_info` -> `dev_dbg`;
   manifest sha256 `619eb15a...`) and the validator also exercises the base-DTB
   `pad-dtb` step (13,041 B -> 13,048 B).
2. **The upstream review form** - the 26 kernel patches 001-026 in commit-ready
   `format-patch` style, with cover letter, per-patch messages, MAINTAINERS proposal,
   tree/order rationale and repeatable validation. The readable overview is its
   [README](upstream/series-6.18/README.md).

The review form is not a merge list: `020`-`026` are included as **draft /
do-not-merge** patches documenting the measured-inventory, disassembly and connectivity
follow-ups. `020` is an arm-gated PIE datapath draft; `021` is a read-only LSW
probe; `022` implements the first reviewed switch writes behind explicit attended gates;
`023` makes the GEMAC netdev conditional on the PIE engine and the verified egress map;
`024` decodes the recovered v250 RX completion and full normal non-TSO TX descriptor;
`025` adds the gated W4 egress/probe-configuration write stage; `026` exposes the
factory MAC through a read-only bdinfo nvmem cell. The ready-for-review split and the
do-not-merge reasoning are in the series
[README](upstream/series-6.18/README.md) and
[01-commit-messages.md](upstream/series-6.18/01-commit-messages.md).

| # | Patch | Contents | Upstream intent |
| --- | --- | --- | --- |
| 0001 | ARM: mach-hisilicon: add Hi5671Y "luofu" machine support | `arch/arm/mach-hisilicon/` (Kconfig, Makefile, luofu.c), `ARCH_LUOFU` under `ARCH_HISI`, UP-only DT machine | RFC |
| 0002 | ARM: luofu: add ARCH_LUOFU to Kconfig and set its text offset | `arch/arm/Kconfig`, `arch/arm/Makefile`, `textofs := 0x00608000` | RFC |
| 0003 | ARM: head: materialize TEXT_OFFSET with `mov_l` | makes the non-immediate `0x00608000` offset assemble in `head.S` / compressed head | RFC |
| 0004 | clk: hisilicon: add Luofu CRG clock and reset controller | 16 gate clocks + 2-cell reset controller over one syscon regmap; DT binding + clock-ID header | ready for review |
| 0005 | watchdog: luofu: add Hi5671Y watchdog support | devm watchdog, child of the CRG syscon node | ready for review |
| 0006 | spi: spi-luofu-fmc: add Luofu FMC SPI-NAND controller | SPI-MEM controller; mainline SPI-NAND stack owns the chip/ECC/partitions | RFC |
| 0007 | net: mdio: luofu: add MDIO bus controller | mainline MDIO-bus API | ready for review |
| 0008 | net: phy: add driver for HiSilicon Luofu "Genius" PHYs | ID-matched PHY driver (no binding) | ready for review |
| 0009 | net: ethernet: hisilicon: add partial Luofu GEMAC driver | binding + resource half; **no TX/RX ring DMA data path**, refuses to probe | do not merge |
| 0010 | power: reset: add Luofu machine restart driver | restart over SYSCTRL/CRG syscons; reset command is provisional | RFC / design |
| 0011 | pinctrl: hisilicon: add Luofu peripheral pin controller | pinmux/pinconf from decoded vendor tables | RFC |
| 0012 | thermal: add Luofu thermal sensor driver | temperature sensor + thermal zone | ready for review |
| 0013 | PCI: dwc: add Luofu PCIe host controller | DWC host draft; link/PHY bring-up unvalidated | RFC |
| 0014 | clk: hisilicon: fix Luofu CRG reset polarity | polarity correction for 0004 (to be folded in) | ready for review |
| 0015 | net: ethernet: hisilicon: add Luofu PIE packet engine draft | register inventory only; refuses to arm the engine | do not merge |
| 0016 | net: ethernet: hisilicon: add Luofu switch probe skeleton | binding/resource/port-model validation; refuses to program the switch | do not merge |
| 0017 | soc: hisilicon: add Luofu A/B boot selector | `current_pair`/`next_pair` sysfs over the shared SYSCTRL cell; loader-style low-byte store + read-back | RFC |
| 0018 | ARM: mach-hisilicon: add Hi5671Y SMP bring-up support | `CPU_METHOD_OF_DECLARE(luofu_smp)`, SCU enable, fail-safe missing-node path; **necessary but not sufficient - the board DTS must carry the loader hand-off nodes (DTS-0001)**; the commit message records the dependency | RFC; bench-verified on the target image once DTS-0001 lands |
| 0019 | clk: hisilicon: add the Luofu CRG clock tree | gate parents on real input clocks; two PLLs + two muxes as **read-only** clocks; seven required input clocks | ready for review |
| 0020 | net: ethernet: hisilicon: add the Luofu PIE datapath and GEMAC front end | extends `015`'s PIE driver into a draft ring engine and `009`'s GEMAC into a draft phylink/netdev front end; the data path arms only behind the kernel-command-line opt-in `luofu_pie.arm=1` | review-form draft; do not merge |
| 0021 | net: ethernet: hisilicon: add the Luofu switch read-only baseline probe | extends `016` to the measured 17-window model, powered-state gate, 30-word W1 baseline and both qid maps; contains no `writel()`, registers no netdev, always fails closed | review-form draft; do not merge |
| 0022 | net: ethernet: hisilicon: add the gated Luofu LSW write path | attended W3/W4/W5 steps behind `luofu_switch.arm`/`qidmap`: gate clocks + resets with a read-only snapshot, then the reviewed W2 qid-map write set with readback and restore-on-mismatch | review-form draft; do not merge |
| 0023 | net: ethernet: hisilicon: gate the Luofu GEMAC netdev on the LSW egress map | registers the GEMAC netdev only when `luofu_pie_armed()` **and** `luofu_switch_ready()`; `PHYLINK_NETDEV` carrier fix; DT MAC address source | review-form draft; do not merge |
| 0024 | net: ethernet: hisilicon: decode Luofu PIE RX completions and TX descriptors | decoded v250 RX completion, per-port delivery and the full normal non-TSO TX descriptor, all inside the never-passed `luofu_pie.arm=1` path | review-form draft (Group 4 RFC); do not merge |
| 0025 | net: ethernet: hisilicon: add the gated Luofu LSW egress write stage | third attended switch stage `luofu_switch.egress=1`: read-only conformance then the vendor R1/R3/R5 probe-configuration words with expectations from the pre-write snapshot and restore-on-mismatch | review-form draft (Group 4 RFC); do not merge |
| 0026 | net: ethernet: hisilicon: use the bdinfo MAC in the Luofu GEMAC netdev | read-only bdinfo MTD nvmem cell (`fixed-layout`/`mac-base`, LAN = base, WAN = base+1), `-EPROBE_DEFER` handling and `ndo_set_mac_address` compatibility | review-form draft (Group 4 RFC); do not merge |

Series facts (all host-only; evidence in
[upstream/series-6.18/validation/](upstream/series-6.18/validation/)):

- 26/26 apply in order to pristine `linux-6.18.60` with `patch(1)` and with
  `git am` (identical trees, clean work tree, 3 offset hunks, 0 fuzz, 0 rejects).
- `checkpatch.pl --no-tree`: **0 errors, 58 warnings** (default and strict), plus 6
  strict CHECKs - review items such as binding split, MAINTAINERS entry and line wrapping,
  not behavioural defects.
- `sha256sum -c upstream/series-6.18/validation/sha256sums.txt` from the series
  directory: **89/89 OK** (26 review-form patches + 64 L0 target patches + DTS-0001; hashes
  refreshed for 054/056 on 2026-11-21).
- The 2026-11-21 board-feature refresh quieted patches 054/056 (`dev_dbg`; manifest sha256
  `4da8b960...`) and refreshed DTS-0001; the review-form content itself is unchanged
  (26/26 apply, 0 errors / 58 warnings).
- The final 001-026 series compiled in an isolated `openwrt main` clone:
  `make target/linux/prepare` RC=0 and `make target/linux/compile` RC=0 with
  **0 errors / 0 compiler warnings** under `CONFIG_WERROR=y`; zImage 3,282,528 B,
  sha256 `08b7674749a0c155285a89fd35fa2433c2c97ce48ee5a1773cb3541365544c67`.
- **The bench-verified image is the release line, not the review form.** The canonical
  `974e6ca6` image (2026-11-21) is patches 001-061 + the DTS fix; the review form stays a
  review artefact of the 26 kernel patches and has no upstream review or Tested-by.
- **No submission has been made.** The "ready-for-review" split, the Group-4 RFC hold and the
  owner decisions still required (real `Signed-off-by` identity, binding split, folding
  0014 into 0004, DTS bindings cleanup) are listed in
  [upstream/series-6.18/README.md](upstream/series-6.18/README.md).

**DTS-0001 (the upstream DTS series dependency).** The kernel review form cannot carry the
board DTS, so the loader hand-off nodes are published as the explicit DTS-series entry
DTS-0001,
[`dts-series/0001-arm-dts-hisilicon-luofu-r116-loader-handoff.patch`](upstream/series-6.18/dts-series/0001-arm-dts-hisilicon-luofu-r116-loader-handoff.patch)
(sha256 `886f3ad0...`, 694 lines; current final 2026-11-21: conservative
comment-only cleanup + the five boot-tested deltas - TWD level-high, MAC-nvmem removal, switch
MMIO-first `reg`/`reg-names`, `cudy,wr3000-v2` first compatible, flashinfo compatible pair).
It adds `arch/arm/boot/dts/hisilicon/luofu-r116.dts` (681 lines; the conservative step left the
same active nodes as the hardware-verified 1,329-line file, and each adopted active delta was
boot-tested: TWD on `f6685b80`, MAC/switch/compatible on `c3ef2a50`, flashinfo on `6e9c54d0`)
and the `dtb-$(CONFIG_ARCH_LUOFU)` Makefile entry; it is the companion of kernel patch 0018 and
is documented in [06-dts-series-dependency.md](upstream/series-6.18/06-dts-series-dependency.md).
The DTS series must also state the **base-DTB padding requirement** (the compiled DTB is 12,678 B
with an unaligned totalsize, so the image build must run `Build/luofu-pad-dtb`; see
[TRAILER-FORMAT.md](docs/TRAILER-FORMAT.md) section 11). Current compiled DTB
`62574b11...` (pad `8d79794e...`). The loader hand-off contract **now expects the adopted
compatible pair** `hisilicon,luofu-flashinfo hsan,flashinfo_reserved` (the validator was updated
accordingly). Every active delta has its own hardware-tested patch; the superseded aggressive
cleanup is recorded as a boot regression in
[07-dts-upstream-bindings.md](upstream/series-6.18/07-dts-upstream-bindings.md).

**Submission package (documentation only).** [`upstream/series-6.18/submission/`](upstream/series-6.18/submission/)
is a dry-run hand-off bundle for the owner: package README (status, artifact checklist, the
**zero remaining blockers** - all five active devicetree deltas and the flashinfo loader hand-off
are boot-tested; the retained driver/YAML and reserved-memory-binding items are maintainer-facing
RFC documentation - reproduction commands), a one-page v3 **RFC cover-email draft**, an explicit
**no-Tested-by policy** (`TESTED-BY-POLICY.md`) and `MANIFEST.sha256` covering the review-form
documents, DTS-0001 (current sha `886f3ad0...`), release-line classification, tooling and
validation logs. The maintainer-facing documents (not applied) are retained under
[docs/incoming/upstream-blockers/](docs/incoming/upstream-blockers/)).
**Nothing has been submitted to any mailing list, account or web service**, and the package
contains no transport. The manifest in this repository was regenerated against the curated tree
(the notebook copy's manifest hashes cover its own layout/original files); verify with
`sha256sum -c MANIFEST.sha256` from inside the package. [V]

---

## Hardware

| Item | Value |
| --- | --- |
| Board | Cudy WR3000 v2.0, HiSilicon "luofu" v2-class design, board **R116**. The enclosure label may say WR3000 **v1**: the PCB is the luofu design, **not** the MediaTek MT7981 board that other "WR3000 v1" documentation describes. A WR3000 v1 / MT7981 image cannot boot this unit. |
| SoC | HiSilicon **Hi5671Y**, 2x Arm Cortex-A9 (ARMv7-A), platform `hsan luofu` |
| RAM | 128 MB DDR3 |
| NAND | 128 MB, ESMT F50L1G41LC (2048 B pages, 64 B OOB, 128 KiB erase blocks); FMC controller at `0x10a20000` |
| Wi-Fi | Hi5622V100 over PCIe (two endpoints `59e7:0005`) + a closed 928,920-byte firmware blob; vendor `hi5622v100_*` modules |
| Stock software | OpenWrt 22.03.6, kernel 5.10.201, 58 vendor modules under `/lib/hisilicon/ko` (all declaring `license=GPL`, no published source) |
| Bootloader | U-Boot 2022.07 "for luofu"; accepts only FIP containers (a legacy uImage followed by a fixed trailer carrying the vendor base DTB + DTBO overlays) |
| Kernel slot budget | 8,650,752 B per kernel slot (`kernela` / `kernelb`) |
| Package arch | `arm_cortex-a9` / musl |

The identity warning matters: public "WR3000 v2" documentation is partly wrong because it
describes the MediaTek v1 hardware. This board is the HiSilicon luofu v2 design.

---

## Target and image recipe

The OpenWrt target lives in [target-skeleton/](target-skeleton/); its
[README](target-skeleton/README.md) is the per-file status page and host-validation record, and
[RELEASE-NOTES.md](target-skeleton/RELEASE-NOTES.md) is the short 2026-11-21 release status. The
image chain is:

    target-skeleton/
    |- dts/                     # board DTS (luofu-r116.dts; conservative comment-only
    |                           # cleanup - active content = hardware-verified) + overlay (.dtso)
    |- target/linux/hisilicon/  # OpenWrt target: Makefile, Kconfig, subtarget luofu/
    |   |- luofu/               # target.mk, config-5.10, config-6.18, device profile
    |   |- patches-6.18/        # the 64 L0 patch files (001-060 + 062-065)
    |   |- image/               # image recipe: luofu.mk, ubinize config, Makefile
    |   |- base-files/          # 02_network, platform.sh (sysupgrade), luofu-mac.sh,
    |                           # luofu-netdev.sh, boot-pair recorder, heartbeat
    |- tools/                   # fip-slice.py, luofu-image.py, ubi-volume-check.py,
    |                           # pad-dtb.py, build-kernel-slice.sh, mocked tests
    |- docs/port-notes/         # durable per-lane integration records
    |- docs/incoming/           # retained read-only reports: device feature inventory,
    |                           # Wi-Fi recon (curated notes explain superseded rows)

What the image chain does, and what it is constrained by:

- **Kernel slot = legacy uImage + FIP trailer.** The stock loader runs
  `mtd read kernel${bootflag} ${loadaddr}; bootfip ${loadaddr}` and refuses a bare uImage.
  The trailer is a base DTB + a 96-byte overlay-container descriptor (gate magic
  `0xd7b7ab1e`) + board DTBO overlays; the loader applies the first matching overlay or
  never jumps. The format is documented byte-for-byte in
  [TRAILER-FORMAT.md](docs/TRAILER-FORMAT.md).
- **The loader hand-off devicetree contract is mandatory.** The board DTS must carry the stock
  base DTB's `/aliases/console` and `/reserved-memory/flashinfo@80600000` node; without
  them `bootfip` aborts staging and SMP never comes up (see above, and
  [docs/TARGET-DESIGN.md](docs/TARGET-DESIGN.md)).
- **The board DTS is comment-clean but semantically the hardware-verified one.** The
  2026-11-21 conservative cleanup removed comments only (1,329 -> 691 lines); the compiled DTB
  is byte-identical to the known-good DTB (13,041 B sha256 `49fe5c47...`). **Do not change
  active DTS content without a boot-test**: the superseded aggressive cleanup (image
  `6f19c6d6`) did not boot on hardware (see the regression record and
  [07-dts-upstream-bindings.md](upstream/series-6.18/07-dts-upstream-bindings.md)).
- **Load/entry address is derived, not guessed:** `(0x83200000 & 0xf8000000) + 0x00608000 =
  0x80608000`, and the recipe recomputes/asserts it from the built `vmlinux`.
- **Trailer generation is fail-closed** and reproduces both recorded vendor containers
  byte-for-byte; the loader accepted a FIP slice assembled from our own base DTB +
  loader-match overlay + uImage (2026-11-19). Per-slot A/B trailer handling remains [U].
- **Rootfs** = squashfs in a static UBI volume **plus an empty dynamic `rootfs_data`
  UBI volume** (`vol_size=1MiB`, `vol_flags=autoresize`) that the kernel's first
  read-write UBIFS mount formats into the persistent overlay - the standard OpenWrt
  squashfs+UBI layout. The sysupgrade container carries `CONTROL`, `kernel`,
  `root`, `fip-manifest.txt` and an honest `INSTALL-STATUS.txt` marker.
- **Device controls:** boot-pair recorder (`luofu-pair-record`) and a userspace
  heartbeat (`heartbeat`, 60 s) are installed in `base-files`; the canonical module gates
  are baked into the built-in cmdline (listed above).
- **Board controls (DT + target config):** gpio-keys-polled (reset/WPS) and six
  `oem:green:*` gpio-leds nodes live in the single-source DTS; the target config enables
  `LEDS_GPIO`, `INPUT`/`INPUT_EVDEV`/`INPUT_KEYBOARD`, `KEYBOARD_GPIO_POLLED` and
  `WATCHDOG_SYSFS` (`INPUT_HISI_POWERKEY` off). Verified on image `7f270b8c`; thermal
  remains unsupported.
- **Base-DTB padding (image-build step):** the compiled base DTB must have an 8-byte-aligned
  FDT totalsize for the stock loader's gate-word load. `Build/luofu-pad-dtb`
  ([tools/pad-dtb.py](target-skeleton/tools/pad-dtb.py)) pads it before `append-dtb`, folds the
  padding into the totalsize and asserts `len(dtb) == totalsize` and `% 8 == 0`; measured
  13,041 B -> 13,048 B ([TRAILER-FORMAT.md](docs/TRAILER-FORMAT.md) section 11).

The earlier host-only image generation (001-026 etc.) is superseded by the bench-verified
canonical and board-feature images; the recipe also carries the pad-dtb step and the
board-feature config, and the known-good baseline `7421bb7a` is reproduced byte-for-byte
by the conservative comment-only DTS rebuild. The stock-mediated install path is verified
(manually and with the published bench installer in [tools/](tools/)), but the image container
keeps the `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` marker: there is no
published install image, `platform_do_upgrade` has never run on a device, and the loader/env
and per-slot-trailer items remain open.

---

## Install path (bench-verified: manual stock install + automated stock-telnet install)

The stock loader keeps two kernel/rootfs slot pairs (`a`/`b`) and selects one
through the environment `bootflag`; a SYSCTRL boot-register cell records the pair and a
failed-boot record. The design ([docs/SELECTOR-0x21.md](docs/SELECTOR-0x21.md)) reconstructs
the contract from local evidence: **`0x21` is the project's bench marker, never a vendor
value, and it never persists** - the vendor userspace writes only `0x10`/`0x11`.

On top of that contract:

- patch **0017** adds the `luofu-selector` sysfs interface (`current_pair` /
  `next_pair`), storing with the loader's low-byte read-modify-write and reading the cell
  back;
- `base-files/lib/upgrade/platform.sh` implements `platform_check_image` and a
  `platform_do_upgrade` that writes only the **inactive** pair, reads back, and moves the
  selector **last**;
- `base-files/etc/init.d/luofu-pair-record` re-records the running pair after a
  successful boot so the loader does not take its failover path on the next reboot (this fixed
  the 2026-11-20 reboot fallback; the canonical image reboots with 2 CPUs and pair b recorded);
- `base-files/lib/functions/luofu-mac.sh` reads the factory MAC from `bdinfo+0xde00`,
  derives LAN = base and WAN = base+1, validates it, and falls back to a logged random pair on
  corruption (the factory partition is only ever read);
- patch **0026** adds the kernel-side half: the read-only bdinfo MTD nvmem provider
  (`fixed-layout` / `mac-base` cell) feeds the GEMAC netdevs (LAN base / WAN
  base+1), and `base-files/lib/functions/luofu-netdev.sh` records the guarded vendor
  LAN/WAN map only when all four GEMAC netdevs exist.

Host validation: `tools/test-platform-abinstall.sh` passes **94/94** assertions against
mocked MTD/sysfs/tools (**100/100** with the real built container),
`tools/test-mac-provisioning.sh` passes **103/103** and
`tools/test-mac-nvmem.sh` passes **82/82** (**96/96** with the compiled-DTS checks).

On **2026-11-19** the fail-closed sequence was executed **manually** on the unit from the
recovered stock shell (which has no SSH): inactive-pair kernel/rootfs writes, read-backs, a
semantic UBI-volume verification (the raw-partition hash cannot match because `ubiformat`
rewrites the EC `image_seq`; verifier:
[target-skeleton/tools/ubi-volume-check.py](target-skeleton/tools/ubi-volume-check.py)), then
the selector moved last as a single write. `platform_do_upgrade` itself still has never
run on a device, and the loader environment re-stamp remains unverified.

### Automated stock-telnet install (verified 2026-11-21)

The vendor stock image has **no SSH/dropbear/opkg**, so the earlier SSH-based first-install
tooling cannot run on this unit. The published bench tooling closes that gap over the stock
**root telnet shell**:

- [`tools/stock-telnet-install.sh`](tools/stock-telnet-install.sh) - fail-closed installer.
  `--inspect` is read-only; `--apply` requires `--attended --recovery-armed`, refuses if
  the bench-link/recovery preflight fails, splits/uploads the sysupgrade members, verifies the
  uploaded hashes, writes **only the inactive pair**, verifies the read-back slot hash, verifies
  the **active pair is unchanged**, moves the selector as a single write, reboots and checks our
  image answers with 2 CPUs.
- [`tools/telnet-exec.py`](tools/telnet-exec.py) - minimal command helper for the stock telnet
  shell (no auth material).
- Verified end-to-end on image `7421bb7a`: pair b written over telnet, active pair a
  unchanged, selector `0x11` verified, reboot into our 6.18 image with 2 CPUs online, six
  LEDs and `event0`. The notebook records the run in
  [target-skeleton/RELEASE-NOTES.md](target-skeleton/RELEASE-NOTES.md) and the retained
  [feature-inventory update](target-skeleton/docs/incoming/feature-inventory/UPDATE-2026-11-21.md).
  [V]

> This tooling is published for evidence and reproducibility, not as a recovery/install guide:
> it needs the vendor stock firmware and the notebook's bench-link/recovery helpers (not
> published), and it only writes the inactive pair. See [tools/README.md](tools/README.md).

---

## Connectivity: the bench workaround and what is not solved

The bench LAN works on the canonical image through **explicit, documented, attended gates** -
it is not a native Ethernet bring-up:

- the PIE engine arms with `luofu_pie.arm=1` and the decoded CPU-copy path delivers
  port-0 copies to the mirror-source netdev
  (`cpucopy_deliver=1 cpucopy_port=11`, `accepterr=1`, `rxcfg=1`);
- the switch side stays read-only-plus-mirror: `luofu_switch.arm=1` (W3 gates) plus the
  reviewed qid-map write path (`qidmap=1`) and the W4 egress stage (`egress=1`),
  with **static CPU-copy mirror** `luofu_switch.cpu_mirror=1 mirror_dst=4` and the
  post-link re-latch from W19;
- the GEMAC registers the netdev only after the PIE backend is armed and the egress map is
  ready; `luofu_gmac.macregs=1` applies the measured static MAC words.

With that stack the canonical image passes ping at 0% loss, SSH, and keeps the overlay across
a reboot. **Native CFE-equivalent L2/VLAN forwarding is a separate project**: the 2026-11-20
live-MMIO campaign showed the vendor forwarding state lives in loader/DDR-carveout state and
is not MMIO-replayable; the CPU-copy mirror is the bench solution, not an upstream feature.
The full write campaign, the EQM qid-to-egress selection and multi-port/WAN behaviour remain
attended work ([docs/LSW-SPEC.md](docs/LSW-SPEC.md),
[target-skeleton/docs/port-notes/lsw-egress/SUMMARY.md](target-skeleton/docs/port-notes/lsw-egress/SUMMARY.md)).
On the quiet image `7421bb7a` the periodic W17/W19/heartbeat dmesg spam is gone (W17 162 -> 0,
W19 165 -> 3 boot-only, HB 61 -> 1 at ~52 s uptime), so a clean boot log can now be captured
for the remaining [U] items.

### Multi-port Ethernet: current limit and the next experiment (2026-11-21)

Only **one jack (eth2)** works on the verified baseline, through the CPU-copy mirror
workaround. The measured limit, recorded in [docs/MULTI-PORT-PLAN.md](docs/MULTI-PORT-PLAN.md):

- the switch ingress mirror is programmed for eth2/port 11 to CPU destination port 4; mirror
  copies arrive at PIE with **completion port == 0**, and the PIE delivery code uses a fixed
  `cpucopy_port` parameter - the CPU cannot tell which jack a mirrored frame came from;
- the far-right bench jack was moved to live: the box stayed alive and its LAN bridge sent
  IPv6 RAs, but host-to-box frames were not delivered (the mirror covers eth2 only) - so that
  jack is a **LAN port, not WAN**, and the limitation is exactly the missing source-port /
  CPU-delivery mechanism;
- delivering all mirrored frames to one netdev would be wrong bridge behaviour, so extending
  the mirror mask alone is **not** a correct multi-port solution.

Next step is the attended **source-port experiment**: add a `luofu_switch.mirror_src_mask`
test parameter, move the cable between eth2 and candidate jacks, capture RXDIAG descriptors
and compare every byte. **Decision gate:** if a source-port field exists, implement a per-port
delivery table; if not, multi-port LAN/WAN is the native L2/VLAN hardware-bridge project
([docs/LSW-SPEC.md](docs/LSW-SPEC.md)), not a CPU-copy hack. The plan includes the attended
gates and the restore-to-known-good step; no multi-port claim is made today.

**Prepared experiment (2026-11-21).** Image **`a91ebcf0`** was built for the run:
the known-good `7421bb7a` userspace/kernel plus the bench-only source-mask probe patch
[`066-net-lsw-mirror-src-mask.patch`](docs/incoming/multi-port-probe/066-net-lsw-mirror-src-mask.patch)
(module parameter `luofu_switch.mirror_src_mask`, default `0x800`) and built-in probe
tokens `luofu_pie.rxdiag=1 luofu_switch.mirror_src_mask=0x1e00`. The patch is not part of the
release line or review form; the build clone was restored after producing the image. The cable
is currently on a non-eth2 jack, so the run is blocked until it is moved back; the image and its
sha256 are recorded in [docs/MULTI-PORT-PLAN.md](docs/MULTI-PORT-PLAN.md).

**Measured result (2026-11-21): NEGATIVE.** The probe image was installed on the unit (active
stock pair untouched, selector verified) and the far-right LAN jack was exercised with the
extended mirror mask (ports 9-12). Frames were mirrored and delivered, but the CPU-copy
completions are **field-for-field identical** to the eth2 baseline: `port=0`, `vport=0`, only
descriptor words 0/1 populated, no source-port field. The decision gate therefore resolves to
the **native L2/VLAN hardware bridge** ([docs/LSW-SPEC.md](docs/LSW-SPEC.md), separate project);
do not extend mirror-mask + CPU-copy delivery as a multi-port solution and do not fake port
identity by MAC heuristics. The board was restored to the known-good image `7421bb7a`
afterwards. Full descriptor evidence: [docs/MULTI-PORT-PLAN.md](docs/MULTI-PORT-PLAN.md) section
"Measured result of the source-port experiment". [V]

**Native-path (E0-a) experiment also negative (2026-11-21).** With `cpu_mirror=0` the unit boots 2
CPUs and native-path completions arrive on port 11, but every frame is `status=37 err=1` and the
IP path is dead (the stock CPU-punt/IFC/tagact classification is missing). The mirror workaround
therefore remains required; the native-forwarding implementation plan and
read-only/print-only instruments are retained under
[docs/incoming/lsw-forwarding/](docs/incoming/lsw-forwarding/).

**W22 update (2026-11-21): native delivery is now the canonical data path.** The driver
initializes the PFE NAPT/flow engine (release-line patch 059) and the native exception class
delivers unicast to the CPU (patch 060), so the canonical image `50023446` needs **no mirror
tokens**; M1a was first proven with image `5800c9ab`. The CPU-copy mirror patches 054-057 stay in
the release line only as the documented fallback (`naptinit=0` + mirror tokens). **Multi-port
(per-port delivery, LAN-to-LAN) is the pending M1b bench step and is not claimed**; the fallback
image rebuild and soak are pending too.

---

## Evidence discipline

Every claim in the documentation carries one of these markers, and this repository tries hard
not to blur them:

| marker | meaning |
| --- | --- |
| **[V]** | verified from a named committed artifact or measurement; reproducible host-side from the named path |
| **[I]** | inferred or planned; follows from [V] evidence but is not directly measured |
| **[U]** | unknown/unmeasured; must be closed by an attended measurement before being relied on |
| **[X]** | externally blocked (vendor GPL source, missing hardware dependency, licensing decision) |

The docs were written inside a live research notebook; paths like `~/openwrt-lab*` are
the author's host build clones (sanitized to `~`), and paths under `build/`,
`dumps/` and `firmware/` are private lab evidence that is **not** published here.
The attended-session logs, the UART captures and the canonical-image soak/reboot records
(`build/evidence-firstboot/`, `build/evidence-bench-gates/`, and the
`build/linux-kbuild/waves/` trees) are the same kind of private evidence. See
[docs/README.md](docs/README.md) for the documentation index and the link conventions.

---

## Repository layout

    README.md                  # this page
    LICENSE-MIT                # documentation and tools
    LICENSE-GPL-2.0            # kernel code (patches, DTS, OpenWrt target/base-files)
    NOTICE.md                  # provenance, no-vendor-code policy, licenses, radio/legal notes
    tools/                     # bench stock-telnet installer + telnet helper (MIT; see README)
    docs/                      # engineering design, RE specifications, publication policy
    |- TARGET-DESIGN.md        # target/DTS/image design + per-driver upstream distance
    |- TARGET-SKELETON.md      # target-skeleton index and status
    |- ROADMAP-OPENWRT.md      # phase plan and milestones (marked where superseded)
    |- GPL-REQUEST.md          # GPLv2 section 3 source-request record (provenance; unanswered)
    |- SELECTOR-0x21.md        # A/B boot-selector contract reconstruction
    |- TRAILER-FORMAT.md       # FIP trailer byte format (copy; canonical next to the tool);
    |                          # section 11 = base-DTB 8-byte totalsize alignment (pad-dtb)
    |- MISC-SOC-BLOCKS.md      # efuse/MAC, RNG, PWM, SYSCTRL/SYSENV RE
    |- INVENTORY-2026-11-18.md # measured read-only silicon inventory (600/600 reads)
    |- BENCH-READONLY-INVENTORY.md # attended read-only inventory procedure + gates
    |- LSW-SPEC.md             # internal LAN switch RE specification + bench forwarding campaign
    |- PIE-DATAPATH.md         # packet-engine/GEMAC data-path RE specification
    |- SMP-RE.md               # CPU1 release/SMP RE + final resolution (sections 21-24)
    |- REMAINING-DRIVERS.md    # remaining-driver evidence survey and port plans
    |- SKELETON-SMOKE.md       # full validation session log, incl. the attended bench sessions
    |- MULTI-PORT-PLAN.md      # multi-port Ethernet limit, source-port experiment, decision gate
    |- REVERSAL-REPORT.md      # consolidated bench-reversal report (from the lab notebook)
    |- PUBLISHING.md           # what is in this curated repo, what is excluded, and why
    |- README.md               # documentation index + link conventions
    target-skeleton/           # OpenWrt target, DTS, image recipe, tools, tests
    |- RELEASE-NOTES.md        # short release status (2026-11-21)
    |- docs/port-notes/        # durable lane records (clk, fmc, lsw, net, pie, pie-decode,
    |                          # lsw-egress, mac-nvmem, pcie, pinctrl, restart, smp, smp-decode,
    |                          # qm-decode, firstinstall, overlay, thermal, ...)
    |- docs/incoming/           # retained read-only reports: device feature inventory (+ update),
    |                           # Wi-Fi recon (curated notes explain superseded rows)
    docs/incoming/multi-port-probe/ # bench-only source-port probe patch 066 + note (not release line)
    docs/incoming/upstream-blockers/ # retained maintainer-facing RFC/review documents (all
    |                           # documented blockers closed; text drafts only)
    docs/incoming/lsw-forwarding/    # retained native L2/VLAN forwarding plan + E0-a/E0-b
    |                           # records + read-only/print-only instruments
    docs/INSTALL-RECOVERY-LESSONS.md # bench-verified install/recovery lessons (pair-b-only,
    |                           # env-bootflag/cell coupling, mixed-pair diagnosis)
    upstream/series-6.18/      # review form (001-026) + release line + DTS-0001 + raw logs
    |- 05-release-line.md      # the 53-patch release line, canonical config, bench evidence
    |- 06-dts-series-dependency.md # DTS-0001 dependency of kernel patch 0018
    |- 07-dts-upstream-bindings.md # DTS cleanup record: hardware regression, conservative
    |                          # cleanup, optional hardware-tested deltas, blockers
    |- submission/             # documentation-only upstream submission package (RFC, not sent):
    |                          # README, cover-email draft, TESTED-BY-POLICY, MANIFEST.sha256
    |- release-line/           # include/exclude manifest (51/11) + hashes
    |- dts-series/             # DTS-0001 commit-ready board-DTS entry
    |- validation/             # raw review-form and release-line validation logs

The owner-only legacy history-purge runbook (`PUBLISH-PURGE-RUNBOOK.md`) and
`PUBLISH-PREP.md`/`PUBLISH-READINESS.md` stay **notebook-side** and are not published in this
repository (they describe destructive operations on the legacy repository); this repository
records the pointer/status only. `PUBLISH-READINESS.md` item 1 is **still unchecked** and no
history rewrite, force-push or cache purge has run.

---

## Verification

The publication claims are checkable with the repository itself:

    # 1) no vendor binaries / build artifacts anywhere in the tree or the history
    git ls-tree -r HEAD --name-only | grep -Ei '\.(o|ko|bin|img|dtb|dtbo|elf|ubi|a|so)$' || echo "clean tree"
    git rev-list --objects --all | grep -Ei '\.(o|ko|bin|img|dtb|dtbo|elf|ubi|a|so)$' || echo "clean history"

    # 2) both licenses present
    ls LICENSE-MIT LICENSE-GPL-2.0 NOTICE.md

    # 3) the review form is reproducible and its hashes verify (89 files)
    python3 upstream/series-6.18/tools/gen-series.py --check
    (cd upstream/series-6.18 && sha256sum -c validation/sha256sums.txt)

    # 4) the 53-patch release line applies and the loader hand-off DTS contract compiles
    upstream/series-6.18/tools/check-release-line.sh /path/to/linux-6.18.60.tar.xz /tmp/luofu-release-line

    # 5) the review form applies to pristine linux-6.18.60 (needs that tarball)
    upstream/series-6.18/tools/run-validation.sh /path/to/linux-6.18.60.tar.xz /tmp/luofu-val

    # 6) image-artifact validator (on an image built from the recipe; not shipped here)
    python3 target-skeleton/tools/luofu-image.py check-container <built-sysupgrade.bin>

**Mocked host checks (2026-11-21 zero-blocker sync).** `test-mac-nvmem.sh` was rewritten for the
adopted 6.18 userspace MAC path after the nvmem-cell removal and now passes **77/77**; the
first-install (**94/94**) and factory-MAC (**103/103**) harnesses also pass. The release-line
validator, the review-form/series checks and the submission manifest were re-run for the final
devicetree (see `docs/PUBLISHING.md`).

The tree has a **fresh git history** (no history was copied from the research repositories,
so no vendor object that ever existed there is present here). The attended-session evidence
(UART logs, images) and the host-only image rebuilds are private lab evidence and are not part
of this repository or its history.

---

## Safety and legal

- **This is not a flashing guide and not a release.** Three bench phases are recorded:
  2026-11-19 first userspace boot (patches 001-017, console only, no network); 2026-11-21
  canonical 2-CPU image `974e6ca6` (bench LAN through the CPU-copy workaround, SSH,
  persistent overlay, 15-min soak, reboot); and 2026-11-21 board-feature image `7f270b8c` /
  quiet image `7421bb7a` (LEDs/keys/watchdog, dmesg spam removed). None is an install or
  recovery procedure; the install path (`platform_do_upgrade`, the loader env re-stamp) has
  never run on a device. If you brick a WR3000 v2.0, recovery is physical (power-off + RESET
  held through the counter + TFTP) and is not covered here. The bench stock-telnet installer
  in [tools/](tools/) is published for evidence, not as a general install guide: it needs the
  vendor stock firmware and the notebook's bench-link/recovery helpers, and `--apply` only
  writes the inactive pair after its attended/recovery gates pass.
- **An active-DTS change must be boot-tested before it is trusted.** The superseded aggressive
  DTS cleanup (image `6f19c6d6`) passed every host check but did **not** boot (MTD partition
  registration stalled, U-Boot watchdog reset, stock fallback). The accepted conservative
  cleanup is comment-only and compiles to the hardware-verified DTB; host checks are necessary
  but not sufficient.
- **Do not flash a WR3000 v1 / MediaTek image** (or any image) to this board. The board is
  the HiSilicon luofu v2 design; wrong images are unrecoverable without a physical recovery
  session.
- **UART on this unit is read-only.** The bench unit's host-to-router serial path is physically
  dead (burnt resistor); the project captured console output read-only. Do not type into a
  UART attached to a board you did not verify.
- **Provenance is GPL-compatible reverse engineering, not "clean-room".** The drivers are
  derived from documented reverse engineering (disassembly, register traces, decoded vendor
  device trees) of the vendor's GPL-declared binaries. No vendor code, binary, blob or
  extracted DTB is redistributed here. The Wi-Fi stack remains vendor-only, its firmware is
  closed, and Wi-Fi is declared **unsupported on the 6.18 target** (the vendor 5.10 stack is
  the only Wi-Fi-capable system). Thermal telemetry is likewise unsupported on this board.
- **GPL source request:** the vendor ships 58 modules declaring `license=GPL` with no
  published source. A GPLv2 section 3 request was sent to the vendor on 2026-10-01; no reply
  or receipt is held (record: [docs/GPL-REQUEST.md](docs/GPL-REQUEST.md)). That record is
  kept for provenance, not as a project gate.
- **Radio regulation:** calibration values, power tables and `txpower` must stay inside
  the regulatory limits of the country of operation. Nothing here instructs otherwise.
- **No warranty.** Host-validated engineering plus three bench phases on one unit; use at your
  own risk.
- Cudy and HiSilicon are trademarks of their respective owners; this project is independent
  and not affiliated with either company.

---

## Links and provenance

- Lab notebook / live research record: **[cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)**
- Security research (CVE write-ups, affected versions, disclosure status): **[cudy-wr3000v2-cve](https://github.com/ShibbityShwab/cudy-wr3000v2-cve)**
- This repository is a curated, fresh-history snapshot of the lab notebook's `opensource/`
  subtree at the **2026-11-21 W22 release sync** (source `opensource` submodule `5a6d29d` on notebook
  parent `8558465`): the **53-patch release line** (L0 64 files: 001-060 + 062-065; manifest
  sha256 `619eb15a...`) including the **W22 native-forwarding patches 059** (PFE NAPT/flow-engine
  init) and **060** (PIE native exception-class delivery), the DTS-only loader hand-off SMP fix
  plus the stock board_id=3 LED/keys nodes, the **conservative comment-only board-DTS cleanup
  plus all five boot-tested deltas** (TWD level-high, MAC-nvmem removal, switch MMIO-first
  `reg`/`reg-names`, `cudy,wr3000-v2` first compatible, flashinfo compatible pair; DTS 681
  lines `381b2e8c...`; DTB `62574b11...` / pad `8d79794e...`; current DTS-purpose baseline image
  `6e9c54d0`), the base-DTB `pad-dtb` build requirement, DTS-0001 (`886f3ad0...`, 694 lines), the
  **26-patch review form** (unchanged by W22 - W22 is release-line only; patch 0021 MMIO-first,
  rerun 26/26, checkpatch 0/58), the **documentation-only upstream submission package** (**zero
  remaining blockers**), the retained maintainer-facing review documents
  (`docs/incoming/upstream-blockers/`), the bench baselines `974e6ca6` / `7f270b8c` / `7421bb7a`
  (original) / `f6685b80` / `c3ef2a50` / `6e9c54d0` (current DTS-purpose) and the **canonical
  W22 data-path image `50023446`** plus the **M1a proof image `5800c9ab`**, the fully verified
  LEDs/RESET/WPS physical checks, the automated stock-telnet install tooling with the
  **bench-verified install/recovery lessons** (`docs/INSTALL-RECOVERY-LESSONS.md`), the measured
  multi-port limit (source-port experiment **negative**) and the bench-only probe record (image
  `a91ebcf0`, patch 066), the **E0-a/E0-b native-forwarding experiments**, the **M1a proof and W22
  release integration**, the retained **native L2/VLAN forwarding docs/instruments** (`m1a/`,
  `w22/`, `m1b/` and analysis under `docs/incoming/lsw-forwarding/`), the DTS regression record,
  the retained read-only device feature inventory (+ update) and Wi-Fi feasibility reports, and the
  updated engineering docs. The owner-only history-purge runbook stays notebook-side (not
  published); `PUBLISH-READINESS.md` item 1 remains unchecked. Publication policy: [docs/PUBLISHING.md](docs/PUBLISHING.md);
  provenance: [NOTICE.md](NOTICE.md).

> **The honest bottom line:** this is real, host-validated and bench-verified engineering on
> one unit - including a **verified install path**: the published bench stock-telnet installer
> wrote the inactive pair from the vendor stock firmware, left the active pair untouched,
> > verified the selector and booted our 2-CPU image. The original known-good baseline is
> `7421bb7a` (2 CPUs, six LEDs, gpio-keys-polled, watchdog sysfs, quiet dmesg, bench LAN through
> the documented CPU-copy workaround), reproduced byte-for-byte by the conservative comment-only
> DTS cleanup; the current DTS-purpose baseline is **`6e9c54d0`** (all five boot-tested devicetree deltas),
> and the **canonical W22 data-path image is `50023446`**: the driver initializes the PFE
> NAPT/flow engine (release-line patch 059) and native exception-class delivery (patch 060) makes
> eth2 work **without the CPU-copy mirror** (2 CPUs, ping 200/200, SSH, reboot determinism); M1a
> was first proven with image `5800c9ab`. **All documented upstream blockers are closed**, and
> **W22 is release-line only - not in the L2 review form**. LEDs and both RESET/WPS buttons are
> physically verified. The mirror patches 054-057 remain only as the documented fallback
> (`naptinit=0` + mirror tokens). **M1b (per-port/LAN-to-LAN), the fallback-image rebuild and the
> soak are pending bench steps**, and multi-port is not claimed. The E0/M1a history and retained
> native-forwarding docs/instruments are under `docs/incoming/lsw-forwarding/`; the **install/recovery
> lessons** are recorded in `docs/INSTALL-RECOVERY-LESSONS.md`. The owner-only legacy history-purge
> runbook is prepared/dry-run validated notebook-side **but not executed** (`PUBLISH-READINESS.md`
> item 1 still open). The base-DTB `pad-dtb` alignment bug was found on hardware and is a guarded
> build step; the aggressive DTS cleanup is a recorded boot regression and was reverted. These
> results supersede the earlier "host-only, never booted" state - but they do **not** make this
> a release: `platform_do_upgrade` has never run on a device, per-slot A/B trailer handling
> and the loader env re-stamp remain open, only one Ethernet jack works through the CPU-copy
> workaround (multi-port currently needs the native bridge), native switch forwarding is a
> separate unresolved project, Wi-Fi is vendor-only and unsupported on 6.18, thermal is
> unsupported on this board, per-driver silicon validation is partial, and no upstream
> submission exists. The 26-patch review form is a set of drafts/RFCs, not merge-ready kernel
> code. Treat the repository as a well-documented, honest starting point for an upstream port -
> and treat the private lab evidence as the evidence of record. The upstream submission package is a
dry-run for the owner only - **not sent, no Tested-by** - and the multi-port source-port
experiment has been run and was **negative**: the CPU-copy path cannot identify jacks, so
multi-port LAN/WAN needs the native L2/VLAN bridge.
