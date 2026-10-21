# ADR-0005: The RB fork and the @foundryrb scope

- **adr_status:** Decided
- **Date:** 2026-10-21
- **Deciders:** Renato Batista
- **Applies to:** `vitest-intent`, `playwright-intent`, `starlight-theme`
- **Relates to:** amends [ADR-0003](0003-public-npm-distribution.md) (same registry, new scope and owner); retires the Python half of this repo (ADR-0004 stays upstream)

## Context

Every `foundryRB` repository imports these runtimes and the docs theme under `@psa-department-of-engineering/*`, published by another organization from `PSA-Department-of-Engineering/csd-library`. Two consequences:

- **The fleet carries another organization's name and look.** The theme's palette is PSA Molten (orange accent, Segoe UI and Cascadia), so every RB docs site had to repaint it in an overlay, and whatever the overlay missed (inline code, for one) showed orange.
- **The fleet's toolchain is not ours to change.** A fix, a palette change or a security bump waits on an upstream release we do not control.

## Decision

Fork the npm half of csd-library into `foundryRB/rb-csd-library` and publish it under the `@foundryrb` scope on the public npm registry, keeping ADR-0003's model: public registry, provenance, trusted publishing, no read credential for consumers.

- Package names move from `@psa-department-of-engineering/<pkg>` to `@foundryrb/<pkg>`; the code, the `intent()` API and the version line carry over unchanged.
- `starlight-theme` carries the RB Software Engineering palettes (neutral greys, the green accent, system font stacks) in place of PSA Molten. The Mermaid layer is untouched: it reads only Starlight tokens.
- The Python runtimes (`csd-intent`, `csd-pytest-intent`) are not forked. Their names carry no brand, and they stay consumed from PyPI as published upstream; the fork's CI installs `csd-intent` from PyPI to audit itself.
- Releases run through `foundryRB/rb-ci`'s `library.yml`. `bootstrap-sha` anchors release-please at the fork point, so the upstream history is not proposed as a release.
- The upstream history is kept, and so is the upstream copyright in every LICENSE (MIT).

## Consequences

- Each package's first publish under `@foundryrb` is manual (npm trusted publishing cannot do a first publish); the steps are in the README. Releases after that publish from CI.
- Consumer repositories move to `@foundryrb/*` in their own pull requests once the packages exist on the registry, since their lockfiles must resolve the published tarballs.
- Upstream fixes no longer arrive by a version bump: they are cherry-picked here when wanted.
