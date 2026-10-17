# A Samba Server

**Scope:** installing Samba and standing up the first
share — validated, running, and listing itself. Every
step verified against Samba 4.19.

## Install

```
sudo apt install samba smbclient
```

That brings `smbd` (the server), and `smbclient` — the
command-line client that will be this chapter's test
instrument, the way `curl` is for web servers.

## The config: /etc/samba/smb.conf

One INI file. The guide's lab version ships as
[`examples/smb.conf`](examples/smb.conf):

```ini
[global]
   workgroup = WORKGROUP
   server string = Lab Samba Server
   security = user
   map to guest = never
   log file = /var/log/samba/log.%m
   max log size = 1000

[labshare]
   comment = Guide lab share
   path = /srv/labshare
   browseable = yes
   read only = no
   valid users = labuser
```

Anatomy:

- **`[global]`** — server-wide settings. `workgroup` is
  the Windows-era grouping name; on a home network it
  matters little. `security = user` means "people log
  in as themselves" (the sane mode). `map to guest =
  never` closes the "unknown user becomes a guest"
  side-door — the Security chapter explains why that's
  the right default.
- **Each other `[...]` section is a share**, named by
  its header. `path` is the real directory being
  offered; `browseable` lets it appear in listings;
  `read only` and `valid users` are access rules the
  next chapter exercises.

## testparm: the config's examiner

```
testparm -s
```

```
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Server role: ROLE_STANDALONE
...
```

`testparm` parses the file and prints the *effective*
configuration — defaults filled in — which is why it's
the debugging tool: it shows what Samba understood,
not what you hoped you wrote. And like systemd, Samba
**ignores misspelled parameters** after a warning:

```
Unknown parameter encountered: "broseable"
Ignoring unknown parameter "broseable"
```

A typo'd `browseable` doesn't stop the server; it just
silently isn't set. Run testparm after every edit.

## Start it, see it

```
sudo mkdir -p /srv/labshare
sudo systemctl start smbd        # or restart, after edits
smbclient -L localhost -U labuser
```

```
Sharename       Type      Comment
---------       ----      -------
labshare        Disk      Guide lab share
IPC$            IPC       IPC Service (Lab Samba Server)
```

`smbclient -L` asks a server "what do you offer?" —
the first smoke test of any share. `IPC$` is a
built-in administrative share, always present, not a
folder. The `labuser` in that command doesn't exist
yet — creating Samba users is its own ceremony, and
it's next.
