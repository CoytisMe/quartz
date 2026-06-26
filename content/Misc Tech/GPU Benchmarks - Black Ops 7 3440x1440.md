---
category: research
date: 2026-06-23
publish: true
tags:
  - gaming
  - gpu
  - benchmarks
  - call-of-duty
---

# GPU Benchmarks — Call of Duty: Black Ops 7

> Researched 2026-06-23. Native 3440x1440 ultrawide benchmarks are not widely published — major outlets (Tweaktown, GameGPU, WCCFTech) tested at standard 1440p and 4K only. Ultrawide estimates extrapolated from those results.

---

## GPU Performance

### Native Resolution Benchmarks (Extreme preset)

| GPU | 1440p | 4K | Est. 3440x1440 |
|---|---|---|---|
| RTX 5080 | ~185 fps | ~100 fps | ~150–160 fps |
| RTX 5070 Ti | ~177 fps | ~88 fps | ~135–150 fps |
| RTX 5070 | ~99 fps | ~60 fps | ~80–95 fps |

> 3440x1440 sits ~10–15% heavier than standard 1440p. Estimates only — check [GameGPU](https://en.gamegpu.com/test-gpu/action-fps-tps/call-of-duty-black-ops-7-test-gpu-cpu) for interactive resolution selector.

### With DLSS (1440p tested)

| GPU | DLSS On | Frame Gen x2 |
|---|---|---|
| RTX 5080 | ~199 fps | ~264 fps |
| RTX 5070 Ti | ~148 fps | ~213 fps |
| RTX 5070 | ~117 fps | ~195 fps |

### Notable: AMD Advantage in BO7
The **RX 9070 XT significantly outperforms the RTX 5070 Ti** in Black Ops 7 (AMD-favoured title) — ~31% faster at 1440p. Worth factoring in if still shopping.

---

## RAM: 16GB vs 32GB

32GB does provide a benefit, primarily in frame consistency rather than raw averages:

| Resolution | 16GB avg | 32GB avg | Difference |
|---|---|---|---|
| 1080p | ~103 fps | ~101 fps | Negligible |
| 1440p | ~166 fps | ~177 fps | ~+11 fps (~7%) |
| 4K | ~103 fps | ~101 fps | Negligible (GPU bottleneck) |

**Verdict:** 32GB helps most at 1440p where the CPU/RAM has headroom. More importantly, 32GB improves **1% lows and frame consistency** across all resolutions — less stuttering during busy scenes. For a high-refresh ultrawide setup, 32GB is recommended.

---

## CPU Comparison: 9800X3D vs 9850X3D vs Core Ultra 9 285K

### Summary

| CPU | Gaming Tier | Notes |
|---|---|---|
| Ryzen 7 9800X3D | S-tier | Best gaming CPU in its generation |
| Ryzen 7 9850X3D | S-tier | ~3–4% faster than 9800X3D on average |
| Core Ultra 9 285K | A-tier | ~19% slower than 9800X3D in CoD titles |

### 9800X3D vs 9850X3D
- Average gaming difference: **3–4%** (Tom's Hardware, 16-game test: 204.6 vs 211.2 fps at 1080p)
- Some titles within margin of error (Cyberpunk, Starfield, DOOM: The Dark Ages)
- Largest gap seen: ~6% (Flight Simulator 24)
- 9850X3D has higher boost clock: 5.6 GHz vs 5.2 GHz
- **Verdict:** Marginal upgrade — not worth it if already on a 9800X3D. Fine to buy new if price difference is small.

### 9800X3D vs Core Ultra 9 285K
- 9800X3D is **~19% faster** in CoD Black Ops 6 (closest available data to BO7)
- 285K trails significantly in AMD-optimised titles
- 285K competitive in productivity/multi-threaded workloads
- **Verdict:** For pure gaming, especially CoD, the 9800X3D/9850X3D wins clearly.

### 9850X3D vs Core Ultra 9 285K
- 9850X3D up to **60% faster** than 285K in some gaming titles (Tweaktown)
- Gap is title-dependent but consistently favours AMD X3D in shooters
- **Verdict:** No contest for gaming. 285K only makes sense if heavy productivity work is the priority.

---

## Sources
- [Tweaktown — BO7 4K & 1440p Benchmarks](https://www.tweaktown.com/articles/11202/call-of-duty-black-ops-7-early-access-multiplayer-4k-and-1440p-benchmarks/index.html)
- [Tweaktown — RTX 5080 needs DLSS at 4K](https://www.tweaktown.com/news/108058/geforce-rtx-5080-needs-dlss-to-hit-144fps-at-4k-in-call-of-duty-black-ops-7/index.html)
- [Tweaktown — 9850X3D up to 60% faster than 285K](https://www.tweaktown.com/news/109521/amds-new-ryzen-7-9850x3d-is-up-to-60-percent-faster-in-gaming-than-the-intel-core-ultra-9-285k/index.html)
- [WCCFTech — RX 9070 XT vs RTX 5070 Ti in BO7](https://wccftech.com/radeon-rx-9070-xt-demolishes-rtx-5070-ti-in-call-of-duty-black-ops-7/)
- [GamersNexus — Ryzen 9 9950X3D Review](https://gamersnexus.net/cpus/amd-ryzen-9-9950x3d-cpu-review-benchmarks-vs-9800x3d-285k-9950x-more)
- [GamersNexus — 9850X3D Review](https://gamersnexus.net/cpus/amd-ryzen-7-9850x3d-cpu-review-benchmarks-gaming-power-thermals-ft-ddr5-4800)
- [Tom's Hardware — 9850X3D vs 9800X3D](https://www.tomshardware.com/pc-components/cpus/amd-ryzen-7-9850x3d-vs-ryzen-7-9800x3d)
- [GameGPU — BO7 GPU Benchmarks](https://en.gamegpu.com/test-gpu/action-fps-tps/call-of-duty-black-ops-7-test-gpu-cpu)
