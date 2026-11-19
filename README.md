# cudy-wr3000v2-openwrt

> **One attended hardware bring-up plus one host-only full-series image rebuild, on top of
> host-validated engineering.** On **2026-11-19** a `linux-6.18.60` kernel built from this
> target tree and an OpenWrt rootfs built from this image recipe booted to a userspace console
> on the WR3000 v2, in an **attended bench bring-up** (recovery door armed, UART captured
> read-only). That is the only hardware result in this repository, and it is **not a release**:
> the booted image was built from patches `001`-`017` only (UP, 1 CPU), it had **no network**
> and **no persistent overlay**, and it is not re-flashed. A later **host-only rebuild** from
> the full series `001`-`023` adds the persistent `rootfs_data` overlay volume and the
> connectivity draft drivers, and is deliberately marked
> `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` - **it has never been flashed or
> booted**. Everything else here is host-validated only, and this repository contains **no
> flashing or recovery instructions**. Milestones:
> [the boot](#2026-11-19-hardware-milestone-first-boot-to-userspace) and
> [the final image](#final-full-series-image-host-only-rebuild-not-flashed).

An upstream-style **OpenWrt / Linux bring-up for the Cudy WR3000 v2.0** - a HiSilicon
**Hi5671Y "luofu"** (board R116) router that ships a vendor OpenWrt 22.03.6 / Linux 5.10.201
image. The goal is a mainline OpenWrt target, device tree and driver set for this board.

This curated repository collects the parts of the research that are **safe and useful to
publish**: a Linux 6.18 kernel patch series (all 23 patches in the upstream review form;
19 merge-intent patches plus four fail-closed data-path drafts marked do-not-merge), an OpenWrt
target skeleton and board DTS, an image/install recipe (with the standard squashfs +
`rootfs_data` overlay UBI layout), reverse-engineering specifications and measured read-only
silicon inventory, and an honest status report.
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

## Status at a glance (2026-11-19)

| Area | Host-validated (evidence in this repository) | Hardware status |
| --- | --- | --- |
| Target/subtarget metadata | `make defconfig` + `make target/linux/prepare` RC=0 on `openwrt main` `27663a0` (linux-6.18.60); generated `CONFIG_TARGET_hisilicon_luofu_DEVICE_cudy_wr3000-v2=y` | the target tree built the kernel and machine DTB used by the 2026-11-19 boot (U-Boot `Image Name: Linux-6.18.60`; boot log machine model `Cudy WR3000 v2.0 (HiSilicon luofu R116)`) |
| Kernel series (patches 001-023) | **23/23** apply in order to pristine linux-6.18.60 with `patch(1)` (90 file-patch events, 0 failed hunks, 0 rejects, 0 fuzz, 3 apply offsets) **and with `git am`** (identical trees, clean work tree); the review form carries all 23 patches and keeps checkpatch 0 errors / 56 warnings (6 strict CHECKs), enumerated in [04-validation.md](upstream/series-6.18/04-validation.md); the four draft follow-ups 020-023 are marked do-not-merge | a 6.18.60 kernel built in this target tree booted to userspace on 2026-11-19, but that image was built from patches **001-017** only; the full-series `001-023` rebuild has never been booted |
| Kernel compile | full `linux-6.18.60` compile of the 001-023 series RC=0 under `CONFIG_WERROR=y`, 0 warnings / 0 errors; zImage 3,281,376 B (`5903db04...`); load/entry `0x80608000` unchanged (`_text = 0xc0608000`) | the 2026-11-19 boot loaded our kernel at `0x80608000`, but it was built from patches 001-017; the recorded 001-023 compile has never run on hardware |
| Board DTS | compiles with the kernel's own dtc (1.7.2), zero warnings; every new YAML binding passes `dt_binding_check` | a DTB compiled from this DTS was applied by the loader and used by the 2026-11-19 boot (machine model matches); full `dtbs_check` is **not** clean |
| FIP trailer generator | reproduces both recorded vendor containers byte-for-byte; fail-closed on every decoded loader check | a FIP kernel slice assembled from our own base DTB + loader-match overlay + uImage was booted from the inactive slot on 2026-11-19; per-slot A/B trailer handling is still [U] |
| Target image | `make world -j16` RC=0 in isolated clones; the **final full-series rebuild** (patches 001-023, persistent `rootfs_data` overlay volume) is **6,205,742 B**, sha256 `15dae772...` (kernel slice `6c99aa3d...`, rootfs UBI `81d04951...`; `check-container` PASS, repo tool re-validates it) | a kernel slice + rootfs UBI assembled with this tree's host tooling were written to the inactive pair and booted (2026-11-19 attended) - but that build was patches 001-017. The final 001-023 container is marked `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` and was **not flashed** |
| A/B install path | `platform_do_upgrade` runs against mocked MTD/sysfs/ubiformat: 94/94 assertions (100/100 with the real built container) | the fail-closed sequence (inactive pair only, read-backs, semantic UBI verify, selector last) was executed **manually** on the unit on 2026-11-19; `platform_do_upgrade` itself still has never run on a device; the loader env re-stamp remains open |
| Factory-MAC provisioning | bdinfo `+0xde00` derivation + validation: 103/103 mocked assertions | no MAC read from a real unit; no netdev wired (the 2026-11-19 boot had no network) |
| Silicon inventory 2026-11-18 | first attended **read-only** inventory of the stock vendor boot: **600/600** planned register/table reads OK (476 LSW + 108 PIE + 13 CRG + 3 SYSCTRL), 0 failed batches, boot_id unchanged; live post-overlay DT windows, queue/flow DDR base, PIE ring geometry, LSW W1 baseline + qid maps A/B, CRG `clk_summary` and SYSCTRL cells measured; 16 base-DT expectation tokens corrected | no write, no reboot: one boot, read-only; no traffic/behaviour claim |
| CRG clock/reset (004, 014, 019) | compiles; `dt_binding_check`; 16 gates + 2 PLLs + 2 muxes modelled **read-only**; the PLL1 control words were read back in the 2026-11-18 inventory | no rate/parent/register write attempted |
| Watchdog, thermal, pinctrl, PCIe, FMC/NAND, restart (005, 006, 010, 011, 012, 013) | compile + binding checks pass; drivers are built-in in the target config | the 2026-11-19 boot registered the FMC/SPI-NAND (ESMT 128 MiB, 17 partitions, UBI rootfs mounted), watchdog, thermal and restart drivers; pinctrl/PCIe were not exercised |
| SMP bring-up (018) | platsmp draft compiles; vendor release sequence decoded and documented | the 2026-11-19 boot log reports `Brought up 1 node, 1 CPU`, so the CPU1 release path is still unproven on hardware |
| MDIO + PHY (007, 008) | complete driver ports, compile clean | bus/PHY never enumerated |
| Ethernet data path (GEMAC/PIE/LSW, 009/015/016/020-023) | patch `020` turns PIE into a draft ring engine (coherent descriptor rings for the measured two active RX/TX channels, recovered channel/IRQ init, NAPI RX path, TX producer, read-back verification) and GEMAC into a phylink/netdev front end; patch `021` makes the LSW probe read-only. **Patches 022/023 add the host-only minimum-connectivity gates:** 022 implements the reviewed W3/W4/W5 switch steps behind `luofu_lsw.arm=1` (power/snapshot) and `luofu_lsw.qidmap=1` (qid-map write + readback) with `luofu_switch_ready()`; 023 registers the GEMAC netdevs only when PIE is armed **and** the egress map is ready, fixes phylink to `PHYLINK_NETDEV`, and takes the MAC from the DT with a random fallback. The PIE engine arms only behind `luofu_pie.arm=1` and every driver fails closed by default | never ran on hardware; **no PIE ring has been armed and no LSW register has been written by our kernel**; RX completion decode and PIE TX egress selection remain [U] |
| A/B boot selector (017) | sysfs contract reconstructed from local evidence; mocked harness; the record cell was read **read-only** in the 2026-11-18 inventory (`0xc08` = `0x10`, pair A, upper 24 bits zero) | the 2026-11-19 handover moved the selector by a **single write from the stock userspace**, and the loader failover then booted slot B; patch 017's own store path still never ran |
| Wi-Fi (Hi5622V100) | reverse-engineering specification only; no open driver | vendor binary stack; GPL section 3 source request sent 2026-10-01, **no reply held** |
| **On-device validation** | - | **2026-11-19: one attended bring-up reached a userspace console** (kernel 6.18.60 + our rootfs in slot B, built from patches 001-017); no network, no persistent overlay in that build; the full-series 001-023 image with the overlay volume was rebuilt host-side and **not flashed** |

The one-line summary: the **design/build layer is real and host-validated**, and the first
hardware milestone now exists - an attended bring-up booted our kernel and our rootfs to a
console. Everything beyond that boot is still unverified: no network, no persistent overlay,
no A/B sysupgrade run, and the Wi-Fi stack is externally blocked. Do not read this repository
as a working firmware.

---

## 2026-11-19 hardware milestone: first boot to userspace

**Result: SUCCESS - first full boot of our own OS on this hardware.** The boot was an
**attended bench session** (recovery door armed; console captured read-only on UART; no
host-to-router serial input). The stock U-Boot's failover path booted the pair written by the
manual install path: `bootflag a bootreg 11` -> `boot from kernela failed, change to
kernelb` -> `Image Name: Linux-6.18.60` (load/entry `0x80608000`) -> our kernel banner ->
UBI attached `mtd14` (`rootfsb`) -> `ubiblock0_0` created from the static squashfs volume
-> VFS mounted the squashfs root read-only -> `/sbin/init` -> preinit -> procd
(early/watchdog/ubus/init) -> kmodloader -> urngd -> overlayfs (tmpfs fallback) -> console
"Please press Enter to activate this console." The captured boot log's SMP line reports `smp:
Brought up 1 node, 1 CPU`; the CPU1 release path was not exercised by this boot.

**The five fixes that turned the first kernel banner into userspace** (all in the target tree
now):

1. **Built-in command line** - the stock loader supplies **no** kernel command line for our
   uImage (first attempt: empty cmdline -> `VFS: Cannot open root device` -> panic), and a
   `/chosen/bootargs` property in our DTB made U-Boot itself data-abort before hand-off.
   The working fix is `CONFIG_CMDLINE` + `CONFIG_CMDLINE_FORCE` in
   `luofu/config-6.18`. The `ubi.mtd=rootfsb` in it is first-install-only; a real A/B image
   needs a per-slot DTB or a loader-supplied cmdline ([TARGET-DESIGN.md](docs/TARGET-DESIGN.md)
   5.5/5.6).
2. **4 KiB squashfs padding** - `ubiblock` rounds a static UBI volume down to a 512-byte
   boundary, so an unpadded squashfs loses its tail and cannot mount. `luofu-image.py
   rootfs-ubi` gained `--pad-align` and the image recipe pads to 4096 B before `ubinize`.
3. **`CONFIG_XZ_DEC_ARM=y`** - the OpenWrt squashfs uses XZ with the ARM BCJ filter
   (`Filters selected: arm`). Without the arm decoder every compressed file block failed
   although the raw FMC/NAND reads were byte-perfect (the debug CRC patch showed 900/900 pages
   matching the host image), which is how the failure was localised.
4. **Semantic UBI-volume verification** - `ubiformat` rewrites the EC-header `image_seq`
   (and its CRC) on every PEB, so a raw full-partition hash can never match a `ubinize`
   image. The first-install read-back now compares the UBI volume payload semantically; the
   verifier is published as
   [`target-skeleton/tools/ubi-volume-check.py`](target-skeleton/tools/ubi-volume-check.py).
5. **Single-write selector** - the SYSENV `boot_reg` sysfs attribute must be written once
   (`echo -n "0x11" > boot_reg`); a tty-split `printf` wrote partial values. The handover
   was done this way from the recovered stock userspace because that shell has no SSH.

**Honest gaps observed during the same boot:**

- **No network.** The LSW/PIE/GEMAC netdevs were not wired yet; the booted image is
  console-only and has no SSH. Re-flashing would need the stock userspace or physical TFTP
  recovery. The host-side connectivity gates (patches `022`/`023`) are described
  [below](#final-full-series-image-host-only-rebuild-not-flashed) and have never run.
- **No persistent overlay in the booted image.** It had no `rootfs_data` volume, so
  `mount_root` fell back to a tmpfs overlay. The full-series rebuild adds the standard empty
  dynamic `rootfs_data` UBI volume (host-validated, never booted).
- **`/lib/functions/board.sh` was missing** for `board_detect` (logged by preinit): the stale
  `board.d/02_network` sourced a helper that does not exist in OpenWrt main. The rebuild ships
  the corrected `board.d` and fails the build if any installed `board.d` sources a missing
  helper (host-validated, never booted).
- The loader env re-stamp and the A/B sysupgrade path remain unexercised.

**Artifacts and the "not re-flashed" caveat.** The image running on the unit is the
**debug-CRC build** used to localise the XZ failure (image `51baf2ac...`, kernel slot
sha256 `897d4926...`), kept running for observation. A clean production image was then
rebuilt without the temporary debug patch: 6,185,262 B, sha256
`e893c388e4ab8be61be0f76cd00b003000516005fef3d20bef5b26ecf853c8b5`, with
`CONFIG_CMDLINE`/`CONFIG_XZ_DEC_ARM` verified and its container re-validated by
`luofu-image.py check-container`. It was **not re-flashed**. The boot logs and the success
record are private lab evidence under `build/evidence-firstboot/` (gitignored, not
published); the session is recorded in
[SKELETON-SMOKE.md](docs/SKELETON-SMOKE.md) and the lab notebook.

> This milestone is not an install or recovery procedure. The published image recipe still
> marks its sysupgrade container `NOT FLASHABLE`, and this repository deliberately contains
> no flashing instructions.

## Final full-series image (host-only rebuild, NOT flashed)

On the afternoon of 2026-11-19 the image recipe was rebuilt from the **full kernel series
`001`-`023`**, closing the two boot-gaps the bring-up exposed: the rootfs slot now carries the
standard OpenWrt **two-volume UBI** - static `squashfs` plus an **empty dynamic `rootfs_data`
overlay volume** (`vol_size=1MiB`, `vol_flags=autoresize`) that the kernel's first read-write
UBIFS mount formats, so the next boot gets a persistent overlay instead of the tmpfs fallback -
and the image recipe fails the build if any installed `/etc/board.d/*` sources a `/lib/...`
helper that is not in the image (the `board_detect` failure class).

| item | value |
| --- | --- |
| sysupgrade image | **6,205,742 B**, sha256 `15dae772984669545f175806cff70216039684e494f6196e0a94b71530026b99` |
| kernel slice | 3,305,732 B, sha256 `6c99aa3d0c7b4e104c74fef5d14f355e483b90ff0fa91aa9fc2c4f4e10ea95ca`; uImage `Linux-6.18.60`, ih_size 3,293,803, load/ep `0x80608000`; FIP trailer 11,865 B (base DTB 11,571 B; gate `0xd7b7ab1e` at `0x326fde`) |
| rootfs UBI | 2,883,584 B (22 PEBs), sha256 `81d049511cb1775a1b6efc9fab9182eda86ea8069f45a75580dada9ab8330028`; volume 0 `squashfs` static (used/reserved 20 PEBs), volume 1 `rootfs_data` dynamic, **EMPTY**, 9 reserved PEBs, flags `0x01` (autoresize); 31 of the slot's 185 PEBs reserved, image well under the 24,313,856 B budget |
| container marker | `STATUS: BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` (the literal `NOT FLASHABLE` substring is kept on purpose: the install/check tooling greps it) |
| host checks | patch application 23/23 (0 rejects), `make defconfig` + `make world -j16` RC=0 with 0 warnings, `check-container` PASS, negative tests refuse the old/single-volume/pre-filled variants, `test-platform-abinstall.sh --container` 100/100, MAC provisioning 103/103 |

**The caveat is the point.** The marker records that an **earlier build of this recipe**
(patches `001`-`017`, UP, 1 CPU, no overlay volume) booted to userspace; **this exact rebuild
was never flashed and never booted.** The next attended boot would be the first test of SMP
(patch `018`), the clock tree (`019`), the PIE datapath (`020`), the LSW probe and gated
write path (`021`/`022`), the GEMAC readiness path (`023`) and the persistent overlay as a set.

### Connectivity gate plan (first network bring-up)

No network has ever run. Patches `022`/`023` implement the deterministic host-side half behind
explicit attended gates, so the first connectivity session is a single-variable sequence with
the recovery door armed - physical RESET + TFTP remains the only recovery:

| step | boot parameters | what it would exercise / prove |
| --- | --- | --- |
| **W3** | `luofu_lsw.arm=1` | the first own-kernel CRG write on the switch block: enable the two gate clocks, release the four resets, re-check the powered state read-only, snapshot the 30 measured words and both qid maps; **no LSW register write** in this step |
| **W4/W5** | `luofu_lsw.arm=1 luofu_lsw.qidmap=1` | pre-write conformance gate (unpopulated qid-map rows must read zero), program the reviewed W2 set (table A ports 4-12, table B ports 40-43, value `0x100 \| qid`), read every programmed word back and restore on mismatch; `luofu_switch_ready()` only after full verification |
| **PIE arm** | `luofu_pie.arm=1` + both LSW gates | arms the PIE rings/IRQs; the four GEMAC netdevs then register (eth0-eth3 in DT order; the bench cable is on **eth2**/gmac3) and phylink drives the carrier |
| **traffic** | + static IP / `udhcpc` on the bench-link host side | expected to expose the remaining gaps: PIE RX completion decode [U], PIE TX descriptor -> egress queue/port selection [U], pinctrl consumer states for MDIO/GEMAC still TODO |

None of those parameters is in the built-in command line, so a normal boot only logs refusals;
passing them needs a loader-supplied command line or a temporary rebuilt cmdline
(`CONFIG_CMDLINE_FORCE=y`). The exact expected console lines, the gate-model table and the
proven/unverified split are in [SKELETON-SMOKE.md](docs/SKELETON-SMOKE.md) and
[target-skeleton/docs/port-notes/net/SUMMARY.md](target-skeleton/docs/port-notes/net/SUMMARY.md).

**Honest remaining gaps:** no network path has ever executed on hardware; the PIE RX completion
decode and the TX-to-egress binding are unwritten; the persistent overlay, SMP/CPU1 and patches
`018`-`023` have never run; `platform_do_upgrade` has still never run on a device and the loader
env re-stamp is unproven; Wi-Fi is externally blocked (vendor binary; GPL request unanswered);
the image is not byte-reproducible across kernel rebuilds (UTS banner/build timestamp in the
zImage); and `platform_check_image` still validates only volume 0 of the rootfs UBI.

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

## The Linux 6.18 kernel series (patches 001-023)

The target tree carries the **23-patch** series, and the upstream **review form** covers the
same 23 patches. Both copies are included, by design:

- [`target-skeleton/target/linux/hisilicon/patches-6.18/`](target-skeleton/target/linux/hisilicon/patches-6.18/) -
  all 23 plain diffs that OpenWrt's patch application consumes (this is what the recorded
  kernel compile evidence used);
- [`upstream/series-6.18/`](upstream/series-6.18/) - the **review form** of patches 001-023:
  cover letter, per-patch commit messages, MAINTAINERS proposal, tree/order rationale, a
  repeatable validation record and raw logs, plus a commit-ready `git am` copy.
  The readable overview is its [README](upstream/series-6.18/README.md).

The review form is not a merge list: `020` (PIE ring engine + GEMAC front end), `021`
(read-only LSW baseline probe), `022` (gated LSW qid-map write path) and `023` (GEMAC netdev
readiness) are included as **draft / do-not-merge** patches documenting the measured-inventory
and connectivity follow-ups. `020` is an arm-gated data-path draft whose engine has never run
(its RX completion decode is still [U]); `021` is a read-only probe of a block our own driver
still does not program; `022` implements the first reviewed switch writes behind explicit
attended gates; `023` makes the GEMAC netdev conditional on both the PIE engine and the
verified egress map. All four are host-compiled and never booted; the ready-for-review split
and the do-not-merge reasoning are in the series
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
| 0018 | ARM: mach-hisilicon: add Hi5671Y SMP bring-up support | `CPU_METHOD_OF_DECLARE(luofu_smp)`, SCU enable, fail-safe missing-node path; board DTS integration is not in the series | RFC / attended |
| 0019 | clk: hisilicon: add the Luofu CRG clock tree | gate parents on real input clocks; two PLLs + two muxes as **read-only** clocks; seven required input clocks | ready for review |
| 0020 | net: ethernet: hisilicon: add the Luofu PIE datapath and GEMAC front end | extends `015`'s PIE driver into a draft ring engine (coherent descriptor rings for the measured two active RX/TX channels, recovered v250 channel/IRQ init, mask/unmask + NAPI RX path, TX producer with poll-only completion, read-back verification that tears down on mismatch) and `009`'s GEMAC into a draft phylink/netdev front end; the data path arms only behind the kernel-command-line opt-in `luofu_pie.arm=1` and refuses before the first register write otherwise | review-form draft; do not merge |
| 0021 | net: ethernet: hisilicon: add the Luofu switch read-only baseline probe | extends `016` to the measured 17-window model (two engine DDR windows validated, never mapped), the CRG powered-state gate (gate word bits + four active-low reset lines), a 30-word W1 register baseline and a word-for-word read-only check of both qid maps; contains no `writel()`, registers no netdev, always fails closed | review-form draft; do not merge |
| 0022 | net: ethernet: hisilicon: add the gated Luofu LSW write path | extends `021` with the attended W3/W4/W5 steps behind `luofu_lsw.arm`/`luofu_lsw.qidmap`: gate clocks + resets with a read-only snapshot (W3), then a pre-write conformance gate, the reviewed W2 qid-map write set and a full read-back with restore-on-mismatch (W4/W5); adds `luofu-switch.h` with `luofu_switch_probed()`/`luofu_switch_ready()` and the measured hardware port number (`hisilicon,port` 9-12) in the port model; no new Kconfig symbol | review-form draft; do not merge |
| 0023 | net: ethernet: hisilicon: gate the Luofu GEMAC netdev on the LSW egress map | registers the GEMAC netdev only when `luofu_pie_armed()` **and** `luofu_switch_ready()`; switches phylink to `PHYLINK_NETDEV` so the resolve work drives the carrier; takes the netdev address from the DT MAC properties with a random locally-administered fallback; `LUOFU_GMAC` now depends on `LUOFU_SWITCH`; the only MAC writes remain the measured 1 Gbps link-up/down words | review-form draft; do not merge |

Series facts (all host-only; evidence in [`upstream/series-6.18/validation/`](upstream/series-6.18/validation/)):

- 23/23 apply in order to pristine `linux-6.18.60` with `patch(1)` (90 file-patch events,
  0 failed hunks, 0 rejects, 0 `.rej`) **and with `git am`** (clean work tree; the two
  resulting trees are identical, 0 rejects, 0 fuzz). Three hunks apply with an offset: the
  pre-existing two in `0008`, plus a 6-line offset for `021` after `020` edits the same
  Kconfig file.
- `checkpatch.pl --no-tree`: **0 errors, 56 warnings** (default and strict), plus 6 strict
  CHECKs - review items such as binding split, MAINTAINERS entry and line wrapping, not
  behavioural defects. The warnings include the expected per-patch review items for the
  `020`-`023` drafts.
- `sha256sum -c` on the recorded `validation/sha256sums.txt`: **46/46 OK** (23 series
  patches + 23 source patches); series `0022` sha256 `955f85ed...`, `0023` `5da6e5fa...`.
- The final 001-023 series compiled in an isolated `openwrt main` clone: `make
  target/linux/prepare` RC=0 (23/23 target patches, 1,223 `patching file` lines, no rejects)
  and `make target/linux/compile V=s -j16` RC=0 with **0 warnings / 0 errors** under
  `CONFIG_WERROR=y`; zImage 3,281,376 B, sha256
  `5903db04928e58214a816a5809f186d579857c8e1c5c0e695cc9035bd68e91ca`. The earlier
  19-patch build gave zImage 3,264,560 B (`4456b0ec...`).
- **The booted kernel is not this artifact.** The 2026-11-19 bring-up booted a 6.18.60 build
  of the same target tree built from patches `001`-`017` (debug-CRC build); neither the boot
  nor the host-only full-series rebuild is the review-form artifact above.
- **No submission has been made.** The series is a review artifact, not an upstream-accepted
  change. Its "ready-for-review" split and the owner decisions still required (real
  `Signed-off-by` identity, binding split, folding 0014 into 0004, SMP staging) are listed in
  [`upstream/series-6.18/README.md`](upstream/series-6.18/README.md).

---

## Target and image recipe

The OpenWrt target lives in [`target-skeleton/`](target-skeleton/); its
[README](target-skeleton/README.md) is the per-file status page and host-validation record
(through the 2026-11-19 boot and the host-only full-series rebuild). The image chain is:

```
target-skeleton/
|- dts/                     # draft board DTS (luofu-r116.dts) + loader-match overlay (.dtso)
|- target/linux/hisilicon/  # OpenWrt target: Makefile, Kconfig, subtarget luofu/
|   |- luofu/               # target.mk, config-5.10, config-6.18, device profile
|   |- patches-6.18/        # the 23 kernel patches (001-023)
|   |- image/               # image recipe: luofu.mk, ubinize config, Makefile
|   |- base-files/          # 02_network, platform.sh (sysupgrade), luofu-mac.sh, boot pair recorder
|- tools/                   # fip-slice.py, luofu-image.py, ubi-volume-check.py, build-kernel-slice.sh, mocked tests
```

What the image chain does, and what it is constrained by:

- **Kernel slot = legacy uImage + FIP trailer.** The stock loader runs
  `mtd read kernel${bootflag} ${loadaddr}; bootfip ${loadaddr}` and refuses a bare uImage.
  The trailer is a base DTB + a 96-byte overlay-container descriptor (gate magic
  `0xd7b7ab1e`) + board DTBO overlays; the loader applies the first matching overlay or
  never jumps. The format is documented byte-for-byte in
  [TRAILER-FORMAT.md](docs/TRAILER-FORMAT.md).
- **Load/entry address is derived, not guessed:** `(0x83200000 & 0xf8000000) + 0x00608000 =
  0x80608000`, and the recipe recomputes/asserts it from the built `vmlinux`.
- **Trailer generation is fail-closed** and reproduces both recorded vendor containers
  byte-for-byte. On 2026-11-19 the loader accepted a FIP slice assembled from our own base
  DTB + loader-match overlay + uImage and booted it (the boot log reports our DTS machine
  model); the per-slot A/B trailer question remains [U].
- **Rootfs** = squashfs in a static UBI volume **plus an empty dynamic `rootfs_data` UBI
  volume** (`vol_size=1MiB`, `vol_flags=autoresize`) that the kernel's first read-write
  UBIFS mount formats into the persistent overlay - the standard OpenWrt squashfs+UBI layout.
  The sysupgrade container carries `CONTROL`, `kernel`, `root`, `fip-manifest.txt` and an
  honest `INSTALL-STATUS.txt` marker.
- **Host build result:** a full `make world -j16` in isolated `openwrt main` clones
  (commit `27663a0`, linux-6.18.60) completes RC=0. The post-bring-up clean rebuild (patches
  001-017, five boot fixes) was 6,185,262 B (`e893c388...`). The **final full-series
  001-023 rebuild** with the persistent overlay volume is **6,205,742 B, sha256
  `15dae772984669545f175806cff70216039684e494f6196e0a94b71530026b99`** (kernel slice
  `6c99aa3d...`, rootfs UBI `81d04951...`, `check-container` PASS; **not flashed and not
  booted**). The image that actually booted is the 001-017 debug-CRC build (`51baf2ac...`).

> **The published container is marked `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED`
> on purpose.** The marker records that an earlier build of the recipe booted to userspace and
> that this exact rebuild has not been flashed. Nothing in this repository is an install or
> recovery procedure; the 2026-11-19 bring-up was a manual attended bench procedure on the lab
> unit, not a documented install path. Do not flash the container.

---

## A/B install path (host-tested; executed manually on hardware 2026-11-19)

The stock loader keeps two kernel/rootfs slot pairs (`a`/`b`) and selects one through the
environment `bootflag`; a SYSCTRL boot-register cell records the pair and a failed-boot
record. The design ([`docs/SELECTOR-0x21.md`](docs/SELECTOR-0x21.md)) reconstructs the
contract from local evidence: **`0x21` is the project's bench marker, never a vendor value,
and it never persists** - the vendor userspace writes only `0x10`/`0x11`.

On top of that contract:

- patch **0017** adds the `luofu-selector` sysfs interface (`current_pair` / `next_pair`),
  storing with the loader's low-byte read-modify-write and reading the cell back;
- `base-files/lib/upgrade/platform.sh` implements `platform_check_image` and a
  `platform_do_upgrade` that writes only the **inactive** kernel/rootfs pair, reads back, and
  moves the selector **last**;
- `base-files/etc/init.d/luofu_record_pair` re-records the running pair after a successful
  boot so the loader does not take its failover path on the next reboot;
- `base-files/lib/functions/luofu-mac.sh` reads the factory MAC from `bdinfo+0xde00`, derives
  LAN = base and WAN = base+1, validates it, and falls back to a logged random pair on
  corruption (the factory partition is only ever read).

Host validation: `tools/test-platform-abinstall.sh` passes **94/94** assertions against
mocked MTD/sysfs/tools (100/100 with the real built container), and
`tools/test-mac-provisioning.sh` passes **103/103**.

On **2026-11-19** the same fail-closed sequence was executed **manually** on the unit from the
recovered stock shell (which has no SSH): inactive-pair kernel/rootfs writes, read-backs, a
semantic UBI-volume verification (the raw-partition hash cannot match because `ubiformat`
rewrites the EC `image_seq`; verifier:
[`target-skeleton/tools/ubi-volume-check.py`](target-skeleton/tools/ubi-volume-check.py)),
then the selector moved last as a single write. `platform_do_upgrade` itself still has never
run on a device, and the loader environment re-stamp remains unverified.

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

The docs were written inside a live research notebook; paths like `~/openwrt-lab*` are the
author's host build clones (sanitized to `~`), and paths under `build/`, `dumps/` and
`firmware/` are private lab evidence that is **not** published here. The 2026-11-19 boot
logs (`build/evidence-firstboot/`) and the afternoon rebuild evidence
(`build/linux-kbuild/waves/2026-11-19-overlay-image/`) are the same kind of private evidence. See
[`docs/README.md`](docs/README.md) for the documentation index and the link conventions.

---

## Repository layout

```
README.md                  # this page
LICENSE-MIT                # documentation and tools
LICENSE-GPL-2.0            # kernel code (patches, DTS, OpenWrt target/base-files)
NOTICE.md                  # provenance, no-vendor-code policy, licenses, radio/legal notes
docs/                      # engineering design, RE specifications, publication policy
|- TARGET-DESIGN.md       # target/DTS/image design + per-driver upstream distance
|- ROADMAP-OPENWRT.md     # phase plan and milestones (marked where superseded)
|- GPL-REQUEST.md         # GPLv2 section 3 source-request record (provenance; unanswered)
|- SELECTOR-0x21.md       # A/B boot-selector contract reconstruction
|- TRAILER-FORMAT.md      # FIP trailer byte format (copy; canonical next to the tool)
|- MISC-SOC-BLOCKS.md     # efuse/MAC, RNG, PWM, SYSCTRL/SYSENV RE
|- INVENTORY-2026-11-18.md    # measured read-only silicon inventory (600/600 reads)
|- BENCH-READONLY-INVENTORY.md # attended read-only inventory procedure + gates
|- LSW-SPEC.md            # internal LAN switch RE specification
|- PIE-DATAPATH.md        # packet-engine/GEMAC data-path RE specification
|- SMP-RE.md              # CPU1 release/SMP RE + staged bring-up plan
|- REMAINING-DRIVERS.md   # remaining-driver evidence survey and port plans
|- SKELETON-SMOKE.md      # full validation session log, incl. the 2026-11-19 boot
|- REVERSAL-REPORT.md     # consolidated bench-reversal report (from the lab notebook)
|- PUBLISHING.md          # what is in this curated repo, what is excluded, and why
|- README.md              # documentation index + link conventions
target-skeleton/           # OpenWrt target, DTS, image recipe, tools, tests (host-built; booted 2026-11-19)
upstream/series-6.18/      # review-form kernel series + raw validation logs
```

---

## Verification

The publication claims are checkable with the repository itself:

```sh
# 1) no vendor binaries / build artifacts anywhere in the tree or the history
git ls-tree -r HEAD --name-only | grep -Ei '\.(o|ko|bin|img|dtb|dtbo|elf|ubi|a|so)$' || echo "clean tree"
git rev-list --objects --all | grep -Ei '\.(o|ko|bin|img|dtb|dtbo|elf|ubi|a|so)$' || echo "clean history"

# 2) both licenses present
ls LICENSE-MIT LICENSE-GPL-2.0 NOTICE.md

# 3) the review-form series is reproducible from the target patches, and those patches
#    apply to pristine linux-6.18.60 (needs that tarball)
python3 upstream/series-6.18/tools/gen-series.py --check
upstream/series-6.18/tools/run-validation.sh /path/to/linux-6.18.60.tar.xz /tmp/luofu-val

# 4) image-artifact validator (on an image built from the recipe; not shipped here)
python3 target-skeleton/tools/luofu-image.py check-container <built-sysupgrade.bin>
```

The tree has a **fresh git history** (no history was copied from the research repositories,
so no vendor object that ever existed there is present here). The 2026-11-19 boot evidence
(UART logs, debug image) is private lab evidence and is not part of this repository or its
history.

---

## Safety and legal

- **This is not a flashing guide and not a release.** One attended bench bring-up booted a
  build of this tree to a userspace console on the lab unit (2026-11-19); that build was
  patches 001-017 and the session is not an install or recovery procedure. No artifact has
  ever run a network path, the persistent overlay is host-validated only, and the final
  full-series 001-023 rebuild was not flashed. The published container is marked
  `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED`. If you brick a WR3000 v2.0,
  recovery is physical (power-off + RESET held through the counter + TFTP) and is not covered
  here.
- **Do not flash a WR3000 v1 / MediaTek image** (or any image) to this board. The board is
  the HiSilicon luofu v2 design; wrong images are unrecoverable without a physical recovery
  session.
- **UART on this unit is read-only.** The bench unit's host-to-router serial path is physically
  dead (burnt resistor); the project captured console output read-only. Do not type into a
  UART attached to a board you did not verify.
- **Provenance is GPL-compatible reverse engineering, not "clean-room".** The drivers are
  derived from documented reverse engineering (disassembly, register traces, decoded vendor
  device trees) of the vendor's GPL-declared binaries. No vendor code, binary, blob or
  extracted DTB is redistributed here. The Wi-Fi stack remains vendor-only and its firmware
  is closed.
- **GPL source request:** the vendor ships 58 modules declaring `license=GPL` with no
  published source. A GPLv2 section 3 request was sent to the vendor on 2026-10-01; no reply
  or receipt is held (record: [`docs/GPL-REQUEST.md`](docs/GPL-REQUEST.md)). That record is
  kept for provenance, not as a project gate.
- **Radio regulation:** calibration values, power tables and `txpower` must stay inside the
  regulatory limits of the country of operation. Nothing here instructs otherwise.
- **No warranty.** Host-validated engineering plus one attended bench boot; use at your own
  risk.
- Cudy and HiSilicon are trademarks of their respective owners; this project is independent
  and not affiliated with either company.

---

## Links and provenance

- Lab notebook / live research record: **[cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)**
- Security research (CVE write-ups, affected versions, disclosure status): **[cudy-wr3000v2-cve](https://github.com/ShibbityShwab/cudy-wr3000v2-cve)**
- This repository is a curated, fresh-history snapshot of the lab notebook's `opensource/`
  subtree at commit `8bb7f2e` (2026-11-19: series `001`-`023`, the persistent
  `rootfs_data` overlay volume, the connectivity drafts `022`/`023` and the final host-only
  image), with the publication policy in [`docs/PUBLISHING.md`](docs/PUBLISHING.md) and
  provenance in [NOTICE.md](NOTICE.md).

> **The honest bottom line:** the target, DTS, 23-patch kernel series (all 23 in review form;
> `020`-`023` are do-not-merge drafts), image recipe and A/B path are real host-validated
> engineering, and on 2026-11-19 an attended bring-up booted our 6.18.60 kernel and our
> OpenWrt rootfs to a userspace console - console only, with no network and a tmpfs overlay.
> A later host-only rebuild from the full series adds the persistent overlay volume and the
> gated connectivity code, but none of that has run on hardware. The remaining gates are the
> network data path (RX completion decode + TX egress binding), the overlay/SMP/018-023 set,
> an A/B sysupgrade run, repeated-boot stability and the Wi-Fi source question. Treat it as a
> well-documented starting point for an upstream port, not as a firmware.
