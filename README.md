# Peripherals

Peripherals live on the front side bus with other fundamental components. Plugging a peripheral in via PCIx, SCSI, or SATA physically locates that device on the system bus. Other devices like USB devices exist on a separate bus from the system bus, but are synchronized with the system bus using a USB controller and a device driver that can translate USB bus messages into system bus messages.

## Network interfaces

Independent processor caches network traffic:

[OSI model](https://en.wikipedia.org/wiki/OSI_model)

Memory address, stack pointer, data transmission rules
Processor communicates asynchronously over network connection in order to satisfy rules of protocol in use - either TCP or UDP.

IP

[Internet Protocol](https://en.wikipedia.org/wiki/Internet_Protocol)

TCP

[Transmission Control Protocol](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)

UDP

[User Datagram Protocol](https://en.wikipedia.org/wiki/User_Datagram_Protocol)

#### More recommended reading

[Just2Good Description of Networking](http://www.just2good.co.uk/networking.php)

## DMA

[Direct Memory Access](https://en.wikipedia.org/wiki/Direct_memory_access)

### Hard disks

Platter-based hard disks used to be a very interesting subject of research and discussion. Imagine ultra-smooth platters spinning 100 times per second, with binary data encoded on them in sections < 100 nanometers. The hard drive is the most space age piece of equipment in your home.

You can learn about them here:
[Engineer Guy Hard Drive](https://en.wikipedia.org/wiki/File:Harddrive-engineerguy.ogv)

Platter based hard disks have an insane storage cost, less than $0.03 per gigabyte:

[BackBlaze](https://www.backblaze.com/blog/hard-drive-cost-per-gigabyte/)

SSDs are less interesting - they are just flash memory with a controller that engages your system's PCI bus.

## Cyclic Redundancy Checking

### Graphics Accelerators

Dedicated graphics pipelines - hardware designed just for manipulating pixels in an array and updating them in a shared memory used by the display. Separate pipeline from the CPU, again, graphics memory can be written to display without needing to be circled around with the CPU.

#### Shaders

C++ software (gsl, actually) that executes simple mathematics on every vertex simultaneously.

#### Pipeline components



#### Software components

OpenGL, DirectX, WebGL. CUDA, GLSL

#### Onboard memory


# Assignment 1

1. The minimum seek time for an HDD is 9msec, and the maximum seek time is 90msec. The block size of this HDD is 4KB. How long on average does it take to read 100MB of data?

2. Describe a TCP/IP packet in detail. Describe the header, how many bytes it is, and which components it contains. What data can come after the header?

3. How does the network protocol guarantee that a TCP/IP packet is complete after transmission?

4. What is the difference between TCP and IP?

5. Why is 3d performance so much higher with a graphics card than without? Modern CPUs are extremely fast, what is limiting their performance?

# Assignment 2

Add interrupts to the LS-8 emulator from the Computer Architecture 1.

**You must have implemented a CPU stack before doing this.**

See the [LS-8
spec](https://github.com/LambdaSchool/Computer-Architecture-One/blob/master/LS8-SPEC.md)
for details on implementation.

The LS-8 should fire a timer interrupt one time per second. This could
be implemented with `setInterval()`, where the handler sets bit 0 of the
IS register (R6).

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

00000100 # LDI R0,0xF8  R0 holds the interrupt vector for I0 (timer)
00000000
11111000
00000100 # LDI R1,17    R1 holds the address of the handler
00000001
00010001
00001001 # ST R0,R1     Store handler addr in int vector
00000000
00000001
00000100 # LDI R5,1     Enable timer interrupts
00000101
00000001
00000100 # LDI R0,15    Load R0 with the spin loop address
00000000
00001111
# Address 15
00010001 # JMP R0       Infinite spin loop right here
00000000

# Interrupt handler
# Address 17
00000100 # LDI R0,65    Load R0 with 'A'
00000000
01000001
00000111 # PRA R0       Print it
00000000
00011010 # IRET         Return from interrupt
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
