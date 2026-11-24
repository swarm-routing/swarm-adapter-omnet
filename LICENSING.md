# Licensing

Copyright (c) 2026 Daniel Joppi (https://github.com/swarm-routing).
Licensed per directory, as AntHocNet
([LICENSING.md](https://github.com/swarm-routing/AntHocNet/blob/main/LICENSING.md),
[ADR-0028](https://github.com/swarm-routing/AntHocNet/blob/main/docs/adr/0028-core-is-dual-licensed.md))
and [swarm-adapter-ns3](https://github.com/swarm-routing/swarm-adapter-ns3/blob/main/LICENSING.md).
Here every path is `GPL-2.0-only`: the repository is an adapter, and has no
dual-licensed part of its own.

| Path | What | Licence |
|------|------|---------|
| `src/` | the OMNeT++/INET module: the adapter's C++ (`.h`, `.cc`) and its NED types (`.ned`) | `GPL-2.0-only` |
| `test/` | the module's tests (OMNeT++ `opp_test` files and unit tests) | `GPL-2.0-only` |
| `examples/`, `simulations/` | example networks and their `omnetpp.ini` | `GPL-2.0-only` |
| `CMakeLists.txt`, `Makefile`, `swarm-core.lock`, `.cz.toml`, `VERSION` | the build, and the swarm-routing/swarm-core release it builds on | `GPL-2.0-only` |
| `tools/` | the repository's scripts (checks, release) | `GPL-2.0-only` |
| `docs/`, `README.md`, `AGENTS.md`, `CLAUDE.md`, `CONTEXT.md` | docs | `GPL-2.0-only` |
| everything else | | `GPL-2.0-only` |

A new directory gets its row here in the same PR (golden rule 1).

<!-- org-sync:begin licence-texts -->
Texts: [`LICENSE`](LICENSE) (GPL-2.0) and [`LICENSE-MIT`](LICENSE-MIT) (the
MIT option). Each source file states its licence in its
`SPDX-License-Identifier` line; CI checks it
(`swarm-routing/.github` `cmd/check-headers`; locally `make headers`).
<!-- org-sync:end licence-texts -->

## What this means in practice

- **The OMNeT++/INET module is GPL-2.0-only.** INET is LGPL-3.0 and
  OMNeT++ ships under its own Academic Public License; a simulation built
  with this module combines them. This repository distributes only its own
  sources and the release tarball of them, never OMNeT++ or INET. Whether
  the OMNeT++ licence lets the org publish a container image with it built in
  is an open question, settled before any image ships
  ([swarm-routing/sim-containers#265](https://github.com/swarm-routing/sim-containers/issues/265),
  the OMNeT++/INET CI image).
- **The licence boundary only goes one way** (golden rule 2). This module
  uses swarm-core's simulator-free ports and primitives (`core/`, `MIT OR
  GPL-2.0-only`, in [swarm-routing/swarm-core](https://github.com/swarm-routing/swarm-core))
  at the release `swarm-core.lock` names, and a protocol's dual-licensed
  `core/`. Nothing here is copied into those: no OMNeT++ or INET header, no
  GPL-only code reaches a dual-licensed `core/`. A primitive or port the
  adapter needs is written in swarm-core, under its licence, never here.
- **No `core/` here.** The repository holds no simulator-free code of its own
  (golden rule 11 of [AGENTS.md](AGENTS.md)), so the licence check runs with
  empty `dual-roots` and `core-roots` (`.github/workflows/licence.yml`).
- **Headers.** Every original `.h`/`.cc`/`.cpp`/`.hpp`/`.py`/`.sh` starts
  with the org copyright line and `SPDX-License-Identifier: GPL-2.0-only`;
  `.ned` files carry the same two lines as `//` comments and `omnetpp.ini`
  files as `#` comments, by convention (the CI check reads the C++ and script
  extensions).
- **Code from elsewhere keeps its licence.** Glue ported from
  swarm-adapter-ns3 or AntHocNet's `ns3/` is GPL-2.0-only and may land here;
  each file names its source and commit. Third-party code (INET's examples,
  say) lands only with its provenance recorded and a compatible licence.
- **Contributions** are accepted under GPL-2.0-only.
