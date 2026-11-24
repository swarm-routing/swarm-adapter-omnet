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
| 6.5.0 (October 2026) | 4.6.0 (February 2026) | the release `swarm-core.lock` names | **target of v0.1.0**: the pair the first build PR pins and CI runs |
| 6.2.x – 6.4.x | 4.6.0 | same | not tested: INET 4.6 requires OMNeT++ 6.2 or later; may build, no promise |
| 6.1.x and older | 4.5.x and older | | not supported |

Release sources: OMNeT++
[tags](https://github.com/omnetpp/omnetpp/tags) and
[omnetpp.org/download](https://omnetpp.org/download/); INET
[tags](https://github.com/inet-framework/inet/tags) and its
[WHATSNEW](https://github.com/inet-framework/inet/blob/v4.6.0/WHATSNEW)
(the OMNeT++ version each INET release requires).

## Linux

Ubuntu 24.04 LTS (x86_64) is the platform CI will run on, through the
OMNeT++/INET image in swarm-routing/sim-containers once it exists. Other
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
  python3 python3-pip python3-venv libxml2-dev zlib1g-dev cmake git
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
```

[opp_env](https://github.com/omnetpp/opp_env) (`opp_env install
omnetpp-6.5.0 inet-4.6.0`) installs the same pair in one step and is a
supported alternative.

### INET

```bash
git clone --branch v4.6.0 --depth 1 https://github.com/inet-framework/inet.git
cd inet && . ./setenv && make makefiles && make -j"$(nproc)" MODE=release
```

### swarm-adapter-omnet

Not buildable yet: the first build PR (a child of epic #2) adds the module,
`swarm-core.lock` and the `make build` / `make test` recipes, and writes the
steps here. They will read `OMNETPP_ROOT` and `INET_ROOT` from the
environment (the two `setenv` scripts above set them).

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

- **INET 4.6's clock model** (WHATSNEW "Clock model refinement"): a clock's
  nominal tick interval must now be configured, it no longer defaults to the
  simulation time resolution. The adapter's clock port reads `simTime()`, not
  an INET clock, so it is unaffected; an example network that adds INET clocks
  must set the interval.
- **INET 4.6 requires OMNeT++ 6.2 or later**; older OMNeT++ releases do not
  build it.
