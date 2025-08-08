# Embedded C Golden Rules – Quick Reference

| Category | Rule | 🚫 Bad Example | ✅ Good Example |
|----------|------|----------------|----------------|
| **Data Types** | Use `<stdint.h>` fixed-width types | `int speed;` | `uint16_t speed_kmh;` |
| | Always specify signed/unsigned | `short temp;` | `int16_t temp_c;` |
| | Avoid `float` unless needed | `float volts = adc/1023.0;` | `uint16_t mv = (adc*3300)/1023;` |
| | Mind endianess | `memcpy(buf, &val, 4);` | Use `htonl(val)` or manual byte packing |
| **Initialization** | Always init variables | `uint8_t x; if(x>0)` | `uint8_t x=0; if(x>0)` |
| | Init HW regs after enabling clocks | `USART1->CR1=...;` | `RCC->APB2ENR \|= RCC_USART1EN; USART1->CR1=...;` |
| **Pointers & Memory** | Check before deref | `*ptr = 5;` | `if(ptr) *ptr = 5;` |
| | Avoid pointer arithmetic | `p += 5;` | Use array indexing: `arr[i+5]` |
| | Don’t return ptr to local var | `return buf;` | Use `static` or caller-provided buffer |
| | Avoid `malloc` in bare-metal | `ptr = malloc(100);` | Use static buffer or pool allocator |
| | Watch stack usage | Deep recursion | Iterative approach / static buffers |
| | Use `const` for RO data | `char msg[] = "Hi";` | `const char msg[] = "Hi";` |
| | Align DMA buffers | Misaligned array | `__attribute__((aligned(4))) uint8_t buf[64];` |
| | Forbid hidden allocation in libs | lib `malloc()` internally | Caller supplies memory / pool handle |
| **Constants & Macros** | Replace magic numbers | `delay_ms(123);` | `#define SENSOR_TIMEOUT_MS 123` |
| | Use `enum` for related constants | `if(mode == 1)` | `enum Mode { MODE_IDLE = 1 }; if(mode == MODE_IDLE)` |
| **Control Flow & Style** | Avoid `goto` | `goto fail;` | Use function cleanup path or `break` |
| | Descriptive names | `tmp` | `battery_voltage_mv` |
| | Comment “why” not “what” | `// increment` | `// compensate for sensor offset` |
| | Always use braces `{}` for if/else/loops | `if(flag) do_something();` | `if(flag) { do_something(); }` |
| **Interrupts & Concurrency** | Keep ISR short | `ISR { process(); log(); }` | `ISR { set_flag(); }` |
| | No blocking in ISR | `ISR { delay_ms(10); }` | Use flag + process in main loop |
| | Mark HW regs/shared vars `volatile` | `uint8_t flag;` | `volatile uint8_t flag;` |
| | Protect shared data | `shared++;` | `DISABLE_IRQ(); shared++; ENABLE_IRQ();` |
| | Clear IRQ priority policy | Random priorities | Define and document priorities |
| | Always use timeouts | `while(!flag);` | `wait_for_flag(1000);` |
| **Bitwise & Low-level Access** | Use macros for bits | `PORT \|= (1<<3);` | `SET_BIT(PORT, BIT3);` |
| | Parenthesize masks/shifts | `x = 1<<n+1;` | `x = (1 << (n+1));` |
| | Check status flags | `UART->DR = data;` | `while(!(UART->SR & TXE)); UART->DR = data;` |
| | Use vendor masks | `REG \|= 0x08;` | `REG \|= REG_TX_ENABLE;` |
| | Debounce inputs | `if(btn)` | Delay/filter before accept |
| **Functions & Modularity** | Keep small, one purpose | 50-line function | Split into multiple small functions |
| | Use `static` for file-scope | `int helper()` | `static int helper()` |
| | Pass dependencies, avoid globals | Uses global `sensor_val` | Pass `sensor_val` as parameter |
| | Prefer pure functions for logic | `int calc() { return TIMER->CNT/f; }` | `int calc(uint16_t ticks) { return ticks/f; }` |
| | Separate I/O from logic | Math inside driver reads/writes regs | Driver only does I/O; logic in separate module |
| | Use dependency injection | `uart_write()` called directly | Pass `io->write()` via interface struct |
| | Make library functions reentrant | Static hidden buffers | Caller provides buffer/context |
| | Make init/deinit idempotent | Double `init()` breaks | `init()` checks state; safe to call twice |
| | Mark inputs/ptrs `const` | `int foo(uint8_t *p)` | `int foo(const uint8_t *p)` |
| | Document/enforce buffer ownership | Callee `free()`s caller’s ptr | Caller owns lifetime; callee never frees |
| **Defensive Coding & Safety** | Validate inputs | `set_speed(speed);` | `if(speed <= MAX_SPEED) set_speed(speed);` |
| | Use `assert()` in debug | `ptr->val = 1;` | `assert(ptr != NULL); ptr->val = 1;` |
| | Pet watchdog only when healthy | Pet everywhere | Pet after full loop OK |
| | Log reset cause | No log | `cause = RCC->CSR;` |
| | Use explicit status enums | `return -3;` | `return STATUS_TIMEOUT;` |
| | Validate early, return errors | Use value before range-check | Check bounds; bail with status code |
| **Protocols & Data Handling** | CRC/checksum data | Trust packet | Verify CRC before use |
| | Never trust incoming sizes | Use given len | Clamp to buffer size |
| | Encode units in names/types | `int temp;` | `int16_t temp_cdeg; // 0.01°C` |
| | Serialize explicitly; no struct cast | `send(&pkt, sizeof pkt);` | Field-by-field pack with endianness |
| **Testing & Debugging** | Unit test logic | No tests | Host-run unit tests |
| | Fault injection hooks | None | Add test-only failure triggers |
| | Keep logs out of core logic | Core math calls `printf()` | Core returns status; outer layer logs |
| | Provide mocks/stubs for HW interfaces | Hardwired HAL calls | `struct UartVTable { write, read }` for mocks |
| **RTOS** | One responsibility per task | One task does all | Split into dedicated tasks |
| | Use events/semaphores | Busy wait | `osSemaphoreWait()` |
| **Miscellaneous Gotchas** | Rate-limit flash writes | Write on every loop | Cache \& batch writes |
| | Downcast only with checks | `uint8_t x = (uint32_t)y;` | `if(y <= UINT8_MAX) x = (uint8_t)y;` |
| | Avoid heavy `printf` | `printf("%f", x);` | Minimal logging or int print |
| | Watch struct padding | `send(sock, &s, sizeof(s));` | Pack manually / serialize |
| | Pass time in, don’t read globally | `if (millis() - t0 > 10)` | `if (now_ms - t0 > 10)` (now passed in) |
| | Isolate RNG, seed explicitly | `val = rand();` | `val = rng->next_u32(rng_ctx);` |
| | Use table/pure transition funcs | Deep nested `if/else` | `state = fsm_step(state, evt);` (pure) |
| | Limit header exposure | Put private APIs in `.h` | Use `.c` static + minimal public `.h` |
| | Keep functions total (all inputs defined) | Undefined for some args | Return error for invalid cases |
| | Avoid hidden global state in logic | Reads global mode flag | Pass mode/config into function |
| | Centralize side effects at top-level | Logic toggles GPIOs | Logic returns actions; caller toggles GPIO |
