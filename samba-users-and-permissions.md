# Samba Users and Permissions

**Scope:** the identity machinery — why a Unix account
isn't enough, how share rules stack with filesystem
rules, and the guest question.

## Two databases, one person

The fact that surprises everyone, learned the direct
way in this guide's lab:

```
smbpasswd -a hatch
```

```
Failed to add entry for user hatch.
```

There was no Unix user named `hatch` on the machine.
**Samba keeps its own password database, but every
Samba account must first be a Unix account.** The Unix
side owns the identity (and the file ownership); the
Samba side stores the *SMB* password hash, which is
different cryptography from the Unix login password —
so Samba can't reuse `/etc/shadow` and needs its own
enrollment:

```
sudo useradd -m labuser                 # if needed
sudo smbpasswd -a labuser               # prompts for the SMB password
sudo pdbedit -L                         # list who's enrolled
```

```
labuser:1000:
```

That number is the Unix UID — keep it in mind for the
Identity chapter. Changing the SMB password later is
the same `smbpasswd` command; the user's Unix password
is a separate thing that Samba never touches. Two
databases, one person, endless confusion when you
forget which one a login prompt is asking about.

## The two gates

Every access passes **two** independent checks:

1. **Samba's rules** — is this share open to this user
   (`valid users`), is it `read only`, is the login
   itself valid?
2. **The filesystem's rules** — the shared directory's
   ordinary Unix permissions, applied as that Unix
   user. Samba cannot grant what Unix denies: a share
   pointing at a directory the user can't read serves
   errors, not files.

When a share "should work but doesn't," one of the two
gates is closed, and the fixes live in different files
(`smb.conf` vs. `chmod`/`chown`). The lab's share
needed `chown labuser /srv/labshare` before uploads
landed — gate 2, as usual.

And `read only = yes` is gate 1 at its clearest —
flipping it in the lab turned uploads into:

```
NT_STATUS_ACCESS_DENIED opening remote file \blocked.txt
```

## Guests: the convenient hazard

A share with `guest ok = yes` (plus server settings
that allow it) lets anyone who can reach the port in,
no password at all. This guide's lab ran the opposite
posture — `map to guest = never`, no guest shares —
and an anonymous attempt earned:

```
tree connect failed: NT_STATUS_ACCESS_DENIED
```

That's the posture to keep: guest access on a network
share means "any device that joins this network —
including the compromised smart bulb — can read and
possibly write these files." If a share is worth
having, it's worth one `smbpasswd`.
