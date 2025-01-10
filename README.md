# Simple and Efficient Ring Buffer Implementation

A high-performance ring buffer implementation specifically designed for embedded systems, featuring cache-aware memory layout and DMA compatibility.

## Key Features
- Cache-aligned memory structure
- DMA-compatible buffer organization
- Lock-free operations for multi-threading
- Static allocation support
- Inline function optimization
- SIMD-friendly memory layout

## Data Structure

```c
#define CACHE_LINE_SIZE 64

typedef struct {
    // Control structure - Aligned to cache line
    struct {
        _Atomic size_t head __attribute__((aligned(CACHE_LINE_SIZE)));
        _Atomic size_t tail;
        const size_t capacity;
        const size_t element_size;    // Size of stored element type
    } control;
    
    // Generic buffer pointer - Aligned to cache line
    void* buffer __attribute__((aligned(CACHE_LINE_SIZE)));
} RingBuffer;

// Memory allocation helper with alignment consideration
#define RING_BUFFER_SIZE(capacity, type) \
    (sizeof(RingBuffer) + ((capacity * sizeof(type) + CACHE_LINE_SIZE - 1) \
    & ~(CACHE_LINE_SIZE - 1)))  // Round up to cache line size
```

## Core API

```c
// Basic initialization
#define RING_BUFFER_INIT(name, type, size) \
    static uint8_t name##_storage[RING_BUFFER_SIZE(size, type)] \
    __attribute__((aligned(CACHE_LINE_SIZE))); \
    RingBuffer* name = ring_buffer_init(name##_storage, size, sizeof(type))

// Core operations
__attribute__((always_inline)) inline RingBuffer* 
ring_buffer_init(void* memory, size_t capacity, size_t element_size);

__attribute__((always_inline)) inline bool 
ring_buffer_push(RingBuffer* rb, const void* data);

__attribute__((always_inline)) inline bool 
ring_buffer_pop(RingBuffer* rb, void* data);

// Helper macros for type safety
#define PUSH_VALUE(rb, type, value) do { \
    type temp = value; \
    ring_buffer_push(rb, &temp); \
} while(0)

#define POP_VALUE(rb, type, ptr) \
    ring_buffer_pop(rb, ptr)
```

## Usage Examples

```c
// Integer buffer example
RING_BUFFER_INIT(int_buffer, int, 1024);
int value = 42;
PUSH_VALUE(int_buffer, int, value);
int result;
POP_VALUE(int_buffer, int, &result);

// Character buffer example
RING_BUFFER_INIT(char_buffer, char, 256);
char c = 'A';
PUSH_VALUE(char_buffer, char, c);
char out_char;
POP_VALUE(char_buffer, char, &out_char);

// Direct void* usage
void* data_ptr = &value;
ring_buffer_push(int_buffer, data_ptr);
```

## Memory Management

- Buffer memory is cache-line aligned
- Simple memory layout for direct DMA access
- Zero-copy design for basic types
- No dynamic memory allocation

## Thread Safety
- Atomic operations for head and tail pointers
- Lock-free implementation for single producer/consumer scenarios
- Memory barriers for multi-core synchronization

## Performance Characteristics
- O(1) push/pop operations
- Cache-miss minimization through alignment
- Zero allocation during operation
- DMA-friendly memory layout
- Atomic operations without locks

For detailed implementation and benchmark results, see the source code documentation.
