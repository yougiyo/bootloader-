# STM32 Custom Bootloader

![Language](https://img.shields.io/badge/language-C-blue.svg)
![Platform](https://img.shields.io/badge/platform-STM32F401RE-orange.svg)
![Board](https://img.shields.io/badge/board-Nucleo--F401RE-green.svg)
![HAL](https://img.shields.io/badge/HAL-none-lightgrey.svg)
![License](https://img.shields.io/badge/license-MIT-yellow.svg)

A custom STM32 bootloader developed from scratch in bare-metal C. The project avoids STM32 HAL and CubeMX, using register-level peripheral access through CMSIS headers and hand-written linker scripts.

## Features

- Custom bootloader with multi-application flash management
- Hand-written linker scripts covering sections, memory regions, and storage layout
- UART-based firmware loading
- Factory App, Default App, and APP1 support
- Self-contained FPU, UART, GPIO, and timebase drivers
- No dependency on the STM32 HAL

## Repository Structure

```text
bootloader-/
├── 0_test/                        # Sandbox and experiments
├── 1_system_drivers-fpu/          # FPU enable/configuration driver
├── 2_system_drivers-uart/         # USART2 UART driver
├── 3_system_drivers-timebase/     # SysTick-based 1 ms timebase
├── 4_system_drivers-gpio/         # GPIO driver for LED and button
├── 5_linkerscript-sections/       # Introduction to linker sections
├── 6_linkerscript-memory/         # Linker memory-region definitions
├── 7_linkerscript-storage/        # Flash storage layout for applications
├── 8_bootloader1/                 # First bootloader implementation
├── 8_bootloader1-app/             # Application used with bootloader v1
├── 9_bootloader2/                 # Improved bootloader implementation
├── 9_bootloader1-app2/            # Second application for bootloader v1
├── 10_final_project-APP1/         # Final user application
├── 10_final_project-Factory-App/  # Factory application used on first boot
├── 10_final_project-default_app/  # Default fallback application
├── 10_final_project_uart_rx/      # UART firmware receiver
├── 11_final_project_bootloader/   # Complete final bootloader
├── chip_headers/CMSIS/            # ARM CMSIS and device headers
├── DUI0553A_cortex_m4_dgug.pdf   # Cortex-M4 generic user guide
├── rm0383-stm32f411xce-advanced-armbased-32bit-mcus-stmicroelectronics.pdf # STM32 reference manual
├── stm32f411re.pdf                # STM32F411RE device documentation
└── um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf # Nucleo-64 board manual
```

## Boot Flow

```text
Power ON
   │
   ▼
Bootloader
   │
   ├── Check UART for new firmware
   │       └── Receive, erase, and program application flash when requested
   │
   ├── Check APP1 validity
   │       └── Jump to APP1 when its vector table is valid
   │
   ├── Check Default App validity
   │       └── Jump to the default application when available
   │
   └── Fall back to Factory App
```

## Flash Memory Layout

| Region       | Address      | Size   |
|--------------|--------------|--------|
| Bootloader   | `0x08000000` | 64 KB  |
| Factory App  | `0x08010000` | 64 KB  |
| Default App  | `0x08020000` | 64 KB  |
| APP1         | `0x08030000` | 320 KB |

> The application linker scripts must place each image at the matching address before programming the device.

## Target Hardware

| Item | Value |
|------|-------|
| Board | Nucleo-F401RE |
| MCU | STM32F401RE |
| Core | Cortex-M4 with FPU |
| Clock | 84 MHz |
| Flash | 512 KB |
| RAM | 96 KB |
| Key pins | PA5 = user LED, PC13 = user button, PA2 = USART2 TX, PA3 = USART2 RX |

## Peripheral Drivers

| Driver | Description |
|--------|-------------|
| FPU | Enables the Cortex-M4 CP10 and CP11 coprocessor access bits in `SCB->CPACR`. |
| UART | Configures USART2 on PA2/PA3 for 115200 baud, 8 data bits, no parity, and 1 stop bit. |
| Timebase | Configures SysTick from the 84 MHz system clock to provide a 1 ms tick. |
| GPIO | Provides register-level control of the PA5 LED and PC13 button. |

## Linker Script Progression

The linker-script examples build the memory model incrementally:

1. `5_linkerscript-sections/` introduces output sections and demonstrates how code and data are collected into the image.
2. `6_linkerscript-memory/` adds explicit FLASH and RAM memory-region definitions and places sections in those regions.
3. `7_linkerscript-storage/` extends the layout for bootloader and application storage, establishing the flash regions used by the multi-image design.

## Getting Started

### Requirements

- `arm-none-eabi-gcc`
- STM32CubeIDE (for project import/debug support; the firmware itself does not use STM32 HAL)
- Nucleo-F401RE board
- Serial terminal configured for 115200 baud

### Clone

```bash
git clone https://github.com/yougiyo/bootloader-.git
cd bootloader-
```

### Build and flash order

Build and flash the images in this order:

1. Flash the bootloader at `0x08000000`.
2. Flash the Factory App at `0x08010000`.
3. Flash the Default App at `0x08020000`.
4. Flash APP1 at `0x08030000`.

Use the linker script associated with each image so that its vector table and code are linked into the intended flash region.

### UART firmware loading

Connect to USART2 through the Nucleo board's Virtual COM Port using PA2/PA3 and a terminal configured for **115200 baud**. The final bootloader checks the UART path for a firmware-loading request, receives the image, erases the required flash sectors, programs the image, and then applies the normal application-selection flow.

## Reference Documents

- `DUI0553A_cortex_m4_dgug.pdf`
- `rm0383-stm32f411xce-advanced-armbased-32bit-mcus-stmicroelectronics.pdf`
- `stm32f411re.pdf`
- `um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf`

## Design Philosophy

- **No HAL:** peripheral access remains explicit and register-level through CMSIS.
- **Hand-written linker scripts:** memory placement is part of the learning and bootloader design.
- **Incremental numbered folders:** each stage records the progression from experiments to the final project.
- **Self-contained repository:** source, CMSIS headers, linker examples, applications, and reference documents live together.

## License

This project is released under the MIT License. See [LICENSE](LICENSE) for the full license text.
