# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static landing page for the **NL-UTG-KRT-REP01** MeshCore solar-powered LoRa repeater in the Netherlands. The entire site is a single `index.html` file — no build process, no framework, no dependencies.

## No Build or Test Commands

There are no npm scripts, Makefile targets, linters, or test suites. This is a direct-deploy static site.

**Local development**: open `index.html` in a browser directly.

**Deployment**: push to `main` — GitHub Actions deploys via FTPS to Cloud86 hosting. Credentials are stored in GitHub Secrets (`SFTP_HOST`, `SFTP_USER`, `SFTP_PASS`, `SFTP_PORT`). See `.github/workflows/deploy.yml`.

The deploy workflow explicitly excludes: `.git*`, `.github/`, `SETUP.md`, `README.md`, `CLAUDE.md`.

## Files

| File | Purpose |
|---|---|
| `index.html` | The entire site |
| `favicon.svg` / `favicon.ico` / `apple-touch-icon.png` | Icons (all deployed) |
| `.htaccess` | Gzip compression via `mod_deflate` |
| `robots.txt` | Blocks AI training crawlers |
| `SETUP.md` | Operational doc for repeater + observer hardware (not deployed to server) |

## Architecture

`index.html` is a single file (~2,300 lines) with embedded CSS and minimal JavaScript.

**Script block** (bottom of file, before `</body>`):
- `STRICT_DATE` — the strict region forwarding deadline (`2026-06-13`). Edit this when the deadline changes. `SWITCH_DATE` no longer exists — the SF7 switch happened 2026-05-09 and its countdown was removed.
- `daysUntil()` — computes days remaining; displays `✓` and relabels the counter once past. Refreshes every 60 seconds.
- Contact form handler — POSTs to Formspree (`https://formspree.io/f/xojrywqb`) with `Accept: application/json`. Resets Cloudflare Turnstile on success.

**External services** (hardcoded in HTML — no env vars):
- Cloudflare Turnstile widget (sitekey `0x4AAAAAADNLUyZbilJ-uyjb`) — anti-spam on the contact form.
- Formspree endpoint above — receives contact form submissions.

**Pending**: NL-UTG-KRT-REP02 is announced in several sections (hardware card, network diagram, footer) but is not yet online. Pubkey placeholder is in the HTML; fill it in once the node is operational.

All CSS is in a `<style>` block. Design tokens are CSS custom properties (`--bg-0`, `--border`, `--ink`, `--accent`, `--rad`, `--rad-lg`). Color palette: dark navy `#0d1117` + MeshCore green `#22c55e`. Fonts: Inter + JetBrains Mono (Google Fonts).

## Key Conventions

- **Edit HTML directly**: node names, pubkeys, Cornmeister URLs, operator info, coordinates — all live as plain text in the HTML. `SETUP.md` section C.3 describes a `window.MESH_CONFIG` object that does not exist in the actual code; ignore it and use find-and-replace instead.
- **Zero dependencies**: no npm, no bundler — keep it that way.
- **Single file**: the whole site is one file so it can be forked cleanly for other node operators.
- **Dutch UI, English technical values**: page language is `nl`, but node names, keys, and technical strings stay in English.
- **Dates in ISO 8601**: all dates stored with explicit timezone offsets; display logic handles conversion.
