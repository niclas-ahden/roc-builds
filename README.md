# roc-builds

Builds of exact [roc-lang/roc](https://github.com/roc-lang/roc) commits for
CI. Our platforms and packages pin the compiler to a commit of roc's main, and
most commits have no [nightly](https://github.com/roc-lang/nightlies), so each
repository used to build the compiler itself, on every system, for every pin.
Here it is built once per commit and published as a release that any
repository downloads.

## Using it

```yaml
- uses: niclas-ahden/roc-builds@<commit of this repository>
```

puts on PATH the compiler of the commit the repository's `flake.lock` locks as
`roc-src`. Pass `commit: <full hash>` to name another one. On Linux (x86_64,
arm64), macOS (Intel, Apple silicon) and Windows (x86_64) it downloads the
archive published for that commit and checks it against the SHA-256 in
`sums/roc-<full hash>.SHA256SUMS` of the action's own commit. Pin the action by
commit, not by tag: the pin then fixes the compilers it accepts too, and an
archive replaced in the release afterwards stops the job.

When the action's commit holds no checksums for the roc commit, it builds that
commit from source and keeps it in the calling repository's Actions cache, and
its log says how to publish the build here.

## Publishing a commit

```sh
gh workflow run build.yml -R niclas-ahden/roc-builds -f commit=<hash>
```

builds the commit on every system with the Zig version its `build.zig.zon`
names, publishes a release tagged `roc-<full hash>` and commits the archives'
checksums to `sums/`. Pin that commit in the repositories that use the new
compiler. The steps and the archive layout follow roc-lang/nightlies, so an
archive holds the `roc` binary, roc's `LICENSE` and `legal_details`, and on
macOS the Darwin sysroot. A commit that is already published is skipped unless
the run passes `force`, which replaces the archives and their checksums, so
repositories pinned to an older commit of this one stop accepting them.

The Apple silicon build runs on `roc-mac`, a self-hosted runner on a Mac, until
GitHub hands this account Apple silicon runners. Workflows from pull requests
of outside contributors wait for approval, so no one else can run code on it.

`gh workflow run test.yml -R niclas-ahden/roc-builds -f commit=<full hash>`
installs a published commit with the action on every system and runs it.

The compiler is roc-lang/roc's, under its license.
