# Security Notes

**Scope:** the posture that keeps a convenience from
becoming an incident — plain statements, no FUD.

## SMB1 is dead; leave it buried

The original SMBv1 protocol is broken cryptography
with a famous worm (EternalBlue/WannaCry) in its
obituary. Modern Samba and modern Windows both default
it off — the lab's share listing even noted "SMB1
disabled." If an ancient device (a very old scanner,
a forgotten NAS) demands SMB1, the secure answers are
"replace it" or "isolate it," in that order. Re-enabling
SMB1 on a server to satisfy one device exposes every
share on it.

## Guests and wide-open shares

- **No guest shares on anything you care about.** The
  lab's `map to guest = never` + per-share `valid
  users` is the template. A guest share trusts every
  device on the network segment — including guests'
  laptops and compromised gadgets.
- **`hosts allow`** can fence Samba to your LAN ranges
  explicitly; the router's firewall should already,
  but defense in depth is cheap here.
- **Read-only by default**: flip `read only = no` per
  share, deliberately, not globally.

## NFS's trust is an IP address — price accordingly

NFS (in its classic AUTH_SYS form) believes the client
machine when it says "I am UID 1000." That's tolerable
exactly when you trust every machine that can reach
the export *and* everyone who can become root on them
(root-squash blunts only the root case). Practical
consequences:

- Export to the **narrowest client list** that works —
  specific hosts or your LAN subnet, never `*` on a
  network you don't fully control.
- Treat an NFS export as "anyone with LAN access and a
  laptop can present any UID" — because they can (a
  rogue machine can claim any address/UID it likes on
  a flat network). For media and documents among your
  own boxes: fine. For anything sensitive: not the tool,
  or not without a VPN layer.
- Samba's per-user passwords age better on mixed-trust
  networks, which is one more reason it fronts the
  Windows side of the world.

## The outer wall still matters

Both protocols assume a friendly-ish local network.
Neither belongs exposed to the internet: no port
forwards to 445 or 2049, ever. Remote access to your
files should ride a VPN or tailnet (this series'
Tailscale guide is one good road) and arrive *as if*
local — the share never learns the internet exists.
