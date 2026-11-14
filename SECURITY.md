# Security policy

## Supported versions

| Version | Supported |
|---|---|
| 2026.11.13-v2 | Yes |
| 2026.11.13 | No, superseded |

## Reporting a vulnerability

Do not open a public issue for a security problem. Report it privately through
GitHub's **Report a vulnerability** button on the Security tab of this
repository, or contact the owner directly through their GitHub profile.

Please include:

- the affected ISO version and its SHA-256
- the hardware or VM you booted it on
- exact reproduction steps
- what you expected and what happened

You will get an acknowledgement, then a fix timeline once the report is
confirmed.

## Scope

In scope:

- the boot chain (GRUB, EFI binaries, squashfs integrity)
- the live session defaults (user, password, permissions, network services)
- the Zaod branding layer scripts under `/usr/lib/zaod/`
- anything that lets a local user escalate privileges or exfiltrate data off a
  live session

Out of scope:

- vulnerabilities in upstream Debian, XFCE, Firefox or other bundled packages
  that are unfixed upstream. Report those upstream, then forward the advisory.
- Secure Boot refusing an unsigned image. That is documented and expected.
- The live password `live` being publicly known. Live images are not meant to
  be secure multi-user systems; do not use one to store secrets.
- Physical attacks on an unlocked live machine.

## Verifying a download

Every release publishes its SHA-256 in the release notes and in
[docs/CHANGELOG.md](docs/CHANGELOG.md). Verify the hash before booting. A
mismatch means the file is corrupt or has been tampered with. Do not boot it.

## Legal notice

Zaod OS is proprietary. Security research on a copy you obtained legally is
welcome, but redistribution of the image, its sources or its branding is not
permitted. See [LICENSE](LICENSE).
