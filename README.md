# dabtx — DAB+ Sender fuer HackRF One

Ein vollstaendiger DAB+ Sender in C/C++ fuer den
[HackRF One](https://greatscottgadgets.com/hackrf/) SDR-Transceiver
unter Windows 11. Realisiert die komplette Signalkette von der
PC-Audioquelle bis zum HF-Signal:

```
PC-Audio → HE-AAC v1 → DAB+ Superframe → COFDM Modulator → HackRF TX
```

## Rechtliche Hinweise

Jede Funkaussendung unterliegt den Gesetzen und Vorschriften des
jeweiligen Landes. Der Nutzer ist **eigenverantwortlich** dafuer,
dass der Betrieb im Einklang mit den geltenden Regelungen erfolgt.
Insbesondere:

- **Frequenzzuteilung**: Sendungen nur auf Frequenzen, fuer die
  eine gueltige Zulassung besteht (z.B. Amateurfunkzeugnis Klasse A
  in Deutschland)
- **Bandplan**: Aktuellen
  [IARU Region 1 Bandplan](https://www.iaru-r1.org/reference/band-plans/)
  bzw. nationalen Bandplan (DE:
  [DARC Bandplaene](https://www.darc.de/der-club/referate/vus/bandplaene/))
  beachten
- **Sendeleistung**: Minimale Leistung verwenden, Dummy-Load oder
  Attenuator einsetzen
- **Kennung**: Rufzeichenpflicht beachten

**Haftungsausschluss**: Der Autor uebernimmt keine Haftung fuer
Schaeden oder rechtliche Konsequenzen, die aus der Nutzung dieser
Software entstehen. Die Nutzung erfolgt auf eigenes Risiko und
eigene Verantwortung.

## Features

- HE-AAC v1 (SBR) Encoder via libfdk-aac (DAB+ Fork)
- Bis zu 2 Audio-Services im Multiplex
- DLS (Dynamic Label Segment) Lauftext
- MOT SlideShow (JPEG/PNG Bilder, max 32 KB)
- EPG Programm-Zeitplan via DLS-Rotation und SPI (TS 102 818)
- WASAPI-Loopback fuer bit-perfect Audio-Capture unter Windows
- 47 konfigurierbare Kanaele (inkl. Amateurfunk 70 cm bis 6 cm)
- Konfigurationsdatei fuer Ensemble/Service/DLS/EPG
- Signal-Handling (Ctrl-C) fuer sauberes Shutdown

## Schnellstart

### Voraussetzungen

- Windows 11 (x64)
- [MSYS2](https://www.msys2.org/) mit UCRT64 Toolchain
- [CMake](https://cmake.org/) 3.20+
- [HackRF One](https://greatscottgadgets.com/hackrf/) mit
  WinUSB-Treiber (via [Zadig](https://zadig.akeo.ie))

### MSYS2-Pakete installieren

```bash
pacman -S mingw-w64-ucrt-x86_64-gcc \
          mingw-w64-ucrt-x86_64-ninja \
          mingw-w64-ucrt-x86_64-pkg-config \
          mingw-w64-ucrt-x86_64-hackrf \
          mingw-w64-ucrt-x86_64-fftw \
          mingw-w64-ucrt-x86_64-portaudio \
          autoconf automake libtool make
```

### Bauen

```bash
# Environment
export PATH="/c/msys64/ucrt64/bin:/c/Program Files/CMake/bin:$PATH"
export PKG_CONFIG_PATH="/c/msys64/ucrt64/lib/pkgconfig"
export CC=gcc CXX=g++

# fdk-aac-dabplus herunterladen und bauen (einmalig)
./setup.sh

# dabtx bauen
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Tests
ctest --test-dir build
```

**Hinweis**: Das MSYS2-Systempaket `fdk-aac` unterstuetzt die
DAB+-spezifische Granule-Length 960 nicht. Daher wird der
[Opendigitalradio Fork](https://github.com/Opendigitalradio/fdk-aac)
ueber `setup.sh` automatisch heruntergeladen und gebaut.

Bei `git clone --recursive` wird der Fork als Git-Submodule
mitgeklont. Bei ZIP-Download von GitHub laedt `setup.sh` ihn nach.

### Senden

```bash
# Audio-Geraete auflisten
./build/src/dabtx.exe --list-audio

# Senden auf 23 cm mit Audio-Device 31, 120 Sekunden, 35 dB
./build/src/dabtx.exe --tx ham-23a 31 120 35

# Mit Konfigurationsdatei
./build/src/dabtx.exe --config afu.cfg --tx ham-13a 31 300 35

# Offline-Test (IQ-Datei fuer qt-dab)
./build/src/dabtx.exe --tx-file ham-23a 31 60 test.raw
```

**Wichtig**: Auf dem gewaehlten Audio-Ausgabegeraet muss aktiv
Audio abgespielt werden — WASAPI-Loopback capturt nur aktives Audio.

## Konfigurationsdatei

Beispiel siehe `afu.cfg.example`. Kopieren und anpassen:

```bash
cp afu.cfg.example afu.cfg
# Rufzeichen, QTH, DLS-Text etc. editieren
```

Config-Keys: `ensemble`, `service`, `dls`, `slide`, `epg.1`..`epg.20`

EPG-Format: `epg.1 = 14:00 | 15:00 | Titel der Sendung`

## Amateurfunk-Kanaele

| Kanal | Frequenz | Band | Eignung |
|-------|----------|------|---------|
| ham-70a | 434.500 MHz | 70 cm | Zu breit fuer 1 MHz Fenster |
| ham-23a | 1298.000 MHz | 23 cm | Gut (DATV-Segment) |
| ham-23b | 1299.500 MHz | 23 cm | Gut |
| ham-13a | 2345.000 MHz | 13 cm | Gut (DATV-Segment) |
| ham-13b | 2395.000 MHz | 13 cm | Gut |
| ham-6a | 5760.000 MHz | 6 cm | Moeglich (HackRF max ~6 GHz) |

## Dokumentation

- `doc/manual.md` — Vollstaendige Bedienungsanleitung
- `doc/manual-afu.md` — Anleitung fuer Amateurfunk-Betrieb

PDF/HTML erzeugen:
```bash
pandoc doc/manual.md -o doc/manual.pdf --pdf-engine=xelatex
pandoc doc/manual-afu.md -o doc/manual-afu.pdf --pdf-engine=xelatex
```

## Technische Eckdaten

| Parameter | Wert |
|-----------|------|
| Modulation | COFDM Mode I, DQPSK, 1536 Traeger |
| Bandbreite | ~1.536 MHz |
| Sample-Rate | 2.048 MS/s (IQ, complex float) |
| Audio-Codec | HE-AAC v1 (SBR), 72 kbps |
| Schutzcodierung | RS(120,110), EEP-3A |
| OFDM-Frame | 96 ms (Null + Ref + 76 Symbole) |
| IFFT | 2048-Punkt (FFTW, float) |

## Abhaengigkeiten

Siehe [THIRD_PARTY.md](THIRD_PARTY.md) fuer vollstaendige
Lizenz- und Quellenangaben.

| Bibliothek | Zweck | Lizenz |
|---|---|---|
| [libhackrf](https://github.com/greatscottgadgets/hackrf) | HackRF USB-Treiber / TX | GPL-2.0+ |
| [FFTW3](https://www.fftw.org/) | 2048-Punkt IFFT | GPL-2.0+ |
| [fdk-aac (ODR Fork)](https://github.com/Opendigitalradio/fdk-aac) | HE-AAC v1 Encoder | FDK License |
| [PortAudio](http://www.portaudio.com/) | Audio-Capture | MIT |
| [libfec](https://github.com/quiet/libfec) | Reed-Solomon RS(120,110) | LGPL-2.1 |

## Credits

### Hardware

Dieses Projekt waere nicht moeglich ohne den
**[HackRF One](https://greatscottgadgets.com/hackrf/)** von
**[Great Scott Gadgets](https://greatscottgadgets.com/)**, designed
by **Michael Ossmann**. Ein brillantes Stueck Open-Source-Hardware,
das SDR-Experimente fuer jedermann zugaenglich macht — vom
Empfang bis zum Senden, von 1 MHz bis 6 GHz. Sowohl die Hardware
als auch die Software (libhackrf, hackrf-tools) sind unter offenen
Lizenzen verfuegbar.

### Software-Referenzen

Die DAB+ Signalverarbeitung basiert auf dem Studium der
**[Opendigitalradio](https://www.opendigitalradio.org/)** Projekte:

- **[ODR-DabMux](https://github.com/Opendigitalradio/ODR-DabMux)** —
  FIG-Byte-Layouts, ETI-Frame-Struktur, Subchannel-Konfiguration
- **[ODR-DabMod](https://github.com/Opendigitalradio/ODR-DabMod)** —
  Convolutional Encoder, Puncturing, Frequency-Interleaver, OFDM
- **[fdk-aac (ODR Fork)](https://github.com/Opendigitalradio/fdk-aac)** —
  HE-AAC Encoder mit DAB+ TT_DABPLUS Transport

Debugging und Verifikation erfolgten mit
**[qt-dab](https://github.com/JvanKatwijk/qt-dab)** von Jan van Katwijk.

### Entwicklung

Dieses Projekt wurde von **Stefan Kammann (DM2TIM)** konzipiert und
mit umfassender Unterstuetzung durch
**[Claude Code](https://claude.ai/code)** von
[Anthropic](https://www.anthropic.com/) entwickelt.

Die komplette Software-Implementierung — von der DSP-Signalverarbeitung
und DAB+ Protokoll-Codierung ueber den COFDM-Modulator bis zur
HackRF-Integration — wurde von Claude Code (Claude Opus) geschrieben,
gesteuert durch Stefans Projektvision, Architekturentscheidungen,
Hardware-Tests und Fachkenntnisse in Amateurfunk und DAB+ Standards.

Die Entwicklung umfasste unter anderem:
- Komplette DAB+ Sendekette (Audio → HE-AAC → Superframe → OFDM → IQ)
- Reed-Solomon Codierung, Faltungscodierer, Frequenz-Interleaving
- FIC/FIG Generierung nach ETSI EN 300 401
- MOT SlideShow und SPI/EPG Implementierung
- WASAPI-Loopback Audio-Backend (Windows COM API)
- Umfangreiches Debugging gegen qt-dab als Referenz-Empfaenger

## Lizenz

Dieses Projekt ist lizenziert unter der
**GNU General Public License v3.0 or later** — siehe
[LICENSE](LICENSE).

Vendored libfec-Dateien unterliegen der LGPL-2.1 — siehe
Copyright-Header in `third_party/fec/`.

## ETSI-Standards

- EN 300 401 — DAB System (COFDM, MUX, FIC/MSC)
- TS 102 563 — DAB+ Audio (Superframe, Reed-Solomon, HE-AAC)
- TS 101 499 — MOT SlideShow
- TS 102 818 — Service and Programme Information (SPI/EPG)
