# Getting Started with PebbleOS Code Exploration

This guide provides a practical overview for developers who want to explore and understand the PebbleOS codebase. It covers key concepts, navigation strategies, and essential entry points.

## Quick Orientation

### What is PebbleOS?

PebbleOS is an open-source real-time operating system designed for Pebble smartwatches. It provides:

- **Real-time kernel** based on FreeRTOS
- **Multi-tasking** with hardware memory protection
- **Graphics framework** with layer-based rendering
- **Bluetooth Low Energy** connectivity
- **JavaScript runtime** for watchfaces (RockyJS)
- **SDK** for third-party app development

### High-Level Architecture

```
┌────────────────────────────────────────────────────┐
│                 User Applications                   │
│           (Watchfaces, Apps, Workers)               │
├────────────────────────────────────────────────────┤
│              Application Library (AppLib)           │
│     (Graphics, UI Widgets, Services, Messaging)     │
├────────────────────────────────────────────────────┤
│                  System Services                    │
│   (Timer, Analytics, Clock, Compositor, BLE)        │
├────────────────────────────────────────────────────┤
│                     Kernel                          │
│   (Tasks, Events, Memory, Fault Handling)           │
├────────────────────────────────────────────────────┤
│               Hardware Drivers                      │
│  (Display, IMU, Battery, Buttons, Flash, BLE)       │
├────────────────────────────────────────────────────┤
│            Hardware Abstraction Layer               │
│         (MCU-specific implementations)              │
├────────────────────────────────────────────────────┤
│                    Hardware                         │
│   (STM32, nRF52, SF32LB52 microcontrollers)         │
└────────────────────────────────────────────────────┘
```

## Essential Entry Points

Start your exploration from these key files:

### 1. Firmware Entry Point

**File:** `src/fw/main.c`

This is where everything begins:
- `main()` - Hardware initialization, creates KernelMain task
- `prv_main_task_init()` - Full system initialization
- `launcher_main_loop()` - Main event processing loop

```c
// Simplified boot sequence
int main(void) {
    gpio_init_all();
    kernel_heap_init();
    dbgserial_init();
    rtc_init();

    pebble_task_create(PebbleTask_KernelMain, ...);
    vTaskStartScheduler();  // Never returns
}
```

### 2. Event System

**File:** `src/fw/kernel/event_loop.c`

The event-driven heart of the system:
- `launcher_main_loop()` - Infinite event processing loop
- Event dispatching to handlers
- System housekeeping

**File:** `src/fw/kernel/events.c`

- Event queue management
- `event_put()` - Add events to queues
- `event_take_timeout()` - Wait for events

### 3. Task Management

**File:** `src/fw/kernel/pebble_tasks.c`

FreeRTOS task organization:
- Task definitions and priorities
- Memory protection regions
- Task creation and management

### 4. Application Framework

**File:** `src/fw/applib/ui/window.c`

Window management for apps:
- Window lifecycle (load, appear, disappear, unload)
- Click configuration
- Background rendering

**File:** `src/fw/applib/ui/layer.c`

Layer-based rendering system:
- Hierarchical layer tree
- Property management
- Update callbacks

### 5. Graphics Pipeline

**File:** `src/fw/services/common/compositor/compositor.c`

System compositor:
- Combines app and modal framebuffers
- Handles transitions
- Manages display updates

**File:** `src/fw/applib/graphics/graphics.c`

Drawing primitives:
- Lines, rectangles, circles
- Fill operations
- Color management

## Navigation Strategies

### Following the Boot Sequence

1. Start at `src/fw/main.c:main()`
2. Follow `prv_main_task_init()` for initialization order
3. See `init_drivers()` for hardware setup
4. End at `launcher_main_loop()` for runtime behavior

### Understanding Event Flow

1. Find where events are generated (drivers, services)
2. Trace `event_put()` calls
3. Follow handling in `event_loop.c`
4. See subscriber notifications in `event_service_handle_event()`

### Exploring Driver Architecture

1. Start with the driver header in `src/fw/drivers/`
2. Find platform-independent API
3. Locate platform-specific implementations
4. Check board configuration in `src/fw/board/`

### Studying the Graphics System

1. Begin with `layer.c` for the rendering tree
2. See `window.c` for window management
3. Study `compositor.c` for final composition
4. Check display driver for hardware output

## Key Concepts

### FreeRTOS Tasks

PebbleOS runs on FreeRTOS with these main tasks:

| Task | Character | Purpose |
|------|-----------|---------|
| KernelMain | 'm' | Main event loop, app management |
| App | 'a' | User application execution |
| Worker | 'w' | Background processing |
| BTHost | 'b' | Bluetooth host stack |
| BTController | 'c' | BLE controller |
| NewTimers | 't' | Timer management |

### Event-Driven Architecture

All communication between components uses events:

```c
// Creating and posting an event
PebbleEvent event = {
    .type = PEBBLE_BUTTON_DOWN_EVENT,
    .button.button_id = BUTTON_ID_SELECT,
};
event_put(&event);

// Processing events (in event loop)
if (event_take_timeout(&e, 1000)) {
    prv_handle_event(&e);
}
```

### Memory Protection

Apps run in isolated memory regions:
- MPU (Memory Protection Unit) enforces boundaries
- Apps cannot access kernel memory
- Stack guards prevent overflow

### Layer-Based Rendering

UI is built from layers:
```
Window
└── Root Layer
    ├── Text Layer "Hello"
    ├── Bitmap Layer (icon)
    └── Custom Layer
        └── Child Layer
```

## Common Development Tasks

### Finding a Feature Implementation

1. **Start with headers:** Look in `src/fw/applib/` for public API
2. **Search for functions:** Use grep for function names
3. **Follow includes:** Trace header dependencies
4. **Check tests:** Look in `tests/fw/` for usage examples

### Understanding a Driver

1. Check the header file (e.g., `drivers/battery.h`)
2. Find the init function (e.g., `battery_init()`)
3. Locate interrupt handlers
4. Study the state machine

### Tracing an Event

1. Find the event type in `src/fw/kernel/events.h`
2. Search for `event_put()` with that type
3. Find handler in `event_loop.c`
4. Follow subscriber callbacks

### Adding a New Feature

1. Understand existing similar features
2. Follow the coding style in `.clang-format`
3. Add tests in `tests/fw/`
4. Update documentation

## Useful Search Patterns

```bash
# Find all event types
grep -r "PEBBLE_.*_EVENT" src/fw/kernel/events.h

# Find task definitions
grep -r "PebbleTask_" src/fw/kernel/

# Find driver init functions
grep -r "_init()" src/fw/drivers/

# Find window handlers
grep -r "window_set_.*_handler" src/fw/applib/

# Find service subscriptions
grep -r "_subscribe(" src/fw/services/
```

## Code Style Guide

PebbleOS follows consistent coding conventions:

### Naming Conventions

- Functions: `snake_case` with module prefix
- Types: `PascalCase` (e.g., `GPoint`, `Window`)
- Constants: `UPPER_SNAKE_CASE`
- Private functions: `prv_` prefix
- Static variables: `s_` prefix

### File Organization

```c
// 1. Copyright header
// 2. Includes (system, then project)
// 3. Defines and constants
// 4. Type definitions
// 5. Static variables
// 6. Private function declarations
// 7. Private function implementations
// 8. Public function implementations
```

### Common Patterns

**Event handling:**
```c
static void prv_handle_event(PebbleEvent *event) {
    switch (event->type) {
        case PEBBLE_BUTTON_DOWN_EVENT:
            prv_handle_button_down(event);
            break;
        // ...
    }
}
```

**Driver initialization:**
```c
void driver_init(void) {
    // Configure hardware
    // Set up interrupts
    // Initialize state
}
```

**Layer update callbacks:**
```c
static void prv_update_proc(Layer *layer, GContext *ctx) {
    // Draw layer content
}
```

## Debugging Tips

### Using Debug Logging

```c
#include "system/logging.h"

PBL_LOG(LOG_LEVEL_DEBUG, "Value: %d", value);
PBL_LOG(LOG_LEVEL_WARNING, "Warning message");
PBL_LOG(LOG_LEVEL_ERROR, "Error occurred");
```

### GDB Integration

The repository includes GDB scripts in `tools/gdb_scripts/`:
- Pretty printers for Pebble types
- Helper commands for debugging

### QEMU Emulation

Test without hardware using QEMU:
```bash
# See docs/development/qemu.md for setup
```

## Next Steps

After familiarizing yourself with this overview:

1. **Read the architecture deep dive** - `04_architecture_deep_dive.md`
2. **Set up the build environment** - `../getting_started.md`
3. **Explore specific subsystems** based on your interests
4. **Run tests** to understand expected behavior
5. **Join the community** for questions and contributions

## Quick Reference Card

| Want to understand... | Start here |
|-----------------------|------------|
| Boot sequence | `src/fw/main.c` |
| Event system | `src/fw/kernel/event_loop.c` |
| Task architecture | `src/fw/kernel/pebble_tasks.c` |
| Window management | `src/fw/applib/ui/window.c` |
| Graphics rendering | `src/fw/applib/graphics/graphics.c` |
| Layer system | `src/fw/applib/ui/layer.c` |
| Compositor | `src/fw/services/common/compositor/` |
| Bluetooth | `src/fw/comm/ble/` |
| System apps | `src/fw/apps/system_apps/` |
| Drivers | `src/fw/drivers/` |
| Board configs | `src/fw/board/` |
| Unit tests | `tests/fw/` |
