# Bench install tooling (published for evidence)

These are the bench tools used to verify the **stock -> our-image install path** on the
Cudy WR3000 v2.0 in this project. They are **bench instruments, not a general install or
recovery guide**, and they operate on the vendor stock firmware's root telnet shell.

| Tool | What it does | Safety |
| --- | --- | --- |
| `stock-telnet-install.sh` | fail-closed sysupgrade installer over the stock telnet shell: splits the image, serves/uploads the members, verifies the uploaded hashes, writes the **inactive** pair, verifies the read-back slot hash, verifies the active pair is unchanged, moves the selector, reboots and checks our image answers | `--inspect` is read-only; `--apply` requires `--attended --recovery-armed` and refuses if the bench-link/recovery checks fail. Verified end-to-end on 2026-11-21 (image `7421bb7a`): active pair untouched, selector `0x11` verified, our 6.18 image booted with 2 CPUs |
| `telnet-exec.py` | minimal helper that runs command lines over the stock root telnet shell (plain socket, no auth material, an `__ALL_DONE__` sentinel bounds the read) | read-only helper; the installer is the only caller |

What is **not** in this repository: the bench helpers the installer calls
(`tools/bench-link.sh`, `tools/recovery-card.sh`), the image build and the physical
recovery procedure. They live in the lab notebook and are not published; the installer
refuses without the bench/recovery preflight, and this page is not a substitute for them.
The tool writes the **inactive** pair only when its gates pass - it does not touch the active
pair, repartition, or replace the recovery card.

Why it is published: the install path is now verified on hardware, and this is the tool that
did it. It contains no vendor binaries, firmware blobs, credentials, keys or per-unit
identifiers; the network addresses are vendor/image defaults (`192.168.10.1` stock,
`192.168.10.44` host HTTP, `192.168.1.1` target image). The staging directory is
taken from `$STAGING_DIR` or `$HOME/staging-fi`; no host-specific path is recorded.

License: **MIT** (see `../LICENSE-MIT`). In the notebook these tools carry a GPL-2.0-only
SPDX header; they are the project's own host tooling, published here under MIT per the
repository's content-triggered license split (see `../NOTICE.md`).

Verification record: `target-skeleton/RELEASE-NOTES.md` ("Install path status") and
`target-skeleton/docs/incoming/feature-inventory/UPDATE-2026-11-21.md` ("Follow-up:
automated install tool is now verified").
