# Publishing notes for cudy-wr3000v2-openwrt

This repository is a **curated, fresh-history publication** of the host-validated OpenWrt/Linux
port work for the Cudy WR3000 v2.0 (HiSilicon Hi5671Y "luofu", board R116). This page records
what was taken from where, what was deliberately left out, how the licenses are split, how the
material was sanitized, and how to verify the publication claims.

## Source and snapshot pin

- Source tree: the research lab notebook's `opensource/` subtree
  ([cudy-wr3000v2](https://github.com/ShibbityShwab/cudy-wr3000v2) at its `opensource`
  submodule), **commit `6bcea12`** ("lsw: read-only D1b baseline probe (021)"; kernel-patch
  series 001–021, 2026-11-18).
- This repository was created fresh: `git init`, curated copy, new initial commit(s). **No git
  history was copied from the source repositories**, so no object that ever existed there
  (including the ten vendor `.o` files that were removed from the old repository's HEAD on
  2026-11-17) exists here.
- Curated copies may differ from the source tree only by: (a) added publication/provenance
  pages (`README.md`, `NOTICE.md`, `docs/PUBLISHING.md`, this index, licenses, `.gitignore`),
  (b) absolute host paths sanitized to `~` and the project SSH-key filename genericised,
  (c) tool SPDX headers changed to MIT per the license split in [`NOTICE.md`](../NOTICE.md),
  (d) one documentation copy under `docs/` whose relative links were adjusted.

## What is included (and why)

| Content | Why it is published |
| --- | --- |
| `README.md` | showcase: hardware, honest status table, series summary, target/image and A/B path, safety/legal |
| `docs/` (15 engineering documents) | the design, RE specifications, the measured read-only silicon inventory and its procedure, GPL request provenance and the consolidated reversal report |
| `target-skeleton/` | the schematic OpenWrt target, board DTS, 21 kernel patches (001–021), image recipe, base-files and tools — the actual deliverable |
| `upstream/series-6.18/` | the review-form submission series (cover letter, commit messages, MAINTAINERS proposal, validation logs) |
| `LICENSE-MIT`, `LICENSE-GPL-2.0`, `NOTICE.md` | licenses and provenance |
| `upstream/series-6.18/validation/` | raw host-only validation evidence (apply logs, checkpatch, hashes) |

## What is excluded (and why)

| Excluded | Reason |
| --- | --- |
| vendor firmware images, sysupgrade containers | third-party binaries; not ours to redistribute |
| vendor kernel modules/objects (`.ko`, `.o`, BSP objects), U-Boot images | vendor binary code; contradicts the no-vendor-code policy |
| Wi-Fi firmware blob (`FIRMWARE.bin`) and calibration data | closed vendor blob; radio-calibration data may be unit-specific |
| extracted vendor DTBs/DTBOs and DTS dumps | vendor-derived binary content; our own DTS is included instead |
| NAND/rootfs/U-Boot dumps, register dumps, `dumps/` | local evidence, large, and partly vendor-derived |
| build trees, toolchains, kernel/tarball caches, `build/` | build artifacts; reproducible by the reader from the recipe |
| per-unit identifiers (MAC/serial/PIN/efuse/`secret`/UUID values) | privacy and responsible disclosure; the docs name the offset instead of printing the value |
| credentials, keys, tokens, Lab/bench secrets | never published (and never committed to the source tree either) |
| the `lab/` programs and vendor-module analysis fixtures | out of scope for the OpenWrt port repo; the relevant specifications are in `docs/` |

The tooling and documentation are written so that an owner of the unit can re-derive the
private inputs locally; nothing in this repository depends on redistributing vendor binaries.

## History policy and the no-vendor-binaries claim

This repository's **entire history** is the curated initial commit(s) on `main`. There is no
remote history, no imported refs and no submodule pointing at a repository that contains
vendor objects.

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

The copied documents are **snapshots** written inside the lab notebook on 2026-11-18. Some of
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

Nothing in this repository has been booted or flashed. It is host-validated research material,
not a firmware release; see the safety and legal section of [`../README.md`](../README.md) and
[`../NOTICE.md`](../NOTICE.md).
