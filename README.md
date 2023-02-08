# Rust OS journey

Original: https://os.phil-opp.com/freestanding-rust-binary/

My journey on creating an OS with Rust to achieve two goals below
- Understand computer architecture better
- Study Rust

## Notes

### Boot process

BIOS boot

  - BIOS loads from memory located on the motherboard
  - BIOS runs self-test and initialize routines of the hardware
  - Check bootable disk, in case there is, control is transferred to bootloader
    + Bootloader: 512-byte portion of executable code stored at disk's beginning
    + Bootloader is usually split to smaller parts
  - Bootloader
    + determines location of kernel image on disk and load into memory
    + decides if it is necessary to switch between modes: 16-bit (real mode), 32-bit (protected mode) and 64-bit (long mode)
    + queries certain information (e.g: memory map) from BIOS and passes to kernel

Multiboot Standard

- Why? To avoid every system implements its own bootloader
- Defines interface between the bootloader and operating system -> any Multiboot-compliant bootloader can load any Multiboot-compliant operating system
- Most famous: [GNU GRUB](https://en.wikipedia.org/wiki/GNU_GRUB)
- To make a kernel compliant, just have to insert Multiboot header at the beginning of the kernel file -- easy to implement
- Problems
  + Only support 32-bit protected mode (need to do extra configurations to switch to 64-bit long mode)
  + The implementation on the kernel side is not that easy
  + GRUB and multiboot are sparsely documented
  + GRUB needs to be installed on the host system to create a bootable disk image from the kernel file

### VGA Text Mode
- Wiki: https://en.wikipedia.org/wiki/VGA_text_mode
- A simple way to print text to the screen
- Two-dimensional array with typically 25 rows and 80 columns
- Bits in range of position have different meanings

> The second byte defines how the character is displayed.
> The first four bits define the foreground color, the next three bits the background color
> The last bit whether the character should blink.

### Spinlock
- A lock that causes a thread trying to acquire it to simple wait in a loop (spin) while repeatedly checking whether the lock is available.
- Held until explicitly released
- Help avoid overhead from operating system process rescheduling or context switching
- Efficient if threads are blocked in a short period
- Usually used by operating system kernel
- If a thread is blocked in a long time, OS scheduler might kills the thread before it releases the spinlock. This causes threads that are trying to acquire the lock stay waiting.
