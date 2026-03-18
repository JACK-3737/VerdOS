# Contributing to ForgeOS

First off — thank you. ForgeOS is a large, long-term project and every contribution matters, whether it's fixing a typo in the docs, squashing a kernel bug, or designing an entire subsystem. This document will walk you through everything you need to know to contribute effectively.

If you have questions not answered here, open a [GitHub Discussion](../../discussions) and we'll help you out.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [What Can I Work On?](#what-can-i-work-on)
- [Reporting Bugs](#reporting-bugs)
- [Suggesting Features](#suggesting-features)
- [Development Setup](#development-setup)
- [Branching & Workflow](#branching--workflow)
- [Commit Message Format](#commit-message-format)
- [Coding Style](#coding-style)
- [Testing Requirements](#testing-requirements)
- [Pull Request Process](#pull-request-process)
- [Architecture Decisions](#architecture-decisions)
- [Getting Help](#getting-help)

---

## Code of Conduct

This project follows a simple standard: **be respectful, be constructive, be patient.**

Specifically:
- Critique code and ideas, never people
- Assume good intent in written communication — tone is hard to read in text
- Welcome contributors of all experience levels; everyone was a beginner once
- No harassment, discrimination, or personal attacks of any kind

Violations can be reported by opening a private issue or emailing the maintainers directly. Repeated or serious violations will result in removal from the project.

---

## What Can I Work On?

### Good First Issues
If you're new to the project or to OS development, look for issues tagged [`good first issue`](../../issues?q=label%3A%22good+first+issue%22). These are intentionally scoped to be approachable without needing deep knowledge of the whole system.

### Current Milestone
Check the [GitHub Project board](../../projects) to see what's actively being worked on and what's up for grabs in the current milestone. Issues tagged `help wanted` are explicitly open for external contributors.

### Documentation
Documentation contributions are always welcome and are a great way to get familiar with the codebase. This includes:
- Fixing typos or unclear wording
- Writing or improving subsystem documentation in `/docs`
- Adding inline code comments to complex kernel sections
- Writing Architecture Decision Records (see [Architecture Decisions](#architecture-decisions))

### Things to Avoid
- Don't start work on a large feature without opening a discussion issue first — we may already be working on it, or have design constraints you're not aware of
- Don't refactor code unrelated to your PR's stated purpose
- Don't submit AI-generated code that hasn't been read, understood, and tested by you

---

## Reporting Bugs

Before filing a bug, please:
1. Check the [existing issues](../../issues) to see if it's already been reported
2. Make sure you're running the latest version of the code on `main`
3. Reproduce the bug in QEMU — do not test on bare metal until the project is much more mature

When you open a bug report, include:

```
**Description**
A clear, one-paragraph description of what went wrong.

**Steps to Reproduce**
1. Build with `make all`
2. Run with `make run`
3. ...

**Expected Behavior**
What you expected to happen.

**Actual Behavior**
What actually happened. Include the full QEMU output or a screenshot.

**Environment**
- Host OS: (e.g. Ubuntu 24.04)
- GCC version: (run `gcc --version`)
- QEMU version: (run `qemu-system-x86_64 --version`)
- Commit hash: (run `git rev-parse HEAD`)
```

If the bug causes a kernel panic, include the full panic output — register dump, stack trace, and the last lines of output before the crash.

---

## Suggesting Features

Feature suggestions are welcome, but please keep in mind that this project has a strict milestone structure. A suggestion for a Bluetooth stack isn't useful feedback when we're still working on basic memory paging.

To suggest a feature:
1. Open a [GitHub Discussion](../../discussions) in the **Ideas** category
2. Describe the feature and *why* it fits the project's goals
3. If the discussion gets traction, a maintainer will convert it to a tracked issue

Do not open a Pull Request implementing a major new feature without prior discussion — it may not align with current priorities and could be closed without merging.

---

## Development Setup

### Required Tools

```bash
# Build tools
sudo apt install gcc nasm binutils make

# QEMU emulator (required — do NOT test on real hardware)
sudo apt install qemu-system-x86

# GRUB for bootable ISO generation
sudo apt install grub-pc-bin grub-efi-amd64-bin xorriso

# Debugging
sudo apt install gdb
```

### Recommended: Cross-Compiler

To avoid conflicts with your host system's GCC, we strongly recommend building a cross-compiler targeting `x86_64-elf`. See [`docs/TOOLCHAIN.md`](./docs/TOOLCHAIN.md) for the full setup guide. This is required for kernel work and optional for documentation or tooling contributions.

### Clone and Build

```bash
git clone https://github.com/your-username/forgeOS.git
cd forgeOS
make all
make run
```

If QEMU opens and you see the bootloader output, your environment is working correctly.

### Debugging

```bash
make debug
```

This launches QEMU in debug mode (paused at startup) and attaches GDB automatically. You can set breakpoints, step through kernel code, and inspect registers. See [`docs/DEBUGGING.md`](./docs/DEBUGGING.md) for a full debugging guide.

---

## Branching & Workflow

We use a simple feature-branch workflow off `main`.

```
main                  ← always builds and boots cleanly
└── feature/my-thing  ← your work lives here
└── fix/crash-on-init
└── docs/update-readme
```

### Branch Naming

| Type | Format | Example |
|------|--------|---------|
| New feature | `feature/short-description` | `feature/page-allocator` |
| Bug fix | `fix/short-description` | `fix/scheduler-overflow` |
| Documentation | `docs/short-description` | `docs/memory-map-spec` |
| Refactor | `refactor/short-description` | `refactor/irq-handler` |

### Rules

- **Never commit directly to `main`** — all changes come in through Pull Requests
- Keep branches focused on a single concern — one feature or fix per branch
- Rebase your branch on `main` before opening a PR to keep history clean

```bash
git fetch origin
git rebase origin/main
```

---

## Commit Message Format

We use a structured commit format inspired by Conventional Commits. Every commit message should read like a short, clear sentence about what changed and where.

```
subsystem/component: short imperative description

Optional longer explanation of why this change was made,
what tradeoffs were considered, or what side effects to
be aware of. Wrap at 72 characters.

Fixes #123
```

### Subsystem Prefixes

| Prefix | Use for |
|--------|---------|
| `kernel/mm` | Memory management |
| `kernel/sched` | Process scheduler |
| `kernel/syscall` | System call interface |
| `kernel/interrupt` | Interrupt handling |
| `hal/drivers` | Device drivers |
| `hal/cpu` | CPU initialization |
| `fs/vfs` | Virtual filesystem |
| `fs/fat32` | FAT32 driver |
| `net/tcp` | TCP/IP stack |
| `runtime` | Userspace standard library |
| `gui` | Window manager / compositor |
| `shell` | Command-line shell |
| `build` | Makefile, toolchain, CI |
| `docs` | Documentation only |

### Good vs Bad Examples

```bash
# Good
kernel/mm: implement buddy allocator for physical page management
fix/kernel/sched: prevent null dereference on idle thread exit
docs: add ADR-004 for filesystem layer design

# Bad
fix stuff
WIP
updated kernel
kernel changes for the scheduler thing
```

---

## Coding Style

Consistent style in kernel code is not optional — inconsistency makes low-level code significantly harder to audit and debug.

### C Code

- **Indentation**: 4 spaces. No tabs.
- **Line length**: 80 characters max. Break long lines at logical points.
- **Braces**: Opening brace on the same line for functions and control flow.
- **Naming**:
  - Functions and variables: `snake_case`
  - Constants and macros: `UPPER_SNAKE_CASE`
  - Structs and typedefs: `snake_case_t`
- **Comments**: Write comments that explain *why*, not *what*. The code says what; the comment explains the reasoning.
- **Headers**: Every `.h` file must have an include guard.

```c
/* Good: explains the why */
/* We use a bitmap allocator here rather than a free list because
   physical memory allocation happens in interrupt context where
   we cannot afford dynamic resizing. */
uint64_t pmm_alloc_frame(void) {
    for (size_t i = 0; i < bitmap_size; i++) {
        if (bitmap[i] != 0xFF) {
            /* found a free frame */
        }
    }
}

/* Bad: explains the what (the code already does that) */
/* loop through bitmap */
for (size_t i = 0; i < bitmap_size; i++) {
```

### Assembly

- Use **Intel syntax** (NASM)
- Comment every non-obvious instruction
- Keep assembly files focused — one logical unit per file
- Label names in `snake_case`

### General Rules

- No magic numbers — use named constants
- No unused variables or includes — keep it clean
- All memory allocations must have a corresponding free path
- Never silence a compiler warning with a cast — fix the underlying issue

---

## Testing Requirements

Kernel code that crashes takes the whole system with it. Testing is not optional.

### Before Every Pull Request

- [ ] The project builds with `make all` without warnings
- [ ] The OS boots to the expected state in QEMU with `make run`
- [ ] Your specific change has been manually exercised in QEMU
- [ ] You have not introduced any regressions in `make test`

### Writing Tests

For any kernel subsystem logic that can be tested in isolation (allocators, schedulers, parsers, data structures), write a unit test in `/kernel/tests/`. Tests run in a hosted userspace environment — they do not require booting.

```bash
make test              # run all unit tests
make test-mm           # run memory management tests only
make test-fs           # run filesystem tests only
```

### QEMU is Mandatory

Do **not** test on real hardware during development. QEMU provides:
- Safe crash recovery (just restart the VM)
- GDB integration for debugging panics
- Reproducible hardware state
- Serial output logging

If your change requires real hardware testing for a specific driver, note that explicitly in your PR.

---

## Pull Request Process

### Opening a PR

1. Ensure your branch is rebased on the latest `main`
2. Fill out the PR template completely — don't delete sections
3. Link to the issue your PR addresses with `Fixes #123` in the description
4. Make sure all CI checks pass before requesting review

### PR Template

```markdown
## What does this PR do?
A clear description of the change and why it's needed.

## How was it tested?
Describe how you tested this in QEMU. Include relevant output.

## Checklist
- [ ] Builds without warnings (`make all`)
- [ ] Boots in QEMU (`make run`)
- [ ] Tests pass (`make test`)
- [ ] Code follows the style guide
- [ ] Documentation updated if needed

## Related Issues
Fixes #
```

### Review Process

- A maintainer will review within **7 days** — if you haven't heard back, leave a comment to bump it
- Address all review comments with either a code change or a written explanation of why you disagree
- Once approved, a maintainer will merge — contributors do not merge their own PRs
- We use **squash merges** to keep the `main` history clean

### What Gets a PR Rejected

- Code that doesn't build
- Code that boots but panics in QEMU
- No testing evidence provided
- Scope creep — the PR does significantly more than it says
- Doesn't follow the coding style guide
- AI-generated code submitted without review or understanding

---

## Architecture Decisions

Significant design choices are documented as **Architecture Decision Records (ADRs)** in `/docs/adr/`. An ADR is a short document that captures:

- The context and problem being solved
- The options considered
- The decision made and why
- The consequences and tradeoffs

If your contribution introduces a significant architectural choice — a new kernel interface, a data structure that affects multiple subsystems, a change to the build system — you should write an ADR alongside your code.

Use [`docs/adr/ADR-000-template.md`](./docs/adr/ADR-000-template.md) as your starting point.

---

## Getting Help

| Channel | Use for |
|---------|---------|
| [GitHub Issues](../../issues) | Bug reports and tracked feature requests |
| [GitHub Discussions](../../discussions) | Questions, ideas, design conversations |
| [`docs/`](./docs/) | Architecture specs and guides |
| [OSDev Wiki](https://wiki.osdev.org) | Low-level x86 reference |

When asking for help, include your host OS, toolchain versions, the exact command you ran, and the full output. The more context you give, the faster someone can help.

---

*Thank you for taking the time to read this. Good contributions start with understanding the project — and you're already doing that.*
