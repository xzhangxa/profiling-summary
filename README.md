# A poor summary of profiling methods

**!!!WIP!!!**

This repo is a poor and incomplete summary and guide for profiling methods/tools and some background knowledge. It is for the purpose of personal collection only thus limited by my knowledge/skill and information collection ability. Also I am by no way able to learn and use all methods mentioned, as it's mainly a summary for later reference.

It's written as notes first but later I guess it's a good idea to organize the summary as markdown files with git history, so here it goes.

## Background

These pages summary background tracing/profiling event sources. These sources are how the profiling tools get the information, from kernel subsystem, compiler support and/or hardware. A few many be directly used by users but it's recommanded to use a front-end tool instead.
- [Event Sources Summary on Linux](./event_sources.md)
- [Supporting Information](./supporting.md)

## Guide for choosing profiling/tracing tools

TBD.

## General-purpose tools

A list from Wikipedia: https://en.wikipedia.org/wiki/List_of_performance_analysis_tools. Many of them are proprietaries or out of date, but it's good to check it for your case.

### Guide to choose among general-purpose tools

TBD. This section guides user which tool to use.

Before check which general-purpose tool to use, it's recommanded to read the [Background](#background) section first, especially the [Event Sources](./event_sources.md) page to know what tracing sources you can use. Because eventually the data are from them and any general-purpose tool depends on your setting to trace from some sources.

### List of some powerful tools

- Linux perf (perf_events): [brief summary](./perf.md)
- BPF (bpftrace & BCC): [brief summary](./bpf.md)
- Intel Vtune profiler, Intel Advisor: TBD
- LTTng: TBD

## Single-purpose tools and procfs/sysfs files for quick check

Many single-purpose cmdline tools are available for profiling/tracing. Also some simple virtual files of procfs/sysfs can be read directly. Here lists some of them by target categories.

### CPU Load/Process

- `uptime`, loadavg from */proc/loadavg*;
- `top`, `htop`;
- `mpstat` for CPU time in different states, `pidstat` similar to `top` and can run in rolling mode. Both from [sysstat](https://github.com/sysstat/sysstat).

### Memory

- */proc/iomem*, */proc/ioports*, */proc/meminfo*, */proc/zoneinfo*;
- `vmstat`, `free`, also `top/htop` for per-process information;
- `slabtop`, data from */proc/slabinfo*.

### Filesystem

- `df`, `mount`, `du`, `strace`;
- `lsof` for opened files;
- `fatrace` using Linux `fanotify` API for trace file accesses (open/close/read/write/create/delete/move_from/move_to).

### Disk IO

- `blktrace`;
- `iostat` from [sysstat](https://github.com/sysstat/sysstat).

### Network

- */proc/net/\**;
- `ss` for sockets;
- `ip`, `lnstat`;
- `tcpdump`.
