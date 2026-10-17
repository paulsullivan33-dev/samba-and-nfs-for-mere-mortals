# Choosing Between Them

**Scope:** the decision table — because half of using
these tools well is knowing when the answer is a
different tool entirely.

## Head to head

| Situation | Reach for |
|---|---|
| Windows or macOS must play | **Samba** — it's their native tongue |
| All-Linux fleet, same owner, speed and simplicity | **NFS** — no password DB, kernel-fast |
| One live folder, many programs, several machines | a share (either, by the rows above) |
| "Just let me browse/move files from my phone/browser" | a **web file manager**, not a share |
| Backups, sync, publishing a tree one-way | **rsync** — a share is the wrong shape entirely |
| Occasional file grab from a terminal | **scp/sftp**, or `smbclient` for an existing share |
| Untrusted or semi-trusted network | neither, exposed raw — put it behind a VPN/tailnet (see Security) |

## The fileList lesson, generalized

This guide exists in a series whose author's own
household is exhibit A: a good web file manager
covered so much of the everyday "get at my files"
need that Samba — once the backbone — became
occasional. That's not a verdict against shares; it's
the correct division of labor emerging. Shares serve
*programs* (mounts that look like disks); web managers
serve *people* (browsers that need no setup). When a
share feels like overkill lately, it usually is —
for that job.

## Mixing them

One server can run both: the same directory exported
by NFS to the Linux boxes and shared by Samba to the
Windows laptop is a legitimate, common layout. Two
cautions if you do: file locking semantics differ
between the protocols (two writers via different
protocols can surprise each other — fine for media and
documents, think twice for databases), and permissions
get debugged in two dialects at once. For a database
file of any kind — SQLite included — the rule from
that guide applies with force: don't let multiple
machines at it over shares at all.
