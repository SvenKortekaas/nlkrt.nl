# nlkrt.nl

Statische landingspagina voor de **NL-UTG-KRT-REP01** MeshCore solar-powered LoRa repeater in Noord-Holland.

## Stack

- Één `index.html` — geen build-stap, geen framework, geen dependencies
- Embedded CSS (design tokens, dark navy + MeshCore groen)
- Minimale JavaScript: alleen de countdown naar strict region forwarding (13 juni 2026)

## Lokaal draaien

Open `index.html` direct in een browser.

## Deployen

Push naar `main` → GitHub Actions deployt via FTPS naar Cloud86-hosting.  
Credentials staan in GitHub Secrets (`SFTP_HOST`, `SFTP_USER`, `SFTP_PASS`, `SFTP_PORT`).

## Inhoud aanpassen

| Wat                          | Waar                                      |
|------------------------------|-------------------------------------------|
| Node-naam, pubkeys, locatie  | Zoek-en-vervang direct in `index.html`    |
| Cornmeister-links            | Zoek-en-vervang direct in `index.html`    |
| Strict region datum          | `STRICT_DATE` bovenaan het `<script>`-blok |

## Forken

Single-file HTML — fork de repo, vervang node-naam en pubkeys via zoek-en-vervang, pas `STRICT_DATE` aan, en je hebt een eigen repeater-pagina. 100% statisch, geen backend nodig.
