# sysutils/qtpass port test bed

CI for the FreeBSD port of [QtPass](https://qtpass.org). Every patch in
`patches/` is the diff that goes to
[Bugzilla](https://bugs.freebsd.org/bugzilla/) for the port; this repository
applies it to a fresh FreeBSD ports tree inside a FreeBSD VM and runs what a
ports committer would before committing:

1. `git apply --check` on the ports tree (`main` by default; a quarterly branch
   on request)
2. `portlint -AC`
3. `make checksum`, then `make makesum` must reproduce the patched `distinfo`
4. `make stage`, `make check-plist`, `make stage-qa`
5. `make package`, `pkg add`, `ldd` of the installed binary
6. QtPass's own test suite (`make check`, offscreen) on the port's build tree

Matrix: FreeBSD 14.5 amd64, 15.1 amd64, 14.5 aarch64 (emulated, slow), via
[vmactions/freebsd-vm](https://github.com/vmactions/freebsd-vm).

## Status

- 1.8.0: [committed](https://cgit.freebsd.org/ports/commit/?id=9c04e8538cc4e00cae53db602d70ccefef79ac7e)
  2026-09-14 (Bugzilla 298494), from `patches/qtpass-1.8.0.patch`; the
  committer added `files/patch-qtpass.desktop` so upstream's translated
  desktop file is installed with absolute `Exec`/`Icon` paths.

## Usage

- Push a new `patches/qtpass-X.Y.Z.patch`: the newest patch (by version) is
  tested on `ports/main`.
- Actions → _sysutils/qtpass port test_ → _Run workflow_: choose a patch, a
  ports branch (`main`, `2026Q3`, …) and whether to run the test suite.

Producing a patch: check out `ports`, edit `sysutils/qtpass/Makefile`, run
`make makesum`, `git diff > qtpass-X.Y.Z.patch`.

## Why

The port sat on 1.4.0 from 2023 to 2026. The QtPass release checklist now
includes bumping the downstream packages on release day; this is where the
FreeBSD half gets verified without a local FreeBSD machine. Maintainer of the
port and of QtPass: Anne Jan Brouwer.

## License

BSD-2-Clause, like the FreeBSD ports tree the patches are diffs against.
