# procs

**procs** is a replacement for `ps` written by [Rust](https://www.rust-lang.org/).

[![Build Status](https://travis-ci.org/dalance/procs.svg?branch=master)](https://travis-ci.org/dalance/procs)
[![Crates.io](https://img.shields.io/crates/v/procs.svg)](https://crates.io/crates/procs)
[![codecov](https://codecov.io/gh/dalance/procs/branch/master/graph/badge.svg)](https://codecov.io/gh/dalance/procs)

## Features

- Output by the colored and human-readable format
- Keyword search over multi-column
- Some additional information which are not supported by `ps`
    - TCP/UDP port
    - Read/Write throughput
    - Docker container name
- Pager support

## Platform

Linux is supported only.

## Installation

### Download binary

Download from [release page](https://github.com/dalance/procs/releases/latest), and extract to the directory in PATH.

### Cargo

You can install by [cargo](https://crates.io).

```
cargo install procs
```

## Usage

### Show all processes

Type `procs` only. It shows the information of all processes.

```console
$ procs
```

![procs](https://user-images.githubusercontent.com/4331004/51976289-9f3a4a00-24c7-11e9-8573-8f746ccf1ed4.png)

### Search by non-numeric keyword

If you add any keyword as argument, it is matched to `USER`, `Command`, `Docker` by default.

```console
$ procs zsh
```

![procs_zsh](https://user-images.githubusercontent.com/4331004/51976319-b24d1a00-24c7-11e9-8060-09e71d18e5ec.png)

### Search by numeric keyword

If a numeric is used as the keyword, it is matched to `PID`, `TCP`, `UDP` by default.
Numeric is treated as exact match, and non-numeric is treated as partial match by default.

```console
$ procs 6000 60000 60001 16723
```

![procs_port](https://user-images.githubusercontent.com/4331004/51976347-c09b3600-24c7-11e9-8d40-2c437efbd5e1.png)

Note that procfs permissions only allow identifying listening ports for processes owned by the current user, so not all ports will show up unless run as root.

### Show Docker container name

If you have access permission to docker daemon ( `unix:///var/run/docker.sock` ), `Docker` column is added.

```console
$ procs growi
```

![procs_docker](https://user-images.githubusercontent.com/4331004/52265847-4d3a6e00-2978-11e9-8186-ea8e934acbb1.png)

### Pager

If output lines exceed terminal height, pager is used automatically.
This behavior and pager command can be specified by configuration file.

## Configuration

You can change configuration by `~/.procs.toml` like below.
The complete example of `~/.procs.toml` can be generated by `--config` option.

```toml
[[columns]]
kind = "Pid"
style = "BrightYellow"
numeric_search = true
nonnumeric_search = false

[[columns]]
kind = "Username"
style = "BrightGreen"
numeric_search = false
nonnumeric_search = true

[style]
header = "BrightWhite"
unit = "BrightWhite"

[style.by_percentage]
color_000 = "BrightBlue"
color_025 = "BrightGreen"
color_050 = "BrightYellow"
color_075 = "BrightRed"
color_100 = "BrightRed"

[style.by_state]
color_d = "BrightRed"
color_r = "BrightGreen"
color_s = "BrightBlue"
color_t = "BrightCyan"
color_z = "BrightMagenta"
color_x = "BrightMagenta"
color_k = "BrightYellow"
color_w = "BrightYellow"
color_p = "BrightYellow"

[style.by_unit]
color_k = "BrightBlue"
color_m = "BrightGreen"
color_g = "BrightYellow"
color_t = "BrightRed"
color_p = "BrightRed"
color_x = "BrightBlue"

[search]
numeric_search = "Exact"
nonnumeric_search = "Partial"

[display]
show_self = false
cut_to_terminal = true
cut_to_pager = false
cut_to_pipe = false
color_mode = "Auto"

[sort]
column = 0
order = "Ascending"

[docker]
path = "unix:///var/run/docker.sock"

[pager]
mode = "Auto"
```

### `[[columns]]` section

`[[columns]]` section defines which columns are used.
The first `[[columns]]` is shown at left side, and the last is shown at right side.
`kind` is column type and `style` is column color.
`numeric_search` and `nonnumeric_search` mean whether this column can be matched by numeric/non-numeric search keyword.
The available list of `kind` and `style` is below.

There are some special styles like `ByPercentage`, `ByState`, `ByUnit`.
These are the styles for value-aware coloring.
For example, if `ByUnit` is choosen, color can be specified for each unit of value ( like `K`, `M`, `G`,,, ).
The colors can be configured in `[style.by_unit]` section.

#### `kind` list

| procs `kind` | `ps` STANDARD FORMAT  | Description                      |
| ------------ | --------------------- | -------------------------------- |
| Command      | args                  | Command with all arguments       |
| CpuTime      | cputime               | Cumulative CPU time              |
| Docker       | -not supported-       | Docker container name            |
| Eip          | eip                   | Instruction pointer              |
| Esp          | esp                   | Stack pointer                    |
| Gid          | egid                  | Group ID                         |
| GidFs        | fgid                  | File system group ID             |
| GidReal      | rgid                  | Real group ID                    |
| GidSaved     | sgid                  | Saved group ID                   |
| Group        | egroup                | Group name                       |
| GroupFs      | fgroup                | File system group name           |
| GroupReal    | rgroup                | Real group name                  |
| GroupSaved   | sgroup                | Saved group name                 |
| MajFlt       | maj_flt               | Major page fault count           |
| MinFlt       | min_flt               | Minor page fault count           |
| Nice         | ni                    | Nice value                       |
| Pid          | pid                   | Process ID                       |
| Ppid         | ppid                  | Parent process ID                |
| Priority     | pri                   | Priority                         |
| Processor    | psr                   | Currently assigned processor     |
| ReadBytes    | -not supported-       | Read bytes from storage          |
| RtPriority   | rtprio                | Real-time priority               |
| Separator    | -not supported-       | Show `|` for column separation   |
| StartTime    | start_time            | Starting time                    |
| State        | s                     | Process State                    |
| TcpPort      | -not supported-       | Binded TCP ports                 |
| Threads      | nlwp                  | Thread count                     |
| Tty          | tty                   | Controlling TTY                  |
| UdpPort      | -not supported-       | Binded UDP ports                 |
| Uid          | euid                  | User ID                          |
| UidFs        | fuid                  | File system user ID              |
| UidReal      | ruid                  | Real user ID                     |
| UidSaved     | suid                  | Saved user ID                    |
| UsageCpu     | %cpu                  | CPU utilization                  |
| UsageMem     | %mem                  | Memory utilization               |
| User         | euser                 | User name                        |
| UserFs       | fuser                 | File system user name            |
| UserReal     | ruser                 | Real user name                   |
| UserSaved    | suser                 | Saved user name                  |
| VmData       | -not supported-       | Data size                        |
| VmExe        | trs                   | Text segments size               |
| VmHwm        | -not supported-       | Peak resident set size           |
| VmLib        | -not supported-       | Library code size                |
| VmLock       | -not supported-       | Locked memory size               |
| VmPeak       | -not supported-       | Peak virtual memory size         |
| VmPin        | -not supported-       | Pinned memory size               |
| VmPte        | -not supported-       | Page table entries size          |
| VmRss        | rss                   | Resident set size                |
| VmSize       | vsz                   | Physical page size               |
| VmStack      | -not supported-       | Stack size                       |
| VmSwap       | -not supported-       | Swapped-out virtual memory size  |
| Wchan        | wchan                 | Process sleeping kernel function |
| WriteByte    | -not supported-       | Write bytes to storage           |

#### `style` list

- BrightRed
- BrightGreen
- BrightYellow
- BrightBlue
- BrightMagenta
- BrightCyan
- BrightWhite
- Red
- Green
- Yellow
- Blue
- Magenta
- Cyan
- White
- ByPercentage
- ByState
- ByUnit

### `[style]` section

`[style]` section defines colors of header and unit line.
The available list of color is below.

#### `color` list

- BrightRed
- BrightGreen
- BrightYellow
- BrightBlue
- BrightMagenta
- BrightCyan
- BrightWhite
- Red
- Green
- Yellow
- Blue
- Magenta
- Cyan
- White

### `[style.by_*]` section

`[style.by_*]` section defines colors of special styles like `ByPercentage`, `ByState`, `ByUnit`.
The available list of color is the same as the list of `[style]` section.

### `[search]` section

`[search]` section defines match policy. Policy can be `Exact` or `Partial`.

### `[display]` section

`[display]` section defines option for display.
`show_self` means whether the self ( `procs` ) process is shown.
`cut_to_*` means whether output lines is truncated upto terminal size.
`color_mode` means the default behavior of output coloring without `--color` commandline option. This can be `Auto`, `Always` or `Disable`.

### `[sort]` section

`[sort]` section defines the column used for sort and sort order.
If `column` is 0, value is sorted by left column.
`order` can be `Ascending` or `Descending`.

### `[docker]` section

`[docker]` section defines how to communicate to docker daemon.
`path` means UNIX domain socket to docker daemon.

### `[pager]` section

`[pager]` section defines the behavior of pager.
`mode` means the default behavior of pager usage without `--pager` commandline option. This can be `Auto`, `Always` or `Disable`.
If `Auto`, pager is used only when output lines exceed terminal height.
Default pager is `less -SR` ( if `less` is not found, `more -f` ),  but you can specify pager comand like below:

```.procs.toml
[pager]
mode = "Auto"
command = "less"
```

If `more` is used, `-f` option is recommended.

```.procs.toml
[pager]
mode = "Auto"
command = "more -f"
```
