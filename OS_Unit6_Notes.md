# Unit 6: Input / Output Device Management
**Course:** CACS251 — Operating System | **Duration:** 4 Hours
**Year/Semester:** BCA II/IV | Tribhuvan University

---

## Syllabus Topics
Principle of I/O Hardware: I/O Devices, Device Controllers, Memory Mapped I/O, Direct Memory Access; Principle of I/O Software: Goals of I/O Software, Program I/O, Interrupt-Driven I/O, I/O Using DMA; I/O Software Layers: Interrupts Handler, Device Drivers, Device Independent I/O Software, User-Space I/O Software; Disk: Disk Hardware, Disk Scheduling — Seek Time, Rational Delay, Transfer Time; Disk Scheduling Algorithms: FCFS, SSTF, SCAN, C-SCAN, Lock Scheduling.

---

## 1. Introduction to I/O Device Management

**I/O Device Management** is the OS subsystem responsible for controlling all input/output devices connected to the computer.

**Goals:**
- Provide a uniform, device-independent interface to applications.
- Handle device differences transparently.
- Maximize device and CPU utilization.
- Ensure error handling and recovery.
- Support concurrent I/O operations.

> **Real-world example:** When you plug in a USB keyboard, USB drive, and webcam simultaneously, the OS manages all three through its I/O management subsystem — each device gets its driver, interrupts are handled independently, and applications never deal with hardware directly.

---

## 2. Classification of I/O Devices

### 2.1 Block Devices
- Transfer data in **fixed-size blocks** (e.g., 512 bytes, 4KB).
- Support random access — any block can be read/written independently.
- **Examples:** Hard disk (HDD), SSD, USB drive, CD-ROM.

```
Block Device:
┌────┬────┬────┬────┬────┬────┐
│ B0 │ B1 │ B2 │ B3 │ B4 │ B5 │  ← Each block independently accessible
└────┴────┴────┴────┴────┴────┘
```

### 2.2 Character Devices
- Transfer data as a **stream of characters** (byte by byte).
- No block structure — **sequential access** only.
- **Examples:** Keyboard, mouse, serial port, terminal, printer.

```
Character Device:
→ h → e → l → l → o → (stream, no random access)
```

### 2.3 Communication Devices
- Handle data communication between systems.
- **Examples:** Network interface card (NIC), modem, Bluetooth adapter.

**Comparison:**

| Feature | Block Device | Character Device | Communication Device |
|---|---|---|---|
| Data unit | Fixed blocks | Byte stream | Packets/frames |
| Access | Random | Sequential | Network protocol |
| Buffering | Yes | Minimal | Protocol buffers |
| Example | HDD, SSD | Keyboard | NIC, Modem |

---

## 3. Device Controllers

A **device controller** is an electronic component (chip/circuit board) that:
- Acts as an **interface between the CPU and the physical device**.
- Contains control registers, status registers, and data buffers.
- Handles low-level device-specific operations.

```
CPU ──── System Bus ──── Device Controller ──── Physical Device
              │                 │
         (high-level)    (low-level control)
         read/write       motor control,
         commands         signal timing, etc.
```

**Controller Registers:**
- **Control Register:** CPU writes commands here (e.g., "start motor", "read sector 5").
- **Status Register:** CPU reads device status (busy, ready, error).
- **Data Register:** Data buffer for transferring bytes between device and RAM.

> **Real-world example:** Your keyboard controller reads key presses, converts them to scan codes, and signals the CPU. The CPU never directly talks to the keyboard switches — only to the controller.

---

## 4. Communication of CPU to I/O Devices

The CPU must communicate with device controllers. Several methods exist:

### 4.1 Special Instruction I/O (Port-Mapped I/O)
- CPU has **dedicated I/O instructions** (IN, OUT in x86).
- Each device controller has a unique **I/O port number**.
- CPU uses `IN port, register` and `OUT port, data` instructions.
- I/O address space is **separate** from memory address space.

```
CPU Instruction: OUT 0x60, AL    (write byte in AL to port 0x60 — keyboard)
CPU Instruction: IN  AL, 0x60   (read byte from port 0x60 into AL)
```

### 4.2 Memory Mapped I/O
- Device controller registers are **mapped to memory addresses**.
- CPU uses **ordinary load/store instructions** to communicate with devices.
- No special I/O instructions needed.
- Device registers appear as regular memory locations.

```
Memory Map:
0x0000 — 0xBFFF : RAM
0xC000 — 0xC00F : Video Controller registers  ← Device mapped here
0xC010 — 0xC01F : Keyboard Controller registers
...
```

**Advantages:** Standard instructions work for I/O, compilers can optimize.

**Disadvantages:** Reduces available memory address space.

### 4.3 Using I/O Port
- A **port** is a connection point through which data flows.
- Each device has one or more I/O port addresses.
- The OS maps device operations to specific port numbers.

### 4.4 Hybrid System
- Combines port-mapped I/O and memory-mapped I/O.
- Control registers use ports; data buffers use memory mapping.
- **Example:** x86 architecture — control via ports, video framebuffer via memory map.

### 4.5 Direct Memory Access (DMA)

**Problem without DMA:** CPU must copy every byte from device to RAM (wastes CPU time).

**DMA Solution:** A **DMA controller** transfers data directly between device and RAM, **without CPU involvement**.

```
Without DMA:                    With DMA:
CPU ←→ Device                   CPU programs DMA controller
(byte by byte, CPU busy)        DMA Controller ←→ Device ←→ RAM
                                CPU free to do other work!
                                DMA interrupts CPU when done.
```

**DMA Transfer Process:**
```
1. CPU programs DMA controller:
   - Source: Device (e.g., disk sector)
   - Destination: RAM address (e.g., 0x5000)
   - Count: Number of bytes (e.g., 512)
   - Direction: Device → RAM (read) or RAM → Device (write)

2. DMA controller takes over the system bus

3. DMA transfers data directly: Device ↔ RAM
   (CPU continues executing other instructions)

4. DMA sends interrupt to CPU when transfer complete

5. CPU processes the data in RAM
```

**DMA Types:**
- **Cycle Stealing:** DMA steals one bus cycle at a time to transfer one word. CPU and DMA alternate.
- **Burst Mode:** DMA takes complete control of bus until entire transfer done. CPU must wait.

> **Real-world example:** When you copy a large file to USB drive, DMA allows the transfer to happen while CPU continues running your browser or music player. Without DMA, your computer would freeze during every file transfer.

---

## 5. Interrupts

An **interrupt** is a signal sent to the CPU by a device or software indicating that an event needs immediate attention.

### 5.1 Working Mechanism of Interrupts

```
Step 1: Device completes operation (e.g., key pressed, data received)

Step 2: Device controller raises interrupt signal on interrupt line

Step 3: CPU finishes current instruction, checks interrupt line

Step 4: CPU saves current state (PC, registers) onto the stack

Step 5: CPU looks up Interrupt Vector Table to find handler address

Step 6: CPU jumps to Interrupt Service Routine (ISR) / Interrupt Handler

Step 7: ISR handles the event (e.g., reads key from keyboard buffer)

Step 8: ISR restores saved state, returns to interrupted program

Step 9: Interrupted program continues exactly where it left off
```

**Interrupt Vector Table:**
```
Address 0: Timer interrupt handler address
Address 1: Keyboard interrupt handler address
Address 2: Disk interrupt handler address
Address 3: Network interrupt handler address
...
```

**Types of Interrupts:**
- **Hardware Interrupt:** Signal from I/O device (keyboard, disk, NIC).
- **Software Interrupt (Trap):** Generated by program (system call, division by zero).
- **Non-Maskable Interrupt (NMI):** Cannot be disabled — critical hardware failures.
- **Maskable Interrupt:** Can be temporarily disabled by CPU (for critical sections).

> **Real-world example:** You are watching a movie on your laptop. The keyboard controller sends an interrupt when you press Pause. The CPU pauses the movie player momentarily, processes the keypress interrupt (loads the scan code), then resumes the movie player. This all happens in microseconds — invisibly to you.

---

## 6. Principles of I/O Software

### 6.1 Goals of I/O Software

**Device Independence:**
- Programs should work with any I/O device without modification.
- `read()` should work identically for file, keyboard, or network socket.

**Uniform Naming:**
- Device names should be consistent (e.g., `/dev/sda` in Linux).
- No need to know hardware specifics.

**Error Handling:**
- Errors should be handled close to the hardware.
- Higher layers notified only of unrecoverable errors.

**Synchronous vs Asynchronous I/O:**
- **Synchronous (Blocking):** Process waits for I/O to complete before continuing.
- **Asynchronous (Non-blocking):** Process continues; notified when I/O completes.

**Buffering:**
- Temporary storage of data during transfer.
- Handles speed mismatch between fast CPU and slow devices.

**Sharing vs Dedicated:**
- Some devices shared (disk — multiple processes), some dedicated (tape drive — one at a time).

---

## 7. I/O Control Techniques

### 7.1 Programmed I/O (Polling / Busy Waiting)

CPU directly controls I/O and **continuously polls** (checks) the device status register until done.

```c
// Polling example:
void write_byte(char c) {
    while (status_register & BUSY);   // Wait until device ready
    data_register = c;                // Write data
    control_register = WRITE;         // Issue write command
}
```

**Advantages:** Simple, no interrupt overhead.
**Disadvantages:** CPU wastes time busy-waiting — cannot do other work. Very inefficient for slow devices.

### 7.2 Interrupt-Driven I/O

CPU **initiates I/O and continues** doing other work. Device interrupts CPU when operation completes.

```
1. CPU issues I/O command to device controller
2. CPU continues executing other processes (no waiting!)
3. Device completes operation → sends interrupt
4. Interrupt handler saves CPU state, processes data
5. Interrupted process resumes
```

**Advantages:** CPU is free during I/O — much better utilization.
**Disadvantages:** Interrupt overhead; each byte may cause an interrupt (for character devices).

### 7.3 I/O Using DMA

Already described in Section 4.5. CPU programs DMA controller; entire block transferred without CPU.

**Comparison:**

| Method | CPU Involvement | Efficiency | Best For |
|---|---|---|---|
| Programmed I/O | High (busy wait) | Very Low | Simple/slow devices |
| Interrupt-Driven | Low (per interrupt) | Medium | Character devices |
| DMA | Minimal (setup only) | High | Block transfers (disk) |

---

## 8. I/O Software Layers

I/O software is organized in **four layers**, each providing services to the layer above:

```
┌─────────────────────────────────────────┐
│       User-Space I/O Software           │  Layer 4: Libraries, spoolers
├─────────────────────────────────────────┤
│  Device-Independent I/O Software (OS)   │  Layer 3: Buffering, error handling
├─────────────────────────────────────────┤
│           Device Drivers                │  Layer 2: Device-specific code
├─────────────────────────────────────────┤
│         Interrupt Handlers              │  Layer 1: Hardware interrupts
├─────────────────────────────────────────┤
│             Hardware                    │  Physical devices
└─────────────────────────────────────────┘
```

### 8.1 Layer 1: Interrupt Handlers
- Lowest software level — directly handles hardware interrupts.
- Saves CPU state, identifies interrupt source, services the interrupt.
- Signals waiting driver when I/O is complete.
- Should be as short as possible.

```
Interrupt occurs → Save registers → Identify device
→ Call device-specific handler → Restore registers → Return
```

### 8.2 Layer 2: Device Drivers
- **Device-specific** code that knows how to control a particular device.
- Translates generic OS commands (read, write) into device-specific commands.
- Each device type needs its own driver (printer driver, disk driver, NIC driver).
- Usually written by device manufacturer.

**Functions of a device driver:**
- Accept abstract read/write requests from OS.
- Initialize the device.
- Manage I/O queuing for the device.
- Handle device-specific errors.

> **Real-world example:** When you install a new graphics card (GPU), you install its driver. The driver tells the OS exactly how to communicate with that specific GPU model. Without the driver, the OS only knows generic VGA commands — low resolution, no acceleration.

### 8.3 Layer 3: Device-Independent I/O Software

Provides **uniform interface** across all devices. Handles:

- **Uniform interfacing for drivers:** Standardizes how drivers communicate with the OS.
- **Buffering:** Temporary storage to handle speed mismatch.
- **Error reporting:** Standardizes error messages.
- **Allocating and releasing dedicated devices:** Managing exclusive device access.
- **Device-independent block size:** Hides differences in sector/block sizes.

**Buffering types:**
- **No buffering:** Data goes directly to user process (simple but slow).
- **Buffering in kernel space:** OS buffers data, copies to user on request.
- **Double buffering:** Two buffers alternate — while one is being filled, other is being processed.

### 8.4 Layer 4: User-Space I/O Software

I/O libraries and spooler programs running in user space:

- **I/O Libraries:** Standard library functions like `printf()`, `scanf()`, `fread()`, `fwrite()` in C.
- **Spooling:** Managing dedicated device access for multiple users.
  - **Example:** Print spooler — multiple users send print jobs, spooler queues them and sends one at a time to printer.

```
User calls printf()
    → C library buffers output
    → Calls write() system call
    → Device-independent I/O (kernel buffers)
    → Printer driver
    → Interrupt handler
    → Printer hardware
```

---

## 9. Disk Structure

### 9.1 Disk Hardware

A **hard disk** consists of:

(Disk Structure)[https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter10/10_01_DiskMechanism.jpg]

**Key components:**
- **Platter:** Circular magnetic disk. Multiple platters stack on spindle.
- **Track:** Concentric circle on a platter surface.
- **Sector:** Arc-shaped segment of a track. Typically 512 bytes or 4KB.
- **Cylinder:** All tracks at the same radius across all platters.
- **Read/Write Head:** Reads/writes data. One head per platter surface.
- **Actuator Arm:** Moves the head across tracks.
- **Spindle:** Motor that rotates platters (typically 5400–15000 RPM).

**Disk Address:** (Cylinder, Head/Track, Sector) — CHS addressing.
**Modern:** Logical Block Addressing (LBA) — sectors numbered sequentially 0, 1, 2...

### 9.2 Disk Access Time

Total time to access data = **Seek Time + Rotational Latency + Transfer Time**

**Seek Time:**
- Time for the read/write head to move to the correct track (cylinder).
- Depends on how far the head must travel.
- Typical: 3–15 ms for HDD.
- **This is the dominant factor — disk scheduling aims to minimize it.**

**Rotational Latency (Rational Delay):**
- Time waiting for the desired sector to rotate under the head.
- Depends on rotation speed.
- Average = half a rotation.
- At 7200 RPM: one rotation = 60/7200 = 8.33ms; average latency = 4.17ms.

```
Rotational Latency (avg) = (60 / RPM) / 2  seconds
```

**Transfer Time:**
- Time to transfer data once head is positioned.
- Depends on data size and rotation speed.
- Typically 0.1–1 ms — usually the smallest component.

```
Transfer Time = (bytes to transfer) / (bytes per track × RPM / 60)
```

**Example:**
```
Seek time = 5ms (average)
Rotational latency = 4ms (7200 RPM)
Transfer time = 0.5ms (small block)
Total = 5 + 4 + 0.5 = 9.5ms
```

---

## 10. Disk Scheduling Algorithms

The OS maintains a **disk request queue** — multiple processes waiting to read/write different disk sectors. The disk scheduling algorithm decides the **order** in which requests are serviced.

**Goal:** Minimize total seek time (minimize total head movement).

**Disk request queue example used throughout:**
```
Current head position: 53
Queue (cylinder requests): 98, 183, 37, 122, 14, 124, 65, 67
```

---

### 10.1 FCFS (First Come First Served)

**Rule:** Requests serviced in the **order they arrive** in the queue.

```
Head starts at: 53
Order: 53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67

Movements:
53→98:   |98-53|   = 45
98→183:  |183-98|  = 85
183→37:  |183-37|  = 146
37→122:  |122-37|  = 85
122→14:  |122-14|  = 108
14→124:  |124-14|  = 110
124→65:  |124-65|  = 59
65→67:   |67-65|   = 2

Total head movement = 45+85+146+85+108+110+59+2 = 640 cylinders
```

**Advantages:** Simple, fair — no starvation.
**Disadvantages:** Large head movement, poor performance.

---

### 10.2 SSTF (Shortest Seek Time First)

**Rule:** Service the request **closest to the current head position** first.

```
Head starts at: 53
Queue: 98, 183, 37, 122, 14, 124, 65, 67

Step 1: At 53. Distances: 98→45, 183→130, 37→16, 122→69, 14→39, 124→71, 65→12, 67→14
        Closest: 65 (distance 12) → Go to 65

Step 2: At 65. Distances: 67→2, 37→28, 14→51, 98→33, 122→57, 124→59, 183→118
        Closest: 67 (distance 2) → Go to 67

Step 3: At 67. Closest: 37 (distance 30) → Go to 37? No: 98 is 31. 37 is 30. → Go to 37

Step 4: At 37. Closest: 14 (distance 23) → Go to 14

Step 5: At 14. Closest: 98 (distance 84) → Go to 98

Step 6: At 98. Closest: 122 (distance 24) → Go to 122

Step 7: At 122. Closest: 124 (distance 2) → Go to 124

Step 8: At 124. Only 183 left → Go to 183

Sequence: 53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183

Total movement = 12+2+30+23+84+24+2+59 = 236 cylinders
```

**Advantages:** Much less head movement than FCFS.
**Disadvantages:** **Starvation** — requests far from the current position may wait indefinitely if new close requests keep arriving.

---

### 10.3 SCAN (Elevator Algorithm)

**Rule:** Head moves in **one direction**, servicing all requests along the way. At the end, **reverses direction** and services requests on the way back. Like an elevator.

```
Head starts at: 53, moving toward higher cylinders (direction: up)
Queue: 98, 183, 37, 122, 14, 124, 65, 67

Moving UP from 53:
53 → 65 → 67 → 98 → 122 → 124 → 183 (hits end)

Then reverses direction (moving DOWN):
183 → 37 → 14

Sequence: 53 → 65 → 67 → 98 → 122 → 124 → 183 → 37 → 14

Total movement = 12+2+31+24+2+59+146+23 = 299 cylinders
```

**Note:** Head goes to the **end of the disk** (track 0 or max track) before reversing, even if no requests are there.

**Advantages:** Fair, no starvation. Better than SSTF worst case.
**Disadvantages:** Requests in the middle get serviced more often (head passes middle twice per sweep). Head goes to disk end even if no requests there.

---

### 10.4 C-SCAN (Circular SCAN)

**Rule:** Head moves in **one direction only** (e.g., always toward higher cylinders), servicing requests. When it reaches the end, it **jumps back to the beginning** and scans again — never services requests on the return trip.

```
Head starts at: 53, moving toward higher cylinders
Queue: 98, 183, 37, 122, 14, 124, 65, 67

Moving UP from 53:
53 → 65 → 67 → 98 → 122 → 124 → 183 → 199 (disk end)

Jump back to 0 (no servicing on return):
0 → 14 → 37

Sequence: 53→65→67→98→122→124→183→(199)→(0)→14→37

Total movement = 12+2+31+24+2+59+(199-183)+(14-0)+23
               = 12+2+31+24+2+59+16+14+23 = 183 cylinders
```

**Advantages:**
- More **uniform wait time** than SCAN — requests in all areas wait at most one full sweep.
- Better fairness than SCAN.

**Disadvantages:** The jump from end to beginning is counted as overhead.

---

### 10.5 LOOK and C-LOOK (Variants)

**LOOK:** Like SCAN but head only goes as far as the **last request in each direction** — does not go to physical disk end.

**C-LOOK:** Like C-SCAN but only goes to the **last request** before jumping back to the **first request** (not disk beginning).

```
C-LOOK with same example:
Queue: 14, 37, 65, 67, 98, 122, 124, 183  (sorted)
Head at 53, moving up:

53 → 65 → 67 → 98 → 122 → 124 → 183 (last high request)
→ Jump to 14 (first low request, not 0)
→ 14 → 37

Total movement = 12+2+31+24+2+59+(183-14)+23 = 306... 
Actually: 12+2+31+24+2+59 = 130 going up
          then jump: doesn't count as movement, or minimal
          14+23 = 37 going in new pass

C-LOOK is generally most efficient in practice.
```

---

### 10.6 Comparison of Disk Scheduling Algorithms

| Algorithm | Total Movement | Starvation | Fairness | Use Case |
|---|---|---|---|---|
| FCFS | 640 | No | High | Light load |
| SSTF | 236 | Yes | Low | Performance priority |
| SCAN | 299 | No | Medium | General purpose |
| C-SCAN | 183 | No | High | High load, uniform access |
| C-LOOK | Similar to C-SCAN | No | High | Modern OS default |

> **Real-world:** Modern SSDs do not need disk scheduling (no physical head) — scheduling is for HDDs only. Linux default scheduler for HDDs is the **deadline scheduler** (based on C-LOOK principles with time limits to prevent starvation).

---

## 11. Laboratory Works

Laboratory programs for Unit 6 implement and compare disk scheduling algorithms:

**Lab 1: FCFS Disk Scheduling**
```c
#include <stdio.h>
#include <stdlib.h>

void fcfs(int requests[], int n, int head) {
    int total = 0;
    int current = head;
    printf("FCFS Order: %d", head);
    for (int i = 0; i < n; i++) {
        total += abs(requests[i] - current);
        current = requests[i];
        printf(" -> %d", current);
    }
    printf("\nTotal Head Movement: %d cylinders\n", total);
}

int main() {
    int requests[] = {98, 183, 37, 122, 14, 124, 65, 67};
    int n = sizeof(requests) / sizeof(requests[0]);
    int head = 53;
    fcfs(requests, n, head);
    return 0;
}
```

**Lab 2: SSTF Disk Scheduling**
```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

void sstf(int requests[], int n, int head) {
    int visited[n], total = 0;
    for (int i = 0; i < n; i++) visited[i] = 0;

    printf("SSTF Order: %d", head);
    for (int count = 0; count < n; count++) {
        int min_dist = INT_MAX, closest = -1;
        for (int i = 0; i < n; i++) {
            if (!visited[i] && abs(requests[i] - head) < min_dist) {
                min_dist = abs(requests[i] - head);
                closest = i;
            }
        }
        visited[closest] = 1;
        total += min_dist;
        head = requests[closest];
        printf(" -> %d", head);
    }
    printf("\nTotal Head Movement: %d cylinders\n", total);
}

int main() {
    int requests[] = {98, 183, 37, 122, 14, 124, 65, 67};
    int n = sizeof(requests) / sizeof(requests[0]);
    sstf(requests, n, 53);
    return 0;
}
```

**Lab 3: SCAN Disk Scheduling**
```c
#include <stdio.h>
#include <stdlib.h>

int compare(const void *a, const void *b) {
    return (*(int*)a - *(int*)b);
}

void scan(int requests[], int n, int head, int disk_size) {
    int total = 0;
    int sorted[n];
    for (int i = 0; i < n; i++) sorted[i] = requests[i];
    qsort(sorted, n, sizeof(int), compare);

    // Find position to split (requests > head)
    int split = 0;
    while (split < n && sorted[split] < head) split++;

    printf("SCAN Order: %d", head);

    // Service requests from head toward higher cylinders
    for (int i = split; i < n; i++) {
        total += abs(sorted[i] - head);
        head = sorted[i];
        printf(" -> %d", head);
    }

    // Go to disk end then reverse
    // Service remaining requests (lower cylinders)
    for (int i = split - 1; i >= 0; i--) {
        total += abs(sorted[i] - head);
        head = sorted[i];
        printf(" -> %d", head);
    }

    printf("\nTotal Head Movement: %d cylinders\n", total);
}

int main() {
    int requests[] = {98, 183, 37, 122, 14, 124, 65, 67};
    int n = sizeof(requests) / sizeof(requests[0]);
    scan(requests, n, 53, 200);
    return 0;
}
```

---

## 12. Summary

### I/O Hardware Summary:
```
I/O Device → Device Controller → System Bus → CPU/Memory
                                      ↕
                               DMA Controller
                               (for bulk transfers)
```

### I/O Software Layers Summary:
```
User Process
    ↓
User-Space Library (printf, fread...)
    ↓
Device-Independent OS Layer (buffering, error handling)
    ↓
Device Driver (device-specific code)
    ↓
Interrupt Handler (hardware signal processing)
    ↓
Hardware
```

### Disk Scheduling Summary:
```
FCFS     → Fair, simple, poor performance
SSTF     → Best performance, starvation possible
SCAN     → No starvation, moderate performance
C-SCAN   → Best uniformity, good performance
C-LOOK   → C-SCAN variant, most practical
```

---

## 13. Exam Tips

Commonly asked questions for Unit 6:

1. **What is a device controller? What are its registers?**
2. **Differentiate block devices, character devices, and communication devices.**
3. **Explain Memory Mapped I/O vs Port-Mapped I/O.**
4. **What is DMA? Explain the DMA transfer process step by step.**
5. **How do interrupts work? Explain the interrupt handling mechanism.**
6. **What are the goals of I/O software?**
7. **Explain the 4 layers of I/O software with functions of each.**
8. **What is the difference between Programmed I/O, Interrupt-Driven I/O, and DMA?**
9. **What is a device driver? Why is it needed?**
10. **Define Seek Time, Rotational Latency, and Transfer Time.**
11. **Solve disk scheduling problems — FCFS, SSTF, SCAN, C-SCAN** *(very common!)*
12. **Compare all disk scheduling algorithms in a table.**
13. **What is starvation in disk scheduling? Which algorithms suffer from it?**

> **Key formula:**
> ```
> Total Access Time = Seek Time + Rotational Latency + Transfer Time
> Rotational Latency (avg) = (60 / RPM) / 2  seconds
> ```

> **Golden rule for disk scheduling numericals:** Always draw a number line (0 to max cylinder), mark the head position, mark all requests, then trace the algorithm step by step — count the absolute differences for total movement.

---

## 14. References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12 — Chapter 5
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014 — Chapters 13 & 10
