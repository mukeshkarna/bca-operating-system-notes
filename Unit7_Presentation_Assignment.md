# Unit 7: File System Interface Management
# Presentation Topic Assignment
**Course:** CACS251 — Operating System
**Year/Semester:** BCA II/IV | Tribhuvan University

---

## Presentation Guidelines

- **Duration:** 10–15 minutes per student
- **Format:** PowerPoint / Google Slides (minimum 8 slides)
- **Language:** English
- **Must include:** Definition, diagram/example, real-world example, and at least 2 exam-type questions with answers

---

## Topic Assignment

---

### Abhijeet — File Concept and File Naming

**Topics to cover:**
1. What is a File System? (Introduction) — Why do we need it? Responsibilities.
2. What is a File? — User perspective vs OS perspective.
3. File Naming — Rules in different OS (Windows, Linux, old FAT/DOS), name vs extension.
4. Common file extensions and what they mean.

**Key points to explain:**
- Difference between a file on disk vs a running process.
- Why Linux is case-sensitive (`report.pdf` ≠ `Report.pdf`) but Windows is not.
- What happens if you name a file with an invalid character (e.g., `/` in Linux).

**Real-world example to include:**
- Show the same file (`notes.txt`) saved on Windows and Linux — naming rules difference.
- Show a table of common extensions: `.c`, `.exe`, `.jpg`, `.zip`, `.pdf`.

**Exam questions to prepare:**
1. What are the components of a file name? Give examples.
2. What is the 8.3 file naming convention in old DOS/FAT systems?

---

### Ankit — File Structure, File Types, and File Attributes

**Topics to cover:**
1. File Structure — Byte Sequence, Record Sequence, Tree Structure (with diagrams).
2. File Types — Regular, Directory, Character Special, Block Special, Symbolic Link, Pipe, Socket.
3. File Attributes — All metadata fields: name, size, owner, permissions, timestamps, etc.
4. How OS identifies file type (extension vs magic number).

**Key points to explain:**
- Why Unix uses "everything is a byte stream" (byte sequence) approach.
- What a magic number is — first few bytes identify file type without extension.
- What each attribute field in `ls -l` output means.

**Real-world example to include:**
- Run `ls -la` in terminal — explain every column of output.
- Show magic numbers: JPEG starts with `FF D8 FF`, PDF with `%PDF`.
- Employee record example for record sequence structure.

**Exam questions to prepare:**
1. Explain the three types of file structures with examples.
2. List any five file attributes and their purpose.

---

### Rajeshower — File Access Methods, File Operations, and File Descriptors

**Topics to cover:**
1. File Access Methods — Sequential Access vs Direct (Random) Access with diagrams.
2. File Operations — Create, Open, Read, Write, Seek, Close, Delete, Rename (all system calls).
3. File Descriptors — What they are, how they work, standard file descriptors (0, 1, 2).
4. Basic C code examples for file operations (`open`, `read`, `write`, `close`).

**Key points to explain:**
- Why sequential access is like a tape — cannot skip forward.
- Why direct access needs fixed-size records (to calculate offset).
- Why file descriptor is just an integer — OS manages the rest.
- What 0 (stdin), 1 (stdout), 2 (stderr) are and why they matter.

**Real-world example to include:**
- Playing a video file = sequential access (can't jump without buffering).
- Database lookup by employee ID = direct access.
- Show what happens step by step when you run `cat file.txt` in terminal.

**Exam questions to prepare:**
1. Differentiate sequential access and direct access with examples.
2. What is a file descriptor? What are the standard file descriptors in Unix?

---

### Sailesh — Directory Structure and Path Names

**Topics to cover:**
1. What is a Directory? — Purpose, directory entry contents.
2. Single-Level Directory — Diagram, advantages, disadvantages.
3. Two-Level Directory — Diagram, advantages, disadvantages.
4. Hierarchical (Tree) Directory — Full diagram, how modern OS uses it.
5. File System Layout — Boot block, super block, inode table, data blocks.
6. Path Names — Absolute path vs Relative path with examples.
7. Special path symbols: `.` (current), `..` (parent), `~` (home).

**Key points to explain:**
- Why single-level failed for multi-user systems (name conflicts).
- How Windows uses `C:\` and Linux uses `/` as their root.
- Why absolute paths always work but relative paths depend on location.
- The difference between `cd documents` and `cd /home/mukesh/documents`.

**Real-world example to include:**
- Draw the Windows directory tree: `C:\Users\Mukesh\Documents\report.pdf`.
- Draw the Linux directory tree: `/home/mukesh/documents/report.pdf`.
- Show how `cd ..` moves up one level using a diagram.

**Exam questions to prepare:**
1. Draw and explain the hierarchical directory structure with a diagram.
2. What is the difference between absolute path and relative path? Give examples.

---

### Shilpa — Implementing Files and I-nodes

**Topics to cover:**
1. Contiguous Allocation — Diagram, advantages, disadvantages.
2. Linked List Allocation — Diagram, advantages, disadvantages.
3. Linked List using Table in Memory (FAT) — FAT table diagram, how it works.
4. Index Allocation (I-node) — I-node structure diagram, direct/indirect pointers.
5. I-node — All fields, multi-level indexing (single, double, triple indirect).
6. Real file systems using each method (ISO 9660 for CD, FAT32 for USB, ext4 for Linux).

**Key points to explain:**
- Why contiguous fails (cannot grow, external fragmentation).
- How FAT solves linked list's random access problem (table in RAM).
- How i-node handles very large files using indirect pointers.
- Calculate: with 12 direct + 1 single indirect (1024 pointers) and 4KB blocks → max size.

**Real-world example to include:**
- USB drives use FAT32 — show how FAT table links blocks.
- Linux uses ext4 with i-nodes — show `ls -i` command to see inode numbers.
- CD-ROM uses contiguous — explain why (written once, never changes).

**Exam questions to prepare:**
1. Explain contiguous file allocation with a diagram. What are its disadvantages?
2. What is an i-node? Draw its structure and explain direct and indirect pointers.

---

### Shreya — Directory Implementation, Shared Files, Free Space Management, and Protection

**Topics to cover:**
1. Directory Implementation — Linear List vs Hash Table (comparison).
2. Shared Files — Hard Links vs Symbolic (Soft) Links with diagrams.
3. Free Space Management — Bitmap method and Linked List method.
4. Protection — Types of Access (Read, Write, Execute, Append, Delete).
5. Access Control Matrix — Diagram, how it works, disadvantage (size).
6. Access Control List (ACL) — How it improves on the matrix.
7. Unix Permission System — rwx bits, octal notation, `chmod` command.

**Key points to explain:**
- Why hash table directory is faster than linear list for large directories.
- Difference between hard link and symlink — what happens if the original is deleted.
- Why bitmap is better for free space tracking than linked list.
- How to read `-rwxr-xr--` and convert to octal (754).
- Row vs column of access control matrix → capabilities vs ACL.

**Real-world example to include:**
- Show `chmod 755 script.sh` and `chmod 644 notes.txt` in terminal.
- Explain why `/tmp` has sticky bit — only owner can delete their files.
- Show `ln` and `ln -s` commands to create hard link and symlink.
- Show ACL example from Windows: right-click → Properties → Security tab.

**Exam questions to prepare:**
1. What is an Access Control List? How is it derived from the Access Control Matrix?
2. Explain Unix file permissions. What does `chmod 755` mean?

---

## Summary — Topic Distribution

| Student | Main Topics | Sections from Notes |
|---|---|---|
| **Abhijeet** | File System Intro, File Concept, File Naming | Sections 1, 2, 3 |
| **Ankit** | File Structure, File Types, File Attributes | Sections 4, 5, 7 |
| **Rajeshower** | File Access Methods, File Operations, File Descriptors | Sections 6, 8 |
| **Sailesh** | Directory Structure, Path Names | Sections 9, 10, 11 |
| **Shilpa** | Implementing Files, I-nodes | Sections 12, 13 |
| **Shreya** | Directory Implementation, Shared Files, Free Space, Protection | Sections 14, 15, 16, 17 |

---

## Evaluation Criteria

| Criteria | Marks |
|---|---|
| Content accuracy and completeness | 30% |
| Clarity of explanation | 20% |
| Diagrams / visual aids | 20% |
| Real-world examples | 15% |
| Q&A handling | 15% |
| **Total** | **100%** |

---

## Tips for All Students

- **Start with a definition** — one clear sentence saying what your topic is.
- **Always draw a diagram** — file system topics are very visual.
- **Give a real-world example** — relate to something students use daily (USB, Google Drive, phone storage).
- **End with 2–3 exam questions** — helps everyone revise.
- **Prepare for questions** — teacher or other students may ask you anything about your topic.
- **Reference:** Use `OS_Unit7_Notes.md` as your primary source + Silberschatz Chapter 11–12.
