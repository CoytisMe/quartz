---
category: tech
date: 2026-10-17
publish: true
tags:
  - kbdraft
  - publickb
  - unifi
  - wireguard
  - vpn
  - networking
---
# UniFi WireGuard VPN Setup Gotchas

## I got this working for my network in 15 mins, the client took forever

## Name resolution: skip DNS, use the hosts file
- For a site with a **single server**, the client config works without the extra DNS/IP settings.
- Add the server's name to the client's `hosts` file so it resolves over the tunnel.
- It isn't a dynamic solution, but with one server it's simpler than pushing DNS.
- When scripting the hosts entry with `echo`, quote the line so both words stay on the same line.

## Clients vanish / won't connect: save first
Symptom: clients losing connection seemingly at random. The first test client worked and the next didn't. Clients were **disappearing from the UniFi control panel** and their connections weren't showing in the logs.

Cause: **if you create a client in UniFi and connect with that config in WireGuard before hitting "Save changes" on the VPN settings, it breaks the client.**

Fix: create the clients, **Save changes**, then export and use the configs.

## Don't put the VPN subnet on 192.168.1.x
- 192.168.1.0/24 is what most home routers use, so it clashes with users' home networks.
- Pick something unusual (e.g. `192.168.192.x`).

## Rebuild plan that worked
1. Create a new VPN server on the new subnet.
2. Recreate every client, saving before you export.
3. Test hard: each config on more than one PC **and** more than one network (home Wi-Fi, phone hotspot). Check both internet and server access.
4. Store the configs somewhere the office can get them back (e.g. the server plus a cloud drive).
5. Remove the old server once everyone's moved.

**Testing tip:** don't test the VPN over RDP from inside the network. Bringing the tunnel up drops the RDP session. Test from a machine you're physically at, or over a remote tool that doesn't route through the tunnel.
