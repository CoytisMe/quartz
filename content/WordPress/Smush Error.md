---
category: work-note
date: 2026-06-24
publish: false
tags:
  - wordpress
  - hosting
  - plugins
  - wpmu-dev
  - arceyeengineering
---

# Smush Error — arceyeengineerin

Site: `arceyeengineerin` (cPanel/WHM)

## What Happened

Client's WordPress site went down with a fatal PHP error. Disabled Smush Pro via WHM to restore the site. See [[WP Smush Pro - Missing File Error]] for the full diagnosis and fix options.

## Original Error

```
E_ERROR in /home/arceyeengineerin/public_html/wp-content/plugins/wp-smush-pro/wp-smush.php:359
Uncaught Error: Failed opening required '...wp-smush-pro/core/backups/class-backups-controller.php'
```

## Resolution

Smush Pro disabled via WHM → site restored. Checked in with Paddy (boss).

## Paddy's Response

> Yep disabling Smush was the right play. In the case of Smush you can leave it disabled — I actually just deleted the plugin from the disabled list. Smush is more for compressing and optimising large images at time of content creation so it doesn't really need to be there.
>
> Re licensing — Defender Pro, Hummingbird Cache, Smush are all from WPMU Dev and I have a business account. All good and thanks for tackling it.

## Outcome

- Smush Pro deleted (not just deactivated) — not needed on this site
- WPMU Dev business account confirmed active (Paddy holds it)
- No reinstall required
