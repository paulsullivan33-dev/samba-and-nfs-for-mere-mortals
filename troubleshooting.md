# Troubleshooting

**Scope:** the failures you'll actually meet, symptom →
fix. Messages are verbatim from real runs (Samba 4.19)
unless marked (documented).

## "session setup failed: NT_STATUS_LOGON_FAILURE"

**Symptom:** `smbclient` (or a Windows/macOS connect)
rejects the login.

**Fix:** it's always one of three: the password is
wrong *for Samba's database* (reset it with
`sudo smbpasswd <user>` — remember, it's not the Unix
password), the user was never enrolled with
`smbpasswd -a`, or the username itself is off
(`pdbedit -L` lists who's real). The server won't say
which — by design.

## "tree connect failed: NT_STATUS_ACCESS_DENIED"

**Symptom:** login accepted far enough to list, but
connecting to the share is refused.

**Fix:** the share's gate: you're not in its `valid
users`, the share doesn't exist under that name
(`smbclient -L` shows the true names), or it's a guest
attempt against a no-guest server. Check the share's
section in `smb.conf` and that `testparm -s` agrees
with your intent.

## "NT_STATUS_ACCESS_DENIED opening remote file ..."

**Symptom:** you can browse the share but writes (or a
specific open) fail.

**Fix:** `read only = yes` on the share, or gate 2 —
the Unix permissions of the directory/file for your
Unix identity. Test on the server directly
(`sudo -u <user> touch <path>/probe`) to find which
gate is shut. New-file ownership surprises live in the
Identity chapter.

## testparm: "Unknown parameter encountered"

```
Unknown parameter encountered: "broseable"
Ignoring unknown parameter "broseable"
```

**Symptom:** a setting silently does nothing.

**Fix:** misspelled parameters are ignored, not fatal —
in Samba exactly as in systemd. The share then runs
with the *default* (browseable happens to default to
yes; a typo'd `read only` could default the dangerous
way). `testparm -s` after every edit, and read its
warnings as errors.

## "mount error: cifs filesystem not supported by the system" / "mount.nfs: No such device"

```
mount error: cifs filesystem not supported by the system
mount error(19): No such device
```

**Symptom:** mounting fails immediately on a minimal
or containerized kernel.

**Fix:** the client kernel lacks the filesystem module
(the guide's lab hit exactly this for both protocols).
Install `cifs-utils`/`nfs-common` for the helpers —
but the module is the kernel's; on a normal desktop or
server kernel both are present. Workaround while
unmounted: `smbclient` still moves files fine, since
it speaks SMB in user space.

## showmount: "RPC: Unable to send" (documented family)

**Symptom:** `showmount -e server` can't reach the
server at all, while the exports file looks right.

**Fix:** the NFS services aren't running or aren't
reachable: `systemctl status nfs-server rpcbind` on
the server, then the firewall path between. Distinguish
layers: `exportfs -v` (config) → services (daemon) →
network (ports/RPC) → the mount itself. (The lab's
instance of this error was the no-kernel-modules case
above.)

## "It works from one machine but not another"

**Fix:** think in identities and addresses. NFS:
client IP outside the export's list, or UID mismatch
(works = can mount, wrong owner = can't write).
Samba: different saved credentials on the failing
client (Windows is famous for silently retrying a
stale saved login) — clear the stored credential and
connect fresh before blaming the server.

## Everything is slow

**Fix (documented):** shares inherit the network's
speed and latency; first checks are boring — Wi-Fi vs.
wired, a server disk that's the real bottleneck, and
NFS `async` temptation (faster, risks data on a server
crash; a decision, not a fix). Small-file workloads
suffer most over any protocol; a folder of 10,000 tiny
files is rsync's natural prey instead.
