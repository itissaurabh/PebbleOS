# Programming Languages and Frameworks

This document provides a comprehensive overview of all programming languages, frameworks, libraries, and tools used in the PebbleOS codebase.

## Language Distribution

### Primary Languages

| Language | Files | Purpose |
|----------|-------|---------|
| **C** | ~2,050 | Core firmware, system libraries, platform code |
| **C Headers** | ~1,963 | Public APIs and internal interfaces |
| **JavaScript** | ~1,799 | Watchface/app development, SDK tools, JerryScript bindings |
| **Python** | ~565 | Build tools, firmware utilities, command-line tools |
| **ARM Assembly** | ~4 | Performance-critical code (memcpy, memset optimized) |
| **Shell Scripts** | ~45 | Build automation, deployment scripts |
| **Configuration** | ~135 | XML, YAML, JSON configs |

### Language by Component

```
src/fw/          → C (1,027 files) - Firmware core
src/apps/        → C (26 files) - System applications
src/libos/       → C (7 files) - OS library
tools/           → Python (195 files) - Build/analysis tools
sdk/             → JavaScript, C headers - Developer SDK
third_party/     → C, JavaScript - External libraries
```

## Build System

### Primary: WAF (Waf - The meta build system)

PebbleOS uses [WAF](https://waf.io/), a Python-based build system that provides flexibility for embedded development.

**Configuration Files:**
- Root: `/wscript` - Main build configuration
- Custom tools: `/waftools/` directory

**Key WAF Tools (Custom Extensions):**

| Tool | Purpose |
|------|---------|
| `asm.py` | ARM assembly compilation |
| `protoc.py` | Protocol Buffer compilation |
| `gettext.py` | Internationalization |
| `ldscript.py` | Linker script processing |
| `openocd.py` | JTAG/SWD debugger integration |
| `nrfutil.py` | Nordic chip utilities |
| `sftool.py` | Sifli chip utilities |
| `compress.py` | Binary compression |
| `binary_header.py` | Firmware header injection |
| `pblboot.py` | Bootloader utilities |
| `pebble_test.py` | Testing framework |

### Secondary Build Tools

| Tool | Location | Purpose |
|------|----------|---------|
| CMake | `third_party/` | Third-party library builds |
| Makefiles | `third_party/jerryscript/` | JerryScript compilation |
| Webpack | `tools/webpack/` | JavaScript bundling |

## Frameworks and Libraries

### Real-Time Operating System

**FreeRTOS** - The foundation of PebbleOS task scheduling

- Location: `/third_party/freertos`
- Purpose: Thread management, inter-task communication, timing
- Features used:
  - Task creation with MPU regions
  - Queue-based messaging
  - Semaphores and mutexes
  - Timer services

### JavaScript Runtime

**JerryScript** - Lightweight JavaScript engine for embedded systems

- Location: `/third_party/jerryscript`
- Purpose: Execute JavaScript watchfaces and apps (RockyJS)
- Integration: `/src/fw/applib/rockyjs/`

### Bluetooth Stack

**NimBLE** (Apache Mynewt Nimble)

- Location: `/third_party/nimble`
- Purpose: Bluetooth Low Energy implementation
- Features:
  - GAP (Generic Access Profile)
  - GATT Client/Server
  - L2CAP channels
  - Security Manager

### Hardware Abstraction Layers

| HAL | Location | Target |
|-----|----------|--------|
| ARM CMSIS Core | `third_party/cmsis_core/` | ARM Cortex-M standard |
| Nordic nRF HAL | `third_party/hal_nordic/` | nRF52 series |
| STM32 HAL | `third_party/hal_stm32/` | STM32F2/F4/F7 |
| Sifli HAL | `third_party/hal_sifli/` | SF32LB52 |
| LIS2DW12 HAL | `third_party/hal_lis2dw12/` | Accelerometer |
| LSM6DSO HAL | `third_party/hal_lsm6dso/` | IMU sensor |

### Data Serialization

**Protocol Buffers (nanopb)**

- Location: `/third_party/nanopb`
- Definition files: `/src/idl/nanopb/`
- Proto files:
  - `activity.proto` - Activity tracking data
  - `event.proto` - Event structures
  - `measurements.proto` - Sensor measurements
  - `payload.proto` - Communication payloads

### Other Third-Party Libraries

| Library | Purpose |
|---------|---------|
| speex | Audio codec for voice compression |
| tinymt | Tiny Mersenne Twister PRNG |
| qr_code_generator | QR code encoding |
| memfault | Firmware analytics and crash reporting |
| TI Bluetooth SP | Bluetooth service pack |

## Python Dependencies

### Core Requirements

From `/requirements.txt`:

**Image Processing:**
```
pillow
freetype-py
svg.path
pypng
```

**Serial/Hardware Communication:**
```
pyusb
pyserial
pyftdi==0.56.0
pexpect
```

**Binary Processing:**
```
pyelftools      # ELF file parsing
intelhex        # Intel HEX format
bitarray
```

**Cryptography:**
```
pycryptodome
```

**Protocol & Serialization:**
```
protobuf
grpcio-tools
```

**Development Tools:**
```
libclang        # C/C++ language binding
ply==3.4        # PLY parser generator
pep8            # Code style checker
nose            # Testing framework
mock            # Mocking library
```

### Internal Python Libraries

Located in `/python_libs/`:

| Library | Purpose |
|---------|---------|
| pbl | Watch interaction (requires libpebble2) |
| pblprog | Firmware programming |
| pblconvert | Format conversion |
| pulse2 | Device communication protocol |
| pebble-loghash | Log hash generation |
| pebble-commander | Command-line device control |

## JavaScript/Node.js Ecosystem

### Package Files

- `/sdk/defaults/rocky/package.json` - RockyJS watchface template
- `/sdk/defaults/app/package.json` - Standard app template
- `/sdk/defaults/lib/package.json` - Library template
- `/third_party/jerryscript/js_tooling/package.json` - JS compiler tools

### JavaScript Frameworks

**RockyJS** - Pebble's optimized JS framework
- Purpose: Efficient rendering for low-power displays
- Features: Canvas API, Pebble-specific extensions

**PKJs** (PebbleKit JavaScript)
- Purpose: Bridge between C and JavaScript code
- Communication with phone companion apps

### TypeScript Support

- Type definitions: `rocky.d.ts` for RockyJS API

## Toolchains and Compilers

### ARM Embedded Toolchain

```
arm-none-eabi-gcc      # ARM cross-compiler
arm-none-eabi-readelf  # ELF reader
arm-none-eabi-objcopy  # Binary conversion
```

### Supporting Tools

| Tool | Purpose |
|------|---------|
| LLVM/Clang | Static analysis, compilation |
| Emscripten | WebAssembly compilation for testing |
| Node.js | JavaScript runtime for tools |

## Development and Debugging Tools

### Debuggers

**GDB (arm-none-eabi-gdb)**
- Configuration: `/.gdbinit`
- Custom scripts: `/tools/gdb_scripts/`

**OpenOCD** - On-Chip Debugger
- Interface configurations for:
  - FTDI adapters
  - J-Link
  - CMSIS-DAP
  - Tigard

### Code Quality Tools

| Tool | Configuration | Purpose |
|------|---------------|---------|
| clang-format | `.clang-format` | C code formatting |
| ESLint | `.eslintrc.js` | JavaScript linting |
| lcov | WAF integration | Code coverage |
| Custom Clang checkers | `/checkers/` | Security analysis |

### Custom Static Analysis

Located in `/checkers/`:

- `MutexChecker.cpp` - Mutex safety verification
- `SyscallSecurityChecker.cpp` - Syscall security analysis

## CI/CD Pipeline

**GitHub Actions Workflows** (`.github/workflows/`):

| Workflow | Purpose |
|----------|---------|
| `build-firmware.yml` | Main firmware compilation |
| `build-bootloader.yml` | Bootloader builds |
| `build-qemu.yml` | QEMU emulation builds |
| `build-prf.yml` | Platform resource format |
| `test.yml` | Unit test execution |
| `compliance.yml` | Code compliance checks |
| `release.yml` | Release packaging |

**Container Environment:**
```
ghcr.io/pebble-dev/pebbleos-docker:v1
```

## Documentation Tools

From `/docs/requirements.txt`:

```
sphinx~=8.2           # Documentation generation
myst-parser~=4.0      # Markdown support
sphinx-book-theme~=1.1 # Theme
sphinx-design~=0.6    # Design extensions
```

**API Documentation:**
- Doxygen configuration: `/Doxyfile`
- SDK template: `/sdk/Doxyfile-SDK.template`

## Summary Table

| Category | Technologies |
|----------|-------------|
| **Compiled Code** | C (primary), ARM Assembly |
| **Scripting** | JavaScript, Python, Shell |
| **Build System** | WAF (primary), CMake, Webpack |
| **RTOS** | FreeRTOS |
| **JavaScript Engine** | JerryScript |
| **Bluetooth** | NimBLE |
| **Debugging** | GDB, OpenOCD, clang-format |
| **Hardware Support** | 11+ platform variants |
| **Compiler** | ARM EABI GCC, LLVM, Emscripten |
| **CI/CD** | GitHub Actions |
| **Documentation** | Sphinx, Doxygen |
| **Protocols** | Protocol Buffers (nanopb), gRPC |
