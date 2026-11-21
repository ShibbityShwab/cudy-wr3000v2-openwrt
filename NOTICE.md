# NOTICE

This repository is a **curated, fresh-history publication** of engineering for the Cudy WR3000
v2.0 (HiSilicon Hi5671Y "luofu", board R116): host-validated target, driver, image and install
work, plus **three bench phases** - the first userspace boot on 2026-11-19 (a build from
kernel patches `001`-`017`; console only, no network, tmpfs overlay); the canonical 2-CPU
baseline verified on 2026-11-21 (image `974e6ca6`: 2 CPUs, wired LAN through the documented
CPU-copy mirror workaround, SSH, persistent overlay, 15-minute soak with 0 failures, reboot
persistence); and the same-day board-feature and quiet images (`7f270b8c` / **`7421bb7a`
(known-good)**: six writable `oem:green:*` LEDs, `event0` gpio-keys-polled, `watchdog0`
sysfs, periodic dmesg spam removed), which required the base-DTB `pad-dtb` alignment step.
The **stock-mediated install path is verified** - manually and with the published bench
stock-telnet installer - writing only the inactive pair and booting our 2-CPU image; the
**known-good baseline `7421bb7a` was reproduced byte-for-byte** by the conservative
comment-only DTS cleanup. The superseded aggressive DTS cleanup (image `6f19c6d6`) is a
recorded hardware regression: it passed the host checks but did not boot (MTD partition scan
stalled, U-Boot watchdog reset, stock fallback). It is not a firmware release and not a general
flashing/recovery guide: no install image is published, `platform_do_upgrade` has never run
on a device, the intermediate host-only rebuilds (`001`-`023` and `001`-`026`) were never flashed
or booted, and the image container keeps the
`BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` marker. Thermal telemetry is unsupported
on this board, Wi-Fi is unsupported on the 6.18 target, and only one Ethernet jack works through
the CPU-copy workaround (multi-port is a separate project). A **documentation-only upstream
submission package** (RFC form, artifact checklist, cover-email draft, explicit no-Tested-by
policy, file manifest) is included for owner review; **nothing has been submitted anywhere**.
A **bench-only multi-port source-port probe patch (066)** and probe image (**a91ebcf0**) are
retained as an experiment record - neither is part of the release line or the upstream series.
The experiment was run on 2026-11-21 and was **negative**: the CPU-copy completions carry no
source-port identity (identical descriptors across jacks), so multi-port LAN/WAN requires the
native L2/VLAN hardware bridge and the single-port workaround stands. The devicetree final also
adopts the **boot-tested TWD level-high delta** (upstream blocker 6 closed; boot-tested image
`f6685b80`): DTS sha256 `13427d8e...` (700 lines), DTS-0001 `e9c9ed58...` (713 lines), DTB
`4fde76df...` / pad `4b50de73...`; **five upstream blockers remain**. The owner-only legacy
history-purge runbook is prepared/dry-run validated **notebook-side** (not published here) and
`PUBLISH-READINESS.md` item 1 remains unchecked - no history rewrite or push has run.

## What is in this repository

- Our own documentation: OpenWrt target/device-tree/image design, reverse-engineering
  specifications, the roadmap, the consolidated reversal report and the validation logs.
- The **retained read-only device reports**: the 2-CPU feature inventory, its follow-up
  update (`target-skeleton/docs/incoming/feature-inventory/`) and the Hi5622V100 Wi-Fi
  feasibility recon (`target-skeleton/docs/incoming/wifi-recon/`). All are text-only; no vendor
  binary, firmware blob, credential or per-unit identifier is included, and the inventory's one
  MAC placeholder uses the documentation prefix `02:00:00:xx:xx:xx`. Their curated notes record
  which findings were later fixed (LEDs/keys/watchdog), root-caused (thermal) or withdrawn
  (the MAC-duplication concern), the verified install path and the multi-port limit.
- The multi-port Ethernet limit and the attended source-port experiment plan:
  [`docs/MULTI-PORT-PLAN.md`](docs/MULTI-PORT-PLAN.md).
- The **documentation-only upstream submission package** (`upstream/series-6.18/submission/`):
  package README/checklist, one-page v3 RFC cover-email draft, explicit no-Tested-by policy and
  `MANIFEST.sha256` covering the review-form documents, DTS-0001, release-line classification,
  tooling and validation logs. No transport, account or list subscription is included; nothing
  has been sent. The manifest in this repository was regenerated against the curated tree.
- The **retained upstream blocker-reduction drafts** (`docs/incoming/upstream-blockers/`):
  documentation + patch drafts only (flashinfo binding, MAC-nvmem removal, switch MMIO-first
  reorder, compatible ordering, driver/YAML review); blocker 6 (TWD) is closed; they are not the
  release series, not applied, and each active delta needs its own hardware boot-test.
- The **bench-only multi-port probe record** (`docs/incoming/multi-port-probe/`): patch 066
  (a draft kernel diff adding `luofu_switch.mirror_src_mask`) and the prepared probe image
  `a91ebcf0`. It is not part of the release line or review form; the experiment plan, the
  **negative measured result** (no source-port id in the CPU-copy completions; native L2/VLAN
  bridge required) and the decision gate are in `docs/MULTI-PORT-PLAN.md`.
- Our own tooling: FIP image-slicing/trailer tools, the base-DTB `pad-dtb` build guard, image
  builders and mocked host tests, plus the verified bench **stock-telnet installer**
  (`tools/stock-telnet-install.sh` + `tools/telnet-exec.py`; fail-closed, inactive-pair
  only, published for evidence - see `tools/README.md`).
- Our own kernel code (as patches): the 62 L0 patch files of the Linux 6.18 target stack
  (`001-058` + `062-065`), of which the **51 include patches form the release line** and the 11
  bench-only SMP diagnostics are excluded (`upstream/series-6.18/release-line/`); the 26-patch
  upstream review form (`001-026`: 19 merge-intent patches plus the seven fail-closed data-path
  drafts `020`-`026`, marked do-not-merge); the board DTS with the loader hand-off contract,
  the stock board_id=3 LED/keys nodes, the **conservative comment-only cleanup plus the
  boot-tested TWD level-high delta** (DTS 700 lines `13427d8e...`; hardware baseline image
  `f6685b80`) and the DTS-0001 companion entry (`e9c9ed58...`, 713 lines); and the schematic
  OpenWrt target/base-files files. The release-line hashes for patches 054/056 were refreshed
  after the `dev_dbg` quieting (manifest sha256 `4da8b960...`). The retained
  [DTS cleanup record](upstream/series-6.18/07-dts-upstream-bindings.md) documents the boot
  regression of the superseded active cleanup, the TWD adoption, and the five remaining blockers.
- The image recipe's standard OpenWrt **squashfs + `rootfs_data` UBI layout**: the static
  read-only rootfs volume plus an empty dynamic overlay volume (`vol_size=1MiB`,
  `vol_flags=autoresize`) that the kernel's first read-write UBIFS mount formats.
- The five first-boot fixes found by the 2026-11-19 attended bring-up: built-in kernel
  command line, 4 KiB squashfs payload padding before ubinize, the XZ ARM-BCJ decoder config,
  the semantic UBI-volume read-back verifier, and the single-write selector recipe.
- The release-critical SMP fix (2026-11-21): the board-DTS **loader hand-off contract**
  (`/aliases/console` and `reserved-memory/flashinfo@80600000`, reg `0x80600000`+`0x4000`,
  `atag-offset = <0x1000>`), shipped as the explicit DTS-series entry DTS-0001 (companion to
  kernel patch 0018; see `upstream/series-6.18/06-dts-series-dependency.md`).
- The board-feature work (2026-11-21): gpio-keys-polled (reset/WPS) and six `oem:green:*`
  gpio-leds in the single-source DTS, the matching config additions (LEDs/input/watchdog
  sysfs), and the **base-DTB padding fix** - the stock loader's gate-word load faults on an
  unaligned FDT totalsize, so `Build/luofu-pad-dtb` (`target-skeleton/tools/pad-dtb.py`) pads
  the base DTB to 13,048 B and folds the padding into the totalsize (TRAILER-FORMAT
  section 11).

## What is deliberately **not** in this repository

- vendor firmware images, rootfs blobs or sysupgrade containers,
- vendor kernel modules (`.ko`/`.o`), TR6560 BSP objects, linker objects or build trees,
- the vendor Wi-Fi firmware blob and any extracted vendor DTB/DTBO bytes,
- NAND/rootfs/U-Boot dumps, calibration data or register dumps,
- per-unit identifiers (MAC addresses, serial numbers, PINs, efuse/`secret` values, UUIDs),
- credentials, keys, tokens or any live values from the bench unit,
- build artifacts and local evidence directories.

The tooling is written so that an owner of the unit can re-derive the local inputs themselves;
the repository never needs the vendor's binaries to be useful. Anything referenced in the docs
as a path under `build/`, `dumps/`, `firmware/`, `rootfs-*/` or `parent:` is
private lab evidence and is **not** published here.

## Provenance of the reverse-engineered code

The drivers, device tree and image recipe were produced by **documented reverse engineering**
of the vendor's binaries: disassembly of the vendor's GPL-declared kernel modules, captured
register/hardware behaviour, and the vendor device trees. They are **not** "clean-room" work,
and this repository does not claim they are.

- The vendor ships 58 kernel modules that declare `license=GPL` but publishes no source.
  The derived code is therefore released under **GPL-2.0-only**, with attribution to that
  provenance, so it stays compatible with the vendor's GPL obligations.
- **No vendor code is copied or redistributed here.** The patches contain our own expression
  of the interfaces, encoded from measured values and decoded structures. No vendor bytes
  (firmware, DTBs, object code) are embedded in the tools or documentation.
- The GPLv2 section 3 source request to the vendor was sent on 2026-10-01 and no reply is
  held; the record is [docs/GPL-REQUEST.md](docs/GPL-REQUEST.md). This is a provenance note,
  not a claim that the request has been answered.
- The Wi-Fi stack (Hi5622V100) is vendor-binary only (a closed cfg80211 full-MAC driver, not
  mac80211) and its firmware is closed. No open, mainline or OpenWrt-6.18 Wi-Fi driver exists;
  Wi-Fi is **declared unsupported on the 6.18 target** and only the vendor 5.10 kernel stack is
  Wi-Fi-capable. Thermal telemetry is likewise unsupported on this board (the temperature
  register reads 0 and the vendor DT disables the sensor).

## License split

| Content | License | SPDX |
| --- | --- | --- |
| Documentation (`README.md`, `docs/`, `NOTICE.md`, `target-skeleton/README.md`, `target-skeleton/docs/`) | MIT — [LICENSE-MIT](LICENSE-MIT) | `MIT` |
| Tools/scripts (`tools/` bench installer, `target-skeleton/tools/`, `upstream/series-6.18/tools/`) | MIT — [LICENSE-MIT](LICENSE-MIT) | `MIT` (SPDX header in each file) |
| Kernel code: the target patch stack (001-058 + 062-065), DTS-0001, the bench-only probe patch 066, draft DTS/DTBO, OpenWrt target, image recipe and `base-files/` | GPL-2.0-only — [LICENSE-GPL-2.0](LICENSE-GPL-2.0) | `GPL-2.0-only` (SPDX header in each file) |

Notes:

- The MIT/GPL split follows the content, not the directory. New kernel source added by a
  patch carries its `GPL-2.0-only` SPDX header inside the patch hunks; the patch files
  themselves are kernel-code diffs and are GPL-2.0-only.
- In the source lab tree some of these tools carried `GPL-2.0-only` SPDX headers; they are
  the project's own code and are published here under MIT as documented above, which is
  compatible with their use in this repository. The kernel code license is unchanged.
- Copyright (c) 2026 ShibbityShwab, except where a file says otherwise.

## No vendor affiliation

Cudy, HiSilicon and other names are trademarks of their respective owners. This project is
independent research and is not affiliated with, endorsed by, or supported by Cudy or
HiSilicon.

## Safety, radio and legal

- **Three bench phases are recorded.** The 2026-11-19 phase booted a build of this tree (kernel
  patches `001`-`017`) to a userspace console - console only, no network, tmpfs overlay. The
  2026-11-21 canonical phase ran image `974e6ca6` (2 CPUs, wired LAN through the documented
  CPU-copy mirror workaround, SSH, persistent ubifs overlay, 15-minute soak with 0 failures,
  reboot persistence). The same-day board-feature phase ran images `7f270b8c` and
  **`7421bb7a` (known-good)** (six LEDs, `event0` gpio-keys-polled, `watchdog0` sysfs,
  dmesg spam removed; thermal unsupported), then the automated stock-telnet install was verified
  end-to-end (inactive pair only, active pair untouched, selector verified, 2-CPU image booted).
  The intermediate host-only full-series rebuilds (`001`-`023` and `001`-`026`) were not
  flashed and not booted, and no install image is published; the image container keeps the
  `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` marker. The bench installer in
  `tools/` is published for evidence, not as a general install/recovery guide - it needs the
  vendor stock firmware and the notebook's bench-link/recovery helpers. A recovery is physical
  (power-off + RESET through the counter + TFTP) and is not covered here.
- **The upstream submission package is a dry run.** It is documentation for the owner to review
  and send manually if they choose; it contains no transport and no Tested-by/Reviewed-by is
  claimed. Do not present target-image bench evidence as an upstream Test-by.
- **The multi-port probe patch is bench-only.** Image `a91ebcf0` was built for an attended
  experiment (built-in `luofu_pie.rxdiag=1` / `luofu_switch.mirror_src_mask=0x1e00` tokens);
  it is not a release artifact. The experiment was run on 2026-11-21 and was **negative** (the
  CPU-copy completions are field-for-field identical across jacks / carry no source-port id);
  multi-port LAN/WAN therefore requires the native L2/VLAN hardware bridge.
- **The legacy history-purge runbook is notebook-side and owner-only.** The prepared,
  dry-run-validated `PUBLISH-PURGE-RUNBOOK.md` (legacy repo: 795 commits, eleven binary blob
  paths including an accidental bytecode file) is **not published in this repository** and has
  **not been executed**; `PUBLISH-READINESS.md` item 1 stays unchecked until a purge runs and a
  fresh clone of the public remote verifies clean. No force-push or cache purge has been done.
  This curated repository is unaffected (fresh history, zero binary blobs).
- **An active-DTS change must be boot-tested.** The superseded aggressive DTS cleanup (image
  `6f19c6d6`) passed every host check but did not boot (MTD partition registration stalled,
  U-Boot watchdog reset, stock fallback); it was reverted. The accepted conservative cleanup
  is comment-only and compiles to the hardware-verified DTB. Host checks are necessary but not
  sufficient.
- **UART on the bench unit is read-only** (the host→router serial path is physically dead).
  Do not write to a UART on a board whose wiring you have not verified.
- **Do not flash a WR3000 v1 / MediaTek image** to this board; it is the HiSilicon luofu v2
  design and the wrong image is not recoverable without a physical recovery session.
- **Radio regulation:** any experimentation with power tables, calibration values or
  `txpower` must remain inside the regulatory limits of the country of operation. The
  documentation notes where those limits live; it does not authorise exceeding them.
- **No warranty.** Host-validated engineering plus three bench phases on one unit, provided "as is".

## Publication policy

This work is intentionally public (owner decision, for portfolio/career purposes). It is
published as separate, curated repositories so that each contains one coherent, licensable
artifact set and no private lab material:

- **cudy-wr3000v2-openwrt** (this repository) — the host-validated OpenWrt/Linux port work.
- **[cudy-wr3000v2-cve](https://github.com/ShibbityShwab/cudy-wr3000v2-cve)** — sanitized
  security research (CVE write-ups, affected versions, disclosure status, responsible-use
  notes).
- **[cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)** — the research lab
  notebook (evidence, tools, analysis; no private unit data).

See [docs/PUBLISHING.md](docs/PUBLISHING.md) for the curation rules, the excluded material and
the verification commands.
