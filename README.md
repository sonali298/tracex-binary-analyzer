# TraceX

A lightweight Linux debugger and binary-analysis tool written in C.

TraceX is a systems-programming project developed by **Jitendra and Anshu**
for exploring Linux process debugging, executable formats, memory,
registers, breakpoints, and low-level program execution.

---

## Authors

**Jitendra**  
**Anshu**
**Sonali**

---

## Overview

TraceX is an interactive command-line debugger for Linux x86-64
programs.

The project focuses on understanding how debugging tools work at the
system level using Linux process-tracing facilities, executable-file
information, and debugging metadata.

The debugger can control a target process, inspect its execution state,
set software breakpoints, examine memory and registers, and inspect
information stored inside ELF binaries.

---

## Features

### Process Control

TraceX supports basic process-control operations:

- Start a target program
- Restart a target program
- Attach to a running process
- Terminate the target process
- Continue execution
- Single-step execution
- Step over instructions
- Finish the current function

---

### Breakpoints

TraceX supports software breakpoints on x86-64 systems.

Breakpoints can be specified using:

- Memory addresses
- Function names
- Source-code line numbers
- Source file and line numbers

Examples:

```text
break main
break 0x401136
break 42
break main.c:42
```

Software breakpoints use the x86 `INT 3` instruction:

```text
0xCC
```

When a breakpoint is reached, the debugger restores the original
instruction before continuing execution.

---

### Registers

TraceX provides access to x86-64 CPU registers.

Available operations include:

```text
register dump
register read <register>
register write <register> <value>
```

Examples:

```text
register dump
register read rip
register read rsp
register write rax 10
```

Register inspection makes it possible to observe the CPU state while
the target program is being debugged.

---

### Memory Inspection

TraceX provides commands for reading and writing target-process memory.

Examples:

```text
memory read 0x401000
memory write 0x401000 0x1234
```

Memory inspection is useful for studying virtual memory and the runtime
state of a process.

---

## ELF and DWARF Support

TraceX works with Linux ELF executables and debugging information.

The project uses:

- `libelf`
- `libdw`
- DWARF debugging information

ELF information can be used to inspect:

- ELF headers
- Sections
- Symbols
- Functions

DWARF information can be used to associate machine-code addresses with
source-level information such as source files and line numbers.

---

## PIE and ASLR Support

Modern Linux programs commonly use Position Independent Executables
(PIE) and Address Space Layout Randomization (ASLR).

TraceX determines the runtime load address of the target process by
examining:

```text
/proc/<pid>/maps
```

The runtime address can then be combined with executable information
to resolve addresses correctly.

---

# Command Line Interface

TraceX provides an interactive debugger shell.

| Command                      | Description                       |
| ---------------------------- | --------------------------------- |
| `run`                        | Start the target program          |
| `restart`                    | Restart the target program        |
| `arguments <arg...>`         | Set arguments for the target      |
| `backtrace`                  | Display the function-call trace   |
| `break <target>`             | Create a breakpoint               |
| `delete <addr>`              | Delete a breakpoint               |
| `enable <addr>`              | Enable a breakpoint               |
| `disable <addr>`             | Disable a breakpoint              |
| `continue`                   | Continue execution                |
| `stepi`                      | Execute one machine instruction   |
| `step`                       | Step through source code          |
| `next`                       | Step over the current instruction |
| `finish`                     | Finish the current function       |
| `header`                     | Display ELF header information    |
| `sections <name>`            | Search ELF sections               |
| `symbols <name>`             | Search symbols                    |
| `functions <name>`           | Search functions                  |
| `register dump`              | Display all CPU registers         |
| `register read <reg>`        | Read a register                   |
| `register write <reg> <val>` | Modify a register                 |
| `memory read <addr>`         | Read process memory               |
| `memory write <addr> <val>`  | Write process memory              |
| `help`                       | Display available commands        |
| `exit`                       | Exit TraceX                       |

---

# Architecture

The debugger is organized into several logical components.

```text
                         ┌────────────────────┐
                         │     TraceX CLI     │
                         └──────────┬─────────┘
                                    │
                                    ▼
                         ┌────────────────────┐
                         │  Command Handler   │
                         └──────────┬─────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
       ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
       │  Breakpoint  │      │   Stepping   │      │  Registers   │
       │    Manager   │      │    Engine    │      │  & Memory    │
       └──────┬───────┘      └──────┬───────┘      └──────┬───────┘
              │                     │                     │
              └─────────────────────┼─────────────────────┘
                                    │
                                    ▼
                         ┌────────────────────┐
                         │     ptrace()       │
                         │   Linux Interface  │
                         └──────────┬─────────┘
                                    │
                                    ▼
                         ┌────────────────────┐
                         │   Target Process   │
                         └────────────────────┘


                         ┌────────────────────┐
                         │    ELF / DWARF     │
                         │      Analysis      │
                         └──────────┬─────────┘
                                    │
                                    ▼
                         ┌────────────────────┐
                         │ Symbol Resolution  │
                         │ Source Line Mapping│
                         └────────────────────┘
```

---

# Implementation Details

## Process Tracing

TraceX uses the Linux `ptrace()` interface to control and inspect the
target process.

Process tracing is used for operations such as:

- Stopping the process
- Continuing execution
- Single-step execution
- Reading registers
- Writing registers
- Reading memory
- Writing memory
- Handling process signals

---

## Software Breakpoint Flow

A software breakpoint works approximately as follows:

```text
          Original instruction
                   │
                   ▼
          Save original byte
                   │
                   ▼
              Write 0xCC
                   │
                   ▼
          Continue execution
                   │
                   ▼
              SIGTRAP
                   │
                   ▼
          Restore instruction
                   │
                   ▼
        Adjust instruction pointer
                   │
                   ▼
          Continue execution
```

This mechanism allows the debugger to stop execution at a selected
instruction address.

---

# Source-Level Stepping

TraceX provides both instruction-level and source-oriented stepping.

Instruction-level stepping:

```text
stepi
```

Source-oriented operations:

```text
step
next
finish
```

Debugging information can be used to associate runtime instruction
addresses with source-code locations.

---

# ELF Analysis

TraceX includes basic ELF inspection capabilities.

Currently supported areas include:

- ELF header information
- Section information
- Symbol information
- Function information

Examples:

```text
header
sections
symbols
functions
```

These facilities help connect the executable file on disk with the
program executing in memory.

---

# Build Requirements

TraceX requires a Linux development environment.

Install the required packages on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install gcc make libreadline-dev libelf-dev libdw-dev
```

Check the compiler:

```bash
gcc --version
```

---

# Building

Build the project using:

```bash
make build
```

A development build can be created using:

```bash
make
```

A debug build can be created using:

```bash
make debug
```

---

# Running TraceX

First compile a test program with debugging information:

```bash
gcc -g -O0 test.c -o myprogram
```

Then start TraceX:

```bash
./tracex ./myprogram
```

---

# Example Program

```c
#include <stdio.h>

int main(void)
{
    int a = 10;
    int b = 20;

    int result = a + b;

    printf("Result = %d\n", result);

    return 0;
}
```

Compile it:

```bash
gcc -g -O0 test.c -o myprogram
```

Run TraceX:

```bash
./tracex ./myprogram
```

---

# Example Debugging Session

Start the debugger:

```text
./tracex ./myprogram
```

Set a breakpoint:

```text
break main
```

Start the target:

```text
run
```

Inspect all registers:

```text
register dump
```

Inspect the instruction pointer:

```text
register read rip
```

Execute one instruction:

```text
stepi
```

Step through source code:

```text
step
```

Continue execution:

```text
continue
```

Inspect executable information:

```text
header
sections
symbols
functions
```

---

# Project Structure

```text
TraceX/
│
├── include/
│   ├── debugger.h
│   ├── commands.h
│   ├── stepping.h
│   ├── breakpoints.h
│   ├── registers.h
│   ├── memory.h
│   └── symbols.h
│
├── src/
│   ├── main.c
│   ├── debugger.c
│   ├── commands.c
│   ├── stepping.c
│   ├── breakpoints.c
│   ├── registers.c
│   ├── memory.c
│   └── symbols.c
│
├── test/
│
├── Makefile
│
└── README.md
```

---

# Learning Objectives

The project is intended to provide practical experience with
low-level Linux systems programming.

Major concepts explored include:

- Linux processes
- Process tracing
- `ptrace()`
- Unix signals
- CPU registers
- Virtual memory
- x86-64 execution
- Software breakpoints
- ELF binaries
- DWARF debugging information
- PIE
- ASLR
- Source-level debugging
- Linux system interfaces
- C programming
- Build systems

---

# Future Development

Possible future extensions include:

- [ ] Variable inspection
- [ ] Conditional breakpoints
- [ ] Watchpoints
- [ ] Improved breakpoint management
- [ ] Dynamic shared-library support
- [ ] x86-64 disassembly
- [ ] Program-header analysis
- [ ] String extraction
- [ ] Relocation analysis
- [ ] PLT/GOT inspection
- [ ] Improved backtrace support
- [ ] Additional automated tests
- [ ] Improved command-line diagnostics

---

# Project Goals

TraceX is designed as an educational systems-programming project.

The main goal is to understand what happens underneath a conventional
debugger:

```text
Source Code
     │
     ▼
Compiler
     │
     ▼
ELF Executable
     │
     ▼
Linux Process
     │
     ▼
ptrace()
     │
     ▼
TraceX
     │
     ├── Registers
     ├── Memory
     ├── Breakpoints
     ├── Signals
     ├── ELF
     └── DWARF
```

The project provides a practical way to study the relationship between
compiled programs, operating-system process control, executable formats,
and CPU execution.

---

# Project Status

TraceX is an educational Linux debugger focused on low-level
systems-programming concepts.

It is intended for learning and experimentation with Linux debugging,
ELF binaries, DWARF information, process tracing, and x86-64 execution.

---

# Authors

**Jitendra & Anshu & Sonali**

TraceX is developed as a collaborative systems-programming project
for learning, experimentation, and further extension.

---

# License

This project is distributed under the MIT License.

```

:contentReference[oaicite:0]{index=0}
```
