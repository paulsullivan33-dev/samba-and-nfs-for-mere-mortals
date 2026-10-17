# What File Sharing Is

**Scope:** the idea underneath both protocols — and
the honest question of whether you need it at all.

## A share is not a copy

Copying a file to another machine gives that machine
*its own* file. Edits diverge; versions multiply;
someone always asks which one is current. A **network
share** keeps exactly one copy, on the machine that
*serves* it, and lets other machines open, read, and
write it in place, as if it were a local folder that
happens to live elsewhere.

The vocabulary differs by protocol but the shape is
the same:

- The server offers a **share** (Samba/SMB) or an
  **export** (NFS): a directory, plus rules about who
  may reach it and what they may do.
- The client **mounts** it (Linux/Unix) or **maps /
  connects** it (Windows drive letter, macOS Finder):
  from then on, programs on the client use ordinary
  paths, and the protocol carries the operations across
  the network invisibly.

That invisibility is the point and the danger: a
mounted share *feels* local, but its speed is the
network's, its availability is the server's, and its
permissions are a negotiation between two machines'
ideas of who you are — the subject of a later chapter.

## The two families

| | SMB (Samba) | NFS |
|---|---|---|
| Born | Windows file sharing | Unix, 1980s |
| Server on Linux | Samba (`smbd`) | kernel NFS + `nfs-server` |
| Trust based on | a username + password per user | the client's IP address + numeric user IDs |
| Natural clients | everything: Windows, macOS, Linux | Linux/Unix (others with add-ons) |
| Ports | 445 (SMB), 139 (legacy) | 2049 |

The trust row is the one to remember: it's why Samba
asks you to create passwords and NFS asks you to think
about addresses and UIDs. Nearly every confusing
behavior in this guide traces back to that row.

## When a share isn't the answer

Mortals now own alternatives their younger selves
didn't:

- **A web file manager** (a fileList-style app) serves
  browsing and moving files from any browser, with no
  mounts, no per-client setup, and no exposed protocol.
  For "I just want to get at my files," it has quietly
  replaced shares in many homes — including,
  reportedly, the household that inspired this series.
- **Copy tools** (`rsync`, `scp`) win when the job is
  *moving* files, not living in them: backups, sync,
  one-way publishing.
- **A share** still wins when several programs on
  several machines must work in the same live folder —
  a media library a player scans, a project tree an
  editor opens in place.

Choose per job. The rest of this guide assumes the
third case is yours.
