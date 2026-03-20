# 🔌 Relay Actuation with STM32 NUCLEO-F103RB

![STM32](https://img.shields.io/badge/STM32-NUCLEO--F103RB-blue?style=flat-square&logo=stmicroelectronics)
![HAL](https://img.shields.io/badge/HAL-STM32CubeIDE-green?style=flat-square)
![Language](https://img.shields.io/badge/Language-C-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Working-brightgreen?style=flat-square)

---

## 📋 Project Overview

This project demonstrates **GPIO-based relay actuation** using the onboard push button (B1) on the STM32 NUCLEO-F103RB development board. When the user button is pressed, a connected relay module is energized via an output pin. The relay is **active-low**, meaning it activates when the control pin is driven **LOW**.

This is a foundational embedded systems project ideal for learning:
- STM32 HAL GPIO read/write
- Active-low input and output handling
- CubeMX pin configuration and code generation
- Flashing and debugging with ST-LINK

---

## 🛠️ Hardware Requirements

| Component | Details |
|---|---|
| **MCU Board** | STM32 NUCLEO-F103RB |
| **Microcontroller** | STM32F103RB (Cortex-M3, 72 MHz, 128KB Flash) |
| **Relay Module** | 5V Single Channel Relay (Active-Low) |
| **User Button** | Onboard B1 (PC13, Active-Low) |
| **Relay Control Pin** | PA0 (Arduino header A0) |
| **Power Supply** | USB (5V from NUCLEO 5V pin to relay VCC) |
| **USB Cable** | Mini-USB data cable (not charge-only) |

---

## 📐 Pin Configuration

```
NUCLEO-F103RB
┌────────────────────────────────────┐
│                                    │
│  PC13 ──── Onboard Button (B1)     │  INPUT  │ Active-LOW (LOW = Pressed)
│                                    │
│  PA0  ──── Relay Control Signal    │  OUTPUT │ Active-LOW (LOW = Relay ON)
│                                    │
│  GND  ──── Relay Module GND        │
│  5V   ──── Relay Module VCC        │
└────────────────────────────────────┘
```

### Relay Module Wiring

```
NUCLEO Board          Relay Module
─────────────         ────────────
PA0 (A0)    ────────► IN
GND         ────────► GND
5V          ────────► VCC
```

---

## ⚙️ CubeMX Configuration

Open the `.ioc` file in STM32CubeIDE and verify the following settings:

| Pin | Label | Mode | Pull | Default State |
|---|---|---|---|---|
| PC13 | USER_BUTTON | GPIO_Input | No Pull | HIGH (idle) |
| PA0 | RELAY_OUT | GPIO_Output PP | No Pull | HIGH (relay OFF) |

**Clock:** HSI 8 MHz (Internal oscillator — no crystal required)

---

## 💻 Software & Tools

| Tool | Version |
|---|---|
| STM32CubeIDE | 1.x or later |
| STM32CubeMX | Integrated with CubeIDE |
| STM32CubeProgrammer | v2.22.0 |
| ST-LINK Firmware | V2J47M34 |
| Compiler | arm-none-eabi-gcc |
| Debug Interface | SWD (Serial Wire Debug) |

---

## 📂 Project Structure

```
RELAY ACTUATION WITH STM32/
│
├── Core/
│   ├── Inc/
│   │   ├── main.h                    # Main header, GPIO defines
│   │   ├── stm32f1xx_hal_conf.h      # HAL module enable/disable
│   │   └── stm32f1xx_it.h            # Interrupt handler declarations
│   └── Src/
│       ├── main.c                    # ★ Main application logic
│       ├── stm32f1xx_hal_msp.c       # MCU-specific HAL init
│       ├── stm32f1xx_it.c            # Interrupt handlers
│       └── system_stm32f1xx.c        # System clock init
│
├── Drivers/
│   ├── CMSIS/                        # ARM Cortex-M3 core headers
│   └── STM32F1xx_HAL_Driver/         # STM32 HAL peripheral drivers
│
├── STM32CubeIDE/
│   ├── STM32F103RBTX_FLASH.ld        # Linker script
│   ├── startup_stm32f103rbtx.s       # Startup assembly
│   └── *.launch / *.cfg              # Debug configurations
│
├── RELAY ACTUATION WITH STM32.ioc    # CubeMX project config
├── .gitignore                        # Git ignore rules
└── README.md                         # This file
```

---

## 🔑 Core Logic — `main.c`

The key logic sits inside the `while(1)` loop:

```c
while (1)
{
    /* PC13 goes LOW when button is PRESSED (active-low button) */
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
    {
        /* Button PRESSED → Drive PA0 LOW → Relay ENERGIZED */
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);
    }
    else
    {
        /* Button RELEASED → Drive PA0 HIGH → Relay OFF */
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);
    }

    HAL_Delay(10); /* 10ms debounce delay */
}
```

### Logic Truth Table

| Button State | PC13 Level | PA0 Level | Relay State |
|---|---|---|---|
| Released | HIGH (1) | HIGH (1) | ❌ OFF |
| Pressed | LOW (0) | LOW (0) | ✅ ON |

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/Mpraveenkumar61/RELAY-ACTUATION-STM32.git
cd RELAY-ACTUATION-STM32
```

### 2. Open in STM32CubeIDE

```
File → Open Projects from File System → Select the cloned folder
```

### 3. Build the Project

```
Project → Build Project   (or Ctrl + B)
```

Expected output:
```
Finished building: RELAY ACTUATION WITH STM32.elf
text: 4532  data: 12  bss: 1572
```

### 4. Flash to Board

1. Connect NUCLEO-F103RB via USB
2. Verify CN2 jumpers are fitted (both SWD_CLK and SWD_DIO)
3. `Run → Debug` or press **F11**
4. Press **Resume (F8)** to start execution

### 5. Test

| Action | Expected Result |
|---|---|
| Press blue **USER button (B1)** | Relay clicks ON, LED on relay module lights |
| Release **USER button** | Relay clicks OFF |

---

## 🐛 Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| `No device found on target` | CN2 jumpers missing or bad USB cable | Re-seat both CN2 jumpers; try different cable |
| `Target not responding` after flash | Debugger timing issue | Press RESET button on board after flash |
| Relay always ON or always OFF | Logic inversion | Verify PA0 init state is HIGH (relay OFF default) |
| Build error: implicit declaration | Code placed outside USER CODE sections | Only add code between `/* USER CODE BEGIN */` tags |
| LD1 blinking red fast | ST-LINK firmware outdated | `Help → ST-LINK Upgrade` in CubeIDE |

---

## 🔄 Debug Configuration (Verified Working)

```
Debug Probe    : ST-LINK (ST-LINK GDB Server)
Interface      : SWD
Frequency      : 480 kHz
Reset Type     : Connect Under Reset
Core Clock     : 8.0 MHz
```

---

## 📌 Important Notes for Collaborators

> ⚠️ **CubeIDE USER CODE sections** — Only write your custom code between the `/* USER CODE BEGIN */` and `/* USER CODE END */` comment blocks. Code outside these sections will be **overwritten** when re-generating from CubeMX.

> ⚠️ **Relay power** — The relay module requires 5V VCC. Connect to the NUCLEO's **5V pin** (not 3.3V) for reliable actuation.

> ⚠️ **Active-load on relay contacts** — Ensure inductive loads (motors, solenoids) connected to relay contacts have a proper flyback diode to protect the relay.

> ✅ **Re-generating code** — After any `.ioc` changes in CubeMX, go to `Project → Generate Code`. Your USER CODE sections are preserved automatically.

---

## 👤 Author

**M Praveen Kumar**
- GitHub: [@Mpraveenkumar61](https://github.com/Mpraveenkumar61)
- Email: mpraveen61205@gmail.com

---

## 📄 License

This project is open-source and available for educational and personal use.

---

## 🙏 Acknowledgements

- [STMicroelectronics](https://www.st.com) — STM32 HAL Libraries & CubeMX
- [ARM](https://www.arm.com) — Cortex-M3 CMSIS headers
- STM32CubeIDE Community & Documentation
