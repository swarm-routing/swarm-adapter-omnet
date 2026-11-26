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

<!-- org-sync:begin known-bugs -->
<!-- Generated from swarm-routing/sim-tools docs/ns3-bugs.md by swarm-routing/.github
     cmd/ns3-bugs (swarm-routing/.github#884); never edit by hand: a change goes to the register. -->
Upstream ns-3 defects our repositories meet, from the org's [ns-3 bug register](https://github.com/swarm-routing/sim-tools/blob/main/docs/ns3-bugs.md).
Each ID links its RCAPA (the reproduction, the root cause, the upstream report).

| ID | What breaks | ns-3 versions | Status | Our fix or workaround |
|----|-------------|---------------|--------|-----------------------|
| [NS3-001](https://github.com/swarm-routing/sim-tools/blob/main/docs/ns3-rcapa/ns3-rcapa-aodv-icmp-livelock.md) | AODV livelocks in a zero-time ICMP "net unreachable" storm | 3.49 and ns-3-dev `master` `8ec9b4a1` (run); not 3.48 | open upstream; reported, fix proposed (not ours); worked around | AODV-family arms on `ns3-dev:3.48` (AntHocNet CI) until the harness guard reaches them: `routing-bench` wraps the `aodv` arm in a guard ([swarm-routing/sim-tools#322](https://github.com/swarm-routing/sim-tools/pull/322)) applying [nsnam/ns-3-dev!3101](https://gitlab.com/nsnam/ns-3-dev/-/merge_requests/3101)'s rules (maintainer decision 2026-11-23, swarm-routing/sim-tools#238); strict XFAIL `3.49 sparse-aodv` in swarm-routing/sim-containers `test/known-failures.txt`; the patch `ns3/patches/NS3-001/` ([nsnam/ns-3-dev!3101](https://gitlab.com/nsnam/ns-3-dev/-/merge_requests/3101) at `d3fbcacb`; `apply.sh` applies it only to a source with the bug, [swarm-routing/sim-tools#378](https://github.com/swarm-routing/sim-tools/issues/378)), which the vcpkg `ns3` port applies by the same rule (maintainer decision 2026-11-25, [swarm-routing/vcpkg-registry#81](https://github.com/swarm-routing/vcpkg-registry/pull/81); its own copy until it takes the released asset, [swarm-routing/sim-tools#380](https://github.com/swarm-routing/sim-tools/issues/380)) |
| [NS3-002](https://github.com/swarm-routing/sim-tools/blob/main/docs/ns3-rcapa/ns3-rcapa-socket-map-heap-order.md) | AODV and DSDV order per-interface sockets by heap address | 3.42, 3.48, 3.49 and ns-3-dev `master` `8ec9b4a1`, `a150de8f` (run); 3.42, 3.48, 3.49 (read) | open upstream; reported; worked around | `--isolateArms` (default since v1.1.0); AntHocNet: its own maps ordered, stock `aodv` in its own process, the order gate on `anthocnet` rows only |
| [NS3-003](https://github.com/swarm-routing/sim-tools/blob/main/docs/ns3-rcapa/ns3-rcapa-dsdv-null-route.md) | DSDV sends on a route with a null output device (assert, or a silent SIGSEGV when optimized) | 3.36–3.48 and ns-3-dev 2026-08 (run); 3.49 (read) | open upstream; reported, fix proposed; worked around | AntHocNet's `isl-grid` refuses `dsdv` on multi-interface topologies, checked by its satellite arm smoke |
| [NS3-004](https://github.com/swarm-routing/sim-tools/blob/main/docs/ns3-rcapa/ns3-rcapa-aodv-hello-restart.md) | AODV never restarts its hello timer when an interface comes back up (a restarted node sends no HELLO for the rest of the run) | 3.49 and ns-3-dev `master` `8a06b457` (run); 3.49, `master` (read) | open upstream; draft ready | none: `--failures` is opt-in; `ns3/routing-bench/README.md` "Scripted failures" names the limit ([swarm-routing/sim-tools#361](https://github.com/swarm-routing/sim-tools/pull/361), open) |
| [NS3-005](https://github.com/swarm-routing/sim-tools/blob/main/docs/ns3-rcapa/ns3-rcapa-aodv-lifetime-overflow.md) | AODV route lifetime overflows int64 when an interface comes up after t=0 (`GetMaximumSimulationTime() + Now()`: UBSan aborts; undefined behaviour otherwise) | 3.49 and ns-3-dev `master` `d07400ba` (run); 3.7–3.49, `master` (read) | open upstream; reported; patch kept here and in the patched images | the patch `ns3/patches/NS3-005/` (`apply.sh` applies it only to a source with the bug), applied in swarm-routing/sim-containers `ns3-dev:3.49-{debug,sanitized}-patched-5c58dfa` (publish run 45712453689, the patch at sim-tools `1b36c739`); `--failures` AODV restarts ([swarm-routing/sim-tools#361](https://github.com/swarm-routing/sim-tools/pull/361)) abort on sanitized stock builds: the suite leaves them out where the source has the bug and runs them on the patched sanitized image (`ns3.yml` `failures-patched`) |

Fixed upstream, but present in an older ns-3 release you may still pin:

| What breaks | ns-3 versions | Fixed in | Our fix or workaround |
|-------------|---------------|----------|-----------------------|
| 802.11b HR/DSSS (5.5, 11 Mb/s) frames dropped as an unsupported modulation: `GetModulationClassForStandard(WIFI_STANDARD_80211b)` returns `WIFI_MOD_CLASS_DSSS`, and since 3.41 `WifiPhy::StartReceivePreamble` refuses any class above it | 3.42 (run); 3.41, 3.42 (read); not 3.40 or earlier (no gate, read) | 3.43 (`da9fcebe`) | strict XFAIL `3.41`/`3.42` `dsdv-manet`, `dense-{aodv,olsr,dsdv}` (`no-packets-delivered`) in swarm-routing/sim-containers `test/known-failures.txt` ([swarm-routing/sim-containers#250](https://github.com/swarm-routing/sim-containers/pull/250)); [swarm-routing/AntHocNet#923](https://github.com/swarm-routing/AntHocNet/issues/923) to check for 802.11b above 2 Mb/s on 3.42 |
| Four wifi unit suites (`wifi-txop`, `wifi-mlo`, `wifi-gcr`, `wifi-emlsr-basic-exchanges`) fail when `test-runner` is built without the `test` module: the tests match frames by packet UID, which needs packet metadata on, and 3.48 turns it on only through `src/test`'s `ns3tcp` suites (a test bug; `wifi-txop` then reads past its frame list) | 3.48 (run) | 3.49 (the tests use the MPDU's own UID) | strict XFAIL `3.48 unit:<suite>` in swarm-routing/sim-containers `test/known-failures.txt` |
<!-- org-sync:end known-bugs -->

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
