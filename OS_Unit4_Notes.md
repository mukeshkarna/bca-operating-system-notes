# Unit 4: Deadlocks
**Course:** CACS251 — Operating System | **Duration:** 4 Hours
**Year/Semester:** BCA II/IV | Pokhara University

---

## Syllabus Topics
System Model, System Resources: Preemptable and Non-Preemptable; Conditions for Resource Deadlocks, Deadlock Modeling, The OSTRICH Algorithm, Method of Handling Deadlocks, Deadlock Prevention, Deadlock Avoidance: Banker's Algorithm, Deadlock Detection: Resource Allocation Graph, Recovery from Deadlock.

---

## 1. System Model

In an OS, processes need **resources** to execute. The general sequence for using a resource is:

```
Request → Allocate → Use → Release
```

**Steps:**
1. **Request:** Process requests the resource. If not available, it waits.
2. **Allocate (Use):** Process uses the allocated resource.
3. **Release:** Process releases the resource after use.

> **Real-world example:** You (process) want to print (resource). You request the printer → wait if busy → print your document → release the printer for the next person.

---

## 2. System Resources

### 2.1 Preemptable Resources
- Resources that **can be taken away** from a process without causing failure.
- The process can resume after the resource is returned.
- **Examples:** CPU (can be preempted by scheduler), Main Memory (pages can be swapped out).

### 2.2 Non-Preemptable Resources
- Resources that **cannot be forcibly taken away** without causing the process to fail.
- Must be held until voluntarily released.
- **Examples:** Printer (cannot pull paper mid-print), CD-ROM writer (cannot interrupt a burn), tape drive.

> **Deadlocks typically occur with Non-Preemptable resources** because they cannot be forcibly reclaimed.

---

## 3. What is a Deadlock?

A **deadlock** is a situation where a set of processes are **permanently blocked**, each waiting for a resource held by another process in the set — and none can proceed.

```
Process A holds Resource 1, waits for Resource 2
Process B holds Resource 2, waits for Resource 1
→ Neither can proceed → DEADLOCK
```

> **Real-world example:** Traffic gridlock at an intersection — each car is blocking the path of another car, and nobody can move forward. Or two people on a narrow bridge, each waiting for the other to step back.

---

## 4. Conditions for Deadlock (Coffman Conditions)

All **four conditions must hold simultaneously** for a deadlock to occur. If even one is broken, deadlock cannot happen.

### 4.1 Mutual Exclusion
- At least one resource must be held in a **non-shareable** mode.
- Only one process can use the resource at a time.
- Example: Only one process can use a printer at a time.

### 4.2 Hold and Wait
- A process must be **holding at least one resource** while **waiting to acquire additional resources** held by other processes.
- Example: Process A holds the printer and waits for the scanner.

### 4.3 No Preemption
- Resources **cannot be forcibly taken** from a process; they must be released voluntarily.
- Example: Cannot pull the printer away from Process A mid-print.

### 4.4 Circular Wait
- A **circular chain** of processes exists where each process holds a resource needed by the next process in the chain.
- Example: P1 → P2 → P3 → P1 (P1 waits for P2, P2 waits for P3, P3 waits for P1).

> **Memory trick:** **M-H-N-C** — **M**utual exclusion, **H**old and wait, **N**o preemption, **C**ircular wait.

---

## 5. Deadlock Modeling (Resource Allocation Graph)

A **Resource Allocation Graph (RAG)** is a directed graph used to model deadlock states.

### Components:
- **Processes** — represented as **circles** (○)
- **Resources** — represented as **rectangles** (□), with dots inside representing instances
- **Request Edge** — process → resource (P → R): process is requesting a resource
- **Assignment Edge** — resource → process (R → P): resource is allocated to a process

### Interpreting RAG:

```
Case 1 — No Deadlock:
  P1 ──→ R1 ──→ P2    (P1 requests R1, R1 assigned to P2)
  (No cycle = No deadlock)

Case 2 — Deadlock:
  P1 ──→ R1 ──→ P2
  P2 ──→ R2 ──→ P1
  (Cycle exists = Deadlock with single-instance resources)

Case 3 — Cycle but No Deadlock:
  If resources have multiple instances, a cycle does NOT necessarily mean deadlock.
```

**Rules:**
- If graph has **no cycle** → **no deadlock**.
- If graph has a **cycle** and resources have **single instances** → **deadlock**.
- If graph has a **cycle** and resources have **multiple instances** → deadlock **may or may not** exist.

---

## 6. The OSTRICH Algorithm

The **Ostrich Algorithm** is the simplest approach to dealing with deadlocks — **ignore the problem** entirely!

```
Strategy: Pretend deadlocks don't exist.
```

**Rationale:**
- Detecting and preventing deadlocks is **expensive** (overhead).
- Deadlocks may be **very rare** in practice.
- It may be more cost-effective to simply **reboot** when a deadlock is detected.

**Used by:** UNIX, Linux, and Windows — they use the Ostrich Algorithm for certain types of deadlocks. Most desktop systems don't implement full deadlock prevention.

> **Real-world analogy:** Ignoring a rare bug in software — the cost of fixing it (extensive testing, rewriting code) is higher than the cost of the bug occurring (user restarts the app). Like an ostrich burying its head in the sand — pretend the danger doesn't exist!

**Pros:** Simple, no overhead.
**Cons:** Can cause system instability; not suitable for critical systems (medical, banking, aerospace).

---

## 7. Methods of Handling Deadlocks

There are **four main strategies** for handling deadlocks:

| Strategy | Approach | Cost |
|---|---|---|
| **Prevention** | Ensure at least one Coffman condition never holds | High (restricts resource usage) |
| **Avoidance** | Dynamically grant resources only if safe | Medium (requires advance info) |
| **Detection & Recovery** | Allow deadlock, detect it, then recover | Medium (periodic overhead) |
| **Ignorance (Ostrich)** | Ignore deadlocks — reboot if needed | Very Low |

---

## 8. Deadlock Prevention

**Goal:** Ensure that **at least one of the four Coffman conditions can never hold**.

### 8.1 Attacking Mutual Exclusion
- Make resources **shareable** where possible.
- **Problem:** Not all resources can be shared (e.g., printer).
- **Solution:** Use **spooling** — a daemon (print spooler) manages the printer, and processes write to a spool queue instead of accessing the printer directly. Only the spooler uses the printer.
- > **Example:** Print spooler in Windows — multiple users "print" to a queue; the printer is not directly locked by any user process.

### 8.2 Attacking Hold and Wait
- Require processes to **request all resources at once** before starting.
- Two approaches:
  1. **Request all at start:** Process requests every resource it will ever need before beginning. If any is unavailable, it waits.
  2. **Release before requesting:** Process must release all currently held resources before requesting new ones.
- **Problem:** Low resource utilization; starvation possible (process that needs many resources may never get all simultaneously).

### 8.3 Attacking No Preemption
- Allow the OS to **preempt resources** from a waiting process.
- If a process requests a resource and must wait:
  - All resources it currently holds are preempted (saved).
  - Process restarts when it can get all needed resources.
- **Problem:** Only feasible for resources whose state can be saved (CPU registers, memory) — not for printers or tape drives.

### 8.4 Attacking Circular Wait
- Impose a **total ordering** on all resource types.
- Processes can only request resources in **increasing order** of their numbers.
- If a process holds resource type R_i, it can only request types R_j where j > i.
- This eliminates circular chains.

> **Example:** Assign number: Printer=1, Scanner=2, Disk=3. Every process must request in order: first Printer (if needed), then Scanner, then Disk. Process B cannot hold Scanner (2) and request Printer (1) — breaks circular wait!

---

## 9. Deadlock Avoidance

**Goal:** Dynamically decide whether granting a resource request is **safe** or might lead to deadlock.

Requires **advance information** from each process:
- Maximum number of resources of each type it may need.

### 9.1 Safe State
A system is in a **safe state** if there exists a **safe sequence** — an order in which all processes can complete, each using resources released by previously completed processes.

```
Safe state → No deadlock (at this moment)
Unsafe state → Possible deadlock (not certain, but risky)
```

**Key principle:** Never enter an unsafe state — even if a request could be granted, refuse it if it leads to an unsafe state.

### 9.2 Banker's Algorithm

Developed by **Dijkstra**, named after a banker who decides whether to grant loans without going bankrupt.

**Data Structures (for n processes, m resource types):**

| Variable | Description |
|---|---|
| `Available[m]` | Number of available instances of each resource type |
| `Max[n][m]` | Maximum demand of each process for each resource |
| `Allocation[n][m]` | Currently allocated resources to each process |
| `Need[n][m]` | Remaining resource need: Need = Max − Allocation |

**Safety Algorithm:**
```
1. Work = Available
   Finish[i] = false for all i

2. Find process i where:
   Finish[i] = false AND Need[i] ≤ Work

3. If found:
   Work = Work + Allocation[i]
   Finish[i] = true
   Go to step 2

4. If Finish[i] = true for all i → SAFE STATE
   Else → UNSAFE STATE
```

**Resource Request Algorithm:**
```
When Process i requests Resources[i]:

1. If Request[i] > Need[i] → Error (exceeded max claim)
2. If Request[i] > Available → Wait (resources not available)
3. Pretend to allocate:
   Available = Available − Request[i]
   Allocation[i] = Allocation[i] + Request[i]
   Need[i] = Need[i] − Request[i]
4. Run Safety Algorithm:
   - If SAFE → grant the request
   - If UNSAFE → rollback and make process wait
```

### Banker's Algorithm — Worked Example

**5 processes (P0–P4), 3 resource types (A, B, C)**

```
            Allocation    Max      Available
            A  B  C    A  B  C    A  B  C
P0          0  1  0    7  5  3    3  3  2
P1          2  0  0    3  2  2
P2          3  0  2    9  0  2
P3          2  1  1    2  2  2
P4          0  0  2    4  3  3
```

**Calculate Need = Max − Allocation:**
```
        Need
        A  B  C
P0      7  4  3
P1      1  2  2
P2      6  0  0
P3      0  1  1
P4      4  3  1
```

**Safety Check (Available = [3,3,2]):**
- P1 needs [1,2,2] ≤ [3,3,2] ✅ → Run P1 → Work = [3,3,2]+[2,0,0] = [5,3,2]
- P3 needs [0,1,1] ≤ [5,3,2] ✅ → Run P3 → Work = [5,3,2]+[2,1,1] = [7,4,3]
- P4 needs [4,3,1] ≤ [7,4,3] ✅ → Run P4 → Work = [7,4,3]+[0,0,2] = [7,4,5]
- P0 needs [7,4,3] ≤ [7,4,5] ✅ → Run P0 → Work = [7,5,5]+[0,1,0] = [7,5,5]
- P2 needs [6,0,0] ≤ [7,5,5] ✅ → Run P2

**Safe sequence: P1 → P3 → P4 → P0 → P2 ✅ System is SAFE**

> **Banker analogy:** A bank has limited cash. A customer (process) tells the bank their maximum loan need upfront. The bank grants loans only if doing so keeps the bank in a state where all customers can eventually be served. If a new loan request would leave the bank unable to serve other customers even in the best case, it's denied.

**Limitations of Banker's Algorithm:**
- Processes must declare maximum resource needs in advance (often unknown).
- Number of processes and resources must be fixed.
- Processes cannot exit early.
- High overhead for large systems.

---

## 10. Deadlock Detection

**Goal:** Allow deadlocks to occur, but **periodically check** for them and recover.

### 10.1 Single Instance Resources — RAG Reduction

For single-instance resources, a **cycle in the Resource Allocation Graph = deadlock**.

**Detection method:**
- Draw the RAG.
- Simplify: A process with no outstanding requests can eventually finish and release resources.
- Remove completed processes and their edges.
- If any processes remain (cannot complete) → **deadlock**.

### 10.2 Multiple Instance Resources — Detection Algorithm

Similar to Banker's Safety Algorithm but checks current state (not future):

```
1. Work = Available
   Finish[i] = false for all i (if Allocation[i] ≠ 0)

2. Find i where:
   Finish[i] = false AND Request[i] ≤ Work

3. If found:
   Work = Work + Allocation[i]
   Finish[i] = true
   Go to step 2

4. If Finish[i] = false for some i → those processes are DEADLOCKED
```

**When to run detection:**
- Every k minutes.
- When CPU utilization drops below a threshold (sign of deadlock).
- When a resource request cannot be satisfied.

---

## 11. Recovery from Deadlock

Once a deadlock is detected, the OS must **break the deadlock**.

### 11.1 Process Termination

**Option A: Abort all deadlocked processes**
- Simple but expensive — all work is lost.

**Option B: Abort one process at a time**
- Kill one deadlocked process, check if deadlock resolved, repeat if needed.
- Which process to abort? Factors:
  - Process priority.
  - How long process has been running / how close to completion.
  - How many resources the process is using.
  - How many more resources it needs.
  - How many processes must be terminated.

### 11.2 Resource Preemption

**Forcibly take resources** from deadlocked processes and give them to others.

**Issues to handle:**
1. **Selecting a victim:** Minimize cost — preempt from the process that has run least/holds least critical resources.
2. **Rollback:** Preempted process must be rolled back to a safe earlier state (checkpoint).
3. **Starvation:** Ensure the same process is not always the victim — limit the number of times a process can be rolled back.

> **Real-world example:** A database system rolling back a transaction that is part of a deadlock cycle — the other transaction proceeds, and the rolled-back transaction is restarted later.

---

## 12. Summary Table

| Approach | Method | Coffman Condition Attacked | Overhead |
|---|---|---|---|
| **Prevention** | Spooling | Mutual Exclusion | Medium |
| **Prevention** | Request all at once | Hold and Wait | High (low utilization) |
| **Prevention** | Allow preemption | No Preemption | Medium |
| **Prevention** | Resource ordering | Circular Wait | Low |
| **Avoidance** | Banker's Algorithm | All (dynamically) | Medium (needs advance info) |
| **Detection** | RAG / Detection Algorithm | — (after the fact) | Periodic overhead |
| **Ignorance** | Ostrich Algorithm | — (ignored) | None |

---

## 13. Exam Tips

Very common exam questions for Unit 4:

1. **What is a deadlock? Give a real-world example.**
2. **State and explain the four Coffman conditions with examples.**
3. **What is the Resource Allocation Graph? Draw and interpret one.**
4. **Explain the Ostrich Algorithm. When is it used?**
5. **Explain deadlock prevention for each Coffman condition.**
6. **What is a safe state? Why is it important in deadlock avoidance?**
7. **Solve Banker's Algorithm numerical — given Allocation, Max, Available; find Need, check if safe, find safe sequence.** *(Very common — must practice!)*
8. **Differentiate deadlock prevention vs deadlock avoidance vs deadlock detection.**
9. **How can a deadlock be recovered? Explain process termination and resource preemption.**

> **Key formula:** `Need[i][j] = Max[i][j] − Allocation[i][j]`
> 
> **For safe sequence:** Find a process whose Need ≤ Work, execute it, add its Allocation to Work, repeat.

---

## 14. References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12 — Chapter 6
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014 — Chapter 7
