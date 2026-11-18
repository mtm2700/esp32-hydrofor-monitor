# ESP32 hydrofor pressure monitor

This DIY monitor records water pressure in a well-pump pressure tank system using an ESP32, an ADS1115 and Home Assistant. The pressure history shows how the system behaves during water use and helps spot sustained pressure changes, missing readings and unusually frequent pressure cycles.

[Polska instrukcja](docs/instrukcja-pl.md) · [Parts list](docs/parts.md) · [ESPHome example](esphome/hydrofor-monitor.example.yaml)

## Wiring and assembly

| Drawing | English | Polski |
|---|---|---|
| Wiring schematic | [SVG](docs/hydrofor-schemat-polaczen-en.svg) | [SVG](docs/hydrofor-schemat-polaczen-pl.svg) |
| Olimex Rev.M connector map, A4 | [PDF](docs/hydrofor-mapa-montazowa-en-a4.pdf) | [PDF](docs/hydrofor-mapa-montazowa-pl-a4.pdf) |
| Voltage divider and A0 filter | [SVG](docs/hydrofor-dzielnik-a0-en.svg) | [SVG](docs/hydrofor-dzielnik-a0-pl.svg) |
| Illustrated wiring overview | [SVG](docs/hydrofor-widok-modulu-en.svg) | [SVG](docs/hydrofor-widok-modulu-pl.svg) |

The overview shows the wiring by function. Sensor terminal order is illustrative. Use the A4 connector map to locate the Olimex pins.

## How it works

1. A 5 V pressure transducer with a nominal 0–150 psi range outputs 0.5–4.5 V.
2. Two nominal 10 kΩ resistors halve the signal to 0.25–2.25 V at A0. The ADS1115 runs from 3.3 V and measures A0 relative to the shared low-voltage ground.
3. An Olimex ESP32-POE-ISO Rev.M with an ESP32-WROOM-32E reads the ADS1115 over I²C (SDA GPIO13, SCL GPIO16, address `0x48`). ESPHome sends A0 voltage and calculated pressure to Home Assistant over Ethernet.

The firmware uses the nominal transfer function `pressure [bar] = 5.17106797 × A0 [V] − 1.29276699`. Here A0 is the voltage **after** the divider. The two nominal endpoints are 0.25 V at 0 bar and 2.25 V at about 10.342 bar. Nominal conversion assumes the specified sensor output and a divider ratio of 0.5. A different sensor or divider needs its own conversion.

## Status and limits

- **10 Nov 2026** — The installed unit was brought online on PoE/Ethernet. Home Assistant received both A0 voltage and pressure.
- **12 Nov 2026** — The firmware switched to the nominal linear conversion. Sixteen sample pairs verified the calculation. At one operating point Home Assistant read about 3.00 bar and the local gauge read about 3.00–3.05 bar. Accuracy across the operating range remains unverified.
- **13–15 Nov 2026** — Home Assistant recorded frequent-pressure-cycle alerts during normal water use. Alert thresholds are installation-specific. Pressure cycles alone cannot establish motor starts or diagnose a pump, leak or bladder fault.

The device provides monitoring only. Use independent pump control and hydraulic protection. One-second pressure readings are unsuitable for measuring brief hydraulic transients. The installed transducer is identified by a seller's listing and confirmation from the builder. A manufacturer datasheet and suitability for potable-water contact remain unverified.

## Documentation still in progress

The Polish guide includes the wiring map, commissioning sequence and troubleshooting. Remaining work includes photographs cleared for sharing, and a reference for the exact transducer and its wetted materials.
