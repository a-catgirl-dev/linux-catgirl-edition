# attributions:
# **cachyos** for the base pkgbuild that i have heavily modified
# **linux-tkg** for their pkgbuilds

# set the following variables to anything but null
#
# the comments above the options offer documentation. read throughly, and do research if needed
# if unsure, ask the maintainers:
# nyameowmeow at duck dot com
# xxdr at duck dot com

# make sure to read all of the possible configuration options. there may be some things you want enabled or disabled.
# don't blindly `makepkg -scf --cleanbuild --skipchecksums` the kernel (though you could, just, its not the best)

# kernel version
# only change this if you need a specific kernel. make sure to run `rm *.patch` prior to ensure you have the right patches
#
# consider `git fetch && git pull` if you're updating the kernel instead
_major=6.14
_minor=8

# include cachyos patchset?
#
# generally, you should keep this enabled
: "${_import_cachyos_patchset:=yes}"

# select a CPU scheduler
#
# possible options & explaination:
# `bore`     is a modified EEVDF scheduler that is less fair with its scheduling in order to provide
#            better interactivity under load. recommended for desktop systems
# `bmq`      is a scheduler inspired by Google's Zircon. It is designed to be small. may benefit slow machines
#            by being more likely to fully fit in L1i cache. possibly less mature than EEVDF (or CFS) and as such
#            may not schedule as well under load (or in general?)
# `hardened` is just `bore` with some patches from linux-hardened. not supported by catgirl-edition
# `eevdf`    is the stock linux scheduler. it promises all tasks will get the CPU in an acceptable amount of time
#            provided that the CPU is not at 100% load. it is fully fair and is therefore the scheduler of choice
#            for server systems.
# `rt`       is a set of patches that run the `eevdf` scheduler to provide realtime latency (latency such as
#            the time between a car crash and airbags deploying). good for latency sensitive systems and systems
#            where latency need to be predictable
# `rt-bore`  is `rt` but runs the `bore` scheduler. more suited for desktop systems that for some reason need realtime
#            or predictable latency (like professional audio workstations.)
#
# note that it seems like realtime kernels probably wont help you lots if you play rhythm games
#
# this will only have a significant performance impact performance under load, as this affects which processes
# get the CPU.
#
# `bore` is a good choice. if you are building for a latency critical system (realtime), `rt` or `rt-bore` may
# work better if you are building for a server, set `eevdf` (the mainline linux kernel scheduler)
# latency critical servers should use `rt` instead of `rt-bore`.
#
# if unsure, select `bore`
: "${_cpusched:=bore}"

# configure the kernel via nconfig?
: "${_makenconfig:=yes}"

# use modprobed-db to reduce kernel size
# disable this option if you are building a generic kernel for everyone
#
# you will want to leave this at `yes` to reduce compile time.
# if you plan on using the same kernel on multiple different computers, you would want to include those
# modules in your modprobed_db or disable this.
#
# if unsure, select `yes` AND read this article: https://wiki.archlinux.org/title/Modprobed-db
: "${_localmodcfg:=yes}"

# where is your modprobed_db?
#
# if _localmodcfg is enabled, this value must be set or compile will fail early on.
#
# if unsure, leave as default.
: "${_localmodcfg_path:="$HOME/.config/modprobed.db"}"

# use -O3 to compile kernel?
# -O3 has been previously [critised by torvalds](https://lore.kernel.org/lkml/CA+55aFz2sNBbZyg-_i8_Ldr2e8o9dfvdSfHHuRzVtP2VMAUWPg@mail.gmail.com/) by historically producing worse code.
# if the kernel breaks, try disabling this option.
# this may hurt performance if the compiler makes mistakes.
# this can make undefined behaviour in the kernel worse, as -O3 assumes UB is impossible (even more than -O2), however, i trust the kernel maintainers.
#
# -O3 may be a safe bet, but if you want to err on the side of caution, say no
#
# if unsure, select yes
: "${_cflags_O3:=yes}"

# use `performance` governor by default?
#
# DEPRECATED: since linux-5.8.y, the `cpufreq.default_governor` cmdline parameter has been introduced. if you wish to set performance as default governor,
#             add `cpufreq.default_governor=performance` to your kernel cmdline.
#             this option will be REMOVED when linux-6.15 comes along.
#
# generally, you should let something like gamemode to set the performance governor at runtime, so leave at 'no',
#            unless you don't care about heat and electricity bills.
#            leave disabled on laptops; thermal throttling will impact performance and battery life will suffer.
: "${_perf_governor_default:=no}"

# enable Google's TCP BBR3
#
# https://www.phoronix.com/news/Google-BBRv3-Linux
# note: this is not an offical Google product.
# it increases throughput and reduces latency of network connections. confirmed to work in production at Google.
#
# if unsure, select yes
: "${_tcp_bbr3:=yes}"

# kernel tickrate
#
# valid options: 1000, 750, 600, 500, 300, 250 and 100.
#
# WARNING: if `_import_cachyos_patches` is not enabled, only 1000, 300, 250 and 100 values are available.
# IMPORTANT: if using the realtime patchset (i.e. _cpusched:=rt or rt-bore), you may want to select 1000Hz
#
# typically, 1000Hz is a good choice for most desktop users. If you're a programmer, you may like 750Hz more as it
# is balanced between throughput and responsiveness.
#
# server systems would pick between 100-300Hz, depending on the hardware and expected workloads.
#
# **100Hz**  provides excellent throughput and power energy efficiency (i.e. file server)[^1]
# **250Hz**  provides a balance between 100 and 300, if you need both power efficiency and latency.
# **300Hz**  provides good throughput for workloads that are latency sensitive (i.e. game servers)
# --- 500 and 600 are in the middle of throughput and interactiveness. you can probably fill in the blanks
# **750Hz**  provides good interactiveness but also has good throughput (i.e. compiling while using social media)
# **1000Hz** provides excellent interactiveness (latency) (i.e. multitasking several browser tabs)
#
# [^1]: this may hurt performance on systems with many cores where many timer interrupts are occuring
#
# if unsure, select 1000
: "${_HZ_ticks:=1000}"

# kernel tickrate
#
# full tickless is where timer ticks stop even when a CPU is active whenever possible. it is aimed at high performance
#               low latency systems. may require special configuration to fully take advantage of. otherwise, behaves
#               exactly the same as idle
# idle tickless stops timer ticks when all CPUs are idle. better energy efficiency
# periodic is suitable for real-time systems, but is not power efficient at all.
#
# if unsure, select idle
: "${_tickrate:=idle}"

# kernel preemption mode
#
# how should the kernel preempt?
# `full` preemption provides best responsiveness to apps by allowing all kernel code to be preempted[^1]
# `lazy` preemption provides good responsiveness by being less eager to preempt SCHED_NORMAL tasks
# `voluntary` preemption only preempts tasks at specific points. slightly worse latency than lazy, but better
#        throughput than lazy
# `none` (or also known as 'server') preemption provides best throughput by only allowing sysret and interrupts as
#        the only source of interrupts. not particually great for desktop systems
#
# NOTE: this option is ignored if `_cpusched` is set to `rt` or `rt-bore`. in that case, all kernel code
#       will be preemptable except for a few critical sections.
#
# if unsure, select `lazy`
#
# [^1]: except critical sections
: "${_preempt:=lazy}"

# manages memory allocation using larger pages (usually 2 MB) instead of 4 KB pages for performance.
#
# `always`:  always attempts hugepages for allocations. this may cause applications to use more memory than needed, if
#            the application is poorly designed and only uses a small portion of the `mmap`ed memory.
#            (applications may opt-out through special syscalls, but most apps probably don't)
# `madvise`: only attempts hugepage allocation when applications specifically requests it. this may be more
#            memory efficient as hugepages are only used when needed
#
# can't choose? you can set this at runtime:
# echo always > /sys/kernel/mm/transparent_hugepage/enabled
# or just set here and https://tryitands.ee
#
# you may perfer `madvise` if your system is low on memory (<1 GB)
#
# if unsure, select `always`
: "${_hugepage:=always}"

# optimize kernel for a specific processor
#
# this can be left empty, but will prompt when building, unless `_use_auto_optimization` is set to `yes`
#
# if you distribute your kernel, set `_use_auto_optimization` (below) to no and set this to GENERIC_V3 (for x86-64-v3), GENERIC_V2 (for x86-64-v2) or GENERIC (x86-64-v1)
#
# if you want to optimize for your current cpu, set native_amd or native_intel depending on your CPU, or if you know your exact arch (in my case, my Ryzen 7 8745HS is ZEN4, my i7-10510U is SKYLAKE, my A4-6210 is JAGUAR[^1])
#
# if unsure, select `GENERIC`
#
# [^1]: the A4-6210 is actually btver2, but the patches demand jaguar for some reason. read the `auto-cpu-optimization.sh` script for all options
: "${_processor_opt:=}"

# autodetects current CPU and optimizes for it
# the _processor_opt option will be ignored if this is set.
#
# this is essentally -march=native
#
# if you distribute your kernel, make sure to set this to n, and set `_processor_opt` to a correct value (see its documentation)
: "${_use_auto_optimization:=yes}"

# supported cpu processor vendors
#
# NOTE: this only affects x86.
#
# useful to reduce the size of the kernel slightly by only supporting certain vendors. this will also disable more things, like for instance, disabling amd support will disable amd pstate drivers and amd MCE support.
#
# valid options:
# - `all`: supports all x86 cpus[^1][^2]
# - `common`: only supports intel and amd
# - `intel`: only supports intel
# - `amd`: only supports amd
#
# if unsure, say `all`
#
# [^1]: won't enable `CONFIG_X86_EXTENDED_PLATFORM`, because it is also
#       disabled on arch linux. do it yourself
# [^2]: techinically `all` won't actually do anything, because by default
#       the config file has them enabled by default.
: "${_supported_cpu_vendor:=all}"

# clang LTO mode
#
# possible options: "none", "full", "thin"
# setting this to an option other than `none` will use clang to build the kernel. if you demand stability
# (the kernel is only offically tested with gcc), select "none"
#
# CAUTION: if compiling on a 32-bit system without PAE, selecting an LTO may surpass the 4 GB usable memory barrier.
#          full LTO takes substantally longer than thin LTO.
#
#          i've seen full LTO use about 13 GB of ram, for a localmodcfg build. it may be **significantly** higher if it is allmodcfg
#          if your system or build cluster does not have this much physical memory, do not enable this
#
# IMPORTANT: LTO in the kernel is experimental. while i have seen no issues with it, you might. if that is the case, consider disabling
#            this optimization
#
# LTO does not hurt performance or memory usage at runtime. it is an optimization that takes a long time to perform when compiling.
#
# if unsure, select thin
: "${_use_llvm_lto:=thin}"

# https://wiki.archlinux.org/title/NVIDIA
# pick the right module to build.

# build nvidia
: "${_build_nvidia:=no}"
# or build nvidia_open
: "${_build_nvidia_open:=no}"

# whether to package headers for this kernel
#
# this will package the linux-catgirl-edition-headers package which will allow users to build their own kernel modules
# for some users, there is no need for headers, since you can build it into the kernel right now.
# setting to `no` is more secure in the event that something obtains root, as any malicious actors cannot simply compile their own kernel
# object, and the kernel will reject kernel objects that are incompatable with the kernel
#
# Q: do i need headers?
# A: check if you have any `-dkms` packages. on pacman distros, the command is `pacman -Q | grep dkms`
#    if so, you probably need headers.
#
# as for performance and size advantages, enabling this alongside `_disable_debugging` will enable the TRIM_UNUSED_KSYMS
# option which allows the compiler (even better with LTO) to optimize more
#
# if unsure, say yes
: "${_package_headers:=yes}"

# automatically answer the default answer in make prepare?
#
# needed in CI (if you cant interact with it), or if there is an overwhelmingly large amount of options.
# may change kernel in ways you may not expect (but catgirl edition tweaks are applied AFTER make prepare)
#
# if you decide to say no, you will be presented with a bunch of options that you have to say `y`, `m`, and `n` to.
# normally, you can just hold down enter (which is what this option does), but may disable some configuration options
# that you may want.
#
# DO enable in a CI.
#
# if unsure, say yes
: "${_unattended_make_prepare:=no}"

# disable module unloading
#
# module unloading makes the kernel smaller, simpler and faster.
# as the name implies, you will not be able to unload modules (so you wont be able to
# modprobe -r i915)
#
# if unsure, say yes
: "${_disable_module_unloading:=yes}"

# disable watchdog driver support
#
# this is a step further to the `nowatchdog` kernel parameter; this wont compile the watchdogs
# at all, which can make the compile faster and kernel smaller (on disk).
# it won't improve performance at all if `nowatchdog` is already a kernel parameter,
# or if your system has no watchdog
#
# desktop users would be more likely to manually hard reset; server kernels should have this
# enabled, especially if there is no intervention if the system hard freezes
#
# NOTE: this will not affect the systemd watchdog.
#
# if unsure, say yes
: "${_disable_watchdog_driver:=yes}"

# disable basic framebuffer driver
#
# this framebuffer is used in early boot prior to the initialization of DRM or KMS
# might improve boot speed slightly, and reduce early userspace memory usage
# if your DRM (e.x. amdgpu, nvidia, i915) drivers dont load, you might get booted
# into a black screen
# otherwise wont change the kernel much besides memory usage (less code)
#
# if unsure, say no; this is experimental
: "${_disable_simpledrm:=no}"

# maximum amount of GPUs supported
#
# reduces the number of GPUs the kernel will support. each gpu adds a (very small)[^1]
# overhead. might be useful for very low memory systems
#
# if unsure, leave at 10
#
# [^1]: kernel documentation says "The overhead for each GPU is very small." might not be worth it?
#       adding the option because i have a low memory machine i want to squeeze every last byte out
#       of it.
: "${_maximum_gpus:=10}"

# disable FUSE support
#
# FUSE allows running a custom filesystem in userspace.
#
# ntfs-3g and osu!lazer needs this.
#
# FUSE is quite a complex component and can be disabled provided that no applications
# use it, in order to save RAM and reduce the size of the kernel
#
# TODO: maybe offer the possibility to build FUSE as a module instead of disabling it fully, so it is
#       only loaded when needed?
#
# if unsure, say no; some applications may be using it.
: "${_disable_fuse:=no}"

# disable module decompression in kernel space
#
# this option implies extra code in the kernel, which is probably a problem due to
# extra code in the kernel (higher attack surface and memory usage)
#
# if unsure, say yes
: "${_disable_kernel_module_decompress:=yes}"

# disable kernel bootconfig support
#
# bootconfig is a feature that allows adding additional cmdline parameters
# in a different way, it seems.
#
# verify that you're not using this: cat /proc/bootconfig
#
# if unsure, say yes
: "${_disable_bootconfig:=yes}"

# disable zswap
#
# use zram instead.
# never use a disk swapfile if you like performance and want the drive to last longer.
#
# if your system is very low on ram, fine. go use zswap.
#
# if unsure, say yes
: "${_disable_zswap:=yes}"

# disable hibernation
#
# as with my advice with a swapfile, you probably don't want this. disable to reduce kernel size
#
# if unsure, say yes
: "${_disable_hibernation:=yes}"

# disable compressed initramfs
#
# this can reduce the kernel size, but probably make the kernel use less ram.
# if you do not compress your initramfs, you can set to yes. it reduces the size of the vmlinuz binary
# so more kernels or configuration data may fit in /boot
# on arch linux, the initramfs is compressed by default.
#
# if unsure, say no
: "${_no_compressed_initramfs:=no}"

# disable VM support
#
# makes the kernel optimized for bare metal by no longer providing virtual machine support
# in particular, this disables KVM support, kernel guest support, xen drivers, et al.
#
# docker **desktop** (not regular docker) uses KVM, so you may not want to disable this if you use docker desktop
#
# if unsure, say yes
: "${_no_vm:=yes}"

# no foreign partitioning schemes
#
# dont support weird partition schemes
# this may break some systems if you use those but chances are, if you installed linux normally with something like
# cfdisk or fdisk or something like that as your partioning, it should be fine
#
# if unsure, say no
: "${_advanced_partition:=no}"

# use smaller data structures for kernel core
#
# this can reduce kernel memory usage, but can hurt performance as the data structures
# are optimized for size rather than performance.
# only enable on a memory constrained system (such as an embedded one)
#
# if unsure, say no
: "${_smaller_page:=no}"

# disable kexec
#
# this can in theory improve security by making it so the kernel can't execute another kernel
# however, in practice, i dont think any malicious programs use this. otherwise, this reduces
# memory usage (less code)
#
# tools like kdump use this
#
# if your BIOS is slow to initialize, you may want to look into kexec to improve rebooting performance
#
# if unsure, say yes
: "${_disable_kexec:=yes}"

# disable memory hotplug
#
# i have never seen a desktop user use this.
#
# if unsure, say yes
: "${_no_memory_hotplug:=yes}"

# disable 16-bit legacy code
#
# disables running 16 bit code on the kernel. this saves 300 bytes on i386 and 16k runtime memory on x86-64
# some very old wine programs rely on this
#
# if unsure, say yes
: "${_no_16bit:=yes}"

# disable 32-bit binaries
#
# removes code that allows the kernel to run 32-bit binaries. this can reduce
# kernel size.
#
# do not enable if you use steam, or have 32-bit apps left (i.e. lib32/multilib repo enabled?)
#
# if unsure, say no
: "${_no_32bit:=no}"

# no LDT syscall
#
# disable per-process Local Descriptor Table using modify_ldt(2) syscall.
# it reduces context switch overhead and reduces kernel attack surface, but
# some low level threading libaries, DOSEMU and certain wine programs use it.
#
# the kernel documentation states saying `yes` here makes sense for embedded or server kernels.
: "${_disable_ldt_syscall:=yes}"

# disable fine granularity task level IRQ time accounting
#
# skips reading a timestamp on each softirq and hardirq transition. saying yes
# here can slightly improve performance, however, its probably too marginal to notice, maybe.
#
# if unsure, say yes
: "${_disable_irq_time_accounting:=yes}"

#
# Bugging
# Disable debugging features for size and performance
#

# disables coredump support
#
# do not enable if you plan on debugging SIGSEGVs or other faults that generate a coredump
# you will need this if you want to investigate segfaults
#
# if you do not know what a 'segfault' is, you probably can disable this to reduce memory usage.
#
# might break systemd-coredumpd[^1]
#
# if unsure, say yes
#
# [^1]: systemd has a history of killing itself over the smallest disabled kernel feature
: "${_no_core_dump:=yes}"

# disables dmesg functionality
#
# this can make the kernel smaller, slightly faster and use less ram; there would be no need to hold
# dmesg logs
#
# say no here if you want to debug the kernel (i.e. developing kernel modules) or otherwise need to use dmesg
# saying yes will make the kernel harder to debug if issues arise.
#
# if unsure, say `no`
: "${_disable_dmesg:=no}"

# disables BUG() support
#
# setting this to `yes` will remove support for BUG() and WARN(), which can make your kernel smaller, but
# ignoring fatal conditions (e.g. null pointer deref)
#
# you may want to enable `panic_on_warn` if you disable dmesg, as there would be no way to see what is wrong.
# otherwise, if you dont care about bugs or you believe you never experience them, you might be able to get
# away with enabling this
#
# if unsure, set to the same option as `_disable_dmesg`
: "${_disable_bug:=no}"

# removes various debugging stuff
#
# this can improve performance and reduce the kernel size
# (binary and in runtime), but can make problems much more difficult to debug.
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
# it is safe to disable debugging for essentally free performance. if you plan on debugging software
# or the kernel itself, leave this enabled
#
# if unsure, say yes
: "${_disable_debugging:=yes}"

# disable BTF type generation
#
# WARNING: kernel compilation may FAIL if this and _package_headers is set to yes
#
# you should leave this on no unless you don't use any BPF/BTF features (such as sched-ext)
#
# if unsure, say no
: "${_no_btf:=no}"

# disables profiling support used by some profilers.
#
# this may break things like `samply` and `perf`, if you are using these, set this to no.
#
# if unsure, say yes
: "${_disable_profiling:=no}"

# check low memory corruption
#
# disables the kernel check for the first 64k of memory every 60 seconds. the documentation
# states it has no overhead, but im offering the choice to you anyway.
#
# if unsure, say yes
: "${_disable_low_corruption:=yes}"

# disable lockup detection
#
# this will disable the detection of soft lockups (interrupt storms), hard lockups[^1], hung tasks (stuck in an uninterruptable 'D' state)
#
# even if detected, the processor or system will stay locked up. this is only a facility to report such issues.
#
# its safe to say yes here, because its not like it'll recover from a lockup.
# for servers, it may be preferable to leave enabled and also additionally enable BOOTPARAM_HUNG_TASK_PANIC
# (kernel hacking -> debug oops... -> panic (reboot) on hung tasks, in nconfig)
#
# (desktop users would be more likely to hard reset if the system freezes or becomes unresponsive, so this is just unnecessary overhead)
#
# if unsure, say yes
: "${_disable_lockup_detection:=yes}"

#
# Unhardening
# Disable security features for performance
# Note that many of these options require purposely malicious programs or programs with bugs to actually impact your security.
#
# Note: choosing the "have security" won't enable it, if you have manually overridden it in `config`. it only disables security options.
#
# Warning: this does not imply that you should enable these in production. i am simply offering the choice.
#          so: only use in embedded or otherwise non network facing systems that ONLY run trusted code
#

# disable Linux Security Modules
#
# i am unclear of the performance improvements this offers.
# this may break certain userspace apps that make use of this, for example, pacman seems to rely on landlock[^1]
# likely to prevent unwanted script execution, and im pretty sure flatpak uses one of these.
#
# CAUTION: LSMs are standard linux kernel features, used in production kernels such as the Android kernel (thats almost a billion kernels).
# Only disable if you can't afford the extra code in ring 0
#
# if unsure, say no
#
# [^1]: to disable the warning, turn on the DisableSandbox option in pacman.conf
: "${_no_lsms:=no}"

# heap zeroing on allocation
#
# according to the documentation, disabling heap zeroing typically has <1% impact on workloads, but some synthetic (benchmark/stress)
# workloads have measured 7%.
#
# this may open up your system to information exposure exploits from previously
# used memory that has since been freed by other processes/threads. this means
# that freed memory may be freely accessable to other processes that happen to
# allocate those same regions of memory. these pieces of memory may include
# cryptographic keys or other sensitive information you want to be zeroed out.
#
# this often requires a purposely malicious or buggy program[^2] to impact your security.
#
# if unsure:
#     if you do not fully trust the apps running on your system[^1], say no
#     if building for an embedded system or system with code you trust, its safe[r] to say yes
#
# [^1]: for example, any applications with proprietary code
# [^2]: buggy as in exploitable from things like memory safety
: "${_no_heap_zeroing:=no}"

# disable stack zeroing
#
# this can improve latency and performance in cases where lots of threads are spawned
#
# same security advisory (the huge wall of text) as `_no_heap_zeroing`
#
# this often requires a purposely malicious or buggy program[^1] to impact your security.
#
# if unsure, follow advice from `_no_heap_zeroing`
#
# [^1]: buggy as in exploitable from things like memory safety
: "${_no_stack_zeroing:=no}"

# disable checking of integrity linked list manipulation
#
# this enables checking for linked-list manipulation. disabling may improve performance very slightly.
#
# interestingly, the linux kernel has this disabled by default, but arch kernel enables this.
#
# if unsure, say yes
: "${_no_checking_linkedlist_integrity:=no}"

# disable stack corruption on schedule()
#
# disables corruption checking. documentation states the performance overhead is minimal, so take as you will.
# if you have never encounted a kernel panic due to this, its probably safe to enable.
: "${_no_schedule_stack_corruption:=no}"

# disable stack protector
#
# disables the detection of a stack overflow in the kernel. this is a security feature that will call panic() if overflowed.
#
# the base CONFIG_STACKPROTECTOR value increases kernel code size by 3%, and the additional CONFIG_STACKPROTECTOR_STRONG
# value increases the kernel code size by another 2%
#
# i have never seen a stack overflow in kernel land, so it seems to be rare
# say yes only if you really need less ram usage
#
# if unsure, say no
: "${_no_stack_protector:=no}"

# disable kstack offset
#
# might improve performance idk lmao.
# makes memory corruption attacks that depend on stack address determinism or cross-syscall address exposures harder, if not flat out impossible
: "${_no_randomize_kstack_offset:=no}"

# disable IBT support
#
# disables Indirect Branch Tracking. having it enabled eliminates ENDBR instructions in the kernel, leading to very slightly better performance,
# lower ram usage and kernel size.
# typically the improvement is very marginal because CPUs are designed to deal with these.
#
# this is a security feature that prevents a malicious program from exploiting memory corruption bugs in the kernel
# to change the control flow of the kernel by telling the CPU that all indirect calls must land on ENDBR. such bugs
# require an existing memory corruption bug.
#
# saying yes here will make the build faster
#
# if unsure, say no
: "${_no_ibt:=no}"

# and thats basically it. after comes the logic
# prepare some bleach if you plan to scroll

_is_lto_kernel() {
    [[ "$_use_llvm_lto" = "thin" || "$_use_llvm_lto" = "full" ]]
    return $?
}

_pkgsuffix=catgirl-edition
pkgbase="linux-$_pkgsuffix"
#_minorc=$((_minor+1))
#_rcver=rc8
pkgver=${_major}.${_minor}
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
)

_patchsource="https://raw.githubusercontent.com/cachyos/kernel-patches/master/${_major}"
_nv_ver=570.133.07
_nv_pkg="NVIDIA-Linux-x86_64-${_nv_ver}"
_nv_open_pkg="NVIDIA-kernel-module-source-${_nv_ver}"
source=(
    "https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.xz"
    "config"
    "auto-cpu-optimization.sh"
    "${_patchsource}/all/0001-cachyos-base-all.patch")

# LLVM makedepends
if _is_lto_kernel; then
    makedepends+=(clang llvm lld)
    source+=("${_patchsource}/misc/dkms-clang.patch")
    BUILD_FLAGS=(
        CC=clang
        LD=ld.lld
        LLVM=1
        LLVM_IAS=1
    )
fi

# NVIDIA pre-build module support
if [ "$_build_nvidia" = "yes" ]; then
    source+=("https://us.download.nvidia.com/XFree86/Linux-x86_64/${_nv_ver}/${_nv_pkg}.run"
             "${_patchsource}/misc/nvidia/0001-Make-modeset-and-fbdev-default-enabled.patch")
fi

if [ "$_build_nvidia_open" = "yes" ]; then
    source+=("https://download.nvidia.com/XFree86/${_nv_open_pkg%"-$_nv_ver"}/${_nv_open_pkg}.tar.xz"
             "${_patchsource}/misc/nvidia/0001-Make-modeset-and-fbdev-default-enabled.patch"
             "${_patchsource}/misc/nvidia/0003-Add-IBT-Support.patch")
fi

case "$_cpusched" in
    bore|rt-bore|hardened) # Bore Scheduler
        source+=("${_patchsource}/sched/0001-bore-cachy.patch");;&
    bmq) # Project C Scheduler
        source+=("${_patchsource}/sched/0001-prjc-cachy.patch");;
    hardened) # Hardened Patches
        source+=("${_patchsource}/misc/0001-hardened.patch");;
    rt|rt-bore) # RT patches
        source+=("${_patchsource}/misc/0001-rt-i915.patch");;
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
        [[ "$src" == "${_patchsource}"/misc/nvidia/*.patch ]] && continue
        src="${src##*/}"
        src="${src%.zst}"
        [[ $src = *.patch ]] || continue
        echo "Applying patch $src..."
        patch -Np1 < "../$src"
    done

    echo "Setting config..."
    cp ../config .config

    if [ -n "$_processor_opt" ]; then
        MARCH="${_processor_opt^^}"

        if [ "$MARCH" != "GENERIC" ]; then
            if [[ "$MARCH" =~ GENERIC_V[1-4] ]]; then
                X86_64_LEVEL="${MARCH//GENERIC_V}"
                scripts/config --set-val X86_64_VERSION "${X86_64_LEVEL}"
            else
                scripts/config -k -d GENERIC_CPU
                scripts/config -k -e "M${MARCH}"
            fi
        fi
    fi

    if [ "$_use_auto_optimization" = "yes" ]; then
        "${srcdir}"/auto-cpu-optimization.sh
    fi

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

    # Select performance governor
    if [ "$_perf_governor_default" = "yes" ]; then
        echo "Setting performance governor..."
        scripts/config -d CPU_FREQ_DEFAULT_GOV_SCHEDUTIL \
            -e CPU_FREQ_DEFAULT_GOV_PERFORMANCE
    fi

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

    ### CI-only stuff
    # if [[ -n "$CI" || -n "$GITHUB_RUN_ID" ]]; then
    #     echo "Detected build inside CI"
    #     scripts/config -d CC_OPTIMIZE_FOR_PERFORMANCE \
    #         -d CC_OPTIMIZE_FOR_PERFORMANCE_O3 \
    #         -e CC_OPTIMIZE_FOR_SIZE \
    #         -d SLUB_DEBUG \
    #         -d PM_DEBUG \
    #         -d PM_ADVANCED_DEBUG \
    #         -d PM_SLEEP_DEBUG \
    #         -d ACPI_DEBUG \
    #         -d LATENCYTOP \
    #         -d SCHED_DEBUG \
    #         -d DEBUG_PREEMPT
    # fi

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
            -d DEFAULT_FQ_CODEL \
            -e DEFAULT_FQ
    fi

    # Select THP
    case "$_hugepage" in
        always) scripts/config -d TRANSPARENT_HUGEPAGE_MADVISE -e TRANSPARENT_HUGEPAGE_ALWAYS;;
        madvise) scripts/config -d TRANSPARENT_HUGEPAGE_ALWAYS -e TRANSPARENT_HUGEPAGE_MADVISE;;
        *) _die "The value '$_hugepage' is invalid. Choose the correct one again.";;
    esac

    echo "Selecting '$_hugepage' TRANSPARENT_HUGEPAGE config..."

    echo "Enable USER_NS_UNPRIVILEGED"
    scripts/config -e USER_NS

    # Optionally load needed modules for the make localmodconfig
    # See https://aur.archlinux.org/packages/modprobed-db
    if [ "$_localmodcfg" = "yes" ]; then
        if [ -e "$_localmodcfg_path" ]; then
            echo "Running Steven Rostedt's make localmodconfig now"
            if [ "$_unattended_make_prepare" = "yes" ]; then
                yes "" | make "${BUILD_FLAGS[@]}" LSMOD="${_localmodcfg_path}" localmodconfig
            else
                make "${BUILD_FLAGS[@]}" LSMOD="${_localmodcfg_path}" localmodconfig
            fi
        else
            _die "No modprobed.db data found"
        fi
    fi
    # fish

    # Rewrite configuration
    echo "Rewrite configuration..."
    make "${BUILD_FLAGS[@]}" prepare
    yes "" | make "${BUILD_FLAGS[@]}" config >/dev/null
    diff -u ../config .config || :

    # --- CATGIRL EDITION CFGS ---

    # needed for certain tweaks
    scripts/config -e EXPERT

    # if [ "$_supported_cpu_vendor" ]
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
        scripts/config -d DRM_SIMPLEDRM
        scripts/config -d FB_VESA
        scripts/config -d FB_EFI
    fi

    echo "Set maximum # of GPUs"
    scripts/config --set-val CONFIG_VGA_ARB_MAX_GPUS "${_maximum_gpus}"

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
            -d HYPERVISOR_GUEST

        # disable drivers as we wont be in a vm
        scripts/config -d VIRTIO_MENU \
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
        scripts/config -d KEXEC -d KEXEC_FILE
    fi

    if [ "$_disable_debugging" = "yes" ]; then
        echo "Disable debugging"
        if [ "$_no_btf" = "yes" ]; then
            echo "Disable BTF and BPF"
            if [ "$_package_headers" = "yes" ]; then
                _die "_package_headers and _no_btf enabled. compile will fail."
            fi
            scripts/config -d AF_KCM \
                -d BPF_SYSCALL \
                -d BPF_JIT \
            # we can disable generating debug information :fire:
            # reduces compile time
            scripts/config -d DEBUG_INFO_DWARF5 -e DEBUG_INFO_NONE
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
            -d DEBUG_RODATA_TEST

        # TODO: does BLK_DEBUG_FS disable changing io scheduler at runtime?
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
        scripts/config -d SECURITY_SELINUX \
            -d SECURITY_SMACK \
            -d SECURITY_TOMOYO \
            -d SECURITY_APPARMOR \
            -d SECURITY_LOADPIN \
            -d SECURITY_YAMA \
            -d SECURITY_SAFESETID \
            -d SECURITY_LOCKDOWN_LSM \
            -d SECURITY_LANDLOCK \
            -d SECURITY_IPE \
            -d INTEGRITY \
            -d SECURITYFS \
            --set-str CONFIG_LSM ""
    fi

    if [ "$_no_heap_zeroing" = "yes" ]; then
        echo "Disable heap zeroing"
        scripts/config -d CONFIG_INIT_ON_ALLOC_DEFAULT_ON
    fi

    if [ "$_no_stack_zeroing" = "yes" ]; then
        echo "Disable stack zeroing"
        scripts/config -d CONFIG_INIT_STACK_ALL_ZERO -e CONFIG_INIT_STACK_NONE
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

    # # Rewrite configuration
    # echo "Rewrite configuration..."
    # make "${BUILD_FLAGS[@]}" prepare
    # yes "" | make "${BUILD_FLAGS[@]}" config >/dev/null
    # diff -u ../config .config || :

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

    if [ "$_build_nvidia" = "yes" ]; then
        cd "${srcdir}"
        sh "${_nv_pkg}.run" --extract-only

        # Use fbdev and modeset as default
        patch -Np1 -i "${srcdir}/0001-Make-modeset-and-fbdev-default-enabled.patch" -d "${srcdir}/${_nv_pkg}/kernel"
    fi

    if [ "$_build_nvidia_open" = "yes" ]; then
        # Use fbdev and modeset as default
        patch -Np1 -i "${srcdir}/0001-Make-modeset-and-fbdev-default-enabled.patch" \
            -d "${srcdir}/${_nv_open_pkg}/kernel-open"
        # Fix for https://bugs.archlinux.org/task/74886
        patch -Np1 --no-backup-if-mismatch -i "${srcdir}/0003-Add-IBT-Support.patch" \
            -d "${srcdir}/${_nv_open_pkg}"
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
    if [ "$_build_nvidia" = "yes" ]; then
        MODULE_FLAGS+=(NV_EXCLUDE_BUILD_MODULES='__EXCLUDE_MODULES')
        cd "${srcdir}/${_nv_pkg}/kernel"
        make "${BUILD_FLAGS[@]}" "${MODULE_FLAGS[@]}" -j"$(nproc)" modules

    fi

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
    provides=(WIREGUARD-MODULE KSMBD-MODULE UKSMD-BUILTIN NTSYNC-MODULE VHBA-MODULE ADIOS-MODULE)

    cd "$_srcname"

    local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

    echo "Installing boot image..."
    install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

    # Used by mkinitcpio to name the kernel
    echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

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
pkgname+=("$pkgbase-headers")
[ "$_build_nvidia" = "yes" ] && pkgname+=("$pkgbase-nvidia")
[ "$_build_nvidia_open" = "yes" ] && pkgname+=("$pkgbase-nvidia-open")
for _p in "${pkgname[@]}"; do
    eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
    }"
done

b2sums=('11835719804b406fe281ea1c276a84dc0cbaa808552ddcca9233d3eaeb1c001d0455c7205379b02de8e8db758c1bae6fe7ceb6697e63e3cf9ae7187dc7a9715e'
        '49f51c9ae64eb5210542a7b5e2cfa58c051c768bca1250969bc51f7efa6b54550e0c221357eb31c01a5e5184e79d87fa6772a8986c77effb431164c9b1266a0e'
        '390c7b80608e9017f752b18660cc18ad1ec69f0aab41a2edfcfc26621dcccf5c7051c9d233d9bdf1df63d5f1589549ee0ba3a30e43148509d27dafa9102c19ab'
        '83460f7c8da099f97cbee7dd7c724eec7be1b8e72640209a6a00c860d0c780b6672a8fa574270c0048f7f2da886ce4b8aacd2a433d871fcdbbaac07a48857312'
        'c7294a689f70b2a44b0c4e9f00c61dbd59dd7063ecbe18655c4e7f12e21ed7c5bb4f5169f5aa8623b1c59de7b2667facb024913ecb9f4c650dabce4e8a7e5452'
        'b8b3feb90888363c4eab359db05e120572d3ac25c18eb27fef5714d609c7cb895243d45585a150438fec0a2d595931b10966322cd956818dbd3a9b3ef412d1e8')
