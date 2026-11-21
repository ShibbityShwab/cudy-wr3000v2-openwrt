# NOTICE

This repository is a **curated, fresh-history publication** of engineering for the Cudy WR3000
v2.0 (HiSilicon Hi5671Y "luofu", board R116): host-validated target, driver, image and install
work, plus **two attended bench sessions** - the first userspace boot on 2026-11-19 (a build
from kernel patches `001`-`017`; console only, no network, tmpfs overlay) and the canonical
2-CPU baseline verified on 2026-11-21 (image `974e6ca6`: 2 CPUs, wired LAN through the
documented CPU-copy mirror workaround, SSH, persistent overlay, 15-minute soak with 0
failures, reboot persistence). It is not a firmware release and not a flashing guide: the
install path has never run on a device, the intermediate host-only rebuilds (`001`-`023` and
`001`-`026`) were never flashed or booted, and the image container keeps the
`BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` marker.

## What is in this repository

- Our own documentation: OpenWrt target/device-tree/image design, reverse-engineering
  specifications, the roadmap, the consolidated reversal report and the validation logs.
- Our own tooling: FIP image-slicing/trailer tools, image builders and mocked host tests.
- Our own kernel code (as patches): the 62 L0 patch files of the Linux 6.18 target stack
  (`001-058` + `062-065`), of which the **51 include patches form the release line** and the 11
  bench-only SMP diagnostics are excluded (`upstream/series-6.18/release-line/`); the 26-patch
  upstream review form (`001-026`: 19 merge-intent patches plus the seven fail-closed data-path
  drafts `020`-`026`, marked do-not-merge); the board DTS with the loader hand-off contract and
  the DTS-0001 companion entry; and the schematic OpenWrt target/base-files files.
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
- The Wi-Fi stack (Hi5622V100) is vendor-binary only and its firmware is closed. No open
  Wi-Fi driver exists in this repository and none is claimed.

## License split

| Content | License | SPDX |
| --- | --- | --- |
| Documentation (`README.md`, `docs/`, `NOTICE.md`, `target-skeleton/README.md`, `target-skeleton/docs/`) | MIT — [LICENSE-MIT](LICENSE-MIT) | `MIT` |
| Tools/scripts (`target-skeleton/tools/`, `upstream/series-6.18/tools/`) | MIT — [LICENSE-MIT](LICENSE-MIT) | `MIT` (SPDX header in each file) |
| Kernel code: the target patch stack (001-058 + 062-065), DTS-0001, draft DTS/DTBO, OpenWrt target, image recipe and `base-files/` | GPL-2.0-only — [LICENSE-GPL-2.0](LICENSE-GPL-2.0) | `GPL-2.0-only` (SPDX header in each file) |

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

- **Two attended bench sessions are recorded.** The 2026-11-19 session booted a build of this
  tree (kernel patches `001`-`017`) to a userspace console - console only, no network, tmpfs
  overlay. The 2026-11-21 session ran the canonical 2-CPU image `974e6ca6` (2 CPUs, wired LAN
  through the documented CPU-copy mirror workaround, SSH, persistent ubifs overlay, 15-minute
  soak with 0 failures, reboot persistence). The intermediate host-only full-series rebuilds
  (`001`-`023` and `001`-`026`) were not flashed and not booted, the install path has never run
  on a device, and the image container keeps the
  `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` marker. There are no install or
  recovery instructions here.
- **UART on the bench unit is read-only** (the host→router serial path is physically dead).
  Do not write to a UART on a board whose wiring you have not verified.
- **Do not flash a WR3000 v1 / MediaTek image** to this board; it is the HiSilicon luofu v2
  design and the wrong image is not recoverable without a physical recovery session.
- **Radio regulation:** any experimentation with power tables, calibration values or
  `txpower` must remain inside the regulatory limits of the country of operation. The
  documentation notes where those limits live; it does not authorise exceeding them.
- **No warranty.** Host-validated engineering plus one attended bench boot, provided "as is".

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
