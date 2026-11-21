# Multi-port source-port probe (bench-only)

This directory retains the **bench-only** multi-port experiment record from the lab notebook:

- `066-net-lsw-mirror-src-mask.patch` - a draft kernel patch (GPL-2.0-only, our own code) that
  replaces the hardcoded ingress-mirror source mask `BIT(11)` in `luofu-switch.c` with a
  `luofu_switch.mirror_src_mask` module parameter (default `0x800`), plus the matching
  read-back check. It was used only to build the prepared probe image **a91ebcf0**
  (known-good userspace/kernel plus the probe patch and the built-in probe tokens
  `luofu_pie.rxdiag=1 luofu_switch.mirror_src_mask=0x1e00`).
- It is **not** part of the release line or the upstream review form; the build clone was
  restored to the canonical patch set and config after the image was produced.
- The experiment plan, attended gates and decision gate are in
  [../../MULTI-PORT-PLAN.md](../../MULTI-PORT-PLAN.md). The probe has not been run; no multi-port
  claim is made.

License: the patch is a kernel-code diff and is GPL-2.0-only (see `../../NOTICE.md`); this
README/note is MIT like the rest of the documentation.
