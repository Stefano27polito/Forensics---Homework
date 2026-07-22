# Laboratory 5 — Cloud Forensics

Computer Forensics and Cyber Crime Analysis (01GZDUV-01GZDUW) — Politecnico di Torino, AA 2025/26
Prof. Andrea Atzeni

**Status:** First draft, submitted to the professor for preliminary review. Answer boxes are
intentionally left empty (to be filled in by students); a version with model answers can be
prepared separately on request.

## What this is

A hands-on laboratory on cloud forensics, following on from Laboratories 2–4 (filesystem,
incident investigation, memory forensics). The scenario ("Operation SILENT BUCKET") continues the
NovaDynamics S.p.A. case used in previous laboratories, this time involving a suspected data
exfiltration from a cloud object storage bucket.

The lab covers: the IaaS/PaaS/SaaS distinction and its impact on evidence availability; hands-on
interaction with an S3-compatible API; interpretation of provider-supplied event logs and their
inherent gaps (management-plane vs. data-plane logging); identification of the authoritative
copy among redundant/replicated cloud objects; cross-validation of findings using independent
tools; and the legal/trust implications of relying on cloud-provider-mediated evidence
(jurisdictional fragmentation, chain of custody for cloud artefacts).

## Repository structure

```
.
├── main.tex                          # Document entry point
├── latex-templates/
│   └── styles/
│       └── polito.sty                # Course template package (title page, headers, colors)
├── sections/
│   ├── section01.tex                 # Purpose and Scenario
│   ├── section02.tex                 # Tool Procurement
│   ├── section03.tex                 # Environment Setup (Kali VM, Docker, MinIO)
│   ├── section04.tex                 # First Contact (hands-on S3 interaction)
│   ├── section05.tex                 # Analysis of Provided Log Material
│   ├── section06.tex                 # Redundancy and Primary Copy Identification
│   ├── section07.tex                 # Cloud Service Model Comparison
│   ├── section08.tex                 # Legal & Trust Module
│   └── section09.tex                 # Wrap-Up
├── artifacts/
│   ├── cloudtrail_events.json         # Synthetic CloudTrail-style event log (contains an
│   │                                  #   intentional data-plane logging gap)
│   ├── s3_object_versions.json        # Synthetic object versioning/metadata export
│   └── data_transfer_metrics.json     # Synthetic independent billing/data-transfer report
├── images/
│   └── polito_logo.png                # PLACEHOLDER — replace with the real course logo
└── main.pdf                           # Compiled preview (regenerate after edits, see below)
```

## Compiling

Requires a standard TeX Live installation with (at least) the following packages: `babel`,
`fontenc`, `fancyhdr`, `hyperref`, `graphicx`, `xcolor`, `tabularx`, `datetime2`, `booktabs`,
`amsmath`, `wrapfig`, `float`, `tcolorbox`, `listings`, `enumitem`, `lmodern`.

```bash
pdflatex main.tex
pdflatex main.tex   # run twice: the first pass resolves the table of contents and
                     # cross-references (e.g. Section \ref{sec:log-analysis})
```

Before compiling, replace `images/polito_logo.png` with the actual course logo, and fill in
`\author{}` in `main.tex` with your name and student ID.

## The "live" hands-on component: why MinIO, not LocalStack

Sections 3–4 have students interact directly with a local S3-compatible bucket via `aws cli`,
running inside the same CAINE14/Kali VM used in previous labs, via Docker.

A real cloud account (AWS/Azure/GCP) was deliberately ruled out: cost, the need for every
student to create their own account, and long-term reliability of a paid third-party service. The
original plan used **LocalStack** for local emulation — but as of March 2026, LocalStack merged
its Community and Pro Docker images into a single image that requires a free account and an auth
token even to start (`Exited (55)`, licensing error), which defeated the "no account, no cost"
design goal. This was discovered by actually running the lab end-to-end in a Kali VM on
VirtualBox, not just by writing the instructions on paper.

**MinIO** was substituted instead: fully open-source, no account required, same S3-compatible
API. The full command sequence (Docker install, MinIO container, `aws cli` configuration, bucket
creation, upload, hash verification, metadata inspection) has been tested end-to-end in a Kali VM
with nested virtualization enabled in VirtualBox.

## The static evidence artifacts

Sections 5–6 use three JSON exports, purpose-built with specific pedagogical traps:

- **`cloudtrail_events.json`** — management-plane events are fully logged, but `GetObject`
  (data-plane) events are absent between two `PutBucketPolicy` calls, simulating disabled
  data-plane logging during the exfiltration window.
- **`data_transfer_metrics.json`** — an independent, billing-derived data-transfer report that
  is *not* affected by the same policy manipulation, and shows an anomalous spike exactly in the
  un-logged window (used to cross-reference/estimate what was exfiltrated).
- **`s3_object_versions.json`** — three object versions, including a cross-region replica whose
  `lastModified` timestamp is *more recent* than the true primary copy, due to replication lag
  rather than an actual content change (the "replication trap" exercise in Section 6).

All three have been validated with the exact `jq` queries used in the lab text; expected outputs
are known and consistent with the intended exercises (including a `grep`-based cross-validation
of the data-plane logging gap in Section 5.3).

## Open items before this becomes final

- [ ] Decide on and prepare a model-answers version, if requested by the professor
- [ ] Replace `\author{}` placeholder and the logo image
- [ ] Incorporate professor's feedback from the preliminary review
