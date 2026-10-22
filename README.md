# AIENOS

## How it works and how to reproduce

I trace one actual source or design boundary in [the worked lesson](docs/TEACHING.md), with a pinned fresh-checkout exercise and [raw replay receipt](docs/teaching-replay-2026-10-22.txt). The lesson states what remains unimplemented or unverified at this revision.

> **Status: experimental / pre-alpha.** The first native boot on the NVIDIA DGX Spark passed on 2026-09-24 ([evidence](evidence/m2_first_boot_2026-09-24.md)); the operating system itself is far from usable. No support, stability, or compatibility promises. Expect breaking changes.

## Help build AIENOS

I am building a sovereign, agent-native operating system in Rust, from the
first instruction after firmware upward, and I welcome help. Kernel isolation,
memory management, interrupts, scheduling, storage, networking and native
inference are all open, and most of it can be developed and tested in QEMU on
any machine: no special hardware needed.

- **[ROADMAP.md](ROADMAP.md)**: the current AIENOS component execution order and live gate status.
- **[docs/PLAN_AUTHORITY.md](docs/PLAN_AUTHORITY.md)**: how AIENOS plans relate to the one whole-system plan in `aien-dev/aien-architecture`.
- **[Open issues](https://github.com/aien-dev/aienos/issues)**: start with
  [`good first issue`](https://github.com/aien-dev/aienos/labels/good%20first%20issue)
  or anything labelled
  [`emulator-ok`](https://github.com/aien-dev/aienos/labels/emulator-ok).
- **[CONTRIBUTING.md](CONTRIBUTING.md)**: build, verify, and the rules for the
  trusted base.
- **[Discussions](https://github.com/aien-dev/aienos/discussions)**: questions,
  design ideas, and introductions.

AIENOS is an agent-native operating system designed around continuous logical agent existence: the agent persists while models, kernels, inference state, power states, and physical machines change beneath it. You turn the machine on, the agent wakes up, knows the machine and your history, operates nearly everything inside it, and asks you only before crossing a boundary you have told it not to cross alone.

AIEN is provisioned once. After that, boot, reboot, sleep, model reload, kernel restart, hardware failure, and migration are execution-state transitions—not agent creation events.

The kernel stays small, deterministic, and non-intelligent. The model is never the kernel, and the agent is never the root of trust.

```text
Firmware
  -> AIEN Boot
  -> AIEN Kernel
  -> AIEN Runtime
  -> AIEN Agent
  -> You
```

An optional compatibility island (for example, Linux with vendor drivers) may sit beside AIENOS while native support is built. It shrinks over time and AIENOS is never designed around it.

## First milestone

Power on, AIENOS boots directly on the NVIDIA DGX Spark (no Linux host), the AIEN agent starts on a local console, a local model loads, you talk to it, and its state persists across reboot. CPU inference is acceptable for this milestone.

Two UEFI images are available. The default diagnostic prints a banner and returns to firmware. The handoff image discovers the CPU topology, memory map, display and GB10 identity, exits UEFI boot services, enters the AIENOS kernel, reports what it found on screen, in a bounded firmware variable and on the serial port, then resets. It booted natively on the DGX Spark on 2026-09-24 (`kernel: alive` at EL2, 20 cores in two efficiency classes, 184 memory-map descriptors with none rejected) and returned to Linux without damage. Kernel isolation, storage, agent-state recovery and model loading are still missing, so this milestone remains open; see [ROADMAP.md](ROADMAP.md).

The AIENOS boot path is a native Rust UEFI entry followed by the AIENOS kernel. It does not use systemd or a Linux init system. The handoff image emits counter-based timings for UEFI entry to kernel handoff and handoff to kernel entry once it runs on hardware. These timings do not include platform firmware time before UEFI starts the image.

AEGIS currently checks capability scope and uses HMAC-SHA256 for capability tokens and operator grants. The broker can own an in-memory J-Space World delta and route `fs.write` and `fs.delete` into it without invoking host handlers; these effects can run without an operator grant only while that World is active. A claimed World ID alone grants nothing. World storage and recovery are still prototypes. Filesystem scope checks enforce lexical path boundaries; native handlers must also resolve symlinks safely before filesystem effects can be considered contained.

## Principles

- **Sovereignty:** no outside organization is required to boot the machine, access your data, authenticate you, authorize the agent, build the trusted core, recover, change models, move hardware, or keep operating.
- **Continuous existence:** the agent is provisioned once; power and substrate changes reconstruct execution, they do not recreate identity.
- **Open trusted base:** boot, kernel, memory management, scheduling, storage, cryptography, identity, AEGIS, capability enforcement, update verification, recovery, and provenance build from inspectable source with a reproducible toolchain. Opaque software may accelerate AIENOS; it may never be required to trust, build, boot, recover, or control it.
- **Fastest thing possible:** close to the metal, measured, with evidence.
- **Free inside reversible state; explicit authorization at irreversible boundaries.**

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for current AIENOS design, [ROADMAP.md](ROADMAP.md) for current AIENOS execution order, [docs/adr/README.md](docs/adr/README.md) for accepted decisions, and [docs/PLAN_AUTHORITY.md](docs/PLAN_AUTHORITY.md) for document precedence. Older blueprint, milestone-matrix, and systems-integration documents are retained as design/reference history and do not override the current roadmap.

## License

Apache-2.0 WITH LLVM-exception. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
