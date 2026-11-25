# 0002. Release a source tarball, not a vcpkg port

- **Status:** accepted
- **Date:** 2026-11-25
- **Issue:** #31

## Context

swarm-routing/.github#881 asks for every C++ release on vcpkg, and the setup
(#24) gave `release.yml` `vcpkg-ports: swarm-adapter-omnet`. After a release
is published, that input makes the org bot render and bump the port in
swarm-routing/vcpkg-registry.

This module cannot build without OMNeT++ and INET (docs/install.md), and vcpkg
packages neither. A port would therefore have to find both outside vcpkg
(`OMNETPP_ROOT`, `INET_ROOT`), which is not what a vcpkg port is for.
Packaging OMNeT++ itself as a port would also redistribute it, and whether
its licence allows that is the open question in
[swarm-routing/sim-containers#265](https://github.com/swarm-routing/sim-containers/issues/265).
Leaving the input in place would make a first release run a port job for a
port that does not exist.

## Decision

A release of this repository ships the module's source tarball
(`make dist`), with the org's checksums, SBOM and signed provenance, and no
vcpkg port. `release.yml` passes no `vcpkg-ports`, and
`.github/org-sync.toml` sets no `vcpkg_ports`. Users build the tarball
against their own OMNeT++/INET install, as docs/install.md describes.

## Consequences

- Releasing does not depend on swarm-routing/vcpkg-registry or on the
  OMNeT++ redistribution question.
- A consumer cannot `vcpkg install` the module. The OMNeT++ way to consume
  it holds instead: a source tree that the user's `opp_makemake` project
  references.
- This is revisited by a new ADR if vcpkg (or the registry) gains OMNeT++
  and INET, or if the registry accepts ports that build against an outside
  install.
