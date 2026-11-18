# Documentation index

These documents are a curated snapshot of the lab notebook's engineering record at commit
`6bcea12` (2026-11-18, kernel-patch series `001`-`021`). The live status of the work is the
lab notebook's `STATUS.md`; the honest summary of what is host-validated vs
hardware-unverified is in
[`../README.md`](../README.md). Read [`PUBLISHING.md`](PUBLISHING.md) for the curation rules,
the license split and the sanitization performed.

## Documents

| File | What it is | Status |
| --- | --- | --- |
| [TARGET-DESIGN.md](TARGET-DESIGN.md) | the concrete target/DTS/image design: target layout, board DTS conversion, FIP/trailer/dual-slot/sysupgrade recipe, driver inventory and upstream distance, acceptance criteria, staged plan — three [U] rows closed by the 2026-11-18 read-only inventory | design proposal (host-validated drafts; not accepted upstream) |
| [ROADMAP-OPENWRT.md](ROADMAP-OPENWRT.md) | the phase plan and milestone history; sections that are superseded are marked as such | phase plan / history |
| [GPL-REQUEST.md](GPL-REQUEST.md) | GPLv2 §3 source-request record: status, artifact checklist, tracking table, follow-up letter | provenance (unanswered) |
| [SELECTOR-0x21.md](SELECTOR-0x21.md) | reconstruction of the stock A/B boot-selector contract (env `bootflag`, SYSCTRL record cell, the project's `0x21` bench marker) | RE specification, host-only |
| [TRAILER-FORMAT.md](TRAILER-FORMAT.md) | byte-level format of the FIP trailer the stock loader requires (base DTB, 96-byte descriptor, gate magic, DTBO match rule) — curated copy; canonical copy sits next to the tool | decoded format, structurally validated |
| [MISC-SOC-BLOCKS.md](MISC-SOC-BLOCKS.md) | RE specification for the misc SoC blocks: factory MAC/efuse, RNG, LED/PMU PWM, CRG clock parents, SYSCTRL/SYSENV — with the 2026-11-18 measured CRG/SYSCTRL corrections | RE specification + measured read-only baseline |
| [LSW-SPEC.md](LSW-SPEC.md) | the internal LAN switch (LSW) RE specification: layering, register map, port model — with the measured live windows, W1 register baseline, qid maps A/B and the derived D1b W2 write set | RE specification + measured read-only baseline; no driver has written the switch |
| [PIE-DATAPATH.md](PIE-DATAPATH.md) | the PIE/GEMAC packet data-path RE specification: layering, registers, rings — with the measured live channel/ring state and the token corrections it forced | RE specification + measured read-only baseline; the engine has never been armed by our driver |
| [SMP-RE.md](SMP-RE.md) | decoding of the vendor CPU1 release sequence plus the staged SMP bring-up plan and its unmeasured items — the live pen word confirms the decode on the vendor boot | RE specification, CPU1 never released |
| [INVENTORY-2026-11-18.md](INVENTORY-2026-11-18.md) | analysis of the first attended read-only silicon inventory: live post-overlay DT windows, the queue/flow DDR base, PIE channel/ring state, LSW W1 baseline and qid tables A/B, CRG `clk_summary`, SYSCTRL cells, the 16 expectation-token corrections and what stays [U] | measured record, read-only: 600/600 reads OK, one stock boot, zero writes |
| [BENCH-READONLY-INVENTORY.md](BENCH-READONLY-INVENTORY.md) | the attended read-only register/table inventory procedure for the stock vendor system: safety contract and gates, static allowlist, evidence format, mock tests, limitations, repeat-run steps | procedure + first run complete (600/600 reads, no writes); repeats need the same attended gates |
| [REMAINING-DRIVERS.md](REMAINING-DRIVERS.md) | evidence survey and port plan for the drivers still missing (Wi-Fi, switch, PIE, GEMAC data path, PCIe, …) | plan |
| [SKELETON-SMOKE.md](SKELETON-SMOKE.md) | the full host-validation session log: every build, apply, binding check, image and mocked-test result the project recorded for the target layer, through the 2026-11-18 PIE-datapath (`020`) and LSW-init (`021`) sessions | validation record |
| [REVERSAL-REPORT.md](REVERSAL-REPORT.md) | the consolidated bench-reversal report (two tracks: the bench reversal and the upstream preparation) | synthesis, host-only |
| [PUBLISHING.md](PUBLISHING.md) | what this curated repository contains, what is excluded, history/license/sanitization policy, verification commands | meta |

Additional status pages:

- [`../target-skeleton/README.md`](../target-skeleton/README.md) — the detailed status of the
  schematic target, DTS, image recipe, tools and tests.
- [`../upstream/series-6.18/README.md`](../upstream/series-6.18/README.md) — what the kernel
  series is and is not, what was validated, and the decisions required before submitting.
- [`../NOTICE.md`](../NOTICE.md) — provenance and the no-vendor-code policy.

## Reading order

1. [`../README.md`](../README.md) — what this is, hardware, status table.
2. [TARGET-DESIGN.md](TARGET-DESIGN.md) — what is being built and why.
3. [SKELETON-SMOKE.md](SKELETON-SMOKE.md) — what was actually run on the host.
4. [`../upstream/series-6.18/README.md`](../upstream/series-6.18/README.md) — the kernel
   submission form and its validation.
5. The RE specifications ([SELECTOR-0x21](SELECTOR-0x21.md),
   [TRAILER-FORMAT](TRAILER-FORMAT.md), [MISC-SOC-BLOCKS](MISC-SOC-BLOCKS.md),
   [LSW-SPEC](LSW-SPEC.md), [PIE-DATAPATH](PIE-DATAPATH.md), [SMP-RE](SMP-RE.md)) for the
   evidence behind the drivers, with [INVENTORY-2026-11-18](INVENTORY-2026-11-18.md) and
   [BENCH-READONLY-INVENTORY](BENCH-READONLY-INVENTORY.md) as the measured baseline and the
   procedure that produced it.
6. [REMAINING-DRIVERS.md](REMAINING-DRIVERS.md) and [ROADMAP-OPENWRT.md](ROADMAP-OPENWRT.md)
   for what is left.

## Link and status conventions

- **`parent:`** references and bare file names such as `STATUS.md`, `BENCH-FACTS.md`,
  `EVIDENCE-INDEX.md` or `mem-entries.md` refer to the lab notebook
  ([cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2)).
- Some documents link to notebook files that are **not curated into this repository** (for
  example `FIRST-INSTALL.md`, `VENDOR-SKB-BRIDGE-PORT.md`, `PUBLISH-READINESS.md`,
  `HAZARDS.md`, `FLASH-PLAN.md`, `ETHERNET-BRINGUP-FRAMING.md`, `TARGET-SKELETON.md`,
  `soc/vendor-dt-notes.md`, the `lab/` programs and the per-phase reports). Those links
  resolve in the lab notebook's source subtree, not here.
- `~/openwrt-lab*` paths were the author's host build clones and were sanitized to `~`.
  `build/`, `dumps/`, `firmware/`, `rootfs-*/` are private local evidence and are **not**
  published in this repository.
- Statements in the copied documents that the old `opensource/` tree is "NOT READY to
  publish" (a vendor-`.o` history purge) describe the **old repository's history**. This
  repository was created fresh and has never contained those objects; see
  [PUBLISHING.md](PUBLISHING.md).
- Evidence markers used throughout: **[V]** verified from an artifact/measurement,
  **[I]** inferred/planned, **[U]** unmeasured, **[X]** externally blocked.
