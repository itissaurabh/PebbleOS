# Repository Structure

This document provides a detailed breakdown of the PebbleOS repository organization, explaining the purpose of each directory and its key contents.

## Top-Level Directory Overview

```
PebbleOS/
├── src/                    # Main source code
├── platform/               # Platform-specific bootloaders
├── tools/                  # Build tools and utilities
├── tests/                  # Unit tests and test infrastructure
├── third_party/            # External dependencies
├── sdk/                    # Software Development Kit
├── applib-targets/         # Emscripten/SDL test targets
├── docs/                   # Documentation
├── resources/              # Media and localization assets
├── stored_apps/            # Pre-packaged applications
├── bin/                    # Binary output directories
├── checkers/               # Static analysis checkers
├── waftools/               # Custom WAF build tools
├── python_libs/            # Python support libraries
└── .github/workflows/      # CI/CD pipelines
```

## Source Code (`src/`)

### Firmware Core (`src/fw/`)

The heart of PebbleOS containing the kernel, drivers, and application framework.

```
src/fw/
├── main.c                      # Firmware entry point
├── freertos_application.c      # FreeRTOS initialization
│
├── kernel/                     # Operating System Kernel (~248 files)
│   ├── core_dump.c             # Core dump handling
│   ├── event_loop.c            # Main event loop
│   ├── events.c                # Event queue management
│   ├── fault_handling.c        # Exception handling
│   ├── pebble_tasks.c          # Task management
│   ├── pbl_malloc.c            # Memory allocation
│   ├── memory_layout.c         # Memory management
│   ├── logging.c               # Debug logging
│   └── ui/                     # Kernel UI (modals, icons)
│
├── drivers/                    # Hardware Drivers (~360 files)
│   ├── display/                # LCD/OLED display
│   ├── battery/                # Battery management
│   ├── imu/                    # IMU sensors (LIS2DW12, LSM6DSO)
│   ├── hrm/                    # Heart rate monitor
│   ├── flash/                  # Flash memory
│   ├── touch/                  # Touchscreen
│   ├── backlight.c             # Display backlight
│   ├── button.c                # Button input
│   ├── vibe.h                  # Vibration motor
│   ├── ambient_light.h         # Ambient light sensor
│   ├── temperature/            # Temperature sensor
│   ├── nrf5/                   # Nordic nRF5 specific
│   ├── stm32f*/                # STM32 specific
│   └── sf32lb52/               # Sifli specific
│
├── applib/                     # Application Library
│   ├── app_*.c                 # App services
│   ├── accel_service.c         # Accelerometer API
│   ├── health_service.c        # Health tracking API
│   │
│   ├── graphics/               # Graphics Engine (~745 files)
│   │   ├── graphics.c          # Core drawing functions
│   │   ├── gbitmap.c           # Bitmap handling
│   │   ├── gpath.c             # Path rendering
│   │   ├── text_layout.c       # Text rendering
│   │   └── 1_bit/, 8_bit/      # Color depth variants
│   │
│   ├── ui/                     # UI Framework (~940 files)
│   │   ├── window.c            # Window management
│   │   ├── window_stack.c      # Window navigation
│   │   ├── layer.c             # Layer hierarchy
│   │   ├── animation.c         # Animation system
│   │   ├── menu_layer.c        # Menu widgets
│   │   ├── text_layer.c        # Text display
│   │   └── dialogs/            # Dialog boxes
│   │
│   ├── app_message/            # Inter-app messaging
│   ├── rockyjs/                # JavaScript support
│   └── bluetooth/              # BLE APIs
│
├── apps/                       # System Applications
│   ├── core_apps/              # Core OS apps
│   └── system_apps/            # Built-in apps (~12 subdirs)
│       ├── launcher/           # App launcher
│       ├── music_app.c         # Music control
│       ├── notifications_app.c # Notifications
│       ├── alarms/             # Alarms
│       ├── health/             # Health tracking
│       ├── settings/           # System settings
│       ├── weather/            # Weather display
│       ├── timeline/           # Timeline view
│       └── workout/            # Workout tracking
│
├── comm/                       # Communication Layer (~43 files)
│   ├── ble/                    # BLE protocol
│   ├── bluetooth_analytics.c   # BT analytics
│   └── bt_conn_mgr.h           # Connection manager
│
├── board/                      # Board Definitions (~60 files)
│   ├── board.h                 # Common definitions
│   ├── board_nrf5.h            # Nordic boards
│   ├── board_stm32.h           # STM32 boards
│   ├── board_sf32lb52.h        # Sifli boards
│   └── displays/               # Display configs
│
├── console/                    # Debug Console
│   ├── dbgserial.h             # Serial debug
│   └── pulse.h                 # Pulse protocol
│
├── services/                   # System Services
│   └── common/
│       ├── compositor/         # Graphics compositor
│       ├── clock.h             # Clock service
│       ├── new_timer/          # Timer service
│       └── analytics/          # Analytics service
│
└── process_management/         # Process Lifecycle
    ├── app_manager.c           # App launch/stop
    └── process_loader.h        # Binary loading
```

### Bluetooth Firmware (`src/bluetooth-fw/`)

Separate firmware for Bluetooth radio management.

```
src/bluetooth-fw/
├── nimble/                 # NimBLE implementation
├── qemu/                   # QEMU emulation support
└── stub/                   # Stub implementations
```

### Libraries

```
src/libos/                  # OS Abstraction Layer
├── mutex_freertos.c        # FreeRTOS mutex
├── platform.c              # Platform abstraction
├── tick.c                  # Clock tick handling
└── mcu/                    # MCU-specific code

src/libc/                   # Custom C Library
├── string/                 # String functions
├── math/                   # Math library
├── vsprintf.c              # Printf implementation
└── setjmp.c                # Stack unwinding

src/libutil/                # Utility Library (~88 files)
├── heap.c                  # Heap allocator
├── circular_buffer.c       # Ring buffer
├── list.c                  # Linked list
├── crc32.c                 # CRC calculation
└── uuid.c                  # UUID handling

src/libbtutil/              # Bluetooth Utilities
```

### Interface Definitions (`src/idl/`)

Protocol definitions for service communication.

```
src/idl/
└── nanopb/                 # Protocol Buffer definitions
    ├── activity.proto
    ├── event.proto
    └── measurements.proto
```

## Platform-Specific Code (`platform/`)

Bootloader configurations for different hardware platforms.

| Platform | Directory | MCU | Watch Models |
|----------|-----------|-----|--------------|
| Tintin | `tintin/` | STM32 | Original Pebble (bb2, v1.5, v2.0) |
| Snowy | `snowy/` | STM32/nRF | Snowy display variants |
| Silk | `silk/` | nRF5840 | Nordic-based |
| Asterix | `asterix/` | nRF5840 | Nordic-based |
| Robert | `robert/` | Various | Multiple variants |

Each platform directory contains:
```
platform/<name>/
├── boot/                   # Bootloader source
├── wscript                 # Platform build config
└── config/                 # Platform configuration
```

## Tools and Build Utilities (`tools/`)

Over 100 scripts and utilities for development.

```
tools/
├── Image Tools
│   ├── bitmapgen.py        # Bitmap generation
│   ├── png2pblpng.py       # PNG conversion
│   └── pbi2png.py          # PBI to PNG
│
├── Analysis Tools
│   ├── analyze_coredump.py # Core dump analysis
│   ├── analyze_fw_static_memory_usage.py
│   └── analyze_mcu_flash_usage_treemap.py
│
├── Packaging
│   ├── pbpack.py           # Resource packaging
│   ├── mkbundle.py         # Bundle creation
│   └── merge_pbz.py        # PBZ merging
│
├── Device Tools
│   ├── bootloader_test.py  # Bootloader testing
│   ├── readcore.py         # Core dump reader
│   └── pulse_flash_imaging.py
│
├── Subdirectories
│   ├── activity/           # Activity tracking tools
│   ├── commander/          # Device communication
│   ├── font/               # Font tools
│   ├── gdb_scripts/        # GDB support
│   ├── log_hashing/        # Log compression
│   ├── power_profiling/    # Power analysis
│   ├── qemu/               # QEMU tools
│   └── resources/          # Resource processing
```

## Tests (`tests/`)

Comprehensive test infrastructure.

```
tests/
├── fw/                     # Firmware unit tests (17 subdirs)
│   ├── applib/
│   ├── drivers/
│   ├── kernel/
│   └── services/
│
├── libc/                   # C library tests
├── libutil/                # Utility library tests
│
├── fakes/                  # Mock objects
├── fixtures/               # Test data
├── overrides/              # Test-specific overrides
├── stubs/                  # Test stubs
├── test_images/            # Image test data
├── test_includes/          # Test headers
└── test_infra/             # Testing infrastructure
```

## Third-Party Dependencies (`third_party/`)

External libraries and components.

| Component | Purpose |
|-----------|---------|
| `freertos/` | Real-time operating system kernel |
| `nimble/` | Bluetooth Low Energy stack |
| `jerryscript/` | JavaScript engine |
| `hal_sifli/` | Sifli MCU HAL |
| `hal_stm32/` | STM32 MCU HAL |
| `hal_nordic/` | Nordic nRF HAL |
| `cmsis_core/` | ARM CMSIS headers |
| `hal_lis2dw12/` | Accelerometer driver |
| `hal_lsm6dso/` | IMU driver |
| `memfault/` | Crash analytics |
| `nanopb/` | Protocol Buffers |
| `qr_code_generator/` | QR encoding |
| `speex/` | Audio codec |
| `tinymt/` | Random number generator |

## SDK (`sdk/`)

Software Development Kit for app developers.

```
sdk/
├── Doxyfile-SDK.template   # API docs template
├── wscript                 # SDK build config
├── pebble_app.ld.template  # Linker script template
├── sdk_package.json        # SDK metadata
│
├── include/                # Public headers
├── defaults/               # Default project templates
│   ├── rocky/              # RockyJS template
│   ├── app/                # C app template
│   └── lib/                # Library template
│
├── docs/                   # SDK documentation
├── tests/                  # SDK tests
└── tools/                  # SDK tools
```

## Resources (`resources/`)

Media and localization assets.

```
resources/
├── normal/                 # Standard resolution (12 subdirs)
│   ├── icons/
│   ├── images/
│   └── fonts/
├── prf/                    # Profile-specific (8 subdirs)
└── common/                 # Shared resources
```

## Documentation (`docs/`)

Project documentation.

```
docs/
├── getting_started.md      # Setup guide
├── index.md                # Documentation index
│
├── boards/                 # Board-specific docs
│   └── asterix/
├── development/            # Development guides
│   ├── options.md          # Build options
│   ├── qemu.md             # QEMU emulation
│   └── prf.md              # PRF profiles
├── reference/              # Reference docs
└── legacy/                 # Legacy documentation
```

## Build Configuration Files

| File | Purpose |
|------|---------|
| `wscript` | Main WAF build configuration |
| `.clang-format` | C code formatting rules |
| `.gdbinit` | GDB debugger setup |
| `.readthedocs.yaml` | ReadTheDocs build config |
| `crowdin.yml` | Localization workflow |
| `Doxyfile` | Doxygen API docs |
| `requirements.txt` | Python dependencies |
| `LICENSE` | Apache 2.0 license |

## CI/CD Workflows (`.github/workflows/`)

| Workflow | Purpose |
|----------|---------|
| `build-firmware.yml` | Compile firmware for all boards |
| `build-bootloader.yml` | Build bootloaders |
| `build-prf.yml` | Build profile firmware |
| `build-qemu.yml` | Build QEMU images |
| `test.yml` | Run unit tests |
| `compliance.yml` | License/code compliance |
| `release.yml` | Release distribution |

## Key File Locations Summary

| What You Need | Where to Find It |
|---------------|------------------|
| Firmware entry point | `src/fw/main.c` |
| Event loop | `src/fw/kernel/event_loop.c` |
| Task management | `src/fw/kernel/pebble_tasks.c` |
| Display driver | `src/fw/drivers/display/` |
| Window management | `src/fw/applib/ui/window.c` |
| Bluetooth stack | `src/fw/comm/ble/` |
| System apps | `src/fw/apps/system_apps/` |
| Board configs | `src/fw/board/` |
| Build config | `wscript` |
| Unit tests | `tests/fw/` |
| Platform bootloaders | `platform/` |
