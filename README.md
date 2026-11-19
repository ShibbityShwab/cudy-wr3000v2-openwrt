# cudy-wr3000v2-openwrt

> **One attended hardware bring-up on top of host-validated engineering.** On **2026-11-19** a
> `linux-6.18.60` kernel built from this target tree and an OpenWrt rootfs built from this
> image recipe booted to a userspace console on the WR3000 v2, in an **attended bench
> bring-up** (recovery door armed, UART captured read-only). That is the only hardware result
> in this repository, and it is **not a release**: there is still **no network** and **no
> persistent overlay**, the image running on the unit is the debug build, and the clean
> production image was **rebuilt but not re-flashed**. Everything else here is host-validated
> only; the sysupgrade container is still deliberately marked `NOT FLASHABLE`; and this
> repository contains **no flashing or recovery instructions**. Milestone detail:
> [below](#2026-11-19-hardware-milestone-first-boot-to-userspace).

An upstream-style **OpenWrt / Linux bring-up for the Cudy WR3000 v2.0** - a HiSilicon
**Hi5671Y "luofu"** (board R116) router that ships a vendor OpenWrt 22.03.6 / Linux 5.10.201
image. The goal is a mainline OpenWrt target, device tree and driver set for this board.

This curated repository collects the parts of the research that are **safe and useful to
publish**: a Linux 6.18 kernel patch series (all 21 patches in the upstream review form;
19 merge-intent patches plus two fail-closed data-path drafts marked do-not-merge), an OpenWrt
target skeleton and board DTS, an image/install recipe, reverse-engineering specifications
and measured read-only silicon inventory, and an honest status report.
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
| Kernel series (patches 001-021) | **21/21** apply in order to pristine linux-6.18.60 with `patch(1)` (82 file-patch events, 0 failed hunks, 0 rejects, 0 fuzz, 3 apply offsets) **and with `git am`** (identical trees, clean work tree); the review form carries all 21 patches and keeps checkpatch 0 errors / 53 warnings (6 strict CHECKs), enumerated in [04-validation.md](upstream/series-6.18/04-validation.md); the two draft follow-ups 020/021 are marked do-not-merge | a 6.18.60 kernel built in this target tree booted to userspace on 2026-11-19. The running build is the debug-CRC build used to localise the XZ failure; the clean post-bring-up rebuild was not booted |
| Kernel compile | full `linux-6.18.60` compile of the 001-021 series RC=0 under `CONFIG_WERROR=y`, 0 warnings / 0 errors; zImage 3,279,096 B (`96b927d3...`); load/entry `0x80608000` unchanged (`_text = 0xc0608000`) | the 2026-11-19 boot loaded our kernel at `0x80608000`; the recorded 001-021 zImage (`96b927d3...`) was not the booted artifact |
| Board DTS | compiles with the kernel's own dtc (1.7.2), zero warnings; every new YAML binding passes `dt_binding_check` | a DTB compiled from this DTS was applied by the loader and used by the 2026-11-19 boot (machine model matches); full `dtbs_check` is **not** clean |
| FIP trailer generator | reproduces both recorded vendor containers byte-for-byte; fail-closed on every decoded loader check | a FIP kernel slice assembled from our own base DTB + loader-match overlay + uImage was booted from the inactive slot on 2026-11-19; per-slot A/B trailer handling is still [U] |
| Target image | `make world -j16` RC=0 in an isolated clone; two consecutive builds byte-identical in the first lane; the post-bring-up clean rebuild is 6,185,262 B (sha256 `e893c388...`, `CONFIG_CMDLINE`/`CONFIG_XZ_DEC_ARM` verified, repo tool re-validates it) | a kernel slice + rootfs UBI assembled with this tree's host tooling were written to the inactive pair and booted (2026-11-19 attended). The published sysupgrade container itself was **not** flashed, and the clean rebuild was **not re-flashed** |
| A/B install path | `platform_do_upgrade` runs against mocked MTD/sysfs/ubiformat: 94/94 assertions (100/100 with the real built container) | the fail-closed sequence (inactive pair only, read-backs, semantic UBI verify, selector last) was executed **manually** on the unit on 2026-11-19; `platform_do_upgrade` itself still has never run on a device; the loader env re-stamp remains open |
| Factory-MAC provisioning | bdinfo `+0xde00` derivation + validation: 103/103 mocked assertions | no MAC read from a real unit; no netdev wired (the 2026-11-19 boot had no network) |
| Silicon inventory 2026-11-18 | first attended **read-only** inventory of the stock vendor boot: **600/600** planned register/table reads OK (476 LSW + 108 PIE + 13 CRG + 3 SYSCTRL), 0 failed batches, boot_id unchanged; live post-overlay DT windows, queue/flow DDR base, PIE ring geometry, LSW W1 baseline + qid maps A/B, CRG `clk_summary` and SYSCTRL cells measured; 16 base-DT expectation tokens corrected | no write, no reboot: one boot, read-only; no traffic/behaviour claim |
| CRG clock/reset (004, 014, 019) | compiles; `dt_binding_check`; 16 gates + 2 PLLs + 2 muxes modelled **read-only**; the PLL1 control words were read back in the 2026-11-18 inventory | no rate/parent/register write attempted |
| Watchdog, thermal, pinctrl, PCIe, FMC/NAND, restart (005, 006, 010, 011, 012, 013) | compile + binding checks pass; drivers are built-in in the target config | the 2026-11-19 boot registered the FMC/SPI-NAND (ESMT 128 MiB, 17 partitions, UBI rootfs mounted), watchdog, thermal and restart drivers; pinctrl/PCIe were not exercised |
| SMP bring-up (018) | platsmp draft compiles; vendor release sequence decoded and documented | the 2026-11-19 boot log reports `Brought up 1 node, 1 CPU`, so the CPU1 release path is still unproven on hardware |
| MDIO + PHY (007, 008) | complete driver ports, compile clean | bus/PHY never enumerated |
| Ethernet data path (GEMAC/PIE/LSW, 009/015/016/020/021) | patch `020` turns PIE into a draft ring engine (coherent descriptor rings for the measured two active RX/TX channels, recovered channel/IRQ init, NAPI RX path, TX producer, read-back verification) and GEMAC into a phylink/netdev front end; the engine only arms behind the attended `luofu_pie.arm=1` opt-in and refuses before the first register write otherwise. Patch `021` makes the LSW probe read-only: validated DDR windows, powered-state gate, 30-word W1 baseline and a word-for-word qid-map check, no `writel()`, always fails closed | never ran on hardware; **no PIE or switch register has ever been written** |
| A/B boot selector (017) | sysfs contract reconstructed from local evidence; mocked harness; the record cell was read **read-only** in the 2026-11-18 inventory (`0xc08` = `0x10`, pair A, upper 24 bits zero) | the 2026-11-19 handover moved the selector by a **single write from the stock userspace**, and the loader failover then booted slot B; patch 017's own store path still never ran |
| Wi-Fi (Hi5622V100) | reverse-engineering specification only; no open driver | vendor binary stack; GPL section 3 source request sent 2026-10-01, **no reply held** |
| **On-device validation** | - | **2026-11-19: one attended bring-up reached a userspace console** (kernel 6.18.60 + our rootfs in slot B); no network, no persistent overlay; the clean image was not re-flashed |

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

- **No network.** The LSW/PIE/GEMAC netdevs are not wired yet; the running image is console-only
  and has no SSH. Re-flashing would need the stock userspace or physical TFTP recovery.
- **No persistent overlay.** Our image has no `rootfs_data` volume; `mount_root` falls back
  to a tmpfs overlay. Changes do not survive a reboot.
- **`/lib/functions/board.sh` is missing** for `board_detect` (logged by preinit); the
  board/network setup stage is therefore incomplete.
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

## The Linux 6.18 kernel series (patches 001-021)

The target tree carries the **21-patch** series, and the upstream **review form** now covers
the same 21 patches. Both copies are included, by design:

- [`target-skeleton/target/linux/hisilicon/patches-6.18/`](target-skeleton/target/linux/hisilicon/patches-6.18/) -
  all 21 plain diffs that OpenWrt's patch application consumes (this is what the recorded
  kernel compile evidence used);
- [`upstream/series-6.18/`](upstream/series-6.18/) - the **review form** of patches 001-021:
  cover letter, per-patch commit messages, MAINTAINERS proposal, tree/order rationale, a
  repeatable validation record and raw logs, plus a commit-ready `git am` copy.
  The readable overview is its [README](upstream/series-6.18/README.md).

The review form is not a merge list: `020` (PIE ring engine + GEMAC front end) and `021`
(read-only LSW baseline probe) are included as **draft / do-not-merge** patches documenting
the measured-inventory follow-ups. `020` is an arm-gated data-path draft whose engine has
never run (its RX completion decode is still [U]); `021` is a read-only probe of a block our
own driver still does not program. Both are host-compiled and never booted; the
ready-for-review split and the do-not-merge reasoning are in the series
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

Series facts (all host-only; evidence in [`upstream/series-6.18/validation/`](upstream/series-6.18/validation/)):

- 21/21 apply in order to pristine `linux-6.18.60` with `patch(1)` (82 file-patch events,
  0 failed hunks, 0 rejects, 0 `.rej`) **and with `git am`** (clean work tree; the two
  resulting trees are identical, 0 rejects, 0 fuzz). Three hunks apply with an offset: the
  pre-existing two in `0008`, plus a 6-line offset for `021` after `020` edits the same
  Kconfig file.
- `checkpatch.pl --no-tree`: **0 errors, 53 warnings** (default and strict), plus 6 strict
  CHECKs - review items such as binding split, MAINTAINERS entry and line wrapping, not
  behavioural defects. The 4 new warnings are the expected per-patch review items for
  `020`/`021` (one MAINTAINERS and one binding-split warning each).
- `sha256sum -c` on the recorded `validation/sha256sums.txt`: **42/42 OK** (21 series
  patches + 21 source patches); series `0020` sha256 `5f3e1e52...`, `0021` `7ed09990...`.
- The final 001-021 series is what the LSW-init lane compiled in an isolated `openwrt main`
  clone: `make target/linux/prepare` RC=0 and `make target/linux/compile` RC=0 with
  **0 warnings / 0 errors** under `CONFIG_WERROR=y`; zImage 3,279,096 B, sha256
  `96b927d35e2e6c6be9980c7683de20c20a288a5df58ddf08ed7fbbe9bb1e92d3`. The earlier
  19-patch build gave zImage 3,264,560 B, sha256
  `4456b0ec74533fa26f3a33e3aeec17776006ba251bcca613aeea5e72967d1237`.
- **The booted kernel is not this artifact.** The 2026-11-19 bring-up booted a 6.18.60 build
  of the same target tree (debug-CRC build); the clean rebuild and the review form remain the
  host-validated artifacts above.
- **No submission has been made.** The series is a review artifact, not an upstream-accepted
  change. Its "ready-for-review" split and the owner decisions still required (real
  `Signed-off-by` identity, binding split, folding 0014 into 0004, SMP staging) are listed in
  [`upstream/series-6.18/README.md`](upstream/series-6.18/README.md).

---

## Target and image recipe

The OpenWrt target lives in [`target-skeleton/`](target-skeleton/); its
[README](target-skeleton/README.md) is the per-file status page and host-validation record
(written before the 2026-11-19 boot). The image chain is:

```
target-skeleton/
|- dts/                     # draft board DTS (luofu-r116.dts) + loader-match overlay (.dtso)
|- target/linux/hisilicon/  # OpenWrt target: Makefile, Kconfig, subtarget luofu/
|   |- luofu/               # target.mk, config-5.10, config-6.18, device profile
|   |- patches-6.18/        # the 21 kernel patches (001-021)
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
- **Rootfs** = squashfs in a static UBI volume; the sysupgrade container carries
  `CONTROL`, `kernel`, `root`, `fip-manifest.txt` and `INSTALL-STATUS.txt`.
- **Host build result:** a full `make world -j16` in an isolated `openwrt main` clone
  (commit `27663a0`, linux-6.18.60) completes RC=0. Earlier lanes built 6,185,262 B
  (`54652e56...`) and 6,175,022 B (`ea143b03...`) containers. The post-bring-up clean
  rebuild with the five boot fixes is **6,185,262 B, sha256
  `e893c388e4ab8be61be0f76cd00b003000516005fef3d20bef5b26ecf853c8b5`** (config
  `CMDLINE`/`XZ_DEC_ARM` verified; `check-container` PASS; not re-flashed). The image
  that actually booted is the debug-CRC build (`51baf2ac...`).

> **The published container is marked `NOT FLASHABLE` on purpose.** Nothing in this
> repository is an install or recovery procedure. The 2026-11-19 bring-up was a manual attended
> bench procedure on the lab unit, not a documented install path. Do not flash the container.

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
logs (`build/evidence-firstboot/`) are the same kind of private evidence. See
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

# 3) the kernel patches apply to pristine linux-6.18.60 (needs that tarball)
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
  build of this tree to a userspace console on the lab unit (2026-11-19); that is not an
  install or recovery procedure. There is still no network or persistent overlay, and the
  clean image was not re-flashed. The published container is marked `NOT FLASHABLE`. If you
  brick a WR3000 v2.0, recovery is physical (power-off + RESET held through the counter +
  TFTP) and is not covered here.
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
  subtree at commit `12eb48c` (2026-11-19: series 001-021 plus the first-boot bring-up fixes),
  with the publication policy in [`docs/PUBLISHING.md`](docs/PUBLISHING.md) and provenance in
  [NOTICE.md](NOTICE.md).

> **The honest bottom line:** the target, DTS, 21-patch kernel series (all 21 in review form;
> `020`/`021` are do-not-merge drafts), image recipe and A/B path are real host-validated
> engineering, and on 2026-11-19 an attended bring-up booted our 6.18.60 kernel and our
> OpenWrt rootfs to a userspace console - console only, with no network and a tmpfs overlay.
> The remaining gates are the network data path, a persistent overlay, an A/B sysupgrade run,
> repeated-boot stability and the Wi-Fi source question. Treat it as a well-documented starting
> point for an upstream port, not as a firmware.
