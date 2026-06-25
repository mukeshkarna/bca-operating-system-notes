**Unit 1: Introduction to Operating System**:

---

# Unit 1: Introduction to Operating System
**Course:** CACS251 — Operating System | **Duration:** 2 Hours

---

## 1. History and Generation of Operating Systems

Operating systems have evolved over decades alongside hardware advancements.

### Generation 1 (1945–1955) — Vacuum Tubes, No OS
- Computers were massive machines using **vacuum tubes**.
- Programmers interacted directly with hardware using **plugboards** and **machine language**.
- No operating system existed — one job at a time, entirely manual.
- **Real-world example:** ENIAC (1945) — required physical rewiring to change programs.

### Generation 2 (1955–1965) — Transistors, Batch Systems
- **Transistors** replaced vacuum tubes — smaller and more reliable.
- **Batch processing** introduced: jobs were grouped (batched) on punch cards and fed to the machine sequentially.
- A human **operator** would collect jobs, run them, and return output.
- **Real-world example:** IBM 7094 used batch processing for payroll systems in large corporations.

### Generation 3 (1965–1980) — Integrated Circuits, Multiprogramming
- **Integrated Circuits (ICs)** made computers faster and cheaper.
- **Multiprogramming** introduced: multiple jobs kept in memory; CPU switches between them when one is doing I/O.
- **Time-sharing** systems emerged — multiple users share the CPU simultaneously.
- **Real-world example:** IBM System/360 family; UNIX was developed in this era (1969) at Bell Labs.

### Generation 4 (1980–Present) — Personal Computers, Modern OS
- **LSI (Large Scale Integration)** chips made personal computers affordable.
- GUI-based operating systems emerged.
- Networking, multitasking, and security became central.
- **Real-world examples:** MS-DOS (1981) → Windows (1985 onwards), macOS, Linux, Android, iOS.

---

## 2. Introduction to Operating System

An **Operating System (OS)** is system software that acts as an **intermediary** between computer hardware and the user/application programs.

> **Analogy:** Think of the OS like the **management staff of a hotel**. Guests (users/apps) make requests. The staff (OS) manages all the rooms (memory), services (CPU), and facilities (devices) — guests never interact with the building's wiring directly.

### Key Points
- It is the **first software loaded** when a computer starts (bootstrapping).
- Without an OS, users would have to program hardware directly in binary.
- **Examples:** Windows 11, Ubuntu Linux, macOS Ventura, Android 14, iOS 17.

---

## 3. Objectives of an Operating System

The OS serves two primary roles:

### A. Resource Manager
The OS manages all hardware resources of the computer system.

| Resource | What OS manages |
|---|---|
| CPU | Decides which process runs and for how long |
| Memory (RAM) | Allocates and deallocates memory to processes |
| Storage (Disk) | Manages file systems and disk space |
| I/O Devices | Controls printers, keyboards, mice, network cards |

> **Real-world example:** When you open Chrome, Spotify, and a Word document simultaneously, the OS allocates CPU time and RAM to each so they all run without crashing each other.

### B. Extended Machine (Virtual Machine)
The OS **hides the complexity** of hardware and provides a clean, simple interface to users and programs.

- Programmers don't need to know how disk sectors are physically addressed — they just call `open("file.txt")`.
- The OS provides **abstractions** like files, processes, and sockets.

> **Real-world example:** A photographer using Photoshop just clicks "Save." They don't write raw bytes to disk sectors — the OS handles all of that invisibly.

---

## 4. Types of Operating Systems

### 1. Batch Operating System
- Jobs are collected and processed in batches without user interaction.
- **Advantage:** Efficient use of CPU.
- **Disadvantage:** No real-time interaction; long turnaround time.
- **Real-world example:** Bank statement generation overnight, payroll processing.

### 2. Time-Sharing (Multitasking) Operating System
- Multiple users/processes share CPU time in small intervals called **time slices (quantum)**.
- Each user feels they have a dedicated computer.
- **Real-world example:** University mainframes where many students run programs simultaneously; modern Linux servers.

### 3. Multiprocessing Operating System
- Uses **two or more CPUs** within a single system.
- Increases throughput and reliability.
- **Real-world example:** Modern laptops with quad-core or octa-core processors; servers running databases.

### 4. Real-Time Operating System (RTOS)
- Designed for systems where tasks **must be completed within strict deadlines**.
- Two types:
  - **Hard RTOS:** Missing a deadline is catastrophic (e.g., airbag systems).
  - **Soft RTOS:** Missing a deadline degrades performance but isn't fatal (e.g., video streaming).
- **Real-world examples:**
  - **Hard:** Anti-lock braking system (ABS) in cars, pacemakers, missile guidance systems.
  - **Soft:** VoIP calls, online video conferencing (Zoom).

### 5. Distributed Operating System
- Manages a group of independent computers and makes them appear as a **single coherent system**.
- **Real-world example:** Google's data centers — thousands of machines work together; cloud platforms like AWS.

### 6. Network Operating System
- Provides features for **managing network resources** — file sharing, printer sharing, user accounts across a network.
- **Real-world example:** Windows Server, Novell NetWare.

### 7. Mobile Operating System
- Designed for smartphones and tablets with touch interfaces, battery efficiency, and app management.
- **Real-world examples:** Android (Linux-based), iOS (Unix-based).

---

## 5. Functions of an Operating System

### 1. Process Management
- Creating, scheduling, and terminating processes.
- Handling process synchronization and communication.
- > **Example:** When you open a browser, the OS creates a new process and allocates CPU time to it.

### 2. Memory Management
- Allocating memory to processes when needed and freeing it when done.
- Preventing one process from accessing another's memory.
- > **Example:** If Chrome uses too much RAM, Windows may move some data to the **page file** (virtual memory on disk).

### 3. File System Management
- Organizing data into files and directories.
- Providing operations: create, read, write, delete, rename.
- > **Example:** When you save a Word document, the OS writes data to disk in organized sectors and updates the directory table.

### 4. I/O Device Management
- Managing communication between the CPU and peripheral devices through **device drivers**.
- > **Example:** When you plug in a USB printer, the OS detects it, loads the driver, and makes it available to applications.

### 5. Security and Protection
- Controlling access to resources — who can read/write which files.
- User authentication, permissions.
- > **Example:** On Linux, file permissions (read/write/execute for owner, group, others) prevent unauthorized access.

### 6. Error Detection and Handling
- Continuously monitors the system for hardware/software errors and takes corrective actions.
- > **Example:** A "Blue Screen of Death" (BSOD) on Windows is the OS detecting a critical unrecoverable error and halting to prevent data corruption.

### 7. User Interface
- Provides **CLI (Command Line Interface)** or **GUI (Graphical User Interface)** for user interaction.
- > **Example:** Windows Explorer (GUI) vs. Command Prompt / PowerShell (CLI) vs. Linux Terminal (Bash).

---

## 6. Summary Table

| Topic | Key Point |
|---|---|
| History | 4 generations: Vacuum tubes → Transistors → ICs → VLSI/Personal Computers |
| Definition | Software bridge between hardware and users/applications |
| As Resource Manager | Manages CPU, Memory, Disk, I/O devices |
| As Extended Machine | Hides hardware complexity; provides abstractions |
| Types | Batch, Time-sharing, Multiprocessing, RTOS, Distributed, Network, Mobile |
| Functions | Process, Memory, File, I/O, Security, Error Handling, User Interface |

---