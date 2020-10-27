# Profiling/tracing Event Sources Summary on Linux

| Source | SW/HW | Dynamic/static | Kernel/user | Front-end |
| --- | --- | --- | --- | --- |
| PMU [^1] | HW | static | | perf bpftrace BCC |
| Kernel SW events | kernel source | static | kernel | perf bpftrace BCC |
| procfs/sysfs | kernel source | static | kernel/user | |
| kprobe | kernel source | dynamic | kernel | perf bpftrace BCC ftrace |
| uprobe | kernel source | dynamic | user | perf bpftrace BCC ftrace |
| tracepoint | kernel source | static | kernel | perf bpftrace BCC ftrace |
| USDT [^2] | kernel source | static | user | bpftrace BCC |

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
- Or more conveniently, use one of the many front-end tools: perf, SystemTap, Ftrace tools (perf-tools, trace-cmd), BPF tracer (BCC, bpftrace).

## Limitations

- Not possible to probe anywhere: not in kprobe code itself, kprobe blacklist;
- May not hit if probe on inline functions;
- Not useable if kernel test section is read only for security reasons;
- Don't step on your own tail: calling a function in probe handler that it just is probing on.

# Uprobe

Similar Linux kernel feature like kprobe, but for dynamically trap and run a handler function on user code address, the user can instrument on ELF file of executable or shared library. Two types: uprobe and uretprobe (for function return).

## How it works

A fast breakpoint inserted to the target instruction then execute uprobe handler. Uprobe is file based, so it means for a function in a library file is traced, the system-wide processes are traced.

## How to use

- Ftrace sysfs interface in */sys/kernel/debug/tracing* (kernel doc [uprobetracer](https://www.kernel.org/doc/html/latest/trace/uprobetracer.html));
- `perf_event_open` syscall;
- More conveniently, use one of the many front-end tools: perf, Ftrace tools (perf-tools, trace-cmd), BPF tracer (BCC, bpftrace).

# Tracepoint

Linux tracepoint is in-kernel static instrumentation point, the kernel source includes tracepoints in key places that could be used for tracing. Because they are coded into the fixed place of kernel source code, they are stable and always there to use for a given kernel version. All syscalls have its tracepoint and many subsystems define their own tracepoints in their key functions. For example Linux 5.9 has 1899 tracepoints according to `bpftrace -l`.

## How it works

Developer needs to define the tracepoint in kernel code with provided API `TRACE_EVENT` first. A nop instruction is inserted to where the tracepoint defines and a handler is generated, which later can iterate the array of registered tracepoint probe callbacks.

At runtime, when the tracepoint is not used it's just a nop instruction. When tracepoint is enabled the nop will be replaced by a jmp instruction to the handler, and the probe callback is added to the array. Multiple probe callbacks can be registered to the same tracepoint. When all probes are disabled it could be reverted back to nop.

## How to use

- Ftrace sysfs interface in */sys/kernel/debug/tracing/events* (kernel doc [tracepoint-analysis](https://www.kernel.org/doc/html/latest/trace/tracepoint-analysis.html));
- `perf_event_open` syscall;
- One of the many front-end tools: perf, Ftrace tools (perf-tools, trace-cmd), BPF tracer (BCC, bpftrace).

# USDT

USDT (User-level Statically Defined Tracking) is derived from Dtrace (from Solaris), it's like tracepoints for user space. USDT can be coded into user space executables and libraries to make it for probing later by an external tracer. A few softwares can be compiled with USDT enabled.

## How it works

The idea is similar to tracepoint in kernel space, adding nop instruction in binary and replacing to probe enabled callbacks at runtime. To define USDT tracepoints in the code, systemtap-sdt-dev or Folly C++ library can be used.

## How to use

- BPF tracer (BCC, bpftrace);
- Systemtap.
