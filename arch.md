Key stored in linkerscript at known address...
Key is basically a stack canary
## Part 1
Comparison value in c code, to check if that key is present

## Part 2
Comparison value in `Reset_Handler` to check if key present
I could probably add a new function which then jumps to `Reset_Handler` or infinite loop 
This would require editing `startup_stm32l433xx.s`, having your initial function (even before (`Reset_Handler`) check that value in memory would be valuable...
1. Add a new function `verify_boot` before `Reset_Handler`
  1. Check against `_sdata`
  1. If equivalent, jump to `Reset_Handler`
  1. If not equivalent, jump to `Infinite_Loop`
1. Update Linker Script
  1. Make `verify_boot` a symbol .ld can read (from asm)
  1. Make `verify_boot` the ENTRY point of function
  1. Make sure that `verify_boot` is at address 0x0, and `Reset_Handler` is after (different in startup.o and HWSEC.elf)
  1. Test
1. Make production ready
  1. Enhancements

## Part 2 options
You could also have the `Reset_Handler` store a value in memory, and then check it from the C script
Option A: Store Linker, Read C
Option B: Store ASM, Read C
Option C: Store Linker, Read ASM
Option D: Store C (String Literal), Read ASM

## What I learned
- Reset sequence clears memory regions and sets sp and interrupt table to known values before calling the c initialization then calling main().
- Linkerscript defines ENTRY, the function which the program will use on startup.
- `Reset_Handler` is defined in `startup_stm32l433xx.s` an assembly startup file, unique to each STM32 type
- GDB has a hard time inserting breakpoints before main is called, with mixed success
- Linkerscript allows QUAD and LONG to put specific values in memory
- `<symbol>` defined in .ld can be read as `extern <type> symbol` in C code
- make didn't detect changes in linkerscript as cause to re-compile...

## Limitations
- I only ever used 8 byte strings, which is fine, but arbitrary length strings are harder
- Looked into using `system_stm32l4xx.c` and having SystemInit check the value, but that wasn't working, and now gdb can't read before main...
