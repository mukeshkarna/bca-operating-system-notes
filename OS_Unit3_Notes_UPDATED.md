# Unit 3: Process Management
**Course:** CACS251 — Operating System | **Duration:** 15 Hours
**Year/Semester:** BCA II/IV | Pokhara University

---

## Syllabus Overview

| Sub-Topic | Hours |
|---|---|
| Process Concepts | 3 Hrs. |
| Threads | 1 Hr. |
| Inter-Process Communication & Synchronization | 6 Hrs. |
| Process Scheduling | 5 Hrs. |
| **Total** | **15 Hrs.** |

---

# PART A — PROCESS CONCEPTS (3 Hours)

## 1. Introduction to Process & Process Management

**Process Management** is the OS module that handles creation, scheduling, synchronization, and termination of processes. It is responsible for:
- Allocating CPU time to processes.
- Handling inter-process communication (IPC).
- Managing process synchronization to prevent race conditions.
- Tracking all processes using the Process Control Block.

A **process** is a **program in execution** — an active entity that has been loaded into memory and is being executed by the CPU.

---

## 2. Program vs Process

| Program | Process |
|---|---|
| Passive — stored on disk | Active — running in memory |
| A static set of instructions | Dynamic — has state, resources, PID |
| One copy on disk | Multiple processes from one program |
| No resource allocation | Has allocated CPU, memory, files |
| Exists permanently | Exists only during execution |
| Example: `chrome.exe` file on disk | Example: 3 Chrome windows = 3 processes |

> **Real-world example:** MS Word document `.exe` on your disk = program. When you double-click it = process. Open 3 Word documents = 3 separate processes, each with their own memory space.

---

## 3. Process Model

A process in memory is divided into four sections:

```
┌─────────────────────────┐  High Address
│         Stack           │  ← Function calls, local variables, return addresses
├─────────────────────────┤
│          ↓              │  Stack grows downward
│                         │
│          ↑              │  Heap grows upward
├─────────────────────────┤
│          Heap           │  ← Dynamic memory (malloc, new)
├─────────────────────────┤
│    Data (BSS/Data)      │  ← Global & static variables
├─────────────────────────┤
│         Text            │  ← Program code (instructions, read-only)
└─────────────────────────┘  Low Address
```

- **Text (Code) Section** — the actual program instructions. Read-only.
- **Data Section** — global and static variables initialized at start.
- **Heap** — dynamically allocated memory at runtime (grows upward).
- **Stack** — function call frames, local variables, return addresses (grows downward).

---

## 4. Process Life Cycle

The **process life cycle** describes all the stages a process goes through from creation to termination.

```
          ┌─────────────────────────────────────────────────────┐
          │                  PROCESS LIFE CYCLE                  │
          │                                                       │
          ▼                                                       │
       [Created]                                                  │
          │                                                       │
          ▼                                                       │
       [Ready] ──────── Scheduler dispatches ────────► [Running] │
          ▲                                                │      │
          │                                                │      │
          │ I/O complete / event done                      │      │
          │                                                ▼      │
       [Waiting] ◄──── I/O request / event wait ──── [Running]   │
                                                          │       │
                                                          │ exit()│
                                                          ▼       │
                                                    [Terminated] ─┘
```

**Stages of Process Life Cycle:**

| Stage | Description |
|---|---|
| **New/Created** | Process is being created — memory allocated, PCB initialized |
| **Ready** | Process is waiting in ready queue for CPU |
| **Running** | CPU is executing process instructions |
| **Waiting/Blocked** | Process waiting for I/O or event completion |
| **Terminated** | Process has finished — resources released, PCB removed |
| **Suspended Ready** | Ready process swapped out of memory to disk |
| **Suspended Waiting** | Waiting process swapped out of memory to disk |

**Key Transitions:**
- **New → Ready:** OS admits the process (allocates memory, PID, initializes PCB).
- **Ready → Running:** CPU scheduler dispatches the process.
- **Running → Waiting:** Process requests I/O (e.g., file read).
- **Waiting → Ready:** I/O completes; interrupt signals OS.
- **Running → Ready:** Preemption — time quantum expired.
- **Running → Terminated:** Process calls `exit()` or is killed.

> **Real-world example:** When you open a PDF in a viewer, the OS creates the process (New), puts it in the ready queue (Ready), gives it CPU (Running), it reads from disk (Waiting), returns (Ready), renders (Running), and when you close it (Terminated).

---

## 5. Process Control Block (PCB)

The **PCB** (also called Task Control Block) is a data structure maintained by the OS kernel for **every process**. It is the "identity card" of a process.

### Role of PCB
- Stores all information about a process needed by the OS.
- Enables **context switching** — saving one process's state and loading another's.
- Used by the scheduler to manage process queues.
- Enables the OS to resume a process from exactly where it stopped.

### Components of PCB

```
┌─────────────────────────────────────┐
│         Process Control Block        │
├─────────────────────────────────────┤
│  Process ID (PID)                   │  Unique identifier, e.g., PID=1234
│  Process State                      │  Ready / Running / Waiting / etc.
│  Program Counter (PC)               │  Address of next instruction to execute
│  CPU Registers                      │  AX, BX, SP, BP, PC — saved on switch
│  CPU Scheduling Information         │  Priority, pointers to scheduling queues
│  Memory Management Information      │  Page tables, segment tables, base/limit
│  Accounting Information             │  CPU time used, clock time, job number
│  I/O Status Information             │  List of open files, I/O devices assigned
│  Parent PID                         │  PID of parent process
│  List of Open Files                 │  File descriptors
└─────────────────────────────────────┘
```

> **Real-world example:** When you Alt+Tab between Chrome and Word, the OS performs a context switch — saves Chrome's PCB (all registers, PC, state), loads Word's PCB. This happens hundreds of times per second, making it appear both run simultaneously.

---

## 6. Operations on Processes

### 6.1 Process Creation
A process (parent) creates another process (child) via system calls.
- **Unix/Linux:** `fork()` — creates an exact copy of parent.
- **Windows:** `CreateProcess()`.

After `fork()`, the child can call `exec()` to load a new program.

**Process Tree:**
```
    init (PID 1)
    ├── sshd (login daemon)
    │   └── bash (your terminal)
    │       └── vim (editor you opened)
    └── httpd (web server)
        ├── worker1
        └── worker2
```

**Resource sharing options after creation:**
1. Parent and child share all resources.
2. Child gets a subset of parent's resources.
3. No sharing — independent resources.

### 6.2 Process Preemption
**Preemption** is the act of **forcibly removing the CPU from a currently running process** and giving it to another process.

- Triggered by: timer interrupt (time quantum expiry), higher-priority process arriving.
- The preempted process's state is saved in its PCB.
- The process returns to the Ready queue.
- Used in: Round Robin, SRTF, Priority (preemptive) scheduling.

**Non-preemptive:** Process runs until it voluntarily yields or finishes (FCFS, SJF).

> **Real-world example:** In Android, when you receive a phone call while playing a game, the OS preempts the game process and gives CPU priority to the phone call handler — a high-priority process preempts a lower-priority one.

### 6.3 Process Termination
- Normal exit: `exit()` system call.
- Error exit: process detects an error and calls `exit()`.
- Fatal error: OS kills the process (e.g., division by zero, segfault).
- Killed by another process: `kill(pid, signal)` in Unix.

**Zombie Process:** Terminated process whose PCB still exists (parent hasn't called `wait()`).
**Orphan Process:** Process whose parent has terminated without calling `wait()`.

---

## 7. Process Hierarchies

On Unix/Linux, processes form a **tree hierarchy** where every process has exactly one parent.

```
           init (PID 1)  — Root of all processes
           /    |    \
         sshd  cron  httpd
          |           |  \
         bash       worker1  worker2
          |
         vim
```

**Key points:**
- In Unix, `init` (PID 1) is the ancestor of all processes — it's started by the kernel at boot.
- When a parent dies, its children may become **orphans** — adopted by `init`.
- Windows has no strict process hierarchy — parent-child relationships are less enforced.
- `ps auxf` on Linux shows the process tree with PID relationships.

---

## 8. Process Implementation

The OS implements processes by maintaining several kernel data structures:

**1. Process Table:**
- An array/list of all PCBs in the system.
- The OS looks up process information via the PID index.

**2. Ready Queue:**
- A queue of PCBs of all processes in the Ready state.
- Scheduler picks from this queue.

**3. Waiting/Blocked Queue:**
- Separate queues per I/O device — processes waiting for each device.

**Process Implementation Steps (fork on Unix):**
```
1. Allocate new PCB
2. Allocate unique PID
3. Copy parent's memory space (or use copy-on-write)
4. Copy parent's open file descriptors
5. Initialize CPU registers in new PCB
6. Add new PCB to ready queue
```

---

## 9. Cooperating Processes & Methods of Cooperation

### Cooperating Processes
Processes that can **affect or be affected by** each other. They share data or communicate.

**Reasons for cooperation:**
- **Information sharing** — share files, data between processes.
- **Computation speedup** — divide tasks into parallel sub-tasks.
- **Modularity** — split application into separate communicating modules.
- **Convenience** — user may do several tasks at once.

### 9.1 Cooperation by Sharing (Shared Memory)
- Processes share a **common region of memory**.
- Fast — no kernel involvement once set up.
- **Requires synchronization** — race conditions possible.
- System calls: `shmget()`, `shmat()` (Unix).

```
Process A ──writes──► [Shared Memory Region] ──reads──► Process B
```

> **Real-world example:** A web server process and a logging process share a memory-mapped log buffer. The server writes log entries; the logger reads and writes to disk. Both access the same memory region.

### 9.2 Cooperation by Communication (Message Passing)
- Processes exchange **messages** using `send()` and `receive()`.
- No shared memory needed — kernel mediates.
- Safer (no race conditions) but slower than shared memory.
- Used naturally in **distributed systems**.

**Types:**
- **Direct:** `send(P2, message)` — explicitly name recipient.
- **Indirect (Mailbox/Port):** Messages go through a named mailbox — `send(mailbox_A, message)`.

```
Process A ──send()──► [OS Kernel] ──receive()──► Process B
```

> **Real-world example:** In Android, apps communicate via **Intents** (a message-passing mechanism). When you share a photo from Gallery to WhatsApp, Gallery sends an Intent (message) that WhatsApp receives.

---

## 10. System Calls for Process Management

A **system call** is a programmatic way for a process to request services from the OS kernel.

### Types of System Calls

**Process Control:**
| System Call | Purpose | Unix | Windows |
|---|---|---|---|
| Create process | Fork/spawn | `fork()` | `CreateProcess()` |
| Execute program | Load new program | `exec()` | `CreateProcess()` |
| Terminate | End process | `exit()` | `ExitProcess()` |
| Wait for child | Parent waits | `wait()` | `WaitForSingleObject()` |
| Get process ID | Return PID | `getpid()` | `GetCurrentProcessId()` |
| Abort | Kill process | `kill()` | `TerminateProcess()` |

**File Management:**
`open()`, `read()`, `write()`, `close()`, `delete()`, `create()`

**Device Management:**
`request()`, `release()`, `read()`, `write()`

**Information Maintenance:**
`gettime()`, `getpid()`, `getprocessinfo()`

**Communication:**
`pipe()`, `shmget()`, `send()`, `receive()`, `socket()`

**Directory Management:**
`mkdir()`, `rmdir()`, `chdir()`, `link()`, `unlink()`

> **Real-world example:** Every time Python's `open("file.txt")` is called, it triggers a `open()` system call under the hood. The Python runtime asks the OS kernel to open the file — the programmer never writes to disk directly.

---

# PART B — THREADS (1 Hour)

## 11. Definition of Threads

A **thread** (also called a **lightweight process**) is the **smallest unit of CPU execution** within a process. A process can have one or more threads, all sharing the same process resources.

```
┌────────────────────────────────────────┐
│              PROCESS                   │
│  ┌──────────────────────────────────┐  │
│  │  Code | Data | Heap | Files      │  │  ← SHARED by all threads
│  └──────────────────────────────────┘  │
│  ┌─────────┐  ┌─────────┐  ┌───────┐  │
│  │Thread 1 │  │Thread 2 │  │Thread3│  │  ← Each has OWN: Stack + Registers + PC
│  └─────────┘  └─────────┘  └───────┘  │
└────────────────────────────────────────┘
```

**Each thread has its own:** Stack, Registers, Program Counter, Thread ID.
**Threads share:** Code section, Data section, Heap, Open Files, Signal handlers.

---

## 12. The Thread Model

### Single-Threaded Process
- One thread of execution.
- Can do only one thing at a time.
- If thread blocks (e.g., I/O), whole process blocks.

### Multi-Threaded Process
- Multiple threads within one process.
- True concurrent execution on multi-core CPUs.
- One thread can block without blocking others.

### Thread vs. Process

| Feature | Process | Thread |
|---|---|---|
| Definition | Program in execution | Smallest unit of execution within a process |
| Memory | Own separate memory space | Shares memory with other threads in same process |
| Communication | IPC (slow — kernel involved) | Direct memory access (fast) |
| Creation cost | Heavy — allocate memory, PCB | Lightweight — share existing process resources |
| Context switch | Slow — save/restore full process state | Fast — save/restore only thread-specific state |
| Failure isolation | Crash doesn't affect other processes | Crash can bring down entire process |
| Example | Chrome browser (process) | Chrome: render thread, JS thread, network thread |

> **Real-world example:** A bank's server application — one process per client would be very heavy. Instead, one process with one thread per client request (thread pool) is far more efficient.

---

## 13. Implementing Threads in User Space

### User-Level Threads (ULT)
Threads managed **entirely by a user-space thread library** — the OS kernel is unaware of them.

```
User Space:   T1    T2    T3    ← Managed by thread library
              ↓     ↓     ↓
              └─────┴─────┘
                    ↓
Kernel Space:    Process P      ← OS only sees one process
```

**Thread library handles:** thread creation, scheduling, switching, synchronization.
**Examples:** POSIX Pthreads (user mode), Java green threads (early Java), GNU Pth.

**Advantages:**
- Very fast thread switching (no kernel call needed).
- Can run on any OS — no kernel support required.
- Thread library can use custom scheduling algorithm.

**Disadvantages:**
- If one thread makes a blocking system call, **all threads block** (kernel blocks the whole process).
- Cannot exploit multiple CPUs — kernel only sees one process.
- Thread priorities not enforced by OS.

### Kernel-Level Threads (KLT)
Threads managed **directly by the OS kernel**.

```
User Space:   T1    T2    T3
              ↓     ↓     ↓
Kernel Space: K1    K2    K3    ← Kernel manages each thread
```

**Examples:** Windows threads, Linux pthreads (NPTL), Java threads (modern JVM).

**Advantages:**
- True parallelism on multi-core CPUs.
- One thread blocks → kernel runs another thread of same process.
- OS-aware scheduling with real priorities.

**Disadvantages:**
- Thread operations require system calls — slower than ULT.
- Higher overhead — kernel must track each thread.

---

## 14. Benefits of Multithreading

| Benefit | Explanation | Example |
|---|---|---|
| **Responsiveness** | App stays usable even if one thread blocks | Browser loads page while you type URL |
| **Resource Sharing** | Threads share process memory — no IPC overhead | Word spellcheck shares document data |
| **Economy** | Creating threads cheaper than processes | Thread pool in web servers |
| **Scalability** | Threads run on multiple CPU cores | Video encoder uses all 8 cores via threads |

---

## 15. Multithreading Models

### 15.1 Many-to-One Model
- Many user threads → **one** kernel thread.
- No true parallelism. If one blocks, all block.
- Example: Early Java green threads.

### 15.2 One-to-One Model
- Each user thread → **one** kernel thread.
- True parallelism. Creating many threads = expensive.
- Example: **Linux (pthreads/NPTL), Windows**.

### 15.3 Many-to-Many Model
- Many user threads → **many** kernel threads (≤ user threads).
- Best of both worlds.
- Example: Solaris, Windows with fibers.

---

# PART C — INTER-PROCESS COMMUNICATION & SYNCHRONIZATION (6 Hours)

## 16. Race Condition

A **race condition** occurs when two or more processes/threads access shared data concurrently and the final result depends on the **order of execution**.

**Example — Bank Account:**
```
Balance = $1000
Process A (Deposit $200):   Process B (Withdraw $300):
1. Read balance = $1000     1. Read balance = $1000
2. balance = 1000 + 200     2. balance = 1000 - 300
3. Write $1200              3. Write $700   ← WINS (writes last)
Final = $700  ← WRONG! Should be $900
```

> **Real-world example:** Two threads simultaneously adding items to a shopping cart — if not synchronized, one addition may be lost.

---

## 17. Critical Section

The **critical section** is a segment of code where **shared resources are accessed** and which must not be executed by more than one process simultaneously.

```
while (true) {
    // ─── Entry Section ─── (request permission)
    
    // ─── CRITICAL SECTION ─── (access shared data)
    
    // ─── Exit Section ─── (release permission)
    
    // ─── Remainder Section ─── (non-critical code)
}
```

### Requirements for Critical Section Solution:
1. **Mutual Exclusion** — Only one process in CS at a time.
2. **Progress** — If CS empty, a waiting process can enter without delay.
3. **Bounded Waiting** — No process waits indefinitely (no starvation).

---

## 18. Avoiding the Critical Region

### 18.1 Mutual Exclusion and Serializability
**Mutual Exclusion** ensures only one process executes in the critical section at a time.

**Serializability** means a concurrent execution is correct if it's equivalent to some **serial (sequential) execution**. Even if processes interleave, the final result must be the same as if they ran one after another.

### 18.2 Mutual Exclusion Conditions
For a valid mutual exclusion solution, it must guarantee:
- No two processes are simultaneously in their CS.
- No assumptions about CPU speeds or number of CPUs.
- No process running outside CS may block others.
- No process waits forever to enter CS.

### 18.3 Proposals for Achieving Mutual Exclusion

**A) Disabling Interrupts**
- Disable all interrupts before entering CS, re-enable on exit.
- **Problem:** Only works on single-core. Dangerous in user mode.

**B) Lock Variables**
- Shared boolean `lock`. If 0, set to 1 and enter CS.
- **Problem:** Race condition on the lock itself.

```c
while (lock == 1);  // busy wait
lock = 1;
// critical section
lock = 0;
```

**C) Turn Variable (Strict Alternation)**
- `turn` variable alternates between 0 and 1.
- **Problem:** Violates Progress — Process A must wait even if B isn't interested.

**D) Peterson's Solution**
Uses `flag[2]` and `turn` for 2 processes:
```c
flag[i] = true;
turn = j;
while (flag[j] && turn == j);  // wait
// critical section
flag[i] = false;
```
✅ Satisfies all 3 requirements. ❌ Works only for 2 processes.

**E) Test and Set Lock (TSL Instruction)**
Hardware atomic instruction reads a lock and sets it to 1 in one uninterruptible step:
```assembly
enter_region:
    TSL REGISTER, LOCK   ; atomically: read LOCK → REGISTER, set LOCK=1
    CMP REGISTER, #0     ; was it zero?
    JNE enter_region     ; if not, spin
    RET                  ; entered CS

leave_region:
    MOVE LOCK, #0        ; release lock
    RET
```
✅ Works on multi-processor. Atomicity prevents race on the lock itself.

---

## 19. Types of Mutual Exclusion Mechanisms

### 19.1 Semaphore
An **integer variable** accessed only through two atomic operations:
- `wait(S)` (P/down): If S > 0, decrement S. If S ≤ 0, sleep.
- `signal(S)` (V/up): Increment S; if any process sleeping, wake one.

**Binary Semaphore (Mutex):** Value 0 or 1 — mutual exclusion.
**Counting Semaphore:** Any non-negative integer — resource pool management.

```c
semaphore mutex = 1;
wait(mutex);       // P — enter CS
// critical section
signal(mutex);     // V — leave CS
```

> **Example:** Parking lot with 10 spaces = counting semaphore initialized to 10.

### 19.2 Monitors
High-level synchronization construct — a module where only **one process is active at a time** (mutual exclusion is automatic).

Uses **condition variables:** `wait()` suspends the process, `signal()` wakes a waiting process.

```c
monitor BoundedBuffer {
    int count = 0;
    condition notFull, notEmpty;
    void produce() {
        if (count == N) wait(notFull);
        // add item
        count++;
        signal(notEmpty);
    }
    void consume() {
        if (count == 0) wait(notEmpty);
        // remove item
        count--;
        signal(notFull);
    }
}
```

> **Real-world example:** Java's `synchronized` keyword implements a monitor.

### 19.3 Message Passing
Processes communicate using `send(message)` and `receive(message)` via the kernel.
- No shared memory needed.
- Safe — kernel mediates all communication.
- Used in distributed systems, microkernel OS (QNX, MINIX).

### 19.4 Producer-Consumer Problem (Bounded Buffer)
Classic IPC problem using 3 semaphores:
```c
semaphore mutex = 1;   // mutual exclusion
semaphore empty = N;   // empty slots
semaphore full = 0;    // full slots

// Producer:
wait(empty); wait(mutex);
// add to buffer
signal(mutex); signal(full);

// Consumer:
wait(full); wait(mutex);
// remove from buffer
signal(mutex); signal(empty);
```

---

## 20. Serializability, Locking & Time Stamp Protocols

**Serializability:** A concurrent schedule is correct if its result equals some serial execution.

**Locking Protocols:**
- **Two-Phase Locking (2PL):** Growing phase (acquire locks only) → Shrinking phase (release locks only). Guarantees serializability.

**Time Stamp Protocols:**
- Each transaction gets a timestamp at start.
- Older transaction = higher priority.
- Ensures conflict-free concurrent execution without locks.

---

## 21. Classical IPC Problems

### 21.1 Dining Philosophers Problem
- 5 philosophers, 5 forks. Each needs 2 forks to eat.
- **Deadlock:** All pick left fork simultaneously and wait for right.
- **Solution:** One philosopher picks right fork first (break symmetry).
- **Alternative:** Allow at most 4 philosophers to sit simultaneously.

### 21.2 The Readers and Writers Problem
- Multiple readers can read simultaneously.
- Writers need exclusive access.
- Solution uses semaphores to track reader count; last reader releases DB access.

> **Real-world example:** Wikipedia — many can read simultaneously; one editor locks the page.

### 21.3 The Sleeping Barber Problem
- 1 barber, N waiting chairs. Barber sleeps if no customers. Customer leaves if all chairs full.
- Models web server thread pool: barber = thread, customer = HTTP request, chairs = queue.

---

# PART D — PROCESS SCHEDULING (5 Hours)

## 22. Process Scheduling

**CPU Scheduling** is the task of deciding **which process in the ready queue gets the CPU next**.

> **Simple analogy:** Nepal ko bank maa token system cha. Kasle pahile token liyecha, kasle kati din holi, kasle priority paaucha — yo decide garne kaam nai CPU Scheduling ho. OS le automatically yo decision lirako hunchha milliseconds maa.

---

## 23. Dispatcher

The **dispatcher** is the OS module that gives control of the CPU to the process selected by the scheduler.

**Dispatcher functions:**
- **Context switching** — save current process state, load next process state.
- **Switching to user mode** — jump from kernel mode to user mode.
- **Jumping to the right location** — resume execution at the saved Program Counter.

**Dispatch Latency:** The time taken by the dispatcher to stop one process and start another. Should be minimized.

```
Scheduler selects Process B
         ↓
Dispatcher:
   1. Save Process A's registers → PCB_A
   2. Load Process B's registers ← PCB_B
   3. Switch to user mode
   4. Jump to PC of Process B
         ↓
Process B resumes executing
```

---

## 24. CPU-I/O Burst Cycle

Processes alternate between **CPU bursts** (computation) and **I/O bursts** (waiting for input/output):

```
[CPU Burst] → [I/O Burst] → [CPU Burst] → [I/O Burst] → ... → [Terminate]
```

- **I/O-bound process:** Many short CPU bursts, long I/O bursts. Example: text editor, web browser.
- **CPU-bound process:** Few long CPU bursts, short I/O bursts. Example: scientific computation, video encoding.

---

## 25. Context Switch

A **context switch** is the process of saving the state of a currently running process and restoring the state of another process.

**Steps:**
```
1. Save Process A's: PC, registers, stack pointer → PCB_A
2. Update PCB_A state to Ready or Waiting
3. Move PCB_A to appropriate queue
4. Load Process B's: PC, registers, stack pointer ← PCB_B
5. Update PCB_B state to Running
6. Jump to Process B's saved Program Counter
```

> **Context switch time is pure overhead** — no useful work is done during the switch. Typical: 1–10 microseconds.

---

## 26. Types of Scheduling

### Preemptive Scheduling
- OS **can forcibly remove** CPU from a running process.
- Triggered by: timer interrupt, higher-priority process arrival.
- Examples: Round Robin, SRTF, Priority (preemptive).

### Non-Preemptive Scheduling
- Once CPU is given, process runs until it **voluntarily yields** (I/O or exit).
- Examples: FCFS, SJF (non-preemptive), Priority (non-preemptive).

---

## 27. Scheduling Criteria

| Criterion | Description | Goal |
|---|---|---|
| **CPU Utilization** | % time CPU is busy | Maximize (target: 40–90%) |
| **Throughput** | Processes completed per unit time | Maximize |
| **Turnaround Time** | Total time from submission to completion | Minimize |
| **Waiting Time** | Time spent in ready queue | Minimize |
| **Response Time** | Time from request to first response | Minimize |

**Formulas:**
```
Turnaround Time (TAT) = Completion Time (CT) − Arrival Time (AT)
Waiting Time (WT)     = Turnaround Time − Burst Time
Response Time         = Time of first response − Arrival Time
Average WT            = Sum of all WT / Number of processes
```

---

## 28. Scheduling Algorithms with C Code

---

### 28.1 FCFS — First Come First Served

**Concept:**
- Simplest scheduling algorithm.
- Process that arrives first gets CPU first.
- **Non-preemptive** — once CPU is given, runs until completion.
- Implemented using a simple **queue (FIFO)**.

**Algorithm Steps:**
```
1. Sort processes by Arrival Time (AT)
2. Pick the first process from queue
3. Run it until completion (CT = previous CT + BT)
4. Calculate TAT = CT - AT
5. Calculate WT = TAT - BT
6. Repeat for all processes
7. Calculate averages
```

**Advantages:**
- Simple to understand and implement.
- No starvation — every process eventually gets CPU.

**Disadvantages:**
- **Convoy Effect** — short processes wait behind long ones.
- Poor average waiting time compared to other algorithms.

**C Program:**

```c
#include <stdio.h>

// Structure to store process information
struct Process {
    int pid;        // Process ID
    int at;         // Arrival Time
    int bt;         // Burst Time
    int ct;         // Completion Time
    int tat;        // Turnaround Time
    int wt;         // Waiting Time
};

// Function to sort processes by Arrival Time (Bubble Sort)
void sortByArrival(struct Process p[], int n) {
    struct Process temp;
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (p[j].at > p[j+1].at) {
                temp = p[j];
                p[j] = p[j+1];
                p[j+1] = temp;
            }
        }
    }
}

// FCFS Scheduling Function
void fcfs(struct Process p[], int n) {
    sortByArrival(p, n);

    int currentTime = 0;  // Track current time

    // Gantt Chart arrays
    int gantt_pid[100], gantt_start[100], gantt_end[100];
    int g = 0;  // Gantt chart index

    for (int i = 0; i < n; i++) {
        // If CPU is idle (gap between processes)
        if (currentTime < p[i].at) {
            currentTime = p[i].at;  // Jump to arrival time
        }

        gantt_pid[g]   = p[i].pid;
        gantt_start[g] = currentTime;

        // Process runs from currentTime to currentTime + BT
        currentTime    += p[i].bt;

        gantt_end[g]   = currentTime;
        g++;

        p[i].ct  = currentTime;             // Completion Time
        p[i].tat = p[i].ct - p[i].at;      // Turnaround Time
        p[i].wt  = p[i].tat - p[i].bt;     // Waiting Time
    }

    // Print Gantt Chart
    printf("\nGantt Chart:\n|");
    for (int i = 0; i < g; i++) {
        printf(" P%d |", gantt_pid[i]);
    }
    printf("\n0");
    for (int i = 0; i < g; i++) {
        printf("    %d", gantt_end[i]);
    }

    // Print Results Table
    printf("\n\nProcess\tAT\tBT\tCT\tTAT\tWT\n");
    float totalTAT = 0, totalWT = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\n",
               p[i].pid, p[i].at, p[i].bt,
               p[i].ct, p[i].tat, p[i].wt);
        totalTAT += p[i].tat;
        totalWT  += p[i].wt;
    }

    printf("\nAverage TAT = %.2f", totalTAT / n);
    printf("\nAverage WT  = %.2f\n", totalWT  / n);
}

int main() {
    int n;
    printf("Enter number of processes: ");
    scanf("%d", &n);

    struct Process p[n];
    for (int i = 0; i < n; i++) {
        p[i].pid = i + 1;
        printf("Enter Arrival Time and Burst Time for P%d: ", i+1);
        scanf("%d %d", &p[i].at, &p[i].bt);
    }

    fcfs(p, n);
    return 0;
}
```

**Code Explanation (Line by Line):**
```
struct Process     → Each process ko data ek saath store garne
sortByArrival()    → Processes lai AT anusaar sort garne (pehile aayeko pehile)
currentTime        → CPU ko clock — abhi kun time chha
currentTime < at   → CPU idle cha (process aaisakekai chha) → jump!
currentTime += bt  → Process run garyo — time badyo
ct = currentTime   → Completion Time = process sakieko time
tat = ct - at      → Kati total time lagyo (submit dekhi complete)
wt = tat - bt      → Ready queue maa kati kur-yo
```

---

**FCFS Detailed Numerical Example:**

**Given:**

| Process | Arrival Time (AT) | Burst Time (BT) |
|---|---|---|
| P1 | 0 | 5 |
| P2 | 1 | 3 |
| P3 | 2 | 8 |
| P4 | 3 | 6 |

**Step 1: Sort by Arrival Time**
```
Already sorted: P1(AT=0), P2(AT=1), P3(AT=2), P4(AT=3)
```

**Step 2: Execute P1**
```
Current Time = 0
P1 arrives at AT=0 → CPU available at 0 → Start immediately
P1 runs from t=0 to t=0+5 = t=5
CT(P1) = 5
Current Time = 5
```

**Step 3: Execute P2**
```
Current Time = 5
P2 arrived at AT=1, but CPU busy with P1 until t=5
P2 was WAITING in ready queue from t=1 to t=5 (4ms kur-yo!)
P2 runs from t=5 to t=5+3 = t=8
CT(P2) = 8
Current Time = 8
```

**Step 4: Execute P3**
```
Current Time = 8
P3 arrived at AT=2, waiting from t=2 to t=8 (6ms kur-yo!)
P3 runs from t=8 to t=8+8 = t=16
CT(P3) = 16
Current Time = 16
```

**Step 5: Execute P4**
```
Current Time = 16
P4 arrived at AT=3, waiting from t=3 to t=16 (13ms kur-yo!)
P4 runs from t=16 to t=16+6 = t=22
CT(P4) = 22
Current Time = 22
```

**Step 6: Gantt Chart**
```
| P1  | P2 | P3       | P4    |
0     5    8         16      22
```

**Step 7: Calculate TAT and WT**
```
TAT = CT - AT
WT  = TAT - BT

P1: TAT = 5  - 0 = 5,   WT = 5  - 5 = 0
P2: TAT = 8  - 1 = 7,   WT = 7  - 3 = 4
P3: TAT = 16 - 2 = 14,  WT = 14 - 8 = 6
P4: TAT = 22 - 3 = 19,  WT = 19 - 6 = 13
```

**Step 8: Final Table**

| Process | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 5 | 5 | 5 | **0** |
| P2 | 1 | 3 | 8 | 7 | **4** |
| P3 | 2 | 8 | 16 | 14 | **6** |
| P4 | 3 | 6 | 22 | 19 | **13** |

```
Average TAT = (5 + 7 + 14 + 19) / 4 = 45 / 4 = 11.25 ms
Average WT  = (0 + 4 + 6 + 13) / 4  = 23 / 4 = 5.75 ms
```

> **Convoy Effect देख्नुस्:** P2 (BT=3 मात्र) ले P1 (BT=5) को कारण 4ms कुर्नुपर्यो। P4 (BT=6) ले 13ms कुर्नुपर्यो! ठूलो process पछाडि सानो process धेरै कुर्नुपर्छ — यही convoy effect हो।

---

### 28.2 SJF — Shortest Job First

**Concept:**
- Among all available (arrived) processes, the one with **shortest Burst Time** gets CPU first.
- **Non-preemptive** — once started, runs to completion.
- **Optimal** — gives minimum average waiting time for non-preemptive algorithms.

**Algorithm Steps:**
```
1. Sort by Arrival Time initially
2. At each decision point, find all arrived processes
3. Among arrived processes, pick the one with MINIMUM BT
4. Run it to completion
5. Update current time, calculate CT, TAT, WT
6. Repeat
```

**Key Challenge:** OS cannot know future CPU burst times exactly. It **estimates** using:
```
Next Burst Estimate = α × (actual previous burst) + (1-α) × (previous estimate)
Where α = 0.5 typically (exponential averaging)
```

**C Program:**

```c
#include <stdio.h>
#include <limits.h>  // For INT_MAX

struct Process {
    int pid;
    int at;
    int bt;
    int remaining_bt;  // For tracking remaining burst
    int ct;
    int tat;
    int wt;
    int completed;     // 0 = not done, 1 = done
};

// SJF Non-Preemptive Scheduling
void sjf(struct Process p[], int n) {
    int completed = 0;
    int currentTime = 0;
    int gantt_pid[100], gantt_start[100], gantt_end[100];
    int g = 0;

    // Initialize
    for (int i = 0; i < n; i++) {
        p[i].completed = 0;
        p[i].remaining_bt = p[i].bt;
    }

    while (completed != n) {
        // Find process with minimum BT among arrived processes
        int minBT = INT_MAX;
        int selected = -1;

        for (int i = 0; i < n; i++) {
            // Process must have arrived AND not yet completed
            if (p[i].at <= currentTime && !p[i].completed) {
                if (p[i].bt < minBT) {
                    minBT = p[i].bt;
                    selected = i;
                }
                // Tie-breaking: if same BT, pick earlier arrival
                else if (p[i].bt == minBT && p[i].at < p[selected].at) {
                    selected = i;
                }
            }
        }

        // No process available — CPU idle, jump to next arrival
        if (selected == -1) {
            currentTime++;
            continue;
        }

        // Execute selected process to completion
        gantt_pid[g]   = p[selected].pid;
        gantt_start[g] = currentTime;

        currentTime += p[selected].bt;  // Run full burst

        gantt_end[g] = currentTime;
        g++;

        p[selected].ct  = currentTime;
        p[selected].tat = p[selected].ct - p[selected].at;
        p[selected].wt  = p[selected].tat - p[selected].bt;
        p[selected].completed = 1;
        completed++;
    }

    // Print Gantt Chart
    printf("\nGantt Chart:\n|");
    for (int i = 0; i < g; i++) printf(" P%d |", gantt_pid[i]);
    printf("\n");
    printf("%d", gantt_start[0]);
    for (int i = 0; i < g; i++) printf("    %d", gantt_end[i]);

    // Print Table
    printf("\n\nProcess\tAT\tBT\tCT\tTAT\tWT\n");
    float totalTAT = 0, totalWT = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\n",
               p[i].pid, p[i].at, p[i].bt,
               p[i].ct, p[i].tat, p[i].wt);
        totalTAT += p[i].tat;
        totalWT  += p[i].wt;
    }
    printf("\nAverage TAT = %.2f", totalTAT / n);
    printf("\nAverage WT  = %.2f\n", totalWT / n);
}

int main() {
    int n;
    printf("Enter number of processes: ");
    scanf("%d", &n);

    struct Process p[n];
    for (int i = 0; i < n; i++) {
        p[i].pid = i + 1;
        printf("Arrival Time and Burst Time for P%d: ", i+1);
        scanf("%d %d", &p[i].at, &p[i].bt);
    }

    sjf(p, n);
    return 0;
}
```

**Code Explanation:**
```
INT_MAX          → Sabai bhandaa thulo number (comparison suru garna)
completed flag   → Kun process sakiyo, kun sakiena track garne
selected = -1    → Koi process chhaninai sakena (CPU idle)
at <= currentTime → Process aaisakekai cha? Yedi ho bhane consider garne
minBT            → Abhi samma bhetteko sabai bhandaa chotto BT
currentTime++    → Koi available chhhaina — 1ms idle time pass
```

---

**SJF Detailed Numerical Example:**

**Given:**

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 6 |
| P2 | 1 | 8 |
| P3 | 2 | 7 |
| P4 | 3 | 3 |

**Step 1: At t=0, who has arrived?**
```
t=0: Only P1 arrived (AT=0)
Available: [P1]
→ Only choice: P1 (BT=6) wins
P1 runs from t=0 to t=0+6 = t=6
```

**Step 2: At t=6, who has arrived?**
```
t=6: P1 done. P2(AT=1)✓, P3(AT=2)✓, P4(AT=3)✓ all arrived
Available: [P2(BT=8), P3(BT=7), P4(BT=3)]
SJF picks: P4 — MINIMUM BT = 3
P4 runs from t=6 to t=6+3 = t=9
```

**Step 3: At t=9, who is available?**
```
t=9: P4 done. P2(BT=8) and P3(BT=7) still waiting
Available: [P2(BT=8), P3(BT=7)]
SJF picks: P3 — BT=7 < BT=8
P3 runs from t=9 to t=9+7 = t=16
```

**Step 4: At t=16, who is left?**
```
t=16: P3 done. Only P2 remaining
Available: [P2(BT=8)]
Only choice: P2
P2 runs from t=16 to t=16+8 = t=24
```

**Step 5: Gantt Chart**
```
| P1    | P4 | P3      | P2       |
0       6    9        16        24
```

**Step 6: Calculate CT, TAT, WT**
```
P1: CT=6,  TAT = 6-0  = 6,  WT = 6-6  = 0
P4: CT=9,  TAT = 9-3  = 6,  WT = 6-3  = 3
P3: CT=16, TAT = 16-2 = 14, WT = 14-7 = 7
P2: CT=24, TAT = 24-1 = 23, WT = 23-8 = 15
```

**Step 7: Final Table**

| Process | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 6 | 6 | 6 | **0** |
| P4 | 3 | 3 | 9 | 6 | **3** |
| P3 | 2 | 7 | 16 | 14 | **7** |
| P2 | 1 | 8 | 24 | 23 | **15** |

```
Average TAT = (6 + 6 + 14 + 23) / 4 = 49 / 4 = 12.25 ms
Average WT  = (0 + 3 + 7 + 15) / 4  = 25 / 4 = 6.25 ms

Compare with FCFS same data:
FCFS Avg WT ≈ 10+ ms → SJF le kaafi ghataayo!
```

> **Note:** P2 (BT=8) ले 15ms कुर्नुपर्यो — यो **Starvation** को सुरुआत हो। यदि continuously short jobs आइरहे भने P2 कहिल्यै चल्दैन!

---

### 28.3 SRTF — Shortest Remaining Time First

**Concept:**
- **Preemptive version of SJF**.
- At every moment, if a new process arrives with **remaining BT less than current process's remaining BT** → preempt!
- Decision is made at **every new arrival** and at **every completion**.

**Algorithm Steps:**
```
1. At each time unit, check: did any new process arrive?
2. If yes: compare new process's BT with current process's remaining BT
3. If new process BT < remaining BT → PREEMPT current, run new
4. If no: continue current process
5. Update remaining BT every time unit
6. When remaining BT = 0 → process complete
```

**C Program:**

```c
#include <stdio.h>
#include <limits.h>

struct Process {
    int pid;
    int at;
    int bt;
    int remaining_bt;
    int ct;
    int tat;
    int wt;
    int started;    // Has process started at least once?
    int completed;
};

void srtf(struct Process p[], int n) {
    int currentTime = 0;
    int completed = 0;
    int prev = -1;  // Previously running process

    // Gantt chart — record every second
    int gantt[1000];  // gantt[t] = which process ran at time t
    int totalTime = 0;

    // Initialize
    for (int i = 0; i < n; i++) {
        p[i].remaining_bt = p[i].bt;
        p[i].completed = 0;
        // Calculate total time needed
        totalTime += p[i].bt;
    }

    // Find last arrival time to set upper bound
    int maxTime = 0;
    for (int i = 0; i < n; i++)
        if (p[i].at > maxTime) maxTime = p[i].at;
    maxTime += totalTime;  // Safe upper bound

    for (currentTime = 0; completed != n; currentTime++) {
        // Find process with minimum remaining BT (arrived and not done)
        int minRT = INT_MAX;
        int selected = -1;

        for (int i = 0; i < n; i++) {
            if (p[i].at <= currentTime && !p[i].completed) {
                if (p[i].remaining_bt < minRT) {
                    minRT = p[i].remaining_bt;
                    selected = i;
                }
            }
        }

        if (selected == -1) {
            gantt[currentTime] = -1;  // CPU idle
            continue;
        }

        gantt[currentTime] = p[selected].pid;
        p[selected].remaining_bt--;  // Run for 1 unit

        // Check if process just completed
        if (p[selected].remaining_bt == 0) {
            p[selected].ct = currentTime + 1;
            p[selected].tat = p[selected].ct - p[selected].at;
            p[selected].wt  = p[selected].tat - p[selected].bt;
            p[selected].completed = 1;
            completed++;
        }
    }

    // Print condensed Gantt Chart
    printf("\nGantt Chart:\n|");
    int i = 0;
    while (i < currentTime) {
        int pid = gantt[i];
        int start = i;
        // Find how long this process ran continuously
        while (i < currentTime && gantt[i] == pid) i++;
        if (pid == -1) printf(" IDLE |");
        else printf(" P%d(%d-%d) |", pid, start, i);
    }

    // Print Results
    printf("\n\nProcess\tAT\tBT\tCT\tTAT\tWT\n");
    float totalTAT = 0, totalWT = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\n",
               p[i].pid, p[i].at, p[i].bt,
               p[i].ct, p[i].tat, p[i].wt);
        totalTAT += p[i].tat;
        totalWT  += p[i].wt;
    }
    printf("\nAverage TAT = %.2f", totalTAT / n);
    printf("\nAverage WT  = %.2f\n", totalWT / n);
}

int main() {
    int n;
    printf("Enter number of processes: ");
    scanf("%d", &n);

    struct Process p[n];
    for (int i = 0; i < n; i++) {
        p[i].pid = i + 1;
        printf("Arrival Time and Burst Time for P%d: ", i+1);
        scanf("%d %d", &p[i].at, &p[i].bt);
    }

    srtf(p, n);
    return 0;
}
```

**Code Explanation:**
```
remaining_bt     → Haalko remaining burst time (ghattaidai jaanchha)
currentTime loop → Har 1ms check garchha — preempt garnu parcha ki pardaina
remaining_bt--   → 1 unit time garey → remaining 1 le ghattaayo
remaining_bt==0  → Process complete! CT, TAT, WT calculate
gantt[]          → Har second kle process chalyako record
```

---

**SRTF Detailed Numerical Example:**

**Given:**

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 7 |
| P2 | 2 | 4 |
| P3 | 4 | 1 |
| P4 | 5 | 4 |

**Step-by-Step Timeline:**

```
t=0:
  Arrived: P1 (BT=7)
  Running: P1 (only choice)
  P1 remaining = 7

t=1:
  Arrived: P1 still running
  No new arrival since last check
  Running: P1
  P1 remaining = 6

t=2: ⚠️ NEW ARRIVAL!
  P2 arrives (BT=4)
  Current: P1 remaining = 5
  Compare: P2 BT(4) vs P1 remaining(5)
  4 < 5 → PREEMPT P1! Switch to P2
  Running: P2
  P2 remaining = 4

t=3:
  No new arrival
  Running: P2
  P2 remaining = 3

t=4: ⚠️ NEW ARRIVAL!
  P3 arrives (BT=1)
  Current: P2 remaining = 2
  Compare: P3 BT(1) vs P2 remaining(2)
  1 < 2 → PREEMPT P2! Switch to P3
  Running: P3
  P3 remaining = 1

t=5: ⚠️ NEW ARRIVAL + COMPLETION!
  P4 arrives (BT=4)
  P3 remaining = 0 → P3 COMPLETES! CT(P3) = 5
  
  Now available: P1(remaining=5), P2(remaining=2), P4(BT=4)
  Minimum: P2 remaining = 2
  Running: P2
  P2 remaining = 2

t=6:
  No new arrival
  Running: P2
  P2 remaining = 1

t=7: P2 COMPLETES! CT(P2) = 7
  Available: P1(remaining=5), P4(remaining=4)
  Minimum: P4 remaining = 4 < P1 remaining = 5
  Running: P4
  P4 remaining = 4

t=8,9,10: P4 running (4,3,2,1)

t=11: P4 COMPLETES! CT(P4) = 11
  Available: P1(remaining=5)
  Only P1 left
  Running: P1

t=12,13,14,15: P1 running (5,4,3,2,1)

t=16: P1 COMPLETES! CT(P1) = 16
```

**Gantt Chart:**
```
|P1(0-2)|P2(2-4)|P3(4-5)|P2(5-7)|P4(7-11)|P1(11-16)|
0       2       4       5       7        11        16
```

**Calculate CT, TAT, WT:**
```
P1: CT=16, TAT = 16-0 = 16, WT = 16-7 = 9
P2: CT=7,  TAT = 7-2  = 5,  WT = 5-4  = 1
P3: CT=5,  TAT = 5-4  = 1,  WT = 1-1  = 0
P4: CT=11, TAT = 11-5 = 6,  WT = 6-4  = 2
```

**Final Table:**

| Process | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 7 | 16 | 16 | **9** |
| P2 | 2 | 4 | 7 | 5 | **1** |
| P3 | 4 | 1 | 5 | 1 | **0** |
| P4 | 5 | 4 | 11 | 6 | **2** |

```
Average TAT = (16+5+1+6) / 4 = 28/4 = 7.0 ms
Average WT  = (9+1+0+2)  / 4 = 12/4 = 3.0 ms ← Best among all!
```

> **Key Observation:** P3 (BT=1) ले आउनासाथ P2 लाई preempt गर्यो र 1ms मै सकियो। यो SRTF को power हो — shortest job always gets through first!

---

### 28.4 Round Robin (RR)

**Concept:**
- Each process gets CPU for a fixed **Time Quantum (q)**.
- After q expires, process is **preempted** and added to **end of ready queue**.
- Continues until all processes complete.
- **No starvation** — every process gets CPU within q × n time.

**Algorithm Steps:**
```
1. Sort by Arrival Time
2. Add first process to ready queue
3. Pick front of queue, run for min(remaining_bt, q) time
4. During this run, add any newly arrived processes to queue
5. If process not done → add to end of queue
6. If process done → calculate CT, TAT, WT
7. Repeat from step 3 until queue empty
```

**C Program:**

```c
#include <stdio.h>

#define MAX 100

struct Process {
    int pid;
    int at;
    int bt;
    int remaining_bt;
    int ct;
    int tat;
    int wt;
    int in_queue;   // Already added to ready queue?
    int completed;
};

// Simple circular queue implementation
int queue[MAX * 10];
int front = 0, rear = 0;

void enqueue(int pid) {
    queue[rear++] = pid;
}

int dequeue() {
    return queue[front++];
}

int isEmpty() {
    return front == rear;
}

void roundRobin(struct Process p[], int n, int quantum) {
    int currentTime = 0;
    int completed = 0;
    int gantt_pid[MAX*10], gantt_start[MAX*10], gantt_end[MAX*10];
    int g = 0;

    // Reset queue
    front = rear = 0;

    // Initialize
    for (int i = 0; i < n; i++) {
        p[i].remaining_bt = p[i].bt;
        p[i].completed = 0;
        p[i].in_queue = 0;
    }

    // Sort by arrival time first
    // (using simple selection sort)
    for (int i = 0; i < n-1; i++) {
        for (int j = i+1; j < n; j++) {
            if (p[i].at > p[j].at) {
                struct Process temp = p[i];
                p[i] = p[j];
                p[j] = temp;
            }
        }
    }

    // Add first arrived process to queue
    // Check if multiple processes arrive at t=0
    for (int i = 0; i < n; i++) {
        if (p[i].at == 0) {
            enqueue(i);
            p[i].in_queue = 1;
        }
    }

    // If nothing at t=0, jump to first arrival
    if (isEmpty()) {
        currentTime = p[0].at;
        enqueue(0);
        p[0].in_queue = 1;
    }

    while (!isEmpty()) {
        int idx = dequeue();  // Get process index from queue

        // Run for quantum or remaining time (whichever is less)
        int runTime = (p[idx].remaining_bt < quantum) ?
                       p[idx].remaining_bt : quantum;

        gantt_pid[g]   = p[idx].pid;
        gantt_start[g] = currentTime;

        // Simulate running one unit at a time to check new arrivals
        for (int t = 0; t < runTime; t++) {
            currentTime++;
            p[idx].remaining_bt--;

            // Check if any new process arrived during this quantum
            for (int i = 0; i < n; i++) {
                if (p[i].at == currentTime && !p[i].in_queue && !p[i].completed) {
                    enqueue(i);
                    p[i].in_queue = 1;
                }
            }
        }

        gantt_end[g] = currentTime;
        g++;

        if (p[idx].remaining_bt == 0) {
            // Process completed
            p[idx].ct = currentTime;
            p[idx].tat = p[idx].ct - p[idx].at;
            p[idx].wt  = p[idx].tat - p[idx].bt;
            p[idx].completed = 1;
            completed++;
        } else {
            // Not done yet — re-add to end of queue
            enqueue(idx);
        }

        // If queue is empty but processes remain, jump to next arrival
        if (isEmpty() && completed != n) {
            for (int i = 0; i < n; i++) {
                if (!p[i].completed && !p[i].in_queue) {
                    currentTime = p[i].at;
                    enqueue(i);
                    p[i].in_queue = 1;
                    break;
                }
            }
        }
    }

    // Print Gantt Chart
    printf("\nGantt Chart (q=%d):\n|", quantum);
    for (int i = 0; i < g; i++) {
        printf(" P%d |", gantt_pid[i]);
    }
    printf("\n%d", gantt_start[0]);
    for (int i = 0; i < g; i++) {
        printf("  %d", gantt_end[i]);
    }

    // Print Results
    printf("\n\nProcess\tAT\tBT\tCT\tTAT\tWT\n");
    float totalTAT = 0, totalWT = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\n",
               p[i].pid, p[i].at, p[i].bt,
               p[i].ct, p[i].tat, p[i].wt);
        totalTAT += p[i].tat;
        totalWT  += p[i].wt;
    }
    printf("\nAverage TAT = %.2f", totalTAT / n);
    printf("\nAverage WT  = %.2f\n", totalWT / n);
}

int main() {
    int n, quantum;
    printf("Enter number of processes: ");
    scanf("%d", &n);

    struct Process p[n];
    for (int i = 0; i < n; i++) {
        p[i].pid = i + 1;
        printf("Arrival Time and Burst Time for P%d: ", i+1);
        scanf("%d %d", &p[i].at, &p[i].bt);
    }

    printf("Enter Time Quantum: ");
    scanf("%d", &quantum);

    roundRobin(p, n, quantum);
    return 0;
}
```

**Code Explanation:**
```
queue[]          → Ready queue — palo raakhney thau
enqueue(idx)     → Process lai queue ko antmaa thapne
dequeue()        → Queue ko aagaadibata process nikalne
runTime          → Quantum ra remaining BT maa minimum line (process sakina sakcha)
in_queue flag    → Ek process lai dui palta queue maa nahalna rokne
at == currentTime → Quantum chaldaa naya process aaunchha? → queue maa thap
remaining_bt==0  → Process sakiyo!
else enqueue     → Saakiyena → queue ko antmaa patha
```

---

**Round Robin Detailed Numerical Example (q=3):**

**Given:**

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 8 |
| P2 | 1 | 4 |
| P3 | 2 | 3 |
| P4 | 5 | 5 |

**Ready Queue State at Each Step:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
t=0: Queue=[P1]
  → P1 starts running (q=3)
  → During P1 run: P2 arrives at t=1, P3 arrives at t=2
  → Queue after additions: [P2, P3] (added while P1 running)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t=3: P1 quantum expired (remaining=5), Queue=[P2, P3]
  → P1 goes to END of queue → Queue=[P2, P3, P1]
  → P2 starts running (q=3)
  → P2 remaining=4, runs for min(4,3)=3 → remaining becomes 1
  → During P2 run: No new arrivals (P4 arrives at t=5)
  → At t=5: P4 arrives → Queue=[P3, P1, P4]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t=6: P2 quantum expired (remaining=1), Queue=[P3, P1, P4]
  → P2 goes to END → Queue=[P3, P1, P4, P2]
  → P3 starts running (q=3)
  → P3 remaining=3, runs for min(3,3)=3 → remaining becomes 0
  → P3 COMPLETES! CT(P3) = 9
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t=9: P3 done. Queue=[P1, P4, P2]
  → P1 starts running (remaining=5, q=3)
  → runs min(5,3)=3 → remaining becomes 2
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t=12: P1 quantum expired (remaining=2), Queue=[P4, P2]
  → P1 goes to END → Queue=[P4, P2, P1]
  → P4 starts running (remaining=5, q=3)
  → runs min(5,3)=3 → remaining becomes 2
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t=15: P4 quantum expired (remaining=2), Queue=[P2, P1]
  → P4 goes to END → Queue=[P2, P1, P4]
  → P2 starts running (remaining=1, q=3)
  → runs min(1,3)=1 → remaining becomes 0
  → P2 COMPLETES! CT(P2) = 16
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t=16: P2 done. Queue=[P1, P4]
  → P1 starts running (remaining=2, q=3)
  → runs min(2,3)=2 → remaining becomes 0
  → P1 COMPLETES! CT(P1) = 18
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t=18: P1 done. Queue=[P4]
  → P4 starts running (remaining=2, q=3)
  → runs min(2,3)=2 → remaining becomes 0
  → P4 COMPLETES! CT(P4) = 20
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Gantt Chart:**
```
|P1(0-3)|P2(3-6)|P3(6-9)|P1(9-12)|P4(12-15)|P2(15-16)|P1(16-18)|P4(18-20)|
0       3       6       9        12        15        16        18        20
```

**Calculate TAT and WT:**
```
P1: CT=18, TAT = 18-0 = 18, WT = 18-8 = 10
P2: CT=16, TAT = 16-1 = 15, WT = 15-4 = 11
P3: CT=9,  TAT = 9-2  = 7,  WT = 7-3  = 4
P4: CT=20, TAT = 20-5 = 15, WT = 15-5 = 10
```

**Final Table:**

| Process | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 8 | 18 | 18 | **10** |
| P2 | 1 | 4 | 16 | 15 | **11** |
| P3 | 2 | 3 | 9 | 7 | **4** |
| P4 | 5 | 5 | 20 | 15 | **10** |

```
Average TAT = (18+15+7+15) / 4 = 55/4 = 13.75 ms
Average WT  = (10+11+4+10) / 4 = 35/4 = 8.75 ms
```

> **Observation:** WT धेरै देखिन्छ तर **no starvation** — P3 (BT=3) ले राम्रै चाँडो सक्यो। Interactive systems मा response time important हुन्छ, average WT होइन।

---

### 28.5 Priority Scheduling

**Concept:**
- Each process has a **priority number** (lower number = higher priority typically).
- Highest priority process gets CPU first.
- Can be **preemptive** (new high-priority process preempts current) or **non-preemptive**.
- **Problem:** Starvation — low priority process may never run.
- **Solution:** Aging — increase priority of waiting processes over time.

**C Program (Non-Preemptive):**

```c
#include <stdio.h>
#include <limits.h>

struct Process {
    int pid;
    int at;
    int bt;
    int priority;   // Lower number = Higher priority
    int ct;
    int tat;
    int wt;
    int completed;
};

void priorityScheduling(struct Process p[], int n) {
    int completed = 0;
    int currentTime = 0;
    int gantt_pid[100], gantt_start[100], gantt_end[100];
    int g = 0;

    for (int i = 0; i < n; i++) p[i].completed = 0;

    while (completed != n) {
        int highestPriority = INT_MAX;
        int selected = -1;

        // Find arrived, incomplete process with HIGHEST priority (lowest number)
        for (int i = 0; i < n; i++) {
            if (p[i].at <= currentTime && !p[i].completed) {
                if (p[i].priority < highestPriority) {
                    highestPriority = p[i].priority;
                    selected = i;
                }
                // Tie-break: earlier arrival wins
                else if (p[i].priority == highestPriority &&
                         p[i].at < p[selected].at) {
                    selected = i;
                }
            }
        }

        if (selected == -1) {
            currentTime++;  // CPU idle
            continue;
        }

        gantt_pid[g]   = p[selected].pid;
        gantt_start[g] = currentTime;

        currentTime += p[selected].bt;

        gantt_end[g] = currentTime;
        g++;

        p[selected].ct  = currentTime;
        p[selected].tat = p[selected].ct - p[selected].at;
        p[selected].wt  = p[selected].tat - p[selected].bt;
        p[selected].completed = 1;
        completed++;
    }

    // Print Gantt Chart
    printf("\nGantt Chart:\n|");
    for (int i = 0; i < g; i++) printf(" P%d |", gantt_pid[i]);
    printf("\n%d", gantt_start[0]);
    for (int i = 0; i < g; i++) printf("    %d", gantt_end[i]);

    // Print Results
    printf("\n\nProcess\tAT\tBT\tPRI\tCT\tTAT\tWT\n");
    float totalTAT = 0, totalWT = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\t%d\n",
               p[i].pid, p[i].at, p[i].bt, p[i].priority,
               p[i].ct, p[i].tat, p[i].wt);
        totalTAT += p[i].tat;
        totalWT  += p[i].wt;
    }
    printf("\nAverage TAT = %.2f", totalTAT / n);
    printf("\nAverage WT  = %.2f\n", totalWT / n);
}

int main() {
    int n;
    printf("Enter number of processes: ");
    scanf("%d", &n);

    struct Process p[n];
    for (int i = 0; i < n; i++) {
        p[i].pid = i + 1;
        printf("AT, BT, Priority for P%d: ", i+1);
        scanf("%d %d %d", &p[i].at, &p[i].bt, &p[i].priority);
    }

    priorityScheduling(p, n);
    return 0;
}
```

**Code Explanation:**
```
priority field    → Process ko priority number store garne
highestPriority   → Abhi samma bhetteko sabai bhandaa thulo priority (sano number)
INT_MAX           → Starting comparison value (sabai bhandaa thulo)
p[i].priority < highestPriority → Sano number = badi priority = select
```

---

**Priority Scheduling Detailed Numerical Example:**

**Given (Lower number = Higher priority):**

| Process | AT | BT | Priority |
|---|---|---|---|
| P1 | 0 | 10 | 3 |
| P2 | 0 | 1 | 1 ← Highest |
| P3 | 0 | 2 | 4 |
| P4 | 0 | 1 | 5 ← Lowest |
| P5 | 0 | 5 | 2 |

**Step 1: t=0 — All processes arrived**
```
Available: P1(pri=3), P2(pri=1), P3(pri=4), P4(pri=5), P5(pri=2)
Highest Priority = smallest number = P2 (pri=1)
P2 runs: BT=1, from t=0 to t=1
CT(P2) = 1
```

**Step 2: t=1**
```
Available: P1(pri=3), P3(pri=4), P4(pri=5), P5(pri=2)
Highest = P5 (pri=2)
P5 runs: BT=5, from t=1 to t=6
CT(P5) = 6
```

**Step 3: t=6**
```
Available: P1(pri=3), P3(pri=4), P4(pri=5)
Highest = P1 (pri=3)
P1 runs: BT=10, from t=6 to t=16
CT(P1) = 16
```

**Step 4: t=16**
```
Available: P3(pri=4), P4(pri=5)
Highest = P3 (pri=4)
P3 runs: BT=2, from t=16 to t=18
CT(P3) = 18
```

**Step 5: t=18**
```
Available: P4(pri=5)
Only P4 left
P4 runs: BT=1, from t=18 to t=19
CT(P4) = 19
```

**Gantt Chart:**
```
|P2|P5    |P1              |P3|P4|
0  1      6               16 18 19
```

**Final Table:**

| Process | AT | BT | Priority | CT | TAT | WT |
|---|---|---|---|---|---|---|
| P2 | 0 | 1 | 1 | 1 | 1 | **0** |
| P5 | 0 | 5 | 2 | 6 | 6 | **1** |
| P1 | 0 | 10 | 3 | 16 | 16 | **6** |
| P3 | 0 | 2 | 4 | 18 | 18 | **16** |
| P4 | 0 | 1 | 5 | 19 | 19 | **18** |

```
Average TAT = (1+6+16+18+19) / 5 = 60/5 = 12.0 ms
Average WT  = (0+1+6+16+18) / 5  = 41/5 = 8.2 ms
```

> **Starvation देख्नुस्:** P4 (lowest priority=5) ले 18ms कुर्नुपर्यो! यदि continuously high-priority processes आइरहे भने P4 **forever** wait गर्छ — **Starvation!**

---

### 28.6 HRN — Highest Response Ratio Next

**Concept:**
- Non-preemptive algorithm that balances short jobs AND long-waiting jobs.
- Calculates **Response Ratio** for each available process.
- Process with **highest ratio** gets CPU.
- **No starvation** — waiting time बढ्दै जाँदा ratio बढ्छ, eventually सबैलाई पालो मिल्छ।

**Formula:**
```
Response Ratio (RR) = (Waiting Time + Burst Time) / Burst Time
                    = (W + BT) / BT

Where Waiting Time = Current Time - Arrival Time
```

**C Program:**

```c
#include <stdio.h>

struct Process {
    int pid;
    int at;
    int bt;
    int ct;
    int tat;
    int wt;
    int completed;
};

void hrn(struct Process p[], int n) {
    int completed = 0;
    int currentTime = 0;
    int gantt_pid[100], gantt_start[100], gantt_end[100];
    int g = 0;

    for (int i = 0; i < n; i++) p[i].completed = 0;

    while (completed != n) {
        double highestRR = -1;
        int selected = -1;

        for (int i = 0; i < n; i++) {
            if (p[i].at <= currentTime && !p[i].completed) {
                // Calculate waiting time for this process
                int waitingTime = currentTime - p[i].at;

                // HRN formula
                double responseRatio = (double)(waitingTime + p[i].bt) / p[i].bt;

                if (responseRatio > highestRR) {
                    highestRR = responseRatio;
                    selected = i;
                }
            }
        }

        if (selected == -1) {
            currentTime++;
            continue;
        }

        // Print ratio for transparency
        printf("t=%d: P%d selected (RR = %.2f)\n",
               currentTime, p[selected].pid, highestRR);

        gantt_pid[g]   = p[selected].pid;
        gantt_start[g] = currentTime;

        currentTime += p[selected].bt;

        gantt_end[g] = currentTime;
        g++;

        p[selected].ct  = currentTime;
        p[selected].tat = p[selected].ct - p[selected].at;
        p[selected].wt  = p[selected].tat - p[selected].bt;
        p[selected].completed = 1;
        completed++;
    }

    // Print Gantt Chart
    printf("\nGantt Chart:\n|");
    for (int i = 0; i < g; i++) printf(" P%d |", gantt_pid[i]);
    printf("\n%d", gantt_start[0]);
    for (int i = 0; i < g; i++) printf("    %d", gantt_end[i]);

    // Print Results
    printf("\n\nProcess\tAT\tBT\tCT\tTAT\tWT\n");
    float totalTAT = 0, totalWT = 0;
    for (int i = 0; i < n; i++) {
        printf("P%d\t%d\t%d\t%d\t%d\t%d\n",
               p[i].pid, p[i].at, p[i].bt,
               p[i].ct, p[i].tat, p[i].wt);
        totalTAT += p[i].tat;
        totalWT  += p[i].wt;
    }
    printf("\nAverage TAT = %.2f", totalTAT / n);
    printf("\nAverage WT  = %.2f\n", totalWT / n);
}

int main() {
    int n;
    printf("Enter number of processes: ");
    scanf("%d", &n);

    struct Process p[n];
    for (int i = 0; i < n; i++) {
        p[i].pid = i + 1;
        printf("Arrival Time and Burst Time for P%d: ", i+1);
        scanf("%d %d", &p[i].at, &p[i].bt);
    }

    hrn(p, n);
    return 0;
}
```

---

**HRN Detailed Numerical Example:**

**Given:**

| Process | AT | BT |
|---|---|---|
| P1 | 0 | 3 |
| P2 | 2 | 6 |
| P3 | 4 | 4 |

**Step 1: t=0**
```
Available: P1 only
RR(P1) = (0 + 3) / 3 = 1.0
→ P1 runs: t=0 to t=3. CT(P1)=3
```

**Step 2: t=3 — Calculate RR for all available**
```
P2: Waited = 3-2 = 1ms, BT=6 → RR = (1+6)/6 = 7/6 = 1.17
P3: Not arrived yet (AT=4)

Only P2 available → P2 runs: t=3 to t=9. CT(P2)=9
```

**Step 3: t=9**
```
P3: Waited = 9-4 = 5ms, BT=4 → RR = (5+4)/4 = 9/4 = 2.25
Only P3 → P3 runs: t=9 to t=13. CT(P3)=13
```

**Gantt Chart:**
```
|P1 |P2      |P3    |
0   3        9     13
```

**Final Table:**

| Process | AT | BT | CT | TAT | WT |
|---|---|---|---|---|---|
| P1 | 0 | 3 | 3 | 3 | **0** |
| P2 | 2 | 6 | 9 | 7 | **1** |
| P3 | 4 | 4 | 13 | 9 | **5** |

```
Average TAT = (3+7+9) / 3 = 19/3 = 6.33 ms
Average WT  = (0+1+5) / 3 = 6/3  = 2.0 ms
```

> **HRN को beauty:** RR formula ले long-waiting processes लाई automatically favor गर्छ — BT बढी भए पनि waiting time बढ्दा ratio बढ्छ। No starvation guaranteed!

---

## 29. Scheduling Algorithms — Summary

### 29.1 Batch System Scheduling
- **FCFS:** Simple, no starvation, convoy effect problem.
- **SJF:** Optimal average WT, starvation possible.
- **SRTF:** Preemptive SJF, best average WT overall.
- **HRN:** No starvation, balances short and long jobs.

### 29.2 Interactive System Scheduling
- **Round Robin:** Fair, no starvation, best for time-sharing.
- **Priority:** Fast for high-priority, starvation for low-priority.
- **MLFQ:** Adaptive, modern OS use this.
- **Lottery:** Probabilistically fair, no starvation.

### 29.3 Real-Time System Scheduling
- **Hard RT (Rate Monotonic, EDF):** Deadline miss = failure. Airbag, pacemaker.
- **Soft RT (Priority-based):** Deadline miss = degradation. YouTube, Zoom.

---

## 30. Complete Comparison Table

| Algorithm | Type | Starvation | Convoy Effect | Best For | Key Formula |
|---|---|---|---|---|---|
| FCFS | Non-preemptive | No | **Yes** | Simple batch | First AT wins |
| SJF | Non-preemptive | Yes | No | Batch, min WT | Min BT wins |
| SRTF | Preemptive | Yes | No | Optimal time-sharing | Min remaining BT |
| HRN | Non-preemptive | **No** | No | Fair batch | (W+BT)/BT |
| Round Robin | Preemptive | **No** | No | Interactive | Time quantum |
| Priority | Both | Yes | No | Real-time | Min priority# |
| Lottery | Preemptive | **No** | No | Fair share | Random ticket |
| MLFQ | Preemptive | **No** | No | Modern OS | Behavior-based |

---

## 31. Exam Tips

Commonly asked questions for Unit 3:

1. **Define process. Compare process vs program.**
2. **Draw and explain the Process Life Cycle diagram.**
3. **What is PCB? List its components. What is its role?**
4. **What is process preemption? Give an example.**
5. **Explain Process Hierarchies in Unix with a diagram.**
6. **What is process implementation? What data structures does the OS use?**
7. **Explain Cooperation by Sharing vs Cooperation by Communication with examples.**
8. **What are the types of system calls? Give examples of each.**
9. **Define thread. Compare thread vs process (table).**
10. **Explain The Thread Model — single vs multi-threaded.**
11. **Compare User-Level Threads vs Kernel-Level Threads.**
12. **What is context switch? What is dispatch latency?**
13. **What is CPU-I/O Burst Cycle? Draw the diagram.**
14. **Explain Process Scheduling Queues (Job Queue, Ready Queue, Device Queue).**
15. **What is the Dispatcher? What are its functions?**
16. **Differentiate preemptive vs non-preemptive scheduling.**
17. **What is a race condition? What is a critical section?**
18. **What are the 3 requirements of a critical section solution?**
19. **Explain Peterson's Solution.**
20. **Explain semaphore with P and V operations.**
21. **Solve scheduling numericals: FCFS, SJF, SRTF, RR, Priority, HRN — always draw Gantt chart first!**
22. **Write C program for FCFS / SJF / Round Robin scheduling.**
23. **Explain Batch, Interactive, and Real-Time scheduling with examples.**
24. **Explain the Dining Philosophers problem and its solution.**

> **Key formulas for exam:**
> ```
> TAT = CT − AT
> WT  = TAT − BT
> HRN Response Ratio = (Waiting Time + BT) / BT
> Average WT = Sum of all WT / n
> ```
> **Golden Rule for Numericals:** Always draw Gantt Chart first — never calculate CT, TAT, WT without it!

---

## 32. References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12 — Chapters 2, 6
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014 — Chapters 3, 4, 5, 6