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

**Status: working, confirmed 2026-07-19.** Linked clones off template `win11template` (VM 104) now rename themselves to match the name given at clone time and auto-enroll on Tailscale at boot — no manual sign-in, no static authkey baked into the template (max 90-day life), no pre-authing the template itself (clones would collide on the same node identity — don't do that).

Related: [[coytishost & Proxmox]], [[my disposable windows]]

---

## Why not just sign in on the template?

Tailscale's node identity lives in its state file (`%ProgramData%\Tailscale` on Windows). If the template has ever completed `tailscale up`, every clone inherits the *same* node key and they fight over one identity on the tailnet — constant flapping instead of distinct machines. The template must stay in the unauthenticated "please sign in" state forever; clones get authenticated individually, post-clone, via a freshly minted key.

---

## How it works

1. Proxmox `qemu-guest-agent` runs inside each clone.
2. A hookscript on the host fires on `post-start`, detects the VM is a linked clone of base-104, and reads the VM's name (whatever was typed into `new-client-vm.ps1`).
3. It renames the Windows guest to that name and reboots it — before touching Tailscale, so the reboot can't interrupt an in-flight auth.
4. Once the guest agent comes back post-reboot, it mints a fresh **single-use, ephemeral, pre-authorized** Tailscale authkey via the API and pushes it into the guest to run `tailscale up` silently — no popup, no manual click-through.
5. Because the key is ephemeral, the node auto-removes from the tailnet a short time after the VM is deleted. No dead entries to clean up.

The old template-side `RenameOnClone` scheduled task (see [[coytishost & Proxmox]]) is now fully superseded by step 3 — it was already dead in practice since its trigger condition (`$env:COMPUTERNAME -eq "WIN11-TEMPLATE"`) never matched the template's actual name (`win11temp`). Worth deleting it from the template next time it's untemplated for other edits; not urgent since it's inert.

---

## Setup checklist

- [x] Install `qemu-guest-agent` on template 104, confirm the Windows service is set to auto-start ✅ 2026-07-19
- [x] Confirm template is still unauthenticated in Tailscale (never click through the sign-in popup on the template) ✅ 2026-07-19
- [x] Tailscale admin console → Settings → OAuth clients → new client, scope to **Auth Keys (write)** only (restrict to `tag:temp` if the option's available) ✅ 2026-07-19
- [x] Drop OAuth creds in a root-only file on coytishost: `/root/.tailscale-oauth.env` (`chmod 600`) ✅ 2026-07-19
- [x] `apt install jq` on coytishost if not already present ✅ 2026-07-19
- [x] Enable **Snippets** content type on the `local` storage (Datacenter → Storage → local → Edit → Content) ✅ 2026-07-19
- [x] Save the hookscript below to `/var/lib/vz/snippets/tailscale-temp-enroll.sh`, `chmod +x` ✅ 2026-07-19
- [x] `qm set 104 --hookscript local:snippets/tailscale-temp-enroll.sh` (config carries to every future clone automatically) ✅ 2026-07-19
- [x] Spin up one test clone via `new-client-vm.ps1`, confirm it shows up on the tailnet tagged `tag:temp` without any manual sign-in ✅ 2026-07-19 — clone `Coytispass` (VM 110) enrolled clean, `tailscale status` showed it connected, `hostname` confirmed the rename stuck
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
# (win11template). Renames the Windows guest to match the name given
# at clone time, then mints a fresh single-use ephemeral Tailscale
# authkey and activates Tailscale via the QEMU guest agent. Template
# must stay unauthenticated -- see note above.
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

wait_for_agent_up() {
  for i in $(seq 1 60); do
    qm agent "$VMID" ping >/dev/null 2>&1 && return 0
    sleep 2
  done
  return 1
}

wait_for_agent_down() {
  for i in $(seq 1 30); do
    qm agent "$VMID" ping >/dev/null 2>&1 || return 0
    sleep 2
  done
  return 1
}

# Base64-encodes the PowerShell payload (UTF-16LE, per -EncodedCommand's
# requirement) so nothing downstream (qm's argv quoting, cmd.exe's /c
# quote-stripping) can mangle nested quotes. cmd.exe /c ... < NUL forces
# stdin closed, since QGA on Windows leaves it open otherwise and some
# commands hang waiting on it.
run_ps() {
  local b64
  b64=$(printf '%s' "$1" | iconv -f UTF-8 -t UTF-16LE | base64 -w0)
  qm guest exec "$VMID" -- "C:\\Windows\\System32\\cmd.exe" /c \
    "powershell.exe -NoProfile -EncodedCommand $b64 < NUL"
}

wait_for_agent_up

NAME=$(qm config "$VMID" | awk -F': ' '/^name:/{print $2}')

run_ps "Rename-Computer -NewName '${NAME}' -Force -Restart"

# Wait for the rename-triggered reboot to actually happen before
# polling for it to come back -- qemu-guest-agent doesn't die the
# instant Rename-Computer -Restart fires, so polling "up" again
# immediately risks a false positive against the still-alive
# pre-reboot agent.
wait_for_agent_down
wait_for_agent_up

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

run_ps "& 'C:\\Program Files\\Tailscale\\tailscale.exe' up --authkey=${AUTHKEY} --hostname=${NAME} --accept-routes"
```

---

## Notes

- The `hookscript` VM option is config-level, not disk-level — set once on template 104, every future clone inherits it, `--full` or linked doesn't matter.
- `post-start` fires the instant QEMU launches, long before Windows has booted — `wait_for_agent_up` handles that.
- OAuth client is scoped to Auth Keys only so a compromised coytishost can't do more than mint temp-tagged keys.
- **Don't pass `--advertise-tags` to `tailscale up`.** It's redundant — the authkey already carries `"tags": ["tag:temp"]` in its capabilities, so the node is tagged automatically on redemption. Worse, passing it too triggered a tag-change confirmation prompt that the Windows CLI has no TTY to answer under `qm guest exec`, hanging the process indefinitely (`tailscale status` kept reporting `Logged out.` while the `up` process never exited). Diagnosed 2026-07-19 — dropped the flag.
- **`--operator` doesn't exist on Windows.** Tried adding `tailscale set --operator=user` so the logged-in "user" account could run tailscale CLI commands without hitting the SYSTEM-ownership 401 — it's Linux/macOS-only (tied to `sudo` splitting root vs. invoking user), Windows has no equivalent flag (confirmed via the `tailscale set` usage dump, exitcode 2). Dropped it 2026-07-19. Doesn't affect tailnet reachability, only local CLI convenience for the logged-in user.
- **Nested quotes across `qm guest exec` → cmd.exe → PowerShell get mangled.** Originally tried passing `cmd.exe /c "powershell.exe -Command \"...\""` as a manually-escaped string; the escaping got double-applied somewhere in that chain (visible as literal `\"` garbage in error output) and either silently no-op'd or threw "not recognized as an internal or external command." Fixed by switching to `-EncodedCommand` with a UTF-16LE-base64 payload (`run_ps` helper) — since the payload is a clean base64 blob, there's nothing left for any layer to mis-parse. Same trick `delete-client-vm.ps1` already uses for shipping its bash payload over SSH.
- **Reboot-detection race.** After `Rename-Computer -Restart`, a short fixed `sleep` before re-polling the guest agent isn't reliable — Windows doesn't tear the agent down instantly, so the first re-poll can succeed against the still-alive pre-reboot session, letting the script race ahead into a guest-exec call that then fails because the real reboot happens moments later. Fixed with `wait_for_agent_down` before `wait_for_agent_up` — explicitly confirm the transition happened instead of guessing a safe delay.
- To check status non-interactively from the host while debugging, query via guest-exec (same SYSTEM identity that owns the session, so no 401): `qm guest exec <vmid> -- "C:\Program Files\Tailscale\tailscale.exe" status`
- **Tailscale hostname now matches the Windows computer name (`$NAME`).** First working version used an auto-generated `temp-${VMID}-$(date +%s)` string instead, which defeated the actual goal (reaching temps by a name you recognize). Fixed 2026-07-19 — reuses `$NAME` from the rename step instead of a separate `HOSTNAME` variable. Tailscale auto-suffixes (`-1`, `-2`, ...) if a name collides with a still-connected device, so reusing names across quick test clones is safe.
