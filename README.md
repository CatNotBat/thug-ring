# The Thug Ring — Mission Brief

> A fully decentralized, peer-to-peer UDP messaging ring.
> No central servers. No router port forwarding.

Before you touch any code, read **[guide.md](./guide.md)**. It covers every concept
you need to understand *why* this works (or doesn't).

---

## Mission Objectives

| # | Objective | Description |
|---|-----------|-------------|
| 1 | **Public Identity** | Programmatically discover the public IP and port your router exposes to the internet. |
| 2 | **The Breach** | Coordinate with your team to establish a direct, two-way UDP connection through your respective NAT firewalls. |
| 3 | **The Ring** | Run a continuous listening loop that processes incoming packets and routes them to the next node in the ring. |

---

## The Protocol (Strictly Enforced)

| Field | Value |
|-------|-------|
| Transport | UDP |
| Format | JSON, UTF-8 encoded |
| On parse failure | Drop silently — no exceptions, no logs |

### Packet Types

**Punch** — used only to open firewalls. Never forward.
```json
{"type": "punch", "sender": "<YourName>"}
```

**Data** — carries a message through the ring.
```json
{
  "type":   "data",
  "sender": "<Name>",
  "dest":   "<TargetName>",
  "ttl":    5,
  "msg":    "<Text>"
}
```

### Routing Logic

```
receive packet
  if type == "punch"  → log it, stop. never forward.
  if type == "data"
    if dest == ME     → print message, stop.
    else
      ttl -= 1
      if ttl > 0      → forward to your assigned Next Node
      if ttl == 0     → drop silently
```

### The Golden Rule

> **You must use the exact same UDP socket for STUN discovery, hole punching,
> and all routing. Do not close or recreate the socket.**

---

## Docker & Development Requirements

- Your entire application runs inside a **Docker container**.
- Use a **`requirements.txt`** for any external Python dependencies.
- Use **at least one environment variable** (node name, peer address, etc.).
- Successfully **map UDP traffic** from your host machine into the container.

Figure out the Dockerfile, the port mapping, and the environment variable injection
yourself — that's part of the exercise.

---

## Fallback Playbook — When the Primary Path Is Blocked

STUN + UDP hole punching is the **primary objective**. Attempt it first.

If you hit this wall, you may pivot. **Each fallback has security trade-offs you must
understand and document.** See [guide.md § Fallback Options](./guide.md#fallback-options).

### Decision Flowchart

```
Start
  │
  ▼
Try STUN hole punching
  │
  ├─ Works? ──────────────────────────────► Mission complete.
  │
  └─ Fails?
        │
        ├─ 1. Try --network host  (removes Docker NAT layer)
        │         │
        │         ├─ Works? ───────────► Done. Document what changed.
        │         │
        │         └─ Still fails?
        │
        └─ 2. Choose a relay strategy:
                  ├─ Have a VPS?  → SSH tunnel or manual relay
                  └─ No VPS?     → WireGuard / Tailscale overlay
```

Whatever path you take — document *why* STUN failed and *what* the security
implications of your workaround are.

---

---

## Side Quest — Pre-Commit & Linting

Your repo has a `.pre-commit-config.yaml` file. It's empty on purpose.

Your mission (optional but encouraged):

1. Install `pre-commit` and get it hooked into the repo.
2. Add a Python linter of your choice to the config and get it running on every commit.
3. Find one more hook that makes sense for this project and add it.

Start here: https://pre-commit.com

There's no right answer — poke around, break things, fix them.

---

*Good luck. Stay curious. Break things responsibly.*
