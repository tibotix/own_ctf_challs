Tags: GOT-overwrite, ROP, static mmap-offset, ld.so leak, one_gadget

TL;DR:

1. leak `_dl_runtime_resolve_xsave` from GOT (index -10 from `primes` array)
2. calculate `ld` base address (`_dl_runtime_resolve_xsave - 0x18EE0`)
3. calculate `libc` base address (as mmap places ld and libc at constant offset, in this libc vesion it is `0x22C000`)
4. calculate rop gadgets in libc to set rsi=0x0 and rdx=0x0
5. calculate rop gadget in libc `add rsp, 0x58`
6. calculate one_gadget in libc
7. overwrtie address to `_dl_runtime_resolve_xsave` in GOT to rop gadget `add rsp, 0x58`
8. trigger lazy binding `__stack_check_fail` function resolving by overwriting "[y/n]" answer with long rop chain
9. when tying to resolve `__stack_check_fail`, normally to `_dl_runtime_resolve_xsave` is jumped, but as we have overwritten the address, it now jumps to our ropchain preparing gadget (`add rsp,0x58; ret`) which will set `rsp` to the beginning of the answer buffer and executes our ropchain using `ret`.
10. Our ropchain will then set the contraints of the one_gadget (`rsi=0x0`, `rdx=0x0`) and jumps to the one_gadget.

One tricky thing is that we cannot overwrite any used function stub addresses in the GOT (`printf@plt`, `strcmp@plt`, ...), as the distance from the `primes` array to these GOT entries are lower than `10*8=80` and this checks prevents any index between `[-9;9]`:

```c
if (abs(prime_idx) < 10)
{
	printf("Nah, thats too easy for me.\n");
	return 0;
}
```

So the only thing left is the general `_dl_runtime_resolve` function that is called whenever the actual address of a not already called function needs to be resolved.

Another small difficulty is the distance between `_dl_runtime_resolve` and `ld.so base`, as well as the distance between `ld.so base` and `libc.so` base. These distances are `ld.so` and `libc.so` specific so to find them i used `/proc/<pid>/maps` of a running process of `primes` in the docker container.

The libc gadgets of course also relates to the specific libc version used. I just identified the used libc vesion in the docker container using the sha256 hash of the `libc.so` file. Then the correct version can be identified using the libc-database with the command command `./identifiy sha256=64c5884701cf52968917726289eb66bfd2e776f9d18d7e37c4449b91efa6d0cf`.
