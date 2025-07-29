# The Linux Catgirl Edition kernel

A custom Linux build system forked from the work by [CachyOS](https://github.com/cachyos/linux-cachyos), aiming to provide the foundation for compiling highly scalable Linux systems.

To achieve this goal, we provide the option to disable certain Linux kernel features and pull in certain patches.

<!-- This kernel is able to allocate several gigabytes over its physical memory and swap it into zram, whereas mainline would livelock immediately. -->
<!-- TODO: show more cases or disprove the above -->

<details>
<summary>List of optimizations</summary>

Note that the precompiled binaries and PKGBUILD do not enable all of these by default. This list is non-exhaustive.

Review the `PKGBUILD` file to see the tradeoff(s) (if any) in more detail.

## perf:

- Use Guess unwinder by default (Has zero overhead as opposed to ORC/frame pointer)
- Fine grained kernel tickrates (default: 1000Hz)
- Remove paravirtualized layer
- Google's [TCP BBRv3 congestion protocol](https://github.com/google/bbr/tree/v3)
- Disable memory zero-init (dangerous, but is ~1% faster :rocket:)
- Disable stack zero-init (also dangerous, but is faster :rocket:)
- No structure corruption checking (disabled in upstream but arch kernel enables)
- `-march=native` (or similar) optimization to guide the compiler to optimize better w.r.t. cache sizes
- CachyOS kernel patches
- Choose between BORE, EEVDF, or BMQ schedulers (and also real-time)
- `-O3` optimization (!)
- Fine grained preemption control (rt, full, lazy, voluntary, none)
- No module unloading
- `modify_ldt(2)` removed for lower context switch latency

## size:

- Remove BUG()
- Remove coredump support
- Remove tracing (profiling) infrastructure
- Remove module decompression in kernel
- Remove 16/32 bit app support

## size & perf:

- Clang [Thin]LTO;
- No printk() support. This reduces kernel size (no more strings) and reduces overhead where `printk()` calls are plenty (eg during boot, resume);
- Remove scheduler debugging;
- Trim unused headers to help LTO and optimization if headers are disabled

... aaaand much more that you can still set yourself in `make nconfig` (in which case, submit a PR!)

[^1]: source: i made it up.

</details>

## What does linux-catgirl-edition not do?

linux-catgirl-edition avoids doing tweaks that can be easier set during runtime instead of compile time. that means sysctl values will still have to be set by you.

this kernel only changes things that cannot be easily changed or straight up impossible to change during runtime (e.g. module support)

---

## Where can I get linux-catgirl-edition?

You can download the precompiled package either from the [releases page](https://github.com/pparaxan/linux-catgirl-edition/releases) or build from the [actions page](https://github.com/pparaxan/linux-catgirl-edition/actions).

## How do I compile linux-catgirl-edition locally?

* Clone this repository, and **c**hange **d**irectory into it.
* This is important, open the `PKGBUILD` file and enable or disable any optimizations you may or may not want[^2].
* Run the command `makepkg -scf --cleanbuild --skipchecksums` to install the package.

[^2]: don't worry, its very documented. and to clarify the important part: many optimizations displayed in the optimizations section are disabled by default because I know some of you will blindly `makepkg -s` anyway.

## I don't use Arch Linux (btw)

You can use an Arch Linux docker container to build the kernel. It's _probably_ trivial to turn a `.tar.zst` into a `.deb` or whatever packaging type (except for flatpak) you prefer

