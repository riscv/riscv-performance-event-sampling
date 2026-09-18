# Antora Migration Guide

How this repository produces **both** the ARC-compliant submission PDF and the
Antora static-site HTML from a **single source tree**, and where that migration
currently stands.

## TL;DR for authors

- Chapter content lives once, in `modules/ROOT/pages/*.adoc`, as standalone
  Antora pages (each starts with a level-0 `= Title`).
- `src/riscv-performance-event-sampling.adoc` is a thin **PDF assembler**: it includes those pages
  with `leveloffset=+1` so their level-0 titles become chapters in the PDF,
  reproducing the historical section numbering.
- Build the PDF: `make` → `build/<short>-v<ver>-<YYYYMMDD>.pdf`.
- Build the site locally: `antora antora-playbook.yml` → `build/site/`.

## The dual-source technique (why it works)

The PDF wants one master document; Antora wants one file per navigable page.
`leveloffset=+1` reconciles them:

```
modules/ROOT/pages/intro.adoc        src/riscv-performance-event-sampling.adoc (PDF assembler)
-----------------------------        -----------------------------------
= Introduction        (level 0)  ->  include::...intro.adoc[leveloffset=+1]
== Sub Section        (level 1)      => renders as "== Introduction" (Ch.1)
                                        and "=== Sub Section" (1.1)
```

So the pages are the single content source; the assembler is PDF-only glue.
PDF-only constructs stay in the assembler and never appear in a page:
- `include::../docs-resources/global-config.adoc[]` (cross-repo relative
  include — would break under Antora, so it lives only here)
- the back-of-book `[index] == Index` macro (Antora generates no index)
- `:title-logo-image:`, `:pdf-theme:`, `:pdf-fontsdir:`, `:doctype: book`, etc.

## Repository layout

```
antora.yml                       # component descriptor (keep MINIMAL — see below)
antora-playbook.yml              # LOCAL preview playbook (mirrors production UI)
modules/ROOT/
  nav.adoc                       # site navigation
  pages/                         # single source of chapter content (Antora pages)
    index.adoc                   #   site landing page (Antora start_page; NOT in PDF)
    intro.adoc  sspesa.adoc  ssplcofi.adoc  spdis.adoc
    contributors.adoc  bibliography.adoc
  resources/example.bib          # bibliography database
src/riscv-performance-event-sampling.adoc             # PDF assembler (Makefile DOCS target)
Makefile                         # asciidoctor-pdf/html via Docker; ARC PDF naming
docs-resources/                  # submodule: PDF fonts/themes/logo + global-config
```

## Modes: spec vs. doc

Everything above (the PDF, the site, the dual-source technique) is common to
every repo built from this template. What's *not* common is ratification:
this template also assumes, by default, that every consumer is a
specification headed for ARC ratification, and bundles that assumption in as
a "ratification layer" — the Document State preface, the phase/milestone
attributes, `SPEC_STATE.md` tracking.

A repo that is documentation rather than a spec (e.g. `riscv/docs-dev-guide`)
needs the Antora-ready layer above but not the ratification layer. It opts
out by committing a `.docmode` file at the repo root containing `doc` (absent,
empty, or any other value ⇒ `spec`, today's behavior, byte-for-byte). Version
identity — semver git tags + build-date stamping — is identical in both modes;
only the phase/milestone surface changes:

- `scripts/release-info.sh mode` resolves `.docmode` and is the single source
  of truth every other consumer reads it from (never re-parse the file
  yourself). In `doc` mode, `phase`, `phase-floor-version`, `display`,
  `milestone`, `notice`, and `revremark` all resolve to `""` — `version` and
  the rest of the version-arithmetic commands are unaffected.
- The Makefile passes `-a doc-mode='doc'` (instead of the phase/milestone `-a`
  flags) when in doc mode, which `src/riscv-performance-event-sampling.adoc` and
  `modules/ROOT/pages/index.adoc` key their `ifdef::doc-mode[]` /
  `ifndef::doc-mode[]` guards on to drop the Document State preface, the
  title-page revremark, and the cover-page phase banner entirely.
- `scripts/stamp-antora-version.sh` requires only `version`/`revnumber`/
  `revdate` in `antora.yml` in doc mode (not `display`/`notice`/`phase`) — see
  "Version stamping" below.
- `.github/workflows/version-bot.yml`'s `milestone-pr` job (which regenerates
  `SPEC_STATE.md`) only runs in spec mode.

See `MIGRATION.md`'s "Doc mode" section for how to adopt this in a derived
repo.

## Production model (important)

The canonical site is built **elsewhere**, by the central playbook at
`github.com/riscv-admin/antora.riscv.org` (dev mirror:
`github.com/riscv-admin/antora-dev.riscv.org`). This repo is just a **content
source** consumed by that playbook.

The central playbook supplies, uniformly to every spec:
- **Extensions**: `asciidoctor-kroki` (diagrams, incl. wavedrom),
  `@djencks/asciidoctor-mathjax` (math), an ASAM extension for
  `cite:`/`bibliography::[]`, plus section/nav numbering extensions.
- **Shared AsciiDoc attributes**: `doctype: book`, `icons: font`, `xrefstyle`,
  `source-highlighter: highlight.js`, kroki config, math entities, etc.
- **UI**: `github.com/riscv-admin/riscv-antora-only-ui` release bundle.

Consequence — **keep `antora.yml` minimal** (name/title/version/nav). Component
attributes override the playbook, so setting rendering attributes here (notably
`sectnums`, which the central section-numbering extension controls) desyncs this
spec from the rest of the library. Component *identity/grouping* keys are fine;
rendering attributes are not. Put preview-only rendering config in
`antora-playbook.yml` instead.

### Registering this spec on the dev site

Add to `content.sources:` in the dev-site `antora-playbook.yml` (push the branch
first — Antora fetches from GitHub, not the worktree):

```yaml
  # component: riscv-performance-event-sampling
  - url: https://github.com/riscv/riscv-performance-event-sampling.git
    branches: main
    start_page: ROOT::index.adoc
    start_path: /
```

No `nav:` key needed (declared in `antora.yml`). Renders at `/riscv-performance-event-sampling/`.

## Publishing to GitHub Pages

Separate from the central site, this repo also publishes a **standalone copy** of
the same content to its own GitHub Pages site, next to the release PDF. For a
repo seeded from this template that is not part of the RISC-V central library,
this is the primary HTML output; for specs that *are* consumed centrally,
`docs.riscv.org` stays canonical and this is a convenience.

- Workflow: `.github/workflows/publish-site.yml` — runs on `v*` tag pushes and on
  manual dispatch, and deploys via `actions/deploy-pages`.
- Build: `scripts/build-pages-site.sh` — the whole build, runnable locally.
- Setup in a seeded repo: **one manual step**, best done before the first push
  to `main` — the workflow runs on every push to `main`, not only on tags, so
  until Pages exists each push leaves a failed run behind.
  `actions/configure-pages` is configured with `enablement: true` and will turn
  Pages on by itself wherever it is allowed to, but the workflow's `GITHUB_TOKEN`
  is refused with `Resource not accessible by integration`: creating a Pages site
  is an admin-level operation that the Actions app cannot perform on a repo that
  has none, regardless of the `pages: write` grant the job holds. This is a
  limitation of the Actions token, **not** an organization policy — `riscv` has
  `members_can_create_pages: true`, so there is no org setting to change and no
  point hunting for one. A repository admin must set *Settings → Pages → Source*
  to "GitHub Actions" once. Equivalent API call, with an admin token:
  `gh api -X POST repos/<org>/<repo>/pages -f build_type=workflow`. Once the
  site exists, `enablement: true` is a no-op and releases publish unattended.
  The job skips itself on private repos, where Pages needs Team/Enterprise.
- Set that source **directly** to "GitHub Actions". If a repo is ever left on
  "Deploy from a branch" first, GitHub queues a run of its built-in
  `pages-build-deployment` workflow that survives the switch, and it can deploy
  *after* the first Actions deployment — overwriting the site with the repository
  root, which has no `index.html`. The symptom is a live 404 from a
  `publish-site.yml` run that reported success; check the `github-pages`
  deployment list for a `pages-build-deployment` entry newer than yours. Re-run
  `publish-site.yml` once the stray build has finished; it does not recur.

### How the playbook is derived

The Pages build does **not** get its own committed playbook. `antora-playbook.yml`
carries a hand-maintained mirror of the central playbook's rendering attributes,
and a second copy of that block would silently drift. Instead
`scripts/gen-pages-playbook.js` loads it and overrides four things into a
generated, gitignored `antora-pages-playbook.yml`:

| Key | Why |
| --- | --- |
| `site.url` | Project Pages sites live at `https://<org>.github.io/<repo>/`, not at a domain root; Antora needs the base path for the sitemap and canonical links. |
| `content.sources` | Which versions to publish (below). |
| `output.dir` | `build/pages-site`, so a publish never clobbers the `build/site` preview output. |
| `cover-logo` | Resolves the cover logo locally instead of from `common::`. |

### The cover logo

`index.adoc` renders the logo through the `cover-logo` attribute rather than a
literal resource ID. It defaults in `antora.yml` to `common::risc-v_logo.svg`,
the shared asset the central build uses. A single-repo build has no `common`
component, so the Pages build stages the copy from the `docs-resources` submodule
into `modules/ROOT/images/` (gitignored) and hard-sets the attribute to it. The
central build is unaffected, and the local-preview behaviour is unchanged — a
bare `npm run preview` still logs the one expected unresolved `common::` image.

`validate-content-source.yml` stages the logo the same way, for a different
reason: it publishes its render as a PR preview artifact, and `index.adoc` is the
start page, so an unresolved cover would be the first thing a reviewer sees. A
side effect is that the gate now actually *validates* the image macro, which it
could not while it was merely exception-listing the error. The exception stays,
but only covers the degraded path where the submodule is unavailable — there it
keeps a missing submodule a warning rather than a red gate.

### Versions, and why only one is published today

### Untagged builds publish as `dev`

A build with no release tag to name it — the first manual publish in a freshly
seeded repo, or any `workflow_dispatch` before the first tag — gets
`release-info.sh`'s dev version, `vX.YY-<sha>-<date>`. That string is published
under a short fixed label (`dev`, override with `DEV_VERSION_LABEL`) instead of
verbatim, for two reasons:

1. **It breaks the navigation.** The RISC-V UI floats the version selector
   beside the nav tree (`.version-box{float:right}` plus
   `.nav-version-group{overflow:hidden}`, which establishes a block formatting
   context), so the nav only gets the width left over next to the selector, and
   the selector is as wide as its longest version string. At 22 characters the
   nav collapses to about three characters per line. Release tags are short
   (`vX.Y`) and render correctly, so this never affects a real release.
2. **It churns URLs.** Every dispatch would otherwise mint a
   `/riscv-performance-event-sampling/<sha>/` path that the next publish orphans, since each deploy
   replaces the whole site. `/riscv-performance-event-sampling/dev/` stays linkable.

The full dev version is not lost: it is still stamped into `page-revnumber` and
shown on the cover page, so a published dev site names the exact commit it came
from.

### Version stamping

The version stamp is applied to the **working tree** at build time from
`scripts/release-info.sh` — the same source the PDF uses — so the published
version matches the PDF even though the tagged commit's committed `antora.yml`
has not been stamped yet. Nothing is committed back; getting the stamp onto
`main` stays `build-pdf.yml`'s job, which opens a review PR for it.

`scripts/stamp-antora-version.sh` asserts each key it stamps was found in
`antora.yml` exactly once — see "Modes" above. In spec mode that's all six of
`version`/`page-revnumber`/`page-revdate`/`page-phase-display`/
`page-phase-notice`/`page-phase`; in doc mode, whose `antora.yml` carries no
`page-phase*` keys, it's only the first three.

The build also scans release tags and publishes any that carry a correctly
stamped descriptor, which is what would give the site a multi-version dropdown. A
tag qualifies only if it contains an `antora.yml` whose `version:` equals the tag
itself. That is deliberately strict, for two reasons:

1. Antora **aborts the entire build** on any ref that has no `antora.yml`, so
   tags predating the Antora layout must be filtered out.
2. A tag stamped with a different version would publish under a wrong or
   duplicate version path.

**Today no tag qualifies**, so the site publishes exactly one version. The reason
is ordering: a release tag is pushed *first*, and `build-pdf.yml` stamps
`antora.yml` on `main` *afterwards* — so a tag never contains its own version.
If the release flow is ever changed to stamp and commit **before** cutting the
tag, past releases begin appearing in the version dropdown automatically, with no
change to this script.

## Section numbering

Chapter/section numbering is applied by the **central playbook**, not this repo.
The `nav_numbering` and `section_numbering` extensions read a per-component rule
from the playbook's `numbering_rules` anchor. That is *why* `antora.yml` sets no
`sectnums` (design rule under "Production model"): the central extension owns it,
and a local override would desync this spec.

The important subtlety: a rule's `chapters: {start, end}` are **line numbers in
`modules/ROOT/nav.adoc`**, not chapter numbers. The extension scans nav lines;
each line in `[start, end]` matching `^*+ xref:…[` becomes a chapter, numbered
sequentially from 1. This spec's nav xrefs currently sit at:

| `nav.adoc` line | page | numbering |
|---|---|---|
| 13 | `index.adoc` | landing/cover — not a chapter |
| 14 | `contributors.adoc` | unnumbered (front matter) |
| 15 | `intro.adoc` | **Chapter 1** |
| 16 | `sspesa.adoc` | **Chapter 2** |
| 17 | `ssplcofi.adoc` | **Chapter 3** |
| 18 | `spdis.adoc` | **Chapter 4** |

so the entry to add to `numbering_rules` in the central playbook
(`riscv-admin/antora-dev.riscv.org`, `antora/antora-playbook.yml`) is:

```yaml
- component: riscv-performance-event-sampling
  module: ROOT
  branches: ['main']             # match the content-source branch/tag
  chapters: {start: 15, end: 18} # nav lines 15-18 = intro, sspesa, ssplcofi, spdis
```

One entry covers both extensions (they share the `&numbering_rules` anchor).
This numbering matches the PDF, where contributors is a `[preface]` and the four
chapters are numbered 1-4.

> ⚠ **This rule is line-coupled to `nav.adoc`.** Adding, removing, or reordering
> entries — or editing the nav header comments — shifts the line numbers, so the
> central rule must be updated in lockstep or site numbering silently drifts.
> Also update `branches:`/`tags:` to match wherever the spec is consumed.

## Migration status

This repository was migrated to the dual-source layout from
`riscv/docs-spec-template` (tracked in issue #80). What landed, in order:

- **ARC PDF compliance.** `scripts/release-info.sh` as the version/phase source
  of truth; ARC-compliant PDF filenames written by AsciiDoctor itself; the
  `Document State` preface and explicit `toc::[]` placement.
- **Dual-source layout.** `src/body.adoc` split into `sspesa.adoc`,
  `ssplcofi.adoc` and `spdis.adoc` under `modules/ROOT/pages/`, with
  `src/riscv-performance-event-sampling.adoc` reduced to a PDF assembler. The
  rendered PDF was verified identical to the pre-split build apart from one
  corrected cross-reference.
- **Antora component.** `antora.yml`, `nav.adoc`, the site landing page, and
  local preview tooling.
- **Version bridge, CI gate and Pages publishing.** `stamp-antora-version.sh`
  plus the release-time stamp PR; `validate-content-source.yml`; and
  `publish-site.yml`.

Known gaps, deliberately left for a decision rather than guessed at:

- `<<ambendis>>` (two uses) and `<<lambi-handling>>` (one use) in `spdis.adoc`
  have no matching anchors anywhere in the repository. They render as broken
  references on the site and need an author decision -- the surrounding text
  suggests they may belong to the Self-hosted Trace Specification this document
  already cites.
- `bibliography.adoc` is not included by the assembler and is absent from the
  nav, matching its absence from the PDF. The specification contains no `cite:`
  macros; `asamBibliography` is wired so that adding one works without further
  configuration.
- Releases are cut by pushing a `v*` tag. `version-bot.yml`, which restores a
  dispatch-driven release with a monotonic guard and automated milestone PRs,
  has not been adopted yet.
- The first release under the two-digit scheme must be named explicitly. The
  repository's five existing tags are three-component semver, which
  `release-info.sh` does not parse, so `latest` falls back to `v0.0`. Cut
  `v0.61` deliberately; `latest`/`next` resolve correctly from then on.
