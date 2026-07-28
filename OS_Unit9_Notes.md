# Unit 9: Distributed Operating System
**Course:** CACS251 — Operating System | **Duration:** 4 Hours
**Year/Semester:** BCA II/IV | Tribhuvan University

---

## Syllabus Topics
Introduction, Advantages of Distributed System over Centralized System, Advantages of Distributed System over Independent PCs, Disadvantages of Distributed System, Hardware and Software Concepts, Communication in Distributed Systems, Message Passing, Remote Procedure Call, Process in Distribution System, Clock Synchronization.

---

## 1. Introduction to Distributed Operating System

A **Distributed Operating System** manages a collection of **independent computers** and makes them appear to users as a **single coherent system**.

```
┌──────────────────────────────────────────────────────────────┐
│              DISTRIBUTED SYSTEM                               │
│                                                              │
│  [Computer 1] ──── Network ──── [Computer 2]                │
│       │                              │                       │
│  [Computer 3] ──────────────── [Computer 4]                 │
│                                                              │
│  To the USER it looks like ONE powerful computer!            │
└──────────────────────────────────────────────────────────────┘
```

**Key definition:** A distributed system is one in which components located on **networked computers** communicate and coordinate their actions by **passing messages**.

**Three types of systems:**

| Type | Description | Example |
|---|---|---|
| **Centralized** | One powerful computer, all users connect to it | Mainframe, University server |
| **Distributed** | Multiple computers appear as one system | Google, Amazon cloud |
| **Independent PCs** | Multiple computers, no coordination | Office computers without sharing |

> **Real-world example:** When you search on Google, your query is processed by **thousands of servers** distributed across multiple data centers worldwide — but to you it feels like one fast system. That is a distributed OS in action.

---

## 2. Network Operating System vs Distributed Operating System

**Network Operating System (NOS):**
- Users are **aware** of multiple machines.
- Users must explicitly log into remote machines.
- Users explicitly transfer files between machines.
- Each machine has its own local OS.
- Example: Windows Server with shared folders — you know files are on "\\server\share".

**Distributed Operating System:**
- Users are **unaware** of multiple machines (transparent).
- Resources accessed transparently regardless of location.
- System appears as one unified computer.
- Example: Google's infrastructure — you don't know which server ran your search.

```
NOS:                          Distributed OS:
User knows "file is on         User just says "open file"
server2, share=docs"           System finds it transparently
                               across hundreds of machines
```

---

## 3. Advantages of Distributed System over Centralized System

### 3.1 Economics
- Multiple cheap microprocessors offer **better price/performance** ratio than one expensive mainframe.
- Cost of adding more machines is much lower than upgrading one central machine.

### 3.2 Speed — Higher Computing Power
- A distributed system can have **more total CPU power** than any single mainframe.
- Tasks can be divided and processed **in parallel** across many CPUs.

```
Single CPU (centralized):     4 CPUs (distributed):
Task time = 100ms             Task time ≈ 25ms (each CPU does 1/4)
```

> **Example:** Weather forecasting models that require enormous computation — split across thousands of CPUs in parallel.

### 3.3 Reliability / Fault Tolerance
- If one machine fails, **others continue working** — no single point of failure.
- System keeps running even when some components fail.
- **Replication:** Critical data kept on multiple machines simultaneously.

```
Centralized:                  Distributed:
┌───────────┐                 [A] [B] [C] [D]
│  Server   │  fails →         ↓   ↓   ↓   ↓
│ (ONE SPOF)│  ALL DOWN!       If [A] fails → B, C, D continue
└───────────┘
```

> **Example:** Google stores every file on **three different servers** in different locations. If one data center burns down, files are still safe and accessible.

### 3.4 Geographical Distribution
- Components can be placed **where needed geographically**.
- A bank can have servers in Kathmandu, Pokhara, and Biratnagar — customers in each city served locally (faster).
- Data can be kept near the users who need it.

### 3.5 Incremental Growth
- Computing power can be **added gradually** by adding more machines.
- No need to replace the entire system — just add nodes.
- Scales easily with growing demand.

---

## 4. Advantages of Distributed System over Independent PCs

### 4.1 Data Sharing
- Users can **share data** (files, databases) with each other.
- No need to physically transfer files via USB.
- One authoritative copy of data accessible to all.

> **Example:** A company database — all employees access the same customer records from their own computers.

### 4.2 Device Sharing
- Expensive peripherals (printer, scanner, plotter) can be **shared** among many users.
- No need for each computer to have its own printer.

> **Example:** One network printer shared by 20 office computers — much cheaper than 20 individual printers.

### 4.3 Communication
- Users can **communicate** via email, messaging, video conferencing.
- Applications can communicate across the network.

### 4.4 Flexibility — Load Distribution
- Work can be done on **the most appropriate machine**.
- If one computer is overloaded, tasks can be moved to an idle computer.

```
Before load balancing:       After load balancing:
PC1: 90% CPU busy            PC1: 45% CPU
PC2: 10% CPU idle            PC2: 45% CPU
→ PC1 slow, PC2 wasted       → Both efficient
```

---

## 5. Disadvantages of Distributed Systems

### 5.1 Software Complexity
- Writing **distributed software is much harder** than centralized software.
- Must handle network failures, partial failures, concurrency, synchronization.
- Less mature than centralized OS software.

### 5.2 Networking Issues
- Performance depends on **network reliability and speed**.
- Network can become a bottleneck.
- Network failure can isolate parts of the system.
- **Latency** (delay) over network is much higher than memory access.

### 5.3 Security Concerns
- **More attack surface** — data travels over network.
- Interception (eavesdropping) of network traffic.
- Unauthorized access from remote machines.
- Harder to establish trust between machines.

### 5.4 Data Consistency
- When data is replicated across machines, keeping all copies **consistent** is challenging.
- If one copy is updated and another is not (due to network failure), which is correct?
- **CAP Theorem:** A distributed system can guarantee only 2 of 3: Consistency, Availability, Partition tolerance.

### 5.5 High Cost of Initial Setup
- Network infrastructure (cables, switches, routers).
- Middleware software licenses.
- Skilled administrators to manage the system.

### 5.6 Difficulty in Management
- Hard to monitor all machines.
- Hard to debug problems (failure could be on any node).
- Software updates must be rolled out across all machines.

---

## 6. Hardware Concepts in Distributed Systems

### 6.1 Processor Coupling

**Tightly Coupled Systems:**
- Processors share the **same memory** (shared memory multiprocessor).
- Communication via shared variables.
- Low latency communication.
- Example: Multi-core processors in a single computer.

```
┌─────────────────────────────────┐
│  CPU1   CPU2   CPU3   CPU4      │
│    ↓      ↓      ↓      ↓       │
│        Shared Memory            │
└─────────────────────────────────┘
```

**Loosely Coupled Systems:**
- Each processor has **its own memory**.
- Communication via **message passing** over a network.
- Higher latency but more scalable.
- Example: A cluster of computers connected by Ethernet.

```
┌──────────┐         ┌──────────┐
│ CPU + RAM│◄──net──►│ CPU + RAM│
└──────────┘         └──────────┘
```

### 6.2 Network Topology Types

**LAN (Local Area Network):**
- Covers a small area (room, building, campus).
- High speed (1Gbps–10Gbps), low error rate.
- Example: University campus network.

**MAN (Metropolitan Area Network):**
- Covers a city or metropolitan area.
- Medium speed, connects multiple LANs.
- Example: City-wide cable TV network.

**WAN (Wide Area Network):**
- Covers large geographic area (country, world).
- Lower speed, higher latency than LAN.
- Example: The Internet.

**Common Topologies:**

```
Bus:          Star:          Ring:
A-B-C-D-E    A   B            A
              ↘ ↙             / \
              Hub             D   B
              ↗ ↖             \ /
              C   D             C

Mesh: Every node connected to every other node
(most fault tolerant — internet routing)
```

---

## 7. Software Concepts in Distributed Systems

### 7.1 Middleware

**Middleware** is software that sits **between the OS and distributed applications**, providing services that make distributed computing easier.

```
┌─────────────────────────────────────┐
│          Application                │
├─────────────────────────────────────┤
│          Middleware                 │  ← Hides distribution complexity
│  (RPC, message queues, naming, etc) │
├─────────────────────────────────────┤
│     Operating System + Network      │
├─────────────────────────────────────┤
│          Hardware                   │
└─────────────────────────────────────┘
```

**Middleware provides:**
- **Naming:** How to find resources across the network.
- **Communication:** How programs talk to each other (RPC, messaging).
- **Synchronization:** Coordinating actions across machines.
- **Replication:** Keeping copies consistent.
- **Security:** Authentication across machines.

**Examples of middleware:** CORBA, Java RMI, Microsoft DCOM, REST APIs, message queues (RabbitMQ, Kafka).

### 7.2 Transparency in Distributed Systems

A key goal — hiding the complexity of distribution from users:

| Transparency Type | What it hides |
|---|---|
| **Location** | Where a resource physically is |
| **Access** | How a resource is accessed (local vs remote) |
| **Migration** | Resource moved to another location |
| **Replication** | Multiple copies exist |
| **Concurrency** | Multiple users sharing a resource |
| **Failure** | Faults are hidden and recovered automatically |

---

## 8. Communication in Distributed Systems

Since processes in a distributed system run on different machines, they must **communicate over a network**.

### 8.1 Message Passing

The fundamental communication mechanism — one process **sends a message**, another **receives it**.

**Basic operations:**
```
send(destination, message)    → Send message to another process
receive(source, &message)     → Receive message from a process
```

**Synchronous (Blocking) Message Passing:**
- **send()** blocks until receiver has received the message.
- **receive()** blocks until a message arrives.
- Simple but can cause deadlock if both sides wait for each other.

```
Process A:                    Process B:
send(B, "Hello")  ──────►    receive(A, msg)
(blocks until B receives)     (blocks until A sends)
```

**Asynchronous (Non-blocking) Message Passing:**
- **send()** returns immediately, message buffered.
- Sender continues without waiting.
- More complex but better performance.

```
Process A:                    Process B:
send(B, "Hello")             ... (doing other work) ...
(returns immediately)         receive(A, msg)  → gets buffered message
(continue other work)
```

**Message Passing Issues:**
- **Lost messages:** Network can drop packets → need acknowledgment + retry.
- **Duplicate messages:** Retry can cause duplicates → need sequence numbers.
- **Out-of-order delivery:** Messages may arrive in wrong order → need ordering.
- **Authentication:** How does receiver know message is from legitimate sender?

> **Real-world example:** Email is asynchronous message passing — you send an email (send() returns immediately), recipient reads it later (whenever they check). WhatsApp messages are also asynchronous — message queued on server until recipient is online.

---

### 8.2 Remote Procedure Call (RPC)

**RPC** allows a program to **call a procedure (function) on another computer** as if it were a local function call. The network communication is hidden from the programmer.

```
WITHOUT RPC (manual):                 WITH RPC (transparent):
client: format message                client: result = add(3, 4)
client: send to server                ← looks like a local call!
server: receive message
server: parse parameters
server: execute add(3, 4)
server: format result
server: send back
client: receive result
client: parse result
→ Very complex!                       → Simple!
```

**How RPC works — Step by Step:**

```
Client Machine:                       Server Machine:

1. Client calls: result = add(3, 4)
   ↓
2. Client stub intercepts the call
   - Packages parameters into message (marshalling)
   - Sends message over network
   ↓ ─────────────────────────────────► 3. Message arrives at server
                                          ↓
                                       4. Server stub receives message
                                          - Unpacks parameters (unmarshalling)
                                          - Calls actual add(3, 4)
                                          ↓
                                       5. Server executes: result = 7
                                          ↓
                                       6. Server stub packs result
                                          - Sends result back
◄──────────────────────────────────── 7. Result arrives at client
↓
8. Client stub returns 7 to caller
   result = 7   ← Client sees just this!
```

**Key terms:**
- **Stub:** Proxy code that handles the network communication (generated automatically from IDL).
- **Marshalling (Serialization):** Packing parameters into a transmittable format.
- **Unmarshalling (Deserialization):** Unpacking received data back into parameters.
- **IDL (Interface Definition Language):** Defines the interface of remote procedures.

**RPC Issues:**
- **What if server crashes?** Client waiting forever.
- **What if network fails mid-call?** Did the call execute?
- **What if call takes too long?** Need timeout mechanism.
- **Different data representations:** Big-endian vs little-endian, integer sizes.

> **Real-world example:** When your smartphone app loads weather data, it makes an API call to a remote server — this is essentially RPC. The app calls `getWeather("Pokhara")` and gets back weather data, without knowing the server is thousands of kilometers away.

**Modern RPC frameworks:**
- **gRPC** (Google) — used by Netflix, Square, Cisco.
- **Apache Thrift** (Facebook) — used by Facebook, Twitter.
- **XML-RPC** — older web services.
- **REST API** — HTTP-based, most common web API style.

---

### 8.3 Process in Distribution System

In a distributed system, processes on different machines must **cooperate** to complete tasks.

**Process Migration:**
- Moving a running process from one machine to another.
- Reasons: Load balancing, fault tolerance, energy saving.
- Requires transferring process state, memory, open files.

**Remote Process Creation:**
- A process on machine A creates a new process on machine B.
- Used for distributing workload.

**Process Groups:**
- Processes organized into groups for collective operations.
- **Broadcast:** Send message to all processes in group.
- **Multicast:** Send to specific subset.
- Example: All replica servers in a database cluster form a group.

**Distributed Shared Memory (DSM):**
- Gives illusion of shared memory across machines with separate physical memory.
- Processes access "shared" variables — system transparently transfers pages between machines.

---

## 9. Clock Synchronization

In a distributed system, each machine has its own physical clock. These clocks **drift apart** over time because:
- Oscillators run at slightly different speeds.
- Temperature and voltage variations.
- No central clock to synchronize with.

**Why clock synchronization matters:**
- **File systems:** If machine A's clock is ahead, a file modified "after" creation looks older.
- **Distributed transactions:** Need to determine order of events.
- **Security:** Certificates and tokens have expiry times.
- **Logging:** Event logs must be in correct order for debugging.

```
Problem:
Machine A clock: 10:00:05
Machine B clock: 10:00:03   ← 2 seconds behind
Machine C clock: 10:00:07   ← 2 seconds ahead

Same event happens at "10:00:05" on all machines
but each machine records a different time!
```

### 9.1 Physical Clock Synchronization

**Goal:** Synchronize all clocks to the same real time (UTC).

**NTP (Network Time Protocol):**
- Standard protocol for clock synchronization over networks.
- Hierarchical structure (strata):
  - **Stratum 0:** Atomic clocks, GPS receivers (ultra-precise reference).
  - **Stratum 1:** Servers directly connected to Stratum 0.
  - **Stratum 2:** Servers that synchronize with Stratum 1.
  - ...and so on.

```
Stratum 0: [Atomic Clock / GPS]
               ↓
Stratum 1: [NTP Primary Server] (±microseconds accuracy)
               ↓
Stratum 2: [NTP Secondary Server] (±milliseconds accuracy)
               ↓
Stratum 3: [Your computer / phone]
```

**Cristian's Algorithm (Simple Clock Sync):**

Client asks time server for current time:
```
Step 1: Client sends TIME REQUEST at time T0
Step 2: Server receives request, reads its clock (Tserver)
Step 3: Server sends reply at time Tserver
Step 4: Client receives reply at time T1

Estimated network delay = (T1 - T0) / 2
Client sets its clock = Tserver + (T1 - T0) / 2

(accounts for the time the message spent in transit)
```

**Problem:** Clock cannot be set backward (this can break software). Instead, the clock is **slowed down or sped up** gradually to converge.

### 9.2 Logical Clocks — Lamport Timestamps

**Problem:** We do not always need real time — we just need to know the **order of events**.

**Lamport's insight:** If event A **causes** event B (A happens before B), then the timestamp of A must be less than the timestamp of B.

**Lamport's "happens-before" relation (→):**
1. If A and B are events in the same process and A occurs before B: A → B.
2. If A is a message send and B is its receipt: A → B.
3. Transitivity: If A → B and B → C then A → C.

**Lamport Timestamp Rules:**
```
Each process maintains counter C:
1. Before each event, increment C: C = C + 1
2. When sending a message: attach current C as timestamp
3. When receiving a message with timestamp Cm:
   C = max(C, Cm) + 1
```

**Example:**
```
Process P1:  C=1(a) ──── send(m1, C=1) ────────────────────────
                                              ↘
Process P2:  C=1(b)   C=2(c)    receive(m1→C=max(2,1)+1=3)(d)
                                                               ↘
Process P3:                                     C=1(e)   receive(m2→C=max(1,3)+1=4)(f)
```

**Result:** Events have consistent ordering — no two causally related events have the same timestamp.

**Limitation:** Lamport timestamps give a partial order. Two unrelated events may have inconsistent timestamps between processes.

### 9.3 Vector Clocks

**Extension of Lamport clocks** that captures **causality** more precisely.

- Each process i maintains a **vector** V[1..n] (one entry per process).
- V[i] = number of events process i has executed.

**Rules:**
```
1. Initially, V[i] = 0 for all i
2. Before each event, increment own entry: Vi[i] = Vi[i] + 1
3. Send message: attach current vector as timestamp
4. Receive message with vector Vm:
   V[j] = max(V[j], Vm[j]) for all j
   V[i] = V[i] + 1
```

**Comparing vector timestamps:**
- V1 < V2 (V1 happened before V2) if every entry of V1 ≤ corresponding entry of V2, and at least one entry is strictly less.
- If neither V1 < V2 nor V2 < V1 → events are **concurrent** (causally unrelated).

> **Real-world example:** **Git version control** uses vector-clock-like mechanisms to detect conflicts. When two developers edit the same file independently (concurrent events), Git detects the conflict because neither change "happened before" the other.

---

## 10. Summary

### Distributed vs Centralized vs Independent PCs

| Feature | Centralized | Distributed | Independent PCs |
|---|---|---|---|
| Architecture | One central computer | Multiple interconnected | Multiple, no connection |
| Appearance to user | Single machine | Single machine | Multiple separate |
| Fault tolerance | Low (single point) | High (redundancy) | N/A (independent) |
| Data sharing | All on one machine | Shared across network | Manual transfer needed |
| Scalability | Limited | High (add more nodes) | N/A |
| Complexity | Low | High | Low |
| Example | Mainframe | Google, Amazon | Home office computers |

### Communication Methods Summary

| Method | Coupling | Use Case | Example |
|---|---|---|---|
| **Message Passing** | Loose | General IPC across network | Email, chat, queues |
| **RPC** | Tight (call-return) | Function call semantics | API calls, microservices |
| **Shared Memory** | Very tight | Same machine or DSM | Multi-threaded programs |

### Clock Synchronization Summary

| Method | Type | Accuracy | Use Case |
|---|---|---|---|
| **NTP** | Physical | Milliseconds | Internet time sync |
| **Cristian's Algorithm** | Physical | Network-limited | Simple client-server |
| **Lamport Timestamps** | Logical | Event ordering only | Distributed events |
| **Vector Clocks** | Logical | Causality detection | Version control, DBs |

---

## 11. Exam Tips

Commonly asked questions for Unit 9:

1. **What is a Distributed Operating System? How is it different from a Network OS?**
2. **What are the advantages of a distributed system over a centralized system?**
3. **What are the advantages of a distributed system over independent PCs?**
4. **What are the disadvantages of distributed systems?**
5. **What is middleware? What services does it provide?**
6. **Explain message passing in distributed systems. Difference between synchronous and asynchronous.**
7. **What is RPC (Remote Procedure Call)? Explain how it works step by step with a diagram.**
8. **What is marshalling and unmarshalling in RPC?**
9. **Why is clock synchronization needed in distributed systems?**
10. **Explain Cristian's Algorithm for clock synchronization.**
11. **What are Lamport Timestamps? Explain the happens-before relation.**
12. **What are Vector Clocks? How do they improve on Lamport timestamps?**
13. **Explain the CAP theorem.**
14. **What is transparency in distributed systems? List any four types.**

> **Key distinctions:**
> - **NOS:** User aware of multiple machines | **DOS:** Transparent, appears as one.
> - **Synchronous RPC:** Caller blocks | **Asynchronous RPC:** Caller continues.
> - **Lamport clocks:** Only ordering | **Vector clocks:** Causality detection.
> - **Physical clock:** Real time synchronization | **Logical clock:** Event ordering only.

---

## 12. References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12 — Chapter 8
- Andrew S. Tanenbaum, *Distributed Operating System*, Pearson
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014 — Chapter 17
