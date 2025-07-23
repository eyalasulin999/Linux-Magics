# Linux-Magics

Linux has many weird, unexpected and funny tricks & behaviors that make you say WTF.

Let's investigate them and learn about how Linux works under the hood.

## 0x01 - Execute ELF without executable permissions

Create new copy of `id` binary without executable permissions

![](images/01-1.png)

Great - not working

Let's take the first string of the binary and use it to execute it

![](images/01-2.png)

WTF

What is that string?

![](images/01-3.png)

We invoked the dynamic loader with the ELF file as an argument

The dynamic loader is responsible for starting dynamically linked programs - loads the ELF binary into memory, resolves and links shared libraries and transfers control to the program’s main() function

Dynamic loader path located in ELF header - PT_INTERP (Program Header)

But how is the dynamic loader invoked normally?

binfmt_elf - the kernel checks binary magic bytes (`\x7fELF`), parses the program headers to extract PT_INTERP and executes the interpreter (aka dynamic loader) instead of the binary directly, passing the ELF file as its first argument

**Resources:**
- `man 8 ld.so`
- `man 5 elf`
- https://elixir.bootlin.com/linux/v4.8/source/fs/binfmt_elf.c

## 0x02 - [

Have you ever used if statements in bash?

![](images/02-1.png)

How clean and elegant it is.

![](images/02-2.png)

WTF, `[` is a binary?

That binary just gets a condition and returns true or false

What about `]`? it is just an argumant, the `[` binary checks that the last argument is `]`

`[` and `test` are effectively the same command

`/usr/bin/[` could be also a symlink to `/usr/bin/test` on some Linux distros

Now, most shells has `[` as a shell built-in command

![](images/02-3.png)

**Resources:**
- `man 1 test`

## 0X03 - Linux boot by Microsoft Corp.

ELF is the format for everything in Linux - also the kernel

Right?

![](images/03-1.png)

WTF, why does my kernel have MZ and PE headers?

![](images/03-2.png)

UEFI is the modern replacement for the old BIOS firmware

It uses PE file format to load bootloaders and operating systems

To support booting on UEFI systems, the Linux kernel is wrapped inside a PE container

This wrapping lets UEFI firmware load the kernel as a normal EFI executable

The actual Linux kernel is still an ELF binary inside this PE wrapper

On BIOS systems, bootloaders ignore the PE wrapper and directly load the compressed ELF kernel - this design allows a single kernel image to boot on both BIOS and UEFI platforms without modification

You could make your kernel supports UEFI booting in build time - `CONFIG_EFI.*`