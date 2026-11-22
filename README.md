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

## TK4- operating quick reference

The following references are specifically useful for the MVS 3.8j TK4-
environment:

- **[TK4- User's Manual (PDF)](https://peppe8o.com/download/python/mainframe/mvs_tk4-_v1.00_users_manual.pdf)** — primary operating reference for TK4-, including TSO logon, system consoles, Hercules HTTP console input, RAKF security, user administration, and the supplied system procedures.
- **[MVS Console Cheat Sheet](https://github.com/FuzzyMainframes/MainframeNotebook/blob/master/MVS/MVS-Console-Cheat-Sheet.md)** — concise MVS 3.8J/TK4- console command reference.
- **[RAKF project and User's Guide](https://github.com/mvslovers/RAKF)** — RAKF security-system documentation. Check the RAKF version installed with TK4- before applying commands from newer RAKF documentation; command availability and syntax can differ.

### Hercules HTTP console and MVS commands

When Hercules is running in daemon mode, the Hercules HTTP server can be used
to enter MVS console commands. The console is also useful for observing
operator messages. Restrict access to the HTTP console (commonly TCP port
8038) to trusted management hosts; do not expose it publicly.

MVS console commands entered through the Hercules HTTP console are prefixed
with `/`. When entered through an alternate MVS console, omit the slash.

| Task | MVS console command | Notes |
|---|---|---|
| Cancel a TSO session | `/C U=userid` | Substitute the exact TSO user ID. This terminates that user's session; unsaved work is lost. |
| Display online DASD devices | `/D U,DASD,ONLINE` | Displays online DASD devices. |
| Display defined DASD devices | `/D U,DASD` | Displays defined devices and their status. |

Example: to release a disconnected or stuck HERC02 TSO session, enter
`/C U=HERC02` in the Hercules HTTP console and wait for the MVS
`CANCEL COMMAND ACCEPTED` response before attempting to log on again.

### TSO session basics

- Enter `LOGOFF` at the TSO `READY` prompt to end a TSO session normally.
- Exit RFE/RPF or other applications using their displayed exit key (often
  PF3) before issuing `LOGOFF`.
- A TSO user ID generally cannot have two concurrent sessions. If a terminal
  disconnects without a clean logoff, the old session may remain active.
  Use the MVS console cancel command above only when necessary.
- The TK4- TSO Applications menu includes RFE/RPF, the IMON-370 system
  monitor, the spool browser, help, utilities, and terminal testing. Use
  option 6 for TSO help and installed utility information.

### User accounts and RAKF

TK4- uses RAKF (the RAcK oFf security system), a RACF-like security system.
The stock distribution includes these initial TSO accounts:

| User ID | Initial password | Intended access |
|---|---|---|
| `HERC01` | `CUL8TR` | Fully authorized; access to RAKF users and profiles tables |
| `HERC02` | `CUL8TR` | Fully authorized; no access to RAKF users and profiles tables |
| `HERC03` | `PASS4U` | Regular user |
| `HERC04` | `PASS4U` | Regular user |
| `IBMUSER` | `IBMPASS` | Recovery account |

These are distribution defaults. Change default passwords and restrict
network access before using the system beyond a trusted lab environment.

**Creating a TSO user requires two steps:**

1. Define the user in the TK4- RAKF users table and activate the updated
   table at an MVS console with `S RAKFUSER`.
2. Define the user's TSO attributes in UADS. The TK4- manual recommends
   submitting the supplied `SYS2.JCLLIB(ADDUSER)` job after the RAKF
   definition is in place.

The RAKF user table and UADS are separate: a RAKF definition alone does not
automatically grant TSO logon capability. Follow the TK4- User's Manual's
**User Management** and **RAKF Administration** sections for the exact
procedure and dataset format on this distribution. Do not assume that
commands documented for newer RAKF releases (for example, `LISTUSER`,
`ADDUSER`, or `ALTUSER`) are available in the TK4- release installed on
your system. Likewise, inspect the actual catalog entries and dataset
organization before attempting to browse or edit a security dataset.

### Persistent mainframe data

For the ZWIEGNET Ansible deployment, persistent Hercules/TK4- files are
stored under `/data/hercules` and exposed to the service at
`/var/lib/hercules` through a bind mount. The emulated DASD files under
`/data/hercules/tk4/dasd/` contain the guest's persistent MVS volumes,
including system datasets, catalogs, security data, and user datasets.

Back up the DASD files with Hercules stopped to obtain a consistent
filesystem-level backup. Do not edit DASD image files directly to change
MVS users, passwords, or datasets; use the guest operating system's
supported utilities and procedures.

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
README.md                          Project, operating, and quick-reference documentation
LICENSE                            License for original repository-authored files
```

## Verify a downloaded release

```bash
sha256sum -c tk4-_v1.00_current.zip.sha256
unzip -t tk4-_v1.00_current.zip
```
