# Parts list

## Electronics

| Qty | Part | Evidence or selection detail |
|---:|---|---|
| 1 | Olimex ESP32-POE-ISO Rev.M with ESP32-WROOM-32E, 4 MB flash | Board and module identified from photos. Flash size was read at commissioning. Check the module shield marking and board revision when sourcing a replacement. |
| 1 | ADS1115 I²C breakout | Photos show the blue module marked ADS1115. The firmware reads channel A0 at address `0x48`. This build supplies the breakout with 3.3 V. The breakout manufacturer is unverified. |
| 1 | Analogue pressure transducer, 0–150 psi, 5 V DC, 0.5–4.5 V output, G1/4 | Installed variant confirmed by the builder against the purchase. `GS07381-05233` is the seller's product code. Manufacturer, body marking, accuracy, pressure reference, overload rating and approval for potable water remain unverified. |
| 2 | 10 kΩ resistors, R1 and R2 | Nominal values. The builder measured both resistors during assembly. For a new build, the assembly design specifies 1% tolerance or better. |
| 3 | 100 nF ceramic capacitors (marking `104`) | The build photo shows three disc capacitors and the assembly design specifies 100 nF at ADS VDD, the sensor's 5 V rail and A0. Capacitance, dielectric and voltage rating of the installed parts were not measured or recorded separately. |
| 1 | Perfboard and three-position screw terminal | Visible in the assembly photos. The terminal carries sensor supply, ground and signal. Label the terminals with their functions. |
| As needed | Pin headers, hookup wire, enclosure, mounting and strain relief | Photos show headers, wires and an enclosure. Exact enclosure model, ingress rating and final cable fittings are unverified. |
| 1 each | 802.3af PoE source and Ethernet cable | The node was commissioned on PoE/Ethernet. Use an active standards-compliant switch or injector compatible with the board. |

A micro-USB data cable and a multimeter are useful for commissioning. Use a suitable local pressure gauge for a comparison. Size the fitting, seal and enclosure for the installation after checking the transducer documentation and the intended medium.

[Olimex Rev.M design files](https://github.com/OLIMEX/ESP32-POE-ISO/tree/master/HARDWARE/ESP32-PoE-ISO-Rev.M) · [Texas Instruments ADS1115 datasheet](https://www.ti.com/lit/ds/symlink/ads1115.pdf)
