# Unit 2: Operating System Structure
**Course:** CACS251 — Operating System | **Duration:** 2 Hours
**Year/Semester:** BCA II/IV | Pokhara University

---

## Syllabus Topics
Introduction, Layered System, Kernel, Types of Kernel (Monolithic/Macro Kernel and Micro / Exo-Kernel), Client-Server Model, Virtual Machines, Shell.

---

## 1. Introduction to OS Structure

An operating system is not a single monolithic program — it is structured internally in various ways to make it efficient, maintainable, and extensible.

**Why does OS structure matter?**
- Determines how easy it is to **debug, maintain, and update** the OS.
- Affects **performance** — how fast the OS responds to system calls.
- Affects **security and reliability** — a poorly structured OS can crash entirely if one component fails.

There are several approaches to structuring an OS:
1. Simple / Monolithic structure
2. Layered structure
3. Microkernel structure
4. Exokernel structure
5. Client-Server model
6. Virtual Machine model

> **Real-world analogy:** Think of OS structure like building architecture. A simple hut (monolithic) is easy to build but hard to expand. A modern skyscraper (layered) has separate floors for separate purposes. A modular building (microkernel) has plug-in sections you can add or remove.

---

## 2. Layered System

### Concept
The OS is divided into a number of **layers (levels)**, each built on top of the one below it. Each layer uses functions and services provided by the **layer below** and provides services to the **layer above**.

### Structure

```
┌─────────────────────────────┐
│  Layer N   — User Programs  │   ← Highest level
├─────────────────────────────┤
│  Layer N-1 — I/O Buffering  │
├─────────────────────────────┤
│     ...     (Middle Layers) │
├─────────────────────────────┤
│  Layer 1   — CPU Scheduling │
├─────────────────────────────┤
│  Layer 0   — Hardware       │   ← Lowest level
└─────────────────────────────┘
```

### Classic Example — THE System (Dijkstra, 1968)
THE (Technische Hogeschool Eindhoven) OS had 6 layers:

| Layer | Function |
|---|---|
| 5 | User Programs |
| 4 | Buffering for I/O devices |
| 3 | Operator-console driver |
| 2 | Memory management |
| 1 | CPU scheduling |
| 0 | Hardware |

### Advantages
- **Modularity** — Each layer can be designed, tested, and debugged independently.
- **Abstraction** — Lower layers hide hardware complexity from higher layers.
- **Easy maintenance** — Changes in one layer don't affect others (if interfaces are preserved).

### Disadvantages
- **Performance overhead** — A system call must pass through multiple layers.
- **Difficult to define layers** — Not always clear which layer a function belongs to.
- **Not used in pure form** — Modern OS use hybrid approaches.

> **Real-world example:** The **OSI model in networking** (7 layers) works similarly — each layer handles a specific task and communicates only with adjacent layers. Windows NT uses a partial layered approach.

---

## 3. Kernel

### Definition
The **kernel** is the **core component** of an operating system. It is the part of the OS that is loaded into memory first and stays in memory throughout the system's operation.

### What does the Kernel do?
- **Process Management** — creates, schedules, and terminates processes.
- **Memory Management** — allocates/deallocates RAM.
- **Device Management** — communicates with hardware via device drivers.
- **System Call Interface** — provides an interface for user programs to request OS services.
- **Interrupt Handling** — responds to hardware and software interrupts.

### Kernel vs. User Space

| | Kernel Space | User Space |
|---|---|---|
| Mode | Privileged (Ring 0) | Unprivileged (Ring 3) |
| Access | Full hardware access | Restricted — via system calls |
| Crash impact | Crashes entire system | Crashes only that process |
| Examples | OS kernel code, drivers | Chrome, Word, your programs |

> **Real-world example:** When Chrome wants to read a file, it doesn't touch the disk directly. It makes a **system call** — a request to the kernel — which safely reads the file and returns the data. This protects the hardware from buggy applications.

---

## 4. Types of Kernel

### 4.1 Monolithic Kernel (Macro Kernel)

**Definition:** All OS services (process management, memory management, file systems, device drivers, networking) run in **a single large program** in kernel space.

```
┌──────────────────────────────────────────────┐
│               Kernel Space                    │
│  ┌──────────┬──────────┬──────────┬────────┐ │
│  │  Process │  Memory  │   File   │ Device │ │
│  │  Mgmt    │  Mgmt    │  System  │Drivers │ │
│  └──────────┴──────────┴──────────┴────────┘ │
└──────────────────────────────────────────────┘
           ↑ System Call Interface ↑
┌──────────────────────────────────────────────┐
│               User Space                      │
│         Applications (Chrome, Word…)          │
└──────────────────────────────────────────────┘
```

**Characteristics:**
- All components communicate directly (no message passing overhead).
- Very **fast** — function calls within kernel space are inexpensive.
- A bug in any component (e.g., a driver) can **crash the entire system**.

**Examples:** Linux, Unix (traditional), MS-DOS

**Advantages:**
- High performance — direct communication between components.
- Well understood and mature.

**Disadvantages:**
- Large, complex codebase — difficult to maintain.
- Less secure — a faulty driver can crash the whole OS.
- Adding new features requires recompiling the kernel.

> **Real-world example:** **Linux** is a monolithic kernel. When you install a GPU driver on Linux, it runs in kernel space. If that driver has a bug, it can cause a kernel panic (Linux equivalent of BSOD). This is why driver quality matters so much in Linux.

---

### 4.2 Microkernel

**Definition:** Only the **most essential functions** run in kernel space. Everything else (device drivers, file systems, networking) runs in **user space** as separate processes.

```
┌──────────────────────────────────────────────────────┐
│                    User Space                         │
│  ┌──────────┐  ┌──────────┐  ┌────────┐  ┌────────┐ │
│  │  File    │  │  Device  │  │Network │  │  Other │ │
│  │  System  │  │  Driver  │  │  Stack │  │Services│ │
│  │ (server) │  │ (server) │  │(server)│  │        │ │
│  └──────────┘  └──────────┘  └────────┘  └────────┘ │
└──────────────────────────────────────────────────────┘
           ↑  Message Passing (IPC)  ↑
┌──────────────────────────────────────────────────────┐
│               Microkernel (Kernel Space)              │
│       IPC  |  Basic Memory Mgmt  |  CPU Scheduling   │
└──────────────────────────────────────────────────────┘
```

**Characteristics:**
- Kernel is very **small and simple** — only IPC, basic memory, and scheduling.
- Services communicate via **message passing (IPC)**.
- A crash in a service (e.g., file system) does **not** crash the kernel.

**Examples:** Mach, MINIX, QNX, L4, Hurd

**Advantages:**
- High **reliability and security** — fault isolation.
- **Easier to extend** — add new services without modifying the kernel.
- Suitable for **embedded and safety-critical systems**.

**Disadvantages:**
- **Slower** — message passing between user-space services adds overhead.
- More complex programming model.

> **Real-world example:** **QNX** is a microkernel OS used in car infotainment systems, medical devices, and industrial equipment. If the audio driver crashes, the car's navigation continues working — fault isolation in action. **macOS** and **iOS** use a modified Mach microkernel at their core (XNU kernel).

---

### 4.3 Exokernel

**Definition:** An extremely minimal kernel that provides **no abstractions at all** — it only multiplexes hardware resources among applications. Applications manage their own abstractions via **library operating systems (LibOS)**.

```
┌───────────────────────────────────────────┐
│          Application 1   Application 2    │
│        ┌──────────┐    ┌──────────┐       │
│        │  LibOS 1 │    │  LibOS 2 │       │  ← Custom OS libraries
│        └──────────┘    └──────────┘       │
└───────────────────────────────────────────┘
          ↑  Hardware Multiplexing  ↑
┌───────────────────────────────────────────┐
│           Exokernel (very tiny)           │
│   Only: Secure access & resource alloc   │
└───────────────────────────────────────────┘
```

**Characteristics:**
- Gives applications **direct (but safe) access** to hardware resources.
- Applications can **customize OS abstractions** for their specific needs.
- Maximum performance possible — no abstraction overhead.

**Examples:** MIT Exokernel (research), Nemesis OS

**Advantages:**
- Maximum performance for specialized applications.
- Applications are not forced to use one-size-fits-all abstractions.

**Disadvantages:**
- Very complex for application programmers.
- Mainly a **research concept** — not mainstream.

> **Real-world example:** A database like Oracle could use an Exokernel to manage disk I/O in its own optimized way, rather than going through the generic OS file system — potentially 10x faster for large databases.

---

### Kernel Types — Quick Comparison

| Feature | Monolithic | Microkernel | Exokernel |
|---|---|---|---|
| Kernel Size | Large | Small | Extremely small |
| Speed | Very Fast | Slower (IPC overhead) | Fastest |
| Reliability | Lower | Higher | Depends on LibOS |
| Security | Lower | Higher | Application-managed |
| Maintenance | Harder | Easier | Very complex |
| Examples | Linux, Unix | QNX, MINIX, macOS core | MIT Exokernel |

---

## 5. Client-Server Model

### Concept
The OS is structured so that the **kernel acts as a message-passing facility**, and OS services are provided by **server processes**. Client programs (user applications) request services from server processes.

```
┌─────────────────────────────────────────────┐
│               User Space                     │
│                                              │
│  ┌──────────┐        ┌──────────────────┐   │
│  │  Client  │─request→│  Server Process  │   │
│  │  (App)   │←response│  (File, Print,   │   │
│  └──────────┘        │   Network, etc.) │   │
│                       └──────────────────┘   │
└─────────────────────────────────────────────┘
              ↑ Message Passing ↑
┌─────────────────────────────────────────────┐
│          Kernel (Message Router)             │
└─────────────────────────────────────────────┘
```

### How It Works
1. A **client** (application) sends a **request message** to the kernel.
2. The kernel routes the message to the appropriate **server process**.
3. The server processes the request and sends a **response message** back.
4. The kernel delivers the response to the client.

### Advantages
- Services can run on **different machines** (distributed systems).
- Highly **modular and extensible**.
- A crashing server doesn't crash the entire OS.

### Disadvantages
- **Overhead** from message passing.
- More complex to implement.

> **Real-world example:** **Networked file systems** like NFS (Network File System) use the client-server model — your computer (client) requests files from a remote server. Windows file sharing (SMB protocol) works the same way. In distributed systems, **REST APIs** (web services) are essentially a client-server model at the network level.

---

## 6. Virtual Machines

### Definition
A **Virtual Machine (VM)** is a software emulation of a complete computer system. It allows multiple operating systems to run simultaneously on the **same physical hardware**, each thinking it has exclusive access to the hardware.

### Architecture

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Guest OS 1 │  │   Guest OS 2 │  │   Guest OS 3 │
│  (Windows)   │  │   (Ubuntu)   │  │   (Kali)     │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                  │
┌──────┴─────────────────┴──────────────────┴───────┐
│           Hypervisor / VMM (Virtual Machine        │
│               Monitor)                             │
└───────────────────────────┬───────────────────────┘
                             │
┌───────────────────────────┴───────────────────────┐
│                  Physical Hardware                  │
│              (CPU, RAM, Disk, Network)              │
└────────────────────────────────────────────────────┘
```

### Types of Hypervisors

**Type 1 — Bare-Metal Hypervisor (runs directly on hardware):**
- Faster, more efficient.
- Examples: VMware ESXi, Microsoft Hyper-V, Xen.
- Used in: Data centers, cloud servers (AWS, Azure, Google Cloud).

**Type 2 — Hosted Hypervisor (runs on top of a host OS):**
- Easier to use, slightly slower.
- Examples: VirtualBox, VMware Workstation, QEMU.
- Used in: Developer machines, testing environments.

### Key Concepts
- **Guest OS** — The OS running inside the VM.
- **Host OS** — The OS on the physical machine (Type 2 only).
- **VMM / Hypervisor** — Software that manages and runs VMs.
- **Snapshot** — A saved state of a VM that can be restored anytime.

### Advantages
- **Isolation** — VMs are isolated; a crash in one doesn't affect others.
- **Multiple OS** — Run Windows and Linux simultaneously on one machine.
- **Security testing** — Run malware in a VM safely.
- **Cloud computing** — The foundation of AWS, Azure, and Google Cloud.
- **Development & testing** — Test software on different OS versions.

### Disadvantages
- **Performance overhead** — Virtualization adds latency.
- **Resource intensive** — Each VM needs its own RAM, CPU share.
- **Licensing** — Running multiple OS copies may require multiple licenses.

> **Real-world examples:**
> - **AWS EC2** — Every cloud server you rent is a virtual machine running on Amazon's physical servers.
> - **VirtualBox / VMware Workstation** — Used by developers to test apps on different OS.
> - **Docker containers** — A lighter form of virtualization (OS-level, not full VM).
> - **Android Emulator** — Your phone emulator in Android Studio is a virtual machine.

---

## 7. Shell

### Definition
The **shell** is a **command-line interpreter** — a program that provides a user interface to the operating system. It reads commands entered by the user and executes them by making system calls to the kernel.

### Shell as an Interface

```
User → types commands → Shell → interprets → System Calls → Kernel → Hardware
```

The shell is **not part of the kernel** — it is a regular user-space program that uses system calls to interact with the OS.

### Types of Shells

**1. Command-Line Interface (CLI) Shells:**

| Shell | OS | Notes |
|---|---|---|
| Bash (Bourne Again Shell) | Linux/macOS | Most popular Linux shell |
| sh (Bourne Shell) | Unix/Linux | Original Unix shell |
| Zsh | Linux/macOS | Default on macOS since Catalina |
| PowerShell | Windows | Object-oriented, very powerful |
| CMD (Command Prompt) | Windows | Legacy Windows shell |
| Fish | Linux | Beginner-friendly, auto-suggest |

**2. Graphical Shell (GUI):**
- Windows Explorer (Windows desktop) is essentially a graphical shell.
- GNOME Shell, KDE Plasma (Linux desktop environments) are graphical shells.

### What Can a Shell Do?

| Feature | Example |
|---|---|
| Run programs | `python script.py` |
| File operations | `cp file1.txt file2.txt` |
| Directory navigation | `cd /home/user/documents` |
| Piping | `ls -l \| grep ".txt"` |
| Scripting (automation) | Write `.sh` scripts to automate tasks |
| Environment variables | `echo $PATH` |
| Process management | `kill 1234`, `ps aux` |
| I/O redirection | `ls > output.txt` |

### Shell Scripting
Shell scripting allows you to write programs using shell commands — useful for automating repetitive tasks.

```bash
#!/bin/bash
# Simple shell script example
echo "Hello, World!"
for i in 1 2 3 4 5
do
   echo "Count: $i"
done
```

> **Real-world examples:**
> - **DevOps engineers** use shell scripts to automate server deployments.
> - **System admins** use Bash scripts to back up databases every night.
> - When you open **Terminal on macOS** and type `ls`, you're using the Zsh shell.
> - **CI/CD pipelines** (GitHub Actions, Jenkins) run shell scripts to build and test code.

---

## 8. Summary

### OS Structure Approaches — At a Glance

| Approach | Key Idea | Used In |
|---|---|---|
| Simple/Monolithic | All services in one large program | Linux, Unix |
| Layered | OS divided into hierarchical layers | Windows NT (partial) |
| Microkernel | Only essentials in kernel; rest in user space | QNX, MINIX, macOS core |
| Exokernel | No abstractions; apps manage hardware | Research (MIT) |
| Client-Server | Kernel routes messages between clients & servers | Distributed systems |
| Virtual Machine | Multiple OS on one hardware via hypervisor | AWS, VirtualBox |

### Kernel Types — One-Liner Summary

| Kernel | One-liner |
|---|---|
| Monolithic | "Everything inside the kernel — fast but risky" |
| Microkernel | "Only essentials in kernel — safe but slower" |
| Exokernel | "No abstractions — maximum performance, maximum complexity" |

### Shell — One-liner
> Shell = The face of the OS that the user talks to. Kernel = The brain that actually does the work.

---

## 9. Exam Tips

Common exam question types for Unit 2:

1. **Define kernel. What are the functions of a kernel?**
2. **Differentiate between Monolithic Kernel and Microkernel** (table form — very common).
3. **What is a Virtual Machine? What are its advantages?** (mention AWS/cloud).
4. **Explain the layered structure of OS with a diagram.**
5. **What is a shell? Give examples of shells.**
6. **Explain the Client-Server model of OS structure.**
7. **What is an Exokernel? How is it different from a Microkernel?**

> **Key tip:** For comparison questions, always draw a table and give a real-world example for each type — examiners appreciate it.

---

## 10. References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014 ISBN: 9788126520510
- Andrew S. Tanenbaum, *Distributed Operating System*, Pearson
