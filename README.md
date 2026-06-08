# HW-274 / CY7C68013A (EZ-USB FX2LP) — Field Manual

A practical, plain-English manual for the **HW-274** FX2LP breakout board — the Cypress
EZ-USB **CY7C68013A** board commonly sold as a *"24 MHz 8-channel USB logic analyzer"*
or an *"EZ-USB FX2LP development board."* These boards almost always ship **with no
manual**; this fills that gap.

> **Verified against a real HW-274 board** (mini-USB, 24 MHz crystal, marked `HW-274`
> on the lower silkscreen). Other FX2LP clones use the same chip but may relabel or
> reorder header pins — always cross-check your own board's silkscreen.

![The HW-274 board](board-hw-274.png)

A fully styled version with diagrams and tables is in **[`index.html`](index.html)**
(open it in any browser, or host it with GitHub Pages).

---

## 1. What it is

The **CY7C68013A** is a USB 2.0 High-Speed (480 Mbps) microcontroller with an 8051 core
and a fast parallel "GPIF/Slave-FIFO" interface. With no firmware loaded it enumerates as:

```
04b4:8613   Cypress Semiconductor Corp. CY7C68013 EZ-USB FX2 USB 2.0 Development Kit
```

It *becomes* whatever firmware is loaded:

| Firmware | The board becomes… |
|---|---|
| `fx2lafw` (sigrok) | A **logic analyzer** — 8 channels, up to 24 MS/s. Most common use. |
| Custom 8051 code | A USB-to-SPI / I²C / JTAG bridge or general 8051 dev board. |
| None | The raw `04b4:8613` device, awaiting an upload. |

**You do not have to permanently flash anything** to use it as a logic analyzer —
sigrok/PulseView upload `fx2lafw` into RAM on the fly each time you scan.

---

## 2. HW-274 silkscreen pinout

The HW-274 routes the FX2 ports to **two pin headers**, one each side of the chip.
Listed top → bottom with the USB connector at top. Labels marked `(?)` sit partly under
header pins in the reference photo — confirm against your own board.

**Left header** (outer / inner column):

```
outer   inner
PD5     PD6
PD7     GND
CLK     GND
RDY1    RDY0
GND     VCC
GND     IFCLK
SCL     SDA
PB0     PB1
PB2     PB3
GND     VCC
```

**Right header** (inner / outer column):

```
inner   outer
PD3     PD2
PD1     PD0
PD0(?)  PA7
PA6     PA5
PA4     PA3
PA2     PA1
PA0     PB6
PB7     CTL1(?)
CTL0    PB5
PB5(?)  PB4
```

Exposed: Port A (PA0–7), Port B (PB0–7), Port D (PD0–7), I²C (SCL/SDA → EEPROM),
clock/ready (CLK, IFCLK, RDY0/RDY1), GPIF control (CTL0/CTL1), power (VCC = **3.3 V**, GND).
**Port C is not broken out** on this board.

---

## 3. Electrical specs

- I/O is **3.3 V** CMOS — **NOT 5 V tolerant.** Do not feed 5 V into any pin.
- As an `fx2lafw` logic analyzer: **8 channels** (on Port B), up to **24 MS/s** (8-ch).
- No on-board sample memory — streams live over USB.
- Sample at **≥ 4×** (ideally 8–10×) your signal's fastest clock.

---

## 4. Using it as a logic analyzer (Linux)

```bash
sudo apt install sigrok pulseview sigrok-firmware-fx2lafw

# Plug the board in, then:
sigrok-cli --scan
#   expected:  fx2lafw - 8 channels: D0 D1 D2 D3 D4 D5 D6 D7
```

If `--scan` shows `fx2lafw`, open **PulseView**, pick the device, set sample rate/count,
and Run. Add a protocol decoder (SPI, I²C, UART…) to decode the traces.

**Example — sniffing an SPI bus** (channels on Port B):

| FX2 pin | Target signal |
|---|---|
| PB0 | SCLK |
| PB1 | MOSI |
| PB2 | MISO |
| PB3 | CS / SS |
| GND | Target ground — **connect first** |

---

## 5. Safety

- ⚠️ **Not 5 V tolerant** — level-shift/divide anything above 3.3 V.
- **Passive sniffing = inputs only.** Never let the FX2 drive a line another device drives.
- **Ground first, signal second.** Remove signals before ground when finished.
- **Don't back-feed the VCC/3V3 pin** — it's a regulator output, a reference only.

---

## 6. Troubleshooting

| Symptom | Fix |
|---|---|
| Shows as `04b4:8613` but `sigrok-cli --scan` finds nothing | Install `sigrok-firmware-fx2lafw`, replug. |
| Not detected at all | Bad/charge-only USB cable, or permissions — try `sudo` / add a udev rule. |
| Traces all flat or all high | No common ground with target, or wrong pins. |
| Decoder output is garbage | Sample rate too low (≥4× clock) or decoder channel mapping wrong. |
| Want it permanent | Write `fx2lafw` to the I²C EEPROM with `cycfx2prog` / `fxload` (advanced). |

---

## Credits

**Authored by Claude Opus 4.8 (Anthropic).** Hardware photo and board verification by the
repository maintainer. Released for free community use — see [LICENSE](LICENSE).

Tools referenced: [sigrok](https://sigrok.org/) · PulseView · fx2lafw.

## Contributing

If your HW-274 (or another FX2LP clone) has different silkscreen labels, open an issue or
PR with a clear photo — corrections and additional board variants are welcome.
