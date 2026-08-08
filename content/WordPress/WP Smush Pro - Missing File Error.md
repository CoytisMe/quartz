---
category: work-note
date: 2026-06-22
publish: true
tags:
  - wordpress
  - hosting
  - plugins
  - wpmu-dev
---

# WP Smush Pro — Missing File Error

## Error

Client's WordPress site threw a fatal `E_ERROR` on load:

```
Uncaught Error: Failed opening required '.../wp-smush-pro/core/backups/class-backups-controller.php'
```

Missing file prevented the plugin from loading. Site was completely down — error fires before WP Admin loads (`wp-login.php`).

## Cause

Corrupt or incomplete plugin installation — most likely a failed/partial update or interrupted file upload.

## Fix Options

1. **Re-upload plugin** — Download a fresh copy from [WPMU Dev dashboard](https://wpmudev.com/hub/plugins/), re-upload via FTP/cPanel, overwriting existing files. Cleanest fix.
2. **Update via WP Admin** — Only works if WP Admin is still accessible: Plugins → Smush Pro → Update.
3. **Rename plugin folder via FTP/SSH** — Rename `wp-smush-pro` → `wp-smush-pro-disabled`. WordPress auto-deactivates it, restoring WP Admin for a clean reinstall.
4. **Deactivate via database** — phpMyAdmin → `wp_options` → `active_plugins` → remove Smush Pro from the serialized array. Or via WP-CLI: `wp plugin deactivate wp-smush-pro`.

## Resolution

Disabled via WHM to restore the site. Checking with boss on where the licence/download comes from.

## Followup

- Confirm whether client's WPMU Dev membership is active — if lapsed, we can't pull a fresh Smush Pro download
- If membership is lapsed: replace with free [Smush](https://wordpress.org/plugins/wp-smushit/) from the WordPress plugin repo — covers basic image compression
- If membership is active: reinstall clean copy from [WPMU Dev Hub](https://wpmudev.com/hub/plugins/) and verify no other WPMU Dev plugins have corrupt installs on the same site
- Check if the partial update was triggered by an auto-update — consider disabling auto-updates for WPMU Dev plugins if this is a recurring risk

## Notes

- WP Smush Pro is bundled under a **WPMU Dev membership** — it's not available on the WordPress plugin repo
- If the membership has lapsed, the free [Smush plugin](https://wordpress.org/plugins/wp-smushit/) is a reasonable drop-in for basic compression (loses bulk smush scheduling and CDN features)
