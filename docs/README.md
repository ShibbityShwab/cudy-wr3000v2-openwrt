# Documentation index

These documents are a curated snapshot of the lab notebook's engineering record at the
**6.18 release-line consolidation** - source `opensource` submodule commit `a6b46a9` (2026-11-21):
the **51-patch release line** (62 patch files: 001-058 + 062-065; the 11 bench-only SMP
diagnostics are excluded), the **DTS-only loader hand-off SMP fix** that is the release-critical
change, the **DTS-0001** upstream DTS-series companion, and the bench-verified 2-CPU baseline
image `974e6ca6`. The upstream review form stays at 26 patches (001-026). The live status of the
work is the lab notebook's `STATUS.md`; the honest summary of what is host-validated and what the
attended bench sessions did - and did **not** - prove is in [`../README.md`](../README.md). Read
[`PUBLISHING.md`](PUBLISHING.md) for the curation rules, the license split, the sanitization
performed and the 2026-11-18/19/21 syncs.

## Documents

| File | What it is | Status |
| --- | --- | --- |
| [TARGET-DESIGN.md](TARGET-DESIGN.md) | the concrete target/DTS/image design: target layout, board DTS conversion, FIP/trailer/dual-slot/sysupgrade recipe, driver inventory and upstream distance, acceptance criteria, staged plan - with the 2026-11-21 release update, the mandatory loader hand-off DTS contract, the closed SMP row and the partially closed install/boot rows | design proposal (host-validated drafts + bench-verified target image; not accepted upstream) |
| [TARGET-SKELETON.md](TARGET-SKELETON.md) | the target-skeleton index: what the schematic target, DTS, image recipe, tools and patch stack are; status and what remains [U]/TODO | index (updated 2026-11-21) |
| [ROADMAP-OPENWRT.md](ROADMAP-OPENWRT.md) | the phase plan and milestone history; the 2026-11-21 update closes the SMP milestone without claiming upstream progress; superseded sections are marked | phase plan / history |
| [GPL-REQUEST.md](GPL-REQUEST.md) | GPLv2 §3 source-request record: status, artifact checklist, tracking table, follow-up letter | provenance (unanswered) |
| [SELECTOR-0x21.md](SELECTOR-0x21.md) | reconstruction of the stock A/B boot-selector contract (env `bootflag`, SYSCTRL record cell, the project's `0x21` bench marker) | RE specification, host-only |
| [TRAILER-FORMAT.md](TRAILER-FORMAT.md) | byte-level format of the FIP trailer the stock loader requires (base DTB, 96-byte descriptor, gate magic, DTBO match rule) - curated copy; canonical copy sits next to the tool | decoded format, structurally validated; a novel trailer is still not boot-tested |
| [MISC-SOC-BLOCKS.md](MISC-SOC-BLOCKS.md) | RE specification for the misc SoC blocks: factory MAC/efuse, RNG, LED/PMU PWM, CRG clock parents, SYSCTRL/SYSENV - with the 2026-11-18 measured corrections and the bdinfo-MAC nvmem exposure | RE specification + measured read-only baseline |
| [LSW-SPEC.md](LSW-SPEC.md) | the internal LAN switch (LSW) RE specification: layering, register map, port model, queue/QID model, tables, CFE/rootif contract - extended by the 2026-11-19/20 W3-W20 write campaign, the **measured bench CPU-copy mirror workaround** (`cpu_mirror=1 mirror_dst=4` + PIE port-0 copy delivery) and the finding that native CFE forwarding is loader/DDR state, not MMIO-replayable | RE specification + measured read-only/write-gated baseline; native forwarding is a separate project |
| [PIE-DATAPATH.md](PIE-DATAPATH.md) | the PIE/GEMAC packet data-path RE specification: layering, registers, rings - with the vendor-exact RX completion decode, the normal non-TSO TX descriptor, the W20 CPU-copy delivery path that made the bench LAN work, and what remains unknown | RE specification + bench-verified through the documented workaround; not an upstream data path |
| [SMP-RE.md](SMP-RE.md) | CPU1 release/SMP RE and the staged bring-up plan, plus the **final diagnosis (sections 21-24)**: the root cause was the loader hand-off devicetree contract, the fix is DTS-only, and the diagnostic probes are excluded from the release line | RE specification + **resolved on hardware** (2026-11-21) |
| [INVENTORY-2026-11-18.md](INVENTORY-2026-11-18.md) | analysis of the first attended read-only silicon inventory: live post-overlay DT windows, the queue/flow DDR base, PIE channel/ring state, LSW W1 baseline and qid tables A/B, CRG `clk_summary`, SYSCTRL cells, the 16 expectation-token corrections and what stays [U] | measured record, read-only: 600/600 reads OK, one stock boot, zero writes |
| [BENCH-READONLY-INVENTORY.md](BENCH-READONLY-INVENTORY.md) | the attended read-only register/table inventory procedure for the stock vendor system: safety contract and gates, static allowlist, evidence format, mock tests, limitations, repeat-run steps | procedure + first run complete (600/600 reads, no writes); repeats need the same attended gates |
| [REMAINING-DRIVERS.md](REMAINING-DRIVERS.md) | evidence survey and port plan for the drivers still missing (Wi-Fi, switch, PIE, GEMAC data path, PCIe, …) | plan |
| [SKELETON-SMOKE.md](SKELETON-SMOKE.md) | the full validation session log: every build, apply, binding check, image and mocked-test result recorded for the target layer - through the 2026-11-19 attended hardware bring-up (first userspace boot), the host-only full-series rebuilds, the 2026-11-20 connectivity campaign (working LAN via the CPU-copy workaround), and the 2026-11-20/21 SMP hand-off root-cause plus the canonical 2-CPU 974e6ca6 verification | validation record + two attended bench sessions + host-only rebuilds |
| [REVERSAL-REPORT.md](REVERSAL-REPORT.md) | the consolidated bench-reversal report (two tracks: the bench reversal and the upstream preparation) | synthesis, host-only |
| [PUBLISHING.md](PUBLISHING.md) | what this curated repository contains, what is excluded, history/license/sanitization policy, the release-line sync, verification commands | meta |

Additional status pages:

- [`../target-skeleton/README.md`](../target-skeleton/README.md) - the detailed status of the target, DTS,
  image recipe, tools and tests, plus the lane records under `../target-skeleton/docs/port-notes/`
  (including the `smp-decode`, `qm-decode`, `firstinstall`, `overlay`, `pie-decode`, `lsw-egress`
  and `mac-nvmem` records).
- [`../target-skeleton/RELEASE-NOTES.md`](../target-skeleton/RELEASE-NOTES.md) - the short release
  status: bench-verified 2-CPU baseline, the DTS-only SMP fix, canonical configuration, what
  works and what is not claimed.
- [`../upstream/series-6.18/README.md`](../upstream/series-6.18/README.md) - what the kernel
  review form is and is not, what was validated, and the decisions required before submitting.
- [`../upstream/series-6.18/05-release-line.md`](../upstream/series-6.18/05-release-line.md) and
  [`../upstream/series-6.18/06-dts-series-dependency.md`](../upstream/series-6.18/06-dts-series-dependency.md) -
  the release line (include/exclude classification, canonical configuration, bench evidence)
  and the explicit DTS-series dependency carried by DTS-0001.
- [`../NOTICE.md`](../NOTICE.md) - provenance and the no-vendor-code policy.

## Reading order

1. [`../README.md`](../README.md) - what this is, the bench-verified baseline, hardware
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
   for what is left.

## Link and status conventions

- **`parent:`** references and bare file names such as `STATUS.md`, `BENCH-FACTS.md`,
  `EVIDENCE-INDEX.md` or `mem-entries.md` refer to the lab notebook
  ([cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)).
- Some documents link to notebook files that are **not curated into this repository** (for
  example `FIRST-INSTALL.md`, `VENDOR-SKB-BRIDGE-PORT.md`, `PUBLISH-READINESS.md`,
  `HAZARDS.md`, `FLASH-PLAN.md`, `ETHERNET-BRINGUP-FRAMING.md`, `soc/vendor-dt-notes.md`,
  the `lab/` programs, the per-phase reports and the lane `docs/incoming/` staging packages).
  Those links resolve in the lab notebook's source subtree, not here.
- `~/openwrt-lab*` paths were the author's host build clones and were sanitized to `~`.
  `build/`, `dumps/`, `firmware/`, `rootfs-*/` are private local evidence and are **not**
  published in this repository. The attended bench evidence (2026-11-19 first boot; the
  2026-11-20 connectivity campaign; the 2026-11-20/21 SMP hand-off work and the canonical
  2-CPU soak/reboot logs such as `2026-11-21-soak-2cpu.log`) lives in the notebook's gitignored
  build tree; the prose record is in [`../README.md`](../README.md),
  [`../target-skeleton/RELEASE-NOTES.md`](../target-skeleton/RELEASE-NOTES.md) and
  [SKELETON-SMOKE.md](SKELETON-SMOKE.md).
- Statements in the copied documents that the old `opensource/` tree is "NOT READY to
  publish" (a vendor-`.o` history purge) describe the **old repository's history**. This
  repository was created fresh and has never contained those objects; see
  [PUBLISHING.md](PUBLISHING.md).
- Evidence markers used throughout: **[V]** verified from an artifact/measurement,
  **[I]** inferred/planned, **[U]** unmeasured, **[X]** externally blocked.
