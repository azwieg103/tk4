# ZWIEGNET TK4- Distribution Mirror

This repository maintains a checksum-verified GitHub Releases mirror of the
TK4- (Tur(n)key 4-) distribution for ZWIEGNET Hercules installations.

TK4- is a third-party turnkey distribution for IBM MVS 3.8j running under the
Hercules emulator. This repository is a mirror and automation project; it does
not claim authorship of TK4-, MVS 3.8j, Hercules, or bundled software.

## Release assets

Each automated release contains:

- `tk4-_v1.00_current.zip` — validated installation archive.
- `tk4-_v1.00_current.zip.sha256` — SHA-256 checksum.
- `tk4-source.txt` — exact source and provenance used to build the archive.

The release tag contains the archive SHA-256 prefix. A new release is created
only when the archive content changes.

## Automatic build and update process

GitHub Actions checks every six hours and can also be started manually from
**Actions → Mirror upstream TK4- → Run workflow**.

The workflow tries the official upstream HTTPS distribution endpoints first:

1. `https://wotho.ethz.ch/tk4-/`
2. `https://wotho.pebble-beach.ch/tk4-/`

Normal TLS certificate verification is required. TLS checks are never
disabled. If both official endpoints are unavailable or return an invalid
archive, the workflow builds a ZIP with the same TK4- directory layout from
the public GitHub source mirror
[`actualquak/tk4`](https://github.com/actualquak/tk4). The release's
`tk4-source.txt` and release notes explicitly identify that fallback as a
repacked source-mirror snapshot, not the original upstream ZIP.

Every candidate archive is tested and must contain the expected
`conf/tk4-.cnf` configuration and `dasd/mvsres.148` system volume before
publication. The workflow calculates SHA-256 and publishes a new immutable
release only when the archive differs from the previous release.

**Update limitation:** when both official endpoints remain unavailable, the
fallback can only reflect changes made to the public GitHub source mirror.
It cannot discover unpublished changes to the inaccessible official archive.
When the official endpoint becomes available again, the workflow automatically
returns to using the official archive.

## Using the mirror

The Ansible role
[`linux_install_hercules`](https://github.com/azwieg103/ansible/tree/main/roles/linux_install_hercules)
reads the latest release through the GitHub Releases API, verifies the
published SHA-256 asset digest, downloads the archive, and installs it under
`/var/lib/hercules/tk4`.

The cache is `/var/lib/hercules/cache`. Runtime data remains under
`/var/lib/hercules`, backed by the persistent `/data/hercules` bind mount.

## Upstream sources and attribution

- Official primary: https://wotho.ethz.ch/tk4-/
- Official alternate: https://wotho.pebble-beach.ch/tk4-/
- GitHub fallback source: https://github.com/actualquak/tk4

The upstream TK4- distribution includes its own documentation, credits, and
third-party license notices. Those notices must be retained. The TK4- author
describes the distribution as public domain to the extent that third-party
copyrights are not affected; bundled components retain their own terms.
Consult `TK4-.CREDITS` and all included notices before redistribution.

The root `LICENSE` applies only to original repository-authored automation
and documentation to the extent those files are copyrightable. It does not
relicense TK4- or any bundled component.

## Repository contents

```text
.github/workflows/mirror-tk4.yml   Scheduled source check and release publisher
README.md                          Project and operating documentation
LICENSE                            License for original repository-authored files
```

## Verify a downloaded release

```bash
sha256sum -c tk4-_v1.00_current.zip.sha256
unzip -t tk4-_v1.00_current.zip
```
