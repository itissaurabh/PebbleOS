# Tutorial 1.3: Computer Science for Embedded Health Devices

## Learning Objectives

By the end of this tutorial, you will:
- Understand embedded systems constraints and real-time requirements
- Know essential data structures for signal processing
- Implement efficient algorithms for resource-constrained devices
- Handle memory management in embedded C
- Design state machines for health monitoring

## Prerequisites
- Basic programming in C and Python
- Understanding of basic data types
- Completed Tutorials 1.1 and 1.2

---

## Table of Contents

1. [Embedded Systems Fundamentals](#1-embedded-systems-fundamentals)
2. [Data Structures for Signal Processing](#2-data-structures-for-signal-processing)
3. [Algorithm Complexity and Optimization](#3-algorithm-complexity-and-optimization)
4. [Memory Management](#4-memory-management)
5. [Real-Time Systems](#5-real-time-systems)
6. [State Machines](#6-state-machines)
7. [Learning Resources](#7-learning-resources)

---

## 1. Embedded Systems Fundamentals

### 1.1 What Makes Embedded Different?

```
    ┌─────────────────────────────────────────────────────────┐
    │           DESKTOP vs. EMBEDDED COMPARISON               │
    ├─────────────────────┬───────────────────────────────────┤
    │ Resource            │ Desktop        │ Smartwatch      │
    ├─────────────────────┼────────────────┼─────────────────┤
    │ CPU Speed           │ 3-5 GHz        │ 50-200 MHz      │
    │ RAM                 │ 8-64 GB        │ 128-512 KB      │
    │ Storage             │ 500+ GB        │ 1-16 MB         │
    │ Power Source        │ Wall outlet    │ 200 mAh battery │
    │ Battery Life        │ N/A            │ Days to weeks   │
    │ OS                  │ Full OS        │ RTOS or bare    │
    │ Floating Point      │ Hardware FPU   │ Often software  │
    └─────────────────────┴────────────────┴─────────────────┘

    Implications for algorithm design:
    - Must minimize memory usage
    - Avoid unnecessary computation
    - Consider fixed-point arithmetic
    - Design for deterministic execution time
```

### 1.2 Microcontroller Architecture

```
    Typical MCU Architecture (ARM Cortex-M)

    ┌─────────────────────────────────────────────────────────┐
    │                    MICROCONTROLLER                      │
    ├─────────────────────────────────────────────────────────┤
    │                                                         │
    │   ┌─────────┐   ┌─────────────┐   ┌─────────────┐      │
    │   │   CPU   │   │    FLASH    │   │    SRAM     │      │
    │   │ Cortex-M│   │  (Program)  │   │   (Data)    │      │
    │   │  Core   │   │   256 KB    │   │    64 KB    │      │
    │   └────┬────┘   └──────┬──────┘   └──────┬──────┘      │
    │        │               │                 │              │
    │        └───────────────┼─────────────────┘              │
    │                        │ BUS                            │
    │   ┌────────────────────┼────────────────────────────┐   │
    │   │                    │                            │   │
    │   ▼        ▼           ▼          ▼         ▼       │   │
    │ ┌────┐  ┌────┐     ┌──────┐  ┌──────┐  ┌──────┐    │   │
    │ │GPIO│  │ADC │     │TIMER │  │ UART │  │ SPI  │    │   │
    │ └────┘  └────┘     └──────┘  └──────┘  └──────┘    │   │
    │   │        │           │          │         │       │   │
    └───┼────────┼───────────┼──────────┼─────────┼───────┘   │
        │        │           │          │         │           │
        ▼        ▼           ▼          ▼         ▼           │
      LEDs    Sensors     Timing    Debug     Wireless        │
      Buttons             PWM       Console   Chip            │
                                                              │
    └─────────────────────────────────────────────────────────┘
```

### 1.3 Memory Map

```c
/*
 * Typical ARM Cortex-M Memory Layout
 *
 *    Address         Region          Description
 *    ─────────────────────────────────────────────────
 *    0x0000_0000    FLASH           Program code, constants
 *    0x2000_0000    SRAM            Variables, stack, heap
 *    0x4000_0000    Peripherals     GPIO, ADC, UART registers
 *    0xE000_0000    System          NVIC, SysTick, debug
 */

// Example: Accessing an LED via GPIO peripheral
#define GPIO_BASE       0x40020000
#define GPIO_ODR_OFFSET 0x14        // Output data register

// Direct register access
volatile uint32_t *gpio_odr = (uint32_t *)(GPIO_BASE + GPIO_ODR_OFFSET);
*gpio_odr |= (1 << 5);  // Set pin 5 high

// Better: Use vendor-provided headers
// #include "stm32f4xx.h"
// GPIOA->ODR |= GPIO_ODR_OD5;
```

### 1.4 Interrupt-Driven Design

Embedded systems are **interrupt-driven**, not polling-based:

```c
/*
 * Interrupt vs. Polling
 *
 * POLLING (inefficient):
 *    while (1) {
 *        if (button_pressed()) {
 *            handle_button();
 *        }
 *        // CPU wastes cycles checking
 *    }
 *
 * INTERRUPT (efficient):
 *    CPU sleeps, wakes only when needed
 *    Hardware signals when event occurs
 */

// Interrupt Service Routine (ISR)
volatile bool data_ready = false;
volatile int16_t accel_data[3];

void ACCEL_IRQHandler(void) {
    // This function is called by hardware when accelerometer
    // has new data (typically at 25-100 Hz)

    // Read data from sensor (very short operation)
    accel_data[0] = read_accel_x();
    accel_data[1] = read_accel_y();
    accel_data[2] = read_accel_z();

    // Signal main loop
    data_ready = true;

    // Clear interrupt flag
    ACCEL->STATUS |= ACCEL_STATUS_DRDY;
}

// Main loop
int main(void) {
    init_accelerometer();
    enable_accel_interrupt();

    while (1) {
        if (data_ready) {
            data_ready = false;
            process_accel_data(accel_data);
        }

        // CPU can sleep here, saving power
        __WFI();  // Wait For Interrupt
    }
}
```

---

## 2. Data Structures for Signal Processing

### 2.1 Circular Buffers (Ring Buffers)

The most important data structure for real-time signal processing:

```
    Circular Buffer Concept

    Linear buffer problem:
    ┌─────────────────────────────────────┐
    │ Old data we don't need anymore      │ New data →
    └─────────────────────────────────────┘
    Must shift entire buffer left to make room!

    Circular buffer solution:
         write
           ↓
    ┌───┬───┬───┬───┬───┬───┬───┬───┐
    │ 5 │ 6 │ 7 │ 8 │ 1 │ 2 │ 3 │ 4 │ ← physical memory
    └───┴───┴───┴───┴───┴───┴───┴───┘
                       ↑
                     read

    Logical order: 1, 2, 3, 4, 5, 6, 7, 8
    Just update pointers, no data movement!
```

```c
// Circular buffer implementation
typedef struct {
    int16_t *buffer;
    size_t size;
    size_t head;  // Write position
    size_t tail;  // Read position
    size_t count; // Number of elements
} CircularBuffer;

void circular_buffer_init(CircularBuffer *cb, int16_t *buf, size_t size) {
    cb->buffer = buf;
    cb->size = size;
    cb->head = 0;
    cb->tail = 0;
    cb->count = 0;
}

bool circular_buffer_push(CircularBuffer *cb, int16_t value) {
    if (cb->count >= cb->size) {
        return false;  // Buffer full
    }

    cb->buffer[cb->head] = value;
    cb->head = (cb->head + 1) % cb->size;
    cb->count++;
    return true;
}

bool circular_buffer_pop(CircularBuffer *cb, int16_t *value) {
    if (cb->count == 0) {
        return false;  // Buffer empty
    }

    *value = cb->buffer[cb->tail];
    cb->tail = (cb->tail + 1) % cb->size;
    cb->count--;
    return true;
}

// Access element at logical index (0 = oldest)
int16_t circular_buffer_get(CircularBuffer *cb, size_t index) {
    size_t physical_index = (cb->tail + index) % cb->size;
    return cb->buffer[physical_index];
}

// Example usage for accelerometer data
#define BUFFER_SIZE 128

static int16_t accel_buffer_mem[BUFFER_SIZE];
static CircularBuffer accel_buffer;

void init_accel_buffer(void) {
    circular_buffer_init(&accel_buffer, accel_buffer_mem, BUFFER_SIZE);
}

void add_sample(int16_t sample) {
    // If buffer is full, this overwrites oldest data
    if (accel_buffer.count >= accel_buffer.size) {
        // Remove oldest
        int16_t dummy;
        circular_buffer_pop(&accel_buffer, &dummy);
    }
    circular_buffer_push(&accel_buffer, sample);
}
```

### 2.2 Fixed-Size Arrays for Features

```c
// Feature storage for activity recognition
typedef struct {
    float mean[3];           // Mean for X, Y, Z axes
    float std[3];            // Standard deviation
    float correlation[3];    // XY, XZ, YZ correlation
    float dominant_freq;     // From FFT
    float spectral_energy;
} AccelFeatures;

// Compute features from circular buffer
void compute_features(CircularBuffer *x, CircularBuffer *y, CircularBuffer *z,
                      AccelFeatures *features) {
    int n = x->count;

    // Mean
    float sum_x = 0, sum_y = 0, sum_z = 0;
    for (int i = 0; i < n; i++) {
        sum_x += circular_buffer_get(x, i);
        sum_y += circular_buffer_get(y, i);
        sum_z += circular_buffer_get(z, i);
    }
    features->mean[0] = sum_x / n;
    features->mean[1] = sum_y / n;
    features->mean[2] = sum_z / n;

    // Standard deviation
    float var_x = 0, var_y = 0, var_z = 0;
    for (int i = 0; i < n; i++) {
        float dx = circular_buffer_get(x, i) - features->mean[0];
        float dy = circular_buffer_get(y, i) - features->mean[1];
        float dz = circular_buffer_get(z, i) - features->mean[2];
        var_x += dx * dx;
        var_y += dy * dy;
        var_z += dz * dz;
    }
    features->std[0] = sqrtf(var_x / (n - 1));
    features->std[1] = sqrtf(var_y / (n - 1));
    features->std[2] = sqrtf(var_z / (n - 1));
}
```

### 2.3 Queue for Events

```c
// Event queue for health monitoring
typedef enum {
    EVENT_STEP_DETECTED,
    EVENT_HEART_BEAT,
    EVENT_SLEEP_STATE_CHANGE,
    EVENT_HIGH_STRESS,
    EVENT_LOW_GLUCOSE
} EventType;

typedef struct {
    EventType type;
    uint32_t timestamp;
    int32_t data;
} HealthEvent;

#define EVENT_QUEUE_SIZE 32

typedef struct {
    HealthEvent events[EVENT_QUEUE_SIZE];
    size_t head;
    size_t tail;
    size_t count;
} EventQueue;

void event_queue_push(EventQueue *q, EventType type, int32_t data) {
    if (q->count >= EVENT_QUEUE_SIZE) {
        // Queue full - could log warning
        return;
    }

    q->events[q->head].type = type;
    q->events[q->head].timestamp = get_system_time_ms();
    q->events[q->head].data = data;

    q->head = (q->head + 1) % EVENT_QUEUE_SIZE;
    q->count++;
}

bool event_queue_pop(EventQueue *q, HealthEvent *event) {
    if (q->count == 0) {
        return false;
    }

    *event = q->events[q->tail];
    q->tail = (q->tail + 1) % EVENT_QUEUE_SIZE;
    q->count--;
    return true;
}
```

---

## 3. Algorithm Complexity and Optimization

### 3.1 Big-O Notation

Understanding computational complexity is critical for embedded systems:

```
    Big-O Complexity

    ┌────────────┬─────────────────────────────────────────────┐
    │ Complexity │ Description              │ Example           │
    ├────────────┼─────────────────────────┼───────────────────┤
    │ O(1)       │ Constant time           │ Array access      │
    │ O(log n)   │ Logarithmic             │ Binary search     │
    │ O(n)       │ Linear                  │ Sum of array      │
    │ O(n log n) │ Linearithmic            │ FFT, merge sort   │
    │ O(n²)      │ Quadratic               │ Naive convolution │
    │ O(2^n)     │ Exponential             │ Brute force       │
    └────────────┴─────────────────────────┴───────────────────┘

    For 128-sample window:
    ┌────────────┬─────────────────┐
    │ O(n)       │ 128 operations  │
    │ O(n log n) │ ~896 operations │
    │ O(n²)      │ 16,384 ops      │ ← Too slow for real-time
    └────────────┴─────────────────┘
```

### 3.2 Optimizing Common Operations

```c
// SLOW: Naive convolution O(n*m)
void convolve_naive(float *x, int nx, float *h, int nh, float *y) {
    for (int n = 0; n < nx + nh - 1; n++) {
        y[n] = 0;
        for (int k = 0; k < nh; k++) {
            if (n - k >= 0 && n - k < nx) {
                y[n] += h[k] * x[n - k];
            }
        }
    }
}

// FASTER: Optimized with loop bounds
void convolve_optimized(float *x, int nx, float *h, int nh, float *y) {
    // Pre-compute loop bounds to avoid inner conditionals
    for (int n = 0; n < nx + nh - 1; n++) {
        y[n] = 0;
        int k_min = (n >= nx) ? n - nx + 1 : 0;
        int k_max = (n < nh - 1) ? n + 1 : nh;

        for (int k = k_min; k < k_max; k++) {
            y[n] += h[k] * x[n - k];
        }
    }
}

// FASTEST: Use FFT for long convolutions O(n log n)
// y = IFFT(FFT(x) * FFT(h))
// More efficient when nh > ~32 for typical embedded FFT implementations
```

### 3.3 Fixed-Point Optimization

```c
// Floating-point (slow without FPU)
float filter_float(float *coeffs, float *state, float input, int order) {
    float output = coeffs[0] * input;
    for (int i = 1; i <= order; i++) {
        output += coeffs[i] * state[i-1];
    }
    // Update state (shift right)
    for (int i = order - 1; i > 0; i--) {
        state[i] = state[i-1];
    }
    state[0] = input;
    return output;
}

// Fixed-point Q15 (fast on all MCUs)
int16_t filter_q15(int16_t *coeffs, int16_t *state, int16_t input, int order) {
    int32_t output = (int32_t)coeffs[0] * input;

    for (int i = 1; i <= order; i++) {
        output += (int32_t)coeffs[i] * state[i-1];
    }

    // Shift down by 15 bits (Q15 format)
    output = (output + (1 << 14)) >> 15;  // Round

    // Saturate to Q15 range
    if (output > 32767) output = 32767;
    if (output < -32768) output = -32768;

    // Update state
    for (int i = order - 1; i > 0; i--) {
        state[i] = state[i-1];
    }
    state[0] = input;

    return (int16_t)output;
}
```

### 3.4 Loop Unrolling

```c
// Standard loop
void vector_add(float *a, float *b, float *c, int n) {
    for (int i = 0; i < n; i++) {
        c[i] = a[i] + b[i];
    }
}

// Unrolled by 4 (fewer loop iterations, better pipelining)
void vector_add_unrolled(float *a, float *b, float *c, int n) {
    int i;
    // Main loop - process 4 elements at a time
    for (i = 0; i < n - 3; i += 4) {
        c[i]     = a[i]     + b[i];
        c[i + 1] = a[i + 1] + b[i + 1];
        c[i + 2] = a[i + 2] + b[i + 2];
        c[i + 3] = a[i + 3] + b[i + 3];
    }
    // Handle remaining elements
    for (; i < n; i++) {
        c[i] = a[i] + b[i];
    }
}
```

---

## 4. Memory Management

### 4.1 Stack vs. Heap

```c
/*
 * Memory Regions in Embedded C
 *
 *    ┌─────────────────────────────┐ High address
 *    │         STACK               │ ← Local variables, grows down
 *    │            ↓                │
 *    │                             │
 *    │            ↑                │
 *    │          HEAP               │ ← Dynamic allocation, grows up
 *    ├─────────────────────────────┤
 *    │          BSS                │ ← Uninitialized global/static
 *    ├─────────────────────────────┤
 *    │          DATA               │ ← Initialized global/static
 *    ├─────────────────────────────┤
 *    │          TEXT               │ ← Program code (in Flash)
 *    └─────────────────────────────┘ Low address
 */

// Stack allocation (automatic, fast, limited size)
void process_data(void) {
    int16_t local_buffer[128];  // On stack - auto-freed when function returns
    // ...
}

// Heap allocation (dynamic, slower, can fragment)
void dynamic_allocation(void) {
    int16_t *buffer = malloc(128 * sizeof(int16_t));
    if (buffer == NULL) {
        // Handle allocation failure!
    }
    // ...
    free(buffer);  // Must explicitly free!
}

// Static allocation (persistent, in BSS/DATA)
static int16_t persistent_buffer[128];  // Lives for entire program lifetime
```

### 4.2 Memory Pools

Avoid heap fragmentation with pre-allocated pools:

```c
// Memory pool for fixed-size objects
#define POOL_SIZE 16
#define OBJECT_SIZE sizeof(HealthEvent)

typedef struct {
    uint8_t memory[POOL_SIZE * OBJECT_SIZE];
    bool used[POOL_SIZE];
} MemoryPool;

static MemoryPool event_pool;

void pool_init(MemoryPool *pool) {
    memset(pool->used, 0, POOL_SIZE);
}

void *pool_alloc(MemoryPool *pool) {
    for (int i = 0; i < POOL_SIZE; i++) {
        if (!pool->used[i]) {
            pool->used[i] = true;
            return &pool->memory[i * OBJECT_SIZE];
        }
    }
    return NULL;  // Pool exhausted
}

void pool_free(MemoryPool *pool, void *ptr) {
    if (ptr == NULL) return;

    // Calculate index from pointer
    int index = ((uint8_t *)ptr - pool->memory) / OBJECT_SIZE;
    if (index >= 0 && index < POOL_SIZE) {
        pool->used[index] = false;
    }
}
```

### 4.3 Avoiding Dynamic Allocation

Best practice: **allocate everything statically** at compile time:

```c
// Bad: Dynamic allocation
int16_t *create_fft_buffer(int size) {
    return malloc(size * sizeof(int16_t));  // May fail, fragments heap
}

// Good: Static allocation with compile-time size
#define FFT_SIZE 128
static int16_t fft_buffer[FFT_SIZE];

int16_t *get_fft_buffer(void) {
    return fft_buffer;  // Always succeeds, no fragmentation
}

// Good: Caller-provided buffer
void compute_fft(int16_t *input, int16_t *output, int16_t *work_buffer, int size) {
    // Caller manages memory, function just uses it
}
```

---

## 5. Real-Time Systems

### 5.1 Real-Time Operating Systems (RTOS)

```
    RTOS Concepts

    ┌─────────────────────────────────────────────────────────┐
    │                   RTOS SCHEDULER                        │
    └─────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
              ▼               ▼               ▼
    ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
    │   High Priority │ │  Medium Priority│ │   Low Priority  │
    │   Sensor Task   │ │   Process Task  │ │   Display Task  │
    │                 │ │                 │ │                 │
    │   Period: 10ms  │ │   Period: 100ms │ │   Period: 1s    │
    │   Deadline: 10ms│ │   Deadline:100ms│ │   Deadline: 1s  │
    └─────────────────┘ └─────────────────┘ └─────────────────┘

    Priority rules:
    - Higher priority task preempts lower priority
    - Same priority: Round-robin scheduling
    - Task must complete before deadline
```

### 5.2 FreeRTOS Task Example

```c
// FreeRTOS task example (PebbleOS uses FreeRTOS)
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"

// Queue for sensor data
static QueueHandle_t sensor_queue;

// High-priority sensor reading task
void sensor_task(void *pvParameters) {
    int16_t accel_sample;

    while (1) {
        // Read accelerometer (runs every 40ms = 25 Hz)
        accel_sample = read_accelerometer_z();

        // Send to processing queue (non-blocking)
        xQueueSend(sensor_queue, &accel_sample, 0);

        // Wait until next sample period
        vTaskDelay(pdMS_TO_TICKS(40));
    }
}

// Lower-priority processing task
void processing_task(void *pvParameters) {
    int16_t sample;
    static int16_t buffer[128];
    static int buffer_idx = 0;

    while (1) {
        // Wait for data from sensor task
        if (xQueueReceive(sensor_queue, &sample, portMAX_DELAY) == pdTRUE) {
            buffer[buffer_idx++] = sample;

            // Process when buffer is full
            if (buffer_idx >= 128) {
                detect_steps(buffer, 128);
                buffer_idx = 0;
            }
        }
    }
}

// Initialization
void init_tasks(void) {
    sensor_queue = xQueueCreate(32, sizeof(int16_t));

    xTaskCreate(sensor_task, "Sensor", 256, NULL, 3, NULL);      // High priority
    xTaskCreate(processing_task, "Process", 512, NULL, 2, NULL); // Medium priority

    vTaskStartScheduler();
}
```

### 5.3 Timing Analysis

```c
// Measuring execution time
#include <stdint.h>

// ARM Cortex-M cycle counter
#define DWT_CYCCNT (*((volatile uint32_t *)0xE0001004))

void enable_cycle_counter(void) {
    // Enable DWT cycle counter
    *((volatile uint32_t *)0xE0001000) |= 1;
}

uint32_t measure_execution_time(void (*func)(void)) {
    uint32_t start = DWT_CYCCNT;
    func();
    uint32_t end = DWT_CYCCNT;
    return end - start;  // Cycles elapsed
}

// Convert cycles to microseconds (example: 64 MHz CPU)
#define CPU_FREQ_HZ 64000000
#define CYCLES_TO_US(cycles) ((cycles) * 1000000 / CPU_FREQ_HZ)

void benchmark_fft(void) {
    enable_cycle_counter();

    uint32_t cycles = measure_execution_time(run_fft_128);
    uint32_t us = CYCLES_TO_US(cycles);

    printf("FFT-128: %lu cycles = %lu us\n", cycles, us);
}
```

### 5.4 Deadline Scheduling

```c
// Ensuring algorithms meet real-time deadlines
typedef struct {
    const char *name;
    uint32_t period_ms;      // How often task runs
    uint32_t deadline_ms;    // Must complete within this
    uint32_t worst_case_us;  // Measured worst-case execution
} TaskTiming;

// Example timing analysis
TaskTiming system_tasks[] = {
    {"Accel Read",    40,    40,    50},    // 25 Hz sampling
    {"Step Detect",   5000,  5000,  2000},  // Every 5 seconds
    {"HR Calc",       1000,  1000,  500},   // Every second
    {"Display",       1000,  1000,  5000},  // Every second (low priority)
};

// Utilization analysis
float calculate_cpu_utilization(TaskTiming *tasks, int n) {
    float utilization = 0;
    for (int i = 0; i < n; i++) {
        utilization += (float)tasks[i].worst_case_us /
                       (tasks[i].period_ms * 1000.0f);
    }
    return utilization;
}

// Rule of thumb: Keep utilization < 70% for real-time systems
```

---

## 6. State Machines

### 6.1 Finite State Machine Concept

State machines are ideal for health monitoring modes:

```
    Sleep Detection State Machine

    ┌─────────────────────────────────────────────────────────┐
    │                                                         │
    │      ┌─────────────────────────────────────────┐        │
    │      │                                         │        │
    │      ▼                                         │        │
    │   ┌──────┐    high VMC     ┌────────┐         │        │
    │   │ WAKE │ ───────────────→│MAYBE   │─────────┘        │
    │   │      │ ←─────────────── │AWAKE   │ 3 min active     │
    │   └──┬───┘    timeout      └────────┘                  │
    │      │                                                  │
    │      │ low VMC > 5 min                                  │
    │      ▼                                                  │
    │   ┌──────┐                                              │
    │   │SLEEP │ ←─────────────────────────────────┐         │
    │   │      │                                   │         │
    │   └──┬───┘                                   │         │
    │      │                                       │         │
    │      │ very low VMC > 20 min                 │         │
    │      ▼                                       │         │
    │   ┌──────┐    movement detected              │         │
    │   │DEEP  │ ──────────────────────────────────┘         │
    │   │SLEEP │                                              │
    │   └──────┘                                              │
    │                                                         │
    └─────────────────────────────────────────────────────────┘
```

### 6.2 State Machine Implementation

```c
// State enumeration
typedef enum {
    STATE_WAKE,
    STATE_MAYBE_AWAKE,
    STATE_SLEEP,
    STATE_DEEP_SLEEP
} SleepState;

// Event enumeration
typedef enum {
    EVENT_HIGH_ACTIVITY,
    EVENT_LOW_ACTIVITY,
    EVENT_VERY_LOW_ACTIVITY,
    EVENT_TIMER_EXPIRED
} SleepEvent;

// State machine context
typedef struct {
    SleepState current_state;
    uint32_t state_entry_time;
    uint32_t activity_sum;
} SleepStateMachine;

// State transition function
void sleep_sm_process_event(SleepStateMachine *sm, SleepEvent event) {
    SleepState next_state = sm->current_state;

    switch (sm->current_state) {
        case STATE_WAKE:
            if (event == EVENT_LOW_ACTIVITY) {
                // Check if low activity for 5 minutes
                if (get_time_ms() - sm->state_entry_time > 5 * 60 * 1000) {
                    next_state = STATE_SLEEP;
                }
            }
            break;

        case STATE_MAYBE_AWAKE:
            if (event == EVENT_HIGH_ACTIVITY) {
                if (get_time_ms() - sm->state_entry_time > 3 * 60 * 1000) {
                    next_state = STATE_WAKE;
                }
            } else if (event == EVENT_TIMER_EXPIRED) {
                next_state = STATE_SLEEP;
            }
            break;

        case STATE_SLEEP:
            if (event == EVENT_HIGH_ACTIVITY) {
                next_state = STATE_MAYBE_AWAKE;
            } else if (event == EVENT_VERY_LOW_ACTIVITY) {
                if (get_time_ms() - sm->state_entry_time > 20 * 60 * 1000) {
                    next_state = STATE_DEEP_SLEEP;
                }
            }
            break;

        case STATE_DEEP_SLEEP:
            if (event == EVENT_HIGH_ACTIVITY || event == EVENT_LOW_ACTIVITY) {
                next_state = STATE_SLEEP;
            }
            break;
    }

    // Handle state transition
    if (next_state != sm->current_state) {
        // Exit actions for old state
        on_state_exit(sm, sm->current_state);

        // Update state
        sm->current_state = next_state;
        sm->state_entry_time = get_time_ms();

        // Entry actions for new state
        on_state_enter(sm, next_state);
    }
}

void on_state_enter(SleepStateMachine *sm, SleepState state) {
    switch (state) {
        case STATE_SLEEP:
            log_event("Sleep started");
            break;
        case STATE_DEEP_SLEEP:
            log_event("Deep sleep started");
            reduce_sampling_rate();  // Save power
            break;
        default:
            break;
    }
}
```

### 6.3 Table-Driven State Machine

For complex state machines, use a transition table:

```c
// State transition table
typedef struct {
    SleepState current;
    SleepEvent event;
    SleepState next;
    void (*action)(void);
} Transition;

// Transition table
static const Transition transitions[] = {
    {STATE_WAKE,        EVENT_LOW_ACTIVITY,      STATE_SLEEP,       start_sleep_timer},
    {STATE_SLEEP,       EVENT_HIGH_ACTIVITY,     STATE_MAYBE_AWAKE, start_wake_timer},
    {STATE_SLEEP,       EVENT_VERY_LOW_ACTIVITY, STATE_DEEP_SLEEP,  reduce_power},
    {STATE_MAYBE_AWAKE, EVENT_HIGH_ACTIVITY,     STATE_WAKE,        wake_up},
    {STATE_MAYBE_AWAKE, EVENT_TIMER_EXPIRED,     STATE_SLEEP,       NULL},
    {STATE_DEEP_SLEEP,  EVENT_HIGH_ACTIVITY,     STATE_SLEEP,       normal_power},
    // ... more transitions
};

#define NUM_TRANSITIONS (sizeof(transitions) / sizeof(transitions[0]))

void process_event_table(SleepStateMachine *sm, SleepEvent event) {
    for (int i = 0; i < NUM_TRANSITIONS; i++) {
        if (transitions[i].current == sm->current_state &&
            transitions[i].event == event) {

            // Execute action if defined
            if (transitions[i].action != NULL) {
                transitions[i].action();
            }

            // Transition to new state
            sm->current_state = transitions[i].next;
            return;
        }
    }
    // No matching transition - stay in current state
}
```

---

## 7. Learning Resources

### 7.1 Books

#### Embedded Systems
| Book | Author | Level | Focus |
|------|--------|-------|-------|
| **"Making Embedded Systems"** | Elecia White | Beginner | Practical embedded design |
| **"Programming Embedded Systems in C and C++"** | Michael Barr | Beginner | C for embedded |
| **"Real-Time Concepts for Embedded Systems"** | Li & Yao | Intermediate | RTOS fundamentals |
| **"The Definitive Guide to ARM Cortex-M"** | Joseph Yiu | Intermediate | ARM MCU architecture |

#### Algorithms and Data Structures
| Book | Author | Level | Focus |
|------|--------|-------|-------|
| **"Introduction to Algorithms"** | CLRS | Intermediate | Comprehensive algorithms |
| **"Algorithms"** | Sedgewick | Intermediate | Practical algorithms |
| **"Numerical Recipes in C"** | Press et al. | Advanced | Scientific computing |

### 7.2 Online Courses

| Course | Platform | Link |
|--------|----------|------|
| **Embedded Systems Shape the World** | edX (UT Austin) | [edx.org](https://www.edx.org/course/embedded-systems-shape-the-world) |
| **Real-Time Bluetooth Networks** | edX (UT Austin) | [edx.org](https://www.edx.org/course/real-time-bluetooth-networks) |
| **Introduction to RTOS** | Digi-Key (YouTube) | [youtube.com](https://www.youtube.com/playlist?list=PLEBQazB0HUyQ4hAPU1cJED6t3DU0h34bN) |
| **FreeRTOS Tutorial** | FreeRTOS | [freertos.org](https://www.freertos.org/Documentation/RTOS_book.html) |

### 7.3 YouTube Channels

| Channel | Focus | Best For |
|---------|-------|----------|
| **Jacob Sorber** | Embedded C, systems | Low-level programming |
| **Phil's Lab** | Hardware and firmware | PCB design, DSP |
| **Ben Eater** | Computer architecture | Understanding hardware |
| **EEVblog** | Electronics | Practical electronics |
| **Digi-Key** | Embedded tutorials | Component-level |

### 7.4 Development Tools

| Tool | Purpose | Link |
|------|---------|------|
| **ARM GCC** | Cross-compiler | [developer.arm.com](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain) |
| **OpenOCD** | Debug interface | [openocd.org](http://openocd.org/) |
| **STM32CubeIDE** | ST development | [st.com](https://www.st.com/en/development-tools/stm32cubeide.html) |
| **PlatformIO** | Multi-platform | [platformio.org](https://platformio.org/) |
| **Valgrind** | Memory debugging | [valgrind.org](https://valgrind.org/) |

### 7.5 Documentation and References

| Resource | Description | Link |
|----------|-------------|------|
| **ARM Documentation** | Cortex-M reference | [developer.arm.com](https://developer.arm.com/) |
| **FreeRTOS Reference** | RTOS API | [freertos.org](https://www.freertos.org/a00106.html) |
| **C Reference** | Language reference | [cppreference.com](https://en.cppreference.com/w/c) |
| **CMSIS** | ARM standard APIs | [arm-software.github.io](https://arm-software.github.io/CMSIS_5/) |

---

## Summary

This tutorial covered essential computer science concepts for embedded health devices:

| Topic | Key Concepts | Application |
|-------|--------------|-------------|
| **Embedded Fundamentals** | MCU architecture, interrupts | Hardware interaction |
| **Data Structures** | Circular buffers, queues | Real-time signal storage |
| **Algorithm Complexity** | Big-O, optimization | Meeting timing requirements |
| **Memory Management** | Stack/heap, pools | Resource-constrained operation |
| **Real-Time Systems** | RTOS, scheduling | Reliable timing |
| **State Machines** | FSM design, transitions | Health monitoring modes |

**Key Takeaways**:
1. Embedded systems require careful resource management
2. Circular buffers are essential for real-time signal processing
3. Avoid dynamic allocation when possible
4. Measure and verify timing for real-time deadlines
5. State machines simplify complex monitoring logic

---

**Next Tutorial**: [2.1 Digital Signal Processing Basics →](../02_signal_processing/01_dsp_basics.md)
