# Developer guide

swarm-adapter-omnet is planned and deferred (swarm-routing/.github ROADMAP
decisions 23 and 24): no code is here yet, only the org's templates and
settings. This guide says what runs today; each section grows with the code.
What it will hold: the OMNeT++/INET adapter of swarm-core's ports, a C++23
project.

## Prerequisites

GNU Make, git, Python 3.11 or newer (the org tooling `make check` fetches),
and either a running Docker daemon or Go for the licence-header gate that
`make check` runs (`make headers`: the org's `check-headers.sh` runs the
ci-tools image with Docker, else builds `cmd/check-headers` with Go). Docker
also runs `make shell`. OMNeT++ and INET come in with the first code, once
their image exists in swarm-routing/sim-containers; the versions and the
install steps are [install.md](install.md).

## Get the code

```bash
git clone https://github.com/swarm-routing/swarm-adapter-omnet.git
cd swarm-adapter-omnet
```

No submodules or sibling checkouts are needed.

## Dependencies

None yet. `make check` fetches swarm-routing/.github's tooling into
`~/.cache/swarm-routing/dotgithub` (the licence-header gate and org-sync).

## Build

Nothing to build yet: `make build` and `make test` say `no CMakeLists.txt yet` until the CMake project exists.

## Test

`make check` is the gate today: lint, test, licence headers and the
org-sync check, as CI runs them.

```bash
make check
```

## Run

Nothing runs yet.

## Docker and dev containers

`make shell` opens a shell in the `ghcr.io/swarm-routing/ns3-dev` image
(the org's default; there is no OMNeT++ image yet) with this repository at
`/src`. There is no `.devcontainer/`.

## Lint and format

`make lint` (clang-format on any C++ files) and `make format`. Licence
headers: `make headers`, the org's `check-headers` gate.

## CI map

| Workflow | What it checks | Required |
| --- | --- | --- |
| `org-sync.yml` | the org-managed files match the pinned templates | yes (`check / org-sync`) |
| `licence.yml` | SPDX headers and the licence texts | yes (`headers / licence`) |
| `pr-title.yml` | the PR title is a lowercase Conventional Commit | yes (`title / title`) |
| `review-gate.yml` | CodeRabbit has reviewed the PR | yes (`review / coderabbit`) |
| `dco.yml` | an outside contributor signed off | no |
| `scorecard.yml`, `bestpractices-evidence.yml` | OpenSSF Scorecard; best-practices evidence | no |
| `settings-apply.yml`, `epics-changed.yml` | nudge the org's repo-settings and epic lists | no |
| `security-autofix.yml` | the org's security-alert fix agent | no |
| `release.yml` | the release (dispatched by the maintainer; a merged bump PR) | no |

The required checks are the ones the `main` ruleset lists (`gh api
repos/swarm-routing/swarm-adapter-omnet/rules/branches/main`), applied from
`.github/org-sync.toml` `[settings]`.

## Release

Nothing is released yet. `.github/workflows/release.yml` calls the org's
`reusable-release.yml` (Commitizen config `.cz.toml`, version in `VERSION`
and `CITATION.cff`; `make dist` builds the asset, and fails until the module
exists; `vcpkg-ports: swarm-adapter-omnet` bumps the module's port in
swarm-routing/vcpkg-registry after publish). A version is cut on the
maintainer's request only, as
[CONTRIBUTING "Cutting a release"](https://github.com/swarm-routing/.github/blob/main/CONTRIBUTING.md#cutting-a-release)
says, once its row in [roadmap.md](roadmap.md) is met.

## Troubleshooting

- `make check` fails fetching the org tooling: it needs network access to
  github.com.
