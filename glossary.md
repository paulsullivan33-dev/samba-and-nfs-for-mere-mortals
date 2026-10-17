# Glossary

**browseable** — whether a share appears when a client
lists a Samba server. Not an access control: hidden
shares are still reachable by name.

**CIFS** — the old name for the SMB dialect family;
survives in `mount -t cifs` and `cifs-utils`.

**Export** — a directory an NFS server offers, with a
client list and options, declared in `/etc/exports`.

**exportfs** — the command that loads `/etc/exports`
into the running server (`-ra`) and shows the
effective table (`-v`).

**Guest** — an unauthenticated share user. Convenient,
and trusted exactly as much as your least-trusted
network device.

**Mount** — attaching a share at a local directory so
programs use it as ordinary files.

**NT_STATUS_...** — SMB's error vocabulary
(`NT_STATUS_LOGON_FAILURE`,
`NT_STATUS_ACCESS_DENIED`). Ugly, greppable, honest.

**passdb** — Samba's own account database (`pdbedit`
inspects it), distinct from the Unix password file but
keyed to Unix accounts.

**root_squash** — NFS mapping a client's root user to
`nobody` on the server. The default; keep it.

**Share** — a directory Samba offers, named by its
`[section]` in `smb.conf`.

**smbclient** — the command-line SMB client: lists
shares, moves files, and serves as the first
diagnostic.

**smb.conf** — Samba's one config file; `[global]` plus
one section per share. `testparm` validates it.

**SMB1** — the ancient, insecure first protocol
version. Disabled by default everywhere modern; keep
it that way.

**testparm** — prints Samba's *effective* config and
warns about the parameters it ignored.

**UID** — the numeric user ID. NFS identity *is* UIDs;
names are local translations.

**valid users** — the Samba share option listing which
enrolled users may connect to that share.
