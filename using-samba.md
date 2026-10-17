# Using Samba

**Scope:** the client side — moving files with
`smbclient` (verified), and mounting a share so
programs see a folder (documented patterns).

## smbclient: the FTP-shaped client

```
smbclient //localhost/labshare -U labuser
```

Inside, it speaks `get`, `put`, `ls`, `cd`, `mget`,
`mput` — an FTP dialect. The lab's round trip:

```
smb: \> get readme.txt
getting file \readme.txt of size 27 as readme.txt
smb: \> put upload.txt
putting file upload.txt as \upload.txt
smb: \> ls
  readme.txt     N    27
  upload.txt     A    15
```

The uploaded file landed in `/srv/labshare` owned by
labuser — the Unix identity behind the login, exactly
as the two-gates model predicts. Non-interactively,
`-c 'put file'` runs one command and exits, which is
how scripts use it. And the authentication failure
you'll meet most, verbatim:

```
session setup failed: NT_STATUS_LOGON_FAILURE
```

Wrong username, wrong password, or a user who exists
in Unix but was never enrolled with `smbpasswd` — the
server deliberately doesn't say which.

## Mounting on Linux (documented)

```
sudo apt install cifs-utils
sudo mount -t cifs //server/labshare /mnt/labshare \
    -o username=labuser,iocharset=utf8
```

It prompts for the password. Mounted, the share is a
folder — editors, players, and backup tools use it
without knowing SMB exists. Permanent mounts go in
`/etc/fstab`
([`examples/fstab-lines.txt`](examples/fstab-lines.txt))
with the password in a root-only `credentials=` file,
never inline. Two honest footnotes: mounted SMB files
carry the *mounting* user's Unix identity unless you
set `uid=`/`gid=` options (the Identity chapter's
theme), and a hung server can hang programs touching
the mount — the mortal reason web managers won some
territory back.

(Documented: this guide's test kernel lacked the CIFS
module — the mount attempt's error is in
Troubleshooting — so mount steps follow the standard
documented procedure rather than a lab run.)

## Windows and macOS (documented)

- **Windows**: File Explorer's address bar takes
  `\\SERVER\labshare`; it prompts for the Samba
  credentials and offers to remember them, and "Map
  network drive" makes it a permanent letter. Modern
  Windows refuses ancient SMB1 and *guest* SMB by
  default — both refusals are features, and both
  produce errors that send people googling; the fix is
  a real Samba user, never re-enabling SMB1.
- **macOS**: Finder → Go → Connect to Server →
  `smb://server/labshare`, same credentials.
