# Constant Bandwidth Server (CBS) for the Linux Kernel

<p align="left">
  <img src="https://img.shields.io/badge/Linux%20Kernel-6.19.9-FCC624?style=flat-square&logo=linux&logoColor=black" alt="Linux Kernel 6.19.9" />
  <img src="https://img.shields.io/badge/C-00599C?style=flat-square&logo=c&logoColor=white" alt="C" />
  <img src="https://img.shields.io/badge/EDF%2FCBS%20Scheduling-EE0000?style=flat-square&logoColor=white" alt="EDF/CBS Scheduling" />
  <img src="https://img.shields.io/badge/Real--Time%20Systems-FF8C00?style=flat-square&logoColor=white" alt="Real-Time Systems" />
</p>

Implementation and evaluation of a **Constant Bandwidth Server (CBS)** as a custom scheduling class within the Linux kernel. CBS is integrated under an **Earliest Deadline First (EDF)** infrastructure to provide **temporal isolation** between hard and soft real-time tasks on single-core systems, allowing soft real-time workloads to share a CPU with hard real-time tasks without ever compromising the schedulability of the latter.

This repository contains both the modified kernel tree and the full technical report documenting the design, implementation, and evaluation of the work.

## Context

Developed for the **Real-Time Operating Systems Programming (RTOPR)** course, part of the **MSc in Critical Computing Systems Engineering (MESCC)** at the Instituto Superior de Engenharia do Porto (ISEP).

## Authors

- Rita Barbosa
- Alfredo Ferreira

## Repository Structure

```
.
├── scheduler-linux-6.19.9-env/   # Modified Linux 6.19.9 kernel tree with the EDF/CBS scheduling class
└── M2_TR_1220841_1220962.pdf     # Full technical report (design, implementation, evaluation)
```

## Background

A CBS reserves a fixed fraction of processor bandwidth, `Us = Qs / Ts`, for a soft real-time task without requiring prior knowledge of its WCET. Jobs bound to a server inherit its current deadline and are scheduled alongside hard tasks using native EDF. The server's behavior is governed by three rules:

1. **Idle arrival with replenishment** — if the remaining budget can't safely cover the time to the current deadline, a new deadline and a full budget are assigned.
2. **Idle arrival with existing budget** — otherwise, the job runs with the current budget and deadline; arrivals while the server is busy are queued FIFO.
3. **Budget exhaustion** — if the budget reaches zero before the job completes, it is replenished and the deadline is postponed by one server period.

This guarantees the **Isolation Property**: for *n* hard periodic tasks with utilization `Up` and a CBS with utilization `Us`, the system is schedulable by EDF iff `Up + Us ≤ 1`, meaning an overrunning soft task can only ever hurt its own deadline, never the hard tasks around it.

## Limitations & Future Work

- Single-core only; multi-core support would require per-CPU run queues and load balancing.
- The user-space syscall interface has not been exhaustively stress-tested with negative/boundary-error cases.
- **CASH (Capacity Sharing)** is identified as a natural extension, allowing idle bandwidth reclamation across servers.

## Report

The full report — [`M2_TR_1220841_1220962.pdf`](./M2_TR_1220841_1220962.pdf) — covers the theoretical background, complete implementation details (including annotated source excerpts), evaluation methodology, and trace analysis in depth.

## References

1. L. Abeni and G. Buttazzo, "Integrating multimedia applications in hard real-time systems," in *Proceedings of the 19th IEEE Real-Time Systems Symposium*, 1998.
2. Bootlin Elixir Cross Referencer — Linux kernel source (v4.5): `schedule_hrtimeout_range_clock`. <https://elixir.bootlin.com/linux/v4.5/ident/schedule_hrtimeout_range_clock>
3. Bootlin — Embedded Linux kernel source browser. <https://bootlin.com>
