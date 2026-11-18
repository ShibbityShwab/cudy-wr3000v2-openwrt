# cudy-wr3000v2-openwrt

> **Host-validated only. Nothing in this repository has been booted or flashed on hardware.**
> Every artifact here was produced on a Linux host from documentation, static analysis and
> clean host builds. None of the artifacts in this repository was ever installed on the board;
> the *separate* bench-reversal track did boot lab kernels on it (that record lives in the lab
> notebook, not here). The built image carries a `NOT FLASHABLE` marker by design, and this
> repository deliberately contains no flashing instructions.

An upstream-style **OpenWrt / Linux bring-up for the Cudy WR3000 v2.0** — a HiSilicon
**Hi5671Y "luofu"** (board R116) router that ships a vendor OpenWrt 22.03.6 / Linux 5.10.201
image. The goal is a mainline OpenWrt target, device tree and driver set for this board.

This curated repository collects the parts of the research that are **safe and useful to
publish**: a Linux 6.18 kernel patch series (a 19-patch upstream review form plus two
fail-closed data-path drafts), an OpenWrt target skeleton and board DTS, an image/install
recipe, reverse-engineering specifications and measured read-only silicon inventory, and an
honest status report.
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

## Status at a glance (2026-11-18)

| Area | Host-validated (evidence in this repository) | Hardware status |
| --- | --- | --- |
| Target/subtarget metadata | `make defconfig` + `make target/linux/prepare` RC=0 on `openwrt main` `27663a0` (linux-6.18.60); generated `CONFIG_TARGET_hisilicon_luofu_DEVICE_cudy_wr3000-v2=y` | never built on a device |
| Kernel series (patches 001–021) | **21/21** apply in order to pristine linux-6.18.60 with `patch(1)`: 82 files patched, 0 failed hunks, 0 rejects; the review-form subset 001–019 also applies with `git am` (identical trees, 0 rejects, 0 fuzz) and keeps checkpatch 0 errors / 49 warnings, while the two new drafts are checkpatch 0/0/0 on their files | no kernel from this work has ever booted the board |
| Kernel compile | full `linux-6.18.60` compile of the 001–021 series RC=0 under `CONFIG_WERROR=y`, 0 warnings / 0 errors; zImage 3,279,096 B (`96b927d3…`); load/entry `0x80608000` unchanged (`_text = 0xc0608000`) | never booted |
| Board DTS | compiles with the kernel's own dtc (1.7.2), zero warnings; every new YAML binding passes `dt_binding_check` | never used to boot; full `dtbs_check` is **not** clean |
| FIP trailer generator | reproduces both recorded vendor containers byte-for-byte; fail-closed on every decoded loader check | **no generated trailer has been booted** |
| Target image | `make world -j16` RC=0 in an isolated clone; sysupgrade container 6,185,262 B (sha256 `54652e56…`); two consecutive builds byte-identical in the first lane | **never flashed**; container says `NOT FLASHABLE` |
| A/B install path | `platform_do_upgrade` runs against mocked MTD/sysfs/ubiformat: 94/94 assertions (100/100 with the real built container) | never wrote flash; one attended A/B validation and the env re-stamp are open |
| Factory-MAC provisioning | bdinfo `+0xde00` derivation + validation: 103/103 mocked assertions | no MAC read from a real unit; no netdev wired yet |
| Silicon inventory 2026-11-18 | first attended **read-only** inventory of the stock vendor boot: **600/600** planned register/table reads OK (476 LSW + 108 PIE + 13 CRG + 3 SYSCTRL), 0 failed batches, boot_id unchanged; live post-overlay DT windows, queue/flow DDR base, PIE ring geometry, LSW W1 baseline + qid maps A/B, CRG `clk_summary` and SYSCTRL cells measured; 16 base-DT expectation tokens corrected | no write, no reboot: one boot, read-only; no traffic/behaviour claim |
| CRG clock/reset (004, 014, 019) | compiles; `dt_binding_check`; 16 gates + 2 PLLs + 2 muxes modelled **read-only**; the PLL1 control words were read back in the 2026-11-18 inventory | no rate/parent/register write attempted |
| Watchdog, thermal, pinctrl, PCIe, FMC/NAND, restart (005, 006, 010, 011, 012, 013) | compile + binding checks pass; drivers are built-in in the target config | none has probed silicon |
| SMP bring-up (018) | platsmp draft compiles; vendor release sequence decoded and documented | CPU1 has never been released |
| MDIO + PHY (007, 008) | complete driver ports, compile clean | bus/PHY never enumerated |
| Ethernet data path (GEMAC/PIE/LSW, 009/015/016/020/021) | patch `020` turns PIE into a draft ring engine (coherent descriptor rings for the measured two active RX/TX channels, recovered channel/IRQ init, NAPI RX path, TX producer, read-back verification) and GEMAC into a phylink/netdev front end; the engine only arms behind the attended `luofu_pie.arm=1` opt-in and refuses before the first register write otherwise. Patch `021` makes the LSW probe read-only: validated DDR windows, powered-state gate, 30-word W1 baseline and a word-for-word qid-map check, no `writel()`, always fails closed | never ran on hardware; **no PIE or switch register has ever been written** |
| A/B boot selector (017) | sysfs contract reconstructed from local evidence; mocked harness; the record cell was read **read-only** in the 2026-11-18 inventory (`0xc08` = `0x10`, pair A, upper 24 bits zero) | SYSCTRL cell never written by our code |
| Wi-Fi (Hi5622V100) | reverse-engineering specification only; no open driver | vendor binary stack; GPL §3 source request sent 2026-10-01, **no reply held** |
| **On-device validation** | — | **none. Nothing has been booted or flashed.** |

The one-line summary: the **design/build layer is real and host-validated**; the
**hardware layer is entirely unverified**. Several drivers are deliberately refusing drafts or
arm only behind an attended opt-in, one read-only inventory of the stock vendor boot now anchors
the specs, and the Wi-Fi stack is externally blocked. Do not read this repository as a working
firmware.

---

## Hardware

| Item | Value |
| --- | --- |
| Board | Cudy WR3000 v2.0, HiSilicon "luofu" v2-class design, board **R116**. The enclosure label may say WR3000 **v1**: the PCB is the luofu design, **not** the MediaTek MT7981 board that other "WR3000 v1" documentation describes. A WR3000 v1 / MT7981 image cannot boot this unit. |
| SoC | HiSilicon **Hi5671Y**, 2× Arm Cortex-A9 (ARMv7-A), platform `hsan luofu` |
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

## The Linux 6.18 kernel series (patches 001–021)

The target tree carries the **21-patch** series; the upstream **review form** covers the first
**19** patches. Both copies are included, by design:

- [`target-skeleton/target/linux/hisilicon/patches-6.18/`](target-skeleton/target/linux/hisilicon/patches-6.18/) —
  all 21 plain diffs that OpenWrt's patch application consumes (this is what the recorded
  kernel compile evidence used);
- [`upstream/series-6.18/`](upstream/series-6.18/) — the **review form** of patches 001–019:
  cover letter, per-patch commit messages, MAINTAINERS proposal, tree/order rationale, a
  repeatable validation record and raw logs, plus a commit-ready `git am` copy.
  The readable overview is its [README](upstream/series-6.18/README.md).

Patches `020` (PIE ring engine + GEMAC front end) and `021` (read-only LSW baseline probe)
are target-tree drafts only: they are deliberately **not** in the upstream review form,
because `020` is an arm-gated data-path draft and `021` is a read-only probe of a block
our own driver still does not program. Both are host-compiled and never booted.

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
| 0020 | net: ethernet: hisilicon: add the Luofu PIE datapath and GEMAC front end | extends `015`'s PIE driver into a draft ring engine (coherent descriptor rings for the measured two active RX/TX channels, recovered v250 channel/IRQ init, mask/unmask + NAPI RX path, TX producer with poll-only completion, read-back verification that tears down on mismatch) and `009`'s GEMAC into a draft phylink/netdev front end; the data path arms only behind the kernel-command-line opt-in `luofu_pie.arm=1` and refuses before the first register write otherwise | target-only draft; do not merge |
| 0021 | net: ethernet: hisilicon: add the Luofu switch read-only baseline probe | extends `016` to the measured 17-window model (two engine DDR windows validated, never mapped), the CRG powered-state gate (gate word bits + four active-low reset lines), a 30-word W1 register baseline and a word-for-word read-only check of both qid maps; contains no `writel()`, registers no netdev, always fails closed | target-only draft; do not merge |

Series facts (all host-only; evidence in [`upstream/series-6.18/validation/`](upstream/series-6.18/validation/)):

- 21/21 apply in order to pristine `linux-6.18.60` with `patch(1)`: 82 files patched,
  0 failed hunks, 0 rejects, 0 `.rej`; `021` applies after `020` with a 6-line offset in a
  Kconfig file. The review-form subset 001–019 also applies with `git am`; both trees are
  identical, 0 rejects, 0 fuzz.
- `checkpatch.pl --no-tree`: **0 errors, 49 warnings** (default and strict), plus 6 strict
  CHECKs for the review-form series — review items such as binding split, MAINTAINERS entry
  and line wrapping, not behavioural defects. The two new drafts are checkpatch **0/0/0** on
  their added/changed files.
- The final 001–021 series is what the LSW-init lane compiled in an isolated `openwrt main`
  clone: `make target/linux/prepare` RC=0 and `make target/linux/compile` RC=0 with
  **0 warnings / 0 errors** under `CONFIG_WERROR=y`; zImage 3,279,096 B, sha256
  `96b927d35e2e6c6be9980c7683de20c20a288a5df58ddf08ed7fbbe9bb1e92d3`. The earlier
  19-patch build gave zImage 3,264,560 B, sha256
  `4456b0ec74533fa26f3a33e3aeec17776006ba251bcca613aeea5e72967d1237`.
- **No submission has been made.** The series is a review artifact, not an upstream-accepted
  change. Its "ready-for-review" split and the owner decisions still required (real
  `Signed-off-by` identity, binding split, folding 0014 into 0004, SMP staging) are listed in
  [`upstream/series-6.18/README.md`](upstream/series-6.18/README.md).

---

## Target and image recipe

The schematic OpenWrt target lives in [`target-skeleton/`](target-skeleton/) (status
`SCHEMATIC / UNTESTED`; its [README](target-skeleton/README.md) is the detailed status page).

```
target-skeleton/
├── dts/                     # draft board DTS (luofu-r116.dts) + loader-match overlay (.dtso)
├── target/linux/hisilicon/  # OpenWrt target: Makefile, Kconfig, subtarget luofu/
│   ├── luofu/               # target.mk, config-5.10, config-6.18, device profile
│   ├── patches-6.18/        # the 21 kernel patches (001–021)
│   ├── image/               # image recipe: luofu.mk, ubinize config, Makefile
│   └── base-files/          # 02_network, platform.sh (sysupgrade), luofu-mac.sh, boot pair recorder
└── tools/                   # fip-slice.py, luofu-image.py, build-kernel-slice.sh, mocked tests
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
  byte-for-byte. A generated trailer is structurally validated only: **it has never been
  booted**.
- **Rootfs** = squashfs in a static UBI volume; the sysupgrade container carries
  `CONTROL`, `kernel`, `root`, `fip-manifest.txt` and `INSTALL-STATUS.txt`.
- **Host build result:** a full `make world -j16` in an isolated `openwrt main` clone
  (commit `27663a0`, linux-6.18.60) completes RC=0. The most recent full-image build used the
  001–018 series plus the MAC-provisioning files and produced a **6,185,262 B** sysupgrade
  container, sha256
  `54652e56241e26eba0e95bc7fa663950c6e69ddb834dba77beb5b1afbeea2cac`. Earlier lanes built
  6,175,022 B (`ea143b03…`) and further 6,185,262 B rebuilds (`e869f105…`, `362ca4ec…`);
  the 001–021 series post-dates the last full-image build, so a rebuild is pending.
  In the first image lane, two consecutive `make world` runs produced a byte-identical image.

> **The container is marked `NOT FLASHABLE` on purpose.** Nothing in this repository is an
> install or recovery procedure. Do not flash it.

---

## A/B install path (host-tested, never executed on a device)

The stock loader keeps two kernel/rootfs slot pairs (`a`/`b`) and selects one through the
environment `bootflag`; a SYSCTRL boot-register cell records the pair and a failed-boot
record. The design ([`docs/SELECTOR-0x21.md`](docs/SELECTOR-0x21.md)) reconstructs the
contract from local evidence: **`0x21` is the project's bench marker, never a vendor value,
and it never persists** — the vendor userspace writes only `0x10`/`0x11`.

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

Host validation only: `tools/test-platform-abinstall.sh` passes **94/94** assertions against
mocked MTD/sysfs/tools (100/100 with the real built container), and
`tools/test-mac-provisioning.sh` passes **103/103**. A real write has never happened; one
attended A/B validation and the environment re-stamp remain unverified.

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
`firmware/` are private lab evidence that is **not** published here. See
[`docs/README.md`](docs/README.md) for the documentation index and the link conventions.

---

## Repository layout

```
README.md                  # this page
LICENSE-MIT                # documentation and tools
LICENSE-GPL-2.0            # kernel code (patches, DTS, OpenWrt target/base-files)
NOTICE.md                  # provenance, no-vendor-code policy, licenses, radio/legal notes
docs/                      # engineering design, RE specifications, publication policy
├── TARGET-DESIGN.md       # target/DTS/image design + per-driver upstream distance
├── ROADMAP-OPENWRT.md     # phase plan and milestones (marked where superseded)
├── GPL-REQUEST.md         # GPLv2 §3 source-request record (provenance; unanswered)
├── SELECTOR-0x21.md       # A/B boot-selector contract reconstruction
├── TRAILER-FORMAT.md      # FIP trailer byte format (copy; canonical next to the tool)
├── MISC-SOC-BLOCKS.md     # efuse/MAC, RNG, PWM, SYSCTRL/SYSENV RE
├── INVENTORY-2026-11-18.md    # measured read-only silicon inventory (600/600 reads)
├── BENCH-READONLY-INVENTORY.md # attended read-only inventory procedure + gates
├── LSW-SPEC.md            # internal LAN switch RE specification
├── PIE-DATAPATH.md        # packet-engine/GEMAC data-path RE specification
├── SMP-RE.md              # CPU1 release/SMP RE + staged bring-up plan
├── REMAINING-DRIVERS.md   # remaining-driver evidence survey and port plans
├── SKELETON-SMOKE.md      # full host-validation session log (the detailed record)
├── REVERSAL-REPORT.md     # consolidated bench-reversal report (from the lab notebook)
├── PUBLISHING.md          # what is in this curated repo, what is excluded, and why
└── README.md              # documentation index + link conventions
target-skeleton/           # schematic OpenWrt target, DTS, image recipe, tools, tests
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
```

The tree has a **fresh git history** (no history was copied from the research repositories,
so no vendor object that ever existed there is present here).

---

## Safety and legal

- **This is not a flashing guide and not a release.** It contains no install or recovery
  procedure; the image is marked `NOT FLASHABLE`; nothing has been booted or flashed. If you
  brick a WR3000 v2.0, recovery is physical (power-off + RESET held through the counter +
  TFTP) and is not covered here.
- **Do not flash a WR3000 v1 / MediaTek image** (or any image) to this board. The board is
  the HiSilicon luofu v2 design; wrong images are unrecoverable without a physical recovery
  session.
- **UART on this unit is read-only.** The bench unit's host→router serial path is physically
  dead (burnt resistor); the project captured console output read-only. Do not type into a
  UART attached to a board you did not verify.
- **Provenance is GPL-compatible reverse engineering, not "clean-room".** The drivers are
  derived from documented reverse engineering (disassembly, register traces, decoded vendor
  device trees) of the vendor's GPL-declared binaries. No vendor code, binary, blob or
  extracted DTB is redistributed here. The Wi-Fi stack remains vendor-only and its firmware
  is closed.
- **GPL source request:** the vendor ships 58 modules declaring `license=GPL` with no
  published source. A GPLv2 §3 request was sent to the vendor on 2026-10-01; no reply or
  receipt is held (record: [`docs/GPL-REQUEST.md`](docs/GPL-REQUEST.md)). That record is kept
  for provenance, not as a project gate.
- **Radio regulation:** calibration values, power tables and `txpower` must stay inside the
  regulatory limits of the country of operation. Nothing here instructs otherwise.
- **No warranty.** Host-validated schematics and measurements only; use at your own risk.
- Cudy and HiSilicon are trademarks of their respective owners; this project is independent
  and not affiliated with either company.

---

## Links and provenance

- Lab notebook / live research record: **[cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)**
- Security research (CVE write-ups, affected versions, disclosure status): **[cudy-wr3000v2-cve](https://github.com/ShibbityShwab/cudy-wr3000v2-cve)**
- This repository is a curated, fresh-history snapshot of the lab notebook's `opensource/`
  subtree at commit `6bcea12` (2026-11-18, series 001–021), with the publication policy in
  [`docs/PUBLISHING.md`](docs/PUBLISHING.md) and provenance in [NOTICE.md](NOTICE.md).

> **The honest bottom line:** the target, DTS, 21-patch kernel series (19 in review form),
> image recipe and A/B path are real host-validated engineering. **Nothing here has run on the router.** The
> hardware gates — one attended boot, one A/B install, the Wi-Fi source question — are still
> open, and the Wi-Fi/LAN-switch work is not close. Treat it as a well-documented
> starting point for an upstream port, not as a firmware.
