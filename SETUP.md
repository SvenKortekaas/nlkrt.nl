# NL-UTG-KRT-REP01 — setup gids

Een complete handleiding voor het deployen van de website plus de operationele
notes voor de twee nodes (repeater + observer).

> Deze opzet is **100% statisch**. Geen backend, geen API, geen Home Assistant.
> Alle live data wordt door de observer naar Cornmeister gepubliceerd, en de
> website linkt daar naartoe.

---

## Architectuur in één oogopslag

```
[ SenseCAP P1-Pro ]                [ Heltec V4.3 OLED ]
   NL-UTG-KRT-REP01                   NL-KRT-OBS01
   (repeater · dak · solar)           (observer · indoor · USB)
        │                                  │
        │  ◄─── LoRa mesh (869.618 MHz · SF7) ───►
        │                                  │
        │                                  │ MQTT (WiFi → internet)
        │                                  ▼
        │                          ┌──────────────────┐
        │                          │  cornmeister.nl  │
        │                          │  (live monitor)  │
        │                          └──────────────────┘
        │                                  ▲
        │                                  │ link uit website
        ▼                                  │
  ┌─────────────────────────┐              │
  │  nlkrt.nl               │ ─────────────┘
  │  (statische website)    │
  └─────────────────────────┘
```

De repeater doet het echte werk (relay van mesh-pakketten).
De observer luistert passief mee en publiceert metadata naar Cornmeister.
De website (`nlkrt.nl`) is een statische landingspagina met deep-links naar
de Cornmeister-views van beide nodes.

---

## Bestanden in deze bundel

| Bestand | Bestemming |
|---|---|
| `index.html` | Cloud86 → publieke webroot |
| `favicon.svg` | Cloud86 → publieke webroot |
| `favicon.ico` | Cloud86 → publieke webroot |
| `apple-touch-icon.png` | Cloud86 → publieke webroot |
| `SETUP.md` | dit document |

> Eerdere bundels bevatten ook `ingest.php`, `api-htaccess.txt`,
> `ha-configuration.yaml` en `ha-secrets-template.yaml`. Deze zijn
> **niet meer nodig** — de site is volledig statisch geworden.

---

## Deel A · Repeater (P1-Pro)

### A.1 Hardware

- SenseCAP Solar Node P1-Pro (nRF52840 + SX1262)
- LongAP 3 dBi · 868 MHz antenne (SMA)
- Locatie: dak, ~5 m boven maaiveld, verticaal
- Naam: `NL-UTG-KRT-REP01`
- Public key: `89115054326394FC1A0BB981B96688C5411AE9E30885DE5424536828A7E3632C`

### A.2 Firmware & radio settings

Zorg dat de repeater op MeshCore **v1.15 of nieuwer** draait. Dit is verplicht
voor The Switch (regio scoping & strict region forwarding).

Radio settings (vanaf 9 mei 2026):

| Parameter | Waarde |
|---|---|
| Preset | `Netherlands` |
| Spreading factor | SF7 |
| Coding rate | CR5 |
| Frequentie | 869.618 MHz |
| Bandbreedte | 62.5 kHz |
| TX power | +22 dBm |

### A.3 The Switch — voorbereiding

CLI op de repeater:

```
set path.hash.mode 1        # multi-byte path hash
set loop.detect minimal     # loop detectie aan
set dutycycle 10            # 10% ETSI duty cycle limit
```

Advert intervals:
- flood adverts: minimaal **50 uur**
- zero-hop adverts: **240 minuten** (4 uur)

### A.4 Region scoping

Volledige region tree zoals deze repeater geconfigureerd is:

```
region put eu
region put nl
region put bx
region put europe
region put nl-nh nl
region put nl-nh-utg nl-haa
region allowf eu
region allowf nl
region allowf bx
region allowf europe
region allowf nl-nh
region allowf nl-nh-utg
region home nl-nh-utg
region save

# Pas vanaf 13 juni 2026 (Fase 8 — strict region forwarding):
# region denyf *
# region save
```

Let op: `nl-haa` is in deze opzet alleen geregistreerd als parent-knoop van
`nl-nh-utg` (via `region put nl-nh-utg nl-haa`) — er wordt geen `allowf` op
gezet, dus de repeater accepteert alleen flood traffic vanaf de zes scopes
hierboven.

Voor het samenstellen van je eigen region-config kun je de Region Configurator
gebruiken die op de website (sectie 07) is ingebed.

---

## Deel B · Observer (Heltec V4.3)

### B.1 Hardware

- Heltec WiFi LoRa 32 V4.3 OLED (ESP32-S3 + SX1262)
- Stock 2 dBi rubber antenne
- Locatie: indoor, op bureau bij router, USB-C voeding
- Naam: `NL-KRT-OBS01`
- Public key: `63F3CBC39A83C96C4CF25C3D6EFE15AA0FC22876E63CDED7ED9E2DF83515FA1C`
- Rol: **Room server / observer** (geen relay)

### B.2 Configuratie

De observer:
- Luistert passief op het mesh (zelfde radio settings als repeater)
- Stuurt zelf geen mesh-traffic (geen relay, geen flood adverts)
- Publiceert metadata via WiFi → MQTT naar `cornmeister.nl`
- Vereist een actieve WiFi-verbinding

> **Belangrijk:** een observer mag niet als reguliere repeater of companion
> ingezet worden — de bedoeling is alleen monitoring. Op switchdag moeten ook
> de observer settings naar SF7/CR5 omgezet worden anders mist hij alle traffic.

### B.3 Cornmeister deep-links

Beide nodes zijn live op Cornmeister te volgen:

- Repeater: `https://cornmeister.nl/#/nodes/89115054326394fc1a0bb981b96688c5411ae9e30885de5424536828a7e3632c`
- Observer: `https://cornmeister.nl/#/observers/63F3CBC39A83C96C4CF25C3D6EFE15AA0FC22876E63CDED7ED9E2DF83515FA1C`

---

## Deel C · Website deployen

### C.1 Cloud86

Je hebt nodig:
- Cloud86 hosting account met domein `nlkrt.nl`
- FTP/SFTP toegang of File Manager via Plesk

### C.2 Upload

Via FTP/SFTP of Plesk File Manager naar `public_html/`:

```
public_html/
├── index.html
├── favicon.svg
├── favicon.ico
└── apple-touch-icon.png
```

Klaar. De site is 100% statisch — geen PHP, geen database, geen `.htaccess`
nodig.

### C.3 Customisatie

Alle waardes (node-naam, pubkeys, locatie, switch-data) zitten bovenin de
`<script>` tag in `index.html` in het `window.MESH_CONFIG` object:

```javascript
window.MESH_CONFIG = {
  nodeName:        'NL-UTG-KRT-REP01',
  pubkeyRepeater:  '89115054...',
  cornmeisterRep:  'https://cornmeister.nl/#/nodes/...',

  observerName:    'NL-KRT-OBS01',
  pubkeyObserver:  '63F3CBC3...',
  cornmeisterObs:  'https://cornmeister.nl/#/observers/...',

  operator:        'NL-KRT-SVN-01',
  location:        'Noord-Holland - NL - ~Uitgeest',
  geo:             '52.51N, 4.71E +/- 2 km',
  altitudeText:    '5 m - dakmontage',
  sinceISO:        '2026-04-14T08:00:00Z',

  switchDate:      '2026-05-09T00:00:00+02:00',  // SF7 switchdag
  strictDate:      '2026-06-13T00:00:00+02:00',  // strict region forwarding
};
```

De rest van de site is gewoon HTML — pubkeys, scope-codes, hardware-specs en
teksten zitten direct in de body en kunnen daar aangepast worden.

### C.4 Test

Na upload, open `https://nlkrt.nl/` en controleer:

- [ ] Topbar toont `NL-UTG-KRT-REP01` en de "LIVE OP CORNMEISTER" link werkt
- [ ] Hero countdown laat dagen-tot-switch zien
- [ ] Sectie 03 (LIVE) toont beide nodes met klikbare Cornmeister deep-links
- [ ] Favicon zichtbaar in browsertab (eventueel hard refresh nodig)

---

## Deel D · Onderhoud

### D.1 Na 9 mei 2026 (switchdag)

Op switchdag zelf:
1. Schakel de repeater over naar de Netherlands preset (SF7/CR5)
2. Schakel de observer ook over (anders mist hij de mesh-feed)
3. Verifieer op `cornmeister.nl` dat beide nodes weer packets ontvangen

De website-countdown toont vanaf 9 mei automatisch een groen vinkje met
"switch live" - geen handmatige update nodig.

### D.2 Na 13 juni 2026 (Fase 8 - strict region)

Voer op de repeater uit:

```
region denyf *
region save
```

De website-countdown voor strict region toont vanaf die dag ook "strict live".

### D.3 Updaten

Voor latere wijzigingen - andere antenne, ander hardware, een derde node -
update `MESH_CONFIG` bovenin `index.html`, plus de bijbehorende stukken HTML
(identity-card, hardware-sectie). Re-upload alleen `index.html`.

---

## Bronnen

- The Switch (NL): https://meshwiki.nl/wiki/The_switch
- Settings hub (NL): https://settings.woodwar.com/nl/
- Mesh-health monitor: https://mc-radar.woodwar.com/mesh-health
- Region Configurator NL: https://github.com/Joehoehoepi/MeshCore-Region-Configurator-NL
- MeshCore docs: https://docs.meshcore.io
- Cornmeister: https://cornmeister.nl
