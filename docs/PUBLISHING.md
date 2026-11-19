# Publishing notes for cudy-wr3000v2-openwrt

This repository is a **curated, fresh-history publication** of the host-validated OpenWrt/Linux
port work for the Cudy WR3000 v2.0 (HiSilicon Hi5671Y "luofu", board R116). This page records
what was taken from where, what was deliberately left out, how the licenses are split, how the
material was sanitized, and how to verify the publication claims.

## Source and snapshot pin

- Source tree: the research lab notebook's `opensource/` subtree
  ([cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2) at its `opensource`
  submodule), **commit `12eb48c`** ("bring-up: first full boot of our kernel+rootfs on
  WR3000 v2; fix cmdline/padding/XZ", 2026-11-19; kernel-patch series 001-021 and the upstream
  review form for 001-021, 21 patches validated). It adds the first-boot bring-up machinery on
  top of the previous curated snapshot at source commit `9d5005c` (2026-11-18).
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
  (f) the second sync from source commit `12eb48c` (2026-11-19) brings the first-boot
      configuration/image/tools changes, the TARGET-DESIGN/SKELETON-SMOKE/DTS updates and one
      new MIT-licensed host tool (`target-skeleton/tools/ubi-volume-check.py`, SPDX header
      added and usage genericised for this repository);
  (g) one factual correction in the synced 2026-11-19 session record: the source text said the
      boot came up with "2 CPUs via the SMP release", but the cited boot log's SMP line
      reports `Brought up 1 node, 1 CPU`; the curated copy states the measured count, and
      the same claim should be corrected in the notebook; and
  (h) a short curated update note at the top of `docs/SKELETON-SMOKE.md` (and an updated
      curated copy header in `target-skeleton/README.md`) so the 2026-11-18 "nothing has
      been booted" statements are not read as current after the 2026-11-19 boot.

## What is included (and why)

| Content | Why it is published |
| --- | --- |
| `README.md` | showcase: hardware, honest status table, series summary, target/image and A/B path, safety/legal |
| `docs/` (14 engineering documents plus the index and this policy) | the design, RE specifications, the measured read-only silicon inventory and its procedure, GPL request provenance, the consolidated reversal report and the 2026-11-19 boot record |
| `target-skeleton/` | the schematic OpenWrt target, board DTS, 21 kernel patches (001–021), image recipe, base-files and tools — the actual deliverable |
| `upstream/series-6.18/` | the 21-patch review-form submission series (cover letter, per-patch commit messages, MAINTAINERS proposal, tree/order rationale, validation logs); patches `020`/`021` are carried as do-not-merge drafts |
| `LICENSE-MIT`, `LICENSE-GPL-2.0`, `NOTICE.md` | licenses and provenance |
| `upstream/series-6.18/validation/` | raw host-only validation evidence (apply logs, checkpatch, hashes) |

### 2026-11-19 sync and the first hardware result

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

The same sync carries the repository's **only hardware result**: a 6.18.60 kernel + OpenWrt
rootfs build booted to a userspace console in an attended bench session on 2026-11-19, behind
an armed recovery door and with a read-only UART capture. The image running on the unit is the
debug build; the clean rebuild (sha256 `e893c388...`) was not re-flashed, and there is still
no network, no persistent overlay and no A/B sysupgrade run. See `README.md`.

The source commit also added one Python bytecode cache
(`target-skeleton/tools/__pycache__/*.pyc`); this repository's `.gitignore` excludes build
products and caches, and the curated tree deliberately does not contain it.

## What is excluded (and why)

| Excluded | Reason |
| --- | --- |
| vendor firmware images, sysupgrade containers | third-party binaries; not ours to redistribute |
| vendor kernel modules/objects (`.ko`, `.o`, BSP objects), U-Boot images | vendor binary code; contradicts the no-vendor-code policy |
| Wi-Fi firmware blob (`FIRMWARE.bin`) and calibration data | closed vendor blob; radio-calibration data may be unit-specific |
| extracted vendor DTBs/DTBOs and DTS dumps | vendor-derived binary content; our own DTS is included instead |
| NAND/rootfs/U-Boot dumps, register dumps, `dumps/` | local evidence, large, and partly vendor-derived |
| build trees, toolchains, kernel/tarball caches, `build/` | build artifacts; reproducible by the reader from the recipe |
| Python bytecode caches and UART/boot evidence logs | build/evidence artifacts from the lab; the source commit `12eb48c` added a `__pycache__/*.pyc` file that is deliberately not synced, and the 2026-11-19 boot logs stay private lab evidence described only in prose |
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
| Kernel code: patches 001–021, draft DTS/DTBO, OpenWrt target, image recipe, `base-files/` | **GPL-2.0-only** (`LICENSE-GPL-2.0`; SPDX header in each file) |

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

The copied documents are **snapshots** written inside the lab notebook on 2026-11-18 (with the
2026-11-19 attended bring-up session appended to `SKELETON-SMOKE.md` and `TARGET-DESIGN.md`
5.5/5.6 updated in the second sync). Some of
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

One attended bench bring-up (2026-11-19) booted a build of this tree to a userspace console;
that is the only hardware result and it is not an install or recovery procedure. The clean
rebuild was not re-flashed, so the repository remains curated research material, not a firmware
release; see the safety and legal section of [`../README.md`](../README.md) and
[`../NOTICE.md`](../NOTICE.md).
