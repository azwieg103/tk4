# ZWIEGNET TK4- Distribution Mirror

This repository maintains a checksum-verified GitHub Releases mirror of the
upstream TK4- (Tur(n)key 4-) distribution for ZWIEGNET Hercules installations.

TK4- is a third-party turnkey distribution for IBM MVS 3.8j running under the
Hercules emulator. This repository is a mirror and automation project; it does
not claim authorship of TK4-, MVS 3.8j, Hercules, or any software bundled in
the upstream archive.

## What is mirrored

The workflow downloads the upstream `tk4-_v1.00_current.zip` archive and
publishes the exact archive as a versioned GitHub Release asset. The archive
is not committed to Git, keeping the repository small and making installations
repeatable.

Each generated release includes:

- `tk4-_v1.00_current.zip` — unmodified upstream distribution archive.
- `tk4-_v1.00_current.zip.sha256` — SHA-256 checksum for the archive.

Release tags include the content digest, for example
`tk4-1.00-<sha256-prefix>`. If the upstream archive changes while retaining
the same filename, the changed content produces a new release.

## Automatic update schedule

GitHub Actions checks the upstream distribution every six hours. It can also
be started manually from **Actions → Mirror upstream TK4- → Run workflow**.

A new GitHub Release is published only when the downloaded archive's SHA-256
differs from the latest mirrored release. The workflow:

1. Downloads from the upstream primary HTTPS endpoint.
2. Tries the documented alternate HTTPS mirror if the primary download fails.
3. Requires normal TLS certificate verification for both endpoints.
4. Tests the ZIP archive before publishing it.
5. Calculates SHA-256 and compares it with the latest mirror release.
6. Publishes a new immutable, digest-tagged release when content changes.

TLS verification is never disabled. If both upstream endpoints fail
certificate or network validation, the workflow fails without publishing an
unverified archive.

## Using the mirror

The Ansible role
[`linux_install_hercules`](https://github.com/azwieg103/ansible/tree/main/roles/linux_install_hercules)
reads the latest release through the GitHub Releases API, verifies the
published SHA-256 asset digest, downloads the archive, and installs it under
`/var/lib/hercules/tk4`.

The archive is stored in the host's Hercules cache at
`/var/lib/hercules/cache`. The role keeps all runtime paths under
`/var/lib/hercules`, backed by the persistent `/data/hercules` bind mount.

## Upstream sources

- Primary distribution: https://wotho.ethz.ch/tk4-/
- Alternate distribution mirror: https://wotho.pebble-beach.ch/tk4-/

The workflow uses the upstream current archive URL. The upstream filename is
not treated as a version identifier; content SHA-256 determines whether a new
mirror release is required.

## Licensing and attribution

The upstream TK4- archive contains its own documentation, credits, and
third-party license notices. Those notices remain part of the unmodified
archive and must be retained with it. The rights and terms for TK4-, MVS 3.8j,
Hercules, and each bundled third-party component are determined by their
respective authors and license documents, not by this mirror.

The root `LICENSE` file applies only to original repository-authored
automation and documentation to the extent those files are copyrightable. It
does not relicense the upstream archive or its bundled components. Review the
upstream credits and notices before redistributing the release assets publicly.

## Repository contents

```text
.github/workflows/mirror-tk4.yml   Scheduled upstream check and release publisher
README.md                          Project and operating documentation
LICENSE                            License for original repository-authored files
```

## Manual verification

After a release is published, verify the archive locally:

```bash
sha256sum -c tk4-_v1.00_current.zip.sha256
unzip -t tk4-_v1.00_current.zip
```

The checksum file contains the digest and the exact archive filename.
