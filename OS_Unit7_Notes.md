# Unit 7: File System Interface Management
**Course:** CACS251 — Operating System | **Duration:** 2 Hours
**Year/Semester:** BCA II/IV | Pokhara University

---

## Syllabus Topics
File Concept: File Naming, File Structure, File Type, File Access, File Attributes, File Operation and File Descriptors; Directories: Single-Level Directory Systems, Hierarchical Directory Systems, Path Names, Directory Operation; Access Methods: Sequential, Direct; Protection: Types of Access, Access Control List, Access Control Matrix.

---

## 1. Introduction — File System

A **file system** is the OS module that manages the storage, retrieval, and organization of data on storage devices (disk, SSD, USB).

**Why do we need a file system?**
- Without a file system, a disk is just a flat array of raw sectors — no organization.
- File system provides **named, structured access** to data.
- Handles persistence — data survives after the process ends.

**Responsibilities of File System:**
- Creating, reading, writing, and deleting files.
- Organizing files into directories.
- Managing free disk space.
- Controlling access (who can read/write which file).
- Ensuring data integrity.

> **Real-world example:** Your `/home/mukesh/Downloads/report.pdf` is managed by the file system — it knows which disk sectors contain that file, who owns it, when it was modified, and who is allowed to read it.

---

## 2. File Overview and Concept

A **file** is a **named collection of related data** stored on secondary storage (disk).

From the OS perspective, a file is the smallest logically addressable storage unit.

**From user perspective:** Documents, photos, programs, videos.
**From OS perspective:** A sequence of bytes organized and named by the file system.

---

## 3. File Naming

**File naming** rules define what names are valid for files.

**Components of a file name:**
- **Name:** Human-readable identifier (e.g., `report`)
- **Extension:** Type indicator after a dot (e.g., `.pdf`, `.txt`, `.exe`)

**Naming rules vary by OS:**

| OS | Rules | Example |
|---|---|---|
| Windows (NTFS) | Case-insensitive, max 255 chars, no `\ / : * ? " < > \|` | `My Document.docx` |
| Linux (ext4) | Case-sensitive, max 255 bytes, no `/` or null | `my_document.txt` ≠ `My_Document.txt` |
| Old FAT (DOS) | 8.3 format — 8 char name + 3 char extension | `REPORT  .TXT` |

**Extension conventions:**

| Extension | Type |
|---|---|
| `.c`, `.py`, `.java` | Source code |
| `.exe`, `.out` | Executable |
| `.txt`, `.md` | Text document |
| `.pdf`, `.docx` | Document |
| `.jpg`, `.png` | Image |
| `.mp3`, `.wav` | Audio |
| `.tar`, `.zip` | Archive |

---

## 4. File Structure

Files can be organized internally in three ways:

### 4.1 Byte Sequence (Unstructured)
- File is simply a **sequence of bytes** — no internal structure.
- OS does not interpret the content.
- Application decides how to interpret bytes.
- **Most common in Unix/Linux** — everything is a byte stream.

```
File = [b0][b1][b2][b3][b4]...[bn]
        ↑
  Any byte can be accessed by position
```

**Example:** A `.txt` file is just bytes; the text editor interprets them as characters.

### 4.2 Record Sequence
- File is a **sequence of fixed-size or variable-size records**.
- Each record is one logical unit (e.g., one employee entry).
- Used in older systems and databases.

```
File = [Record 1][Record 2][Record 3]...[Record n]
        [Name|Age|Salary][Name|Age|Salary]...
```

**Example:** Employee database where each record = one employee's data.

### 4.3 Tree Structure
- File is organized as a **tree of records** (like a B-tree).
- Each record has a **key field**.
- OS manages the tree structure; records fetched by key.
- Used in mainframe systems and some database files.

```
           [Key: M]
          /         \
    [Key: E]        [Key: S]
   /      \        /      \
[Emp1]  [Emp2]  [Emp3]  [Emp4]
```

---

## 5. File Types

File types tell the OS and applications what kind of data a file contains.

| Type | Description | Examples |
|---|---|---|
| **Regular Files** | User data — text or binary | `.txt`, `.jpg`, `.exe` |
| **Directory Files** | Contains list of files | `/home`, `/etc` |
| **Character Special Files** | Character devices | `/dev/keyboard`, `/dev/tty` |
| **Block Special Files** | Block devices | `/dev/sda` (disk) |
| **Symbolic Links** | Pointer to another file | Shortcuts/aliases |
| **Pipes** | Inter-process communication | Named pipes (`mkfifo`) |
| **Sockets** | Network communication endpoints | Network sockets |

**How OS identifies file type:**
- **Extension** (Windows primarily): `.exe`, `.dll`, `.txt`
- **Magic number** (Unix/Linux): First bytes of file contain a signature
  - ELF executable: `7F 45 4C 46` (ELF)
  - JPEG image: `FF D8 FF`
  - PDF: `25 50 44 46` (%PDF)

---

## 6. File Access Methods

### 6.1 Sequential Access
- Records/bytes read or written **one after another** in order.
- Cannot skip forward without reading everything in between.
- **Simplest access method**.
- Good for: tape drives, log files, streaming.

```
Read: [Rec1] → [Rec2] → [Rec3] → [Rec4] → ... (forward only)
Write: append to end

Operations: read_next(), write_next(), reset() (go back to start)
```

```c
// Sequential access in C:
FILE *f = fopen("file.txt", "r");
char c;
while ((c = fgetc(f)) != EOF) {
    process(c);   // read each character sequentially
}
```

**Example:** Reading a log file line by line. Playing an audio/video file.

### 6.2 Direct Access (Random Access)
- Any record can be **read or written directly** by its position number.
- No need to read preceding records.
- Requires fixed-size records (to calculate offset).
- Good for: databases, indexed files.

```
Direct access to Record n:
Offset = n × record_size
Seek to offset → Read/Write

File: [Rec0][Rec1][Rec2][Rec3][Rec4]...[RecN]
                    ↑
              Read Rec2 directly: offset = 2 × record_size
```

```c
// Direct access in C:
FILE *f = fopen("data.bin", "rb");
int n = 5;  // Want record 5
fseek(f, n * sizeof(Record), SEEK_SET);  // Jump directly
fread(&record, sizeof(Record), 1, f);    // Read it
```

**Example:** Database looking up employee by ID. Array-based file access.

---

## 7. File Attributes

**File attributes** (also called **metadata**) are properties stored about a file by the file system.

| Attribute | Description | Example |
|---|---|---|
| **Name** | Human-readable file name | `report.pdf` |
| **Identifier** | Unique number (inode) | `inode 12345` |
| **Type** | Regular, directory, link, etc. | `regular` |
| **Location** | Disk address (first block) | Sector 4096 |
| **Size** | Current size in bytes | `2,048,576 bytes` |
| **Protection** | Access permissions | `rwxr-xr--` |
| **Owner** | Who owns the file | `mukesh` |
| **Time stamps** | Created, modified, accessed | `2025-06-01 10:30` |
| **Link count** | Number of directory entries pointing to file | `2` |
| **Hidden** | Whether file is hidden | `false` |
| **Read-only** | Whether file can be written | `false` |

```bash
# View file attributes in Linux:
ls -la report.pdf
-rw-r--r-- 1 mukesh staff 2048576 Jun 1 10:30 report.pdf
↑ Type+Permissions, Links, Owner, Group, Size, Modified, Name
```

---

## 8. File Operations

The OS provides these **system calls** for file operations:

| Operation | Description | C System Call |
|---|---|---|
| **Create** | Create a new empty file | `creat()`, `open(O_CREAT)` |
| **Open** | Open existing file, get file descriptor | `open()` |
| **Read** | Read bytes from file | `read()` |
| **Write** | Write bytes to file | `write()` |
| **Seek** | Move file position pointer | `lseek()` |
| **Close** | Release file descriptor | `close()` |
| **Delete** | Remove file (unlink) | `unlink()` |
| **Truncate** | Shorten file to given size | `truncate()` |
| **Append** | Write at end of file | `open(O_APPEND)` |
| **Rename** | Change file name | `rename()` |

### File Descriptors
A **file descriptor** (fd) is a small non-negative integer that the OS uses to identify an open file within a process.

```
After open("file.txt"):
  fd = 3   (0=stdin, 1=stdout, 2=stderr are already taken)

Per-process open file table:
┌───┬──────────────────────────────────────────┐
│ 0 │ stdin  (keyboard)                         │
│ 1 │ stdout (terminal)                         │
│ 2 │ stderr (terminal)                         │
│ 3 │ file.txt → points to inode 12345          │
│ 4 │ another_file.txt → inode 67890            │
└───┴──────────────────────────────────────────┘
```

**Standard File Descriptors:**
- **0 (stdin):** Standard input — keyboard by default.
- **1 (stdout):** Standard output — terminal by default.
- **2 (stderr):** Standard error — terminal by default.

```c
// File operations in C:
int fd = open("data.txt", O_RDWR | O_CREAT, 0644);  // Open/create
char buf[100];
int bytes_read = read(fd, buf, 100);                  // Read
write(fd, "Hello\n", 6);                              // Write
lseek(fd, 0, SEEK_SET);                               // Seek to beginning
close(fd);                                             // Close
```

---

## 9. Directory Structure

A **directory** (folder) is a special file that contains a **list of files and sub-directories**.

**Directory entry** contains at minimum:
- File name
- Pointer to file attributes (inode number in Unix)

### 9.1 Single-Level Directory

All files in the **entire system** are in one directory — no subdirectories.

```
Root Directory:
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ file1.txt│ prog.exe │ photo.jpg│ report.pdf│ data.csv │
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

**Advantages:** Very simple to implement.
**Disadvantages:**
- Name conflicts — two users cannot have files with same name.
- Difficult to find files as number grows.
- No organization — poor for multi-user systems.

> **Example:** Early MS-DOS on floppy disks. Embedded systems with simple file storage.

### 9.2 Two-Level Directory

Separate **directory for each user** — solves name conflict problem.

```
Master Directory:
┌──────────────────────────────────┐
│  mukesh/ │  ram/ │  sita/ │ root/│
└──────────────────────────────────┘
     ↓           ↓          ↓
┌─────────┐  ┌─────────┐  ┌─────────┐
│file1.txt│  │file1.txt│  │data.csv │
│prog.c   │  │photo.jpg│  │notes.txt│
└─────────┘  └─────────┘  └─────────┘
```

**Advantages:** Name conflicts between users solved. Simple.
**Disadvantages:** Users cannot create subdirectories. No sharing between users. Limited organization.

### 9.3 Hierarchical Directory (Tree Structure)

The **standard structure** in modern OS — tree of directories allowing arbitrary nesting.

```
/ (root)
├── home/
│   ├── mukesh/
│   │   ├── documents/
│   │   │   ├── report.pdf
│   │   │   └── notes.txt
│   │   └── photos/
│   │       └── vacation.jpg
│   └── ram/
│       └── code/
│           └── main.c
├── etc/
│   └── os-release
├── usr/
│   └── bin/
│       └── ls
└── tmp/
    └── temp.txt
```

**Advantages:**
- Unlimited nesting depth.
- Excellent organization.
- Multiple users with complete isolation.
- Easy to find files through paths.

**Used by:** All modern OS — Windows (C:\), Linux (/), macOS (/).

### 9.4 File System Layout

The disk is organized into a file system with these regions:

```
┌──────────┬──────────┬───────────┬────────────────────────┐
│  Boot    │  Super   │  Inode    │      Data Blocks        │
│  Block   │  Block   │  Table    │  (file contents)        │
└──────────┴──────────┴───────────┴────────────────────────┘

Boot Block: Contains bootloader code to start the OS
Super Block: File system metadata (size, free blocks, inode count)
Inode Table: Array of inodes (one per file, stores metadata)
Data Blocks: Actual file content stored here
```

---

## 10. Path Names

A **path name** is the full specification of a file's location in the directory tree.

### 10.1 Absolute Path
- Starts from the **root directory** (`/` in Linux, `C:\` in Windows).
- Complete path from root to file.

```
Linux:   /home/mukesh/documents/report.pdf
Windows: C:\Users\Mukesh\Documents\report.pdf
```

### 10.2 Relative Path
- Starts from the **current working directory**.
- Does not start with `/`.

```
If current directory is /home/mukesh/:
  Relative path: documents/report.pdf
  (Same file as /home/mukesh/documents/report.pdf)

Special path names:
  .   = current directory
  ..  = parent directory
  ~   = home directory (Linux)

Examples:
  ../ram/code/main.c   = go up one level, then into ram/code/
  ./test.txt           = test.txt in current directory
```

---

## 11. Directory Operations

| Operation | Description | Linux Command |
|---|---|---|
| **Create directory** | Make a new directory | `mkdir dirname` |
| **Delete directory** | Remove an empty directory | `rmdir dirname` |
| **Open directory** | Begin reading directory entries | `opendir()` |
| **Close directory** | Finish reading directory | `closedir()` |
| **Read directory** | Get next entry in directory | `readdir()` |
| **Rename** | Change directory name | `mv oldname newname` |
| **Link** | Create directory entry pointing to a file | `ln` |
| **Unlink** | Remove directory entry | `unlink()` |
| **Change directory** | Set current working directory | `cd dirname` |
| **List directory** | Show contents | `ls` |

---

## 12. Implementing Files

How does the file system store file data on disk?

### 12.1 Contiguous Allocation

Each file occupies a **set of contiguous blocks** on disk.

```
Directory entry: file.txt → start=5, length=4

Disk blocks:
0    1    2    3    4    5    6    7    8    9    10
[OS] [OS] [OS] [OS] [free][F1 ][F1 ][F1 ][F1 ][free][free]
                          └────── file.txt (4 blocks) ──────┘
```

**Advantages:**
- Very fast sequential read (no seeking between blocks).
- Direct access easy — block n at disk_start + n.

**Disadvantages:**
- External fragmentation — free space scattered.
- File cannot grow beyond allocated space without moving.
- Must know file size in advance.

> **Example:** CD-ROM and DVD file systems (ISO 9660) use contiguous allocation since files are written once and never modified.

### 12.2 Linked List Allocation

Each block contains **a pointer to the next block** in the file. Last block pointer = NULL.

```
Directory: file.txt → start=5

Block 5:  [data][→ block 8]
Block 8:  [data][→ block 3]
Block 3:  [data][→ block 11]
Block 11: [data][→ NULL]      (end of file)
```

**Advantages:**
- No external fragmentation.
- File can grow dynamically.
- No need to know file size in advance.

**Disadvantages:**
- Random access is slow (must follow chain of pointers).
- Pointer takes space in each block.
- Reliability — broken pointer loses rest of file.

### 12.3 Linked List Allocation Using Table in Memory (FAT)

**File Allocation Table (FAT):** Pointers stored in a **separate table in memory** (not in data blocks).

```
FAT Table (in RAM):        Disk Data Blocks:
┌─────┬───────┐            0: [OS data]
│  0  │  OS   │            1: [OS data]
│  1  │  OS   │            2: [file.txt data]
│  2  │   7   │ ←────────  3: [other.txt data]
│  3  │  EOC  │            ...
│  4  │  free │            7: [file.txt data]
│  5  │  free │            ...
│  6  │  free │            9: [file.txt data]
│  7  │   9   │
│  8  │  free │
│  9  │  EOC  │  (End of Chain)
└─────┴───────┘

file.txt starts at block 2: 2 → 7 → 9 → EOC
```

**Advantages:**
- Fast random access — scan FAT in memory (no disk read for chain).
- No space wasted in data blocks for pointers.

**Disadvantages:**
- FAT must be kept in RAM — large for big disks.
- FAT size proportional to disk size.

> **Real-world:** **FAT32** (File Allocation Table) is used in USB drives and SD cards. It's compatible with all major OS.

### 12.4 Index Allocation (I-node based)

Each file has an **index block (i-node)** containing pointers to all its data blocks.

```
I-node for file.txt:
┌──────────────────────────────┐
│ File attributes (metadata)   │
│ Size, permissions, timestamps│
├──────────────────────────────┤
│ Direct pointer 0  → Block 5  │  (first 12 blocks direct)
│ Direct pointer 1  → Block 8  │
│ Direct pointer 2  → Block 3  │
│ ...                          │
│ Single indirect   → Block 15 │  Block 15 contains more pointers
│ Double indirect   → Block 20 │  Block 20 → block of pointers → blocks
│ Triple indirect   → Block 25 │  For very large files
└──────────────────────────────┘
```

**Multi-level indexing** for large files:
```
Direct blocks (12): 12 × 4KB = 48KB
Single indirect (1): 1024 pointers × 4KB = 4MB
Double indirect (1): 1024 × 1024 × 4KB = 4GB
Triple indirect (1): 1024³ × 4KB = 4TB
```

**Advantages:**
- Fast direct access — jump directly to any block via i-node.
- No external fragmentation.
- Supports large files.

**Disadvantages:**
- Small files waste space (i-node overhead).
- Very large files need multi-level indexing (multiple disk reads).

> **Real-world:** **Unix/Linux ext2/ext3/ext4** file systems use i-nodes. Every file has exactly one i-node. `ls -i` shows i-node numbers in Linux.

---

## 13. I-nodes

An **i-node** (index node) stores all metadata about a file **except its name**.

```
I-node structure:
┌─────────────────────────────────────┐
│ File type (regular, dir, link...)   │
│ Permissions (rwxrwxrwx)             │
│ Owner UID and GID                   │
│ File size (bytes)                   │
│ Timestamps (created, modified, accessed) │
│ Hard link count                     │
│ 12 direct block pointers            │
│ 1 single-indirect pointer           │
│ 1 double-indirect pointer           │
│ 1 triple-indirect pointer           │
└─────────────────────────────────────┘
```

**Key point:** A file's name is stored in its **directory entry** (not in the i-node). Multiple directory entries (hard links) can point to the same i-node.

---

## 14. Directory Implementation

How does the OS implement directories internally?

### 14.1 Linear List
- Directory = **simple list** of (filename, inode/attributes) pairs.
- To find a file: scan the list sequentially.

```
Directory /home/mukesh/:
┌─────────────────┬──────────┐
│   Filename      │  Inode # │
├─────────────────┼──────────┤
│ report.pdf      │   1234   │
│ notes.txt       │   5678   │
│ photo.jpg       │   9012   │
│ code.py         │   3456   │
└─────────────────┴──────────┘
```

**Advantages:** Simple to implement.
**Disadvantages:** Slow search — O(n) for n files. Poor for large directories.

### 14.2 Hash Table
- Directory entries stored in a **hash table** keyed by filename.
- File lookup: hash(filename) → direct access to entry.

```
Hash Table (size 8):
┌───┬────────────────┬──────────┐
│ 0 │                │          │
│ 1 │ notes.txt      │   5678   │
│ 2 │ report.pdf     │   1234   │
│ 3 │                │          │
│ 4 │ code.py        │   3456   │
│ 5 │                │          │
│ 6 │ photo.jpg      │   9012   │
│ 7 │                │          │
└───┴────────────────┴──────────┘
```

**Advantages:** O(1) average lookup — very fast.
**Disadvantages:** Hash collisions require handling. Table size limits entries (may need rehashing).

> **Real-world:** Linux `ext4` uses **htree** (hash-tree) — a B-tree of hash values for very large directories. Modern macOS (APFS) also uses hash-based directory lookup.

---

## 15. Shared Files

A **shared file** is accessible from **multiple directory entries** (multiple locations in the directory tree).

### Hard Links
- Multiple directory entries point to the **same i-node**.
- All links are equal — no "original" and "copy".
- File is deleted only when **all hard links are removed** (link count = 0).

```
/home/mukesh/docs/report.pdf  ─────┐
                                   ├──► i-node 1234 ──► disk blocks
/home/shared/reports/report.pdf ───┘    (link count = 2)
```

```bash
ln source.txt link.txt    # Create hard link
```

**Limitations:** Cannot create hard links to directories (risk of cycles). Cannot link across file systems.

### Symbolic Links (Soft Links)
- A special file containing the **path to another file**.
- If target file is deleted, symlink becomes a "dangling link".
- Can link across file systems and to directories.

```
/home/mukesh/shortcut → /home/ram/original.txt
         ↑
   (symlink, stores path string)
```

```bash
ln -s /home/ram/original.txt /home/mukesh/shortcut    # Create symlink
```

---

## 16. Free Space Management

OS must track which disk blocks are free for allocation.

### 16.1 Bitmaps Free Space Management

A **bitmap** (bit vector) with one bit per disk block:
- **0** = free block
- **1** = allocated block

```
Disk has 16 blocks:
Block:   0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15
Bitmap: [1][1][1][0][1][1][0][0][1][0][0][1][0][0][0][0]

Free blocks: 3, 6, 7, 9, 10, 12, 13, 14, 15
```

**Finding n contiguous free blocks:**
- Scan bitmap for n consecutive 0s.
- Use bit manipulation for fast scanning (e.g., find all-zero word in one instruction).

**Advantages:** Simple, fast with bit manipulation.
**Disadvantages:** Bitmap must be in RAM for speed — large for big disks.

> **Example:** A 1TB disk with 4KB blocks = 256 million blocks → bitmap = 32MB. Manageable.

### 16.2 Linked List Free Space Management

**Free list:** All free blocks linked together in a chain.

```
Free List Head → Block 3 → Block 6 → Block 7 → Block 9 → NULL
```

**Alternative — Grouping:** First free block contains addresses of n free blocks; last points to another group.

**Advantages:** No wasted space for the free list itself (uses the free blocks).
**Disadvantages:** Cannot easily find contiguous free blocks. Traversal is slow.

---

## 17. Protection

Protection controls **who can access which files** and **what operations** are allowed.

### 17.1 Types of Access

Basic file operations that can be controlled:

| Access Type | Description |
|---|---|
| **Read (R)** | Read contents of file |
| **Write (W)** | Modify or overwrite file contents |
**Execute (X)** | Run file as a program |
| **Append (A)** | Add data to end of file only |
| **Delete (D)** | Remove the file |
| **List (L)** | List directory contents |
| **Rename** | Change file name |
| **Change permissions** | Modify access rights |

### 17.2 Access Control Matrix

An **access control matrix** lists all subjects (users) and objects (files) and their allowed operations.

```
Access Control Matrix:

           File1   File2   File3   Dir1
User A  │  RWX  │  RW   │  R    │  RWX  │
User B  │  R    │  RWX  │  RW   │  R    │
User C  │  —    │  R    │  RWX  │  —    │
```

**Reading the matrix:**
- User A can Read, Write, Execute File1 and has full access to Dir1.
- User B can only Read File1.
- User C has no access to File1.

**Disadvantages:** Matrix can be very large — n users × m files. Sparse for large systems (most entries are empty).

### 17.3 Access Control List (ACL)

Each **file/object** has a list of (user, permissions) pairs. Derived from columns of the access control matrix.

```
File1 ACL:
├── User A: RWX
├── User B: R
└── User C: (none)

File2 ACL:
├── User A: RW
├── User B: RWX
└── User C: R
```

**Advantages:** Compact storage — only non-empty entries stored. Easy to answer "who has access to this file?"

**Disadvantages:** Hard to answer "what files can User A access?" (must scan all ACLs).

### 17.4 Unix Permission Model (Simplified ACL)

Unix uses a **9-bit permission model** — three groups of 3 bits each:

```
Permission string: rwxr-xr--
                   ↑↑↑↑↑↑↑↑↑
                   │││││││││
                   │││││││└┘ Other (everyone else): r-- (read only)
                   │││└└┘│   Group: r-x (read and execute)
                   └└┘└──┘   Owner: rwx (read, write, execute)

r = read    (4)
w = write   (2)
x = execute (1)
- = no permission (0)

Octal: rwx=7, r-x=5, r--=4 → 754
```

```bash
# Examples:
chmod 755 script.sh   # rwxr-xr-x (owner full, group+other read+exec)
chmod 644 data.txt    # rw-r--r-- (owner read+write, others read only)
chmod 700 private.key # rwx------ (owner full, others no access)

# Special bits:
# Setuid (4): executable runs with owner's privileges
# Setgid (2): executable runs with group's privileges
# Sticky (1): only owner can delete file in shared directory (/tmp)
```

---

## 18. Path Names — Detailed

### Absolute vs Relative Path

```
File system tree:
/
├── home/
│   ├── mukesh/
│   │   ├── docs/
│   │   │   └── report.pdf
│   │   └── .bashrc
│   └── ram/
│       └── project/
│           └── main.c
└── etc/
    └── hosts

Current working directory: /home/mukesh/

Absolute path to report.pdf: /home/mukesh/docs/report.pdf
Relative path to report.pdf: docs/report.pdf

Absolute path to main.c: /home/ram/project/main.c
Relative path to main.c: ../ram/project/main.c
   (.. goes up to /home/, then into ram/project/)
```

---

## 19. Summary

### File System Components:

```
User Application
      ↓
System Calls (open, read, write, close)
      ↓
VFS (Virtual File System — uniform interface)
      ↓
Specific File System (ext4, NTFS, FAT32, APFS)
      ↓
Buffer Cache (block cache)
      ↓
Device Driver (disk driver)
      ↓
Physical Disk
```

### Key Structures:

| Structure | Purpose | Location |
|---|---|---|
| **Superblock** | File system metadata | Disk (near start) |
| **I-node table** | File metadata array | Disk |
| **Directory** | Maps names to i-nodes | Disk (as files) |
| **FAT table** | Block chains (FAT systems) | Disk + RAM |
| **Bitmap** | Free space tracking | Disk + RAM |
| **Open file table** | Per-process open files | RAM |
| **Buffer cache** | Recently accessed blocks | RAM |

---

## 20. Exam Tips

Commonly asked questions for Unit 7:

1. **What is a file? What attributes does a file have?**
2. **Explain the three file structures: byte sequence, record sequence, tree.**
3. **Differentiate sequential access vs direct access.**
4. **What is a file descriptor? What are the standard file descriptors?**
5. **Explain single-level, two-level, and hierarchical directory structures with diagrams.**
6. **What is the difference between absolute path and relative path?**
7. **Explain contiguous allocation, linked list allocation, and i-node based allocation with advantages and disadvantages.**
8. **What is an i-node? What information does it store?**
9. **Explain linear list vs hash table implementation of directories.**
10. **What is a hard link vs symbolic link?**
11. **Explain bitmap and linked list methods for free space management.**
12. **What is an Access Control Matrix? What is an ACL?**
13. **Explain Unix file permissions (rwxr-xr-x) with octal representation.**
14. **What are the types of file access (read, write, execute, etc.)?**

> **Key formulas:**
> ```
> Unix permission octal: r=4, w=2, x=1
>   rwxr-xr-- = 754
>   rw-rw-r-- = 664
>   rwx------ = 700
>
> I-node addressing capacity:
>   12 direct × 4KB = 48KB
>   1 single indirect × 1024 × 4KB = 4MB
>   1 double indirect × 1024² × 4KB = 4GB
> ```

---

## 21. References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12 — Chapter 4
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014 — Chapters 11 & 12
