# Contribution Guidelines

As an open-source project, we appreciate and encourage community members to submit patches directly to the project. To maintain a well-organized development environment, we have established standards and methods for submitting changes. This document outlines the process for submitting patches to the project, ensuring that your contribution is swiftly incorporated into the codebase.

# Repository layout

Chapter prose lives under `modules/ROOT/pages/`, one file per chapter:

| File | Chapter |
| --- | --- |
| `modules/ROOT/pages/intro.adoc` | Introduction |
| `modules/ROOT/pages/sspesa.adoc` | Precise Event Sample Attribution (Sspesa) |
| `modules/ROOT/pages/ssplcofi.adoc` | Precise Local Counter Overflow Interrupt (Ssplcofi) |
| `modules/ROOT/pages/spdis.adoc` | Precise Decoded Instruction Sampling (Smpdis/Sspdis) |
| `modules/ROOT/pages/contributors.adoc` | Contributors |

These files are the single source of content. They are published directly as
pages of the HTML site, and `src/riscv-performance-event-sampling.adoc` -- the
PDF assembler -- includes them to build the specification PDF. Edit the pages;
the assembler only carries the document header, the prefaces, and the include
list.

Two conventions follow from a file being both a chapter and a web page:

- **Each page starts with a level-0 title** (`= Chapter Name`), and its
  subsections start at `==`. The assembler includes each page with
  `leveloffset=+1`, which demotes those titles to `==`/`===` in the PDF, so
  the PDF's section structure is unaffected.
- **Cross-references within a chapter** use `<<anchor>>` as before.
  **Cross-references between chapters** need `xref:file.adoc#anchor[]`, which
  the PDF build cannot resolve. Where one is needed, put the reference in an
  attribute with an `ifndef` default (see `sspesa.adoc` for the worked
  example) so both builds render it correctly.

Prefer an explicit `[[anchor]]` over referring to a section by its title.
Two sections in this specification are both called "CSRs", and a by-title
reference to one of them silently resolved to the wrong chapter for some time.

# Building

    make                                  # PDF + HTML into build/
    make VERSION=v0.8 DATE=2026-06-12     # build a specific version

The build runs in a container and initializes the `docs-resources` submodule
if it is missing.

# Licensing

Licensing is crucial for open-source projects, as it guarantees that the software remains available under the conditions specified by the author.

This project employs the Creative Commons Attribution 4.0 International license, which can be found in the LICENSE file within the project's repository.

Licensing defines the rights granted to you as an author by the copyright holder. It is essential for contributors to fully understand and accept these licensing rights. In some cases, the copyright holder may not be the contributor, such as when the contributor is working on behalf of a company.

# Developer Certificate of Origin (DCO)
To uphold licensing criteria and demonstrate good faith, this project mandates adherence to the Developer Certificate of Origin (DCO) process.

The DCO is an attestation appended to every contribution from each author. In the commit message of the contribution (explained in greater detail later in this document), the author adds a Signed-off-by statement, thereby accepting the DCO.

When an author submits a patch, they affirm that they possess the right to submit the patch under the designated license. The DCO agreement is displayed below and at https://developercertificate.org.


Developer's Certificate of Origin 1.1

By making a contribution to this project, I certify that:

(a) The contribution was created in whole or in part by me and I
    have the right to submit it under the open source license
    indicated in the file; or

(b) The contribution is based upon previous work that, to the best
    of my knowledge, is covered under an appropriate open source
    license and I have the right under that license to submit that
    work with modifications, whether created in whole or in part
    by me, under the same open source license (unless I am
    permitted to submit under a different license), as indicated
    in the file; or

(c) The contribution was provided directly to me by some other
    person who certified (a), (b), or (c), and I have not modified
    it.

(d) I understand and agree that this project and the contribution
    are public and that a record of the contribution (including all
    personal information I submit with it, including my sign-off) is
    maintained indefinitely and may be redistributed consistent with
    this project or the open source license(s) involved.

# DCO Sign-Off Methods
The DCO necessitates the inclusion of a sign-off message in the following format for each commit within the pull request:

Signed-off-by: Stephano Cetola <scetola@linuxfoundation.org>

Please use your real name in the sign-off message.

You can manually add the DCO text to your commit body or include either -s or --signoff in your standard Git commit commands. If you forget to incorporate the sign-off, you can also amend a previous commit with the sign-off by executing git commit --amend -s. If you have already pushed your changes to GitHub, you will need to force push your branch afterward using git push -f.

Note:

Ensure that the name and email address associated with your GitHub account match the name and email address in the Signed-off-by line of your commit message.
