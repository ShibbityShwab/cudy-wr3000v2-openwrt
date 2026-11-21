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
  The **sixth sync (2026-11-21)** moves it again to the board-feature refresh:
  `opensource` submodule commit `f136cf3` (notebook parent `ab0d3cb`), which adds the
  stock board_id=3 LED/keys nodes, the base-DTB `pad-dtb` alignment fix, the W17/W19
  `dev_dbg` quieting, the refreshed DTS-0001 and the retained read-only feature-inventory
  and Wi-Fi recon reports - see the sixth-sync section below.
  The **seventh sync (2026-11-21)** moves it to the DTS/install follow-up:
  `opensource` submodule commit `d6db2e1` (notebook parent `0983229`), which adds the
  conservative comment-only board-DTS cleanup (active content byte-identical to the
  hardware-verified known-good DTB), the verified automated stock-telnet install path, the
  multi-port Ethernet plan, the DTS cleanup/regression record and the feature-inventory update
  - see the seventh-sync section below.
  The **eighth sync (2026-11-21)** moves it to the submission-package follow-up:
  `opensource` submodule commit `c931c74` (notebook parent `fa76c97`), which adds the
  **documentation-only upstream submission package** (`submission/`), the bench-only multi-port
  source-port probe patch 066 + prepared probe image `a91ebcf0`, and the final validation
  records - see the eighth-sync section below.
  The **ninth sync (2026-11-21)** records the **measured result of the multi-port source-port
  experiment: NEGATIVE** (`opensource` submodule commit `117ea55`, notebook parent
  `75a4d8c`) - CPU-copy completions carry no source-port identity, so multi-port LAN/WAN
  requires the native L2/VLAN hardware bridge - see the ninth-sync section below.
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
| `docs/` (16 engineering documents plus the index and this policy) | the design, RE specifications, the measured read-only silicon inventory and its procedure, GPL request provenance, the consolidated reversal report, the trailer-format doc including the base-DTB alignment section, the multi-port plan (incl. the prepared probe image), and the validation records through the 2026-11-21 board-feature/quiet-image, DTS/install and submission-package follow-up passes |
| `target-skeleton/` | the OpenWrt target, board DTS with the loader hand-off contract, **the stock board_id=3 LED/keys nodes and the conservative comment-only cleanup** (active content byte-identical to the known-good DTB), the 62 L0 patch files (001-058 + 062-065; the release line is the 51 include patches), image recipe (squashfs + empty dynamic `rootfs_data` overlay volume) **including the `Build/luofu-pad-dtb` base-DTB alignment step**, base-files (MAC provisioning, guarded LAN/WAN map, boot-pair recorder, heartbeat), tools (incl. `pad-dtb.py`) and mocked host tests — the actual deliverable. It also carries the **retained read-only device reports** under `docs/incoming/` (feature inventory + its 2026-11-21 update, Wi-Fi recon; curated notes explain superseded findings and the install/multi-port follow-ups) |
| `upstream/series-6.18/` | the 26-patch review-form submission series (cover letter, per-patch commit messages, MAINTAINERS proposal, tree/order rationale, validation logs); patches `020`-`026` are carried as do-not-merge drafts. It now also carries the **release line** (`release-line/manifest.tsv`: 51 include / 11 exclude; 054/056 hashes refreshed after the `dev_dbg` quieting, manifest sha256 `4da8b960...`; `05-release-line.md`), the **DTS-0001** DTS-series entry (`dts-series/`, sha256 `8ee4b8e71...`, refreshed with the board LED/keys nodes; `06-dts-series-dependency.md`) and the `tools/check-release-line.sh` validator (which also exercises the pad-dtb step) with `validation/release-line/` raw logs, plus `07-dts-upstream-bindings.md` (the DTS cleanup/hardware-regression record and the retained optional deltas) |
| `upstream/series-6.18/submission/` | the **documentation-only upstream submission package** (RFC form, dry run): package README/status/artifact checklist, one-page cover-email draft, explicit no-Tested-by policy and `MANIFEST.sha256`. Nothing has been sent; no transport/account/list subscription; the manifest is regenerated against this curated tree and verifies `sha256sum -c` (102/102) |
| `docs/incoming/multi-port-probe/` | retained **bench-only** multi-port source-port probe record: patch 066 (GPL-2.0-only kernel diff, not part of the release line) plus its README; the probe image `a91ebcf0` is documented in the multi-port plan. The experiment was run on 2026-11-21 and was **negative**; native L2/VLAN bridge required |
| `tools/` | the verified bench **stock-telnet installer** (`stock-telnet-install.sh` + `telnet-exec.py`) and its `README.md`. Fail-closed, inactive-pair only; published for evidence/reproducibility, not as a general install/recovery guide. Read-only inspect default; `--apply` needs `--attended --recovery-armed`. MIT SPDX headers in this repository |
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

## The sixth sync (2026-11-21): board features, base-DTB padding and the quiet image

Source pin: **`f136cf3`**, the notebook's `opensource/` submodule after the board-feature
pass (notebook parent `ab0d3cb`; commits `a8914d8` (read-only feature inventory),
`aff7e27` (LED/keys DTS, pad-dtb, watchdog sysfs) and `f136cf3` (release-line/DTS-0001
refresh, quiet image)). It post-dates the release-line consolidation by the same day's bench
work and is the current pin.

| Path | Change |
| --- | --- |
| `target-skeleton/dts/luofu-r116.dts` | stock **board_id=3** overlay nodes: gpio-keys-polled (reset = GPIO22, WPS = GPIO23, active-low, 20 ms, `KEY_RESTART`/`KEY_WPS_BUTTON`) and six `oem:green:*` gpio-leds (status=GPIO24, wan=25, lan=3, internet=2, wifi=0, wifi5g=1) |
| `target-skeleton/target/linux/hisilicon/luofu/config-6.18` | built-in cmdline now bakes the canonical module gates plus `oops=panic panic=0`; adds `CONFIG_DEVMEM`/`CONFIG_STRICT_DEVMEM=n` and `LEDS_GPIO`, `INPUT`/`INPUT_EVDEV`/`INPUT_KEYBOARD`, `KEYBOARD_GPIO_POLLED`, `WATCHDOG_SYSFS` (`INPUT_HISI_POWERKEY` off) |
| `target-skeleton/target/linux/hisilicon/image/luofu.mk` | new `Build/luofu-pad-dtb` step before `append-dtb` |
| `target-skeleton/tools/pad-dtb.py` | **new** MIT host tool: pads the compiled base DTB to an 8-byte FDT totalsize, folds the padding into the header and asserts `len(dtb) == totalsize` / `totalsize % 8 == 0`. Measured 13,041 B -> 13,048 B: the unaligned `bootfip` gate-word load data-aborted, reset the CPU and fell back to the stock pair before the fix (TRAILER-FORMAT.md section 11) |
| patches 054 / 056 | periodic W17/W19 logs quieted `dev_warn`/`dev_info` -> `dev_dbg` (release-line hashes refreshed) |
| `base-files/etc/init.d/heartbeat` | interval 5 s -> 60 s |
| `target-skeleton/docs/incoming/feature-inventory/{SUMMARY,REPORT}.md` | **new retained read-only reports**: the 2-CPU device feature matrix and gaps; the curated note records the later LED/keys/watchdog fixes, the thermal root cause and the withdrawn MAC-duplication concern |
| `target-skeleton/docs/incoming/wifi-recon/{SUMMARY,REPORT}.md` | **new retained host-only reports**: Hi5622V100 vendor-binary stack, no 6.18 driver, option C (unsupported) recommended, hybrid 5.10 as the only Wi-Fi-capable system |
| `target-skeleton/RELEASE-NOTES.md` | updated: board-feature baseline `7f270b8c`, quiet image `7421bb7a`, thermal unsupported, the pad-dtb requirement |
| `target-skeleton/README.md` | re-synced status page (curated note updated for the board-feature pass) |
| `docs/TRAILER-FORMAT.md` | section 11 (base-DTB 8-byte totalsize alignment) folded into this repository's canonical and `docs/` copies from the notebook's standalone TRAILER-FORMAT note |
| `upstream/series-6.18/` | **DTS-0001 regenerated** for the board nodes (sha256 `8ee4b8e71...`, 1,329-line board DTS); manifest 054/056 hashes refreshed (manifest sha256 `4da8b960...`, still **51 include / 11 exclude**); 00-cover-letter / 04-validation / 05-release-line / 06-dts-series-dependency / README updated; `check-release-line.sh` now exercises the pad-dtb step; validation logs and `sha256sums.txt` refreshed (**89 entries**) |
| `README.md`, `NOTICE.md`, `docs/README.md`, this page | current status, sixth-sync provenance, retained reports, thermal/Wi-Fi/MAC statements |

Curation rules applied to this sync: absolute host paths sanitized to `~`; the host-only
`pad-dtb.py` carries the MIT SPDX header per the license split (the DTS/config/image/base-files
content stays GPL-2.0-only); no vendor binaries, firmware blobs, credentials or secrets are
added; the retained reports are text-only and their per-unit MAC placeholder was redacted to
the documentation prefix `02:00:00:xx:xx:xx` (the real unit OUI is removed); the two reports are
a deliberate, documented exception to the "no incoming/staging material" rule (curated notes in
their `SUMMARY.md` files mark superseded rows). The DTS-0001 hash, release-line manifest hash and
`sha256sums.txt` match the source notebook.

### What this sync changes about the repository's honest status

The previous snapshot recorded two bench sessions and host-only board features. Now the
stock board_id=3 features are **verified on hardware** (image `7f270b8c`: six writable LEDs,
`event0` gpio-keys-polled, `watchdog0` sysfs; image `7421bb7a`: same plus the periodic dmesg
spam removed), the base-DTB alignment bug is found and guarded, and **thermal** and **Wi-Fi** are
recorded as unsupported on this board/6.18 target (with the reasons in the retained reports).
The earlier "duplicate MAC" concern is **withdrawn**: LAN ports share the vendor base MAC and
WAN = base+1 by design. Still **not** a release: the install path has never run on a device,
native switch forwarding is a separate project, and nothing is upstream-reviewed.

### Checks run for this sync (2026-11-21)

- `python3 upstream/series-6.18/tools/gen-series.py --check` - **PASS** (26 review-form patches, no drift);
- `(cd upstream/series-6.18 && sha256sum -c validation/sha256sums.txt)` - **89/89 OK**;
- `upstream/series-6.18/tools/check-release-line.sh <linux-6.18.60.tar.xz> <workdir>` -
  **PASS**: 51/51 include patches apply with `patch(1)` RC=0 (146 file events, 3 offset hunks,
  0 fuzz, 0 rejects), strict `git apply` clean, the board DTS compiles to DTB 13,041 B
  sha256 `49fe5c47...` with the loader hand-off contract, the unaligned totalsize triggers
  `pad-dtb.py` on a copy (13,048 B, contract preserved), and DTS-0001 applies with its board
  DTS byte-identical to the target DTS;
- mocked host tests: first-install **94/94**, factory-MAC **103/103**, MAC-nvmem **82/82**;
- `pad-dtb.py` guard smoke test on a copy of the compiled DTB (pads, folds totalsize, refuses a
  non-FDT / size-mismatch input);
- repository hygiene: no binary objects in the tree or history, no credentials/keys, no
  per-unit identifiers, no host paths; link checks clean; working tree clean.

## The seventh sync (2026-11-21): conservative DTS cleanup, verified install path and the regression record

Source pin: **`d6db2e1`**, the notebook's `opensource/` submodule after the DTS/install
follow-up (notebook parent `0983229`; commits `77dff89`, `ba3b3e8`,
`ddeaa63`, `538377c`, `7dcb745`, `7ae18f2`, `8e73c0f`, `d6db2e1`). It
post-dates the board-feature sync by the same day's bench work and is the current pin.

| Path | Change |
| --- | --- |
| `target-skeleton/dts/luofu-r116.dts` | **conservative comment-only cleanup**: 1,329 -> **691 lines**; every active node name, unit address, property order/value, `reg` order, status, label, phandle, child name and alias unchanged; the compiled DTB is the **byte-identical known-good DTB** (13,041 B sha256 `49fe5c47...`; padded by `Build/luofu-pad-dtb` to 13,048 B sha256 `402e4f71...`) |
| `upstream/series-6.18/dts-series/` | DTS-0001 regenerated from the conservative single source: **sha256 `072d0b8b...`**, 704 lines, board DTS byte-identical to the target DTS |
| `upstream/series-6.18/{00,04,05,06,07}*.md`, `README.md` | the DTS cleanup record: `04-validation.md` sections 22 (superseded active cleanup, with the regression warning) and 23 (conservative re-validation), `05-release-line.md` updated (three verified baselines, DTB/pad hashes, 07 reference), `06-dts-series-dependency.md` updated (DTS-0001 hash/lines, regression refs), **new `07-dts-upstream-bindings.md`** (hardware regression, conservative cleanup, optional hardware-tested deltas, manual binding review, remaining blockers), `README.md` updated; `validation/git-log.txt` refreshed |
| `target-skeleton/RELEASE-NOTES.md` | install-path status (manual repeats + the stock-telnet tool), the **active-DTS hardware-regression record**, the multi-port status and the known-good baseline/byte-identical rebuild facts |
| `target-skeleton/docs/incoming/feature-inventory/UPDATE-2026-11-21.md` | **new retained update**: matrix corrections (LEDs/keys/watchdog fixed, thermal root-caused, MAC concern withdrawn), the install-path finding and the verified automated stock-telnet follow-up, and the multi-port follow-up |
| `docs/MULTI-PORT-PLAN.md` | **new**: multi-port Ethernet limit (only eth2 works; CPU-copy completions report port 0/no source id; the far-right jack is a LAN port), the attended source-port experiment and the decision gate |
| `tools/` | **new published bench tooling**: `stock-telnet-install.sh` (fail-closed, inactive-pair only, verified end-to-end) + `telnet-exec.py` + `README.md`. Staging path sanitized to `$HOME/staging-fi`; MIT SPDX per the project-tool split |
| `README.md`, `NOTICE.md`, `docs/README.md`, this page | known-good baseline (`7421bb7a`), verified install path, the DTS regression record, the multi-port limit and the updated layout/license/scope text |

Curation rules applied to this sync: host paths sanitized to `~` (the installer's staging
path became `$HOME/staging-fi`); the two published bench tools carry the MIT SPDX header per
the license split while the DTS/patch/base-files content stays GPL-2.0-only; no vendor
binaries, firmware blobs, credentials, keys or per-unit identifiers are added; the retained
feature-inventory update is text-only and reuses the redacted documentation MAC prefix; the
notebook's bench helpers (`bench-link.sh`/`recovery-card.sh`) and the image build are
**not** published (the installer refuses without their preflight; documented in
`tools/README.md`). DTS-0001 and the release-line hashes match the source notebook.

### What this sync changes about the repository's honest status

- The **known-good hardware baseline is image `7421bb7a`** (sha256
  `7421bb7ace41e2e03613a0314874fabfcd16aeef60dbdd713ebc6c2b735cd4b3`); the conservative
  comment-only DTS rebuild reproduces it **byte-for-byte**, so the devicetree semantics are the
  hardware-verified ones.
- The **install path is verified end-to-end** with the published stock-telnet installer: from
  the vendor stock firmware, only the inactive pair is written, the active pair is verified
  unchanged, the selector is verified and our 2-CPU image boots. No install image is published
  and no recovery procedure is included.
- The **superseded aggressive DTS cleanup (image `6f19c6d6`) is documented as a boot
  regression**: it passed every host check but MTD partition registration stalled and the
  U-Boot watchdog reset the unit (stock fallback). The lesson is recorded: host checks are
  necessary but not sufficient; every active-DTS change must be boot-tested.
- **Multi-port Ethernet is explicitly limited** today: only eth2 works through the CPU-copy
  mirror; the CPU-copy completion reports port 0 with no confirmed source-port identifier, so
  multi-port needs the attended source-port experiment or the native L2/VLAN bridge. The plan
  and decision gate are published.
- Still **not** a release: `platform_do_upgrade` has never run on a device, per-slot A/B
  trailer handling and the loader env re-stamp remain open, Wi-Fi and thermal are unsupported,
  and nothing is upstream-reviewed.

### Checks run for this sync (2026-11-21)

- `python3 upstream/series-6.18/tools/gen-series.py --check` - **PASS** (26 review-form patches, no drift);
- `(cd upstream/series-6.18 && sha256sum -c validation/sha256sums.txt)` - **89/89 OK**;
- `upstream/series-6.18/tools/check-release-line.sh <linux-6.18.60.tar.xz> <workdir>` -
  **PASS**: 51/51 include patches with `patch(1)` RC=0 (146 file events, 3 offset hunks, 0 fuzz,
  0 rejects), strict `git apply` clean, the conservative board DTS compiles to DTB **13,041 B
  sha256 `49fe5c47...`** with the loader hand-off contract, the unaligned totalsize triggers
  `pad-dtb.py` on a copy (**13,048 B sha256 `402e4f71...`**, contract preserved), and
  DTS-0001 (sha256 `072d0b8b...`) applies with its board DTS byte-identical to the target DTS;
- mocked host tests: first-install **94/94**, factory-MAC **103/103**, MAC-nvmem **82/82**;
- new tools: `bash -n` and `py_compile` pass; `stock-telnet-install.sh` with no image exits
  non-zero ("--image is required") and refuses `--apply` without the attended/recovery flags;
- repository hygiene: no binary objects in the tree or history, no credentials/keys, no per-unit
  identifiers, no host paths; link checks clean; working tree clean.

## The eighth sync (2026-11-21): documentation-only submission package and the prepared multi-port probe

Source pin: **`c931c74`**, the notebook's `opensource/` submodule after the submission-package
follow-up (notebook parent `fa76c97`; commits `4640631` - multi-port source-port probe
patch + prepared experiment image - and `c931c74` - documentation-only upstream submission
package + final validation records). It is the current pin.

| Path | Change |
| --- | --- |
| `docs/MULTI-PORT-PLAN.md` | updated with the **prepared experiment image `a91ebcf0`** (save copy sha256 `a91ebcf04892ac9d5128c6ae5144eb9d6e09674b98938aac0aa8af289b3bc653`): the known-good `7421bb7a` userspace/kernel plus the source-mask probe patch and built-in probe tokens `luofu_pie.rxdiag=1 luofu_switch.mirror_src_mask=0x1e00`; run plan and restore step included. Bench-only. The experiment was subsequently **run and was negative** (see the ninth sync): the CPU-copy completions carry no source-port identity |
| `docs/incoming/multi-port-probe/` | **new retained bench-only record**: `066-net-lsw-mirror-src-mask.patch` (adds the `luofu_switch.mirror_src_mask` module parameter, default `0x800`; GPL-2.0-only kernel diff) plus a curated README. Not part of the release line, review form or submission package |
| `upstream/series-6.18/submission/` | **new documentation-only upstream submission package**: `README.md` (status, artifact checklist, reproduction commands, six merge-intent blockers, out-of-scope list), `cover-email-draft.md` (one-page v3 RFC draft, **not sent**), `TESTED-BY-POLICY.md` (no upstream Tested-by/Reviewed-by; target-image bench evidence only) and `MANIFEST.sha256` (102 entries, regenerated against this curated tree; `sha256sum -c` passes 102/102) |
| `upstream/series-6.18/{04-validation,05-release-line,07-dts-upstream-bindings,README}.md`, `validation/git-log.txt` | final records: `04-validation.md` section 24 (submission-package validation run: release line 51/51, DTB 13,041 B `49fe5c47...`, pad 13,048 B `402e4f71...`, review form 26/26, checkpatch 0/58, hash checks 89/89), conservative-final references (`7ae18f2`), the six upstream blockers, and the README pointer to the submission package; the review-form git log was regenerated |
| `README.md`, `NOTICE.md`, `docs/README.md`, this page | submission package, prepared multi-port probe, manifest-regeneration note and the updated documentation index |

Curation rules applied to this sync: host paths sanitized to `~` (none were new); no vendor
binaries, firmware blobs, credentials, keys or per-unit identifiers; the submission package's
`MANIFEST.sha256` was **regenerated against this curated tree** (8 entries differ from the
notebook package because of curation: review-form README curated note, path-adapted
`01-commit-messages.md`, MIT SPDX headers on the host tools) and its README carries a curated
note explaining this - `sha256sum -c MANIFEST.sha256` passes 102/102 from inside the package;
the bench-only probe patch 066 is kept verbatim with a curated README and is not part of the
release line or upstream series; the package contains no email transport, account, list
subscription, secret or Tested-by claim.

### What this sync adds to the honest status

- The upstream path now has a **reviewable dry-run package**, but **nothing has been submitted**
  and no third-party review/Tested-by exists; the six merge-intent blockers are recorded in the
  package and in `07-dts-upstream-bindings.md`.
- The multi-port work has a **prepared experiment** (image `a91ebcf0` + patch 066); it was
  subsequently **run and was negative** (no source-port identity in the CPU-copy completions) -
  see the ninth sync. No multi-port claim is made; the native L2/VLAN bridge is the required
  path.
- Everything else (known-good `7421bb7a` baseline, verified install path, conservative DTS,
  thermal/Wi-Fi non-support, single-jack LAN workaround) is unchanged from the seventh sync.

### Checks run for this sync (2026-11-21)

- `python3 upstream/series-6.18/tools/gen-series.py --check` - **PASS** (26 review-form patches, no drift);
- `(cd upstream/series-6.18 && sha256sum -c validation/sha256sums.txt)` - **89/89 OK**;
- `upstream/series-6.18/tools/check-release-line.sh <linux-6.18.60.tar.xz> <home-workdir>` -
  **PASS**: 51/51 include patches with `patch(1)` RC=0 (146 file events, 3 offset hunks, 0 fuzz,
  0 rejects), strict `git apply` clean, DTB 13,041 B sha256 `49fe5c47...`, pad-dtb exercised
  to 13,048 B sha256 `402e4f71...` with the contract preserved, DTS-0001 (`072d0b8b...`)
  applies with its board DTS byte-identical to the target DTS;
- `(cd upstream/series-6.18/submission && sha256sum -c MANIFEST.sha256)` - **102/102 OK**;
- mocked host tests: first-install **94/94**, factory-MAC **103/103**, MAC-nvmem **82/82**;
- bench tools: `bash -n` / `py_compile` pass and the fail-closed `--apply` gates are intact;
- repository hygiene: no binary objects in the tree or history, no credentials/keys, no per-unit
  identifiers, no host paths; link checks clean; working tree clean.

## The ninth sync (2026-11-21): the multi-port source-port experiment result (negative)

Source pin: **`117ea55`** (notebook parent `75a4d8c`), the commit that records the
multi-port source-port experiment result. The only changes are two documentation files; no code,
DTS, series, tooling or manifest entries change.

| Path | Change |
| --- | --- |
| `docs/MULTI-PORT-PLAN.md` | **new section "Measured result of the source-port experiment"**: probe image `a91ebcf0` was installed (active stock pair untouched, selector `0x11` verified), the far-right LAN jack was exercised with the extended mirror mask (ports 9-12), and the captured descriptors are **field-for-field identical** to the eth2 baseline - `port=0`, `vport=0`, only descriptor words 0/1 populated, no source-port field. The mirror mask does cover the other jacks, but the CPU cannot tell them apart. Conclusion: CPU-copy multi-port is **not feasible**; the native L2/VLAN hardware bridge (LSW-SPEC) is required; do not fake port identity by MAC heuristics. The board was restored to known-good `7421bb7a` |
| `target-skeleton/RELEASE-NOTES.md` | **new section "Multi-port source-port experiment: negative"**: the single-port CPU-copy bench workaround stands; multi-port LAN/WAN requires the native L2/VLAN bridge (separate project) |
| `README.md`, `NOTICE.md`, `docs/README.md`, this page | the multi-port status is updated from prepared/not-run to the measured **negative** result and the native-bridge conclusion |

### What this sync changes about the repository's honest status

The multi-port question is now **closed for the CPU-copy path**: the experiment was run on the
unit and the CPU-copy completions carry no source-port/vport identifier (identical descriptors
across jacks), so the CPU cannot distinguish LAN jacks. The repository therefore records
multi-port LAN/WAN as requiring the **native L2/VLAN hardware bridge** (separate project,
`docs/LSW-SPEC.md`), not the single-port bench workaround; the verified bench topology remains
host on the eth2 jack. No code, image or DTS change is implied by this sync.

### Checks run for this sync (2026-11-21)

- documentation-only sync: no code/DTS/series/tooling change, so the release-line validation,
  `sha256sum -c validation/sha256sums.txt` (89/89) and the submission manifest (102/102) are
  unchanged from the eighth sync and were re-verified;
- `python3 upstream/series-6.18/tools/gen-series.py --check` - **PASS** (26 patches, no drift);
- link checks clean (only the documented notebook-only links unresolved); hygiene scans clean
  (no binaries/secrets/per-unit identifiers/host paths); the pushed-remote fresh clone verified.

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
| the notebook bench helpers the installer calls (`tools/bench-link.sh`, `tools/recovery-card.sh`), the image build and the physical recovery procedure | they are live-lab instruments, not publication content; the published installer refuses without their preflight and `tools/README.md` states the dependency. No recovery procedure is published |
| the `lab/` programs and vendor-module analysis fixtures | out of scope for the OpenWrt port repo; the relevant specifications are in `docs/` |
| incoming/lane staging packages, **except** the retained read-only reports and the bench-only probe record | a deliberate exception: `target-skeleton/docs/incoming/feature-inventory/` and `target-skeleton/docs/incoming/wifi-recon/` are text-only, read-only device/Wi-Fi reports needed to explain the current status (curated notes; no vendor binaries, secrets or per-unit identifiers; the one MAC placeholder uses the documentation prefix `02:00:00:xx:xx:xx`), and `docs/incoming/multi-port-probe/` retains the bench-only probe patch 066 + note because the multi-port experiment plan references it. None is part of the release line or upstream series |

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
- **The retained device reports' MAC placeholder** was redacted to the documentation prefix
  `02:00:00:xx:xx:xx`; the unit's real OUI does not appear (the reports are otherwise reproduced
  with curated status notes only).
- **The bench installer's staging path** was sanitized from the author's home directory to
  `$HOME/staging-fi`; the project host tools published under `tools/` carry the MIT SPDX
  header per the license split (their notebook copies are GPL-2.0-only project code).
- **The submission package's `MANIFEST.sha256` was regenerated for this curated tree** (the
  notebook manifest hashes cover the notebook layout and original files; this repository's
  review-form README carries a curated note, `01-commit-messages.md` uses this repository's path
  layout, and the host tools carry MIT SPDX headers). The submission README carries a curated
  note describing this; `sha256sum -c MANIFEST.sha256` passes 102/102 from inside the package.
  The package is documentation-only - no account, list subscription or email transport is
  included, and nothing has been sent.
- **The multi-port probe patch 066** is kept verbatim from the notebook (a GPL-2.0-only kernel
  diff, our own code) with a curated README; it is not part of the release line, the review form
  or the submission package, and the prepared image `a91ebcf0` is a bench instrument only.
- The kernel patches and validation logs are the author's host output; they contain no device
  state.

## Documentation status and link conventions

The copied documents are **snapshots** written inside the lab notebook between 2026-11-18 and
2026-11-21 (with the attended bring-up session, the host-only full-series rebuild and the
connectivity session appended to `SKELETON-SMOKE.md` in the second and third syncs, the
LSW-egress, bdinfo-MAC-nvmem and PIE-completion sessions added in the fourth sync, the
release-line consolidation in the fifth, and the 2026-11-21 board-feature/pad-dtb/quiet-image
pass in the sixth; TARGET-DESIGN.md sections 5.1/5.3/5.5/5.6 updated). The two retained device
reports under `target-skeleton/docs/incoming/` are the only copied "incoming" material and
carry curated status notes; their per-unit MAC placeholder was redacted to the documentation
prefix. Some of
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

Three bench phases are recorded. The 2026-11-19 phase booted a build of this tree (patches
`001`-`017`) to a userspace console - console only, no network, tmpfs overlay - and was the
first hardware result. The 2026-11-21 canonical phase ran image `974e6ca6` built from the L0
stack (001-058 + 062-065; numbered 001-061 at the time) plus the DTS loader hand-off fix: 2
CPUs, LAN through the documented CPU-copy mirror workaround, SSH, persistent ubifs overlay,
15-minute soak with 0 failures and reboot persistence. The same-day board-feature phase ran
`7f270b8c` (six writable `oem:green:*` LEDs, `event0` gpio-keys-polled, `watchdog0` sysfs)
and the quiet image **`7421bb7a` (known-good baseline, sha256
`7421bb7ace41e2e03613a0314874fabfcd16aeef60dbdd713ebc6c2b735cd4b3`; periodic W17/W19/heartbeat
dmesg spam removed)**, and it required the base-DTB `pad-dtb` alignment step. On the same day
the **automated stock-telnet install path** was verified end-to-end (inactive pair only, active
pair unchanged, selector verified, 2-CPU image booted), and the superseded aggressive active-DTS
cleanup (image `6f19c6d6`) was documented as a **hardware boot regression** and reverted; the
accepted conservative cleanup is comment-only and compiles to the known-good DTB. Thermal
telemetry is unsupported on this board (temperature register reads 0, vendor DT disables the
sensor), Wi-Fi is unsupported on the 6.18 target, only one Ethernet jack works through the
CPU-copy workaround (multi-port is an open experiment/native-bridge project), and the earlier
MAC-duplication concern is withdrawn (LAN ports share the vendor base MAC, WAN = base+1 by
design). The intermediate host-only full-series rebuilds (001-023 with the persistent
`rootfs_data` volume, and the 001-026 image with the PIE completion, LSW egress and bdinfo-MAC
nvmem follow-ups) were never flashed or booted. No install image is published, `platform_do_upgrade`
has never run on a device, and the bench installer in `tools/` is published for evidence, not
as a general install/recovery guide. The **upstream submission package** is a documentation-only
dry run - **nothing sent, no Tested-by** - and the **multi-port source-port experiment** (patch 066 / image
`a91ebcf0`) was run and was **negative** - the CPU-copy path cannot identify jacks, so
multi-port LAN/WAN requires the native L2/VLAN hardware bridge; the repository remains curated
research material, not a firmware release. See the safety and legal section of [`../README.md`](../README.md) and
[`../NOTICE.md`](../NOTICE.md).
