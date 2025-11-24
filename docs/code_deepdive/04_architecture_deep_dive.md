# Architecture Deep Dive

This document provides a comprehensive analysis of PebbleOS architecture, covering the boot sequence, task management, event system, application lifecycle, graphics pipeline, and hardware driver organization.

## Table of Contents

1. [Boot Sequence](#1-boot-sequence)
2. [Task Architecture](#2-task-architecture)
3. [Event System](#3-event-system)
4. [Application Lifecycle](#4-application-lifecycle)
5. [Graphics Pipeline](#5-graphics-pipeline)
6. [Communication Flow](#6-communication-flow)
7. [Driver Architecture](#7-driver-architecture)
8. [Memory Management](#8-memory-management)
9. [Design Patterns](#9-design-patterns)

---

## 1. Boot Sequence

The firmware initialization follows a carefully orchestrated multi-stage process.

### Stage 1: Hardware Reset to main()

**Location:** `src/fw/main.c:180`

```
HARDWARE RESET
     │
     ▼
main() Entry
     │
     ├── board_early_init()         # MCU-specific early setup
     ├── gpio_init_all()            # GPIO configuration
     ├── Enable/disable MCU debug   # Debug interface setup
     ├── Set NVIC priority grouping # Interrupt priorities
     ├── enable_fault_handlers()    # Exception handling
     ├── kernel_heap_init()         # Dynamic memory
     ├── mbuf_init()                # Message buffers
     ├── delay_init()               # Delay utilities
     ├── periph_config_init()       # Peripheral clocks
     ├── dbgserial_init()           # Debug serial output
     ├── pulse_early_init()         # Pulse protocol
     ├── print_splash_screen()      # Boot message
     ├── rtc_init()                 # Real-time clock
     │
     ├── Create KernelMain Task
     │   └── pebble_task_create(PebbleTask_KernelMain, ...)
     │
     ├── stop_mode_disable()        # Prevent sleep during init
     ├── pwr_flash_power_down_stop_mode()
     │
     └── vTaskStartScheduler()      # Start FreeRTOS (never returns)
```

### Stage 2: KernelMain Task Initialization

**Location:** `src/fw/main.c:404` (`prv_main_task_init`)

```
KernelMain Task Starts
     │
     ├── Clear watchdog flags
     ├── pulse_init() / pulse_logging_init()
     ├── pebble_task_configure_idle_task()
     ├── task_init()
     ├── memory_layout_setup_mpu()      # Memory protection
     │
     ├── display_show_splash_screen()   # Visual feedback
     │
     ├── kernel_applib_init()           # App library state
     ├── system_task_init()
     ├── events_init()                  # Event queues
     ├── new_timer_service_init()
     ├── regular_timer_init()
     │
     ├── task_watchdog_init()           # Watchdog (paused 30s)
     ├── analytics_init()
     ├── register_system_timers()
     │
     ├── init_drivers()                 # Hardware drivers
     │   ├── board_init()
     │   ├── dbgserial_input_init()
     │   ├── serial_console_init()
     │   ├── voltage_monitor_init()
     │   ├── battery_init()
     │   ├── vibe_init()
     │   ├── accessory_init()
     │   ├── pmic_init()
     │   ├── flash_init()
     │   ├── mic_init()
     │   ├── touch_sensor_init()
     │   ├── imu_init()
     │   ├── backlight_init()
     │   ├── ambient_light_init()
     │   ├── temperature_init()
     │   └── rtc_init_timers()
     │
     ├── clock_init()
     ├── debug_init()
     ├── services_early_init()
     ├── check_prf_update()
     ├── resource_init()
     ├── system_resource_init()
     ├── hrm_init() (if present)
     │
     ├── display_init()                 # Main display
     ├── compositor_init()              # Graphics compositor
     ├── kernel_ui_init()
     │
     ├── bt_driver_init()               # Bluetooth
     ├── services_init()                # All services
     │
     ├── rtc_calibrate_frequency()
     ├── clear_reset_loop_detection_bits()
     │
     ├── Setup timers (low power, uptime)
     ├── debounced_button_init()        # Last: prevent spurious input
     │
     └── launcher_main_loop()           # Main event loop (never returns)
```

### Initialization Dependencies

```
GPIO init ──► Periph config ──► Device drivers ──► Services
                  │
                  ▼
            Clock setup ──► Timer services ──► Watchdog
                  │
                  ▼
            Display init ──► Compositor ──► UI ready
                  │
                  ▼
            Bluetooth init ──► Communication ready
```

---

## 2. Task Architecture

PebbleOS uses FreeRTOS with carefully designed task isolation.

### Task Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                     FreeRTOS Scheduler                          │
└─────────────────────────────────────────────────────────────────┘
                              │
    ┌─────────────────────────┼─────────────────────────┐
    │                         │                         │
    ▼                         ▼                         ▼
┌─────────┐            ┌─────────────┐           ┌───────────┐
│ BT Tasks│            │ App/Worker  │           │  Kernel   │
│ (High)  │            │  (Medium)   │           │  (Base)   │
└─────────┘            └─────────────┘           └───────────┘
│ BTHCI   │            │ App Task    │           │ KernelMain│
│ BTCtrl  │            │ Worker Task │           │ KernelBG  │
│ BTHost  │            │             │           │ NewTimers │
└─────────┘            └─────────────┘           └───────────┘
```

### Task Definitions

| Task | ID | Priority | Stack | Purpose |
|------|----|----------|-------|---------|
| KernelMain | 'm' | IDLE+3 | Configurable | Main event loop, app lifecycle |
| KernelBackground | 's' | - | - | Background kernel tasks |
| App | 'a' | IDLE+2 | App-specific | User application code |
| Worker | 'w' | IDLE+1 | Worker-specific | Background processing |
| BTHost | 'b' | High | BT stack | Bluetooth host protocol |
| BTController | 'c' | High | BT stack | BLE link layer |
| BTHCI | 'd' | Highest | BT stack | Hardware interface |
| NewTimers | 't' | High | Timer stack | Timer callbacks |
| PULSE | 'p' | - | - | Debug/profiling |

### Memory Protection (MPU)

Each task has defined memory access regions:

```c
// App task can access:
// - App code region (read-only)
// - App data region (read-write)
// - Shared resources (controlled)

// App task CANNOT access:
// - Worker region
// - Kernel memory
// - Other app memory
// - Hardware registers (syscalls only)
```

### Task Communication

```
                    Event Queues
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    ▼                    ▼                    ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ s_kernel_   │   │ s_to_app_   │   │ s_to_worker │
│ event_queue │   │ event_queue │   │ event_queue │
│ (32 events) │   │ (32 events) │   │ (5 events)  │
└─────────────┘   └─────────────┘   └─────────────┘
       │                 │                 │
       ▼                 ▼                 ▼
  KernelMain         App Task         Worker Task
```

---

## 3. Event System

The event system is the central communication mechanism.

### Event Queue Architecture

**Location:** `src/fw/kernel/event_loop.c`, `src/fw/kernel/events.c`

```
┌──────────────────────────────────────────────────────────────────┐
│                  System Event Queue Set                          │
│                 (s_system_event_queue_set)                       │
└──────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ s_kernel_     │    │ s_from_app_   │    │ s_from_worker │
│ event_queue   │    │ event_queue   │    │ event_queue   │
│               │    │               │    │               │
│ Hardware/     │    │ App → Kernel  │    │ Worker → K    │
│ System events │    │ (10 max)      │    │ (5 max)       │
│ (32 max)      │    │               │    │               │
└───────────────┘    └───────────────┘    └───────────────┘
```

### Event Types

**Location:** `src/fw/kernel/events.h`

```c
typedef enum {
    // Input Events (0-9)
    PEBBLE_BUTTON_DOWN_EVENT,
    PEBBLE_BUTTON_UP_EVENT,
    PEBBLE_ACCEL_SHAKE_EVENT,
    PEBBLE_TOUCH_EVENT,

    // Rendering Events (10-19)
    PEBBLE_RENDER_REQUEST_EVENT,      // Kernel → App
    PEBBLE_RENDER_READY_EVENT,        // App → Kernel
    PEBBLE_RENDER_FINISHED_EVENT,     // Kernel → App

    // App Lifecycle (20-39)
    PEBBLE_APP_LAUNCH_EVENT,
    PEBBLE_APP_WILL_CHANGE_FOCUS_EVENT,
    PEBBLE_APP_DID_CHANGE_FOCUS_EVENT,
    PEBBLE_PROCESS_DEINIT_EVENT,

    // Communication (40-59)
    PEBBLE_COMM_SESSION_EVENT,
    PEBBLE_NEW_APP_MESSAGE_EVENT,
    PEBBLE_APP_OUTBOX_SENT_EVENT,
    PEBBLE_APP_OUTBOX_MSG_EVENT,

    // System (60-79)
    PEBBLE_BATTERY_CONNECTION_EVENT,
    PEBBLE_BATTERY_STATE_CHANGE_EVENT,
    PEBBLE_TICK_EVENT,
    PEBBLE_SET_TIME_EVENT,
    PEBBLE_PANIC_EVENT,

    // Bluetooth (80-99)
    PEBBLE_BLE_CONNECTION_EVENT,
    PEBBLE_BLE_GATT_CLIENT_EVENT,

    // ... (143 total event types)
} PebbleEventType;
```

### Event Flow

```
Event Source (Hardware/Service/App)
              │
              ▼
       event_put(&event)
              │
              ├── Determine target queue
              ├── Copy event to queue
              └── Wake waiting task
              │
              ▼
    KernelMain Event Loop (launcher_main_loop)
              │
              ├── event_take_timeout(&e, 1000)
              │
              ▼
       Event Received
              │
              ├── Check task mask
              ├── prv_minimal_event_handler()  ← Critical events
              │   ├── Button events
              │   ├── App launch/close
              │   ├── Battery events
              │   └── Panic events
              │
              ├── Check popup blocking
              │
              ├── prv_extended_event_handler() ← Non-critical
              │   ├── App messages
              │   ├── BLE events
              │   └── Time sync
              │
              ├── shell_event_loop_handle_event()
              │
              ├── event_service_handle_event()
              │   └── Notify all subscribers
              │
              └── event_cleanup(&e)
```

### Event Loop Implementation

**Location:** `src/fw/kernel/event_loop.c`

```c
void launcher_main_loop(void) {
    while (1) {
        // Keep watchdog happy
        task_watchdog_bit_set(PebbleTask_KernelMain);

        PebbleEvent e;
        if (event_take_timeout(&e, 1000)) {
            // Check if event should be processed by this task
            if (!(e.task_mask & (1 << PebbleTask_KernelMain))) {
                // Handle critical events first
                prv_minimal_event_handler(&e);

                // Check if popups should be blocked
                if (!prv_should_block_popup(&e)) {
                    prv_extended_event_handler(&e);
                }

                // Shell-specific handling
                shell_event_loop_handle_event(&e);

                // Notify subscribers
                event_service_handle_event(&e);
            }

            // Clean up event resources
            event_cleanup(&e);

            // Periodic housekeeping
            event_loop_upkeep();
        }
    }
}
```

---

## 4. Application Lifecycle

### App States

```
                    ┌──────────────┐
                    │   STOPPED    │
                    └──────┬───────┘
                           │ Launch Event
                           ▼
                    ┌──────────────┐
                    │   LOADING    │  Load from flash
                    └──────┬───────┘  Allocate memory
                           │          Initialize context
                           ▼
                    ┌──────────────┐
                    │   RUNNING    │  Process events
                    └──────┬───────┘  Render UI
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
     ┌─────────┐     ┌─────────┐     ┌─────────┐
     │GRACEFUL │     │ FORCE   │     │WATCHDOG │
     │ CLOSE   │     │ CLOSE   │     │ TIMEOUT │
     └────┬────┘     └────┬────┘     └────┬────┘
          │               │               │
          └───────────────┴───────────────┘
                          │
                          ▼
                    ┌──────────────┐
                    │  UNLOADING   │  Call deinit
                    └──────┬───────┘  Free memory
                           │
                           ▼
                    ┌──────────────┐
                    │   STOPPED    │
                    └──────────────┘
```

### App Launch Flow

**Location:** `src/fw/process_management/app_manager.c`

```
User Action (Button Press / API Call)
              │
              ▼
       PEBBLE_APP_LAUNCH_EVENT
              │
              ▼
    KernelMain processes event
              │
              ▼
    process_manager_launch_process(&config)
              │
              ├── Validate app
              ├── Stop current app (if any)
              ├── Load app binary from flash
              │   ├── Verify signature
              │   ├── Check memory requirements
              │   └── Load into App RAM
              │
              ├── Create FreeRTOS task
              └── Start app task
              │
              ▼
    prv_app_task_main() [App Task]
              │
              ├── app_state_init()
              │   ├── Initialize graphics context
              │   ├── Setup window stack
              │   └── Initialize services
              │
              ├── Enter unprivileged mode (optional)
              │
              ├── Call user main() function
              │   └── App code runs here
              │
              ├── app_state_deinit()
              │   ├── Unsubscribe services
              │   └── Free resources
              │
              └── sys_exit()
```

### App Memory Layout

```
╔════════════════════════════════════════════╗
║         APP_RAM Region (Allocated)         ║
╠════════════════════════════════════════════╣
║  App Code (.text)                          ║
║  - Read-only, execute                      ║
║  - Loaded from flash                       ║
╠════════════════════════════════════════════╣
║  App Data (.data)                          ║
║  - Initialized variables                   ║
║  - Read-write                              ║
╠════════════════════════════════════════════╣
║  App BSS (.bss)                            ║
║  - Zero-initialized                        ║
║  - Read-write                              ║
╠════════════════════════════════════════════╣
║  App Heap                                  ║
║  - Dynamic allocation                      ║
║  - Grows upward ↑                          ║
╠════════════════════════════════════════════╣
║  Guard Area                                ║
║  - Stack overflow protection               ║
╠════════════════════════════════════════════╣
║  App Stack                                 ║
║  - Local variables                         ║
║  - Grows downward ↓                        ║
╚════════════════════════════════════════════╝
```

### Window Lifecycle

**Location:** `src/fw/applib/ui/window.c`

```c
// Window handlers called in order:
window_set_window_handlers(window, (WindowHandlers) {
    .load = prv_window_load,        // 1. Called when pushed
    .appear = prv_window_appear,    // 2. Called when visible
    .disappear = prv_window_disappear, // 3. Called when hidden
    .unload = prv_window_unload,    // 4. Called when popped
});

// Push window onto stack
window_stack_push(window, animated);

// Pop window from stack
window_stack_pop(animated);
```

---

## 5. Graphics Pipeline

### Rendering Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  Application Layer                          │
│           (Running in App task - PebbleTask_App)            │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Window Stack (window_stack.c)   │
           │  - Manages window transitions     │
           │  - Handles animations            │
           │  - Click routing                 │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Window Management (window.c)    │
           │  - Window load/unload            │
           │  - Background color              │
           │  - Fullscreen mode               │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Layer Tree (layer.c)            │
           │  - Hierarchical rendering        │
           │  - Bounds clipping               │
           │  - Update callbacks              │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Graphics Context (GContext)     │
           │  - Drawing primitives            │
           │  - Fill operations               │
           │  - Text rendering                │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Framebuffer (framebuffer.c)     │
           │  - Double buffering              │
           │  - Pixel format handling         │
           └──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   COMPOSITOR (Kernel)                        │
│              (Running in KernelMain task)                   │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Compositor States:              │
           │  - CompositorState_App           │
           │  - CompositorState_Modal         │
           │  - CompositorState_AppAndModal   │
           │  - CompositorState_Transitioning │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Display Driver (display.c)      │
           │  - SPI/parallel interface        │
           │  - DMA transfers                 │
           │  - VSYNC handling                │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │  Physical Display (LCD/OLED)     │
           └──────────────────────────────────┘
```

### Render Event Flow

```
App Task: window_dirty(window)
              │
              ▼
       Mark window dirty
              │
              ▼
    app_post_render_ready()
              │
              ▼
    PEBBLE_RENDER_READY_EVENT → KernelMain
              │
              ▼
    compositor_app_render_ready()
              │
              ├── Check compositor state
              │
              ├── If transitioning:
              │   └── Schedule render for later
              │
              └── If ready:
                  └── compositor_render()
                      │
                      ├── Render app framebuffer
                      ├── Render modal (if any)
                      ├── Apply effects (corners)
                      └── Send to display
                      │
                      ▼
            PEBBLE_RENDER_FINISHED_EVENT → App
```

### Layer Tree Structure

```
Window Root Layer
       │
       ├─ Layer 1 (Custom)
       │   └─ Update: draw_background()
       │
       ├─ Text Layer ("Hello World")
       │   └─ Update: text_layer_update_proc()
       │
       ├─ Bitmap Layer (icon.png)
       │   └─ Update: bitmap_layer_update_proc()
       │
       └─ Scroll Layer
           │
           └─ Content Layer
               │
               ├─ Menu Layer
               └─ ...
```

### Layer Update Process

**Location:** `src/fw/applib/ui/layer.c`

```c
void layer_render_tree(Layer *layer, GContext *ctx) {
    // For each layer in tree (depth-first):

    // 1. Save graphics state
    graphics_context_push(ctx);

    // 2. Apply layer transforms
    //    - Offset by frame origin
    //    - Set clip rectangle

    // 3. Call update proc (if set)
    if (layer->update_proc) {
        layer->update_proc(layer, ctx);
    }

    // 4. Render children
    Layer *child = layer->first_child;
    while (child) {
        layer_render_tree(child, ctx);
        child = child->next_sibling;
    }

    // 5. Restore graphics state
    graphics_context_pop(ctx);
}
```

---

## 6. Communication Flow

### Bluetooth Architecture

```
┌──────────────────────────────────────────────────────────────┐
│  Application Layer (App task)                                │
│  - AppMessage outbox/inbox                                   │
│  - Service subscriptions                                     │
└──────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│  Pebble Services Layer (KernelMain)                          │
│  - Connection management                                     │
│  - Notification handling (ANCS)                              │
│  - Music metadata (AMS)                                      │
└──────────────────────────────────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │ GATT Services                    │
           ├──────────────────────────────────┤
           │ - PPoGATT (Pebble Protocol)      │
           │ - ANCS (Apple Notifications)     │
           │ - AMS (Apple Media)              │
           │ - DIS (Device Info)              │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │ Bluetooth Host (BTHost task)     │
           │ - GAP management                 │
           │ - GATT client                    │
           │ - Security Manager               │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │ Bluetooth Controller (BTCtrl)    │
           │ - Link layer                     │
           │ - HCI commands                   │
           └──────────────────────────────────┘
                           │
                           ▼
           ┌──────────────────────────────────┐
           │ BLE Radio Hardware               │
           └──────────────────────────────────┘
```

### AppMessage Flow

```
App Outbox (Sending):
    app_message_outbox_begin()
              │
              ▼
    app_message_outbox_get() → Dictionary
              │
              ▼
    dict_write_*() calls
              │
              ▼
    app_message_outbox_send()
              │
              ▼
    PEBBLE_APP_OUTBOX_MSG_EVENT
              │
              ▼
    Serialize to PPoGATT
              │
              ▼
    Send via Bluetooth
              │
              ▼
    Phone receives data

App Inbox (Receiving):
    Phone sends data
              │
              ▼
    Bluetooth receives
              │
              ▼
    PEBBLE_NEW_APP_MESSAGE_EVENT
              │
              ▼
    app_message_inbox_received callback
              │
              ▼
    dict_read_*() to parse
```

---

## 7. Driver Architecture

### Driver Organization

```
src/fw/drivers/
├── Platform-Independent APIs
│   ├── display/           # Display abstraction
│   ├── battery.h          # Battery interface
│   ├── button.h           # Button interface
│   ├── imu/               # IMU interface
│   └── ...
│
├── Platform-Specific Implementations
│   ├── imu/
│   │   ├── lis2dw12/      # STM32 accelerometer
│   │   ├── lsm6ds3/       # Alternative IMU
│   │   └── kionix/        # Kionix accelerometer
│   │
│   ├── flash/
│   │   ├── mxic/          # Macronix flash
│   │   ├── winbond/       # Winbond flash
│   │   └── sf32lb52/      # Sifli flash
│   │
│   └── display/
│       ├── sharp/         # Sharp memory LCD
│       └── ...
│
└── MCU-Specific
    ├── nrf5/              # Nordic nRF52
    ├── stm32f2/           # STM32F2
    ├── stm32f4/           # STM32F4
    ├── stm32f7/           # STM32F7
    └── sf32lb52/          # Sifli
```

### Driver Initialization Pattern

```c
// Typical driver structure:

// 1. Include headers
#include "drivers/foo.h"
#include "board/board.h"

// 2. Static state
static FooState s_state;

// 3. Private functions
static void prv_configure_hardware(void) {
    // Setup GPIO, SPI, I2C, etc.
}

static void prv_interrupt_handler(void) {
    // Handle hardware interrupt
}

// 4. Public initialization
void foo_init(void) {
    // Configure hardware
    prv_configure_hardware();

    // Setup interrupts
    NVIC_EnableIRQ(FOO_IRQn);

    // Initialize state
    s_state.initialized = true;
}

// 5. Public API
FooStatus foo_get_status(void) {
    return s_state.current_status;
}
```

### Driver State Machine Example

```
Battery Driver State Machine:

    ┌────────────┐
    │  UNKNOWN   │ ◄─── Power on
    └─────┬──────┘
          │ Read voltage
          ▼
    ┌────────────┐
    │ Check USB  │
    └─────┬──────┘
          │
    ┌─────┴─────┐
    │           │
    ▼           ▼
┌────────┐  ┌─────────────┐
│CHARGING│  │DISCHARGING  │
└────┬───┘  └──────┬──────┘
     │             │
     └──────┬──────┘
            │
            ▼
    PEBBLE_BATTERY_STATE_CHANGE_EVENT
            │
            ▼
    Notify subscribers
```

---

## 8. Memory Management

### Memory Regions

```
Flash Memory Layout:
┌───────────────────────────────────┐  High Address
│  Bootloader (protected)           │
├───────────────────────────────────┤
│  Firmware Slot 0                  │
├───────────────────────────────────┤
│  Firmware Slot 1                  │
├───────────────────────────────────┤
│  System Resources                 │
├───────────────────────────────────┤
│  App Storage                      │
├───────────────────────────────────┤
│  File System                      │
└───────────────────────────────────┘  Low Address

RAM Layout:
┌───────────────────────────────────┐  High Address
│  Kernel Stack                     │
├───────────────────────────────────┤
│  Kernel Heap                      │
├───────────────────────────────────┤
│  Static Data (.data, .bss)        │
├───────────────────────────────────┤
│  App/Worker RAM Region            │
│  (MPU protected)                  │
├───────────────────────────────────┤
│  Framebuffer(s)                   │
├───────────────────────────────────┤
│  FreeRTOS Heap                    │
└───────────────────────────────────┘  Low Address
```

### Heap Management

**Location:** `src/fw/kernel/pbl_malloc.c`

```c
// Kernel heap allocation
void* kernel_malloc(size_t size);
void kernel_free(void* ptr);

// App heap allocation (restricted to app region)
void* app_malloc(size_t size);
void app_free(void* ptr);
```

### MPU Configuration

```c
// Memory Protection Unit regions:

// Region 0: Flash (read-only, execute)
// Region 1: Kernel RAM (privileged only)
// Region 2: App RAM (app task only)
// Region 3: Worker RAM (worker task only)
// Region 4: Shared RAM (read-only for apps)
// Region 5: Peripherals (privileged only)
```

---

## 9. Design Patterns

### Common Patterns in PebbleOS

#### 1. Event-Driven Communication

```c
// Producer: Generate event
PebbleEvent event = {
    .type = PEBBLE_CUSTOM_EVENT,
    .data = custom_data,
};
event_put(&event);

// Consumer: Handle event
static void prv_event_handler(PebbleEvent *event) {
    if (event->type == PEBBLE_CUSTOM_EVENT) {
        process_custom_event(event->data);
    }
}
```

#### 2. Service Subscription

```c
// Subscribe to service
static void prv_battery_handler(BatteryState state) {
    // Handle battery change
}

battery_state_service_subscribe(prv_battery_handler);

// Unsubscribe
battery_state_service_unsubscribe();
```

#### 3. Layer Update Callback

```c
// Define update procedure
static void prv_layer_update(Layer *layer, GContext *ctx) {
    GRect bounds = layer_get_bounds(layer);
    graphics_fill_rect(ctx, bounds, 0, GCornerNone);
}

// Assign to layer
layer_set_update_proc(layer, prv_layer_update);

// Trigger redraw
layer_mark_dirty(layer);
```

#### 4. Timer Callback

```c
// Create timer
static void prv_timer_callback(void *context) {
    // Timer fired
}

AppTimer *timer = app_timer_register(
    1000,  // milliseconds
    prv_timer_callback,
    NULL   // context
);

// Cancel timer
app_timer_cancel(timer);
```

#### 5. Animation Framework

```c
// Create animation
Animation *anim = animation_create();

// Configure
animation_set_duration(anim, 250);
animation_set_curve(anim, AnimationCurveEaseOut);

// Set handlers
animation_set_handlers(anim, (AnimationHandlers) {
    .started = prv_anim_started,
    .stopped = prv_anim_stopped,
}, context);

// Schedule
animation_schedule(anim);
```

### Architectural Principles

1. **Task Isolation**: Bluetooth, App, and Kernel tasks are isolated with MPU
2. **Event-Driven**: All inter-task communication via FreeRTOS queues
3. **Layered Architecture**: Hardware → Drivers → Services → Applications
4. **Memory Safety**: Stack guards, MPU regions, watchdog timers
5. **Graceful Degradation**: Apps get up to 3 seconds for cleanup
6. **Power Efficiency**: Event-driven (not polling), stop mode support

---

## Summary Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          PebbleOS ARCHITECTURE                              │
└─────────────────────────────────────────────────────────────────────────────┘

                         FREERTOS REAL-TIME KERNEL
    ┌──────────────────────────────────────────────────────────────────────┐
    │                                                                      │
    │  App Task          Worker Task      KernelMain Task   BT Tasks      │
    │   (User App)       (Background)     (Launcher/Events) (3 tasks)     │
    │                                                                      │
    │  ├─ Window Stack    ├─ Timers       ├─ Event Loop    ├─ Host       │
    │  ├─ Graphics        ├─ Services     ├─ App Mgmt      ├─ Ctrl       │
    │  └─ Events          └─ Events       ├─ Button Mgmt   └─ HCI        │
    │                                     └─ Services                     │
    └──────────────────────────────────────────────────────────────────────┘
                           ↕ (Events via FreeRTOS Queues)
    ┌──────────────────────────────────────────────────────────────────────┐
    │                    EVENT SYSTEM (Kernel)                             │
    │  ├─ Hardware: Buttons, Accelerometer, Battery                       │
    │  ├─ System: Timers, Alerts, Notifications                          │
    │  ├─ Communication: Bluetooth, App messages                         │
    │  └─ Graphics: Render requests, Render complete                     │
    └──────────────────────────────────────────────────────────────────────┘
                                     ↕
    ┌──────────────────────────────────────────────────────────────────────┐
    │                    GRAPHICS PIPELINE                                 │
    │  App → Window Stack → Layer Tree → Compositor → Display             │
    └──────────────────────────────────────────────────────────────────────┘
                                     ↕
    ┌──────────────────────────────────────────────────────────────────────┐
    │                    HARDWARE DRIVERS                                  │
    │  Display │ IMU │ Battery │ Backlight │ Flash │ BLE │ Buttons │ ... │
    └──────────────────────────────────────────────────────────────────────┘
                                     ↕
    ┌──────────────────────────────────────────────────────────────────────┐
    │               HARDWARE (STM32, NRF52, SF32LB52, etc.)                │
    └──────────────────────────────────────────────────────────────────────┘
```
