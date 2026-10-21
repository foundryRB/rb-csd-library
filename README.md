# rb-csd-library

RB Software Engineering's fork of [csd-library](https://github.com/PSA-Department-of-Engineering/csd-library) (MIT, Rafael Pires): the npm runtimes and the shared Starlight look the `foundryRB` fleet builds on, published under the `@foundryrb` scope. Why the fork exists and what it changes is recorded in [ADR-0005](docs/adr/0005-rb-fork.md).

Reusable implementation artifacts for projects following CSD methodology. Methodology stays separate (the canonical CSD source, `foundryRB/rb-csd`); this repo is the *implementation* counterpart.

## Layout

```
rb-csd-library/
├── README.md            ← you are here
├── vitest-intent/       ← @foundryrb/vitest-intent: intent() for vitest
├── playwright-intent/   ← @foundryrb/playwright-intent: intent() for Playwright e2e
└── starlight-theme/     ← @foundryrb/starlight-theme: the shared Starlight look (stylesheet + mermaidConfig)
```

The Python runtimes are not forked: `csd-intent` (the cross-runtime `intent.yaml` auditor) and `csd-pytest-intent` (the `@intent(...)` decorator for pytest) carry no brand and are consumed from PyPI as published upstream.

## How the pieces relate

| Piece              | What                                                          | Consumed by |
|--------------------|---------------------------------------------------------------|-------------|
| `vitest-intent`    | TS+vitest runtime: the `intent()` wrapper, same shape as the pytest decorator | Any TS/JS project that wants CSD-style intent testing |
| `playwright-intent`| TS+Playwright runtime: the `intent()` wrapper for browser e2e | Any frontend project doing CSD-style e2e |
| `starlight-theme`  | Shared Starlight look: a token-based stylesheet (the RB Dark/Light palettes + Mermaid contrast layer) plus the `astro-mermaid` `mermaidConfig` object | Any Starlight docs site, via `customCss` + a config import |
| `csd-intent` (PyPI, upstream) | Standalone Python CLI: validates `intent.yaml` and checks every claim is attested across pytest / vitest / Playwright | Every repo's CI |

`csd-intent` finds vitest and Playwright markers by their `intent('INT-...', ...)` call shape, not by the package they are imported from, so a repo moving from the upstream scope to `@foundryrb` keeps every attestation.

## Expected layout on disk

The playbook's skills (`apply-intent-bundle`, `bootstrap-vitest-intent`, `bootstrap-starlight`, ...) reach in through a sibling clone when one exists:

```
<workspace>/
├── rb-playbook/      ← the playbook
└── rb-csd-library/   ← this repo
```

## Releasing

Releases are automated with [release-please](https://github.com/googleapis/release-please) through the org's shared CI (`foundryRB/rb-ci` → `library.yml`). The decision is recorded in [ADR-0002](docs/adr/0002-release-automation.md); distribution in [ADR-0003](docs/adr/0003-public-npm-distribution.md) as amended by [ADR-0005](docs/adr/0005-rb-fork.md).

How a release happens:

1. Land work on `main` with Conventional Commits scoped by package (`feat(vitest-intent): …`) so only that package's version moves.
2. release-please maintains one **release PR per package**, accumulating the version bump + CHANGELOG from those commits.
3. The repo's tests (`.github/workflows/test.yml`) run on that PR. Merge it when you want to cut the release. **Merge by rebase, never squash**: a squash merge appends a `Co-authored-by: github-actions[bot]` trailer, which the org's no-co-author hygiene gate rejects.
4. On merge, release-please tags `<pkg>-vX.Y.Z`, creates the GitHub release, and publishes the released npm packages to the public npm registry with a sigstore provenance attestation. Consumers install with no `.npmrc` and no token.

**First publish.** npm trusted publishing cannot do a package's first publish, so each `@foundryrb/*` package is published once by hand, then handed to CI:

```bash
npm login                                   # an npm account that owns the foundryrb organization
cd vitest-intent     && npm ci && npm publish --access public && cd ..
cd starlight-theme   && npm ci && npm publish --access public && cd ..
cd playwright-intent && npm ci && npm publish --access public && cd ..
```

Then, on npmjs.com, open each package → Settings → Trusted publishing → GitHub Actions: organization `foundryRB`, repository `rb-csd-library`, workflow `release.yml`. From then on every release publishes from CI with no stored credential.

Versioning is per-package SemVer; `release-please-config.json` and `.release-please-manifest.json` hold the package map and current versions (`bootstrap-sha` anchors release-please at the fork point, so the upstream history is not re-released). Two one-time GitHub settings are needed: allow Actions to create pull requests (Settings → Actions → General), and mark the `tests` checks required on `main`.

A committed `.pre-commit-config.yaml` ships a bypassable local `commit-msg` hook that checks each message is a Conventional Commit:

```bash
pip install pre-commit
pre-commit install --install-hooks
```

## License

MIT. The upstream copyright (Rafael Pires) is kept in every LICENSE; the fork's modifications are RB Software Engineering's.
