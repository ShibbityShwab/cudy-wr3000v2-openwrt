# Publishing notes for cudy-wr3000v2-openwrt

This repository is a **curated, fresh-history publication** of the host-validated OpenWrt/Linux
port work for the Cudy WR3000 v2.0 (HiSilicon Hi5671Y "luofu", board R116). This page records
what was taken from where, what was deliberately left out, how the licenses are split, how the
material was sanitized, and how to verify the publication claims.

## Source and snapshot pin

- Source tree: the research lab notebook's `opensource/` subtree
  ([cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2) at its `opensource`
  submodule), **commit `7475fc5`** ("upstream: extend review-form series to 001-026 and
  revalidate", 2026-11-19; kernel-patch series `001`-`026` and the upstream review form for
  001-026, 26 patches validated). It adds the final `024`-`026` follow-ups (PIE RX
  completion/TX descriptor decode, gated LSW W4 egress/init stage, read-only bdinfo-MAC nvmem)
  on top of the previous curated snapshots at source commits `8bb7f2e` (2026-11-19: 23-patch
  review form, persistent `rootfs_data` overlay volume, connectivity drafts `022`/`023`),
  `12eb48c` (2026-11-19: first-boot bring-up machinery, series 001-021) and `9d5005c`
  (2026-11-18: 21-patch review form).
  The **fifth sync (2026-11-21)** moves the pin to the release-line consolidation:
  `opensource` submodule commit `a6b46a9` (notebook parent `a66a14f`), which adds the
  51-patch release line, the DTS-only loader hand-off SMP fix, DTS-0001, the canonical
  2-CPU image `974e6ca6` and the updated docs - see the fifth-sync section below.
- This repository was created fresh: `git init`, curated copy, new initial commit(s). **No git
  history was copied from the source repositories**, so no object that ever existed there
  (including the ten vendor `.o` files that were removed from the old repository's HEAD on
  2026-11-17) exists here.
- Curated copies may differ from the source tree only by: (a) added publication/provenance
  pages (`README.md`, `NOTICE.md`, `docs/PUBLISHING.md`, this index, licenses, `.gitignore`),
  (b) absolute host paths sanitized to `~` and the project SSH-key filename genericised,
  (c) tool SPDX headers changed to MIT per the license split in [`NOTICE.md`](../NOTICE.md),
  (d) one documentation copy under `docs/` whose relative links were adjusted,
  (e) in `upstream/series-6.18/`, the repo-layout adaptation for this repository: the series
  generator (`tools/gen-series.py`) locates `target-skeleton/` in either layout and its
  mapping table, the review-form README and the `validation/sha256sums.txt` comment name
  `target-skeleton/...` (in the notebook this tree is `opensource/target-skeleton/`).  The
  generated `series/` patches are byte-identical to the source tree and the validation logs
  are copied verbatim apart from that `sha256sums.txt` comment;
  (f) the second sync from source commit `12eb48c` (2026-11-19) brought the first-boot
      configuration/image/tools changes, the TARGET-DESIGN/SKELETON-SMOKE/DTS updates and one
      new MIT-licensed host tool (`target-skeleton/tools/ubi-volume-check.py`, SPDX header
      added and usage genericised for this repository);
  (g) the third sync from source commit `8bb7f2e` (2026-11-19) brings the final full-series
      state: the persistent `rootfs_data` overlay volume in the image recipe/tool, kernel
      patches `022`/`023` and the regenerated 23-patch review form, the port-notes overlay/net
      lane records (converted from incoming-only packages to durable integration records),
      the TARGET-DESIGN/LSW-SPEC/PIE-DATAPATH/SKELETON-SMOKE updates and the final host-only
      image facts; the same curation rules as the earlier syncs apply (host paths to `~`, MIT
      headers for tools, the `target-skeleton/` layout adaptation in the series
      generator/README/checksums, no incoming/lane-private material); and
  (h) short curated status notes at the top of `docs/SKELETON-SMOKE.md`, the review-form
      README and `target-skeleton/README.md` so snapshot-time statements are not read as
      current. The 2026-11-19 CPU-count correction the curated copy previously carried locally
      is now in the source tree itself (commit `4030e86`), so this sync has no such
      divergence;
  (i) the fourth sync from source commit `7475fc5` (2026-11-19) brings the final review-form
      state: kernel patches `024` (PIE RX completion + normal non-TSO TX descriptor decode and
      delivery), `025` (gated LSW W4 R1/R3/R5 egress/init write stage) and `026` (read-only
      bdinfo-MAC MTD nvmem source + guarded LAN/WAN board.d map), the corresponding board DTS
      `hisilicon,port`/nvmem wiring, `luofu-netdev.sh`, `test-mac-nvmem.sh`, the
      LSW-SPEC/PIE-DATAPATH/MISC-SOC-BLOCKS/TARGET-DESIGN/SKELETON-SMOKE updates, the
      `pie-decode`/`lsw-egress`/`mac-nvmem` port-note records (converted from incoming
      packages to durable integration records), and the regenerated 26-patch review form
      (cover letter, mapping, MAINTAINERS, rationale, validation logs and hashes). The same
      curation rules as the earlier syncs apply; the curated README additionally records the
      final host-only `001`-`026` image rebuild (6,205,742 B, sha256 `26dde6fe...`) from the
      lab notebook's status/ledger, which post-dates this source snapshot and is not part of
      its session log.

## What is included (and why)

| Content | Why it is published |
| --- | --- |
| `README.md` | showcase: hardware, honest status table, series summary, target/image and A/B path, safety/legal |
| `docs/` (15 engineering documents plus the index and this policy) | the design, RE specifications, the measured read-only silicon inventory and its procedure, GPL request provenance, the consolidated reversal report and the 2026-11-19 boot + host-only full-series rebuild records |
| `target-skeleton/` | the OpenWrt target, board DTS with the loader hand-off contract, the 62 L0 patch files (001-058 + 062-065; the release line is the 51 include patches), image recipe (squashfs + empty dynamic `rootfs_data` overlay volume), base-files (MAC provisioning, guarded LAN/WAN map, boot-pair recorder, heartbeat), tools and mocked host tests — the actual deliverable |
| `upstream/series-6.18/` | the 26-patch review-form submission series (cover letter, per-patch commit messages, MAINTAINERS proposal, tree/order rationale, validation logs); patches `020`-`026` are carried as do-not-merge drafts. It now also carries the **release line** (`release-line/manifest.tsv`: 51 include / 11 exclude, `05-release-line.md`), the **DTS-0001** DTS-series entry (`dts-series/`, `06-dts-series-dependency.md`) and the `tools/check-release-line.sh` validator with `validation/release-line/` raw logs |
| `LICENSE-MIT`, `LICENSE-GPL-2.0`, `NOTICE.md` | licenses and provenance |
| `upstream/series-6.18/validation/` | raw host-only validation evidence (apply logs, checkpatch, hashes) |

### The 2026-11-19 syncs: hardware result and final host-only rebuild

A second curated sync brought this repository to source commit `12eb48c` (2026-11-19):

| Path | Change |
| --- | --- |
| `target-skeleton/target/linux/hisilicon/luofu/config-6.18` | built-in command line (`CONFIG_CMDLINE` + `CONFIG_CMDLINE_FORCE`) and `CONFIG_XZ_DEC_ARM=y` |
| `target-skeleton/target/linux/hisilicon/image/luofu.mk` | `--pad-align 4096` for the squashfs payload before ubinize |
| `target-skeleton/tools/luofu-image.py` | `rootfs-ubi --pad-align` implementation |
| `target-skeleton/tools/ubi-volume-check.py` | **new** semantic UBI-volume read-back verifier (MIT SPDX header added for this repository) |
| `target-skeleton/dts/luofu-r116.dts` | command-line comment updated to the measured stock-loader behaviour |
| `docs/TARGET-DESIGN.md` | sections 5.5/5.6 updated: the first-install path executed on hardware and the boot-argument question closed |
| `docs/SKELETON-SMOKE.md` | 2026-11-19 attended hardware bring-up session added |
| `README.md`, `NOTICE.md`, `docs/README.md`, this page | milestone status, honest remaining gaps, snapshot pin |

That sync carries the repository's **only hardware result**: a 6.18.60 kernel + OpenWrt
rootfs build booted to a userspace console in an attended bench session on 2026-11-19, behind
an armed recovery door and with a read-only UART capture. The image running on the unit is the
debug build, built from patches `001`-`017`; the clean rebuild (sha256 `e893c388...`) was not
re-flashed, and there was still no network, no persistent overlay and no A/B sysupgrade run.
See `README.md`.

A third curated sync brought this repository to source commit `8bb7f2e` (2026-11-19), the
final full-series state:

| Path | Change |
| --- | --- |
| `target-skeleton/target/linux/hisilicon/image/luofu-ubinize.cfg` | second UBI volume `rootfs_data`: dynamic, empty, `vol_size=1MiB`, `vol_flags=autoresize` (the persistent overlay) |
| `target-skeleton/target/linux/hisilicon/image/luofu.mk` | two-volume rootfs UBI description, `LUOFU_ROOTFS_DATA_VOLUME_NAME` and a fail-closed `/etc/board.d` helper guard |
| `target-skeleton/tools/luofu-image.py` | v0.2-overlay: two-volume cfg parser/validator (flags, reserved PEBs, empty-overlay enforcement, `vol_size` cross-check, reserve-vs-slot budget) and the `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED` marker |
| `target-skeleton/target/linux/hisilicon/patches-6.18/022-...`, `023-...` | gated LSW qid-map write path (W3/W4/W5 behind `luofu_lsw.arm`/`qidmap`) and GEMAC netdev readiness |
| `target-skeleton/dts/luofu-r116.dts`, `.../luofu/config-6.18` | net-lane DTS node/port-number and config provenance updates |
| `target-skeleton/docs/port-notes/overlay/SUMMARY.md` | **new** durable lane record for the persistent overlay + full-series image rebuild |
| `target-skeleton/docs/port-notes/net/SUMMARY.md` | connectivity-lane record, converted from an incoming package to a durable integration record |
| `docs/TARGET-DESIGN.md`, `docs/LSW-SPEC.md`, `docs/PIE-DATAPATH.md`, `docs/SKELETON-SMOKE.md` | overlay design sections 5.1/5.3/5.4/5.5, gated write path, netdev readiness, afternoon rebuild + connectivity session |
| `upstream/series-6.18/` | regenerated 23-patch review form: `0022`/`0023` messages + patches, updated cover letter/rationale/validation, raw logs re-run (23/23 apply, 0 errors, 56 warnings, 46/46 hashes) |
| `target-skeleton/README.md`, `README.md`, `NOTICE.md`, `docs/README.md`, this page | status, final image facts, connectivity gate plan, snapshot pin |

The third sync carries the repository's **final host-side artifact**: the full-series `001`-`023`
sysupgrade image, **6,205,742 B**, sha256 `15dae772...`, with kernel slice `6c99aa3d...` and
rootfs UBI `81d04951...` carrying the static `squashfs` volume plus the empty dynamic
`rootfs_data` overlay volume. The container is marked
`BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED`: the marker records the earlier
`001`-`017` userspace boot and states that **this exact rebuild was neither flashed nor
booted**. No network has ever run and no A/B sysupgrade has been executed. See `README.md`.

The intermediate source commit `12eb48c` had also added one Python bytecode cache
(`target-skeleton/tools/__pycache__/*.pyc`); the source tree dropped it again in `4030e86`,
this repository's `.gitignore` excludes build products and caches in any case, and the curated
tree contains no caches.

A fourth curated sync brought this repository to source commit `7475fc5` (2026-11-19), the
final review-form state:

| Path | Change |
| --- | --- |
| `target-skeleton/target/linux/hisilicon/patches-6.18/024-...`, `025-...`, `026-...` | **new** group-4 do-not-merge drafts: PIE RX completion + normal non-TSO TX descriptor decode/delivery (024), gated LSW W4 egress/init write stage (025), read-only bdinfo-MAC MTD nvmem + guarded LAN/WAN board.d map (026) |
| `target-skeleton/dts/luofu-r116.dts` | required `hisilicon,port` (8..12) on the five MAC nodes, read-only `bdinfo` partition with a `fixed-layout`/`mac-base` cell at `+0xde00`, `nvmem-cells` on gmac1-4 |
| `target-skeleton/target/linux/hisilicon/base-files/lib/functions/luofu-netdev.sh` | **new** guarded LAN/WAN interface map (activates only when all four GEMAC netdevs exist) |
| `target-skeleton/target/linux/hisilicon/base-files/etc/board.d/02_network` | calls the guarded map before the unchanged factory-MAC provisioning |
| `target-skeleton/tools/test-mac-nvmem.sh` | **new** host test for the nvmem wiring + interface map (82/82 assertions without a kernel tree, 96/96 with the compiled-DTS checks); SPDX changed to MIT per the license split |
| `target-skeleton/docs/port-notes/pie-decode`, `lsw-egress`, `mac-nvmem`/`SUMMARY.md` | **new** durable lane records (converted from incoming packages; absolute host paths sanitized to `~`) |
| `docs/PIE-DATAPATH.md`, `docs/LSW-SPEC.md`, `docs/MISC-SOC-BLOCKS.md`, `docs/TARGET-DESIGN.md`, `docs/SKELETON-SMOKE.md` | recovered v250 RX completion/TX descriptor tables and flows; W4 egress stage, EQM indirect command word, `port_bitmap` and the TX-egress split; kernel-side bdinfo-MAC nvmem plan (§1.4a); open question 5 kernel half; the 024/025/026 sessions |
| `target-skeleton/README.md`, `docs/README.md`, `README.md`, `NOTICE.md`, this page | status, group-4 gate plan and the final host-only `001`-`026` image facts |
| `upstream/series-6.18/` | regenerated 26-patch review form: `0024`-`0026` messages + patches, updated cover letter/mapping/MAINTAINERS/rationale/validation, raw logs re-run (26/26 apply with `patch(1)` and `git am`, 0 rejects, 0 checkpatch errors / 58 warnings, 52/52 hashes) |

The fourth sync also carries the repository's **final host-side artifact**: the full-series
`001`-`026` sysupgrade image, **6,205,742 B**, sha256
`26dde6fe14b3b3c9b9141efce019cd3e0d0993b41d09ea644e68d480ed420bb0`, with kernel slice
3,307,134 B carrying the shared `hisilicon,port` properties and a 2,883,584 B rootfs UBI
holding the static `squashfs` volume plus the empty dynamic `rootfs_data` overlay volume;
`check-container` PASS and host harnesses 144/144 (first-install), 100/100 (A/B install),
103/103 (factory-MAC provisioning) and 82/82 (MAC nvmem, without the compiled-DTS checks). It
supersedes the third sync's `15dae772...` image and the intermediate `aa642862...` build. The
container remains marked `BOOT-VERIFIED RECIPE / NOT FLASHABLE-UNTIL-INSTALLED`: **nothing has
been flashed or booted** since the `001`-`017` bring-up, and no network path, SMP release,
persistent overlay or A/B sysupgrade has ever run. The final image facts come from the lab
notebook's dated ledger/STATUS (they post-date the pinned `7475fc5` source snapshot), and the
rebuild evidence stays private lab evidence; this repository records the facts, not the logs.

## The fifth sync (2026-11-21): the 6.18 release line and the bench-verified 2-CPU baseline

Source pin: **`a6b46a9`**, the notebook's `opensource` submodule at the 6.18 release-line
consolidation (notebook parent `a66a14f`; release-line commits `c7ffeeb`,
`2052b7e`, `9e17d5c`, `5a1b5b3`, and the 2026-11-20 net/bench patches `027`-`058`). It
post-dates the fourth snapshot by two days of bench work. The curated copies are still
byte-identical to the source apart from the curation rules listed above, plus the
repository-layout adaptation of the new release-line checker.

| Path | Change |
| --- | --- |
| `target-skeleton/target/linux/hisilicon/patches-6.18/` | 36 new patch files: `027`-`058` (bench/network campaign, canonical config, boot-pair recorder, SMP diagnostics) and the renumbered SMP diagnostics `062`-`065` - the L0 stack is now **62 files, 001-058 + 062-065** |
| `target-skeleton/dts/luofu-r116.dts` | the release-critical **loader hand-off contract**: stock `/aliases/console` and `/reserved-memory/flashinfo@80600000` (`compatible = "hsan,flashinfo_reserved"`, `reg = <0x80600000 0x4000>`, `atag-offset = <0x1000>`) - the DTS-only SMP fix |
| `target-skeleton/target/linux/hisilicon/luofu/config-6.18` | canonical command line (`maxcpus=2 nr_cpus=2 additional_cpus=1`) and the canonical bench configuration |
| `target-skeleton/.../base-files/` | boot-pair recorder (`luofu-pair-record` plus rc.d links; fixes the vendor-slot reboot fallback) and a userspace heartbeat |
| `target-skeleton/RELEASE-NOTES.md` | **new**: the one-page 2026-11-21 release status (verified baseline, DTS-only SMP fix, canonical configuration, what works and what is not claimed) |
| `target-skeleton/README.md`, `target-skeleton/docs/port-notes/` | 2026-11-21 status block; new/updated lane records `smp-decode` (the final SMP diagnosis), `qm-decode`, `firstinstall`, `overlay`, `net` |
| `docs/` | `TARGET-SKELETON.md` added; TARGET-DESIGN / ROADMAP-OPENWRT / LSW-SPEC / PIE-DATAPATH / MISC-SOC-BLOCKS / SMP-RE / SKELETON-SMOKE / TRAILER-FORMAT updated - including SMP-RE sections 21-24 (the final root cause) and the W3-W20 bench record |
| `upstream/series-6.18/` | the review form revalidated for the 0018 DTS dependency (26/26 apply, checkpatch 0 errors / 58 warnings); **new release line**: `release-line/manifest.tsv` (62 patches: 51 include, 11 exclude), `INCLUDED.txt`, `EXCLUDED.txt`; **new docs** `05-release-line.md` and `06-dts-series-dependency.md`; **new DTS-series entry** `dts-series/0001-arm-dts-hisilicon-luofu-r116-loader-handoff.patch`; **new tools** `check-release-line.sh` (layout-adapted) and `gen-dts-series-patch.py`; refreshed `validation/` logs plus `validation/release-line/`; sha256sums regenerated - **89/89** entries |
| `README.md`, `NOTICE.md`, `docs/README.md`, this page | current status, the fifth-sync provenance and the updated publication claims |

Curation rules applied to this sync (the same split as before): absolute host paths sanitized
to `~`; the project SSH key is referred to generically; no incoming/lane-private staging
packages are copied; the host-only tools added or touched here (`check-release-line.sh`,
`gen-dts-series-patch.py`) carry the **MIT** SPDX header per the license split, while the
kernel patch series, DTS and `base-files/` stay GPL-2.0-only; the release-line checker locates
`target-skeleton/` in either the curated or the notebook layout, and its `sha256sums` comment
names this repository's paths; the validation logs are copied verbatim apart from those path
adaptations.

### What this sync changes about the repository's honest status

The previous snapshot said the only hardware result was the 2026-11-19 userspace boot of a
patches `001`-`017` build. That is now **history, not the current status**: a second attended
session (2026-11-20/21) brought up the wired LAN (documented CPU-copy mirror workaround), SSH
and the persistent overlay, and then solved SMP (DTS-only loader hand-off fix). The canonical
image **`974e6ca6`** is bench-verified with 2 CPUs, LAN, SSH, a ubifs overlay, a 15-minute soak
with 0 failures and reboot persistence. It is still **not** a flashable/public release: the
install path has never run on a device, Wi-Fi and native switch forwarding remain absent, and
nothing is upstream-reviewed. The earlier "only hardware result" wording in the sections above
is left in place as history.

### Checks run for this sync (2026-11-21)

- `python3 upstream/series-6.18/tools/gen-series.py --check` - **PASS** (26 patches, no drift);
- `(cd upstream/series-6.18 && sha256sum -c validation/sha256sums.txt)` - **89/89 OK**;
- `upstream/series-6.18/tools/check-release-line.sh <linux-6.18.60.tar.xz> <workdir>` -
  **PASS**: 51/51 include patches apply with `patch(1)` RC=0 (146 file events, 3 offset
  hunks, 0 fuzz, 0 rejects), strict `git apply` clean, the board DTS compiles to DTB
  12,296 B sha256 `50daf3d0...` with the loader hand-off contract, and DTS-0001 applies
  with its board DTS byte-identical to the target DTS;
- repository hygiene: no binary objects in the tree or history, no credentials/keys, no
  per-unit identifiers (the same scans as before).

## What is excluded (and why)

| Excluded | Reason |
| --- | --- |
| vendor firmware images, sysupgrade containers | third-party binaries; not ours to redistribute |
| vendor kernel modules/objects (`.ko`, `.o`, BSP objects), U-Boot images | vendor binary code; contradicts the no-vendor-code policy |
| Wi-Fi firmware blob (`FIRMWARE.bin`) and calibration data | closed vendor blob; radio-calibration data may be unit-specific |
| extracted vendor DTBs/DTBOs and DTS dumps | vendor-derived binary content; our own DTS is included instead |
| NAND/rootfs/U-Boot dumps, register dumps, `dumps/` | local evidence, large, and partly vendor-derived |
| build trees, toolchains, kernel/tarball caches, `build/` | build artifacts; reproducible by the reader from the recipe |
| Python bytecode caches and UART/boot evidence logs | build/evidence artifacts from the lab; the intermediate source commit `12eb48c` had a `__pycache__/*.pyc` file that is deliberately not synced (the source tree removed it in `4030e86`), and the 2026-11-19 boot/rebuild logs - including the final `001`-`026` image rebuild - stay private lab evidence described only in prose |
| per-unit identifiers (MAC/serial/PIN/efuse/`secret`/UUID values) | privacy and responsible disclosure; the docs name the offset instead of printing the value |
| credentials, keys, tokens, Lab/bench secrets | never published (and never committed to the source tree either) |
| the `lab/` programs and vendor-module analysis fixtures | out of scope for the OpenWrt port repo; the relevant specifications are in `docs/` |

The tooling and documentation are written so that an owner of the unit can re-derive the
private inputs locally; nothing in this repository depends on redistributing vendor binaries.

## History policy and the no-vendor-binaries claim

This repository's **entire history** is the curated commits on `main` (initial curation
2026-11-18, plus the sync commits since). There is no remote history, no imported refs and no
submodule pointing at a repository that contains vendor objects.

Verify locally:

```sh
# every object in every ref (this repository has only this history)
git rev-list --objects --all | grep -Ei '\.(o|ko|bin|img|dtb|dtbo|elf|ubi|a|so)$' || echo "no binary objects"

# tracked tree check
git ls-tree -r HEAD --name-only | grep -Ei '\.(o|ko|bin|img|dtb|dtbo|elf|ubi)$' || echo "clean"

# size check: the whole repository is a few MB of text
du -sh .git
```

Verify from the GitHub API after publication:

```sh
gh api repos/ShibbityShwab/cudy-wr3000v2-openwrt/git/trees/main?recursive=1 \
  --jq '.tree[].path' | grep -Ei '\.(o|ko|bin|img|dtb|dtbo|elf|ubi)$' || echo "remote tree clean"
gh api repos/ShibbityShwab/cudy-wr3000v2-openwrt --jq '{private, default_branch, license}'
```

## License split

| Content | License |
| --- | --- |
| Documentation (`README.md`, `NOTICE.md`, `docs/`, `target-skeleton/README.md`, `target-skeleton/docs/`) | **MIT** (`LICENSE-MIT`) |
| Tools/scripts (`target-skeleton/tools/`, `upstream/series-6.18/tools/`) | **MIT** (`LICENSE-MIT`; SPDX `MIT` header in each file) |
| Kernel code: the target patch stack (001-058 + 062-065), DTS-0001, draft DTS/DTBO, OpenWrt target, image recipe, `base-files/` | **GPL-2.0-only** (`LICENSE-GPL-2.0`; SPDX header in each file) |

The GPL-2.0-only choice is deliberate provenance: the drivers were derived by documenting the
interface of the vendor's GPL-declared binaries (see [NOTICE.md](../NOTICE.md)). The MIT/GPL
split follows the content, not the directory. In the source lab tree some of the tools carried
`GPL-2.0-only` SPDX headers; they are the project's own code and are published here under MIT,
which is compatible with this repository's use. Kernel code licensing is unchanged from the lab
tree.

## Sanitization performed

- **Absolute host paths** (`/home/<user>/...`) were replaced with `~` in all copied
  documentation. Relative evidence paths (`build/...`, `dumps/...`, `firmware/...`,
  `rootfs-*/...`, `parent:...`) are kept because they identify how a claim was produced; they
  are **not** published content and are explicitly labelled private lab evidence in
  [NOTICE.md](../NOTICE.md).
- **Per-unit data** was checked and none is present: the docs name offsets (for example the
  factory-MAC location `bdinfo+0xde00`), never the values. `MISC-SOC-BLOCKS.md` states the
  redaction rule it follows.
- **Secrets/keys/tokens** were checked and none are present; the source tree keeps live values
  in a gitignored directory. The one procedure document that names the project's bench SSH key
  file refers to it as "the project SSH key" in this curated copy; no key material is published.
- **No vendor binaries or dumps** were copied in any form (verified by extension scan, by
  `file(1)` type scan — every file is text — and by the repository size).
- The kernel patches and validation logs are the author's host output; they contain no device
  state.

## Documentation status and link conventions

The copied documents are **snapshots** written inside the lab notebook between 2026-11-18 and
2026-11-19 (with the attended bring-up session, the host-only full-series rebuild and the
connectivity session appended to `SKELETON-SMOKE.md` in the second and third syncs, the
LSW-egress, bdinfo-MAC-nvmem and PIE-completion sessions added in the fourth sync, and
TARGET-DESIGN.md sections 5.1/5.3/5.5/5.6 updated). Some of
them describe the *old* `opensource/` repository as "not publishable" because ten vendor
`.o` files were removed from its HEAD but remained in its history. **That statement does not
apply to this repository**, which never had that history. Where a document says "publication
NOT READY", read it as history of the old repository, not as a statement about this curated
tree; the current honest status of the work is in [`../README.md`](../README.md) and
[`README.md`](README.md).

Link conventions used by the copied documents:

- `parent:` and bare names such as `STATUS.md`, `BENCH-FACTS.md`, `EVIDENCE-INDEX.md`,
  `mem-entries.md` refer to the lab notebook
  ([cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)).
- `opensource/docs/...` and files not curated here (for example `FIRST-INSTALL.md`,
  `VENDOR-SKB-BRIDGE-PORT.md`, `PUBLISH-READINESS.md`, `HAZARDS.md`, `FLASH-PLAN.md`, phase
  reports) also resolve in the lab notebook's source subtree.
- `~/openwrt-lab*` paths were the author's host build clones (sanitized); `build/`, `dumps/`,
  `firmware/`, `rootfs-*/*` are private local evidence, not part of this repository.

## Relationship to the sibling repositories

- **[cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)** — the research lab
  notebook: live status, evidence, tools, the bench-reversal track. This repository is the
  curated OpenWrt-port subset of it.
- **[cudy-wr3000v2-cve](https://github.com/ShibbityShwab/cudy-wr3000v2-cve)** — sanitized
  security research from the same project (CVE write-ups, affected versions, disclosure
  status, responsible-use notes).
- An earlier publication attempt (`cudy-wr3000v2-open-firmware`) was superseded by this
  curated repository; its history is **not** a basis for this one.

## Publication intent

This work is **intentionally public** (owner decision, for portfolio/career purposes). The
repositories are curated so that no vendor binary, no per-unit secret and no private lab
evidence is published. If you find something that should not be public, please open an issue
and it will be removed promptly; corrections are welcome.

## Scope reminder

Two attended bench sessions are recorded. The 2026-11-19 session booted a build of this tree
(patches `001`-`017`) to a userspace console - console only, no network, tmpfs overlay - and
was the first hardware result. The 2026-11-21 session ran the canonical image `974e6ca6` built
from the L0 stack (001-058 + 062-065; numbered 001-061 at the time) plus the DTS loader hand-off fix: 2 CPUs, LAN through the documented
CPU-copy mirror workaround, SSH, persistent ubifs overlay, 15-minute soak with 0 failures and
reboot persistence. The intermediate host-only full-series rebuilds (001-023 with the
persistent `rootfs_data` volume, and the 001-026 image with the PIE completion, LSW egress and
bdinfo-MAC nvmem follow-ups) were never flashed or booted. Neither bench session is an install
or recovery procedure and the install path has never run on a device, so the repository
remains curated research material, not a firmware release; see the safety and legal section of
[`../README.md`](../README.md) and [`../NOTICE.md`](../NOTICE.md).
