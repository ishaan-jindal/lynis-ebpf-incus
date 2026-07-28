# lynis-ebpf-incus

Private Lynis custom tests: eBPF kernel hardening + Incus container security.

Drop-in `CUST-*` plugins for Lynis `audit system`. No fork of Lynis required.

## Contents

```
include/tests_ebpf_security   CUST-8500 – CUST-8508
include/tests_incus_security  CUST-8600 – CUST-8608
LICENSE                       GPLv3 (matches upstream Lynis)
```

## Script 1 — eBPF Kernel Security

**File:** `include/tests_ebpf_security` · **OS:** Linux

Audits five sysctl knobs that limit eBPF-based privilege escalation and kernel info disclosure:

| ID | Check | Target |
|---|---|---|
| CUST-8500 | `kernel.unprivileged_bpf_disabled` | 1 or 2 |
| CUST-8502 | `net.core.bpf_jit_harden` | 2 |
| CUST-8504 | `kernel.kptr_restrict` | 2 |
| CUST-8506 | `kernel.perf_event_paranoid` | ≥2 (prefer 3) |
| CUST-8508 | `kernel.dmesg_restrict` | 1 |

Missing key = `NOT_FOUND` (old kernel), bad value = `WARNING` with `sysctl:` fix pointing at `/etc/sysctl.d/99-ebpf.conf`. Skips cleanly without `sysctl` binary.

## Script 2 — Incus Container & VM Security

**File:** `include/tests_incus_security` · **OS:** Linux

| ID | Check | Target |
|---|---|---|
| CUST-8600 | `incusd` / `incus` binary presence | Installed |
| CUST-8602 | `/var/lib/incus/` dir perms | 700 `root:root` |
| CUST-8604 | `/etc/incus/` server key/cert perms | 600 / 644 |
| CUST-8606 | `security.privileged=true` instances | None |
| CUST-8608 | Daemon running & socket perms | Running, 660 |

Notes:
- Daemon search: `/usr/bin/incusd`, `/usr/lib/incus/incusd`, `/usr/sbin/incusd`, then `PATH`.
- Missing `/etc/incus/` (e.g. Arch) reports `NOT_FOUND` — expected.
- `CUST-8606` gates on `incus info` reachability, then parses `incus list --format json`.

## Install

Copy/symlink into your Lynis checkout, then register in `lynis` `INCLUDE_TESTS` if not already (see upstream `lynis` script):

```bash
cp include/tests_* /path/to/lynis/include/
# or
ln -s $PWD/include/tests_ebpf_security /path/to/lynis/include/tests_ebpf_security
ln -s $PWD/include/tests_incus_security /path/to/lynis/include/tests_incus_security
```

## Run

```bash
# from your lynis checkout
./lynis audit system --quick --usecwd --tests-from-group ebpf_security
./lynis audit system --quick --usecwd --tests-from-group incus_security

# both together
./lynis audit system --quick --usecwd

# individual IDs
./lynis audit system --quick --usecwd --tests "CUST-8500 CUST-8606"
```

`--usecwd` matters: without it Lynis may resolve `/usr/share/lynis/include` instead of `./include` and your custom tests won't appear.

## License

GPLv3 — see `LICENSE`.
