# ocaml-browser-vm

Bootable Linux VM image (OCaml 5.4 + dune + bisect_ppx) served as
lazy chunks to the in-browser v86 emulator for the NPTEL course
*Functional Programming with OCaml*
(<https://github.com/fplaunchpad/ocaml_nptel>).

This repo holds only the built, student-downloadable VM data,
served via GitHub Pages. The build tooling, design notes, and the
terminal component live in the course repo:
<https://github.com/fplaunchpad/ocaml_nptel> (directories
`tools/vm-image/` and `assets/vm/` there, not here).

## Layout

One immutable directory per image build:

- `vN/ocaml-state.bin.zst`: zstd-compressed post-boot snapshot
  (downloaded by every visitor at click-to-boot, ~9 MB).
- `vN/ocaml-fs.json`: 9p filesystem metadata.
- `vN/ocaml-rootfs-flat/`: content-addressed zstd chunk store
  (~153 MB on disk; visitors fetch only the chunks their commands
  touch, ~12-53 MB per session).

The three artifacts in a directory MUST come from the same rootfs
build; never mix versions. The course-side component pins one
directory, so adding `v2/` never breaks deployed pages.

## Version log

### v3 (built 2026-06-02; current)

- As v2, plus: bowling's instrumented `_build` is pre-baked (the
  first `dune runtest --instrument-with bisect_ppx` drops from
  ~90 s to ~20 s; the ppx-driver link dominates and is cached),
  and fast-writeback sysctls (`vm.dirty_*_centisecs = 200`) so
  freshly written guest files reach the 9p backend promptly.

### v2 (built 2026-06-02)

- As v1, plus opam libraries `ounit2` and `qcheck` (module 9's
  testing libraries); the `morse` sample's tests are now a real
  OUnit2 suite.

### v1 (built 2026-06-02)

- Base: `i386/alpine:3.23`; OCaml 5.4.0 (bytecode), dune 3.20.2.
- bisect_ppx: pinned to aantron/bisect_ppx PR #448 head `7061d64`
  (ppxlib >= 0.36 / OCaml 5.4 port).
- Sample projects: `hello`, `morse`, `bowling`.
- Built by the course repo's `tools/vm-image/` at commit
  [`fd3ac60`](https://github.com/fplaunchpad/ocaml_nptel/commit/fd3ac60).

## Rebuilding

All paths below are inside a clone of the course repo
(<https://github.com/fplaunchpad/ocaml_nptel>), not this one.
Requirements: Docker, node, python3 with the `zstandard` module,
and the `zstd` CLI. The first command creates `_vm-prototype/`, an
untracked scratch directory at the course repo's root that holds
the pinned third-party inputs and all build outputs.

```sh
bash tools/vm-image/setup-scratch.sh
bash tools/vm-image/image/build.sh
node tools/vm-image/make-state.mjs
zstd -19 -f _vm-prototype/images/ocaml-state.bin \
     -o _vm-prototype/images/ocaml-state.bin.zst
node tools/vm-image/run-workflow.mjs   # must print "workflow complete"
```

Then copy `ocaml-rootfs-flat/`, `ocaml-state.bin.zst`, and
`ocaml-fs.json` from `_vm-prototype/images/` into a NEW `vN/`
directory in THIS repo, update the version log above, and bump
`DEFAULT_BASE` in the course repo's `assets/vm/vm-terminal.js`.
