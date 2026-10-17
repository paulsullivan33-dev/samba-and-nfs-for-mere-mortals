# Using NFS

**Scope:** the client side of NFS — seeing exports,
mounting them, keeping them in fstab — and the UID
rule that explains most NFS weirdness. (Documented
procedures; the lab's kernel boundary is stated in
the README.)

## See what's offered

```
showmount -e server
```

lists a server's exports and who may use them — the
`smbclient -L` of NFS. If this fails with an RPC error
while the server's `exportfs -v` looks right, the
problem is the service or the network path, not the
exports file: layers, in order.

## Mount it

```
sudo apt install nfs-common
sudo mount -t nfs server:/srv/nfsshare /mnt/nfs
```

No credentials — your client's IP was the credential.
The mount shows the server's directory with the
*server's* ownerships, and that fact drives the rule
below. fstab permanence is in
[`examples/fstab-lines.txt`](examples/fstab-lines.txt):
add `_netdev` (wait for networking at boot) and
consider `soft,timeo=30` so a sleeping server doesn't
freeze client programs solid — a hard mount of a dead
server can hang a machine in ways that feel like
possession.

## The UID rule (the chapter in one box)

**NFS trusts numeric user IDs, not names.** When you,
UID 1000 on the client, write to an export, the server
records owner 1000. If the server's UID 1000 is a
different person — or nobody — the file is theirs (or
no one's) as far as every tool can tell. Names in
`ls -l` are each machine's local translation of the
same numbers.

Mortals' countermeasures, in order of sanity:

1. **Keep UIDs aligned** across your own machines —
   create your account in the same order, or set the
   UID explicitly at user creation. A one-person fleet
   can simply be consistent.
2. **`all_squash,anonuid=1000`** on the export: force
   all access to arrive as one chosen server user.
   Loses per-person ownership; gains never thinking
   about it again. For a single-user share, a fair trade.
3. **NFSv4 idmapping** exists to translate properly in
   multi-machine setups; on a home fleet it's usually
   machinery beyond the problem.

And `root_squash` fits here: a client's root (UID 0)
is mapped to `nobody` precisely because letting a
remote machine's root be *your* root, on the strength
of an IP address, would be madness. NFSv4 is the
current default version family; the exports and
options in this guide apply across v3/v4 for mortal
purposes.
