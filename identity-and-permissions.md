# Identity and Permissions

**Scope:** the chapter both protocols have been
pointing at — what "who owns this file" means when
two machines are involved, and the three confusions
that generate most share problems.

## One file, two opinions

A file on a share has exactly one owner: a number. On
the server, that number maps to a name through the
server's user database; on an NFS client, the *same
number* maps through the client's. SMB adds its own
layer: you arrive as an authenticated Samba user, and
Samba performs file operations *as that user's Unix
identity on the server* — so the server's opinion is
the operative one, and clients see what Samba reports.

Three confusions, in descending frequency:

## 1. "It says the file belongs to someone else"

The NFS UID mismatch from the previous chapter: your
1000 is their 1002. Diagnose with numbers, not names —
`ls -ln` shows raw UIDs on any machine; compare across
machines and the mystery usually ends on the spot.

## 2. "I can read but not write, and permissions look fine"

Two gates (Samba chapter): the share's own `read
only`, `valid users`, or `write list` — then the
directory's Unix permissions, then the *parent* path's
execute permission all the way up. For NFS, the same
shape minus the login: export `ro` vs. directory mode.
The professional habit: test as the actual identity
(`sudo -u labuser touch /srv/labshare/x` on the server)
to separate "the server would allow it" from "the
protocol path is blocking it."

## 3. "New files have the wrong owner / group"

Created-over-a-share files inherit by rules worth
knowing once:

- Owner: the identity doing the writing (Samba: the
  logged-in user; NFS: your numeric UID, squashed if
  root).
- Group & mode: the client's umask and the directory's
  setgid bit play their usual Unix roles for NFS. Samba
  adds its own knobs — `create mask`, `directory mask`,
  and the blunt `force user` / `force group`, which pin
  every operation on a share to one identity. `force
  user` is inelegant and genuinely useful for a
  single-purpose share (a drop folder, a media tree):
  it deletes an entire category of permission bugs by
  making everyone the same person.

## The mortal's summary

Shares don't have permissions problems; they have
*identity translation* problems, and the translations
are few: Samba login → Unix user on the server; NFS
UID → the same number on the server, root squashed to
nobody. Write those two sentences on the inside of
your eyelids and most share mysteries become lookups.
