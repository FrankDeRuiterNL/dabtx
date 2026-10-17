# dabtx — DAB+ Transmitter for HackRF One

A complete DAB+ transmitter written in C/C++ for the
[HackRF One](https://greatscottgadgets.com/hackrf/) SDR transceiver
under Windows 11. Implements the complete signal chain from the
PC audio source to the RF signal:

```
PC audio → HE-AAC v1 → DAB+ Superframe → COFDM Modulator → HackRF TX
```

## Legal Notice

All radio transmissions are subject to the laws and regulations of the
respective country. The user is **solely responsible** for ensuring that
operation complies with the applicable regulations. In particular:

- **Frequency allocation**: Transmit only on frequencies for which a
  valid authorization exists (e.g. an Amateur Radio Class A license
  in Germany).
- **Band plan**: Follow the current
  [IARU Region 1 Band Plan](https://www.iaru-r1.org/reference/band-plans/)
  or the applicable national band plan (Germany:
  [DARC band plans](https://www.darc.de/der-club/referate/vus/bandplaene/)).
- **Transmit power**: Use the minimum power necessary; use a dummy load
  or attenuator.
- **Identification**: Observe any callsign identification requirements.

**Disclaimer of Liability**: The author accepts no liability for
damage or legal consequences arising from the use of this software.
Use is at your own risk and responsibility.

## Features

- HE-AAC v1 (SBR) encoder via libfdk-aac (DAB+ fork)
- Up to 2 audio services in the multiplex
- DLS (Dynamic Label Segment) scrolling text
- MOT SlideShow (JPEG/PNG images, max. 32 KB)
- EPG programme schedule via DLS rotation and SPI (TS 102 818)
- WASAPI loopback for bit-perfect audio capture on Windows
- 47 configurable channels (including amateur radio bands from 70 cm to 6 cm)
- Configuration file for ensemble/service/DLS/EPG
- Signal handling (Ctrl-C) for clean shutdown

## Quick Start

### Requirements

- Windows 11 (x64)
- [MSYS2](https://www.msys2.org/) with the UCRT64 toolchain
- [CMake](https://cmake.org/) 3.20+
- [HackRF One](https://greatscottgadgets.com/hackrf/) with
  the WinUSB driver (via [Zadig](https://zadig.akeo.ie))

### Install MSYS2 Packages

```bash
pacman -S mingw-w64-ucrt-x86_64-gcc \
          mingw-w64-ucrt-x86_64-ninja \
          mingw-w64-ucrt-x86_64-pkg-config \
          mingw-w64-ucrt-x86_64-hackrf \
          mingw-w64-ucrt-x86_64-fftw \
          mingw-w64-ucrt-x86_64-portaudio \
          autoconf automake libtool make
```

### Build

```bash
# Environment
export PATH="/c/msys64/ucrt64/bin:/c/Program Files/CMake/bin:$PATH"
export PKG_CONFIG_PATH="/c/msys64/ucrt64/lib/pkgconfig"
export CC=gcc CXX=g++

# Download and build fdk-aac-dabplus (once)
./setup.sh

# Build dabtx
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build

# Tests
ctest --test-dir build
```

**Note**: The MSYS2 system package `fdk-aac` does not support the
DAB+-specific granule length of 960. Therefore, the
[Opendigitalradio fork](https://github.com/Opendigitalradio/fdk-aac)
is downloaded and built automatically by `setup.sh`.

When using `git clone --recursive`, the fork is cloned as a Git submodule.
When downloading a ZIP archive from GitHub, `setup.sh` downloads it afterward.

### Transmit

```bash
# List audio devices
./build/src/dabtx.exe --list-audio

# Transmit on 23 cm using audio device 31, for 120 seconds, at 35 dB
./build/src/dabtx.exe --tx ham-23a 31 120 35

# Transmit using a configuration file
./build/src/dabtx.exe --config afu.cfg --tx ham-13a 31 300 35

# Offline test (IQ file for qt-dab)
./build/src/dabtx.exe --tx-file ham-23a 31 60 test.raw
```

**Important**: Audio must be actively playing on the selected audio output
device — WASAPI loopback captures only active audio.

## Configuration File

See `afu.cfg.example` for an example. Copy it and customize it:

```bash
cp afu.cfg.example afu.cfg
# Edit callsign, QTH, DLS text, etc.
```

Configuration keys: `ensemble`, `service`, `dls`, `slide`, `epg.1`..`epg.20`

EPG format: `epg.1 = 14:00 | 15:00 | Programme title`

## Amateur Radio Channels

| Channel | Frequency | Band | Suitability |
|---------|-----------|------|-------------|
| ham-70a | 434.500 MHz | 70 cm | Too wide for a 1 MHz window |
| ham-23a | 1298.000 MHz | 23 cm | Good (DATV segment) |
| ham-23b | 1299.500 MHz | 23 cm | Good |
| ham-13a | 2345.000 MHz | 13 cm | Good (DATV segment) |
| ham-13b | 2395.000 MHz | 13 cm | Good |
| ham-6a | 5760.000 MHz | 6 cm | Possible (HackRF max. ~6 GHz) |

## Documentation

- `doc/manual.md` — Complete user manual
- `doc/manual-afu.md` — Instructions for amateur radio operation

Generate PDF/HTML:

```bash
pandoc doc/manual.md -o doc/manual.pdf --pdf-engine=xelatex
pandoc doc/manual-afu.md -o doc/manual-afu.pdf --pdf-engine=xelatex
```

## Technical Specifications

| Parameter | Value |
|-----------|-------|
| Modulation | COFDM Mode I, DQPSK, 1536 carriers |
| Bandwidth | ~1.536 MHz |
| Sample rate | 2.048 MS/s (IQ, complex float) |
| Audio codec | HE-AAC v1 (SBR), 72 kbps |
| Error protection | RS(120,110), EEP-3A |
| OFDM frame | 96 ms (Null + Ref + 76 symbols) |
| IFFT | 2048-point (FFTW, float) |

## Dependencies

See [THIRD_PARTY.md](THIRD_PARTY.md) for complete license and source information.

| Library | Purpose | License |
|---------|---------|---------|
| [libhackrf](https://github.com/greatscottgadgets/hackrf) | HackRF USB driver / TX | GPL-2.0+ |
| [FFTW3](https://www.fftw.org/) | 2048-point IFFT | GPL-2.0+ |
| [fdk-aac (ODR fork)](https://github.com/Opendigitalradio/fdk-aac) | HE-AAC v1 encoder | FDK License |
| [PortAudio](http://www.portaudio.com/) | Audio capture | MIT |
| [libfec](https://github.com/quiet/libfec) | Reed-Solomon RS(120,110) | LGPL-2.1 |

## Credits

### Hardware

This project would not be possible without the
**[HackRF One](https://greatscottgadgets.com/hackrf/)** from
**[Great Scott Gadgets](https://greatscottgadgets.com/)**, designed
by **Michael Ossmann**. It is a brilliant piece of open-source hardware
that makes SDR experimentation accessible to everyone — from reception
to transmission, from 1 MHz to 6 GHz. Both the hardware and the software
(libhackrf, hackrf-tools) are available under open licenses.

### Software References

The DAB+ signal processing is based on studying the
**[Opendigitalradio](https://www.opendigitalradio.org/)** projects:

- **[ODR-DabMux](https://github.com/Opendigitalradio/ODR-DabMux)** —
  FIG byte layouts, ETI frame structure, subchannel configuration
- **[ODR-DabMod](https://github.com/Opendigitalradio/ODR-DabMod)** —
  convolutional encoder, puncturing, frequency interleaver, OFDM
- **[fdk-aac (ODR fork)](https://github.com/Opendigitalradio/fdk-aac)** —
  HE-AAC encoder with DAB+ TT_DABPLUS transport

Debugging and verification were performed using
**[qt-dab](https://github.com/JvanKatwijk/qt-dab)** by Jan van Katwijk.

### Development

This project was conceived by **Stefan Kammann (DM2TIM)** and developed
with extensive assistance from
**[Claude Code](https://claude.ai/code)** by
[Anthropic](https://www.anthropic.com/).

The complete software implementation — from DSP signal processing and
DAB+ protocol encoding through the COFDM modulator to HackRF integration —
was written by Claude Code (Claude Opus), guided by Stefan's project
vision, architectural decisions, hardware testing, and expertise in
amateur radio and DAB+ standards.

Development included, among other things:

- Complete DAB+ transmission chain (audio → HE-AAC → superframe → OFDM → IQ)
- Reed-Solomon encoding, convolutional encoder, frequency interleaving
- FIC/FIG generation according to ETSI EN 300 401
- MOT SlideShow and SPI/EPG implementation
- WASAPI loopback audio backend (Windows COM API)
- Extensive debugging against qt-dab as the reference receiver

## License

This project is licensed under the
**GNU General Public License v3.0 or later** — see
[LICENSE](LICENSE).

Vendored libfec files are subject to LGPL-2.1 — see the copyright
headers in `third_party/fec/`.

## ETSI Standards

- EN 300 401 — DAB System (COFDM, MUX, FIC/MSC)
- TS 102 563 — DAB+ Audio (Superframe, Reed-Solomon, HE-AAC)
- TS 101 499 — MOT SlideShow
- TS 102 818 — Service and Programme Information (SPI/EPG)
