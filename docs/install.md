# Install

How to install OMNeT++ and INET on Linux and build swarm-adapter-omnet
against them. There is no code yet (#24): this page is the contract the first
build PR implements, and it grows with the module. The developer workflow
(`make check`, CI, the release) is [development.md](development.md).

## Supported versions

The module builds against **one pinned OMNeT++/INET pair** (golden rule 12 of
[AGENTS.md](../AGENTS.md)). A version change is a single PR that moves the
pin, this table and the CI image together.

| OMNeT++ | INET | swarm-core | Status |
|---------|------|------------|--------|
| 6.5.0 (October 2026) | 4.7.0 (July 2026) | the release `swarm-core.lock` names | **target of v0.1.0**: the pair the first build PR pins and CI runs |
| 6.4.x | 4.7.0 | same | not tested: INET 4.7 requires OMNeT++ 6.4 or later; may build, no promise |
| 6.2.x – 6.5.x | 4.6.x | same | not tested: INET 4.6 requires OMNeT++ 6.2 or later; may build, no promise |
| 6.1.x and older | 4.5.x and older | | not supported |

Release sources: OMNeT++
[tags](https://github.com/omnetpp/omnetpp/tags) and
[omnetpp.org/download](https://omnetpp.org/download/); INET
[tags](https://github.com/inet-framework/inet/tags) and its
[WHATSNEW](https://github.com/inet-framework/inet/blob/v4.7.0/WHATSNEW)
(the OMNeT++ version each INET release requires).

## Licences

OMNeT++ is under its **Academic Public License 1.1**
(`doc/License` in the release tarball). It permits use "for noncommercial
purposes, including teaching and academic research". Commercial use,
"joint research with a commercial entity, government-funded, EU-funded,
military or similar research projects" need an OMNEST licence
(www.omnest.com). INET is LGPL-3.0-or-later. Check that your use is
noncommercial under APL §1 before you install OMNeT++. How this bounds what
the org distributes: [LICENSING.md](../LICENSING.md),
[swarm-routing/sim-containers#265](https://github.com/swarm-routing/sim-containers/issues/265).

## Linux

Ubuntu 24.04 LTS (x86_64) is the platform CI will run on, through the
OMNeT++/INET image `ghcr.io/swarm-routing/omnetpp-dev:6.5.0-inet4.7.0`
once it is published
([swarm-routing/sim-containers#268](https://github.com/swarm-routing/sim-containers/issues/268)). Other
recent distributions work if the tools below are new enough; macOS and
Windows are not supported by this repository (OMNeT++ itself supports them).

### Required tools

| Tool | Version | Why |
|------|---------|-----|
| GCC or Clang | GCC 13+ or Clang 17+ | the module and swarm-core are C++23; OMNeT++ itself still builds as C++17, which links with C++23 code on the same libstdc++ |
| GNU Make, CMake | Make 4.3+, CMake 3.28+ | OMNeT++'s `opp_makemake` build; swarm-core's CMake package |
| bison, flex, perl | distribution versions | building OMNeT++ from source |
| Python 3 | 3.11+ | OMNeT++'s `opp_*` tools and its Python requirements (`python/requirements.txt` in the OMNeT++ tree); the org tooling `make check` fetches |
| libxml2, zlib | `libxml2-dev`, `zlib1g-dev` | OMNeT++'s NED and XML parameter support |
| git, Docker | any recent | the sources; `make headers` and the CI image |

Headless builds (CI, batch runs with `Cmdenv`) need nothing graphical. Qtenv
(the GUI) also needs Qt 6 and OpenGL; OMNeT++'s Install Guide
(`doc/InstallGuide.pdf` in its release tarball) has the full package list per
distribution.

On Ubuntu 24.04:

```bash
sudo apt-get update
sudo apt-get install -y build-essential clang lld gdb bison flex perl \
  python3 python3-dev python3-pip python3-venv libxml2-dev zlib1g-dev cmake git
```

### OMNeT++

```bash
# the release tarball from omnetpp.org/download, then:
tar xzf omnetpp-6.5.0-linux-x86_64.tgz && cd omnetpp-6.5.0
python3 -m venv .venv && . .venv/bin/activate
python3 -m pip install -r python/requirements.txt
. ./setenv
./configure WITH_QTENV=no WITH_OSG=no   # headless; drop both for the GUI
make -j"$(nproc)"
export OMNETPP_ROOT="$PWD"   # OMNeT++'s setenv does not export it; the adapter's build reads it
```

[opp_env](https://github.com/omnetpp/opp_env) is a supported alternative.
In an empty directory, run `opp_env init`, then
`opp_env install omnetpp-6.5.0 inet-4.7.0`. The installed packages are usable
only inside `opp_env shell` or `opp_env run`.

### INET

```bash
git clone --branch v4.7.0 --depth 1 https://github.com/inet-framework/inet.git
cd inet && . ./setenv && make makefiles && make -j"$(nproc)" MODE=release
```

### swarm-adapter-omnet

Not buildable yet: the first build PR (a child of epic #2) adds the module,
`swarm-core.lock` and the `make build` / `make test` recipes, and writes the
steps here. They will read `OMNETPP_ROOT` (exported by hand after the
OMNeT++ build above) and `INET_ROOT` (exported by INET's `setenv`) from the
environment.

## Known simulator bugs

An OMNeT++ or INET bug that affects this module, or a workaround for one,
gets an RCAPA ticket first (swarm-routing/sim-tools, as for ns-3; CONTRIBUTING
"RCAPA"), then a row here, and every workaround in the code links its ID
(golden rule 12). Upstream reports go to the OMNeT++ or INET issue tracker
and are linked from the row.

| ID | Simulator, versions | Symptom | Workaround | Upstream |
|----|---------------------|---------|------------|----------|
| | | none recorded yet | | |

Known incompatibilities that are not bugs:

- **INET 4.6's clock model**, kept in 4.7 (WHATSNEW "Clock model
  refinement"): a clock's nominal tick interval must now be configured, it no
  longer defaults to the simulation time resolution. The adapter's clock port
  reads `simTime()`, not an INET clock, so it is unaffected; an example
  network that adds INET clocks must set the interval.
- **INET 4.7's RNG grouping** (WHATSNEW "RNG grouping"): the new
  `GroupedRngManager` shares RNGs per module (the default), per node or
  network-wide, chosen by the `rng-grouping` option. The adapter's RNG port
  must take its stream from one documented grouping, and the example
  networks must set `rng-grouping` explicitly, so that a fixed seed keeps
  giving byte-identical output (golden rule 5) whatever INET's default.
- **INET 4.7 requires OMNeT++ 6.4 or later** (4.6 requires 6.2 or later);
  older OMNeT++ releases do not build it.
