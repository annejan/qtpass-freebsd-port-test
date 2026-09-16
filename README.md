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

A second job runs `poudriere testport -o sysutils/qtpass` on 14.5 and 15.1
amd64 and in a native i386 jail on 14.5: a clean jail with only the
dependencies the port declares, so a missing `*_DEPENDS` fails there even
though the first job (which pre-installs the dependencies with `pkg`) would
not notice. Dependencies are fetched from pkg.FreeBSD.org
(`PACKAGE_FETCH_*`); only qtpass itself is built — unless the package set
lags the ports tree, in which case poudriere rebuilds whatever the fetched
packages no longer match (a glib bump on `main` once meant rebuilding the
Qt6 chain: 84 minutes on 14.5 while 15.1, whose set was current, took 13).

A third job tests the OpenBSD port, `security/qtpass`, from
`patches/openbsd/` (a diff against ports -current) on an OpenBSD 7.9 VM:
`portcheck`, `makesum`, `build`, `fake`, `update-plist` (must reproduce the
patched `PLIST`), `port-lib-depends-check`, `package`, `install`. The ports
tree is the 7.9 release's with `security/qtpass` taken from -current; a full
-current tree on a release does not work (its shared-library versions and
package names are ahead of the release packages). The dependencies are
pre-installed from the 7.9 package mirror, so like the first FreeBSD job it
does not notice a missing dependency.

### Architectures and BSDs not covered, and why

- FreeBSD powerpc64/powerpc64le: pkg.FreeBSD.org publishes only `pkgbase`
  for them, no ports packages, so Qt6 would have to be built from source
  under emulation.
- FreeBSD armv7: packages exist but the set is stale (`qt6-base` 6.9.1
  against 6.11 in the tree), so poudriere would rebuild Qt6.
- FreeBSD riscv64: no official packages; the community repository used by
  the riscv64 VM images has Qt5 only.
- Native aarch64 runners (`ubuntu-24.04-arm`): vmactions/freebsd-vm
  documents them as slower than the emulated VM on an x86_64 runner.
- NetBSD/pkgsrc: no qtpass package.
- DragonFly: DPorts is derived from the FreeBSD ports tree; nothing to test
  separately.

## Status

FreeBSD `sysutils/qtpass`:

- 1.8.1: [committed](https://cgit.freebsd.org/ports/commit/?id=afc35ca3a267dea25f379e57a5c88a22f52767b4)
  2026-09-15, from `patches/qtpass-1.8.1.patch`.
- 1.8.0: [committed](https://cgit.freebsd.org/ports/commit/?id=9c04e8538cc4e00cae53db602d70ccefef79ac7e)
  2026-09-14 (Bugzilla 298494), from `patches/qtpass-1.8.0.patch`; the
  committer added `files/patch-qtpass.desktop` so upstream's translated
  desktop file is installed with absolute `Exec`/`Icon` paths.

OpenBSD `security/qtpass` (maintainer Stefan Hagen, currently 1.7.0, Qt5):

- 1.8.1: `patches/openbsd/qtpass-1.8.1.patch`, not yet sent to ports@.

## Usage

- Push a new `patches/qtpass-X.Y.Z.patch`: the newest patch (by version) is
  tested on `ports/main`. Once the patch has been committed to that branch it
  no longer applies; the workflow notices (it applies in reverse) and tests
  the tree as committed.
- Push a new `patches/openbsd/qtpass-X.Y.Z.patch`: likewise for the OpenBSD
  port.
- Actions → _sysutils/qtpass port test_ → _Run workflow_: choose a patch, a
  ports branch (`main`, `2026Q3`, …), whether to run the test suite, and
  optionally an OpenBSD patch.

Producing a FreeBSD patch: check out `ports`, edit `sysutils/qtpass/Makefile`,
run `make makesum`, `git diff > qtpass-X.Y.Z.patch`. For OpenBSD: clone
[openbsd/ports](https://github.com/openbsd/ports), edit
`security/qtpass/{Makefile,pkg/PLIST}`, run `make makesum`, `git diff`.

## Why

The port sat on 1.4.0 from 2023 to 2026. The QtPass release checklist now
includes bumping the downstream packages on release day; this is where the
FreeBSD half gets verified without a local FreeBSD machine. Maintainer of the
port and of QtPass: Anne Jan Brouwer.

## License

BSD-2-Clause, like the FreeBSD ports tree the patches are diffs against.
