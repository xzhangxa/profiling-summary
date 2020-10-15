# A poor summary of profiling methods

This repo is a poor and incomplete summary and guide for profiling methods/tools and some background knowledge. It is for the purpose of personal collection only thus limited by my knowledge/skill and information collection ability. Also I am by no way able to learn and use all methods mentioned, as it's mainly a summary for later reference.

It's written as notes first but later I guess it's a good idea to organize the summary as markdown files with git history, so here it goes.

## Background

These pages summary background tracing/profiling event sources. These sources are how the profiling tools get the information, from kernel subsystem, compiler support and/or hardware. A few many be directly used by users but it's recommanded to use a front-end tool instead.
- [Event Sources Summary on Linux](./event_sources.md)
- [Supporting Information](./supporting.md)

## Guide for choosing profiling/tracing tools

TBD.

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
- `fatrace` using Linux `fanotify` API for trace file accesses (ppen/close/read/write/+/delete/</>).

### Disk IO

- `blktrace`;
- `iostat` from [sysstat](https://github.com/sysstat/sysstat).

### Network

- */proc/net/\**;
- `ss` for sockets;
- `ip`, `lnstat`;
- `tcpdump`.

## General-purpose tools

This page [General-purpose Tools](./general_purpose_tools.md) summaries some tracing/profiling tools. It's not a complete list, however it tries to be a quick start for choosing a tool to use.

Before check which general-purpose tool to use, it's recommanded to read the [Background](#background) section first, especially the [Event Sources](./event_sources.md) page to know what tracing sources you can use. Because eventually the data are from them and any general-purpose tool depends on your setting to trace from some sources.
