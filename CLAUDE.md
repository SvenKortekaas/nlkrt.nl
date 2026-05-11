# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static landing page for the **NL-UTG-KRT-REP01** MeshCore solar-powered LoRa repeater in the Netherlands. The entire site is a single `index.html` file — no build process, no framework, no dependencies.

## No Build or Test Commands

There are no npm scripts, Makefile targets, linters, or test suites. This is a direct-deploy static site.

**Local development**: open `index.html` in a browser directly.

**Deployment**: push to `main` — GitHub Actions deploys via FTPS to Cloud86 hosting. Credentials are stored in GitHub Secrets (`SFTP_HOST`, `SFTP_USER`, `SFTP_PASS`, `SFTP_PORT`). See `.github/workflows/deploy.yml`.

## Architecture

Everything lives in `index.html` (~2,050 lines):

- **`SWITCH_DATE` / `STRICT_DATE`** (near the bottom, top of the `<script>` block) — the two community transition dates. Edit these when deadlines change.
- **`$()`** — shorthand for `getElementById()`.
- **Countdown timers** — `daysUntil()` computes days remaining; displays `✓` once the target date has passed. Timers refresh every 60 seconds; the Amsterdam clock refreshes every 1 second.

All CSS is embedded in a `<style>` block. Design tokens use CSS custom properties (`--bg-0`, `--border`, `--ink`, `--accent`, `--rad`, `--rad-lg`). Color palette: dark navy `#0d1117` + MeshCore green `#22c55e`. Fonts: Inter + JetBrains Mono (Google Fonts).

## Key Conventions

- **Edit HTML directly**: node names, pubkeys, Cornmeister URLs, operator info, coordinates — all live as plain text in the HTML. Find and replace to update.
- **Zero dependencies**: no npm, no bundler — keep it that way.
- **Single file**: the whole site is one file so it can be forked cleanly for other node operators.
- **Dutch UI, English technical values**: page language is `nl`, but node names, keys, and technical strings stay in English.
- **Dates in ISO 8601**: all dates stored with explicit timezone offsets; display logic handles conversion.
