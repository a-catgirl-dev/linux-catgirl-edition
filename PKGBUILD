# attribution:
# **cachyos** for the base pkgbuild that i have heavily modified
# **linux-tkg** for their pkgbuilds

# The comments above each variable include useful documentation as I understand it. Read throughly, and do your own
# research if needed.
#
# If you are unsure, feel free to ask the maintainers, via email:
# - paws [at] a-catgirl [dot] dev
# - xxdr [at] duck [dot] com
# or alternatively via github issues:
# - https://github.com/a-catgirl-dev/linux-catgirl-edition/issues/new/choose

# when you are done with configuration, execute `makepkg -scf --cleanbuild --skipchecksums` to begin the build.

# if you change this, you will want to execute `rm *.patch` to ensure the kernel is patched properly
#
# if you are updating the kernel, consider running `git stash && git fetch && git pull && git stash pop` instead
# of updating these values
_major=6.18
_minor=.4

# select custom patchset(s)
#
# these patchsets can apply together, but regressions may occur.
# if you are unsure, select only `_import_cachyos_patchset`

# include cachyos patchset
: "${_import_cachyos_patchset:=yes}"

# include partial clear linux[^1] patches
#
# [^1]: community maintained clearlinux patchset: https://git.staropensource.de/StarOpenSource/Linux-Tachyon
: "${_import_clear_patchset:=no}"

# include partial xanmod patches
: "${_import_xanmod_patchset:=no}"

# patchset specific tweaks

# prevent sleep demotion (PSD) (depends on `_import_clear_patchset`)
#
# PSD is a custom clearlinux feature to prevent the CPU[^1] from entering a sleep state immediately
# after disk IO begins, instead waiting for a bit _before_ entering a sleep state
#
# This can reduce latency on high end NVMe SSDs, possibly on a RAID configuration, but some consumer NVMe drives
# may also benefit, but to a lesser extent.
#
# Drives that are not NVMe are not affected by the behaviour of this subsystem.
#
# If unsure, select yes
#
# [^1]: Intel has found that ~85-90% of IO are serviced by the same CPU/Processor that issued them
: "${_clearlinux_prevent_sleep_demotion:=yes}"

# select a CPU scheduler
#
# Available options:
# - `bore`:     is a modified EEVDF scheduler that is less fair with its scheduling in order to provide
#               better interactivity under load. recommended for desktop systems
# - `bmq`:      is a scheduler inspired by Google's Zircon. It is designed to be small. may benefit slow machines
#               by being more likely to fully fit in L1i cache. possibly less mature than EEVDF (or CFS) and as such
#               may not schedule as well under load (or in general?)
# - `hardened`: is just `bore` with some patches from linux-hardened. not supported by catgirl-edition
# - `eevdf`:    is the stock linux scheduler. it promises all tasks will get the CPU in an acceptable amount of time
#               provided that the CPU is not at 100% load. it is fully fair and is therefore the scheduler of choice
#               for server systems.
# - `rt`:       is a set of patches that run the `eevdf` scheduler to provide realtime latency (latency such as
#               the time between a car crash and airbags deploying). good for latency sensitive systems and systems
#               where latency need to be predictable
# - `rt-bore`:  is `rt` but runs the `bore` scheduler. more suited for desktop systems that for some reason need realtime
#               or predictable latency (like professional audio workstations.)
#
# If you require maximum throughput, select `eevdf`.
# Desktop users should select `bore`.
# Real-time systems should run `rt`, and professional audio workstations may want `rt-bore`
#
# If unsure, select `bore`
: "${_cpusched:=bore}"

# configure the kernel via nconfig?
: "${_makenconfig:=yes}"

# Use modprobed-db
#
# Uses modprobed-db to disable kernel modules that you do not need.
# This will significantly reduce compile time & reduce the size of your kernel.
# Usage guide: https://wiki.archlinux.org/title/Modprobed-db
#
# Incompatible with `_diet_kernel`
#
# If unsure, select `yes`
: "${_localmodcfg:=yes}"

# modprobed.db path
#
# If `_localmodcfg` is enabled, this value must be set or compile will fail early on.
#
# If unsure, leave as default.
: "${_localmodcfg_path:="$HOME/.config/modprobed.db"}"

# Assume a baseline of modules
#
# Uses linux-tkg's "diet kernel" option, which includes a baseline of modules that are assumed to be able to work for
# most systems, making the kernel more portable across different hardware configurations when compared to `_diet_kernel`
# Reduces build times when compared to a full kernel, but still will take a longer time compared to `_localmodcfg`
#
# Note that if you have unusual hardware, your kernel will not have support for those.
#
# Courtesy of https://github.com/Frogging-Family/linux-tkg
#
# Incompatible with `_localmodcfg`
#
# If unsure, select `yes`
: "${_diet_kernel:=no}"

# Use -O3 to compile kernel?
#
# -O3 has been previously critised by torvalds[^1] by historically producing worse code.
#
# If the kernel breaks, disable this option.
#
# If unsure, select no
#
# [^1]: https://lore.kernel.org/lkml/CA+55aFz2sNBbZyg-_i8_Ldr2e8o9dfvdSfHHuRzVtP2VMAUWPg@mail.gmail.com/
: "${_cflags_O3:=no}"

# Enable Google's TCP BBR3
#
# It increases throughput and reduces latency of network connections. Confirmed working at scale in Google's products[^1][^2]
# https://www.phoronix.com/news/Google-BBRv3-Linux
# Note: this is not an offical Google product.
#
# If unsure, select yes
#
# [^1]: https://cloud.google.com/blog/products/networking/tcp-bbr-congestion-control-comes-to-gcp-your-internet-just-got-faster
# [^2]: and more: https://github.com/google/bbr/tree/master?tab=readme-ov-file#information-about-bbr
: "${_tcp_bbr3:=yes}"

# Tickrate configuration
#
# valid options: 1000, 750, 600, 500, 300, 250 and 100.
# IMPORTANT: if `_import_cachyos_patches` is not enabled, only 1000, 300, 250 and 100 values are available.
#
# Controls how often the timer interrupt fires. 100Hz makes the timer interrupt fire every 10ms, and 1000Hz every 1ms.
# Lower values often improves throughput, whereas higher values often improves responsiveness.
# Also, higher values may consume more power.
#
# If you have a system with >32 units of parallization (i.e. a threadripper, for instance), you will want to select 100
# as those systems may have reduced performance from the amount of timer interrupts.
#
# Most people usually have a system thats 8c/16t. In that case, you can just select 1000
#
# If unsure, select 1000
: "${_HZ_ticks:=1000}"

# Tickless configuration
#
# Tickless stops timer ticks whenever it is not needed.
#
# Available options:
# - `full`: tickless is where timer ticks stop even when a CPU is active whenever possible. it is aimed at high
#           performance low latency systems. requires special configuration to fully take advantage of. Otherwise,
#           behaves exactly the same as idle tickless.
# - `idle`: stops timer ticks when all CPUs are idle.
# - `periodic`: system always runs at `_HZ_ticks` tickrate; suitable for realtime low-latency systems where
#               predictable response time is crucial. Not energy efficient.
#
# If unsure, select idle
: "${_tickrate:=idle}"

# Preemption configuration
#
# The Linux kernel is advanced enough to yield itself at any point and give way for userspace processes, notably to
# improve responsiveness while under load.
#
# Available options:
# - `full`: provides best responsiveness to apps by allowing all kernel code to be preempted[^1]
# - `lazy`: provides good responsiveness by being less eager to preempt SCHED_NORMAL tasks
# - `voluntary`: only preempts tasks at specific points. slightly worse latency than lazy, but better
#                throughput than lazy
# - `none`: (AKA 'server preemption') provides best throughput by only allowing sysret and interrupts as
#           the only source of preemption. Often you'll still get acceptable latency, but no promises are made.
#
# NOTE: this option is ignored if `_cpusched` is set to `rt` or `rt-bore`. in that case, all kernel code
#       will be preemptable except for a few critical sections that use `raw_spinlock_t`
#
# If unsure, select `lazy`
#
# [^1]: except critical sections
: "${_preempt:=lazy}"

# Optimize kernel for a specific processor
#
# If you distribute your kernel, set this to either one of `x86-64`, `x86-64-v2` or `x86-64-v3`)
# If you want to optimize for a specific machine, use their GCC -march= value, for example, zen4 is `znver4`,
# skylake is `skylake`, etc.
# If you want to optimize for the local machine, use `native`
#
# don't worry about getting this option wrong; the compile will simply fail, showing you the available values
#
# optimizing the kernel can offer some advantages. by supplying the architecture to optimize for, the compiler
# can make better decisions on loop unrolling, code layout and inlining w.r.t. cache sizes (L1i, L2, L3)
#
# The kernel is unlikely to run any special instructions because `-mno-avx` and friends are passed through various
# kernel makefiles, preventing use of FPU related instructions except when explicitly used by code calling
# `kernel_fpu_begin()` and terminated by `kernel_fpu_end()`.
#
# Note that many crypto, compression and CRC related algorithms already have hand optimized SIMD/AVX-512 varients,
# so specifying `native` here won't make those routines significantly faster than they already are
#
# NOTE: On non x86_64 architectures, it likely won't have AVX. In the case of ARM, you have NEON instead of AVX.
#       Hence, the documentation above is mostly x86_64 only.
#
# If unsure, select `x86-64` (baseline x86_64, should work everywhere)
: "${_processor_opt:=x86-64}"

# supported cpu processor vendors
#
# NOTE: x86_64 only
#
# Useful to reduce the size of the kernel slightly by only supporting certain vendors. This will also disable more
# things, for instance, disabling AMD support will also disable AMD pstate drivers and AMD MCE support.
#
# Available options:
# - `all`:    supports all x86 cpus[^1][^2]
# - `common`: only supports Intel and AMD
# - `intel`:  only supports Intel
# - `amd`:    only supports AMD
#
# If unsure, select `all`
#
# [^1]: won't enable `CONFIG_X86_EXTENDED_PLATFORM`, because it is also
#       disabled on arch linux. do it yourself
# [^2]: techinically `all` won't actually do anything, because by default
#       the config file has them enabled by default.
: "${_supported_cpu_vendor:=all}"

# clang LTO (Link Time Optimization) mode
#
# CAUTION: If compiling on a 32-bit system without PAE, LTO may surpass the 4 GB usable memory barrier.
# IMPORTANT: LTO is experimental in the kernel. I personally run my kernels with `thin` LTO and I have no issues
#            with it, but should things break, consider disabling this optimization
#
# LTO is done in the linking phase (as implied by the name) and is a form of "Interprocedural Optimization". It often
# reduces the size of binaries through tree-shaking (dead code elimination), better inlining decisions,
# constant folding, argument promotion, function merging, hot/cold code splitting, devirtualization[^1] etc etc etc.
# Read more about this form of optimization at wikipedia: https://en.wikipedia.org/wiki/Interprocedural_optimization
#
# LTO can be pretty expensive to run. I have seen the linker use 13 GB of ram for Fat LTO.
#
# Available options:
# - `none`: Build the kernel without LTO. This will also additionally use GCC for compiling instead of clang.
# - `thin`: Use Thin LTO (multithreaded). Preferred option - usually reaches about the same optimization as Fat LTO
# - `full`: Use Fat LTO (singlethreaded). Theoretically better optimized executable, but will take a long time to link
#
# If unsure, select thin
#
# [^1]: Well, to my knowledge devirtualization isn't applicable in Linux's case, as C does not do virtual calls[^2]
# [^2]: Except vDSO, but to my knowledge, vDSO does not benefit from LTO because it is compiled separately.
: "${_use_llvm_lto:=thin}"

# pick the right module to build.

# You can optionally build the nvidia-open module into the kernel as a module
# Use only if you have a Turing+ GPU. If not, just install the `nvidia` package.
#
# For more information, visit https://wiki.archlinux.org/title/NVIDIA
: "${_build_nvidia_open:=no}"

# Header packaging
#
# The `linux-catgirl-edition-headers` package provides headers for the kernel, which allows users to build their own
# ad-hoc kernel modules for this kernel.
# setting to `no` is more secure in the event that someone obtains root, as any malicious actors cannot simply compile
# their own kernel object, and the kernel will reject kernel objects that are incompatable with the kernel
#
# You can be pretty sure you do not have need for this if you do not have any DKMS modules:
# `$ pacman -Q | grep -dkms`
#
# as for performance and size advantages, enabling this alongside `_disable_debugging` will enable the
# `TRIM_UNUSED_KSYMS` option which allows LTO to optimize better.
#
# If unsure, select yes
: "${_package_headers:=yes}"

# Full monolithic kernel?
#
# Remove the entire dynamic module loader infrastructure in Linux, statically linking all the modules into the kernel
# binary. This means that you cannot unload modules and you can't dynamically insert modules.
#
# Might improve boot performance and lower memory usage, but there are a few variables:
#  1. How many modules are compiled?
#  2. Do the total modules compiled & loaded at once outweigh the cost of the module loader?
#     Usually this just means "are all the modules compiled into the kernel loaded anyway?"
#  3. Are all the modules already loaded on boot? If not, this might slow down the boot.
#
# This is because kernel code/memory cannot be swapped out, meaning if you compile a ton of modules that you don't
# always need (that are normally dynamically loaded) into the kernel itself, you will still pay the price for both the
# module code and the module initialization.
#
# The utility use for this option is to create a standalone kernel binary such that you do not need to copy
# `/lib/modules/kernel-name` in order to make the kernel work. It is simply standalone.
#
# Might be useful for server and embedded systems
#
# NOTE: you will need to manually put in firmwares into the firmware loader at build time. Without it, you will boot
#       into a broken or otherwise maimed system.
#
# NOTE: this isn't a 100% guarntee that CONFIG_MODULES will be disabled; it may be switched back on by the kernel
#       build system if something requires `CONFIG_MODULES=y`
#
# WARNING: do NOT enable if you are compiling a full kernel (i.e. `_localmodcfg = no`) because ALL modules will be
#          loaded.
#
# If unsure, select no
: "${_full_monolithic:=no}"

# Automatically answer the default answer in `make prepare`
#
# Needed in CI/CD pipelines. It will automatically speeeeeed through the configurator and use the default for all
# questions.
#
# if you decide to say no, you will be presented with a bunch of options that you have to say `y`, `m`, and `n` to.
# normally, you can just hold down enter (which is what this option does), but may disable some configuration options
# that you may want.
#
# DO enable in a CI.
#
# If unsure, select no
: "${_unattended_make_prepare:=no}"

# Disable module unloading
#
# According to the documentation, module unloading makes the kernel smaller, simpler and faster.
# as the name implies, you will not be able to unload modules (so you wont be able to
# `modprobe -r i915`)
#
# If unsure, select yes
: "${_disable_module_unloading:=yes}"

# Disable watchdog driver support
#
# This is a step further to the `nowatchdog` kernel parameter; this wont compile the watchdogs
# at all, which can make the compile faster and kernel smaller (on disk).
# it won't improve performance at all if `nowatchdog` is already a kernel parameter,
# or if your system has no watchdog
#
# Desktop users should probably enable, as if the system freezes or hangs, the end user
# might end up hard resetting anyway.
#
# For mission critical infrastructure or servers, this should remain enabled to automatically reset the system
# if the system hangs, instead of relying on a human to reset the system after a hang.
#
# If unsure, select yes
: "${_disable_watchdog_driver:=yes}"

# Disable basic framebuffer driver
#
# This framebuffer is used in early boot prior to the initialization of DRM or KMS
# might improve boot speed slightly, and reduce early userspace memory usage.
#
# If the DRM drivers do not get to load, or fail to load, you will be booted into a blank screen without debugging
# information.
#
# You should say no if your system takes a bit to boot, as you would have no indication[^1] if the system fails to
# boot
#
# If unsure, select no
#
# [^1]: Fine, blinking caps lock during panic, but only one of my computers have that.
: "${_disable_simpledrm:=no}"

# Maximum amount of GPUs supported
#
# Reduces the number of GPUs the kernel will support. each gpu adds very little[^1] overhead. Useful for very
# memory constrained systems
#
# If unsure, leave at 10
#
# [^1]: kernel documentation says "The overhead for each GPU is very small." might not be worth it?
#       adding the option because i have a low memory machine i want to squeeze every last byte out
#       of it.
: "${_maximum_gpus:=10}"

# maximum amount of CPUs supported
#
# reduces the amount of CPUs[^1] the kernel will support. each CPU adds an approx ~8 KB to the kernel image and
# therefore kernel memory usage.
#
# available options:
# - `default`: does not touch this option at all. use when compiling a generic kernel or if you don't know the hardware the kernel
#              run at
# - ANY INTEGER: configures the kernel to only support x CPUs. (i.e. saying 4 will make the kernel only support 4 CPUs)
#
# If unsure, select `default`
#
# [^1]: CPUs in this case does not refer to physical CPU packages. it refers to threads[^2].
# [^2]: Or parallization units. however you want to call it.
: "${_maximum_cpus:=default}"

# Disable FUSE support
#
# FUSE allows running a custom filesystem in userspace.
#
# ntfs-3g, osu!lazer and anything else that needs to implement a filesystem usually needs this.
#
# FUSE is quite a complex component and can be disabled provided that no applications
# use it, in order to save RAM and reduce the size of the kernel
#
# TODO: maybe offer the possibility to build FUSE as a module instead of disabling it fully, so it is
#       only loaded when needed?
#
# If unsure, select no
: "${_disable_fuse:=no}"

# Disable module decompression in kernel space
#
# Moves decompression related code to userspace so the kernel does not decompress any code. This will reduce the
# amount of code in the kernel
#
# If unsure, select yes
: "${_disable_kernel_module_decompress:=yes}"

# Disable kernel bootconfig support
#
# bootconfig is a feature that allows adding additional cmdline parameters
# in a different way, it seems.
#
# Verify that you're not using this: cat /proc/bootconfig
#
# If unsure, select yes
: "${_disable_bootconfig:=yes}"

# Disable zswap
#
# Use zram instead. zswap interferes with zram because zswap works on evicted memory pages first, before zram can intercept them.
# alternatively, you may add the `zswap.enabled=0` kernel parameter to disable zswap if you will be tentatively using zswap
#
# zram is often more responsive for desktop systems as it avoids the need to read from disk for memory. zram can be
# configured to do that anyway if you really want to, through ZRAM_WRITEBACK
#
# If unsure, select yes
: "${_disable_zswap:=yes}"

# Disable hibernation
#
# I don't like hibernation - it stores your entire system memory into disk, which on SSDs, reduces its lifespan
# dramatically.
#
# Use suspend-to-ram instead. I have no clue how modern laptops are draining battery quickly while in s2idle/deep sleep.
# Also, using suspend-then-hibernate won't fix the draining issue; the keyboard is often kept active waiting for
# keypresses, for instance.
#
# There's also the fact that Linux mainline (currently) doesnt support hibernation if secure boot is enabled and the
# lockdown LSM is also active.
#
# If unsure, select yes
: "${_disable_hibernation:=yes}"

# Disable compressed initramfs
#
# This can reduce the kernel size, but probably make the kernel use less ram.
# if you do not compress your initramfs, you can set to yes. it reduces the size of the vmlinuz binary
# so more kernels or configuration data may fit in /boot
# on arch linux, the initramfs is compressed by default.
#
# If unsure, select no
: "${_no_compressed_initramfs:=no}"

# Disable Virtual Machine support
#
# Makes the kernel optimized for bare metal by no longer providing virtual machine support
# in particular, this disables KVM support, kernel guest support, xen drivers, et al.
#
# docker **desktop** (not regular docker) uses KVM.
#
# If unsure, select no
: "${_no_vm:=no}"

# No foreign partitioning schemes
#
# Don't support weird partition schemes
# This may break some systems if you use those, but chances are, if you partitioned your drive normally with something
# like cfdisk or fdisk, it should be fine
#
# If unsure, select no
: "${_advanced_partition:=no}"

# Use smaller data structures for kernel core
#
# This can reduce kernel memory usage, but can hurt performance as the data structures
# are optimized for size rather than performance.
# Only enable on a memory constrained system (such as an embedded one).
#
# If unsure, select no
: "${_smaller_page:=no}"

# Disable kexec
#
# kexec is a feature analgous to the exec(2) syscall, but in kernel space. It allows the current kernel to
# be replaced by another kernel.
#
# It is useful for kdump and making reboots faster (by bypassing a potentally slow BIOS)
#
# Disabling it can in theory improve performance because the entire kexec infrastructure would be absent, but
# in practice, i dont think any malicious programs use this.
#
# If unsure, select yes
: "${_disable_kexec:=yes}"

# Disable memory hotplug
#
# Memory hotplug code. Mostly only useful for servers and VMs that can dynamically change the amount of memory
# allocated to the VM
#
# If unsure, select yes
: "${_no_memory_hotplug:=yes}"

# Remove 16-bit legacy code support
#
# NOTE: x86_64 only
#
# Removes 16 bit compatibility. this saves 300 bytes on i386 or 6k text + 16k runtime memory on x86_64.
# Wine depends on this to run 16 bit code.
#
# If unsure, select yes
: "${_no_16bit:=yes}"

# Remove 32-bit support
#
# NOTE: x86_64 only
#
# Removes code that allows the kernel to run 32-bit binaries.
# Disabling reduces the amount of compat code in the kernel.
#
# Apps like steam require 32 bit libaries.
#
# If unsure, select no
: "${_no_32bit:=no}"

# Remove LDT syscall
#
# Removes the per-process Local Descriptor Table via `modify_ldt(2)` syscall.
# It reduces context switch overhead and reduces kernel attack surface, but
# some low level threading libaries, DOSEMU and certain wine programs use it.
#
# the kernel documentation states saying `yes` here makes sense for embedded or server kernels.
: "${_disable_ldt_syscall:=yes}"

# Disable fine granularity task level IRQ time accounting
#
# Skips reading a timestamp on each softirq and hardirq transition. saying yes here can improve performance
#
# Disabling this would make time spent in IRQs not show up in htop. Additionally, /proc/pressure/irq would
# disappear
#
# If unsure, select yes
: "${_disable_irq_time_accounting:=yes}"

# Pressure Stall Information tracking (PSI)
#
# PSI is a linux kernel feature that provides insight into resource contention. Its a nifty feature for finding
# which parts of your system are stalled.
# Those parts include: CPU; IO; memory; and IRQs (if irq time accounting is enabled, see above).
#
# it is used by some monitoring applications to proactively try to restabilize a system during
# times of high contention, such as oomd, systemd-oomd and nohang[^1]
#
# valid options:
# - `yes`: PSI tracking enabled by default; has runtime performance overhead by default
# - `optin`: require kernel parameter[^2] to enable PSI tracking; negligible runtime performance overhead by default[^3]
# - `no`: PSI tracking not compiled; no performance overhead and lower memory usage
#
# If unsure, select `no`
#
# [^1]: nohang has optional PSI integration and is not a hard dependency
# [^2]: that being `psi=1`
# [^3]: the kernel docs say that this option adds some code to task wakeup and sleep in the scheduler that may show up
#       in stress tests, but is in practice negligible
: "${_psi_mode:=no}"

# Remove legacy features
#
# (Not part of disabling 32 bit because steam isn't 'legacy' per se, even though it runs 32 bit binaries)
#
# disables:
# - `X86_VSYSCALL_EMULATION`: Required by legacy programs prior to 2013, otherwise it will segfault, citing the
#                             0xffffffffff600?00 address.
#                             it saves 7kb kernel size and 4kb pagetable memory
# - `X86_IOPL_IOPERM`: disables `ioperm()` and `iopl()` syscalls which are needed by legacy applications
# enables:
# - `CONFIG_LEGACY_VSYSCALL_NONE`: no vsyscall mappings at all to eliminate risks of ASLR bypass
#
# If unsure, select yes. if any apps crash, say no
: "${_disable_legacy_features:=yes}"

#
# Bugging
# Disable debugging features for size and performance
#

# Remove coredump support
#
# Coredumps dump the entire memory of a process to disk when a process terminates improperly, for example, SIGSEGV or
# SIGABRT. It can be caught by userspace apps like `systemd-coredumpd` to do something special.
#
# Coredumps can take up a ton of space over time. In my case, migrating from arch to artix revealed that systemd
# controlled folders were taking up 1 GB of space from coredumps.
#
# Note that many application developers may request a coredump in response to a bug report. If you anticipate that
# you will be reporting bugs, don't disable this.
#
# If unsure, select yes
: "${_no_core_dump:=yes}"

# Remove kmsg
#
# The kmsg infrastructure powers `dmesg(8)` and allows you to view kernel diagnostic/warning/error messages. It provides
# things like `printk` which is used by the kernel panic to print the diagnostic message at the `KERN_EMERG` level.
#
# Disabling this will significantly reduce the kernel size because potential log messages do not need to be stored
# in the binary and the entire kmsg ring buffer is not allocated.
#
# Note that this will make debugging the kernel nearly impossible, and if you experience problems with the kernel, you
# will most certainly need the kernel log.
#
# say no here if you want to debug the kernel (i.e. developing kernel modules) or otherwise need to use dmesg
# saying yes will make the kernel harder to debug if issues arise.
#
# If unsure, select `no`
: "${_disable_dmesg:=no}"

# Remove BUG() support
#
# WARNING BIG WARNING BIG WARNING: Filesystems also utilize the BUG() infrastructure and can prevent data loss
#
# The Linux BUG() macros detect critical failure states. If the kernel reaches an undefined state, the kernel
# would BUG() and do one of two things:
#  1. Kill the process that caused the BUG()
#  2. Kill the kernel itself (kernel panic), if the BUG() was triggered inside interrupt/scheduler context
#
# Removing BUG() will cause the kernel to ignore potentially fatal conditions, and if those fatal conditions are
# reached, you will probably corrupt data.
#
# Removing BUG() will make the macro a no-op, which will then get optimized out by the compiler, making the kernel
# smaller.
#
# If unsure, select `no`
: "${_disable_bug:=no}"

# Remove a ton of debugging information
#
# This can improve performance and reduce the kernel size
# (binary and in runtime), but can make problems much more difficult to debug.
#
# Note that a lot of debugging messages and symbols will be disabled by this, which most kernel developers will ask that
# you provide when you report issues. see `_disable_dmesg`
#
# changes:
# SLUB_DEBUG removed to reduce code size
# guess unwinder (no overhead kernel unwinder)
# scheduler debugging info disabled
# disable KALLSYMS_ALL
# no shared irq debugging
# no tracers (and no stack backtrace)
# no early printk
# if building headers disabled AND BTF type information disabled, will also disable DWARF v5 debuginfo
# disable scheduler debugging
# shrinker subsystem debugging
# PnP debug messages
#
# It is safe to disable debugging for essentally free performance. if you plan on debugging the kernel itself,
# leave this enabled.
#
# If unsure, select yes
: "${_disable_debugging:=yes}"

# Remove debugfs
#
# The debugfs filesystem is a virtual filesystem the kernel developers use to put any form of data into it that
# can be read, usually for debugging purposes (as the name implies)
#
# Setting this to yes tries to disable the debugfs, but may be re-enabled by kbuild if another module depends on it
#
# If unsure, select no
: "${_disable_debugfs:=no}"

# Remove ACPI Platform Error Interface (APEI)
#
# APEI allows error reporting to the OS. It improves NMI handling should it occur, but if you encounter these, i'd say
# you have a worse problem to deal with.
#
# You can also do error injection, but EI is mostly for kernel and hardware developers to ensure the system is robust
# enough to handle errors.
#
# As a treat, this also disables CONFIG_PSTORE (if possible)
#
# If unsure, select yes
: "${_disable_apei:=yes}"

# Remove BTF & its type generation
#
# WARNING: kernel compilation may FAIL if this and _package_headers is set to yes
#
# BTF (Berkley Packet Filter) allows bytecode execution inside the kernel.
#
# Software like docker and scx-scheds utilize BTF
#
# If unsure, select no
: "${_no_btf:=no}"

# Remove profiling support used by some profilers.
#
# This may break things like `samply` and `perf`, if you are using these, set this to no.
#
# If unsure, select yes
: "${_disable_profiling:=no}"

# Check low memory corruption
#
# Disables the kernel check for the first 64k of memory every 60 seconds. The documentation
# states it has no overhead, but im offering the choice to you anyway.
#
# If unsure, select yes
: "${_disable_low_corruption:=yes}"

# Remove lockup detection
#
# This will disable the detection of soft lockups (interrupt storms), hard lockups, hung tasks (tasks stuck in an
# uninterruptable 'D' state)
#
# Even if lockups are detected, there is no recovery mechanism and the system will remain locked up.
#
# For servers, it may be preferable to leave enabled and also additionally enable BOOTPARAM_HUNG_TASK_PANIC for
# unattended recovery
# (kernel hacking -> debug oops... -> panic (reboot) on hung tasks, in nconfig)
#
# (desktop users would be more likely to hard reset if the system freezes or becomes unresponsive, so this is just
# unnecessary overhead. see `_disable_watchdog_driver`)
#
# If unsure, select yes
: "${_disable_lockup_detection:=yes}"

#
# Unhardening
# Disable security features for performance
# Note that many of these options require purposely malicious programs or programs with bugs to actually impact your security.
#
# Note: If you use a custom config file, this will not enable any disabled security options. The default config
#       file has them all enabled by default, and these settings only disable them.
#
# Warning: I am not inviting you to make your system insecure. These are offered for embedded systemssystems or if
#          you just don't give a crap.
#

# Disable Linux Security Modules (LSM)
#
# WARNING: More popular and "just-works" distros like Ubuntu have some LSMs preconfigured. Disabling this option
#          probably _will_ reduce security or otherwise stop logging security actions (some distros use permissive
#          mode by default instead of enforcing)
#
# The LSM infrastructure provides a means for security checks to be hooked in by kernel extensions that may be set at
# boot/compile time. Usually these checks are Mandatory Access Control (MAC) extensions or other security models that
# extend/are more secure than traditional POSIX permissions.
#
# On do-it-yourself distros like arch, most LSMs (like SELinux, AppArmor, etc) are not configured and enabled by
# default.
#
# LSMs host more than MAC extensions; things like kernel_lockdown(7), loadpin, and other components that interact with
# the LSM system will also be excluded from the compile.
#
# This may break certain userspace apps that always expect certain LSMs to be active, for example, pacman emits a
# warning when landlock[^1] is not available
#
# If unsure, select no
#
# [^1]: to disable the warning, turn on the DisableSandbox option in pacman.conf
: "${_no_lsms:=no}"

# Disable kernel stack zeroing
#
# The kernel automatically initializes the kernel stacks with 0xAA to prevent stack exposure bugs. This option
# disables automatic kernel stack variable auto-initialization.
#
# Requires two things to impact your security:
#  1. A malicious program
#  2. A buggy kernel
#
# If unsure, select `no`
: "${_no_stack_zeroing:=no}"

# Erase kernel stack on sysret
#
# This reduces the lifetime of sensitive stack contents and protects the kernel against uninitialized stack variable
# exposures.
#
# Documentation states a single CPU kernel compile sees a 1% slowdown.
#
# Same security advisory (the two points) in `_no_stack_zeroing`
#
# If unsure, select `no`
: "${_no_erase_kstack:=no}"

# Disable checking of integrity linked list manipulation
#
# This disables checking for linked-list manipulation. Disabling may improve performance very slightly.
#
# interestingly, the upstream Linux kernel has this disabled by default, but Arch kernel enables this.
#
# If unsure, select no
: "${_no_checking_linkedlist_integrity:=no}"

# Disable stack corruption on schedule()
#
# Disables stack corruption checking. Documentation states the performance overhead is minimal, so take as you will.
#
# If unsure, select no
: "${_no_schedule_stack_corruption:=no}"

# Disable stack protector
#
# Disables the detection of a stack overflow in the kernel. This is a security feature and the kernel will call
# `panic()` if a kernel stack overflows.
#
# The base CONFIG_STACKPROTECTOR value increases kernel code size by 3%, and the additional CONFIG_STACKPROTECTOR_STRONG
# value increases the kernel code size by another 2%
#
# If unsure, select no
: "${_no_stack_protector:=no}"

# Disable Indirect Branch Tracking (IBT) support
#
# This is a security feature that prevents a malicious program from exploiting the kernel & changing the control flow
# of the kernel. It does this by telling the CPU that all indirect calls must land on the ENDBR instruction.
# Such bugs require an existing memory corruption bug.
#
# Disabling this will:
#  1. save 4 bytes per landing site.
#  2. slightly reduce L1i cache utilization, but its literally 4 bytes.
#  3. build the kernel faster.
#  4. make kernel binary smaller
#
# If unsure, select no
: "${_no_ibt:=no}"

# Disable hardening of the kernel slab allocater freelist
#
# This is a security feature that protects the slab cache metadata. It incurs a tiny performance penalty for security.
#
# If unsure, select no
: "${_no_harden_freelist_metadata:=no}"

# Disable CPU mitigations
#
# WARNING: You cannot turn on the mitigations at boot-time if this option is disabled.
#
# CPU mitigations protect the kernel against speculative execution flaws like meltdown, zenbleed, spectre, etc.
# Those mitigations often have a performance penalty on most systems.
#
# Disabling it reduces overhead during syscalls as conditional checks[^1] are compiled out.
#
# NOTE: zen4 (and possibly zen5) systems are designed from the ground up to handle spectre mitigations, so you may
#       actually regress performance for less security[^2].
#
# If unsure, select no
#
# [^1]: you can disable/enable mitigations as you see fit during boot-time
# [^2]: source: https://www.phoronix.com/news/AMD-Zen-4-Mitigations-Off & https://www.phoronix.com/review/amd-zen4-spectrev2
: "${_no_cpu_mitigations:=no}"

# aaaand the configurator is done. beyond this line is the actual build.

_is_lto_kernel() {
    [[ "$_use_llvm_lto" = "thin" || "$_use_llvm_lto" = "full" ]]
    return $?
}

_pkgsuffix=catgirl-edition
pkgbase="linux-$_pkgsuffix"
#_minorc=$((_minor+1))
#_rcver=rc8
pkgver=${_major}${_minor}
#_stable=${_major}.${_minor}
_stable=${_major}
#_stablerc=${_major}-${_rcver}
_srcname=linux-${pkgver}
# _srcname=linux-${_major}
pkgdesc='linux catgirl edition! meow~'
pkgrel=1
_kernver="$pkgver-$pkgrel"
_kernuname="${pkgver}-${_pkgsuffix}"
arch=('x86_64')
url="https://a-catgirl.dev"
license=('GPL-2.0-only')
options=('!strip' '!debug' '!lto')
makedepends=(
  bc
  cpio
  gettext
  libelf
  pahole
  perl
  python
  tar
  xz
  zstd
  flex
  bison
)

_patchsource_cachyos="https://raw.githubusercontent.com/cachyos/kernel-patches/master/${_major}"
_patchsource_xanmod="https://gitlab.com/xanmod/linux-patches/-/raw/master/linux-6.18.y-xanmod"
_nv_ver=590.48.01
_nv_pkg="NVIDIA-Linux-x86_64-${_nv_ver}"
_nv_open_pkg="NVIDIA-kernel-module-source-${_nv_ver}"
source=(
    "https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.xz"
    "config"
    "${_patchsource_cachyos}/all/0001-cachyos-base-all.patch"
)

# apply xanmod patchset
if [ "${_import_xanmod_patchset:=yes}" = "yes" ]; then
    source+=(
    "${_patchsource_xanmod}/xanmod/0010-XANMOD-block-Set-rq_affinity-to-force-complete-I-O-r.patch"
    "${_patchsource_xanmod}/xanmod/0014-XANMOD-mm-Raise-max_map_count-default-value.patch"
    )
fi

# LLVM makedepends
if _is_lto_kernel; then
    makedepends+=(clang llvm lld)
    source+=("${_patchsource_cachyos}/misc/dkms-clang.patch")
    BUILD_FLAGS=(
        CC=clang
        LD=ld.lld
        LLVM=1
        LLVM_IAS=1
        KCFLAGS=-march=${_processor_opt}
    )
fi

if [ "$_build_nvidia_open" = "yes" ]; then
    source+=("https://download.nvidia.com/XFree86/${_nv_open_pkg%"-$_nv_ver"}/${_nv_open_pkg}.tar.xz"
             "${_patchsource_cachyos}/misc/nvidia/0001-Enable-atomic-kernel-modesetting-by-default.patch"
             "${_patchsource_cachyos}/misc/nvidia/0002-Add-IBT-support.patch")
fi

case "$_cpusched" in
    bore|rt-bore|hardened) # Bore Scheduler
        source+=("${_patchsource_cachyos}/sched/0001-bore-cachy.patch");;&
    bmq) # Project C Scheduler
        source+=("${_patchsource_cachyos}/sched/0001-prjc-cachy.patch");;
    hardened) # Hardened Patches
        source+=("${_patchsource_cachyos}/misc/0001-hardened.patch");;
    rt|rt-bore) # RT patches
        source+=("${_patchsource_cachyos}/misc/0001-rt-i915.patch");;
esac

export KBUILD_BUILD_USER="$pkgbase"
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

_die() { error "$@" ; exit 1; }

prepare() {
    cd "$_srcname"

    echo "Setting version..."
    echo "-$pkgrel" > localversion.10-pkgrel
    echo "${pkgbase#linux}" > localversion.20-pkgname

    local src
    for src in "${source[@]}"; do
        src="${src%%::*}"
        # Skip nvidia patches
        [[ "$src" == "${_patchsource_cachyos}"/misc/nvidia/*.patch ]] && continue
        src="${src##*/}"
        src="${src%.zst}"
        [[ $src = *.patch ]] || continue
        echo "Applying patch $src..."
        patch -Np1 < "../$src"
    done

    # HACK: pkgbuild pisses me off. simply doing `source+=("patches/clear-linux-patchset.patch")`... makes pkgbuild
    # NOT search in patches/. it runs off and says this gem:
    # ==> ERROR: clear-linux-patchset.patch was not found in the build directory and is not a URL.
    # look in patches/ pLEASE
    if [[ $_import_clear_patchset == yes ]]; then
        echo "Applying patch clear-linux-patchset.patch"
        # srcdir is src/, back up a bit into root (where pkgbuild is) again
        # engineering 101
        patch -Np1 < "$srcdir/../patches/clear-linux-patchset.patch"
    fi

    echo "Setting config..."
    cp ../config .config

    # Selecting CachyOS config
    if [ "$_import_cachyos_patchset" = "yes" ]; then
        echo "Bringing in cachyos patches"
        scripts/config -e CACHY
    fi

    case "$_cpusched" in
        cachyos|bore|hardened) scripts/config -e SCHED_BORE;;
        bmq) scripts/config -e SCHED_ALT -e SCHED_BMQ;;
        eevdf) ;;
        rt) scripts/config -e PREEMPT_RT;;
        rt-bore) scripts/config -e SCHED_BORE -e PREEMPT_RT;;
        *) _die "The value $_cpusched is invalid. Choose the correct one again.";;
    esac

    echo "Selecting ${_cpusched^^} CPU scheduler..."

    # Select LLVM level
    case "$_use_llvm_lto" in
        thin) scripts/config -e LTO_CLANG_THIN;;
        full) scripts/config -e LTO_CLANG_FULL;;
        none) scripts/config -e LTO_NONE;;
        *) _die "The value '$_use_llvm_lto' is invalid. Choose the correct one again.";;
    esac

    echo "Selecting '$_use_llvm_lto' LLVM level..."

    # Select tick rate
    case "$_HZ_ticks" in
        100|250|500|600|750|1000)
            scripts/config -d HZ_300 -e "HZ_${_HZ_ticks}" --set-val HZ "${_HZ_ticks}";;
        300)
            scripts/config -e HZ_300 --set-val HZ 300;;
        *)
            _die "The value $_HZ_ticks is invalid. Choose the correct one again."
    esac

    echo "Setting tick rate to ${_HZ_ticks}Hz..."

    # Select tick type
    case "$_tickrate" in
        perodic) scripts/config -d NO_HZ_IDLE -d NO_HZ_FULL -d NO_HZ -d NO_HZ_COMMON -e HZ_PERIODIC;;
        idle) scripts/config -d HZ_PERIODIC -d NO_HZ_FULL -e NO_HZ_IDLE  -e NO_HZ -e NO_HZ_COMMON;;
        full) scripts/config -d HZ_PERIODIC -d NO_HZ_IDLE -d CONTEXT_TRACKING_FORCE -e NO_HZ_FULL_NODEF -e NO_HZ_FULL -e NO_HZ -e NO_HZ_COMMON -e CONTEXT_TRACKING;;
        *) _die "The value '$_tickrate' is invalid. Choose the correct one again.";;
    esac

    echo "Selecting '$_tickrate' tick type..."

    # Select preempt type

    # We should not set up the PREEMPT for RT kernels
    if [[ "$_cpusched" != "rt" || "$_cpusched" != "rt-bore" ]]; then
        case "$_preempt" in
            full) scripts/config -e PREEMPT_DYNAMIC -e PREEMPT -d PREEMPT_VOLUNTARY -d PREEMPT_LAZY -d PREEMPT_NONE;;
            lazy) scripts/config -e PREEMPT_DYNAMIC -d PREEMPT -d PREEMPT_VOLUNTARY -e PREEMPT_LAZY -d PREEMPT_NONE;;
            voluntary) scripts/config -d PREEMPT_DYNAMIC -d PREEMPT -e PREEMPT_VOLUNTARY -d PREEMPT_LAZY -d PREEMPT_NONE;;
            none) scripts/config -d PREEMPT_DYNAMIC -d PREEMPT -d PREEMPT_VOLUNTARY -d PREEMPT_LAZY -e PREEMPT_NONE;;
            *) _die "The value '$_preempt' is invalid. Choose the correct one again.";;
        esac

        echo "Selecting '$_preempt' preempt type..."
    fi

    ### Enable O3
    if [ "$_cflags_O3" = "yes" ]; then
        echo "Enabling KBUILD_CFLAGS -O3..."
        scripts/config -d CC_OPTIMIZE_FOR_PERFORMANCE \
            -e CC_OPTIMIZE_FOR_PERFORMANCE_O3
    fi

    # Enable bbr3
    if [ "$_tcp_bbr3" = "yes" ]; then
        echo "Disabling TCP_CONG_CUBIC..."
        scripts/config -m TCP_CONG_CUBIC \
            -d DEFAULT_CUBIC \
            -e TCP_CONG_BBR \
            -e DEFAULT_BBR \
            --set-str DEFAULT_TCP_CONG bbr \
            -m NET_SCH_FQ_CODEL \
            -e NET_SCH_FQ \
            -d CONFIG_DEFAULT_FQ_CODEL \
            -e CONFIG_DEFAULT_FQ
    fi

    echo "Selecting '$_hugepage' TRANSPARENT_HUGEPAGE config..."

    echo "Enable USER_NS_UNPRIVILEGED"
    scripts/config -e USER_NS

    # Optionally load needed modules for the make localmodconfig
    # See https://aur.archlinux.org/packages/modprobed-db
    if [[ "$_localmodcfg" = "yes" && "$_diet_kernel" = "yes" ]]; then
        _die "_localmodcfg is incompatible with _diet_kernel"
    fi

    if [ "$_localmodcfg" = "yes" ]; then
        __localmodcfg_path="$_localmodcfg_path"
    elif [ "$_diet_kernel" = "yes" ]; then
        __localmodcfg_path="../../modlist-diet.db"
    fi

    if [[ "${__localmodcfg_path:-}" ]]; then
        if [ -e "$__localmodcfg_path" ]; then
            echo "Running Steven Rostedt's make localmodconfig now"
            if [ "$_unattended_make_prepare" = "yes" ]; then
                yes "" | make "${BUILD_FLAGS[@]}" LSMOD="${__localmodcfg_path}" localmodconfig
            else
                make "${BUILD_FLAGS[@]}" LSMOD="${__localmodcfg_path}" localmodconfig
            fi
        else
            _die "No modprobed.db data found"
        fi
    fi

    # Rewrite configuration
    echo "Rewrite configuration..."
    make "${BUILD_FLAGS[@]}" prepare
    yes "" | make "${BUILD_FLAGS[@]}" config >/dev/null
    diff -u ../config .config || :

    # --- CATGIRL EDITION CFGS ---

    # needed for certain tweaks
    scripts/config -e EXPERT

    if [ "$_clearlinux_prevent_sleep_demotion" = "yes" ]; then
        echo "Prevent sleep demotion"
        scripts/config -e CPU_IDLE_PSD
    fi

    scripts/config -e CONFIG_PROCESSOR_SELECT
    case "$_supported_cpu_vendor" in
        all) ;; # nothing, already enabled
        common) scripts/config -d CPU_SUP_HYGON -d CPU_SUP_CENTAUR -d CPU_SUP_ZHAOXIN;;
        intel) scripts/config -d CPU_SUP_HYGON -d CPU_SUP_CENTAUR -d CPU_SUP_ZHAOXIN \
            -d CPU_SUP_AMD \
            -d X86_MCE_AMD \
            -d AMD_NUMA \
            -d X86_AMD_PSTATE \
            -d X86_POWERNOW_K8 \
            -d X86_AMD_PLATFORM_DEVICE;;
        amd) scripts/config -d CPU_SUP_HYGON -d CPU_SUP_CENTAUR -d CPU_SUP_ZHAOXIN \
            -d CPU_SUP_INTEL \
            -d X86_MCE_INTEL \
            -d X86_INTEL_LPSS \
            -d X86_INTEL_PSTATE;;
        *) _die "The value $_supported_cpu_vendor is invalid. Pick the right option.";;
    esac

    if [ "$_disable_module_unloading" = "yes" ]; then
        echo "Disable module unloading"
        scripts/config -d MODULE_UNLOAD
    fi

    if [ "$_disable_watchdog_driver" = "yes" ]; then
        echo "Disable watchdog"
        scripts/config -d WATCHDOG
    fi

    if [ "$_disable_simpledrm" = "yes" ]; then
        echo "Disable framebuffer & simpledrm"
        scripts/config -d DRM_SIMPLEDRM \
            -d FB_VESA \
            -d FB_EFI \
            -d DRM_EFIDRM \
            -d DRM_VESADRM
    fi

    echo "Set maximum # of GPUs"
    scripts/config --set-val CONFIG_VGA_ARB_MAX_GPUS "${_maximum_gpus}"

    case "$_maximum_cpus" in
        default) ;;
        *) scripts/config -d MAXSMP --set-val NR_CPUS $_maximum_cpus;;
    esac

    if [ "$_disable_fuse" = "yes" ]; then
        echo "Disable FUSE"
        scripts/config -d FUSE_FS
    fi

    if [ "$_disable_kernel_module_decompress" = "yes" ]; then
        echo "Disable kernel module decompression"
        scripts/config -d MODULE_DECOMPRESS
    fi

    if [ "$_disable_bootconfig" = "yes" ]; then
        echo "Disable bootconfig support"
        scripts/config -d CONFIG_BOOT_CONFIG
    fi

    if [ "$_disable_zswap" = "yes" ]; then
        echo "Disable zswap"
        scripts/config -d ZSWAP
    fi

    if [ "$_disable_hibernation" = "yes" ]; then
        echo "Disable hibernation"
        scripts/config -d HIBERNATION
    fi

    if [ "$_no_compressed_initramfs" = "yes" ]; then
        echo "Remove compressed initramfs support"
        scripts/config -d RD_GZIP \
            -d RD_BZIP2 \
            -d RD_LZMA \
            -d RD_XZ \
            -d RD_LZO \
            -d RD_LZ4 \
            -d RD_ZSTD
    fi

    if [ "$_no_vm" = "yes" ]; then
        echo "Disable virtual machine support"
        scripts/config -d VIRTUALIZATION \
            -d HYPERVISOR_GUEST \
            -d VIRTIO_MENU \
            -d VHOST_MENU \
            -d VIRT_DRIVERS \
            -d VIRTIO_BLK \
            -d PVPANIC \
            -d VIRTIO_FS
    fi

    if [ "$_no_16bit" = "yes" ]; then
        echo "Dropping 16 bit"
        scripts/config -d X86_16BIT
    fi

    if [ "$_no_32bit" = "yes" ]; then
        echo "Dropping 32-bit"
        scripts/config -d IA32_EMULATION
    fi

    if [ "$_disable_ldt_syscall" = "yes" ]; then
        echo "Disable LDT"
        scripts/config -d MODIFY_LDT_SYSCALL
    fi

    if [ "$_disable_irq_time_accounting" = "yes" ]; then
        echo "Disable irq time accounting"
        scripts/config -d IRQ_TIME_ACCOUNTING
    fi

    echo "Set PSI mode to $_psi_mode"
    case "$_psi_mode" in
        yes) ;; # nothing; default in kernel
        optin) scripts/config -e PSI_DEFAULT_DISABLED ;;
        no) scripts/config -d PSI ;;
        *) _die "The value $_psi_mode is invalid. Pick the right option." ;;
    esac

    if [ "$_disable_legacy_features" = "yes" ]; then
        echo "Disable legacy features"
        scripts/config -d CONFIG_LEGACY_VSYSCALL_XONLY -e CONFIG_LEGACY_VSYSCALL_NONE \
            -d X86_VSYSCALL_EMULATION \
            -d X86_IOPL_IOPERM
    fi

    if [ "$_advanced_partition" = "no" ]; then
        echo "Disable advanced partition"
        scripts/config -d PARTITION_ADVANCED
    fi

    if [ "$_no_core_dump" = "yes" ]; then
        echo "Disable coredumps"
        scripts/config -d COREDUMP
    fi

    if [ "$_disable_dmesg" = "yes" ]; then
        echo "Disable dmesg"
        scripts/config -d PRINTK
        scripts/config -d SYMBOLIC_ERRNAME
    fi

    if [ "$_disable_bug" = "yes" ]; then
        echo "Disable BUG()"
        scripts/config -d BUG
    fi

    echo "Disable pcspkr"
    scripts/config -d PCSPKR_PLATFORM

    if [ "$_smaller_page" = "yes" ]; then
        echo "Enable smaller sized structures"
        scripts/config -e BASE_SMALL
    fi

    if [ "$_disable_profiling" = "yes" ]; then
        echo "Disable profiling support"
        scripts/config -d PROFILING
    fi

    if [ "$_disable_lockup_detection" = "yes" ]; then
        echo "Disable lockup detection"
        scripts/config -d DETECT_HUNG_TASK \
            -d HARDLOCKUP_DETECTOR \
            -d SOFTLOCKUP_DETECTOR_INTR_STORM \
            -d SOFTLOCKUP_DETECTOR
    fi

    if [ "$_disable_low_corruption" = "yes" ]; then
        echo "Disable BIOS corruption check"
        scripts/config -d X86_CHECK_BIOS_CORRUPTION
    fi

    if [ "$_disable_kexec" = "yes" ]; then
        scripts/config -d KEXEC -d KEXEC_FILE -d KEXEC_HANDOVER
    fi

    if [ "$_disable_debugging" = "yes" ]; then
        echo "Disable debugging"
        if [ "$_no_btf" = "yes" ]; then
            echo "Disable BTF and BPF"
            if [ "$_package_headers" = "yes" ]; then
                _die "_package_headers and _no_btf enabled. compile will fail."
            fi

            # disabling BTF means we can also disable debug info generation
            scripts/config -d AF_KCM \
                -d BPF_SYSCALL \
                -d BPF_JIT \
                -d DEBUG_INFO_DWARF5 \
                -e DEBUG_INFO_NONE
        fi

        scripts/config -d KPROBES \
            -d SCHED_DEBUG \
            -d SCHEDSTATS \
            -d KALLSYMS_SELFTEST \
            -d KALLSYMS_ALL \
            -d EARLY_PRINTK \
            -d SLUB_DEBUG \
            -d FTRACE \
            -d KALLSYMS \
            -d KFENCE \
            -d STACKTRACE \
            -e UNWINDER_GUESS \
            -d PM_DEBUG \
            -d DEBUG_SHIRQ \
            -d PM_ADVANCED_DEBUG \
            -d ACPI_DEBUG \
            -d SHRINKER_DEBUG \
            -d DEBUG_MEMORY_INIT \
            -d PNP_DEBUG_MESSAGES \
            -d ATA_VERBOSE_ERROR \
            -d MAC80211_DEBUGFS \
            -d BT_DEBUGFS \
            -d BLK_DEBUG_FS \
            -d CFG80211_DEBUGFS \
            -d DEBUG_BOOT_PARAMS \
            -d DEBUG_RODATA_TEST \
            -d SCSI_CONSTANTS \
            -d SCSI_LOGGING \
            -d ZSMALLOC_STAT \
            -d OVMF_DEBUG_LOG
    fi

    if [ "$_disable_debugfs" = "yes" ]; then
        echo "Disable debugfs"
        scripts/config -d DEBUG_FS
    fi

    if [ "$_disable_apei" = "yes" ]; then
        echo "Disable APEI & pstore"
        scripts/config -d ACPI_APEI -d PSTORE
    fi

    if [ "$_package_headers" = "no" ]; then
        echo "Trim unused ksyms"
        scripts/config -e TRIM_UNUSED_KSYMS
    fi

    if [ "$_no_memory_hotplug" = "yes" ]; then
        echo "Disable memory hotplug"
        # why no  here?
        scripts/config -d MEMORY_HOTPLUG
    fi

    if [ "$_no_lsms" = "yes" ]; then
        echo "Disable LSMs"
        scripts/config -d SECURITY \
            -d SECURITYFS \
            --set-str CONFIG_LSM ""
    fi

    if [ "$_no_heap_zeroing" = "yes" ]; then
        echo "Disable heap zeroing"
        scripts/config -d CONFIG_INIT_ON_ALLOC_DEFAULT_ON
    fi

    if [ "$_no_stack_zeroing" = "yes" ]; then
        echo "Disable stack zeroing on sysenter"
        scripts/config -d CONFIG_INIT_STACK_ALL_ZERO -e CONFIG_INIT_STACK_NONE
    fi

    if [ "$_no_erase_kstack" = "yes" ]; then
        echo "Disable stack erase on sysret"
        scripts/config -d KSTACK_ERASE
    fi

    if [ "$_no_checking_linkedlist_integrity" = "yes" ]; then
        echo "Disable linkedlist integrity checking"
        scripts/config -d CONFIG_LIST_HARDENED
    fi

    if [ "$_no_schedule_stack_corruption" = "yes" ]; then
        echo "Disable stack corruption on schedule() check"
        scripts/config -d SCHED_STACK_END_CHECK
    fi

    if [ "$_no_stack_protector" = "yes" ]; then
        echo "No stack protector"
        scripts/config -d STACKPROTECTOR
    fi

    if [ "$_no_randomize_kstack_offset" = "yes" ]; then
        echo "No kstack offset randomization"
        scripts/config -d RANDOMIZE_KSTACK_OFFSET
    fi

    if [ "$_no_ibt" = "yes" ]; then
        echo "No IBT"
        scripts/config -d X86_KERNEL_IBT
    fi

    if [ "$_no_harden_freelist_metadata" = "yes" ]; then
        echo "No freelist hardening"
        scripts/config -d SLAB_FREELIST_HARDENED
    fi

    if [ "$_no_cpu_mitigations" = "yes" ]; then
        echo "No CPU mitigations"
        scripts/config -d CPU_MITIGATIONS
    fi

    if [ "$_full_monolithic" = "yes" ]; then
        echo "Full monolithic"
        scripts/config -d MODULES
    fi

    ### Prepared version
    make -s kernelrelease > version
    echo "Prepared $pkgbase version $(<version)"

    ### Running make nconfig
    [ "$_makenconfig" = "yes" ] && make "${BUILD_FLAGS[@]}" nconfig
    # fish

    ### Save configuration for later reuse
    echo "Save configuration for later reuse..."
    local basedir="$(dirname "$(readlink "${srcdir}/config")")"
    cat .config > "${basedir}/config-${pkgver}-${pkgrel}${pkgbase#linux}"

    if [ "$_build_nvidia_open" = "yes" ]; then
        patch -Np1 -i "${srcdir}/0001-Enable-atomic-kernel-modesetting-by-default.patch" -d "${srcdir}/${_nv_open_pkg}/kernel-open"
        patch -Np1 -i "${srcdir}/0002-Add-IBT-support.patch" -d "${srcdir}/${_nv_open_pkg}/"
    fi
}

build() {
    cd "$_srcname"
    make "${BUILD_FLAGS[@]}" -j"$(nproc)" all
    if [ "$_package_headers" = "yes" ]; then
        make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1
    fi

    local MODULE_FLAGS=(
       KERNEL_UNAME="${_kernuname}"
       IGNORE_PREEMPT_RT_PRESENCE=1
       SYSSRC="${srcdir}/${_srcname}"
       SYSOUT="${srcdir}/${_srcname}"
    )

    if [ "$_build_nvidia_open" = "yes" ]; then
        cd "${srcdir}/${_nv_open_pkg}"
        MODULE_FLAGS+=(IGNORE_CC_MISMATCH=yes)
        CFLAGS= CXXFLAGS= LDFLAGS= make "${BUILD_FLAGS[@]}" "${MODULE_FLAGS[@]}" -j"$(nproc)" modules
    fi
}

_package() {
    pkgdesc="linux catgirl edition! meow~"
    depends=('coreutils' 'kmod' 'initramfs')
    optdepends=('wireless-regdb: to set the correct wireless channels of your country'
                'linux-firmware: firmware images needed for some devices'
                'scx-scheds: to use sched-ext schedulers')
    provides=(WIREGUARD-MODULE KSMBD-MODULE V4L2LOOPBACK-MODULE NTSYNC-MODULE VHBA-MODULE ADIOS-MODULE)

    cd "$_srcname"

    local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

    echo "Installing boot image..."
    install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

    # Used by mkinitcpio to name the kernel
    echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

    if [ "$_full_monolithic" = "yes" ]; then
        return
    fi

    echo "Installing modules..."
    ZSTD_CLEVEL=19 make "${BUILD_FLAGS[@]}" INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
        DEPMOD=/doesnt/exist  modules_install  # Suppress depmod

    # remove build links
    rm "$modulesdir"/build
}

_package-headers() {
    pkgdesc="linux catgirl edition headers! meow~"
    depends=('pahole' "${pkgbase}")

    if [[ $_package_headers == "no" ]]; then
        echo "skipping header packaging"
        return 0
    fi

    cd "${_srcname}"
    local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

    echo "Installing build files..."
    if [ "$_package_headers" = "yes" ]; then
        install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
            localversion.* version vmlinux tools/bpf/bpftool/vmlinux.h
    fi
    install -Dt "$builddir/kernel" -m644 kernel/Makefile
    install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
    cp -t "$builddir" -a scripts
    ln -srt "$builddir" "$builddir/scripts/gdb/vmlinux-gdb.py"

    # required when STACK_VALIDATION is enabled
    install -Dt "$builddir/tools/objtool" tools/objtool/objtool

    # required when DEBUG_INFO_BTF_MODULES is enabled
    if [ -f tools/bpf/resolve_btfids/resolve_btfids ]; then
        install -Dt "$builddir"/tools/bpf/resolve_btfids tools/bpf/resolve_btfids/resolve_btfids || ( warning "$builddir/tools/bpf/resolve_btfids was not found. This is undesirable and might break dkms modules !!! Please review your config changes and consider using the provided defconfig and tweaks without further modification." && read -rp "Press enter to continue anyway" )
    fi

    echo "Installing headers..."
    cp -t "$builddir" -a include
    cp -t "$builddir/arch/x86" -a arch/x86/include
    install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

    install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
    install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

    # https://bugs.archlinux.org/task/13146
    install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

    # https://bugs.archlinux.org/task/20402
    install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
    install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
    install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

    # https://bugs.archlinux.org/task/71392
    install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

    echo "Installing KConfig files..."
    find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

    echo "Removing unneeded architectures..."
    local arch
    for arch in "$builddir"/arch/*/; do
        [[ $arch = */x86/ ]] && continue
        echo "Removing $(basename "$arch")"
        rm -r "$arch"
    done

    echo "Removing documentation..."
    rm -r "$builddir/Documentation"

    echo "Removing broken symlinks..."
    find -L "$builddir" -type l -printf 'Removing %P\n' -delete

    echo "Removing loose objects..."
    find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

    echo "Stripping build tools..."
    local file
    while read -rd '' file; do
        case "$(file -Sib "$file")" in
            application/x-sharedlib\;*)      # Libraries (.so)
                strip -v $STRIP_SHARED "$file" ;;
            application/x-archive\;*)        # Libraries (.a)
                strip -v $STRIP_STATIC "$file" ;;
            application/x-executable\;*)     # Binaries
                strip -v $STRIP_BINARIES "$file" ;;
            application/x-pie-executable\;*) # Relocatable binaries
                strip -v $STRIP_SHARED "$file" ;;
        esac
    done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

    echo "Stripping vmlinux..."
    strip -v $STRIP_STATIC "$builddir/vmlinux"

    echo "Adding symlink..."
    mkdir -p "$pkgdir/usr/src"
    ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

_package-nvidia(){
    pkgdesc="nvidia module of ${_nv_ver} driver for the ${pkgbase} kernel"
    depends=("$pkgbase=$_kernver" "nvidia-utils=${_nv_ver}" "libglvnd")
    provides=('NVIDIA-MODULE')
    conflicts=("$pkgbase-nvidia-open")
    license=('custom')

    cd "$_srcname"
    local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

    cd "${srcdir}/${_nv_pkg}"
    install -dm755 "${modulesdir}"
    install -m644 kernel/*.ko "${modulesdir}"
    install -Dt "$pkgdir/usr/share/licenses/${pkgname}" -m644 LICENSE
    find "$pkgdir" -name '*.ko' -exec zstd --rm -19 -T0 {} +
}

_package-nvidia-open(){
    pkgdesc="nvidia open modules of ${_nv_ver} driver for the ${pkgbase} kernel"
    depends=("$pkgbase=$_kernver" "nvidia-utils=${_nv_ver}" "libglvnd")
    provides=('NVIDIA-MODULE')
    conflicts=("$pkgbase-nvidia")
    license=('MIT AND GPL-2.0-only')

    cd "$_srcname"
    local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

    cd "${srcdir}/${_nv_open_pkg}"
    install -dm755 "${modulesdir}"
    install -m644 kernel-open/*.ko "${modulesdir}"
    install -Dt "$pkgdir/usr/share/licenses/${pkgname}" -m644 COPYING

    find "$pkgdir" -name '*.ko' -exec zstd --rm -19 -T0 {} +
}

pkgname=("$pkgbase")
[ "$_package_headers" = "yes" ] && pkgname+=("$pkgbase-headers")
[ "$_build_nvidia_open" = "yes" ] && pkgname+=("$pkgbase-nvidia-open")
for _p in "${pkgname[@]}"; do
    eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
    }"
done

# TODO: make this right
b2sums=('11835719804b406fe281ea1c276a84dc0cbaa808552ddcca9233d3eaeb1c001d0455c7205379b02de8e8db758c1bae6fe7ceb6697e63e3cf9ae7187dc7a9715e'
        '49f51c9ae64eb5210542a7b5e2cfa58c051c768bca1250969bc51f7efa6b54550e0c221357eb31c01a5e5184e79d87fa6772a8986c77effb431164c9b1266a0e'
        '390c7b80608e9017f752b18660cc18ad1ec69f0aab41a2edfcfc26621dcccf5c7051c9d233d9bdf1df63d5f1589549ee0ba3a30e43148509d27dafa9102c19ab'
        '83460f7c8da099f97cbee7dd7c724eec7be1b8e72640209a6a00c860d0c780b6672a8fa574270c0048f7f2da886ce4b8aacd2a433d871fcdbbaac07a48857312'
        'c7294a689f70b2a44b0c4e9f00c61dbd59dd7063ecbe18655c4e7f12e21ed7c5bb4f5169f5aa8623b1c59de7b2667facb024913ecb9f4c650dabce4e8a7e5452'
        'b8b3feb90888363c4eab359db05e120572d3ac25c18eb27fef5714d609c7cb895243d45585a150438fec0a2d595931b10966322cd956818dbd3a9b3ef412d1e8')
