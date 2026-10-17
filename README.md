# Samba and NFS for Mere Mortals

Two protocols own the job of *sharing folders over a
network* — making files on one machine appear, live, on
another:

- **Samba** speaks **SMB**, the Windows file-sharing
  protocol, from Linux — so Windows, macOS, and Linux
  can all meet at the same folder.
- **NFS** is the Unix-native equivalent: Linux and Unix
  machines exporting directories to each other, simply
  and fast.

This guide sets both up for real, explains the
permission models that make them confusing, and says
plainly when a share is the wrong tool entirely.

It assumes you use a terminal and have edited config
files before. Networking basics (what an IP address
and a port are) are assumed; everything about shares,
exports, and mounts is explained from zero.

## What this guide covers

- [**What file sharing is**](what-file-sharing-is.md) —
  shares vs. copying, and the two protocol families
- [**A Samba server**](samba-server.md) — install,
  `smb.conf`, `testparm`, and the first working share
- [**Samba users and permissions**](samba-users-and-permissions.md) —
  the two-password-database truth, `valid users`,
  read-only shares, and guests
- [**Using Samba**](using-samba.md) — `smbclient` for
  real; mounting on Linux, Windows, and macOS
- [**An NFS server**](nfs-server.md) — `/etc/exports`,
  `exportfs`, and what the options really mean
- [**Using NFS**](using-nfs.md) — mounting, fstab, and
  the user-ID matching rule
- [**Identity and permissions**](identity-and-permissions.md) —
  who owns a file across a share, and why it goes wrong
- [**Choosing between them**](choosing-between-them.md) —
  Samba vs. NFS vs. the tools that aren't shares at all
- [**Security notes**](security-notes.md) — SMB1,
  guest shares, and NFS's trust model, plainly
- [**Troubleshooting**](troubleshooting.md) — symptom →
  fix, with the real `NT_STATUS` messages
- [**Glossary**](glossary.md) — the terms, collected in
  one place

## What it doesn't cover

Active Directory domains (Samba as a domain
controller is a book of its own), clustered
filesystems, and performance tuning beyond mortal
needs.

## How to use this guide

The Samba half was run for real (Samba 4.19): a share
was configured, validated with `testparm`, and used —
files moved both ways through `smbclient`, and the
classic failures (wrong password, denied guest,
read-only write) were produced and are quoted verbatim.
The NFS export side was likewise configured and
verified with `exportfs`. One boundary, stated plainly:
the test machine's kernel had no CIFS or NFS modules,
so actually *mounting* shares there failed with errors
that are themselves reproduced in Troubleshooting —
mount procedures are marked **(documented)**, as are
Windows/macOS client steps.
