# Documentation index

These documents are a curated snapshot of the lab notebook's engineering record at the
**2026-11-22 multi-port LAN close-out** - source `opensource` submodule `fa36d9b`
(notebook parent `210548b`; W22 release integration plus the M1b bench results): the **53-patch
release line** (64 patch files: 001-060 + 062-065; the 11 bench-only SMP diagnostics are excluded)
including the **W22 native-forwarding patches 059** (PFE NAPT/flow-engine init) and **060** (PIE
native exception-class delivery; canonical cmdline `naptinit=1` with no mirror tokens; the
CPU-copy mirror patches 054-057 remain only as the documented fallback), the **DTS-only loader
hand-off SMP fix** plus the stock **board_id=3 LED/keys nodes**, the **conservative comment-only board-DTS
cleanup plus all five boot-tested deltas** (TWD level-high, MAC-nvmem removal, switch MMIO-first
order, cudy-first compatible, flashinfo compatible-first; DTS `381b2e8c...`; DTB `62574b11...` / pad `8d79794e...`;
current DTS-purpose baseline image `6e9c54d0` and the **canonical W22 data-path image
`50023446`** with the **M1a proof image `5800c9ab`**), the base-DTB **`pad-dtb` 8-byte alignment build step**, the
re-generated **DTS-0001** (`886f3ad0...`, 694 lines), the verified automated
**stock-telnet install path**, the **multi-port Ethernet limit and experiment plan** (with the
bench-only probe patch 066 / image `a91ebcf0`, whose source-port experiment was run and was
**negative** - CPU-copy completions carry no port identity; the W22 native path plus the software
bridge now provide multi-port LAN forwarding without a hardware L2/VLAN offload (M1b-1/M1b-3 PASS),
and the WAN L3 path is verified as well (host DHCP lease 192.168.50.115/24, default route, ICMP
both ways; host ufw UDP 67/68 gotcha recorded), so **no bench items remain**), the
**documentation-only upstream submission package** (zero remaining blockers),
and the current baselines: **`50023446`** (canonical W22, native CPU delivery), `6e9c54d0` (current
DTS-purpose; previous `c3ef2a50`; earlier f6685b80), and `974e6ca6`/`7f270b8c`/`7421bb7a` for the
pre-W22 bring-up. The upstream review form stays at 26 patches (001-026) - **W22 is release-line only,
not in the L2 form**. Bench close-out: **M1b-1 PASS** (native per-port CPU delivery on all four
jacks), **M1b-3 LAN-to-LAN PASS on both tested LAN pairs** (eth2<->eth0 and eth2<->eth1; broadcast
+ learned unicast both directions through the software bridge; **no hardware L2 offload
required**) and a **15-minute soak PASS 900/900 0% loss**. The WAN L3 path is verified as well (host DHCP lease 192.168.50.115/24, default route, ICMP both
ways), so **no bench items remain**; the fallback-image test is retired as untested-by-design (known
limitation). The superseded
aggressive DTS cleanup is recorded as a **hardware boot regression** (image `6f19c6d6`). The live
status of the work is the lab notebook's `STATUS.md`; the honest summary of what is host-validated
and what the bench phases did - and did **not** - prove is in [`../README.md`](../README.md). Read
[`PUBLISHING.md`](PUBLISHING.md) for the curation rules, the license split, the sanitization
performed and the 2026-11-18/19/21 syncs.

## Documents

| File | What it is | Status |
| --- | --- | --- |
| [TARGET-DESIGN.md](TARGET-DESIGN.md) | the concrete target/DTS/image design: target layout, board DTS conversion, FIP/trailer/dual-slot/sysupgrade recipe, driver inventory and upstream distance, acceptance criteria, staged plan - with the 2026-11-21 release and board-feature updates, the mandatory loader hand-off DTS contract and the closed/partly-closed rows | design proposal (host-validated drafts + bench-verified target images; not accepted upstream) |
| [TARGET-SKELETON.md](TARGET-SKELETON.md) | the target-skeleton index: what the schematic target, DTS, image recipe, tools and patch stack are; status and what remains [U]/TODO | index |
| [ROADMAP-OPENWRT.md](ROADMAP-OPENWRT.md) | the phase plan and milestone history; the 2026-11-21 update closes the SMP milestone without claiming upstream progress; superseded sections are marked | phase plan / history |
| [incoming/lsw-forwarding/](incoming/lsw-forwarding/) | retained native L2/VLAN forwarding lane: implementation plan, E0-a/E0-b analyses and results (negative mirror-off run; read-only stock-vs-ours captures that found the IFC->NAPT unicast path), the reviewed **M1a probes and proof** (`m1a/napt-init/proof-image/RESULT.md`; proof image `5800c9ab` - native port=11 deliveries, ping 200/200, SSH), the **W22 release-integration drafts** (`w22/`; now release-line patches 059/060, canonical image `50023446`) and the **M1b plan + results** (`m1b/M1B-PLAN.md`, `m1b/M1B-RESULTS.md`: native per-port delivery
on all four jacks; **LAN-to-LAN PASS on eth2<->eth0 and eth2<->eth1** through the software bridge,
no hardware L2 offload), plus read-only capture/diff tools and a print-only write planner | bench
records + gated instruments only; nothing executed by this repository; LAN forwarding complete on
the tested ports and WAN L3 verified; no bench items remain |
| [incoming/upstream-blockers/](incoming/upstream-blockers/) | **ZERO remaining upstream blockers** - the last (flashinfo loader hand-off binding) was closed by boot-tested image `6e9c54d0` with `compatible = "hisilicon,luofu-flashinfo", "hsan,flashinfo_reserved"`; for the record: the pre-clean active deltas (TWD, 6.18 MAC provisioning, switch MMIO-first, cudy-first) were closed on f6685b80/c3ef2a50. The retained files are maintainer-facing documentation/drafts for normal RFC review | not blocker tasks; text-only, not applied; any new active delta still needs its own boot-test |
| [INSTALL-RECOVERY-LESSONS.md](INSTALL-RECOVERY-LESSONS.md) | bench-verified install/recovery lessons (telnet installer end-to-end PASS, pair-b-only rule, **active-pair flashes refused** - installer guard `f188360`, **two-stage post-recovery pair rule**, env-bootflag/cell coupling, mixed-pair diagnosis, physical recovery) **⚠ note** | bench-only record, not a general install or recovery guide |
| [MULTI-PORT-PLAN.md](MULTI-PORT-PLAN.md) | the pre-W22 multi-port Ethernet limit and the source-port experiment (negative), the decision gate and the bench-only probe image `a91ebcf0` / patch 066; a curated note records that **W22 makes eth2 native** (no mirror tokens) and that **M1b-1/M1b-3 proved native per-port delivery on all four jacks and LAN-to-LAN on both tested LAN pairs** (software bridge, no hardware L2 offload) | experiment plan + prepared image; source-port experiment 2026-11-21: negative; W22 supersedes the mirror; LAN multi-port proven on the tested ports and WAN L3 verified; no bench items remain |
| [incoming/multi-port-probe/](incoming/multi-port-probe/) | retained bench-only record: patch 066 (`luofu_switch.mirror_src_mask`) and its note; not part of the release line or upstream series | bench instrument (GPL-2.0-only patch); experiment run 2026-11-21, **negative**; retained as artifact |
| [GPL-REQUEST.md](GPL-REQUEST.md) | GPLv2 §3 source-request record: status, artifact checklist, tracking table, follow-up letter | provenance (unanswered) |
| [SELECTOR-0x21.md](SELECTOR-0x21.md) | reconstruction of the stock A/B boot-selector contract (env `bootflag`, SYSCTRL record cell, the project's `0x21` bench marker) | RE specification, host-only |
| [TRAILER-FORMAT.md](TRAILER-FORMAT.md) | byte-level format of the FIP trailer the stock loader requires (base DTB, 96-byte descriptor, gate magic, DTBO match rule) plus **section 11: the base-DTB totalsize must be 8-byte aligned** for the `bootfip` gate-word load (the `pad-dtb` build requirement) - curated copy; canonical copy sits next to the tool | decoded format, structurally validated + one measured unaligned-load failure |
| [MISC-SOC-BLOCKS.md](MISC-SOC-BLOCKS.md) | RE specification for the misc SoC blocks: factory MAC/efuse, RNG, LED/PMU PWM, CRG clock parents, SYSCTRL/SYSENV - with the measured corrections and the bdinfo-MAC nvmem exposure | RE specification + measured read-only baseline |
| [LSW-SPEC.md](LSW-SPEC.md) | the internal LAN switch (LSW) RE specification: layering, register map, port model, queue/QID model, tables, CFE/rootif contract - including the W3-W20 write campaign, the **measured bench CPU-copy mirror workaround** and the finding that native CFE forwarding is loader/DDR state, not MMIO-replayable | RE specification + measured read-only/write-gated baseline; native forwarding is a separate project |
| [PIE-DATAPATH.md](PIE-DATAPATH.md) | the PIE/GEMAC packet data-path RE specification: layering, registers, rings - with the vendor-exact RX completion decode, the normal non-TSO TX descriptor, and what remains unknown | RE specification + bench-verified through the documented workaround; not an upstream data path |
| [SMP-RE.md](SMP-RE.md) | CPU1 release/SMP RE and the staged bring-up plan, plus the **final diagnosis (sections 21-24)**: the root cause was the loader hand-off devicetree contract, the fix is DTS-only, and the diagnostic probes are excluded from the release line | RE specification + **resolved on hardware** (2026-11-21) |
| [INVENTORY-2026-11-18.md](INVENTORY-2026-11-18.md) | analysis of the first attended read-only silicon inventory: live post-overlay DT windows, the queue/flow DDR base, PIE channel/ring state, LSW W1 baseline and qid tables A/B, CRG `clk_summary`, SYSCTRL cells, the 16 expectation-token corrections and what stays [U] | measured record, read-only: 600/600 reads OK, one stock boot, zero writes |
| [BENCH-READONLY-INVENTORY.md](BENCH-READONLY-INVENTORY.md) | the attended read-only register/table inventory procedure for the stock vendor system: safety contract and gates, static allowlist, evidence format, mock tests, limitations, repeat-run steps | procedure + first run complete (600/600 reads, no writes); repeats need the same attended gates |
| [REMAINING-DRIVERS.md](REMAINING-DRIVERS.md) | evidence survey and port plan for the drivers still missing (Wi-Fi, switch, PIE, GEMAC data path, PCIe, …) | plan |
| [SKELETON-SMOKE.md](SKELETON-SMOKE.md) | the full validation session log: every build, apply, binding check, image and mocked-test result recorded for the target layer - through the 2026-11-19 attended hardware bring-up (first userspace boot), the host-only full-series rebuilds, the 2026-11-20 connectivity campaign (working LAN via the CPU-copy workaround), the 2026-11-20/21 SMP hand-off root-cause and canonical 2-CPU verification, and the board-feature/quiet-image work | validation record + attended bench phases + host-only rebuilds |
| [REVERSAL-REPORT.md](REVERSAL-REPORT.md) | the consolidated bench-reversal report (two tracks: the bench reversal and the upstream preparation) | synthesis, host-only |
| [PUBLISHING.md](PUBLISHING.md) | what this curated repository contains, what is excluded, history/license/sanitization policy, the release-line/board-feature syncs, verification commands | meta |

Additional status pages and retained reports:

- [`../target-skeleton/README.md`](../target-skeleton/README.md) - the detailed status of the target, DTS,
  image recipe, tools and tests, plus the lane records under
  `../target-skeleton/docs/port-notes/` (including `smp-decode`, `qm-decode`, `firstinstall`,
  `overlay`, `pie-decode`, `lsw-egress` and `mac-nvmem`).
- [`../target-skeleton/RELEASE-NOTES.md`](../target-skeleton/RELEASE-NOTES.md) - the short release
  status: bench-verified baselines (known-good `7421bb7a`), the DTS-only SMP fix, the base-DTB
  pad requirement, board features, the verified install path, the DTS cleanup regression, thermal/
  Wi-Fi non-support, canonical configuration and what is not claimed.
- [`../target-skeleton/docs/incoming/feature-inventory/SUMMARY.md`](../target-skeleton/docs/incoming/feature-inventory/SUMMARY.md) -
  retained read-only 2-CPU device feature inventory (full report and the
  [UPDATE-2026-11-21.md](../target-skeleton/docs/incoming/feature-inventory/UPDATE-2026-11-21.md)
  follow-up next to it; the curated notes record the later fixes, the thermal root cause, the
  withdrawn MAC-plan concern, the verified install path and the multi-port limit).
- [`../target-skeleton/docs/incoming/wifi-recon/SUMMARY.md`](../target-skeleton/docs/incoming/wifi-recon/SUMMARY.md) -
  retained host-only Hi5622V100 Wi-Fi feasibility recon: vendor-binary stack, no 6.18 driver,
  option C (unsupported) recommended, hybrid 5.10 as the only Wi-Fi-capable system.
- [`../upstream/series-6.18/README.md`](../upstream/series-6.18/README.md) - what the kernel
  review form is and is not, what was validated, and the decisions required before submitting.
- [`../upstream/series-6.18/05-release-line.md`](../upstream/series-6.18/05-release-line.md) and
  [`../upstream/series-6.18/06-dts-series-dependency.md`](../upstream/series-6.18/06-dts-series-dependency.md) -
  the release line (include/exclude classification, canonical configuration, bench evidence; now
  **53 patches / L0 64 files incl. W22 059-060**, manifest `619eb15a...`, W22 record in
  section 7) and the explicit DTS-series dependency carried by DTS-0001.
- [`../upstream/series-6.18/07-dts-upstream-bindings.md`](../upstream/series-6.18/07-dts-upstream-bindings.md) -
  the board-DTS cleanup record: the hardware regression of the superseded active cleanup, the
  conservative comment-only cleanup, the boot-tested deltas and the closed blocker list (zero remain).
- [`../upstream/series-6.18/submission/README.md`](../upstream/series-6.18/submission/README.md) -
  the **documentation-only upstream submission package** (RFC form): status/artifact checklist,
  one-page cover-email draft, explicit no-Tested-by policy and a file manifest. It now reflects
  the **zero-blocker** final state (flashinfo compatible pair boot-tested on image `6e9c54d0`)
  and the current artifact hashes (DTS `381b2e8c...`; DTS-0001 `886f3ad0...`; DTB
  `62574b11...` / pad `8d79794e...`). **The L2 review form/package is unchanged by W22**
  (patches 059/060 are release-line only), while the release-line manifest is now `619eb15a...`.
  Nothing has been sent anywhere; the manifest was regenerated for this curated tree.
- [`../docs/incoming/upstream-blockers/00-INDEX.md`](../docs/incoming/upstream-blockers/00-INDEX.md) -
  the retained blocker-reduction documentation index (all blockers closed; maintainer-facing
  drafts only, not the release series, not applied).
- [`../NOTICE.md`](../NOTICE.md) - provenance and the no-vendor-code policy.

## Reading order

1. [`../README.md`](../README.md) - what this is, the bench-verified baselines, hardware
   and the honest status table.
2. [RELEASE-NOTES.md](../target-skeleton/RELEASE-NOTES.md) - the 2026-11-21 release status in
   one page.
3. [TARGET-DESIGN.md](TARGET-DESIGN.md) - what is being built and why.
4. [SKELETON-SMOKE.md](SKELETON-SMOKE.md) - what was actually run, on the host and at the bench.
5. [`../upstream/series-6.18/README.md`](../upstream/series-6.18/README.md) - the kernel review
   form, the release line and its validation.
6. The RE specifications ([SELECTOR-0x21](SELECTOR-0x21.md),
   [TRAILER-FORMAT](TRAILER-FORMAT.md), [MISC-SOC-BLOCKS](MISC-SOC-BLOCKS.md),
   [LSW-SPEC](LSW-SPEC.md), [PIE-DATAPATH](PIE-DATAPATH.md), [SMP-RE](SMP-RE.md)) for the
   evidence behind the drivers, with [INVENTORY-2026-11-18](INVENTORY-2026-11-18.md) and
   [BENCH-READONLY-INVENTORY](BENCH-READONLY-INVENTORY.md) as the measured baseline and the
   procedure that produced it.
7. [REMAINING-DRIVERS.md](REMAINING-DRIVERS.md) and [ROADMAP-OPENWRT.md](ROADMAP-OPENWRT.md)
   for what is left, with the retained [feature inventory](../target-skeleton/docs/incoming/feature-inventory/SUMMARY.md)
   and [Wi-Fi recon](../target-skeleton/docs/incoming/wifi-recon/SUMMARY.md) for the device
   matrix and the radio options.

## Link and status conventions

- **`parent:`** references and bare file names such as `STATUS.md`, `BENCH-FACTS.md`,
  `EVIDENCE-INDEX.md` or `mem-entries.md` refer to the lab notebook
  ([cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)).
- Some documents link to notebook files that are **not curated into this repository** (for
  example `FIRST-INSTALL.md`, `VENDOR-SKB-BRIDGE-PORT.md`, `PUBLISH-READINESS.md`,
  `HAZARDS.md`, `FLASH-PLAN.md`, `ETHERNET-BRINGUP-FRAMING.md`, `soc/vendor-dt-notes.md`,
  the `lab/` programs, the per-phase reports and the lane `docs/incoming/` staging packages
  other than the two retained reports named above). Those links resolve in the lab notebook's
  source subtree, not here.
- The retained reports under `../target-skeleton/docs/incoming/` (feature inventory +
  its update, and the Wi-Fi recon) and the bench-only probe record under
  `../docs/incoming/multi-port-probe/` are a deliberate curation exception: they are read-only,
  text-only device reports that document the feature matrix, the install-path findings, the
  multi-port limit and the Wi-Fi feasibility decision. Their `SUMMARY.md` files carry curated
  status notes; no other incoming/staging package is published.
- The owner-only legacy history-purge runbook (`PUBLISH-PURGE-RUNBOOK.md`) and
  `PUBLISH-PREP.md`/`PUBLISH-READINESS.md` are **notebook-side** and not published here;
  `PUBLISH-READINESS.md` item 1 (vendor-object history purge) remains **unchecked** - no rewrite,
  force-push or cache purge has run.
- The bench install tooling lives at [`../tools/`](../tools/): `stock-telnet-install.sh` and
  `telnet-exec.py` are published for evidence, not as a general install/recovery guide (see
  [`../tools/README.md`](../tools/README.md)).
- `~/openwrt-lab*` paths were the author's host build clones and were sanitized to `~`.
  `build/`, `dumps/`, `firmware/`, `rootfs-*/` are private local evidence and are **not**
  published in this repository. The attended bench evidence (2026-11-19 first boot; the
  2026-11-20 connectivity campaign; the 2026-11-20/21 SMP hand-off work and the canonical
  2-CPU soak/reboot logs; the 2026-11-21 board-feature/quiet-image runs) lives in the
  notebook's gitignored build tree; the prose record is in [`../README.md`](../README.md),
  [`../target-skeleton/RELEASE-NOTES.md`](../target-skeleton/RELEASE-NOTES.md) and
  [SKELETON-SMOKE.md](SKELETON-SMOKE.md).
- Statements in the copied documents that the old `opensource/` tree is "NOT READY to
  publish" (a vendor-`.o` history purge) describe the **old repository's history**. This
  repository was created fresh and has never contained those objects; see
  [PUBLISHING.md](PUBLISHING.md).
- Evidence markers used throughout: **[V]** verified from an artifact/measurement,
  **[I]** inferred/planned, **[U]** unmeasured, **[X]** externally blocked.
