# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

**_NOTE:_** PROJECTS BUILT USING THE TEMPLATE SHOULD UPDATE THE BELOW SECTIONS AS-NEEDED.

## [Unreleased]

### Changed
- Specification advanced to v0.8 (Stabilized) for the TG Stabilization
  Milestone vote: `SPEC_STATE.md` and `antora.yml` stamped with the
  template's `scripts/update-spec-state.sh` and `make stamp-antora`.
- Migrated to the dual-source layout from `riscv/docs-spec-template` (#80).
  Chapter prose moved from `src/body.adoc` to one file per chapter under
  `modules/ROOT/pages/`; `src/riscv-performance-event-sampling.adoc` is now a
  PDF assembler that includes those pages. The rendered PDF is unchanged apart
  from one corrected cross-reference.
- Version and ARC phase now derive from `scripts/release-info.sh` and the git
  tag list. PDFs are named `<spec>-v<version>-<YYYYMMDD>.pdf` and carry the
  ARC-required `Document State` preface.
- Releases are cut by pushing a `v*` tag; `workflow_dispatch` is preview-only.

### Added
- Antora component (`antora.yml`, `modules/ROOT/nav.adoc`) and a local preview
  (`npm run preview`, with Kroki via `docker compose`).
- GitHub Pages publishing, plus a per-PR rendered-site artifact.
- `ANTORA.md`, `ARC_SUBMISSION.md` and `UPGRADING.md`.

### Fixed
- Two `<<CSRs>>` cross-references resolved to the Sspesa chapter's CSRs section
  instead of the PDIS one, because the reference was by section title and two
  sections share that title.

## [4.0.0] - 2004-01-27
- Workflow improvements
- Makefile refactoring
- Readme updates
