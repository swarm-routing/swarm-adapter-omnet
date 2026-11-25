# swarm-adapter-omnet

<!-- org-sync:begin badges -->
| Group | Badges |
| --- | --- |
| Build | [![Built on: ns3-dev](https://img.shields.io/badge/built%20on-ns3--dev%20image-blue?logo=docker)](https://github.com/orgs/swarm-routing/packages/container/package/ns3-dev) |
| Quality | [![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=swarm-routing_swarm-adapter-omnet&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=swarm-routing_swarm-adapter-omnet) [![Bugs](https://sonarcloud.io/api/project_badges/measure?project=swarm-routing_swarm-adapter-omnet&metric=bugs)](https://sonarcloud.io/summary/new_code?id=swarm-routing_swarm-adapter-omnet) [![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=swarm-routing_swarm-adapter-omnet&metric=security_rating)](https://sonarcloud.io/summary/new_code?id=swarm-routing_swarm-adapter-omnet) [![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=swarm-routing_swarm-adapter-omnet&metric=sqale_rating)](https://sonarcloud.io/summary/new_code?id=swarm-routing_swarm-adapter-omnet) [![CodeRabbit Pull Request Reviews](https://img.shields.io/coderabbit/prs/github/swarm-routing/swarm-adapter-omnet?utm_source=oss&utm_medium=github&utm_campaign=swarm-routing%2Fswarm-adapter-omnet&labelColor=171717&color=FF570A&link=https%3A%2F%2Fcoderabbit.ai&label=CodeRabbit+Reviews)](https://coderabbit.ai) |
| Release | [![Release](https://img.shields.io/github/v/release/swarm-routing/swarm-adapter-omnet?sort=semver&cacheSeconds=1800)](https://github.com/swarm-routing/swarm-adapter-omnet/releases) [![Cite](https://img.shields.io/badge/cite-CITATION.cff-blue)](CITATION.cff) [![Licence: GPL-2.0-only](https://img.shields.io/badge/licence-GPL--2.0--only-blue)](LICENSING.md) |
| Trust | [![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/swarm-routing/swarm-adapter-omnet/badge)](https://scorecard.dev/viewer/?uri=github.com/swarm-routing/swarm-adapter-omnet) [![OpenSSF Best Practices](https://www.bestpractices.dev/projects/17073/badge)](https://www.bestpractices.dev/projects/17073) |
| Stack | [![C++23](https://img.shields.io/badge/C%2B%2B-23-blue?logo=cplusplus)](AGENTS.md) |
| Community & docs | [![Colony Arena: swarm-adapter-omnet](https://img.shields.io/badge/Colony%20Arena-project%20page-orange)](https://swarm-routing.github.io/projects/swarm-adapter-omnet/) [![Open RCAPA](https://img.shields.io/github/issues/swarm-routing/swarm-adapter-omnet/rcapa?label=open%20RCAPA&color=blue)](https://github.com/swarm-routing/swarm-adapter-omnet/blob/main/docs/rcapa/README.md) [![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow)](https://www.conventionalcommits.org) |
<!-- org-sync:end badges -->

swarm-adapter-omnet, the OMNeT++/INET adapter of swarm-core's ports
(swarm-routing/.github#672, layer 2): the in-process simulator adapter a
protocol's simulator-free `core/` runs on inside OMNeT++, as
[swarm-adapter-ns3](https://github.com/swarm-routing/swarm-adapter-ns3) is
for ns-3. Every in-process C++ simulator gets its own `swarm-adapter-<sim>`
repository (ROADMAP decision 24); orchestration, statistics and the
non-adapter glue stay in [sim-tools](https://github.com/swarm-routing/sim-tools).

It gives a protocol's core the clock, randomness, timers, neighbours and link
state, the deferred packet queue and the control-packet send path on
OMNeT++/INET, and carries out the core's decisions. It holds no routing logic.

**Status: set up, no code yet.** Code lands by PR from epic #2's children,
in the order of the [roadmap](docs/roadmap.md): v0.1.0 is the module built
against a pinned OMNeT++/INET release, running one protocol core. The
simulator track's schedule is
[ROADMAP decision 23](https://github.com/swarm-routing/.github/blob/main/ROADMAP.md#architecture-and-migration-decided-2026-11-22)'s;
results across simulators are qualitative only (decision 8). Install and the
supported versions: [docs/install.md](docs/install.md).

## Rules

The org's golden rules 1–9 hold here (full text in [AGENTS.md](AGENTS.md)):
licensing per directory; the licence boundary only goes one way; POD,
versioned wire formats; no contrib module named `core`; determinism from a
fixed seed; paired comparisons; bounded structures stay bounded; CI hygiene;
nothing shipped depends on `.claude/`. This adapter adds its own:

10. **No routing logic**: it converts OMNeT++/INET events into swarm-core's
    port calls and carries out the core's decisions, never makes one.
11. **Built on swarm-core's ports by release lock**: no `core/` or
    primitive of its own; a missing port is an issue in
    [swarm-core](https://github.com/swarm-routing/swarm-core).
12. **Pinned simulator, seeded randomness**: the OMNeT++/INET versions
    [docs/install.md](docs/install.md) lists; randomness from OMNeT++'s seeded
    streams only.
13. **Every glue change carries a test.**
14. **Downstream API stability** for the protocols that build on it.

<!-- org-sync:begin develop -->
## Develop

New here? [docs/development.md](docs/development.md) is the developer guide:
the tools to install, how to get the code and its dependencies, how to
build, test, run and lint it, the Docker images, what CI checks and how to
read a red run. How changes are proposed, reviewed and released is the org's
[CONTRIBUTING.md](https://github.com/swarm-routing/.github/blob/main/CONTRIBUTING.md).
Whether a regression here gets an RCAPA (root cause analysis with corrective
and preventive actions), and how: [docs/rcapa/README.md](docs/rcapa/README.md).
<!-- org-sync:end develop -->

<!-- org-sync:begin citing -->
## Releases & citing

Versions follow [SemVer](https://semver.org). Each release is listed under [Releases](https://github.com/swarm-routing/swarm-adapter-omnet/releases), built by the org's release workflow and archived on [Zenodo](https://zenodo.org) with its own DOI.

To cite this repository, use the “Cite this repository” button (from [CITATION.cff](CITATION.cff)). It has no DOI yet: the concept DOI appears here and in the badge above once Zenodo archives its first release.

Please also cite the work it builds on:

- D. H. Joppi, *swarm-core*, 2026, [doi:10.5281/zenodo.24476619](https://doi.org/10.5281/zenodo.24476619). The ports this repository is the OMNeT++/INET adapter of (swarm-routing/swarm-core), built on by release lock.
- A. Varga, R. Hornig, *AN OVERVIEW OF THE OMNeT++ SIMULATION ENVIRONMENT*, Proceedings of the First International ICST Conference on Simulation Tools and Techniques for Communications Networks and Systems, 2008, [doi:10.4108/icst.simutools2008.3027](https://doi.org/10.4108/icst.simutools2008.3027). The simulator the adapter is written for.

Using it in your work? Contributions are welcome, and citing it helps just as much.
<!-- org-sync:end citing -->

## Epics

<!-- epics:begin -->
<!-- Generated by swarm-routing/.github tools/epics/epics.py from the issues labelled `epic`; edit the issues, then re-run it. -->

What is planned and in progress here: the issues labelled [`epic`](https://github.com/swarm-routing/swarm-adapter-omnet/issues?q=label%3Aepic). An epic's open sub-issues carry a `priority:*` and a `model:*` label, so a contributor or an AI agent can pick one up ([how issues work](https://github.com/swarm-routing/.github/blob/main/CONTRIBUTING.md#issues)). The whole org's plan: the [roadmap](https://github.com/swarm-routing/.github/blob/main/ROADMAP.md#epics).

| Epic | Status | Sub-issues closed |
|------|--------|-------------------|
| [#2](https://github.com/swarm-routing/swarm-adapter-omnet/issues/2) OMNeT++/INET adapter of swarm-core's ports (deferred: decisions 23, 24) | Planned | 0 / 7 |

<!-- epics:end -->

## Licence

GPL-2.0-only, the whole repository: see [LICENSING.md](LICENSING.md).
