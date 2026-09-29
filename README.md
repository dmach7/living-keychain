# 🔑 Living Keychain

![Status](https://img.shields.io/badge/status-WIP-orange?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)
![Platform](https://img.shields.io/badge/platform-XIAO%20ESP32--S3-green?style=flat-square)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)

> **A tiny living keychain — a Seeed XIAO ESP32-S3, an OLED, and a battery, built around one obsession: the lowest possible power draw.**

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://github.com/user-attachments/assets/a8a71e4d-2ab7-4d04-96e7-5bfe4163a00e">
  <source media="(prefers-color-scheme: light)" srcset="https://github.com/user-attachments/assets/20b72292-3fd7-4f70-aeb3-fc5b6c5dbafd">
  <img alt="Living Keychain banner" width="1200" height="300" src="https://github.com/user-attachments/assets/a8a71e4d-2ab7-4d04-96e7-5bfe4163a00e">
</picture>

---

## 📑 Table of Contents

- [Overview](#overview)
- [Hardware](#-hardware)
  - [Bill of Materials](#bill-of-materials)
- [Display & Animation](#️-display--animation)
- [Power](#-power)
- [PinMode](#pinmode)
- [Configuration](#️-configuration)
- [Dependencies](#-dependencies)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#-license)

---

## Overview

No buttons, no menus, no interaction — the keychain powers on and just **lives**: a pair of animated robot eyes looping on a tiny OLED, blinking and glancing around on their own. Everything about the hardware choice is driven by one constraint: squeeze the most battery life possible out of the smallest possible board.

```
LiPo battery → XIAO ESP32-S3 → I2C → SSD1306 OLED (128×64)
                     └── deep sleep between animation frames
```

---

## 🔋 Hardware

### Bill of Materials

| Component | Description | Qty | Notes |
| :--- | :--- | :---: | :--- |
| Seeed XIAO ESP32-S3 | Main microcontroller | 1 | Chosen specifically for its 14µA deep sleep draw — the lowest in the XIAO ESP32 lineup (vs. 44µA on the ESP32-C3) |
| SSD1306 OLED | 0.96", 128×64, I2C | 1 | OLED pixels only draw current when lit — black background costs nothing |
| LiPo battery | 3.7V, small capacity to match keychain size | 1 | Charged directly through the XIAO's onboard charge IC via USB-C |

That's the whole BOM — no sensors, no buttons, no extra chips. Every part on this list exists purely to display eyes for as long as possible on a single charge.

---

## 🖥️ Display & Animation

The OLED loops a small set of idle animation frames — blinking, glancing side to side, occasionally a longer "sleepy" blink — rendered as simple bitmap eyes on a black background.

> [!NOTE]
> Because it's a monochrome OLED, a black background genuinely draws near-zero current — only the lit eye pixels cost power. This is part of why OLED (not an LCD/TFT) was the right call for a battery-first project like this.

---

## 🔌 Power

The entire project exists to a simple goal: **The longer the battery lasts, the better.**

I am not prioritizing no menu, no configuration AP, i just need a simple keychain that looks cool

- **XIAO ESP32-S3** was picked over the C3/C6 variants specifically for its **14µA deep sleep** current — the lowest Seeed publishes across the XIAO ESP32 family
- Between animation frames, the firmware drops into **light/deep sleep** rather than busy-waiting, waking only on a timer to draw the next frame
- No WiFi/BLE radio is kept active during normal operation — both draw far more current (tens of mA) than the sleep-and-blink loop needs

> [!WARNING]
> Actual runtime depends heavily on how often the animation wakes the chip. A faster blink/glance cycle looks livelier but burns through the battery quicker — this is a tuning knob, not a fixed number.

---

## PinMode

| GPIO | Function | Peripheral | Bus | Notes |
| --- | --- | --- | --- | --- |
| D4 (GPIO5) | SDA | SSD1306 OLED | I2C | |
| D5 (GPIO6) | SCL | SSD1306 OLED | I2C | |
| BAT+ / BAT- | — | LiPo battery | Onboard charge IC | Native XIAO battery header, no external charge circuit needed |

---

## ⚙️ Configuration

```cpp
#define OLED_WIDTH       128
#define OLED_HEIGHT      64
#define OLED_I2C_ADDR    0x3C

#define FRAME_INTERVAL_MS   80    // animation frame pacing while awake
#define SLEEP_BETWEEN_MS    3000  // deep/light sleep duration between animation bursts
```

> [!CAUTION]
> Lowering `SLEEP_BETWEEN_MS` too much trades battery life directly for a livelier animation — there's no free lunch here.

---

## 📦 Dependencies

- [Adafruit SSD1306](https://github.com/adafruit/Adafruit_SSD1306) — OLED driver
- [Adafruit GFX](https://github.com/adafruit/Adafruit-GFX-Library) — graphics primitives for drawing the eyes

> [!IMPORTANT]
> Library versions are not pinned. If a dependency updates and breaks the build, lock versions in your package manager.

---

## Roadmap

> [!NOTE]
> None of the items below are implemented yet — this is a planning list for future versions, not current behavior.

| Milestone | Target | Status |
| :---: | :--- | :---: |
| M1 | Battery percentage estimation via ADC voltage read | 🔲 Planned |
| M2 | More expression variety (happy, sleepy, surprised) in the idle loop | 🔲 Planned |
| M3 | Further deep sleep tuning to squeeze out more runtime per charge | 🔲 Planned |
| M4 | 3D-printed keychain enclosure design | 🔲 Planned |
| M5 | Charging-status indicator on the OLED when plugged into USB-C | 🔲 Planned |

---

## Contributing

Contributions are very welcome — animation frames, power optimizations, enclosure designs, docs, anything.

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/your-feature`
3. Commit your changes: `git commit -m "feat: describe your change"`
4. Push and open a Pull Request against `main`

Please open an issue first for anything larger than a bug fix, so we can discuss direction before you invest time building it.

> [!IMPORTANT]
> When contributing firmware changes, always test battery runtime on real hardware before submitting a PR — sleep/wake timing behaves very differently in simulation than on an actual battery.

---

## 📄 License

This project is licensed under the **MIT License**. See [LICENSE](./LICENSE) for the full text.

---
