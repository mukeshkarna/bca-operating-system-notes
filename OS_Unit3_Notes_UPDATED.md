# Unit 3: Process Management
**Course:** CACS251 — Operating System | **Duration:** 15 Hours
**Year/Semester:** BCA II/IV | Tribhuvan University

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

Every method below tries to solve one problem: **stop a second process from entering the critical section while the first is still inside.**

---

**A) Disabling Interrupts**

**Idea:** The CPU only switches from one process to another when an **interrupt** arrives (usually the timer interrupt). If a process turns interrupts **off** before its critical section, nobody can take the CPU away from it. It turns interrupts back **on** when it leaves.

> **Real-world example:** You go into a room, lock the door and switch off your phone. Nobody can disturb you until you come out and switch the phone back on.

```c
void Process(void)
{
    while (true)
    {
        disable_interrupts();   // CLI → no timer interrupt, no process switch
        // critical section (safe on 1 CPU)
        enable_interrupts();    // STI → switching allowed again
        // remainder section
    }
}
```

**Problems:**
1. **Fails on multi-core CPUs.** Disabling interrupts only affects *your* CPU. A process on another core can still touch `count`.
   *Example:* You switched off your phone, but your roommate in the next room is still using the same fridge.
2. **Dangerous for user programs.** If a user program disables interrupts and forgets to enable them (bug or infinite loop), the whole system freezes. The OS never gets the CPU back.
3. **Fine for the kernel.** The OS kernel uses it for a few instructions at a time, for example while updating its ready list.

---

**B) Lock Variables**

**Idea:** Keep one shared variable `lock`.
- `lock = 0` → the critical section is free
- `lock = 1` → someone is inside

Before entering, a process checks the lock. If it is 0, the process sets it to 1 and goes in. When leaving, it sets it back to 0.

> **Real-world example:** The "VACANT / OCCUPIED" sign on a toilet door. Check the sign. If it says vacant, turn it to occupied and go in. Turn it back to vacant when you leave.

```c
int lock = 0;          // 0 = free, 1 = busy

void Process(void)
{
    while (true)
    {
        while (lock == 1);   // someone inside → keep waiting (busy wait)
        lock = 1;            // I am going in
        // critical section
        lock = 0;            // I am out
        // remainder section
    }
}
```

**Problem: a race condition on the lock itself.** Checking the lock (`while (lock == 1)`) and setting it (`lock = 1`) are **two separate steps**. A process switch can happen in between:

| Step | P1 | P2 | lock |
|------|----|----|------|
| 1 | reads lock → 0 (free!) | | 0 |
| 2 | *(switched out before setting lock = 1)* | | 0 |
| 3 | | reads lock → 0 (free!) | 0 |
| 4 | | sets lock = 1, enters CS | 1 |
| 5 | resumes, sets lock = 1, enters CS | (inside CS) | 1 |
| ❌ | **inside CS** | **inside CS** | |

Both processes are in the critical section, so mutual exclusion is broken.

> **Real-world example:** Two people look at the toilet sign at the same moment. Both see "VACANT" and both walk in.

---

**C) Turn Variable (Strict Alternation)**

**Idea:** Keep a shared variable `turn` that says **whose turn it is**.
- `turn = 0` → only P0 may enter
- `turn = 1` → only P1 may enter

When a process leaves, it hands the turn to the other process.

> **Real-world example:** Two brothers share one cycle with a strict rule: "Ram, then Shyam, then Ram, then Shyam..." Ram cannot ride twice in a row, even if Shyam is asleep and doesn't want the cycle.

```c
int turn = 0;              // P0 goes first

// ---------- Process P0 ----------
while (true)
{
    while (turn != 0);     // not my turn → wait
    // critical section
    turn = 1;              // give turn to P1
    // remainder section
}

// ---------- Process P1 ----------
while (true)
{
    while (turn != 1);     // not my turn → wait
    // critical section
    turn = 0;              // give turn to P0
    // remainder section
}
```

✅ **Mutual exclusion works.** `turn` is either 0 or 1, never both.

❌ **Progress is violated.** Here is how:
1. P0 enters the CS, finishes, and sets `turn = 1`.
2. P1 is busy with a long remainder section, or doesn't need the CS at all.
3. P0 wants to enter again, but `turn == 1`, so P0 waits even though **the CS is empty**.

A process outside the CS (P1) is blocking another process (P0). That breaks requirement 2 (Progress).

❌ Busy waiting: the `while` loop wastes CPU time.

---

**D) Peterson's Solution**

**Idea:** Combine the two earlier ideas:
- `flag[i]` → "I **want** to enter" (like the lock variable)
- `turn` → "if we both want to, **whose turn** is it?" (like strict alternation)

Each process raises its flag and then **politely gives the turn to the other**. If both want to enter at the same time, whoever gave the turn away **last** waits.

> **Real-world example:** Two people arrive at a narrow door at the same time. Each says "I want to go" (flag) and then "but you go first" (turn = other). If both say "you go first" at almost the same moment, the person who said it **last** waits, and the other goes through. If only one person is at the door, they walk straight in.

```c
#define FALSE 0
#define TRUE  1

int turn;                         // whose turn is it?
int flag[2] = {FALSE, FALSE};     // flag[i] = TRUE → Pi wants to enter

void Process(int i)               // i = 0 or 1
{
    int j = 1 - i;                // j = the other process

    while (true)
    {
        flag[i] = TRUE;           // 1. I want to enter
        turn = j;                 // 2. but you go first
        while (flag[j] == TRUE && turn == j);   // 3. wait while the other
                                                //    wants in AND it's their turn
        // critical section
        flag[i] = FALSE;          // 4. I am done, no longer interested
        // remainder section
    }
}
```

**Case 1: only P0 wants to enter**

| Step | P0 | flag[0] | flag[1] | turn |
|------|----|---------|---------|------|
| 1 | flag[0] = TRUE | T | F | – |
| 2 | turn = 1 | T | F | 1 |
| 3 | check: flag[1] is FALSE → **enter CS** ✅ | T | F | 1 |

P0 doesn't wait, because P1 isn't interested. **Progress is satisfied** (this is where strict alternation failed).

**Case 2: both want to enter at the same time**

| Step | P0 | P1 | flag[0] | flag[1] | turn |
|------|----|----|---------|---------|------|
| 1 | flag[0] = TRUE | | T | F | – |
| 2 | | flag[1] = TRUE | T | T | – |
| 3 | turn = 1 | | T | T | 1 |
| 4 | | turn = 0 (**last** to write) | T | T | **0** |
| 5 | check: flag[1]=T but turn=0 → **enter CS** ✅ | check: flag[0]=T and turn=0 → **wait** | T | T | 0 |
| 6 | leaves, flag[0] = FALSE | | F | T | 0 |
| 7 | | check: flag[0]=F → **enter CS** ✅ | F | T | 0 |

Only one process is inside at a time (**mutual exclusion**). P1 enters right after P0 leaves, so it never waits forever (**bounded waiting**).

✅ Satisfies all 3 requirements: Mutual Exclusion, Progress, Bounded Waiting.
❌ Works for **only 2 processes**.
❌ Still uses **busy waiting**.

---

**E) Test and Set Lock (TSL Instruction)**

**Idea:** The lock variable (B) failed because "check the lock" and "set the lock" were two separate steps, and a process switch could happen between them. **TSL is a hardware instruction that does both in ONE step that cannot be interrupted.** While TSL runs, the CPU also locks the memory bus, so other CPUs cannot touch the lock either.

> **Real-world example:** A shop has a single toilet key hanging on a hook. Taking the key off the hook is one action: you check whether it's free and claim it at the same moment. Two people can't both take the same key. If the hook is empty, you wait. When you finish, you hang the key back.

**What TSL does (in C, for understanding; the hardware does this ATOMICALLY):**

```c
int TestAndSet(int *lock)
{
    int old = *lock;   // read the old value
    *lock = 1;         // set lock to 1 (busy)
    return old;        // return what it was before
}
// The three lines above run as ONE uninterruptible instruction.
```

**Using it:**

```c
int lock = 0;                         // 0 = free

void Process(void)
{
    while (true)
    {
        while (TestAndSet(&lock) == 1);   // old value 1 → someone inside → spin
                                          // old value 0 → it was free, and now
                                          //               it is set to 1 for me
        // critical section
        lock = 0;                         // hang the key back
        // remainder section
    }
}
```

**The same thing in assembly:**

```assembly
enter_region:
    TSL REGISTER, LOCK   ; atomically: REGISTER = LOCK, LOCK = 1
    CMP REGISTER, #0     ; was the lock 0 (free)?
    JNE enter_region     ; no → it was busy, try again (spin)
    RET                  ; yes → enter critical section

leave_region:
    MOVE LOCK, #0        ; release lock (LOCK = 0)
    RET
```

**Trace (lock = 0 at the start):**

| Step | P1 | P2 | lock |
|------|----|----|------|
| 1 | TSL → gets old value **0**, lock becomes 1 → **enters CS** | | 1 |
| 2 | (inside CS) | TSL → gets old value **1** → spins | 1 |
| 3 | (inside CS) | TSL → gets **1** → spins | 1 |
| 4 | leaves: lock = 0 | | 0 |
| 5 | | TSL → gets **0**, lock becomes 1 → **enters CS** | 1 |

✅ Works on **multi-processor** systems.
✅ Works for **any number** of processes.
✅ Atomicity removes the race on the lock itself.
❌ **Busy waiting (spinlock):** waiting processes keep running the loop and burn CPU time.

---

#### Summary of the 5 proposals

| Method | Mutual Exclusion | Progress | Multi-core? | Main problem |
|--------|:---:|:---:|:---:|--------------|
| Disabling Interrupts | ✅ (1 CPU) | ✅ | ❌ | Unsafe for user programs |
| Lock Variable | ❌ | ✅ | ❌ | Race on the lock itself |
| Strict Alternation | ✅ | ❌ | ✅ | Must take turns even if other isn't interested |
| Peterson's | ✅ | ✅ | ✅* | Only 2 processes, busy waiting |
| TSL | ✅ | ✅ | ✅ | Busy waiting |

\* on hardware that doesn't reorder memory operations.

**The common problem is busy waiting.** Methods B to E all wait with a loop like `while (...);`. The waiting process keeps using the CPU just to ask "can I go in now?"

> **Real-world example:** Busy waiting is like standing at a doctor's door and knocking every second. A better system is a waiting room: you sit down (**sleep**) and the nurse calls your name when it's your turn (**wakeup**). **Semaphores** work this way.

---

## 19. Types of Mutual Exclusion Mechanisms

### 19.1 Semaphore

A **semaphore** is an **integer variable** plus a **list (queue) of sleeping processes**. It can only be used through two operations, and the OS guarantees each one runs **atomically** (it can't be interrupted halfway):

| Operation | Other names | What it means |
|-----------|-------------|---------------|
| `Down(S)` | `P(S)`, `wait(S)` | "I want to use a resource." Take one, or sleep if none are left. |
| `Up(S)` | `V(S)`, `signal(S)` | "I'm done with the resource." Give it back and wake a sleeper if there is one. |

> **Real-world example (parking lot):** A parking lot has a board at the gate: **"FREE SPACES: 3"**.
> - A car arrives → the board goes down by 1 → if a space was available, the car enters. If not, the car waits in a queue at the gate.
> - A car leaves → the board goes up by 1 → if cars are waiting in the queue, the guard lets the first one in.
>
> The board is the **semaphore value**, a car arriving is **Down()**, a car leaving is **Up()**, and the queue at the gate is the **suspended list**.

**Implementation (no busy waiting: processes sleep instead of looping):**

```c
struct Semaphore
{
    int   value;        // number of free resources
    Queue L;            // suspended list (PCBs of sleeping processes)
};

void Down(Semaphore S)
{
    S.value = S.value - 1;
    if (S.value < 0)
    {
        put process (PCB) in suspended list S.L;
        sleep();                       // block, don't waste CPU
    }
    else
        return;                        // resource available → go ahead
}

void Up(Semaphore S)
{
    S.value = S.value + 1;
    if (S.value <= 0)                  // someone is still waiting
    {
        select a process P from suspended list S.L;
        wakeup(P);                     // move P to ready queue
    }
}
```

**How to read the value:**

| S.value | Meaning |
|---------|---------|
| **+3** | 3 resources free, nobody waiting |
| **0** | no resources free, nobody waiting |
| **−2** | no resources free, **2 processes sleeping** in the list |

> **Why `<= 0` in Up()?** If the value is still 0 or negative *after* adding 1, it was negative before, so at least one process is sleeping. Wake one up.

**Trace: counting semaphore, S = 3 (3 printers), 5 processes:**

| Step | Action | S.value | Result | Suspended list |
|------|--------|---------|--------|----------------|
| 0 | start | 3 | | – |
| 1 | P1: Down | 2 | gets printer | – |
| 2 | P2: Down | 1 | gets printer | – |
| 3 | P3: Down | 0 | gets printer | – |
| 4 | P4: Down | −1 | **sleeps** | P4 |
| 5 | P5: Down | −2 | **sleeps** | P4, P5 |
| 6 | P1: Up | −1 | wakes **P4** → P4 gets printer | P5 |
| 7 | P2: Up | 0 | wakes **P5** → P5 gets printer | – |
| 8 | P3: Up | 1 | nobody waiting | – |

#### Types of Semaphore

**1. Binary Semaphore (Mutex):** the value is only **0 or 1**. It is used for **mutual exclusion**, so only one process is in the CS.

> **Real-world example:** A restaurant has one washroom and **one key** at the counter. Take the key (Down) → use the washroom → return the key (Up). If the key is gone, you wait.

```c
Semaphore mutex = 1;          // 1 = key available

void Process(void)
{
    while (true)
    {
        Down(mutex);          // take the key (1 → 0); if 0 → sleep
        // critical section
        Up(mutex);            // return the key; wake a waiter if any
        // remainder section
    }
}
```

| Step | P1 | P2 | mutex |
|------|----|----|-------|
| 1 | Down → 0, enters CS | | 0 |
| 2 | (inside CS) | Down → −1, **sleeps** | −1 |
| 3 | Up → 0, wakes P2 | | 0 |
| 4 | | enters CS | 0 |
| 5 | | Up → 1 | 1 |

**2. Counting Semaphore:** the value can be **any non-negative number** (when initialized). It is used when there are **N identical resources**.

> **Real-world example:** A cyber café has 10 computers, so S = 10. The 11th customer waits until someone leaves.

```c
Semaphore computers = 10;

void Customer(void)
{
    Down(computers);     // take a computer (or wait if all 10 busy)
    use_computer();
    Up(computers);       // free the computer
}
```

**Common mistakes with semaphores** (the reason monitors were invented):
- Forgetting `Up()` → the resource is never released → everyone waits forever.
- Writing `Up()` before `Down()` → two processes can enter the CS.
- Calling `Down()` on two semaphores in the wrong order → **deadlock** (see 19.4).

---

### 19.2 Monitors

**Problem with semaphores:** The programmer has to put every `Down()` and `Up()` in exactly the right place. One mistake anywhere in the program breaks everything, and this kind of bug is very hard to find.

**Idea of a monitor:** Put the shared data and the functions that use it **inside one box (module)**. The **compiler** makes sure that **only one process can be running inside the box at a time**. The programmer doesn't write any locking code; mutual exclusion is automatic.

> **Real-world example:** An ATM room with a guard at the door. Only **one person** is allowed inside at a time, and the guard enforces this automatically. You don't lock the door yourself.
> If the ATM has no cash, you sit on a **waiting bench** inside (a **condition variable**). You step away from the machine so someone else can use it. When the cash is refilled, the guard tells you to come back.

**Structure of a monitor:**

```c
monitor MonitorName
{
    // shared variables (can only be touched from inside)

    // condition variables
    condition x, y;

    procedure P1(...) { ... }
    procedure P2(...) { ... }

    initialization_code() { ... }
}
```

**Condition variables:** used when a process is inside the monitor but must **wait for something** (for example, "buffer is full"):

| Operation | What happens |
|-----------|--------------|
| `x.wait()` | The process **sleeps** on condition x **and leaves the monitor**, so another process can enter. |
| `x.signal()` | Wakes up **one** process sleeping on x. If nobody is sleeping, **nothing happens**. |

> ⚠️ **Difference from a semaphore:** `Up(S)` is **remembered** because the value increases. `x.signal()` with no waiter is **lost**. It's like a guard calling a name when nobody is on the bench: the call is wasted.

**Example: Bounded Buffer using a monitor:**

```c
monitor BoundedBuffer
{
    int buffer[N];
    int count = 0, in = 0, out = 0;
    condition notFull, notEmpty;

    void insert(int item)                // called by producer
    {
        if (count == N)
            notFull.wait();              // buffer full → sleep on bench
        buffer[in] = item;
        in = (in + 1) % N;
        count++;                         // safe! only one process inside
        notEmpty.signal();               // wake a waiting consumer
    }

    int remove(void)                     // called by consumer
    {
        if (count == 0)
            notEmpty.wait();             // buffer empty → sleep on bench
        int item = buffer[out];
        out = (out + 1) % N;
        count--;
        notFull.signal();                // wake a waiting producer
        return item;
    }
}

// Producer and consumer just call the monitor's functions — no locks!
void Producer(void) { while (true) { item = produce_item(); BoundedBuffer.insert(item); } }
void Consumer(void) { while (true) { item = BoundedBuffer.remove(); consume_item(item); } }
```

Here `count++` is safe without any `Down()`/`Up()`, because the monitor allows only one process inside at a time.

> **Real-world example:** Java's `synchronized` keyword works like a monitor.

---

### 19.3 Message Passing

**Idea:** Processes **don't share any memory**. They communicate by **sending and receiving messages** through the OS kernel. Since nothing is shared, there is no critical section to protect.

> **Real-world example:** Two friends in different cities can't write in the same notebook. They send each other **letters by post** (or WhatsApp messages). The post office (kernel) carries the message.

**Two basic operations:**

```c
send(destination, &message);    // send a message to a process
receive(source, &message);      // wait for a message from a process
```

**Types:**

| Type | Meaning | Real-world example |
|------|---------|--------------------|
| **Direct** | Name the process: `send(P2, m)` | Handing a letter to a person |
| **Indirect (Mailbox)** | Send to a mailbox: `send(box1, m)` | Dropping a letter in a P.O. box |
| **Blocking (synchronous)** | Sender waits until the message is received | Phone call: both must be present |
| **Non-blocking (asynchronous)** | Sender sends and continues working | SMS: send and move on |

✅ No shared memory → no race conditions.
✅ Works across different computers (**distributed systems**).
✅ Used in microkernel OSes (**QNX, MINIX**).
❌ Slower: every message goes through the kernel (copying + system calls).

---

### 19.4 Producer-Consumer Problem (Bounded Buffer)

**The problem:**
- A **producer** makes items and puts them into a buffer with **N slots**.
- A **consumer** takes items out of the buffer.
- The producer must **wait if the buffer is full**.
- The consumer must **wait if the buffer is empty**.
- Only **one** of them may touch the buffer at a time.

> **Real-world example:** A bakery with a shelf of **5 slots**. The **baker** (producer) puts bread on the shelf. The **customer** (consumer) takes bread.
> - Shelf full → the baker waits.
> - Shelf empty → the customer waits.
> - Only one person at the shelf at a time, so they don't bump into each other.

**Without synchronization (wrong):** producer and consumer share a variable `count`:

```c
int count = 0;

void Producer(void)                    
{                                      
    int itemp;                              
    while (true)                           
    {                                       
        produce_item(itemp);          
        while (count == n);  // full            
        Buffer[in] = itemp;                   
        in = (in + 1) mod n;                    
        count = count + 1;   // ← race!         
    }                                      
} 

void Consumer(void)
{
    int itemc;
    while (true)
    {
        while (count == 0);   // empty → wait
        itemc = Buffer[out];
        out = (out + 1) mod n;
        count = count - 1;    // ← race!
        consume_item(itemc);
    }
}
```

**Why is this wrong?** `count = count + 1` looks like **one** line, but the CPU runs it as **three** instructions:

```assembly
LOAD  Rp, m[count]     ; 1. copy count from memory into register Rp
INCR  Rp               ; 2. add 1 to the register
STORE m[count], Rp     ; 3. write the register back to memory
```

The consumer does the same thing with `count = count - 1` (LOAD Rc, DECR Rc, STORE).

**What goes wrong (count = 5 at the start):**

| Step | Producer | Consumer | Rp | Rc | count in memory |
|------|----------|----------|----|----|-----------------|
| 1 | LOAD Rp, m[count] | | 5 | | 5 |
| 2 | INCR Rp | | 6 | | 5 |
| 3 | *(timer interrupt → switch)* | LOAD Rc, m[count] | 6 | 5 | 5 |
| 4 | | DECR Rc | 6 | 4 | 5 |
| 5 | | STORE m[count], Rc | 6 | 4 | **4** |
| 6 | STORE m[count], Rp | | 6 | 4 | **6** ❌ |

One item was added and one was removed, so count should still be **5**. It ended up as **6**. With a different interleaving it ends up as **4**.

> **Real-world example:** Two cashiers update the same stock register. Both read "5 pens", one sells a pen and writes "4", the other receives a pen and writes "6". Whoever writes last wins, and the register is wrong.

Both loops also busy wait (`while (count == n);`), wasting CPU time.

**Correct solution with 3 semaphores:**

| Semaphore | Initial value | Job | Bakery meaning |
|-----------|---------------|-----|----------------|
| `mutex` | 1 | Only one process touches the buffer at a time | One person at the shelf |
| `empty` | N | Counts **empty** slots | Free places on the shelf |
| `full` | 0 | Counts **filled** slots | Breads on the shelf |

```c
#define N 5
Semaphore mutex = 1;    // buffer lock
Semaphore empty = N;    // empty slots (all empty at start)
Semaphore full  = 0;    // filled slots (none at start)

int buffer[N];
int in = 0, out = 0;

void Producer(void)
{
    int item;
    while (true)
    {
        item = produce_item();     // bake bread
        Down(empty);               // wait for a free slot   (empty − 1)
        Down(mutex);               // lock the shelf
        buffer[in] = item;         // put bread on shelf
        in = (in + 1) % N;
        Up(mutex);                 // unlock the shelf
        Up(full);                  // one more bread         (full + 1)
    }
}

void Consumer(void)
{
    int item;
    while (true)
    {
        Down(full);                // wait for a bread       (full − 1)
        Down(mutex);               // lock the shelf
        item = buffer[out];        // take bread
        out = (out + 1) % N;
        Up(mutex);                 // unlock the shelf
        Up(empty);                 // one more free slot     (empty + 1)
        consume_item(item);        // eat bread
    }
}
```

**Trace (N = 2, to keep it short):**

| Step | Action | empty | full | mutex | Result |
|------|--------|-------|------|-------|--------|
| 0 | start | 2 | 0 | 1 | |
| 1 | Consumer: Down(full) | 2 | **−1** | 1 | shelf empty → consumer **sleeps** |
| 2 | Producer: Down(empty), Down(mutex), put item, Up(mutex) | 1 | −1 | 1 | item 1 on shelf |
| 3 | Producer: Up(full) | 1 | 0 | 1 | **wakes consumer** |
| 4 | Producer: puts item 2 | 0 | 1 | 1 | shelf now full |
| 5 | Producer: Down(empty) | **−1** | 1 | 1 | shelf full → producer **sleeps** |
| 6 | Consumer takes item, Up(empty) | 0 | 1 | 1 | **wakes producer** |

> ⚠️ **The order of Down() matters!** Suppose the producer is written as `Down(mutex); Down(empty);` (swapped):
> 1. The buffer is full, so `empty = 0`.
> 2. The producer does `Down(mutex)` → locks the buffer.
> 3. The producer does `Down(empty)` → sleeps **while holding the lock**.
> 4. The consumer does `Down(full)` → OK, then `Down(mutex)` → sleeps, because the producer has the lock.
> 5. Each is waiting for the other → **DEADLOCK**.
>
> *Real-world example:* The baker locks the shelf, then falls asleep waiting for space. The customer can't get to the shelf to make space. **Always wait for the resource (empty/full) first, then lock (mutex).**

---

## 20. Serializability, Locking & Time Stamp Protocols

These ideas come from **databases**, where many **transactions** (groups of read/write operations) run at the same time on shared data.

> **Real-world example:** A bank account is the shared data. A deposit and a withdrawal are two transactions that may happen at the same moment from two ATMs.

### 20.1 Serializability

- **Serial schedule:** transactions run **one after another** (T1 fully, then T2). Always correct, but slow.
- **Concurrent schedule:** operations of T1 and T2 are **mixed (interleaved)**. Faster, but can give wrong results.
- **Serializable schedule:** a concurrent schedule whose **final result is the same as some serial schedule**. This is how we decide that a concurrent schedule is correct.

**Example:** Account A = Rs. 1000. T1 deposits 100, T2 withdraws 50.
Any serial order gives **1000 + 100 − 50 = 1050**.

| Schedule | Steps | Final A | Serializable? |
|----------|-------|---------|---------------|
| Serial T1 → T2 | T1: read 1000, write 1100. T2: read 1100, write 1050 | **1050** | ✅ |
| Serial T2 → T1 | T2: read 1000, write 950. T1: read 950, write 1050 | **1050** | ✅ |
| Bad interleaving | T1 reads 1000 → T2 reads 1000 → T1 writes 1100 → T2 writes 950 | **950** ❌ | ❌ (T1's deposit is lost) |

The bad schedule doesn't match any serial order, so it is **not serializable**. This "lost update" is the same problem as the `count = count + 1` race.

**Conflicting operations:** Two operations **conflict** if all of these are true:
1. They belong to **different** transactions,
2. They access the **same** data item, and
3. **At least one** is a **write**.

| | Read(A) | Write(A) |
|---|---|---|
| **Read(A)** | No conflict | Conflict |
| **Write(A)** | Conflict | Conflict |

Two reads never conflict, because reading doesn't change anything.

### 20.2 Locking Protocols

A transaction must **lock** a data item before using it.

| Lock | Used for | Can others also lock? | Real-world example |
|------|----------|-----------------------|--------------------|
| **Shared (S) lock** | Reading | Others can take S, **not X** | Many students can **read** the notice board together |
| **Exclusive (X) lock** | Writing | **No one** else can lock | Only one person can **edit** the notice; others must wait |

**Compatibility matrix:**

| Held → / Requested ↓ | S | X |
|---|---|---|
| **S** | ✅ Yes | ❌ No |
| **X** | ❌ No | ❌ No |

#### Two-Phase Locking (2PL)

Every transaction works in **two phases**:
1. **Growing phase:** it may **acquire** locks but **cannot release** any.
2. **Shrinking phase:** it may **release** locks but **cannot acquire** any new one.

Once a transaction releases its **first** lock, it can **never** take a new lock. The moment it holds all its locks is called the **lock point**.

> **Real-world example (shopping):** In a supermarket you first **collect** all the items you need into your cart (growing). Then you go to the counter and **hand them over** one by one (shrinking). Once checkout has started, you can't run back for more items.

```
T1: Transfer Rs. 100 from A to B (follows 2PL ✅)

lock-X(A);                  ┐
lock-X(B);                  ┘ Growing phase   ← lock point here
read(A);  A = A - 100;  write(A);
read(B);  B = B + 100;  write(B);
unlock(A);                  ┐
unlock(B);                  ┘ Shrinking phase
```

```
T2: does NOT follow 2PL ❌

lock-X(A);
read(A);  A = A - 100;  write(A);
unlock(A);                  ← started shrinking...
lock-X(B);                  ← ...then took a new lock → violates 2PL
read(B);  B = B + 100;  write(B);
unlock(B);
```

In T2, between `unlock(A)` and `lock-X(B)` another transaction can read A (already reduced) and B (not yet increased). It sees Rs. 100 "missing". 2PL prevents this.

✅ 2PL **guarantees serializability**.
❌ 2PL can still cause **deadlock**: T1 holds A and wants B, while T2 holds B and wants A.
- **Strict 2PL** (used in practice): hold all **X locks until the transaction commits**. This also avoids "cascading rollback".

### 20.3 Time Stamp Protocols

**Idea:** No locks. Each transaction gets a **timestamp (TS)** when it starts, like a **token number**. A smaller timestamp means an **older** transaction, and the **older one has priority**. The system makes sure conflicting operations run **in timestamp order**. If a transaction tries to break that order, it is **rolled back** and restarted with a new timestamp.

> **Real-world example:** The token system at a bank or hospital. Token #10 (older) must be served before token #20 (newer). If #20 has already done something that #10 was supposed to do first, #20's work doesn't count. It is sent back to take a new token.

Each data item Q stores two values:
- **R-TS(Q):** the largest timestamp of any transaction that **read** Q
- **W-TS(Q):** the largest timestamp of any transaction that **wrote** Q

**Rules:** transaction T with timestamp TS(T):

```
Read(Q):
    if TS(T) < W-TS(Q)          // a YOUNGER transaction already wrote Q
        reject → rollback T     // T is too late to read the old value
    else
        allow read;  R-TS(Q) = max(R-TS(Q), TS(T))

Write(Q):
    if TS(T) < R-TS(Q)          // a YOUNGER transaction already read Q
       or TS(T) < W-TS(Q)       // a YOUNGER transaction already wrote Q
        reject → rollback T     // T's write is too late
    else
        allow write;  W-TS(Q) = TS(T)
```

**Example:** T1 has TS = 10 (older), T2 has TS = 20 (newer). At the start, R-TS(A) = W-TS(A) = 0.

| Step | Operation | Check | Result | R-TS(A) | W-TS(A) |
|------|-----------|-------|--------|---------|---------|
| 1 | T1: Read(A) | 10 < W-TS 0? No | ✅ allowed | 10 | 0 |
| 2 | T2: Read(A) | 20 < W-TS 0? No | ✅ allowed | 20 | 0 |
| 3 | T2: Write(A) | 20 < R-TS 20? No. 20 < W-TS 0? No | ✅ allowed | 20 | 20 |
| 4 | T1: Write(A) | 10 < R-TS 20? **Yes** | ❌ **T1 rolled back** | 20 | 20 |

At step 4, T1 (older) tries to write A after T2 (younger) has already read and written it. Allowing it would break the timestamp order, so T1 restarts with a new timestamp.

✅ **No deadlock**, because nobody ever waits for a lock.
✅ Guarantees serializability.
❌ Many **rollbacks** when there are lots of conflicts. A long transaction may restart again and again (**starvation**).

### 20.4 2PL vs Timestamp: Quick Comparison

| Point | Two-Phase Locking | Timestamp Ordering |
|-------|-------------------|--------------------|
| Uses locks? | Yes | No |
| On conflict | Transaction **waits** | Transaction is **rolled back** |
| Deadlock possible? | Yes | No |
| Order decided by | Order of acquiring locks | Start time (timestamp) |
| Real-world example | Waiting in line for the key | Token number at the bank |

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