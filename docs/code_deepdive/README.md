# PebbleOS Code Deep Dive Documentation

This documentation provides a comprehensive analysis of the PebbleOS codebase, designed to help developers understand the architecture, code structure, and implementation details of the open-source Pebble smartwatch operating system.

## Documentation Structure

| Document | Description |
|----------|-------------|
| [01_languages_and_frameworks.md](01_languages_and_frameworks.md) | Programming languages, frameworks, and tools used in the project |
| [02_repository_structure.md](02_repository_structure.md) | Repository organization and directory layout |
| [03_getting_started.md](03_getting_started.md) | Overview and guide for exploring the codebase |
| [04_architecture_deep_dive.md](04_architecture_deep_dive.md) | Deep dive into code flow, boot sequence, and system architecture |

## Quick Overview

**PebbleOS** is an open-source operating system for Pebble smartwatches, originally developed by Pebble Technology and now maintained by the community. The codebase is built primarily in C with FreeRTOS as the real-time kernel, supporting multiple hardware platforms.

### Key Statistics

| Metric | Value |
|--------|-------|
| Primary Language | C (~2,050 source files) |
| Scripting | JavaScript (~1,799 files), Python (~565 files) |
| Build System | WAF (Python-based) |
| RTOS | FreeRTOS |
| License | Apache License 2.0 |
| Supported Platforms | 11+ hardware variants |

### Major Components

```
PebbleOS
├── Firmware Core (src/fw/)
│   ├── Kernel - Task management, event system, memory
│   ├── Drivers - Hardware abstraction layer
│   ├── AppLib - Application framework
│   └── Services - System services
├── Bluetooth Stack (src/bluetooth-fw/)
├── Platform Support (platform/)
├── SDK (sdk/)
├── Tools & Build System (tools/, waftools/)
└── Third-Party Libraries (third_party/)
```

### Architecture Highlights

1. **FreeRTOS-based**: Multi-task architecture with hardware task isolation
2. **Event-driven**: All inter-task communication via message queues
3. **Graphics Pipeline**: Layer-based rendering with hardware compositor
4. **BLE Stack**: NimBLE-based Bluetooth Low Energy implementation
5. **JavaScript Support**: JerryScript engine for RockyJS watchfaces

## Getting Started

If you're new to the codebase, we recommend reading the documentation in order:

1. Start with **[Languages and Frameworks](01_languages_and_frameworks.md)** to understand the technology stack
2. Read **[Repository Structure](02_repository_structure.md)** to learn the codebase organization
3. Follow **[Getting Started](03_getting_started.md)** for practical exploration tips
4. Dive into **[Architecture Deep Dive](04_architecture_deep_dive.md)** for detailed system understanding

## Related Documentation

- [Getting Started Guide](../getting_started.md) - Build setup and compilation
- [Board Documentation](../boards/) - Hardware-specific information
- [Development Guides](../development/) - Build options, QEMU, profiling
