# CONTEXT.md — orientation and history

Background an agent or a new contributor needs that does not belong in
`AGENTS.md`: domain notes, current state, glossary, open questions. Read the
section a task needs, not the whole file.

## Current state

**Set up, no code yet** (#24, part of swarm-routing/.github#881, 2026-11-24).
The org templates, LICENSING.md, CITATION.cff, the release ladder
([docs/roadmap.md](docs/roadmap.md)), the install guide
([docs/install.md](docs/install.md)) and `release.yml` are in place. The
implementation is epic #2; its v0.1.0 children carry `release:v0.1.0`.
The first code PR needs the OMNeT++ 6.5.0 / INET 4.7.0 CI image from
swarm-routing/sim-containers#268, which is not published yet. Its licence was
settled on swarm-routing/sim-containers#265: publishing is allowed with the
full source trees inside the image.

The model is swarm-routing/swarm-adapter-ns3: its `SwarmRoutingBase` is what
the ns-3 side gives a core, and this adapter gives a core the same on INET.

## Glossary

- **Port**: an interface swarm-core's simulator-free code calls for what only
  a runtime can give (clock, randomness, timers, neighbours, sending). This
  repository implements the ports on OMNeT++/INET (an adapter).
- **Core**: a protocol's simulator-free `core/` (dual-licensed), which makes
  every routing decision.
- **NED**: OMNeT++'s network description language; the module's simple
  modules and their parameters are declared in `.ned` files.
- **INET**: the OMNeT++ model library for the Internet stack, MANET routing
  and wireless; the adapter plugs into its routing hooks.

## Open questions

- Keep this module GPL-2.0-only and ship source only, or relicense it so a
  binary combined with OMNeT++ (APL) can be distributed (#38).
- Which INET hook the adapter uses for deferred packets (INET's netfilter
  hooks, as INET's own MANET routing protocols do, is the likely answer); the
  first port PR decides and records it in an ADR.
