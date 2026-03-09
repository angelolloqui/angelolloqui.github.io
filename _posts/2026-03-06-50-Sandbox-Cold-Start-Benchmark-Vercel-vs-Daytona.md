---
layout: post
title:  "Sandbox Cold-Start Benchmark: Vercel vs Daytona"
date:   2026-03-06 10:00:00
categories:
    - ai
    - engineering
    - agents
permalink: /blog/:title
---

I've been building an AI agent that needs to run arbitrary code — execute scripts, call tools, inspect outputs. For this I need an **execution sandbox**: an isolated Linux environment I can spin up on demand, run a command in, and tear down. The faster it starts, the more responsive the agent feels.

I evaluated two providers: **Vercel Sandbox** and **Daytona**. A third candidate, **Cloudflare Sandbox**, was on the list but requires the Workers Paid plan to use containers — I skipped it for now and may revisit.

The full benchmark code is available on [GitHub](https://github.com/angelolloqui/sandbox-benchmark).

---

## What I measured

Each sandbox is pre-configured with [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed via a snapshot (Vercel) or a custom image (Daytona). On every run I measure from a cold start:

- **Startup** — time for the SDK `create()` call to return a ready sandbox
- **Exec** — time to run `claude --version` inside it
- **Time-to-first-command (TTFC)** — startup + exec, the end-to-end latency an agent would actually experience

Both providers run with comparable specs: **1 vCPU** and **1–2 GB RAM** (Vercel's minimum is 2 GB per vCPU; Daytona defaults to 1 GB).

---

## The code

```typescript
// Vercel: create from snapshot, measure startup + first command
async function runVercel(snapshotId: string): Promise<BenchmarkRun> {
  const t0 = performance.now();
  const sandbox = await VercelSandbox.create({
    source: { type: "snapshot", snapshotId },
    resources: { vcpus: 1 }, // 1 vCPU, 2048 MB RAM
  });
  const startupMs = performance.now() - t0;

  const t1 = performance.now();
  const ver = await sandbox.runCommand("claude", ["--version"]);
  const execMs = performance.now() - t1;

  await sandbox.stop();
  return { startupMs, execMs, timeToFirstCmdMs: startupMs + execMs, ... };
}

// Daytona: create from snapshot, measure startup + first command
async function runDaytona(snapshotId: string): Promise<BenchmarkRun> {
  const daytona = new Daytona();

  const t0 = performance.now();
  const sandbox = await daytona.create({ snapshot: snapshotId });
  const startupMs = performance.now() - t0;

  const t1 = performance.now();
  const ver = await sandbox.process.executeCommand("claude --version");
  const execMs = performance.now() - t1;

  await sandbox.delete();
  return { startupMs, execMs, timeToFirstCmdMs: startupMs + execMs, ... };
}
```

Both providers run in parallel across 3 iterations. Results are averaged.

---

## Results

3 runs per provider, measured from a MacBook Pro in Europe (Daytona EU region, Vercel auto-region).

```
Provider        │ Avg Startup │ Avg Exec │ Time-to-1st-Cmd │ Mem Used │ Mem Total
────────────────┼─────────────┼──────────┼─────────────────┼──────────┼──────────
Vercel Sandbox  │   1986 ms   │  1632 ms │      3618 ms    │   92 MB  │  2048 MB
Daytona         │   1230 ms   │   879 ms │      2109 ms    │   23 MB  │  1024 MB
```

Per-run breakdown:

```
Vercel  #1 │ startup: 2210 ms │ exec: 1594 ms │ ttfc: 3804 ms
Vercel  #2 │ startup: 1891 ms │ exec: 1721 ms │ ttfc: 3612 ms
Vercel  #3 │ startup: 1857 ms │ exec: 1581 ms │ ttfc: 3438 ms

Daytona #1 │ startup: 1280 ms │ exec:  998 ms │ ttfc: 2278 ms
Daytona #2 │ startup: 1060 ms │ exec:  847 ms │ ttfc: 1907 ms
Daytona #3 │ startup: 1349 ms │ exec:  793 ms │ ttfc: 2142 ms
```

Daytona is consistently faster: **~38% faster startup** and **~42% faster time-to-first-command**. Both providers show low run-to-run variance, which means the numbers are predictable.

Memory usage inside the sandbox is very low for both (~23–92 MB used at the time of measurement, before any heavy workload), so memory is not a concern at this stage.

---

## Observations

**Daytona** was faster across every single run. The cold-start latency around **1.1–1.4 seconds** is solid. The exec time is also notably lower (~850 ms vs ~1600 ms for Vercel), which may reflect differences in how the toolbox proxy routes commands.

**Vercel Sandbox** is more consistent in startup times (1.8–2.2 s) and has a slightly cleaner SDK — the `runCommand` API is straightforward and the types feel more polished. The higher exec latency is the main drawback.

One note on **Daytona reliability**: I observed occasional outlier runs with 6+ second startups and one unexplained failure during testing. These didn't show up in the final clean runs above but are worth monitoring in production.

---

## Conclusion

Both providers deliver solid, reliable sandboxes. The difference in time-to-first-command — ~2.1 s for Daytona vs ~3.6 s for Vercel — is real, but in practice **sub-3 seconds is perfectly fine for the vast majority of use cases**. An agent waiting an extra second or two to get a sandbox is not a meaningful bottleneck unless you're spinning up sandboxes at very high frequency or building something extremely latency-sensitive.

If cold-start speed is your top priority, Daytona has the edge. But if you value a cleaner SDK, tighter Vercel ecosystem integration, or simply want fewer moving parts, Vercel Sandbox is a perfectly good choice. I'd be comfortable shipping with either.

I'll revisit Cloudflare Sandbox once I have access to the paid plan — running the sandbox co-located inside a Worker could bring TTFC well under 1 second, which would be a more meaningful leap.
