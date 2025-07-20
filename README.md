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