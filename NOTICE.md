# NOTICE

This repository is a **curated, fresh-history publication** of engineering for the Cudy WR3000
v2.0 (HiSilicon Hi5671Y "luofu", board R116): host-validated target, driver, image and install
work, plus **one attended hardware result** - a build of this target booted to a userspace
console on 2026-11-19. It is not a firmware release and not a flashing guide; the booted build
was kernel patches `001`-`017` only, it had no network and no persistent overlay, and later
host-only rebuilds from the full series - `001`-`023` (persistent `rootfs_data` overlay
volume, gated connectivity drafts) and the final `001`-`026` image (`24`-`26` add the PIE
completion/TX-descriptor decode, the gated LSW egress stage and the bdinfo-MAC nvmem source) -
have **never been flashed or booted**. The published container stays marked
`BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED`.

## What is in this repository

- Our own documentation: OpenWrt target/device-tree/image design, reverse-engineering
  specifications, the roadmap, the consolidated reversal report and the validation logs.
- Our own tooling: FIP image-slicing/trailer tools, image builders and mocked host tests.
- Our own kernel code (as patches): the 26-patch Linux 6.18 series, carried in the upstream
  review form as a 001-026 set (19 merge-intent patches plus the seven fail-closed data-path
  drafts `020`-`026`, marked do-not-merge), the draft board DTS and the schematic OpenWrt
  target/base-files files.
- The image recipe's standard OpenWrt **squashfs + `rootfs_data` UBI layout**: the static
  read-only rootfs volume plus an empty dynamic overlay volume (`vol_size=1MiB`,
  `vol_flags=autoresize`) that the kernel's first read-write UBIFS mount formats.
- The five first-boot fixes found by the 2026-11-19 attended bring-up: built-in kernel
  command line, 4 KiB squashfs payload padding before ubinize, the XZ ARM-BCJ decoder config,
  the semantic UBI-volume read-back verifier, and the single-write selector recipe.

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
| Kernel code: patches 001–026, draft DTS/DTBO, OpenWrt target, image recipe and `base-files/` | GPL-2.0-only — [LICENSE-GPL-2.0](LICENSE-GPL-2.0) | `GPL-2.0-only` (SPDX header in each file) |

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

- **One attended bench bring-up booted a build of this tree to a userspace console on
  2026-11-19.** That is the only hardware result, and the booted build was kernel patches
  `001`-`017` only, with no network and no persistent overlay. The later host-only full-series
  rebuilds (the `001`-`023` persistent-overlay image and the final `001`-`026` image with the
  PIE completion, LSW egress and MAC-nvmem follow-ups) were not flashed and not booted; the
  published image remains marked `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED`. There
  are no install or recovery instructions here.
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
