<div align="center">

<br/>

```
____   ____               .___________    _________
\   \ /   /___________  __| _/\_____  \  /   _____/
 \   Y   // __ \_  __ \/ __ |  /   |   \ \_____  \ 
  \     /\  ___/|  | \/ /_/ | /    |    \/        \
   \___/  \___  >__|  \____ | \_______  /_______  /
              \/           \/         \/        \/ 
```

### *A modern operating system built from scratch, powered by AI.*

<br/>

![Status](https://img.shields.io/badge/status-pre--alpha-orange?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)
![Language](https://img.shields.io/badge/language-C%20%2F%20Assembly-lightgrey?style=flat-square)
![Platform](https://img.shields.io/badge/platform-x86__64-blueviolet?style=flat-square)
![Contributions](https://img.shields.io/badge/contributions-welcome-brightgreen?style=flat-square)

<br/>

[Overview](#-overview) · [Architecture](#-architecture) · [Roadmap](#-roadmap) · [Getting Started](#-getting-started) · [Contributing](#-contributing) · [FAQ](#-faq)

<br/>

</div>

---

## 🧠 Overview

**VerdOS** is an open-source, general-purpose operating system being built from scratch with the goal of achieving feature parity with modern desktop operating systems like Windows and macOS. The project is unique in that it leverages AI-assisted development — using large language models to help design subsystems, generate kernel modules, review code, and document architecture decisions.

This is not a toy or a hobby kernel. The long-term vision is a fully usable operating system with:

- A preemptive, multitasking kernel
- A modern, composited graphical user interface
- Full networking and storage support
- A rich application ecosystem
- First-class developer tooling

Whether you're here to learn, contribute, or follow along — welcome.

---

## 🏗️ Architecture

The OS is organized into six horizontal layers, each building on the one below it.

```
┌──────────────────────────────────────────────────────────────┐
│                        Applications                          │  L6
│         Browser · File Manager · Shell · Utilities           │
├──────────────────────────────────────────────────────────────┤
│              Window Manager / GUI   │   CLI Shell            │  L5
│         Compositor, widgets, themes │ Command interpreter    │
├──────────────────────────────────────────────────────────────┤
│                   User-Space Runtime                         │  L4
│        Standard library · App APIs · Dynamic linker          │
├────────────────┬───────────────┬────────────────┬────────────┤
│    Security    │   IPC / RPC   │    Storage     │ Networking │  L3
│  Auth, ACLs    │ Pipes, signals│  VFS, FS layer │  TCP/IP    │
├────────────────┴───────────────┴────────────────┴────────────┤
│                          Kernel                              │  L2
│   Memory · Scheduler · Syscalls · Interrupts · Drivers       │
├──────────────────────────┬───────────────────────────────────┤
│        Bootloader        │   Hardware Abstraction Layer      │  L1
│     UEFI/BIOS loader     │     I/O, IRQs, device drivers     │
├──────────────────────────┴───────────────────────────────────┤
│              Hardware (CPU · RAM · Storage · GPU)            │  L0
└──────────────────────────────────────────────────────────────┘
```

### Repository Structure

```
VerdOS/
├── bootloader/          # Stage-1 & Stage-2 UEFI/BIOS bootloaders
├── kernel/              # Core kernel: scheduler, memory, syscalls, IPC
│   ├── mm/              #   Memory management (paging, heap, VMM)
│   ├── sched/           #   Process & thread scheduler
│   ├── syscall/         #   System call interface
│   └── interrupt/       #   IDT, IRQ handlers
├── hal/                 # Hardware Abstraction Layer & device drivers
│   ├── cpu/             #   CPU detection, CPUID, GDT
│   ├── pci/             #   PCI bus enumeration
│   └── drivers/         #   NIC, storage, display, input drivers
├── fs/                  # Virtual Filesystem + FS implementations
│   ├── vfs/             #   VFS abstraction layer
│   ├── fat32/           #   FAT32 driver
│   └── ext2/            #   ext2 driver (planned)
├── net/                 # TCP/IP network stack
│   ├── ethernet/
│   ├── ipv4/
│   └── tcp/
├── runtime/             # Userspace standard library & APIs
├── gui/                 # Window compositor & widget toolkit
├── shell/               # Command-line interpreter
├── apps/                # Bundled system applications
├── tools/               # Build toolchain, QEMU configs, scripts
└── docs/                # ADRs, specs, design documents
    ├── adr/             #   Architecture Decision Records
    └── specs/           #   Subsystem specifications
```

---

## 🗺️ Roadmap

The project is divided into milestones. Each milestone produces something you can boot and interact with.

| Milestone | Name | Status | Description |
|:---------:|------|:------:|-------------|
| **M0** | Bootable Stub | 🔵 In Progress | Boots in QEMU, prints to screen, halts cleanly |
| **M1** | Minimal Kernel | ⬜ Planned | Memory paging, round-robin scheduler, basic syscalls |
| **M2** | Storage & Filesystem | ⬜ Planned | Read/write disk image, FAT32 support |
| **M3** | Networking | ⬜ Planned | Ping works, basic TCP socket API |
| **M4** | Userspace & Shell | ⬜ Planned | Load ELF binaries, interactive shell |
| **M5** | GUI Compositor | ⬜ Planned | Window manager renders overlapping windows |
| **M6** | Daily Driver Baseline | ⬜ Planned | Browser, file manager, settings app functional |

> Detailed tickets for the current milestone live in the [GitHub Project board](../../projects).

---

## 🚀 Getting Started

### Prerequisites

You'll need the following tools installed:

```bash
# Compiler & assembler
sudo apt install gcc nasm binutils

# QEMU for emulation (do NOT develop on bare metal yet)
sudo apt install qemu-system-x86

# GRUB for bootable ISO creation
sudo apt install grub-pc-bin grub-efi-amd64-bin xorriso

# (Optional) Cross-compiler — strongly recommended
# See docs/TOOLCHAIN.md for setup instructions
```

### Build & Run

```bash
# Clone the repo
git clone https://github.com/JACK-3737/VerdOS.git
cd VerdOS

# Build the project
make all

# Boot in QEMU
make run
```

You should see the bootloader output in the QEMU window. If it prints and halts, everything is working.

### Running Tests

```bash
make test        # Unit tests for kernel subsystems
make test-net    # Network stack tests (requires tap interface)
```

---

## 🤖 AI-Assisted Development

This project uses AI (primarily Claude by Anthropic) as a development accelerator. AI is used for:

- **Architecture design** — Brainstorming subsystem structure and tradeoffs
- **Boilerplate generation** — Kernel module skeletons, data structures, build scripts
- **Code review** — Catching bugs in memory management, pointer arithmetic, assembly
- **Documentation** — Drafting ADRs and subsystem specs
- **Learning** — Explaining low-level concepts (page tables, IDTs, ELF format) before implementation

All AI-generated code is reviewed, tested, and owned by the project contributors. AI output is a starting point, not a final answer.

---

## 🛠️ Tech Stack

| Component | Technology | Notes |
|-----------|-----------|-------|
| Kernel language | C (C17) | With inline Assembly where required |
| Bootloader | x86-64 Assembly + C | Targets UEFI via GNU-EFI |
| Assembler | NASM | Intel syntax |
| Build system | GNU Make | May migrate to CMake |
| Emulator | QEMU | x86_64 target |
| Debugging | GDB + QEMU stub | `make debug` attaches GDB |
| CI | GitHub Actions | Builds and boots on every push |

---

## 🤝 Contributing

Contributions are very welcome — this is a big project and there's plenty to do at every skill level.

### Before You Start

1. Read [`CONTRIBUTING.md`](./CONTRIBUTING.md) for code style and workflow expectations
2. Check the [GitHub Project board](../../projects) for open issues
3. For large changes, open a discussion issue first so we can align on design

### Good First Issues

Look for issues tagged [`good first issue`](../../issues?q=label%3A%22good+first+issue%22) — these are well-scoped tasks that don't require deep knowledge of the whole system.

### Development Workflow

```bash
# 1. Fork and clone
git clone https://github.com/JACK-3737/VerdOS.git

# 2. Create a feature branch
git checkout -b feature/my-subsystem

# 3. Make changes, commit with clear messages
git commit -m "kernel/mm: implement physical page allocator"

# 4. Push and open a Pull Request
git push origin feature/my-subsystem
```

Commit messages follow the format: `subsystem/component: short description`.

---

## 📚 Learning Resources

New to OS development? Start here:

- **[OSDev Wiki](https://wiki.osdev.org)** — The definitive reference for x86 OS development
- **[Writing a Simple OS from Scratch](https://www.cs.bham.ac.uk/~exr/lectures/opsys/10_11/lectures/os-dev.pdf)** — Nick Blundell's free PDF guide
- **[The Little Book About OS Development](https://littleosbook.github.io)** — Practical, modern walkthrough
- **[Intel® 64 and IA-32 Architectures SDM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)** — The authoritative hardware reference
- **[UEFI Specification](https://uefi.org/specifications)** — For bootloader work

---

## ❓ FAQ

**Q: Is this a Linux fork?**
No. This is written from scratch. We may borrow *ideas* from Linux but not code.

**Q: Can I actually use this as my daily OS?**
Not for a very long time. The project is pre-alpha. Run everything in QEMU.

**Q: What architecture is targeted?**
x86-64 (AMD64) for now. ARM support is on the long-term roadmap.

**Q: Why not just use Linux?**
This project exists for education, experimentation, and the challenge. Building an OS from scratch teaches things that using one never can.

**Q: How is AI being used — isn't that cheating?**
We treat AI the same way developers treat Stack Overflow or documentation: as a tool to help you understand and build faster. Every line of code is reviewed and understood by a human before it lands.

---

## 📄 License

This project is licensed under the **MIT License** — see [`LICENSE`](./LICENSE) for details.

---

<div align="center">

Built with curiosity, coffee, and AI. ☕

*If you find this project interesting, please consider giving it a ⭐ — it helps others find it.*

</div>
