---
category: reference
date: 2026-08-09
publish: true
tags:
  - terminal
  - ssh
  - tmux
  - claude
---

For when I'm SSH'd into a box from the laptop and don't want to lose everything because I closed a window like an idiot.

## The problem

SSH into something, start a long job (or a Claude session), close the terminal — it all dies. The process is chained to the terminal that started it. Shut the lid, same thing.

tmux fixes the "I closed the terminal" half of that. It does **not** fix the "my wifi dropped" half. Those are two different problems and it took me a minute to get that straight, so it's written down properly further below.

## What tmux actually is

It runs your shell inside a session that lives independently of the terminal window. You detach from it, the session keeps running, you reattach later and everything's exactly where you left it.

Install it:
```bash
sudo dnf install tmux     # Fedora
sudo apt install tmux     # Ubuntu
```

## The basic loop

Start a named session:
```bash
tmux new -s claude
```

Then do whatever inside it — SSH somewhere, run a job, whatever:
```bash
ssh Rick@rickstreampc.taild9faf8.ts.net
```

Detach with `Ctrl-b` then `d`. Terminal closes, session keeps running.

Come back:
```bash
tmux ls                   # what sessions exist
tmux attach -t claude     # get back in (tmux a -t claude also works)
```

Nuke one:
```bash
tmux kill-session -t claude
```

## Keys worth knowing

`Ctrl-b` is the **prefix**. You tap it, let go, *then* hit the next key. It's not a hold-both-down thing. This trips everyone up at the start.

| Keys | Does |
|---|---|
| `Ctrl-b` `d` | Detach, session keeps running |
| `Ctrl-b` `c` | New window |
| `Ctrl-b` `n` / `p` | Next / previous window |
| `Ctrl-b` `%` | Split vertical |
| `Ctrl-b` `"` | Split horizontal |
| `Ctrl-b` `arrow` | Move between panes |
| `Ctrl-b` `Ctrl-b` | Send the prefix to a *nested* tmux |

That last one matters if you ever run tmux inside tmux (laptop session, then another one on the remote box). The prefix only reaches the outer one by default, so you double-tap to punch through to the inner.

## tmux vs Tilix

Tilix already does tiling, so the splits above are mostly redundant for me.

- **Tilix tiles locally.** Panes live on the laptop. Close Tilix, they're gone.
- **tmux persists.** Sessions live on whatever box tmux is running on and survive detaching.

So: Tilix for the local window layout, tmux for the not-losing-things. No need to nest them just for splits — two layers of keybindings gets annoying fast.

## The bit I keep having to re-think

Running tmux **on the laptop**, SSH'd into somewhere else. Two different failure modes, two different fixes:

**Terminal closed / lid shut, laptop still awake and online**
The SSH connection inside tmux is fine. `tmux attach` and the remote process is *literally still running*, same process, mid-whatever. Nothing else needed.

**Wifi actually dropped, or laptop slept/rebooted**
TCP connection dies, and the remote process dies with it. Laptop-side tmux can't save you from this — it's holding a connection that no longer exists. This is what `claude --continue` is for.

Short version: tmux protects the laptop end. Network death is a separate problem.

(`mosh` survives roaming and would cover the second case, but there's no mosh-server for Windows so it's no help for the streampc.)

## Getting a Claude session back

Not crash recovery — it reads saved transcripts off disk. Clean exit, dropped connection, reboot a week later, all the same.

```bash
ssh Rick@
cd vmhost
claude --continue    # grabs the most recent session in this folder
claude --resume      # picker, choose from older ones
```

`--continue` is the one I'll want basically always. Short flags are `-c` and `-r`.

Two things that'll catch me out:

- **`cd` into the right folder first.** Sessions are scoped per directory. Running `--continue` from home won't find the vmhost one.
- **It restores the conversation, not the process.** Full history comes back, but anything live is gone — background commands it had running, shell state, that sort of thing. Rarely matters in practice.

## Related

- [[Linux commands]]
- [[coytishost & Proxmox]]
