---
category: how-to
date: 2026-07-26
publish: true
tags:
  - ssh
  - linux
  - remote-access
  - security
---
## Got a Linux box set up with SSH key auth from one PC, want to SSH in from others too?

If you set up passwordless SSH the normal way, you generated a keypair on one PC and dropped the public half into `~/.ssh/authorized_keys` on the remote machine. That means only that one PC can get in. Don't fix this by copying the private key onto every other device — private keys shouldn't travel. Instead, give each device its own identity.

### 1. Generate a keypair on each additional PC

```bash
ssh-keygen -t ed25519 -C "some-name-for-this-device"
```

Leave the passphrase blank if you want fully passwordless, or set one if the device isn't fully trusted (laptop that leaves the house, etc).

### 2. Add each new public key to the remote machine's authorized_keys

Get the Key
```Powershell
Get-Content C:\Users\Rick\.ssh\id_ed25519.pub
```

Easiest way, from a PC that already has access:

```bash
ssh user@remote-host "echo '<paste-new-pubkey-here>' >> ~/.ssh/authorized_keys"
```

Or from the new PC itself, if password auth is still enabled on the remote host:

```bash
ssh-copy-id user@remote-host
```

Each key gets its own line in `authorized_keys` — they stack, they don't overwrite each other, and you can remove one device's access later by deleting its line without touching the rest.

### 3. Reachability

This only gets you SSH access from devices **on the same LAN/subnet** as the remote box. If you want access from off-network (laptop out and about, phone, etc), key auth alone doesn't solve that — you also need one of:

- **Tailscale / WireGuard** on the remote box — simplest, no port forwarding, devices join a private mesh network
- **Port forward + reverse proxy or bastion** — more exposure, only do this if you know what you're locking down

### Notes

- If you disable password auth entirely (`PasswordAuthentication no` in `/etc/ssh/sshd_config`), `ssh-copy-id` needs an existing authorized key or physical/console access to seed the first one.
- Rotate or remove a device's key any time by deleting its line from `authorized_keys` — no need to touch anyone else's.
