# Hydrofor pressure monitor — setup and commissioning guide

[Project overview](../README.md) · [Polska instrukcja](instrukcja-pl.md) · [Parts list](parts.md) · [Photo gallery, EN/PL](photos/README.md)

This guide is based on the Polish guide and the configuration published in this repository. It covers the documented Olimex ESP32-POE-ISO Rev.M build, from wiring checks to Home Assistant integration and a pressure-gauge comparison.

The module reports the measurement-circuit voltage and water pressure in bar to Home Assistant. Pressure history helps show what happens during water use. **The module provides monitoring only. Pump control and hydraulic protection must operate independently.**

## 1. Hardware and software you need

Use the [parts list](parts.md) to check component identification and the limits of the available evidence. The documented build uses:

- Olimex ESP32-POE-ISO **Rev.M with ESP32-WROOM-32E and 4 MB flash**;
- ADS1115 breakout powered from 3.3 V;
- a 5 V pressure transducer with nominal 0–150 psi range and 0.5–4.5 V output;
- two nominal 10 kΩ resistors and three external 100 nF ceramic capacitors, marked `104`;
- an active, board-compatible 802.3af PoE switch or injector and an Ethernet cable;
- a micro-USB **data** cable, a multimeter and a suitable local pressure gauge;
- ESPHome, either through ESPHome Device Builder or its command-line tools, and Home Assistant with network access to the module.

Read the ESP32 variant from the metal module shield and check the board revision. The `ESP32-WROVER` text on the red PCB is a PCB marking; it does not identify the fitted module in this build. Do not transfer this pin map to another board revision or ESP32 variant without checking its documentation.

The installed transducer variant was confirmed by the builder against the purchase. Its manufacturer datasheet, accuracy, pressure reference, overload rating and suitability for potable-water contact remain unverified. The seller's product code is not a substitute for those specifications. Check your own sensor documentation and identify VCC, GND and OUT before applying power; wire colours and the terminal order in the illustrations are not a pinout guarantee.

## 2. Drawings and module photos

| Drawing | English | Polski |
|---|---|---|
| Wiring schematic | [SVG](hydrofor-schemat-polaczen-en.svg) | [SVG](hydrofor-schemat-polaczen-pl.svg) |
| Olimex Rev.M connector map, A4 | [PDF](hydrofor-mapa-montazowa-en-a4.pdf) | [PDF](hydrofor-mapa-montazowa-pl-a4.pdf) |
| Voltage divider and A0 filter | [SVG](hydrofor-dzielnik-a0-en.svg) | [SVG](hydrofor-dzielnik-a0-pl.svg) |
| Illustrated wiring overview | [SVG](hydrofor-widok-modulu-en.svg) | [SVG](hydrofor-widok-modulu-pl.svg) |

The illustrated overview shows connections by function. Sensor terminal order is illustrative. Use the A4 connector map to locate the Olimex pins. The [five-photo gallery](photos/README.md) shows the module interior, Olimex board, voltage divider, solder side and enclosure with its clear cover screwed in place. Photos complement the wiring schematic; they do not establish every hidden connection.

## 3. Wiring checks with power disconnected

Disconnect USB and PoE before changing connections. For initial electrical checks, leave the sensor disconnected from the water installation and its supply disconnected until the power rails have been checked.

| Signal | Connection in this build |
|---|---|
| +5 V | Olimex EXT1.1 → sensor VCC |
| +3.3 V | Olimex EXT1.2 → ADS1115 VDD |
| Shared ground | Olimex EXT1.3 → ADS1115 GND and sensor GND |
| SDA | Olimex EXT2.10 / GPIO13 → ADS1115 SDA |
| SCL | Olimex EXT2.7 / GPIO16 → ADS1115 SCL |
| I²C address | ADS1115 ADDR → GND, address `0x48` |
| Sensor signal | Sensor OUT → R1 10 kΩ → A0 node / ADS1115 A0 |
| Divider lower leg | A0 node → R2 10 kΩ → shared GND |
| C1ext | 100 nF between ADS1115 VDD, 3.3 V, and GND, near the ADS1115 |
| C2ext | 100 nF between sensor +5 V and GND, near the sensor connector |
| C4ext | 100 nF between A0 and GND, in parallel with R2 |

This build uses the **EXT1/EXT2 side headers**. The UEXT socket is unused. **GPIO17 is the Ethernet clock; do not use it for I²C.** ADS1115 ALRT and inputs A1–A3 are unused in this configuration.

The builder measured both resistors during assembly; their nominal values are 10 kΩ each. Check the values and topology in a new build. R1 is the series resistor from OUT to A0; R2 connects A0 to ground. The three external `104` capacitors are nominally 100 nF and are non-polarised. Do not replace the breakout's fitted SMD capacitors as part of these instructions.

Check continuity against the schematic and look for shorts, loose strands and solder bridges before powering the board. **Do not connect the 0.5–4.5 V sensor output directly to A0.** With the equal-resistor divider, the nominal input range at A0 is 0.25–2.25 V.

## 4. Prepare your local ESPHome configuration

1. Install or open ESPHome using the [official getting-started guide](https://esphome.io/install/getting-started/).
2. Copy [hydrofor-monitor.example.yaml](../esphome/hydrofor-monitor.example.yaml) into your local ESPHome configuration directory as `hydrofor-monitor.yaml`. Use a separate local copy for your own build. Preserve a backup of any existing working configuration before replacing it.
3. Create or edit `secrets.yaml` in the same directory. Add your own ESPHome Native API encryption key:

   ```yaml
   # Placeholder only: replace it locally before validating the configuration.
   hydrofor_api_key: "REPLACE_WITH_YOUR_OWN_32_BYTE_BASE64_KEY"
   ```

   The key must contain 32 random bytes encoded in Base64. The [official Native API page](https://esphome.io/components/api/) provides a browser-generated key. Keep the real key and `secrets.yaml` local; do not commit them or include them in screenshots and shared logs. Use the same key when Home Assistant requests the encryption key.
4. Review the example against your hardware. It uses `esp32dev`, 4 MB flash and the ESP-IDF framework; LAN8720 Ethernet with MDC GPIO23, MDIO GPIO18, clock output GPIO17, PHY address 0 and power GPIO12; I²C SDA GPIO13 and SCL GPIO16 at 100 kHz. It contains no Wi-Fi credentials or fixed network address.
5. Validate the local file in ESPHome Device Builder. With the command-line tools, run this from the directory containing your YAML and `secrets.yaml`:

   ```sh
   esphome config hydrofor-monitor.yaml
   ```

   Resolve validation errors before flashing. Check the indentation, missing secret and hardware variant first. Record the ESPHome version used for your build. This guide does not claim that the example has been compiled or tested against every ESPHome release.

See the [ESPHome command-line reference](https://esphome.io/guides/cli/) for validation, compilation and upload options. The example has no `ota:` component; it does not provide over-the-air firmware updates as published.

## 5. First flash and electrical checks

1. Keep PoE disconnected. Connect the Olimex board to your computer with the micro-USB data cable. Keep the sensor supply disconnected for the initial power-rail checks.
2. In ESPHome Device Builder, install the firmware using the USB/serial method appropriate to your setup. Alternatively, with the command-line tools, use:

   ```sh
   esphome run hydrofor-monitor.yaml
   ```

   Select the board's USB serial port when prompted. This command validates, compiles, uploads and opens the logs. Flash only the intended new or deliberately reconfigured board.
3. Measure the sensor supply rail and ADS1115 supply relative to the shared low-voltage ground. Confirm the nominal 5 V and 3.3 V rails against the wiring table. Stop and correct any wrong supply or unexpected short before connecting the sensor.
4. Check the startup logs for the ADS1115 at I²C address `0x48`. If it is missing, check VDD, common ground, ADDR and SDA/SCL rather than changing pins at random.
5. Disconnect USB power before attaching the sensor to its verified supply, ground and signal terminals through the divider. Keep the sensor out of the water installation at this stage. Check the completed wiring again.
6. Disconnect USB, then connect the Ethernet cable to the compatible PoE source. Confirm that the board starts and joins your local network. The example uses DHCP; find the assigned address in your router or PoE network's client list if discovery does not work.

The USB stage is for initial flashing and electrical checks; the documented operating connection is PoE/Ethernet. Do not change the power arrangement or pin assignments merely to bypass an unexplained failure.

## 6. Add the device to Home Assistant

1. Open **Settings → Devices & services** in Home Assistant and look for the discovered ESPHome device.
2. If discovery is unavailable, add the **ESPHome** integration manually and enter the module's local hostname or assigned address. The default Native API port is `6053`. Home Assistant must be able to reach the device on the local network.
3. Enter your local API encryption key when requested. A key mismatch is an integration problem, not a reason to alter the sensor conversion.
4. Confirm that the device exposes **ADC A0 voltage** in volts and **Water pressure** in bar. Entity IDs are assigned by your Home Assistant installation; do not assume another installation's entity IDs.
5. Check both values and their update history. The example requests an A0 reading every second, using single-ended `A0_GND`, a ±4.096 V PGA range and a 128 samples/s ADC data-rate setting. The ADC data-rate setting does not mean Home Assistant receives 128 readings each second.

The pressure sensor copies the A0 voltage and applies the nominal linear conversion. After an initial sample, its local `timeout: 5s` filter publishes `NaN` if no further input reaches that filter for five seconds, rather than substituting 0 bar. **This filter runs on the ESP32. It cannot send an update when the board has lost power or network access, and it does not guarantee a five-second outage indication in Home Assistant.** Check device availability and data freshness separately. Missing data means pressure is unknown.

See the [Home Assistant ESPHome integration guide](https://www.home-assistant.io/integrations/esphome/) and [ESPHome sensor filters](https://esphome.io/components/sensor/#timeout).

## 7. Pressure conversion and gauge comparison

The nominal sensor output is 0.5–4.5 V over 0–150 psi, approximately 10.342 bar. For the divider:

```text
A0 = sensor OUT × R2 / (R1 + R2)
R1 = R2 → A0 = sensor OUT / 2

P [bar] = (A0 [V] − 0.25) / 2.00 × 10.34213594
        ≈ 5.17106797 × A0 [V] − 1.29276699
```

Here **A0 is the voltage after the divider**. ESPHome uses two nominal endpoints: 0.25 V → 0 bar and 2.25 V → 10.34213594 bar. The 0–150 psi range is not exactly 0–10 bar. A different sensor output, range or resistor ratio requires its own conversion.

To compare your own unit:

1. Before fitting it to a water line, confirm the allowed medium, temperature range and pressure ratings of the sensor, fitting and seals. For potable water, obtain confirmation that all wetted components are suitable. The seller listing alone does not establish this.
2. Use a suitable pressure gauge on the same section of the installation. Record A0 voltage, Home Assistant pressure and the gauge reading at the same time during an ordinary stable operating condition.
3. Record the difference and the gauge's readability. Do not change the conversion merely to make one reading agree. A single comparison does not establish accuracy across the range.
4. Observe normal pressure cycles and data continuity. Keep installation-specific alert thresholds separate from the nominal sensor transfer function. Do not force higher pressures to extend a check beyond the installation's normal safe operating conditions.

In the documented deployment, one point after the conversion change read about **3.00 bar in Home Assistant versus 3.00–3.05 bar on the local gauge**. Accuracy elsewhere in the range remains unestablished. The nominal transfer function is not a full-range calibration certificate.

## 8. Troubleshooting

| Symptom | What to check |
|---|---|
| USB serial port is missing or flashing fails | Use a data-capable micro-USB cable, check the correct port and follow the board's documented programming procedure. Keep PoE disconnected during the USB stage. |
| ADS1115 is missing from the I²C scan | Check 3.3 V supply, common ground, ADDR to GND, and SDA/SCL at both boards. Confirm EXT2.10 / GPIO13 for SDA and EXT2.7 / GPIO16 for SCL. |
| No Ethernet connection | Check the compatible active PoE source, cable, network port, DHCP assignment and LAN8720 settings for this board. GPIO17 is reserved for the Ethernet clock. |
| Home Assistant cannot connect | Check local network reachability, the device's current address and Native API port, and the encryption key. Check the ESPHome logs. |
| A0 is close to zero despite pressure | Check sensor power, OUT continuity and the R1/R2 junction. R2 may pull A0 to ground if the signal wire is open. At nominal 0 bar, A0 should be about 0.25 V. |
| Pressure differs from the gauge | Check the exact sensor variant, resistor values and topology, 5 V supply, and whether readings were taken at the same time on the same section. |
| Readings stop updating | Check PoE, Ethernet, the ESPHome integration, availability and report freshness. Do not interpret a missing or stale reading as 0 bar. |
| Frequent-cycle alert | Compare pressure history with actual water use. In the documented installation, such alerts occurred during ordinary use. Pressure history alone does not measure motor starts. |

## 9. Operating limits and remaining work

- Pump control and hydraulic safeguards remain independent of this monitor.
- One-second readings cannot characterise brief hydraulic transients.
- Pressure cycles alone cannot diagnose a pump, leak or pressure-tank bladder fault. Alert thresholds must be assessed for the installation.
- The exact transducer datasheet and wetted-material suitability remain to be confirmed. Full-range measurement accuracy remains unverified.
- Enclosure model, ingress rating and final cable fittings are not established by the photos. Select protection and strain relief for the installation.

This documentation adds no new physical test, firmware deployment or calibration result. The example configuration and the documented working installation are not changed by this guide.

## Reference documentation

- [ESPHome ADS1115 component](https://esphome.io/components/sensor/ads1115/)
- [ESPHome Ethernet component](https://esphome.io/components/ethernet/)
- [ESPHome Native API and encryption](https://esphome.io/components/api/)
- [ESPHome command-line tools](https://esphome.io/guides/cli/)
- [Home Assistant ESPHome integration](https://www.home-assistant.io/integrations/esphome/)

[Back to the project README](../README.md) · [Polska instrukcja](instrukcja-pl.md)
