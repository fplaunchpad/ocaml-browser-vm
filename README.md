# ocaml-browser-vm

Bootable Linux VM image (OCaml 5.4 + dune + bisect_ppx) served as
lazy chunks to the in-browser v86 emulator for the NPTEL course
*Functional Programming with OCaml*
(<https://github.com/fplaunchpad/ocaml_nptel>).

This repo holds only the built, student-downloadable VM data,
served via GitHub Pages. The build tooling, design notes, and the
terminal component live in the course repo under `tools/vm-image/`
and `assets/vm/`.

## Layout

One immutable directory per image build:

- `v1/ocaml-state.bin.zst`: zstd-compressed post-boot snapshot
  (downloaded by every visitor at click-to-boot, ~9 MB).
- `v1/ocaml-fs.json`: 9p filesystem metadata.
- `v1/ocaml-rootfs-flat/`: content-addressed zstd chunk store
  (~153 MB on disk; visitors fetch only the chunks their commands
  touch, ~12-53 MB per session).

The three artifacts in a directory MUST come from the same rootfs
build; never mix versions. The course-side component pins one
directory, so adding `v2/` never breaks deployed pages.

## Versions in v1 (built 2026-06-02)

- Base: `i386/alpine:3.23`; OCaml 5.4.0 (bytecode), dune 3.20.2.
- bisect_ppx: pinned to aantron/bisect_ppx PR #448 head `7061d64`
  (ppxlib >= 0.36 / OCaml 5.4 port).
- Sample projects: `hello`, `morse`, `bowling`.
- Built by `tools/vm-image/` at course-repo commit `fd3ac60`.

## Rebuilding

In the course repo (needs Docker, node, python3 + zstandard, zstd):

```sh
bash tools/vm-image/setup-scratch.sh
bash tools/vm-image/image/build.sh
node tools/vm-image/make-state.mjs
zstd -19 -f _vm-prototype/images/ocaml-state.bin \
     -o _vm-prototype/images/ocaml-state.bin.zst
node tools/vm-image/run-workflow.mjs   # must print "workflow complete"
```

Then copy `ocaml-rootfs-flat/`, `ocaml-state.bin.zst`, and
`ocaml-fs.json` into a NEW `vN/` directory here, update this
README's version log, and point the course-side component at the
new directory.
