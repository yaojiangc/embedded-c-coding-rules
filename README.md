# Embedded C Golden Rules

| Category | Rule | 🚫 Bad Example | ✅ Good Example |
|----------|------|----------------|----------------|
| **Data Types & Type Safety** |  |  |  |
|  | Use <stdint.h> fixed-width types | `int speed;` | `uint16_t speed_kmh;` |
|  | Always specify signedness | `short temp;` | `int16_t temp_c;` |
|  | Avoid implicit conversions (signed↔unsigned) | `uint8_t a = -1;` | `uint8_t a = (uint8_t)-1;` |
|  | Don’t rely on default char signedness | `char c;` | `signed char c;` / `unsigned char c;` |
|  | Mind endianness when sharing data (**[EXC-ENDIANNESS](#exc-endianness)**) | `memcpy(buf,&val,4);` | Manual pack/htonl()/defined wire format |
|  | Casting: downcast only with range checks | `u8 = (uint32_t)y;` | `if (y <= UINT8_MAX) u8 = (uint8_t)y;` |
| **Pointers, Memory & Performance** |  |  |  |
|  | Check pointers before deref | `*p = 5;` | `if (p) {*p = 5;}` |
|  | Avoid pointer arithmetic in app logic (**[EXC-POINTER-ARITH](#exc-pointer-arith)**) | `p += 5;` | Use indexing; pointer step only for HW tables |
|  | Avoid malloc in bare-metal (**[EXC-MALLOC](#exc-malloc)**) | `p = malloc(100);` | Static buffers/pools; RTOS heap w/ bounds |
|  | Watch stack usage; add guards | deep recursion | iterative / static scratch |
|  | Limit recursion (**[EXC-RECURSION](#exc-recursion)**) | quicksort on large data | iterative or bounded-depth with tests |
|  | Use const to place RO data in flash | `char s[]="Hi";` | `const char s[]="Hi";` |
|  | Align DMA buffers & handle cache | `misaligned DMA buf` | `__attribute__((aligned(32))) uint8_t dma_buf[...] ;` |
|  | Hoist invariants out of loops | `for(...) k=calc();` | precompute k outside the loop |
|  | Avoid expensive div/mod in hot paths | `/ 8` | `>> 3` (when exact) |
|  | Inline tiny hot helpers (**[EXC-INLINE](#exc-inline)**) | call tiny func in tight loop | `static inline` (verify size/speed)` |
|  | Minimize volatile traffic (**[EXC-VOLATILE](#exc-volatile)**) | read same reg repeatedly | cache once if safe per RM |
| **Functions, Modularity & Testability** |  |  |  |
|  | Keep functions small; single responsibility | long monolith | split helpers |
|  | Use static for internal linkage | `int helper()` | `static int helper()` |
|  | Pass dependencies; avoid hidden globals | uses global `sensor_val` | `foo(sensor_val)` |
|  | Prefer pure functions for logic | `calc(){return TIMER->CNT/f;}` | `calc(ticks){return ticks/f;}` |
|  | Separate I/O from logic | math inside driver hitting regs | driver does I/O; logic module is pure |
|  | Reentrant libs: no hidden state | static hidden buf | caller-owned ctx&buffers |
|  | Idempotent init/deinit | `init()` twice breaks | `if (!inited) init();` |
|  | Mark pointer params const where possible | `int f(uint8_t *p)` | `int f(const uint8_t *p)` |
|  | Document buffer ownership/lifetime | callee frees caller’s mem | caller owns, callee never frees |
| **Protocols & Data Handling** |  |  |  |
|  | Validate external input sizes | trust len | clamp len to buffer capacity |
|  | CRC/checksum persistent/comms data | none | verify before use |
|  | Encode units in names/types | `int temp;` | `int16_t temp_cdeg; // 0.01°C` |
|  | Serialize explicitly (no struct casts) | `send(&pkt,sizeof pkt);` | field-by-field pack; defined endianness |
|  | Heavy printf is expensive (**[EXC-PRINTF](#exc-printf)**) | `printf("%f",x);` | minimal logs; int print; deferred drain |
|  | Watch struct padding/ABI | send raw struct | packed/serialized layout |
| **Defensive Coding & Safety** |  |  |  |
|  | Validate inputs | `set_speed(speed);` | `if (speed <= MAX_SPEED) {set_speed(speed);}` |
|  | Use assert() in debug builds | `ptr->val = 1;` | `assert(ptr != NULL); ptr->val = 1;` |
|  | Pet watchdog only when system is healthy | `wdt_pet(); // everywhere` | `if (system_ok()) {wdt_pet();}` |
|  | Log reset cause at boot | — | `uint32_t cause = RCC->CSR; // store/log` |
|  | Use explicit status enums (no magic ints) | `return -3;` | `return STATUS_TIMEOUT;` |
|  | Fail fast: validate early and bail | `if (config_ok(cfg)) { init_hw(); start_services(); run_main_loop(); }` | `if (!config_ok(cfg)) { return ERR_CONFIG; } init_hw(); start_services(); run_main_loop(); return OK;` |
| **Interrupts & Concurrency** |  |  |  |
|  | Keep ISR short and non-blocking | `ISR { process(); delay_ms(10); }` | `ISR { set_flag(); } // handle in task` |
|  | Protect shared data (critical sections/atomics) | `shared++;` | `DISABLE_IRQ(); shared++; ENABLE_IRQ();` |
|  | Mark shared vars appropriately volatile | `uint8_t flag;` | `volatile uint8_t flag;` |
|  | Mark volatile only where needed (**[EXC-VOLATILE](#exc-volatile)**) | `volatile struct {...}` | `struct { volatile uint8_t status; uint8_t buf[8]; } s;` |
|  | Clear and document IRQ priority policy | `NVIC_SetPriority(IRQx, rand());` | `Policy doc + fixed priorities` |
|  | Always use timeouts on waits | `while (!flag);` | `wait_for_flag(1000); // ms` |
|  | Use memory barriers when needed (**[EXC-VOLATILE](#exc-volatile)**) | `flag = 1;` | `__DSB(); flag = 1;` |
|  | Double-buffer ISR↔main data | ISR writes into live buffer | Write into shadow; swap pointers atomically |
| **Bitwise & Low-Level Hardware Access** |  |  |  |
|  | Use named masks/macros, not magic numbers | `REG \|= 0x08;` | `REG \|= REG_TX_ENABLE;` |
|  | Parenthesize masks and shifts | `x = 1<<n+1;` | `x = (1u << (n+1));` |
|  | Check hardware status flags before I/O | `UART->DR = d;` | `while (!(UART->SR & TXE)){}; UART->DR = d;` |
|  | Debounce mechanical inputs | `if (btn)` | Filter/timeout before accept |
|  | Read volatile registers once if bits may change | `if (REG & A && REG & B)` | `const uint32_t v = REG; if (v & A && v & B)` |
|  | Write registers atomically (RMW with mask) | `REG = val;` | `REG = (REG & ~MASK) \| NEW_BITS;` |
|  | Mask interrupts for multi-byte peripheral updates | `Update low/high w/o masking` | `DISABLE_IRQ(); write_low(); write_high(); ENABLE_IRQ();` |
| **Initialization & Bring-Up** |  |  |  |
|  | Follow vendor-recommended init order for HW regs/clocks (**[EXC-HW-INIT-ORDER](#exc-hw-init-order)**) | `USART->BRR=...; RCC->ENR \|= USARTEN;` | `RCC->ENR \|= USARTEN; USART->BRR=...; // or pre-config then enable (e.g., PWM/ADC)` |
|  | Always initialize variables/structs | `uint8_t x; if (x) {...}` | `uint8_t x = 0;` |
|  | Initialize arrays fully | `int a[10];` | `int a[10] = {0};` |
| **Control Flow & Style** |  |  |  |
|  | Always use braces "{}" for if/else/loops (**[EXC-BRACES](#exc-braces)**) | `if (ok) do_it();` | `if (ok) { do_it(); }` |
|  | Avoid backward `goto`; forward `goto` only for structured cleanup | `start: if (err) goto start;` | `if (err) { goto cleanup; } cleanup: release();` |
|  | Avoid unmarked fall-through in switch (**[EXC-FALLTHROUGH](#exc-fallthrough)**) | `case 0: a(); case 1: b();` | `case 0: a(); __attribute__((fallthrough)); case 1: b();` |
|  | Always include default in switch | `switch(m){ case 0: ... }` | `switch(m){ case 0: ... default: handle_error(); }` |
|  | Limit function parameter count | `f(a,b,c,d,e,f);` | `f((struct Cfg){...});` |
|  | Use descriptive names; comment why, not what | `tmp; // inc` | `battery_mv; // compensates sensor offset` |
| **RTOS** |  |  |  |
|  | One responsibility per task | god task | dedicated tasks per concern |
|  | Use events/semaphores (no busy-wait) | `while(!flag);` | `osSemaphoreWait(..., timeout);` |
|  | Respect ISR-safe API subset | call blocking API in ISR | use *_FromISR APIs only |

---

## Exceptions

### EXC-HW-INIT-ORDER
Many MCUs require clock first, then config (e.g., STM32 USART). Some peripherals (PWM/ADC/DAC/comparators) may start output/sampling immediately on enable; pre-configure safe defaults before enabling to avoid glitches or unsafe pulses.

### EXC-POINTER-ARITH
Avoid in application logic for readability/safety. Acceptable for hardware register maps, DMA descriptor walking, or tight binary parsers when documented and reviewed.

### EXC-ENDIANNESS
Use a single, versioned wire format and pack/unpack with shifts or `hton*/ntoh*`. Don’t `memcpy` raw structs across interfaces.

### EXC-MALLOC
On bare-metal, prefer static/pool allocators. In RTOS systems, bounded heaps with fragmentation checks and allocation policy (startup-only, no free in real-time paths) can be OK.

### EXC-RECURSION
Only if you can prove bounded depth and stack headroom (watermarks/guards). Otherwise avoid.

### EXC-VOLATILE
Some regs require multi-read sequences (e.g., read-to-clear). Cache only when the reference manual permits; otherwise re-read per spec. Also use memory barriers where hardware ordering matters.

### EXC-PRINTF
Pulling in float formatting drags big libs and stalls timing. Prefer integer fixed-point logs, deferred draining, or ring-buffered logging.

### EXC-INLINE
Use `static inline` sparingly; verify code size vs. speed in the map file/metrics.

### EXC-BRACES
Always use `{}` in human-written code. Generated code with enforced formatting may omit, but treat as an exception with review.

### EXC-FALLTHROUGH
If intentional, mark it: `__attribute__((fallthrough));` (GCC/Clang) or `[[fallthrough]];` where supported.
