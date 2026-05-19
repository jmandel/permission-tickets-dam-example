# SMART Permission Tickets DAM Example

This repository contains a working Domain Analysis Model (DAM) example for
SMART Permission Tickets, based on stakeholder interview synthesis.

## Contents

- `dam.md` - narrative DAM source document
- `dam.html` - standalone formal DAM artifact set with Mermaid visualizations
- `.github/workflows/build.yml` - GitHub Actions workflow that validates and
  packages the standalone HTML as a GitHub Pages-ready static site

## Viewing Locally

Open `dam.html` directly in a browser. The file is standalone except for Mermaid,
which is loaded from the jsDelivr CDN so the diagrams can render.

## GitHub Actions

The build workflow checks that:

- `dam.md` and `dam.html` exist
- the HTML contains the expected DAM title
- the page includes Mermaid from the CDN
- at least eight Mermaid diagram blocks are present
- the narrative Markdown contains the expected DAM heading

On pushes to `main`, the workflow also packages `dam.html` as `index.html` and
uploads it through the GitHub Pages artifact flow.

## Status

Working analysis artifact. This is not an implementation guide or normative
specification.
