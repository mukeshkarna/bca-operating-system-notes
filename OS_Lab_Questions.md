# CACS251 — Operating System
# Laboratory Questions (Unit 1 – Unit 7)
**Year/Semester:** BCA II/IV | Pokhara University
**Language:** C Programming

---

## Instructions for Students

- All programs must be written in **C language**.
- Compile using: `gcc filename.c -o output`
- Run using: `./output`
- Each program must include:
  - Your **name, roll number, and date** in comments at the top.
  - Proper **input validation**.
  - Clear **output formatting**.
- Show the **output/screenshot** along with your code.

---

# LAB 1 — Process Scheduling Algorithms
### (Unit 3: Process Management)

---

## Q1. Implement FCFS CPU Scheduling Algorithm

**Problem:** Write a C program to implement **First Come First Served (FCFS)** CPU scheduling algorithm. The program should:
- Accept number of processes, their Arrival Time (AT) and Burst Time (BT) as input.
- Calculate Completion Time (CT), Turnaround Time (TAT), and Waiting Time (WT) for each process.
- Print the **Gantt Chart**.
- Calculate and print **Average TAT** and **Average WT**.

**Sample Input:**
```
Number of processes: 4
Process  AT  BT
P1        0   5
P2        1   3
P3        2   8
P4        3   6
```

**Expected Output:**
```
Gantt Chart:
| P1 | P2 | P3 | P4 |
0    5    8   16   22

Process  AT  BT  CT  TAT  WT
P1        0   5   5    5   0
P2        1   3   8    7   4
P3        2   8  16   14   6
P4        3   6  22   19  13

Average TAT = 11.25 ms
Average WT  =  5.75 ms
```

---

## Q2. Implement SJF (Non-Preemptive) Scheduling Algorithm

**Problem:** Write a C program to implement **Shortest Job First (SJF)** non-preemptive CPU scheduling algorithm. The program should:
- Accept number of processes, AT and BT as input.
- At each scheduling point, select the process with the **minimum Burst Time** among all arrived processes.
- Calculate CT, TAT, WT for each process.
- Print Gantt Chart and averages.

**Sample Input:**
```
Number of processes: 4
Process  AT  BT
P1        0   6
P2        1   8
P3        2   7
P4        3   3
```

**Expected Output:**
```
Gantt Chart:
| P1 | P4 | P3 | P2 |
0    6    9   16   24

Process  AT  BT  CT  TAT  WT
P1        0   6   6    6   0
P4        3   3   9    6   3
P3        2   7  16   14   7
P2        1   8  24   23  15

Average TAT = 12.25 ms
Average WT  =  6.25 ms
```

---

## Q3. Implement Round Robin Scheduling Algorithm

**Problem:** Write a C program to implement **Round Robin (RR)** CPU scheduling algorithm. The program should:
- Accept number of processes, their AT and BT, and the **Time Quantum**.
- Simulate the ready queue and time slices.
- Correctly handle new arrivals **during** a running quantum.
- Print Gantt Chart, CT, TAT, WT for each process, and averages.

**Sample Input:**
```
Number of processes: 3
Process  AT  BT
P1        0   8
P2        1   4
P3        2   3

Time Quantum: 3
```

**Expected Output:**
```
Gantt Chart:
| P1 | P2 | P3 | P1 | P2 | P1 |
0    3    6    9   12  13   16

Process  AT  BT  CT  TAT  WT
P1        0   8  16   16   8
P2        1   4  13   12   8
P3        2   3   9    7   4

Average TAT = 11.67 ms
Average WT  =  6.67 ms
```

---

## Q4. Implement Priority Scheduling Algorithm (Non-Preemptive)

**Problem:** Write a C program to implement **Priority Scheduling** (non-preemptive). The program should:
- Accept number of processes, AT, BT, and Priority (lower number = higher priority).
- At each scheduling point, select the arrived process with the **highest priority (lowest number)**.
- Print Gantt Chart, CT, TAT, WT, and averages.
- **Bonus:** Mention which processes could face starvation.

**Sample Input:**
```
Number of processes: 5
Process  AT  BT  Priority
P1        0  10     3
P2        0   1     1
P3        0   2     4
P4        0   1     5
P5        0   5     2
```

**Expected Output:**
```
Execution Order: P2 → P5 → P1 → P3 → P4

Process  AT  BT  PR  CT  TAT  WT
P2        0   1   1   1    1   0
P5        0   5   2   6    6   1
P1        0  10   3  16   16   6
P3        0   2   4  18   18  16
P4        0   1   5  19   19  18

Average TAT = 12.00 ms
Average WT  =  8.20 ms

Note: P4 (priority 5) could face starvation if higher priority
      processes keep arriving continuously.
```

---

## Q5. Compare All Scheduling Algorithms

**Problem:** Write a C program that implements **FCFS, SJF, and Round Robin** in a single program and compares their performance for the same set of processes.

- Input one set of processes (AT and BT).
- Run all three algorithms.
- Print a **comparison table** showing Average TAT and Average WT for each.
- Identify which algorithm performs best for the given input.

**Sample Output:**
```
===== SCHEDULING ALGORITHM COMPARISON =====

Algorithm    Avg TAT    Avg WT
FCFS          11.25      5.75
SJF           12.25      6.25
Round Robin   11.67      6.67

Best Average Waiting Time: FCFS (5.75 ms)
```

---

# LAB 2 — Process Synchronization (IPC)
### (Unit 3: IPC & Synchronization)

---

## Q6. Simulate Producer-Consumer Problem using Semaphores

**Problem:** Write a C program to simulate the **Producer-Consumer (Bounded Buffer) Problem** using semaphores and mutex. The program should:
- Use a **circular buffer** of size N (input from user).
- Producer: produces items and adds to buffer (waits if buffer full).
- Consumer: removes items from buffer and consumes (waits if buffer empty).
- Use **POSIX semaphores** (`sem_init`, `sem_wait`, `sem_post`).
- Use **threads** (`pthread_create`) — one producer, one consumer.
- Show buffer state after each produce/consume operation.

**Compile:** `gcc q6.c -o q6 -lpthread`

**Sample Output:**
```
Buffer size: 5

Producer produced: 1  | Buffer: [1][ ][ ][ ][ ]  | Count: 1
Producer produced: 2  | Buffer: [1][2][ ][ ][ ]  | Count: 2
Consumer consumed: 1  | Buffer: [2][ ][ ][ ][ ]  | Count: 1
Producer produced: 3  | Buffer: [2][3][ ][ ][ ]  | Count: 2
Producer produced: 4  | Buffer: [2][3][4][ ][ ]  | Count: 3
Consumer consumed: 2  | Buffer: [3][4][ ][ ][ ]  | Count: 2
```

---

## Q7. Simulate the Dining Philosophers Problem

**Problem:** Write a C program to simulate the **Dining Philosophers Problem** for 5 philosophers. The program should:
- Use **5 threads** (one per philosopher).
- Use **5 semaphores** (one per fork).
- Each philosopher: thinks → picks up left fork → picks up right fork → eats → puts down forks.
- To avoid deadlock: **philosopher 4 picks up right fork first** (breaks symmetry).
- Print the state of each philosopher (thinking/waiting/eating).
- Run for a fixed number of meals (e.g., each philosopher eats 3 times).

**Sample Output:**
```
Philosopher 0: THINKING
Philosopher 1: THINKING
Philosopher 0: HUNGRY — waiting for forks
Philosopher 0: EATING  (forks 4 and 0)
Philosopher 2: THINKING
Philosopher 1: HUNGRY — waiting for forks
Philosopher 0: DONE EATING — releasing forks
Philosopher 1: EATING  (forks 0 and 1)
...
```

---

## Q8. Implement Mutex Lock for Critical Section

**Problem:** Write a C program demonstrating a **race condition** and then fixing it using a **mutex lock**. The program should:

**Part A — Show Race Condition:**
- Create 2 threads that both increment a shared counter 100,000 times.
- Run without synchronization.
- Show that the final count is incorrect (less than 200,000).

**Part B — Fix with Mutex:**
- Same program but protect the increment with `pthread_mutex_lock` / `pthread_mutex_unlock`.
- Show that the final count is now exactly 200,000.

**Sample Output:**
```
===== Part A: WITHOUT Mutex (Race Condition) =====
Thread 1 incrementing 100000 times...
Thread 2 incrementing 100000 times...
Expected counter: 200000
Actual counter:   187432   ← WRONG! Race condition occurred.

===== Part B: WITH Mutex (Synchronized) =====
Thread 1 incrementing 100000 times...
Thread 2 incrementing 100000 times...
Expected counter: 200000
Actual counter:   200000   ← CORRECT! Mutex prevented race condition.
```

---

# LAB 3 — Deadlocks
### (Unit 4: Deadlocks)

---

## Q9. Implement Banker's Algorithm — Safety Check

**Problem:** Write a C program to implement the **Banker's Algorithm Safety Check**. The program should:
- Accept number of processes (n) and resource types (m) from user.
- Accept Allocation matrix, Max matrix, and Available vector.
- **Calculate the Need matrix** (Need = Max − Allocation).
- **Run the Safety Algorithm** to find if the system is in a safe state.
- Print the **safe sequence** if it exists.
- Print **"UNSAFE STATE"** if no safe sequence found.

**Sample Input:**
```
Number of processes: 5
Number of resource types: 3

Allocation Matrix:
P0: 0 1 0
P1: 2 0 0
P2: 3 0 2
P3: 2 1 1
P4: 0 0 2

Max Matrix:
P0: 7 5 3
P1: 3 2 2
P2: 9 0 2
P3: 2 2 2
P4: 4 3 3

Available: 3 3 2
```

**Expected Output:**
```
Need Matrix:
P0: 7 4 3
P1: 1 2 2
P2: 6 0 0
P3: 0 1 1
P4: 4 3 1

Safety Algorithm Execution:
Step 1: P1 selected (Need[1,2,2] <= Work[3,3,2]) Work = [5,3,2]
Step 2: P3 selected (Need[0,1,1] <= Work[5,3,2]) Work = [7,4,3]
Step 3: P4 selected (Need[4,3,1] <= Work[7,4,3]) Work = [7,4,5]
Step 4: P0 selected (Need[7,4,3] <= Work[7,4,5]) Work = [7,5,5]
Step 5: P2 selected (Need[6,0,0] <= Work[7,5,5]) Work = [10,5,7]

System is in SAFE STATE.
Safe Sequence: P1 -> P3 -> P4 -> P0 -> P2
```

---

## Q10. Implement Banker's Algorithm — Resource Request

**Problem:** Extend Q9 to also handle **Resource Requests**. After finding the safe state, the program should:
- Ask which process wants to make a request.
- Accept the request vector.
- Check: Request ≤ Need? Request ≤ Available?
- **Pretend to allocate** and run Safety Algorithm.
- Print whether request is **GRANTED or DENIED** with reason.
- If granted, show the **new safe sequence**.

**Sample Input (continuing from Q9):**
```
Enter process making request: P1
Enter request vector: 1 0 2
```

**Expected Output:**
```
P1 requests: [1, 0, 2]

Check 1: Request [1,0,2] <= Need[P1] [1,2,2]? YES
Check 2: Request [1,0,2] <= Available [3,3,2]? YES

Trial allocation:
  Available:     [3,3,2] - [1,0,2] = [2,3,0]
  Allocation[P1]: [2,0,0] + [1,0,2] = [3,0,2]
  Need[P1]:       [1,2,2] - [1,0,2] = [0,2,0]

Running Safety Algorithm on new state...
Safe Sequence found: P1 -> P3 -> P4 -> P0 -> P2

REQUEST GRANTED. System remains in SAFE STATE.
```

---

## Q11. Deadlock Detection using Resource Allocation Graph (Cycle Detection)

**Problem:** Write a C program to detect a deadlock using **cycle detection in a Resource Allocation Graph (RAG)**. The program should:
- Represent the RAG as an **adjacency matrix**.
- Accept number of processes and resources, and edges (request/assignment).
- Use **DFS** to detect if a cycle exists.
- Print whether **DEADLOCK EXISTS** or system is **DEADLOCK FREE**.
- If deadlock exists, print the **cycle** (which processes are involved).

**Sample Input:**
```
Number of processes: 3
Number of resources: 3
Total nodes: 6 (P0,P1,P2 = nodes 0,1,2; R0,R1,R2 = nodes 3,4,5)

Edges (from to):
P0 -> R0  (0 3)   P0 requests R0
R0 -> P1  (3 1)   R0 assigned to P1
P1 -> R1  (1 4)   P1 requests R1
R1 -> P2  (4 2)   R1 assigned to P2
P2 -> R0  (2 3)   P2 requests R0

Number of edges: 5
```

**Expected Output:**
```
Graph edges:
P0 -> R0 (request)
R0 -> P1 (assigned)
P1 -> R1 (request)
R1 -> P2 (assigned)
P2 -> R0 (request)

Running DFS cycle detection...

DEADLOCK DETECTED!
Cycle found: P0 -> R0 -> P1 -> R1 -> P2 -> R0

Deadlocked processes: P0, P1, P2
```

---

# LAB 4 — Memory Management
### (Unit 5: Memory Management)

---

## Q12. Simulate Memory Allocation Strategies

**Problem:** Write a C program to simulate **memory allocation strategies**. The program should:
- Initialize memory with a list of free holes (start address and size).
- Accept memory requests (process name and size).
- Implement and compare all three strategies:
  - **First Fit** — allocate first hole large enough.
  - **Best Fit** — allocate smallest sufficient hole.
  - **Worst Fit** — allocate largest available hole.
- After each allocation, show the **updated memory state**.
- Show which hole was chosen and how much **internal fragmentation** occurred.

**Sample Input:**
```
Memory holes (address, size):
Hole 1: Address=100, Size=50KB
Hole 2: Address=200, Size=20KB
Hole 3: Address=300, Size=30KB
Hole 4: Address=450, Size=45KB

Memory Requests:
Process A: 25KB
Process B: 18KB
Process C: 40KB
```

**Expected Output:**
```
===== FIRST FIT =====
Process A (25KB): Hole 1 (50KB) selected. Remaining: 25KB. Fragmentation: 0KB
Process B (18KB): Hole 1 remaining (25KB) selected. Remaining: 7KB. Fragmentation: 0KB
Process C (40KB): Hole 4 (45KB) selected. Remaining: 5KB. Fragmentation: 0KB

===== BEST FIT =====
Process A (25KB): Hole 3 (30KB) selected. Remaining: 5KB. Fragmentation: 0KB
Process B (18KB): Hole 2 (20KB) selected. Remaining: 2KB. Fragmentation: 0KB
Process C (40KB): Hole 4 (45KB) selected. Remaining: 5KB. Fragmentation: 0KB

===== WORST FIT =====
Process A (25KB): Hole 1 (50KB) selected. Remaining: 25KB. Fragmentation: 0KB
Process B (18KB): Hole 4 remaining (45KB)... etc.
```

---

## Q13. Implement FIFO Page Replacement Algorithm

**Problem:** Write a C program to implement the **FIFO page replacement algorithm**. The program should:
- Accept the **reference string** and **number of frames** from user.
- Simulate page replacement step by step.
- For each reference: show current frames, whether it is a **HIT or MISS**.
- Count and display total number of **page faults**.
- Demonstrate **Belady's Anomaly** by running the same reference string with 3 frames and 4 frames and comparing results.

**Sample Input:**
```
Reference string: 1 2 3 4 1 2 5 1 2 3 4 5
Number of frames: 3
```

**Expected Output:**
```
FIFO Page Replacement (Frames = 3)

Ref  Frame1  Frame2  Frame3  Fault?
 1     1       -       -      YES
 2     1       2       -      YES
 3     1       2       3      YES
 4     4       2       3      YES (evict 1)
 1     4       1       3      YES (evict 2)
 2     4       1       2      YES (evict 3)
 5     5       1       2      YES (evict 4)
 1     5       1       2      NO
 2     5       1       2      NO
 3     5       3       2      YES (evict 1)
 4     5       3       4      YES (evict 2)
 5     5       3       4      NO

Total Page Faults = 9

===== Belady's Anomaly Demonstration =====
Same string with 3 frames: 9 page faults
Same string with 4 frames: 10 page faults
MORE faults with MORE frames! This is Belady's Anomaly.
```

---

## Q14. Implement LRU Page Replacement Algorithm

**Problem:** Write a C program to implement the **LRU (Least Recently Used) page replacement algorithm**. The program should:
- Accept reference string and number of frames.
- Track the **last used time** of each page in frames.
- On page fault: evict the page with the **oldest last-used time**.
- Display step-by-step execution with HIT/MISS for each reference.
- Display total page faults.

**Sample Input:**
```
Reference string: 7 0 1 2 0 3 0 4 2 3 0 3 2 1 2
Number of frames: 4
```

**Expected Output:**
```
LRU Page Replacement (Frames = 4)

Ref  Frames (MRU to LRU)          Fault?
 7   [7]                            YES
 0   [0, 7]                         YES
 1   [1, 0, 7]                      YES
 2   [2, 1, 0, 7]                   YES
 0   [0, 2, 1, 7]                   NO  (hit, 0 moved to front)
 3   [3, 0, 2, 1]                   YES (evict 7 — LRU)
 0   [0, 3, 2, 1]                   NO  (hit)
 4   [4, 0, 3, 2]                   YES (evict 1 — LRU)
 2   [2, 4, 0, 3]                   NO  (hit)
 3   [3, 2, 4, 0]                   NO  (hit)
 0   [0, 3, 2, 4]                   NO  (hit)
 3   [3, 0, 2, 4]                   NO  (hit)
 2   [2, 3, 0, 4]                   NO  (hit)
 1   [1, 2, 3, 0]                   YES (evict 4 — LRU)
 2   [2, 1, 3, 0]                   NO  (hit)

Total Page Faults = 8
```

---

## Q15. Compare FIFO, LRU, and Optimal Page Replacement

**Problem:** Write a C program that implements **all three page replacement algorithms** (FIFO, LRU, Optimal) and compares their performance for the same reference string and number of frames.

- Input one reference string and one frame count.
- Run all three algorithms.
- Print a **comparison table** of total page faults.
- Show which algorithm is best.

**Sample Output:**
```
Reference String: 1 2 3 4 1 2 5 1 2 3 4 5
Number of Frames: 3

========== PAGE REPLACEMENT COMPARISON ==========

Algorithm   Page Faults
FIFO             9
LRU             10
Optimal          7

Best performance: Optimal (7 faults) — theoretical minimum
Most practical:   LRU (approximates optimal without future knowledge)
```

---

# LAB 5 — I/O & Disk Scheduling
### (Unit 6: I/O Device Management)

---

## Q16. Implement FCFS Disk Scheduling Algorithm

**Problem:** Write a C program to implement **FCFS disk scheduling**. The program should:
- Accept current head position and disk request queue from user.
- Service requests in **arrival order**.
- Show movement from one cylinder to the next.
- Display total head movement in cylinders.
- Draw a simple **text-based seek diagram**.

**Sample Input:**
```
Current head position: 53
Request queue: 98 183 37 122 14 124 65 67
```

**Expected Output:**
```
FCFS Disk Scheduling
Head starts at: 53
Request queue: 98 183 37 122 14 124 65 67

Execution order:
53 -> 98   (movement: 45)
98 -> 183  (movement: 85)
183 -> 37  (movement: 146)
37 -> 122  (movement: 85)
122 -> 14  (movement: 108)
14 -> 124  (movement: 110)
124 -> 65  (movement: 59)
65 -> 67   (movement: 2)

Total Head Movement: 640 cylinders
```

---

## Q17. Implement SSTF Disk Scheduling Algorithm

**Problem:** Write a C program to implement **SSTF (Shortest Seek Time First) disk scheduling**. The program should:
- At each step, find the **unserviced request closest** to the current head.
- Show step-by-step execution with distances.
- Display total head movement.
- **Bonus:** Identify if any request could face **starvation** (remained in queue the longest).

**Sample Input:**
```
Current head position: 53
Request queue: 98 183 37 122 14 124 65 67
```

**Expected Output:**
```
SSTF Disk Scheduling
Head starts at: 53

Step 1: At 53. Distances: [98:45][183:130][37:16][122:69][14:39][124:71][65:12][67:14]
        Closest: 65 (distance: 12). Move to 65.

Step 2: At 65. Distances: [98:33][183:118][37:28][122:57][14:51][124:59][67:2]
        Closest: 67 (distance: 2). Move to 67.

Step 3: At 67. Closest: 37 (distance: 30). Move to 37.
Step 4: At 37. Closest: 14 (distance: 23). Move to 14.
Step 5: At 14. Closest: 98 (distance: 84). Move to 98.
Step 6: At 98. Closest: 122 (distance: 24). Move to 122.
Step 7: At 122. Closest: 124 (distance: 2). Move to 124.
Step 8: At 124. Last: 183 (distance: 59). Move to 183.

Order: 53 -> 65 -> 67 -> 37 -> 14 -> 98 -> 122 -> 124 -> 183
Total Head Movement: 236 cylinders
```

---

## Q18. Implement SCAN and C-SCAN Disk Scheduling

**Problem:** Write a C program implementing both **SCAN** and **C-SCAN** disk scheduling. The program should:
- Accept head position, disk size, and request queue.
- Ask for initial **direction** (towards higher or lower cylinders) for SCAN.
- For SCAN: move in one direction, reverse at end.
- For C-SCAN: move in one direction, jump back to beginning.
- Show step-by-step execution for both.
- Print **comparison table** of total movements.

**Sample Input:**
```
Current head position: 53
Disk size (max cylinder): 199
Initial direction: towards higher cylinders
Request queue: 98 183 37 122 14 124 65 67
```

**Expected Output:**
```
===== SCAN Disk Scheduling =====
Direction: increasing cylinders

Moving UP from 53:
53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183 -> 199 (end)

Reversing direction (moving DOWN):
199 -> 37 -> 14

Order: 53->65->67->98->122->124->183->199->37->14
Total Head Movement: 331 cylinders

===== C-SCAN Disk Scheduling =====
Direction: increasing cylinders only

Moving UP from 53:
53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183 -> 199 (end)

Jump to beginning: 199 -> 0

Moving UP from 0:
0 -> 14 -> 37

Order: 53->65->67->98->122->124->183->(199->0)->14->37
Total Head Movement: 382 cylinders (including end-to-start jump)

===== COMPARISON =====
Algorithm    Total Movement
SCAN              331
C-SCAN            382
```

---

## Q19. Compare All Disk Scheduling Algorithms

**Problem:** Write a C program that implements **all four disk scheduling algorithms** (FCFS, SSTF, SCAN, C-SCAN) for the same request queue and prints a comparison.

**Sample Output:**
```
Current head: 53
Queue: 98 183 37 122 14 124 65 67
Disk size: 199

========= DISK SCHEDULING COMPARISON =========

Algorithm    Order of Service                        Total Movement
FCFS         53->98->183->37->122->14->124->65->67     640
SSTF         53->65->67->37->14->98->122->124->183      236
SCAN         53->65->67->98->122->124->183->37->14      299
C-SCAN       53->65->67->98->122->124->183->14->37      183

Best performance: C-SCAN (183 cylinders)
Worst performance: FCFS  (640 cylinders)

Note: SSTF gives good performance but can cause starvation.
      C-SCAN gives most uniform wait times.
```

---

# LAB 6 — File System Operations
### (Unit 7: File System Interface Management)

---

## Q20. Simulate File Allocation Methods

**Problem:** Write a C program to simulate and compare **file allocation methods**. The program should simulate:

**Part A — Contiguous Allocation:**
- User inputs disk blocks (total), files with their start block and size.
- Show which blocks each file occupies.
- Show free blocks.
- Demonstrate direct access to block n of a file.

**Part B — Linked List Allocation:**
- Simulate FAT (File Allocation Table) as an array.
- User inputs file names and their block chains.
- Show the FAT table.
- Traverse from start block to find any block of a file.

**Sample Output:**
```
===== Part A: CONTIGUOUS ALLOCATION =====
Disk has 20 blocks (0-19)

File     Start  Size  Blocks Used
file.txt   3      4    3, 4, 5, 6
prog.exe   8      3    8, 9, 10
data.csv  14      2    14, 15

Free blocks: 0,1,2,7,11,12,13,16,17,18,19

Direct access: Block 2 of file.txt = Block (3+2) = Block 5 ✓

===== Part B: LINKED LIST (FAT) ALLOCATION =====
FAT Table:
Block  Next_Block
  0      -1 (free)
  1       4
  2      -1 (free)
  3      -1 (EOF)
  4       7
  5      -1 (free)
  6      -1 (free)
  7       3

file.txt starts at block 1: 1 -> 4 -> 7 -> 3 -> EOF
```

---

## Q21. Simulate Directory Operations

**Problem:** Write a C program to simulate a **simple hierarchical file system** with directory operations. The program should:
- Implement a **tree structure** using linked lists/structs.
- Support the following operations through a **menu**:
  1. `mkdir <name>` — create directory
  2. `touch <name>` — create file
  3. `ls` — list current directory contents
  4. `cd <name>` — change directory
  5. `cd ..` — go to parent directory
  6. `pwd` — print current path (absolute path)
  7. `rm <name>` — remove file
  8. `rmdir <name>` — remove empty directory
  9. `exit` — quit

**Sample Session:**
```
[/] $ mkdir home
[/] $ cd home
[/home] $ mkdir mukesh
[/home] $ cd mukesh
[/home/mukesh] $ touch report.pdf
[/home/mukesh] $ touch notes.txt
[/home/mukesh] $ ls
  [DIR]  .
  [DIR]  ..
  [FILE] report.pdf
  [FILE] notes.txt
[/home/mukesh] $ pwd
/home/mukesh
[/home/mukesh] $ cd ..
[/home] $ ls
  [DIR]  .
  [DIR]  ..
  [DIR]  mukesh
[/home] $ exit
Goodbye!
```

---

## Q22. Simulate Free Space Management (Bitmap)

**Problem:** Write a C program to simulate **bitmap-based free space management**. The program should:
- Initialize a disk of N blocks (input from user).
- Maintain a **bitmap array** (0=free, 1=allocated).
- Support operations through menu:
  1. **Allocate** — find and allocate n contiguous free blocks for a file.
  2. **Deallocate** — free all blocks of a given file.
  3. **Display** — show current bitmap and list all free/used blocks.
  4. **Find** — find first available block for allocation.
- Show the bitmap after each operation.

**Sample Output:**
```
Disk: 16 blocks (0 to 15)
Initial bitmap: 0000000000000000 (all free)

1. Allocate file.txt (4 blocks)
   Found contiguous free blocks: 0,1,2,3
   Allocated blocks: 0,1,2,3
   Bitmap: 1111000000000000

2. Allocate prog.exe (3 blocks)
   Found contiguous free blocks: 4,5,6
   Allocated blocks: 4,5,6
   Bitmap: 1111111000000000

3. Deallocate file.txt
   Freed blocks: 0,1,2,3
   Bitmap: 0000111000000000

4. Allocate data.csv (5 blocks)
   Found contiguous free blocks: 0,1,2,3,7 — NOT contiguous!
   Try different location: 7,8,9,10,11
   Allocated blocks: 7,8,9,10,11
   Bitmap: 0000111111110000

Display current state:
Bitmap: 0 0 0 0 1 1 1 1 1 1 1 1 0 0 0 0
Block:  0 1 2 3 4 5 6 7 8 9 ...
Free blocks: 0,1,2,3,12,13,14,15 (8 free)
Used blocks: 4,5,6,7,8,9,10,11  (8 used)
```

---

## Q23. Simulate Unix File Permission System

**Problem:** Write a C program to simulate the **Unix file permission system**. The program should:
- Store files with their permission bits (owner, group, others), owner name, and file type.
- Implement a function `check_permission(user, file, operation)` that returns whether access is allowed.
- Support menu-driven operations:
  1. **Create file** with permissions (e.g., 755, 644, 700)
  2. **Change permission** (`chmod`)
  3. **Check access** — given a user and operation (read/write/execute), allow or deny
  4. **Display** file info with permission string (e.g., `-rwxr-xr--`)
  5. **List all files** with permissions

**Sample Output:**
```
===== Unix File Permission Simulator =====

1. Create file: report.pdf  Owner: mukesh  Permission: 644
   Stored as: -rw-r--r--  Owner: mukesh

2. Create file: script.sh   Owner: mukesh  Permission: 755
   Stored as: -rwxr-xr-x  Owner: mukesh

3. Check access:
   User: mukesh, File: report.pdf, Operation: WRITE
   Permission: rw-r--r--
   mukesh is owner → checking owner bits: rw- → WRITE ALLOWED ✓

   User: ram, File: report.pdf, Operation: WRITE
   ram is not owner, not in group → checking other bits: r-- → WRITE DENIED ✗

   User: ram, File: script.sh, Operation: EXECUTE
   Checking other bits: r-x → EXECUTE ALLOWED ✓

4. Display: report.pdf
   Type:        Regular File (-)
   Permissions: rw-r--r-- (644)
   Owner:       mukesh
   Readable by: owner, group, others
   Writable by: owner only
   Executable:  nobody
```

---

# Summary — Lab Schedule

| Lab # | Topic | Questions | Unit |
|---|---|---|---|
| Lab 1 | CPU Scheduling | Q1–Q5 | Unit 3 |
| Lab 2 | Process Synchronization | Q6–Q8 | Unit 3 |
| Lab 3 | Deadlocks | Q9–Q11 | Unit 4 |
| Lab 4 | Memory Management | Q12–Q15 | Unit 5 |
| Lab 5 | Disk Scheduling | Q16–Q19 | Unit 6 |
| Lab 6 | File System | Q20–Q23 | Unit 7 |

---

# Evaluation Criteria

| Criteria | Marks |
|---|---|
| Program runs correctly with correct output | 40% |
| Code is well-structured and readable | 20% |
| Proper comments and documentation | 15% |
| Input validation and error handling | 15% |
| Viva questions (explanation of program) | 10% |
| **Total** | **100%** |

---

# Important Notes

1. **Unit 1 & 2** (Introduction and OS Structure) are theoretical — no specific lab programs needed.
2. For **Unit 3 labs**, POSIX threads (`pthread`) are required. Include `-lpthread` flag when compiling.
3. For **Unit 5 and 6 labs**, draw the **step-by-step execution** manually first, then verify with program output.
4. All programs should handle **edge cases**:
   - Empty input / zero processes
   - Single process / single frame
   - All pages already in frames (no faults)
   - Disk queue empty
5. **Bonus marks** are given for programs that accept input from command line arguments or file instead of just keyboard input.

---

## References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014
- GNU C Library Documentation: https://www.gnu.org/software/libc/manual/
- POSIX Threads: `man pthread_create`, `man sem_init`
