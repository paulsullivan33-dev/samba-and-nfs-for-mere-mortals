# An NFS Server

**Scope:** the Unix way — exporting a directory with
`/etc/exports` and `exportfs`. The export machinery
was configured and verified for real; the serving
daemon itself is (documented) for the kernel reason
given in the README.

## The whole config is one file of one-liners

Install: `sudo apt install nfs-kernel-server`. Then
[`examples/exports`](examples/exports):

```
/srv/nfsshare 127.0.0.1(rw,sync,no_subtree_check,root_squash)
```

Read it left to right: **this directory**, offered to
**this client** (an IP, a hostname, a network like
`192.168.1.0/24`, or `*`), with **these options**. No
users, no passwords — NFS's trust is the client's
address, plus the user IDs it presents (next chapter).

Apply and inspect:

```
sudo exportfs -ra          # re-export everything in /etc/exports
sudo exportfs -v           # the effective export table
```

```
/srv/nfsshare  127.0.0.1(sync,wdelay,hide,no_subtree_check,sec=sys,rw,root_squash,no_all_squash)
```

`exportfs -v` is NFS's testparm: it shows the options
*actually in force*, defaults included — `root_squash`
present, `wdelay` added, and so on. After any exports
edit, `-ra` then `-v` is the ritual.

## The options that matter

| Option | Meaning |
|---|---|
| `ro` / `rw` | read-only / read-write (default ro) |
| `sync` | reply only after data is truly written (safe; `async` is faster and risks loss on a crash — know which you're choosing) |
| `root_squash` | the client's root user is mapped to `nobody` on the server (the default, and a good one) |
| `no_root_squash` | client root = server root. Almost never what you want; see Security |
| `no_subtree_check` | skips a check that mostly matters when exporting a subdirectory of a filesystem; standard on modern setups |
| `all_squash` + `anonuid` | map *everyone* to one chosen user — the blunt but effective fix for UID mismatches |

## The daemon, honestly

On a normal Ubuntu server, `systemctl start
nfs-server` (with `rpcbind`) starts the kernel's NFS
service and your exports go live. In this guide's lab,
that step is where the sandbox said no: the test
kernel had no NFS modules at all (`nfs-server` stayed
inactive, and even `showmount` couldn't reach an RPC
service). The configuration half — file, exportfs,
effective table — is verified above; the daemon half
is the documented standard procedure. The distinction
matters less for NFS than most things: the mortal's
NFS work *is* the exports file and the client mount.
