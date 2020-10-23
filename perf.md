# Linux Perf (perf_events)

Linux perf is a subsystem of Linux kernel and also has a front-end tool for user space. Summary of perf quoted from [official perf wiki](https://perf.wiki.kernel.org/index.php/Main_Page):

> It can instrument CPU performance counters, tracepoints, kprobes, and uprobes (dynamic tracing). It is capable of lightweight profiling.

> The userspace perf command present a simple to use interface with commands like:
> * perf stat: obtain event counts
> * perf record: record events for later reporting
> * perf report: break down events by process, function, etc.
> * perf annotate: annotate assembly or source code with event counts
> * perf top: see live event count
> * perf bench: run different kernel microbenchmarks

And `perf script` could dump more information from `perf record` for other tools to use. There's some nice tool out there: [FlameGraph](https://github.com/brendangregg/FlameGraph).

The `perf` is like `git`, consists of many separate commands. It has a syscall `perf_event_open` to access some in-kernel facilities, and the user space tool `perf` source code is located at kernel source's *tools/perf* folder with documents.

`perf` can make use of (all?) in-kernel event sources: PMU, tracepoints, kprobe/uprobe etc. Run `perf list` to know what event sources it can use.

## Install and support

Because `perf` uses the kernel facilities to tracing/profiling and not all event sources are stable enough, it's always better to use the user space `perf` with matching kernel. Debian provides it in package `linux-perf*` and Ubuntu in package `linux-tools*`.

When setting kprobe with `perf probe`, if need to trace kernel function args and retval, vmlinux with debuginfo is needed, either by -k/--vmlinux option or user can install distribution's kernel debug package to have vmlinux installed so `perf` can find it automatically.

## Links

And other useful links, but better read the perf wiki above first:
- [Official perf wiki](https://perf.wiki.kernel.org/index.php/Main_Page)
- [Unoffical page for low level by Vince Weaver](http://web.eece.maine.edu/~vweaver/projects/perf_events/)
- [Brendan Gregg's perf examples](http://www.brendangregg.com/perf.html)