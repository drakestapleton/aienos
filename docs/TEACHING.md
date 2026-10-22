# Learn this Rust OS line

At `603c91d8ec2e6b1d7e2db25591275662eb5c92f0`, this personal repository contains a substantial experimental Rust UEFI/kernel implementation, not just an old sketch. Its [roadmap](../ROADMAP.md) is a component plan, while [plan authority](PLAN_AUTHORITY.md) assigns whole-system order elsewhere. The separately maintained [aienux](https://github.com/aien-dev/aienux) is the current C-kernel teaching route; do not treat its commands or evidence as a replay of this Rust tree.

A concrete boundary in this tree is [`aienos-evidence::verify::check`](../crates/aienos-evidence/src/verify.rs): it validates a captured Config A bundle's checksum, schema, host and hardware fields, then separately reports `m0_open` when a sound capture still lacks rollback qualification. That split prevents a structurally good host capture from being sold as completed native boot. [The first native boot record](../evidence/m2_first_boot_2026-09-24.md) reports one Spark event; [the README](../README.md) still says isolation, storage, recovery and model loading remain incomplete. QEMU receipts are emulator evidence, not physical hardware proof.

## Fresh checkout exercise (offline, no boot)

Prerequisites: Git and Python 3. Checkout `603c91d8ec2e6b1d7e2db25591275662eb5c92f0`, then run from repository root:

```sh
python3 - <<'PYEX'
from pathlib import Path
s=Path('crates/aienos-evidence/src/verify.rs').read_text()
assert 'pub fn check(bundle: &Value) -> Report' in s
assert 'bundle checksum mismatch' in s and 'm0_open' in s
print('CAPTURE CHECK AND OPEN-GATE PATH PRESENT')
PYEX
```

Expected that line, exit 0. This only inspects the actual guard source. Native boot and QEMU reproduction require their specific toolchains and safe target images; neither is claimed by this host check. The negative case is a checksum mismatch reported by the function, whose behavior needs a Rust test to qualify independently. [Replay receipt](teaching-replay-2026-10-22.txt).
