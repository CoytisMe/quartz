---
category: reference
date: 2026-07-18
publish: true
tags:
  - server
  - windows
  - proxmox
  - tailscale
  - template
  - vm
---

# coytishost — Tailscale Auto-Enroll for Temp VMs

**Status: not yet implemented.** Plan for getting linked clones off template `win11template` (VM 104) onto Tailscale automatically at boot, without baking a static authkey into the template (max 90-day life) or pre-authing the template itself (clones would collide on the same node identity — don't do that).

Related: [[coytishost & Proxmox]], [[my disposable windows]]

---

## Why not just sign in on the template?

Tailscale's node identity lives in its state file (`%ProgramData%\Tailscale` on Windows). If the template has ever completed `tailscale up`, every clone inherits the *same* node key and they fight over one identity on the tailnet — constant flapping instead of distinct machines. The template must stay in the unauthenticated "please sign in" state forever; clones get authenticated individually, post-clone, via a freshly minted key.

---

## How it works

1. Proxmox `qemu-guest-agent` runs inside each clone.
2. A hookscript on the host fires on `post-start`, detects the VM is a linked clone of base-104, mints a fresh **single-use, ephemeral, pre-authorized** Tailscale authkey via the API, and pushes it into the guest with `qm guest exec` to run `tailscale up` silently — no popup, no manual click-through.
3. Because the key is ephemeral, the node auto-removes from the tailnet a short time after the VM is deleted. No dead entries to clean up.

---

## Setup checklist

- [ ] Install `qemu-guest-agent` on template 104, confirm the Windows service is set to auto-start
- [ ] Confirm template is still unauthenticated in Tailscale (never click through the sign-in popup on the template)
- [ ] Tailscale admin console → Settings → OAuth clients → new client, scope to **Auth Keys (write)** only (restrict to `tag:temp` if the option's available)
- [ ] Drop OAuth creds in a root-only file on coytishost: `/root/.tailscale-oauth.env` (`chmod 600`)
- [ ] `apt install jq` on coytishost if not already present
- [ ] Enable **Snippets** content type on the `local` storage (Datacenter → Storage → local → Edit → Content)
- [ ] Save the hookscript below to `/var/lib/vz/snippets/tailscale-temp-enroll.sh`, `chmod +x`
- [ ] `qm set 104 --hookscript local:snippets/tailscale-temp-enroll.sh` (config carries to every future clone automatically)
- [ ] Spin up one test clone via `new-client-vm.ps1`, confirm it shows up on the tailnet tagged `tag:temp` without any manual sign-in
- [ ] Add a `tag:temp` rule to the Tailscale ACL policy (e.g. `tag:personal → tag:temp`, temp otherwise unable to initiate — same pattern as the existing `tag:server` rule)

---

## Credentials file

```bash
# /root/.tailscale-oauth.env  (chmod 600, owned by root)
TS_OAUTH_CLIENT_ID="k123456CNTRL"
TS_OAUTH_CLIENT_SECRET="tskey-client-xxxxxxxxxxxx"
TAILNET="coytis.me"
```

## Hookscript — `/var/lib/vz/snippets/tailscale-temp-enroll.sh`

```bash
#!/bin/bash
# Fires on every VM start. Only acts on linked clones of base-104
# (win11template). Mints a fresh single-use ephemeral Tailscale
# authkey and activates Tailscale in the guest via the QEMU guest
# agent. Template must stay unauthenticated -- see note above.
set -euo pipefail

VMID="$1"
PHASE="$2"

[ "$PHASE" = "post-start" ] || exit 0

source /root/.tailscale-oauth.env

# Only fire for linked clones of template 104 -- same check used in
# delete-client-vm.ps1
if ! lvs --noheadings -o name,origin 2>/dev/null | grep -q "vm-${VMID}.*base-104"; then
  exit 0
fi

ACCESS_TOKEN=$(curl -fsS -X POST "https://api.tailscale.com/api/v2/oauth/token" \
  -d "client_id=${TS_OAUTH_CLIENT_ID}" \
  -d "client_secret=${TS_OAUTH_CLIENT_SECRET}" \
  | jq -r '.access_token')

AUTHKEY=$(curl -fsS -X POST \
  "https://api.tailscale.com/api/v2/tailnet/${TAILNET}/keys" \
  -H "Authorization: Bearer ${ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
        "capabilities": {
          "devices": {
            "create": {
              "reusable": false,
              "ephemeral": true,
              "preauthorized": true,
              "tags": ["tag:temp"]
            }
          }
        },
        "expirySeconds": 600
      }' \
  | jq -r '.key')

if [ -z "$AUTHKEY" ] || [ "$AUTHKEY" = "null" ]; then
  echo "tailscale-temp-enroll: failed to mint authkey for VM $VMID" >&2
  exit 1
fi

# Windows takes a while to boot -- wait for the guest agent
for i in $(seq 1 30); do
  qm agent "$VMID" ping >/dev/null 2>&1 && break
  sleep 2
done

HOSTNAME="temp-${VMID}-$(date +%s)"

qm guest exec "$VMID" -- \
  "C:\\Program Files\\Tailscale\\tailscale.exe" \
  up --authkey="${AUTHKEY}" --hostname="${HOSTNAME}" \
  --advertise-tags=tag:temp --accept-routes
```

---

## Notes

- The `hookscript` VM option is config-level, not disk-level — set once on template 104, every future clone inherits it, `--full` or linked doesn't matter.
- `post-start` fires the instant QEMU launches, long before Windows has booted — the `qm agent ping` poll loop handles that.
- OAuth client is scoped to Auth Keys only so a compromised coytishost can't do more than mint temp-tagged keys.
