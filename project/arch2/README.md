# Part 1: Short Answer, Optional

1. The minimum seek time for an HDD is 9msec, and the maximum seek time is
   90msec. The block size of this HDD is 4KB. How long on average does it take
   to read 100MB of data?

2. Describe a TCP/IP packet in detail. Describe the header, how many bytes it
   is, and which components it contains. What data can come after the header?

3. How does the network protocol guarantee that a TCP/IP packet is complete
   after transmission?

4. What is the difference between TCP and IP?

5. Why is 3d performance so much higher with a graphics card than without?
   Modern CPUs are extremely fast, what is limiting their performance?

# Part 2: Add Interrupts to the LS-8, Optional

Add interrupts to the LS-8 emulator from the Computer Architecture 1.

**For this assignment, keep using the repo you cloned for
Computer-Architecture-One. Don't put it in this repo.**

**You must have implemented a CPU stack before doing this.**

**You must have implmented the `ST` instruction before doing this.**

See the [LS-8
spec](https://github.com/LambdaSchool/Computer-Architecture-One/blob/master/LS8-SPEC.md)
for details on implementation.

The LS-8 should fire a timer interrupt one time per second. This could be
implemented with `setInterval()`, where the handler sets bit 0 of the IS
register (R6). (This is in addition to the normal `tick()` timer you already
have.)

Later in the main instruction loop, you'll check to see if bit 0 of the
IS register is set, and if it is, you'll push the registers on the
stack, look up the interrupt handler address in the interrupt vector
table at address `0xF8`, and set the PC to it. Execution continues in
the interrupt handler.

Then when an `IRET` instruction is found, the registers and PC are
popped off the stack and execution continues normally.

## Example

This code prints out the letter `A` from the timer interrupt handler
that fires once per second.

```
# interrupts.ls8

10011001 # LDI R0,0xF8
00000000
11111000
10011001 # LDI R1,INTHANDLER
00000001
00010001
10011010 # ST R0,R1  Set the interrupt vector at 0xF8 to INTHANDLER
00000000
00000001
10011001 # LDI R5,1  Set the IM register so we receive timer interrupts
00000101
00000001
10011001 # LDI R0,LOOP  Infinite loop forever
00000000
00001111
# LOOP (15):
01010000 # JMP R0
00000000

# Timer interrupt Handler
# When the timer interrupt occurs, output an 'A'
# INTHANDLER (17):
10011001 # LDI R0,65   A is 65 in ASCII
00000000
01000001
01000010 # PRA R0      Print it
00000000
00001011 # IRET        Return from interrupt
```


The assembly program is interested in getting timer interrupts, so it sets the
IM register to `00000001` with `LDI R5,1`.

The interrupt `setInterval()` timer fires and sets bit #0 in IS.

At the beginning of `tick()`, the CPU checks to see if interrupts are enabled.
If not, it continues processing instructions as normal. Otherwise:

Bitwise-AND the IM register with the IS register. This masks out all the
interrupts we're not interested in, leaving the ones we are interested in:

```javascript
let interrupts = this.reg[IM] & this.reg[IS];
```

Step through each bit of `interrupts` and see which interrupts are set.

```javascript
for (let i = 0; i < 8; i++) {
  // Right shift interrupts down by i, then mask with 1 to see if that bit was set
  let interruptHappened = ((interrupts >> i) & 1) == 1;

  ...
}
```

(If the no interrupt bits are set, then stop processing interrupts and continue
executing the next instruction as per usual.)

If `interruptHappened`, check the LS-8 spec for details on what to do.
