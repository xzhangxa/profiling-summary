# Profiling/tracing Event Sources Summary on Linux

| Source | SW/HW | Dynamic/static | Kernel/user | Front-end |
| --- | --- | --- | --- | --- |
| PMU [^1] | HW | static | | |
| procfs/sysfs | kernel source | static | kernel/user | |
| kprobe | kernel source | dynamic | kernel | |
| uprobe | kernel source | dynamic | user | |
| tracepoints | kernel source | static | kernel | |
| USDT [^2] | kernel source | static | user | |

[^1]: Performance Monitoring Unit in CPU HW, or called PMC (Performance Monitoring Counters).

[^2]: User-level Statically Defined Tracking, tracepoints for user space.

# PMU

CPU provides facilities for monitoring perfermance by Performance Monitoring Unit (PMU). Intel has provided document about PMU in [Intel Software Developer's Manual Vol.3B](https://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-software-developer-vol-3b-part-2-manual.html). AMD has its version as well. The events are the only and final sources for many HW information.

The events can be counted, and sampled.
- The counting mode is simple, HW will record the events so later it could be read.
- The sampling mode works as the user sets the sampling frequency or number of occurrences of the event, PMU interrupts the CPU as the sampling counter overflow.

The events differ by architectures and can be disabled in VM guest. Normal user could use user space tools to use it indirectly. Also if need to check exact some of them, run `perf list` and see items marked like [Hardware cache event] or [Kernel PMU event] to know what your machine provides and `perf` can use.

# procfs/sysfs

Linux kernel by default provides many information about kernel itself and user processes, also many kernel subsystems and device drivers provide useful information via sysfs. Linux kernel exports some internal information via procfs/sysfs so user space can use them.

There are too many of them so it's not possible to list here, and many are used by front-end tools so users don't need to check some attributes in sysfs, also sysfs is not intened as a stable API of the kernel. However, device driver developers may find it very useful since many kernel subsystems provide valueable information via sysfs.

# Kprobe

Dynamically trap and run a handler function on kernel code address. Two types: kprobe and kretprobe (for function return).

Detailed introduction in kernel doc of [kprobe](https://www.kernel.org/doc/html/latest/trace/kprobes.html).

## How it works

At kernel runtime, bytes from the target address are copied and saved by kprobes, and replaced with breakpoint instruction (int3 on x86) or jmp (if possible).

When instruction flow hits this, kprobe handler is executed and then jump back. For function return kretprobe, function entry kprobe is hit and it will save a substitute function to run kprobe handler when function returns.

## How to use

- Write your own module and use `register_kprobe` function or family;
- Ftrace sysfs interface in */sys/kernel/debug/tracing* (kernel doc [kprobetrace](https://www.kernel.org/doc/html/latest/trace/kprobetrace.html));
- `perf_event_open` syscall;
- Or more conveniently, use one of the many front-end tools: perf, SystemTap, Ftrace tools, BPF tracer BCC and bpftrace.

## Limitations

- Not possible to probe anywhere: not in kprobe code itself, kprobe blacklist;
- May not hit if probe on inline functions;
- Not useable if kernel test section is read only for security reasons;
- Don't step on your own tail: calling a function in probe handler that it just is probing on.

# Uprobe

## How it works

Similar as kprobe, but for user space ELF files. A fast breakpoint inserted to the target instruction then execute uprobe handler.

Uprobe is file based, so it means for a function in a library file is traced, the system-wide processes are traced.

## How to use

- Ftrace sysfs interface in */sys/kernel/debug/tracing* (kernel doc [uprobetracer](https://www.kernel.org/doc/html/latest/trace/uprobetracer.html));
- `perf_event_open` syscall;
- More conveniently, use one of the many front-end tools: perf, BPF traces BCC and bpftrace.
