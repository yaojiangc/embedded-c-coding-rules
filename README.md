# Embedded C Golden Rules – Quick Reference

| # | Category | Rule | 🚫 Bad Example | ✅ Good Example |
|---|----------|------|----------------|----------------|
| **Data Types** |||||
| 1 | Data Types | Use `<stdint.h>` fixed-width types | `int speed;` | `uint16_t speed_kmh;` |
| 2 | Data Types | Always specify signed/unsigned | `short temp;` | `int16_t temp_c;` |
| 3 | Data Types | Avoid `float` unless needed | `float volts = adc/1023.0;` | `uint16_t mv = (adc*3300)/1023;` |
| 4 | Data Types | Mind endianess | `memcpy(buf, &val, 4);` | Use `htonl(val)` or manual byte packing |
| **Initialization** |||||
| 5 | Init | Always init variables | `uint8_t x; if(x>0)` | `uint8_t x=0; if(x>0)` |
| 6 | Init | Init HW regs after enabling clocks | `USART1->CR1=...;` | `RCC->APB2ENR \|= RCC_USART1EN; USART1->CR1=...;` |
| **Pointers & Memory** |||||
| 7 | Pointers | Check before deref | `*ptr = 5;` | `if(ptr) *ptr = 5;` |
| 8 | Pointers | Avoid pointer arithmetic | `p += 5;` | Use array indexing: `arr[i+5]` |
| 9 | Pointers | Don’t return ptr to local var | `return buf;` | Use `static` or caller-provided buffer |
| 10 | Memory | Avoid `malloc` in bare-metal | `ptr = malloc(100);` | Use static buffer or pool allocator |
| 32 | Memory | Watch stack usage | Deep recursion | Iterative approach / static buffers |
| 33 | Memory | Use `const` for RO data | `char msg[] = "Hi";` | `const char msg[] = "Hi";` |
| 34 | Memory | Align DMA buffers | Misaligned array | `__attribute__((aligned(4))) uint8_t buf[64];` |
| 62 | Memory | Forbid hidden allocation in libs | lib `malloc()` internally | Caller supplies memory / pool handle |
| **Constants & Macros** |||||
| 11 | Constants | Replace magic numbers | `delay_ms(123);` | `#define SENSOR_TIMEOUT_MS 123` |
| 12 | Constants | Use `enum` for related constants | `if(mode == 1)` | `enum Mode { MODE_IDLE = 1 }; if(mode == MODE_IDLE)` |
| **Control Flow & Style** |||||
| 13 | Control | Avoid `goto` | `goto fail;` | Use function cleanup path or `break` |
| 23 | Style | Descriptive names | `tmp` | `battery_voltage_mv` |
| 24 | Style | Comment “why” not “what” | `// increment` | `// compensate for sensor offset` |
| 71 | Style | Always use braces `{}` for if/else/loops | `if(flag) do_something();` | `if(flag) { do_something(); }` |
| **Interrupts & Concurrency** |||||
| 14 | ISR | Keep ISR short | `ISR { process(); log(); }` | `ISR { set_flag(); }` |
| 15 | ISR | No blocking in ISR | `ISR { delay_ms(10); }` | Use flag + process in main loop |
| 18 | Volatile | Mark HW regs/shared vars | `uint8_t flag;` | `volatile uint8_t flag;` |
| 19 | Concurrency | Protect shared data | `shared++;` | `DISABLE_IRQ(); shared++; ENABLE_IRQ();` |
| 35 | Concurrency | Clear IRQ priority policy | Random priorities | Define and document priorities |
| 36 | Concurrency | Always use timeouts | `while(!flag);` | `wait_for_flag(1000);` |
| **Bitwise & Low-level Access** |||||
| 16 | Bitwise | Use macros for bits | `PORT \|= (1<<3);` | `SET_BIT(PORT, BIT3);` |
| 17 | Bitwise | Parenthesize masks/shifts | `x = 1<<n+1;` | `x = (1 << (n+1));` |
| 27 | HW Access | Check status flags | `UART->DR = data;` | `while(!(UART->SR & TXE)); UART->DR = data;` |
| 28 | HW Access | Use vendor masks | `REG \|= 0x08;` | `REG \|= REG_TX_ENABLE;` |
| 29 | HW Access | Debounce inputs | `if(btn)` | Delay/filter before accept |
| **Functions & Modularity** |||||
| 20 | Functions | Keep small, one purpose | 50-line function | Split into multiple small functions |
| 21 | Functions | Use `static` for file-scope | `int helper()` | `static int helper()` |
| 22 | Functions | Pass dependencies, avoid globals | Uses global `sensor_val` | Pass `sensor_val` as parameter |
| 50 | Functions | Prefer pure functions for logic | `int calc() { return TIMER->CNT/f; }` | `int calc(uint16_t ticks) { return ticks/f; }` |
| 51 | Architecture | Separate I/O from logic | Math inside driver reads/writes regs | Driver only does I/O; logic in separate module |
| 52 | Testability | Use dependency injection | `uart_write()` called directly | Pass `io->write()` via interface struct |
| 55 | Reentrancy | Make library functions reentrant | Static hidden buffers | Caller provides buffer/context |
| 56 | Idempotence | Make init/deinit idempotent | Double `init()` breaks | `init()` checks state; safe to call twice |
| 57 | Const-correctness | Mark inputs/ptrs `const` | `int foo(uint8_t *p)` | `int foo(const uint8_t *p)` |
| 58 | Ownership | Document/enforce buffer ownership | Callee `free()`s caller’s ptr | Caller owns lifetime; callee never frees |
| **Defensive Coding & Safety** |||||
| 25 | Defensive | Validate inputs | `set_speed(speed);` | `if(speed <= MAX_SPEED) set_speed(speed);` |
| 26 | Defensive | Use `assert()` in debug | `ptr->val = 1;` | `assert(ptr != NULL); ptr->val = 1;` |
| 37 | Safety | Pet watchdog only when healthy | Pet everywhere | Pet after full loop OK |
| 38 | Safety | Log reset cause | No log | `cause = RCC->CSR;` |
| 59 | Errors | Use explicit status enums | `return -3;` | `return STATUS_TIMEOUT;` |
| 60 | Fail-fast | Validate early, return errors | Use value before range-check | Check bounds; bail with status code |
| **Protocols & Data Handling** |||||
| 39 | Protocols | CRC/checksum data | Trust packet | Verify CRC before use |
| 40 | Protocols | Never trust incoming sizes | Use given len | Clamp to buffer size |
| 64 | Units | Encode units in names/types | `int temp;` | `int16_t temp_cdeg; // 0.01°C` |
| 65 | Serialization | Serialize explicitly; no struct cast | `send(&pkt, sizeof pkt);` | Field-by-field pack with endianness |
| **Testing & Debugging** |||||
| 41 | Testing | Unit test logic | No tests | Host-run unit tests |
| 42 | Testing | Fault injection hooks | None | Add test-only failure triggers |
| 61 | Logging | Keep logs out of core logic | Core math calls `printf()` | Core returns status; outer layer logs |
| 70 | Testing | Provide mocks/stubs for HW interfaces | Hardwired HAL calls | `struct UartVTable { write, read }` for mocks |
| **RTOS** |||||
| 43 | RTOS | One responsibility per task | One task does all | Split into dedicated tasks |
| 44 | RTOS | Use events/semaphores | Busy wait | `osSemaphoreWait()` |
| **Miscellaneous Gotchas** |||||
| 46 | Persistence | Rate-limit flash writes | Write on every loop | Cache \& batch writes |
| 47 | Casting | Downcast only with checks | `uint8_t x = (uint32_t)y;` | `if(y <= UINT8_MAX) x = (uint8_t)y;` |
| 48 | Gotchas | Avoid heavy `printf` | `printf("%f", x);` | Minimal logging or int print |
| 49 | Gotchas | Watch struct padding | `send(sock, &s, sizeof(s));` | Pack manually / serialize |
| 53 | Time | Pass time in, don’t read globally | `if (millis() - t0 > 10)` | `if (now_ms - t0 > 10)` (now passed in) |
| 54 | Randomness | Isolate RNG, seed explicitly | `val = rand();` | `val = rng->next_u32(rng_ctx);` |
| 63 | FSM | Use table/pure transition funcs | Deep nested `if/else` | `state = fsm_step(state, evt);` (pure) |
| 66 | Boundaries | Limit header exposure | Put private APIs in `.h` | Use `.c` static + minimal public `.h` |
| 67 | APIs | Keep functions total (all inputs defined) | Undefined for some args | Return error for invalid cases |
| 68 | Determinism | Avoid hidden global state in logic | Reads global mode flag | Pass mode/config into function |
| 69 | Side effects | Centralize side effects at top-level | Logic toggles GPIOs | Logic returns actions; caller toggles GPIO |
