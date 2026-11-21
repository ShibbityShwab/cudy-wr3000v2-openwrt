# Install and recovery lessons (bench-verified, 2026-11-21)

**Status: bench-only engineering record - NOT a general install or recovery guide.** Every
hardware statement below was observed on the single bench unit with a human present and the
physical recovery door armed. **Recovery is physical** (power-off + RESET-hold + TFTP stock
image) and cannot be performed by an agent. The installer published under [`tools/`](../tools/) is
evidence/reproducibility tooling, not a supported installer or recovery service.

## Verified install transport

The path that actually ran is the **stock telnet root shell** infrastructure
(`tools/stock-telnet-install.sh` + `tools/telnet-exec.py`; handover fix in the notebook's
installer commit `6e69aa3`):

- read-only inspection by default; writes only with `--apply --attended --recovery-armed`;
- writes **pair b only** (kernelb `mtd12`, rootfsb `mtd14`), leaving the active pair unchanged;
- verifies uploads, read-backs and the active-pair-unchanged check, and sets + reads back the
  U-Boot env `bootflag`;
- moves the selector cell, reboots, and requires the booted cmdline to contain
  `ubi.mtd=rootfsb` and the project marker.

Verified end-to-end on 2026-11-21 (flashinfo-test image `6e9c54d0` PASS; earlier runs the same day
installed the `974e6ca6`/`7421bb7a`-generation images). Image hashes and evidence markers are in
[`../target-skeleton/RELEASE-NOTES.md`](../target-skeleton/RELEASE-NOTES.md).

## Lessons learned on the unit

1. **Set the U-Boot env `bootflag` together with the selector cell.** The loader reads the
   kernel slot named by the env; a cell-only handover after a RESET-hold recovery can pair our
   kernel with the stock rootfs. The telnet installer refuses when env and cell disagree.
2. **Our image can only be written to pair b.** Its baked cmdline uses `ubi.mtd=rootfsb`, so
   writing pair a after a recovery produces the mixed our-kernel/stock-rootfs state.
3. **The broker channel (CVE-2026-71961) only works on a consistent vendor pair.** The vendor
   bdinfo checkuuid reads `/proc/sys/dev/flash_uuid`; in the mixed state neither the vendor web
   wizard nor cmagent works.
4. **Recovery is physical:** RESET-hold + TFTP stock image to restore a consistent pair, then the
   env-bootflag + selector handover above. UART is read-only on this unit.
5. **The older first-install design needs SSH** (`first-install-stock.sh` + the notebook's
   `FIRST-INSTALL.md`); the vendor image has no dropbear and no opkg, so that path was never
   hardware-run and the telnet installer is the bench path.

## Safety invariants carried into this repository

- no blind flash writes: slot pair, env bootflag, selector and post-boot cmdline all verified;
- recovery door armed before any fire; UART captured read-only; no agent-only operation;
- no credentials, per-unit identifiers, vendor binaries or host paths in the published files.

Source: notebook `opensource/docs/FIRST-INSTALL.md` 2026-11-21 bench update,
[`../target-skeleton/RELEASE-NOTES.md`](../target-skeleton/RELEASE-NOTES.md) install-path section, and the
parent bench ledger entries for 2026-11-21.
