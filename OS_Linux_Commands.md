# Linux Terminal Commands — Practical Guide
**Course:** CACS251 — Operating System
**Year/Semester:** BCA II/IV | Pokhara University

---

## How to Open Terminal

| OS | How to Open |
|---|---|
| **Ubuntu/Linux** | Right-click desktop → "Open Terminal" OR press `Ctrl + Alt + T` |
| **macOS** | Applications → Utilities → Terminal OR `Cmd + Space` → type "terminal" |
| **Windows** | Install WSL (Windows Subsystem for Linux) OR use Git Bash |
| **Lab (any)** | Find "Terminal" or "Console" in applications menu |

---

## Understanding the Terminal Prompt

```
mukesh@ubuntu:~/documents$
  ↑       ↑        ↑      ↑
username hostname  current  command
                  directory  starts here

~  = your home directory (/home/mukesh)
$  = regular user prompt
#  = root (admin) user prompt — be careful!
```

---

# SECTION 1 — Navigation Commands

## 1. `pwd` — Print Working Directory
Shows your **current location** in the file system.

```bash
$ pwd
/home/mukesh/documents
```

> Think of it as "Where am I right now?"

---

## 2. `ls` — List Directory Contents
Shows **files and folders** in the current directory.

```bash
$ ls
documents  downloads  music  notes.txt  photo.jpg

# With details (permissions, size, date):
$ ls -l
total 48
drwxr-xr-x 2 mukesh mukesh 4096 Jun  1 10:30 documents
drwxr-xr-x 2 mukesh mukesh 4096 Jun  1 09:15 downloads
-rw-r--r-- 1 mukesh mukesh 2048 Jun  1 11:00 notes.txt

# Show hidden files (files starting with .):
$ ls -a
.  ..  .bashrc  .profile  documents  downloads  notes.txt

# Human-readable file sizes:
$ ls -lh
-rw-r--r-- 1 mukesh mukesh 2.0K Jun  1 notes.txt

# Most useful combination:
$ ls -la
(shows all files including hidden, with full details)
```

**Output of `ls -l` explained:**
```
-rw-r--r-- 1 mukesh mukesh 2048 Jun 1 11:00 notes.txt
↑          ↑ ↑      ↑      ↑    ↑           ↑
type+perms links owner group size date        name
```

---

## 3. `cd` — Change Directory
**Navigate** between directories.

```bash
# Go into a directory:
$ cd documents

# Go back to parent directory:
$ cd ..

# Go back two levels:
$ cd ../..

# Go to home directory (three ways):
$ cd
$ cd ~
$ cd /home/mukesh

# Go to root directory:
$ cd /

# Go to previous directory:
$ cd -

# Use absolute path:
$ cd /home/mukesh/documents/os_notes

# Use relative path (from current location):
$ cd ../downloads
```

**Tip:** Press `Tab` to auto-complete directory/file names!

---

## 4. `tree` — Show Directory Tree (if installed)
```bash
$ tree
.
├── documents
│   ├── os_notes
│   │   └── unit1.md
│   └── report.pdf
├── downloads
└── notes.txt

# Install if not available:
$ sudo apt install tree
```

---

# SECTION 2 — File & Directory Management

## 5. `mkdir` — Make Directory
Create **new folders**.

```bash
# Create one directory:
$ mkdir os_lab

# Create multiple directories at once:
$ mkdir lab1 lab2 lab3

# Create nested directories (parent + child at once):
$ mkdir -p projects/os/unit3
# -p = create parent directories if they don't exist

# Verify:
$ ls
lab1  lab2  lab3  os_lab  projects
```

---

## 6. `touch` — Create Empty File / Update Timestamp
```bash
# Create a new empty file:
$ touch hello.c
$ touch notes.txt

# Create multiple files:
$ touch file1.c file2.c file3.c

# Update timestamp of existing file:
$ touch existing_file.txt
```

---

## 7. `cp` — Copy Files and Directories
```bash
# Copy a file:
$ cp source.txt destination.txt

# Copy to another directory:
$ cp notes.txt documents/

# Copy and rename:
$ cp notes.txt documents/notes_backup.txt

# Copy directory (recursive):
$ cp -r lab1/ lab1_backup/
# -r = recursive (copies all contents)

# Copy multiple files to a directory:
$ cp file1.c file2.c file3.c ~/documents/

# Verbose (show what is being copied):
$ cp -v source.txt dest.txt
'source.txt' -> 'dest.txt'
```

---

## 8. `mv` — Move or Rename Files
```bash
# Rename a file:
$ mv old_name.txt new_name.txt

# Move file to another directory:
$ mv notes.txt documents/

# Move and rename at same time:
$ mv notes.txt documents/my_notes.txt

# Move directory:
$ mv lab1/ projects/

# Move multiple files:
$ mv file1.c file2.c file3.c ~/lab/
```

---

## 9. `rm` — Remove Files and Directories
```bash
# Delete a file:
$ rm notes.txt

# Delete multiple files:
$ rm file1.txt file2.txt file3.txt

# Delete directory and all its contents:
$ rm -r old_directory/
# -r = recursive

# Force delete (no confirmation prompt):
$ rm -f file.txt

# Force delete directory recursively (USE WITH CARE!):
$ rm -rf directory_name/

# Interactive (asks before each deletion):
$ rm -i file.txt
rm: remove regular file 'file.txt'? y
```

> **WARNING:** `rm` is permanent — no Recycle Bin! Double-check before running.

---

## 10. `rmdir` — Remove Empty Directory
```bash
# Only works on empty directories:
$ rmdir empty_folder

# If directory has files, use rm -r instead:
$ rm -r folder_with_files/
```

---

# SECTION 3 — Viewing File Contents

## 11. `cat` — Display File Contents
```bash
# Show full file content:
$ cat notes.txt
This is my OS notes.
Chapter 1: Introduction

# Show with line numbers:
$ cat -n notes.txt
     1	This is my OS notes.
     2	Chapter 1: Introduction

# Combine multiple files:
$ cat file1.txt file2.txt

# Create a file by typing content (Ctrl+D to save):
$ cat > new_file.txt
Type your content here
Press Ctrl+D when done
```

---

## 12. `less` — View File Page by Page
For **large files** — view one page at a time.

```bash
$ less large_file.txt

# Navigation inside less:
# Space or Page Down = next page
# b or Page Up       = previous page
# /search_term       = search forward
# n                  = next search result
# q                  = quit
```

---

## 13. `head` and `tail` — View Beginning or End of File
```bash
# Show first 10 lines:
$ head file.txt

# Show first 20 lines:
$ head -n 20 file.txt

# Show last 10 lines:
$ tail file.txt

# Show last 20 lines:
$ tail -n 20 file.txt

# Watch file as it grows in real-time (very useful for logs!):
$ tail -f /var/log/syslog
```

---

## 14. `wc` — Word Count
```bash
# Count lines, words, and characters:
$ wc file.txt
  10   50  300 file.txt
  ↑    ↑    ↑
lines words chars

# Count only lines:
$ wc -l file.txt

# Count only words:
$ wc -w file.txt
```

---

# SECTION 4 — Text Editing in Terminal

## 15. `nano` — Simple Text Editor (Beginner Friendly)
```bash
# Open or create a file:
$ nano hello.c

# Inside nano:
# Type your code normally
# Ctrl + O  = Save (Write Out)
# Ctrl + X  = Exit
# Ctrl + W  = Search
# Ctrl + K  = Cut line
# Ctrl + U  = Paste line
# Ctrl + G  = Help
```

---

## 16. `vim` — Advanced Text Editor
```bash
# Open a file:
$ vim hello.c

# vim has TWO modes:
# NORMAL mode   = navigate, run commands (default)
# INSERT mode   = type text

# Press 'i'       → Enter INSERT mode (you can now type)
# Press Esc       → Go back to NORMAL mode
# Type ':w'       → Save
# Type ':q'       → Quit
# Type ':wq'      → Save and quit
# Type ':q!'      → Quit WITHOUT saving
# Type '/word'    → Search for "word"
# Type 'dd'       → Delete current line (in NORMAL mode)
# Type 'yy'       → Copy current line
# Type 'p'        → Paste
```

---

# SECTION 5 — Compiling and Running C Programs

## 17. `gcc` — GNU C Compiler

```bash
# Compile a C program:
$ gcc hello.c -o hello

# hello.c   = source file
# -o hello  = output executable named "hello"

# Run the compiled program:
$ ./hello
Hello, World!

# Compile with warnings (recommended for learning):
$ gcc -Wall hello.c -o hello

# Compile with threading support (for Lab 2):
$ gcc program.c -o program -lpthread

# Compile with math library:
$ gcc program.c -o program -lm

# Compile multiple source files:
$ gcc file1.c file2.c -o output
```

**Full workflow example:**
```bash
# Step 1: Create the file
$ nano hello.c

# Step 2: Write your code in nano, save with Ctrl+O, exit with Ctrl+X

# Step 3: Compile
$ gcc hello.c -o hello

# Step 4: Run
$ ./hello
Hello, World!

# If there are errors, fix them in nano and recompile
```

---

## 18. Common Compilation Errors and Fixes

```bash
# Error: file not found
$ gcc helo.c -o hello
gcc: error: helo.c: No such file or directory
# Fix: Check filename spelling → ls to see files

# Error: function not declared
hello.c:3:5: error: implicit declaration of function 'printf'
# Fix: Add #include <stdio.h> at top

# Error: permission denied to run
$ hello
bash: hello: Permission denied
# Fix: Use ./hello instead, or: chmod +x hello

# Warning: unused variable (program still compiles)
hello.c:5:9: warning: unused variable 'x'
# Fix: Either use the variable or remove it
```

---

# SECTION 6 — File Permissions

## 19. `chmod` — Change File Permissions
```bash
# Give execute permission to owner:
$ chmod u+x script.sh

# Remove write permission from others:
$ chmod o-w file.txt

# Set permissions using octal (rwx = 4+2+1 = 7):
$ chmod 755 script.sh    # rwxr-xr-x
$ chmod 644 notes.txt    # rw-r--r--
$ chmod 700 private.key  # rwx------

# Make all .c files in directory executable:
$ chmod +x *.c

# Recursive (apply to directory and all contents):
$ chmod -R 755 my_folder/

# View permissions:
$ ls -l
-rwxr-xr-x 1 mukesh mukesh 8720 Jun 1 hello
```

**Permission number quick reference:**
```
7 = rwx (read + write + execute)
6 = rw- (read + write)
5 = r-x (read + execute)
4 = r-- (read only)
0 = --- (no permission)

chmod 755 = owner:rwx, group:r-x, others:r-x
chmod 644 = owner:rw-, group:r--, others:r--
chmod 600 = owner:rw-, group:---, others:---
```

---

## 20. `chown` — Change File Owner
```bash
# Change owner:
$ sudo chown ram file.txt

# Change owner and group:
$ sudo chown ram:staff file.txt

# Recursive:
$ sudo chown -R mukesh:mukesh my_folder/
```

---

# SECTION 7 — Searching and Filtering

## 21. `grep` — Search for Text in Files
```bash
# Search for "hello" in file:
$ grep "hello" file.txt

# Case-insensitive search:
$ grep -i "hello" file.txt

# Show line numbers:
$ grep -n "hello" file.txt

# Search in all .c files:
$ grep "malloc" *.c

# Search recursively in all files in directory:
$ grep -r "deadlock" ~/os_notes/

# Show lines that do NOT match:
$ grep -v "hello" file.txt

# Search for exact word:
$ grep -w "int" program.c
```

---

## 22. `find` — Find Files and Directories
```bash
# Find file by name:
$ find . -name "hello.c"
./programs/hello.c

# Find all .c files:
$ find . -name "*.c"

# Find files modified in last 24 hours:
$ find . -mtime -1

# Find files larger than 1MB:
$ find . -size +1M

# Find and execute command on results:
$ find . -name "*.o" -delete    # Delete all .o files
```

---

## 23. `sort` — Sort File Contents
```bash
# Sort alphabetically:
$ sort names.txt

# Sort numerically:
$ sort -n numbers.txt

# Sort in reverse:
$ sort -r file.txt

# Sort and remove duplicates:
$ sort -u file.txt
```

---

# SECTION 8 — Process Management (OS Related!)

## 24. `ps` — Show Running Processes
```bash
# Show your processes:
$ ps

# Show ALL processes (detailed):
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mukesh    1234  0.0  0.1   8720  1024 pts/0    S    10:30   0:00 bash
mukesh    5678  1.2  0.5  65536  5120 pts/0    R    10:35   0:02 ./myprogram

# Show process tree:
$ ps auxf

# Find a specific process:
$ ps aux | grep myprogram
```

---

## 25. `top` — Live Process Monitor
```bash
$ top

# Inside top:
# q        = quit
# k        = kill a process (enter PID)
# M        = sort by memory usage
# P        = sort by CPU usage
# h        = help
```

---

## 26. `kill` — Terminate a Process
```bash
# Kill process by PID (graceful):
$ kill 1234

# Force kill (if process is stuck):
$ kill -9 1234

# Kill by process name:
$ killall firefox

# Find PID first, then kill:
$ ps aux | grep myprogram
mukesh 5678 ...
$ kill 5678
```

---

## 27. Running Programs in Background
```bash
# Run in background (& at end):
$ ./long_program &
[1] 5678           ← Job number and PID

# See background jobs:
$ jobs
[1]+  Running    ./long_program &

# Bring back to foreground:
$ fg 1

# Send running program to background:
# Press Ctrl+Z (suspend), then:
$ bg 1
```

---

# SECTION 9 — Useful Shortcuts and Tips

## 28. Keyboard Shortcuts in Terminal

| Shortcut | Action |
|---|---|
| `Tab` | Auto-complete command or filename |
| `Tab Tab` | Show all possible completions |
| `Ctrl + C` | Cancel/kill current running program |
| `Ctrl + Z` | Suspend (pause) current program |
| `Ctrl + D` | End of input (logout from terminal) |
| `Ctrl + L` | Clear the screen (same as `clear`) |
| `Ctrl + A` | Move cursor to beginning of line |
| `Ctrl + E` | Move cursor to end of line |
| `Ctrl + U` | Delete from cursor to beginning |
| `Ctrl + K` | Delete from cursor to end |
| `Up Arrow` | Previous command (command history) |
| `Down Arrow` | Next command in history |
| `Ctrl + R` | Search command history |

---

## 29. Redirection and Pipes

```bash
# Redirect output to file (overwrite):
$ ls > file_list.txt

# Redirect output to file (append):
$ echo "new line" >> file_list.txt

# Redirect error messages:
$ gcc bad.c 2> errors.txt

# Redirect both output and errors:
$ gcc program.c > output.txt 2>&1

# Pipe — send output of one command to another:
$ ls -l | grep ".c"          # List only .c files
$ ps aux | grep firefox       # Find firefox process
$ cat file.txt | wc -l        # Count lines in file
$ ls | sort | head -5         # First 5 files alphabetically
```

---

## 30. `history` — Command History
```bash
# Show last commands:
$ history
  1  ls
  2  cd documents
  3  gcc hello.c -o hello
  4  ./hello

# Run command number 3 again:
$ !3

# Run last command again:
$ !!

# Search history:
$ history | grep gcc
```

---

## 31. `man` — Manual Pages (Built-in Help)
```bash
# Get help for any command:
$ man ls
$ man gcc
$ man chmod

# Inside man:
# Space    = next page
# b        = previous page
# /word    = search
# q        = quit

# Short help (most commands support --help):
$ ls --help
$ gcc --help
```

---

# SECTION 10 — System Information

## 32. Useful System Commands
```bash
# Show current user:
$ whoami
mukesh

# Show system information:
$ uname -a
Linux ubuntu 5.15.0 #1 SMP x86_64 GNU/Linux

# Show disk usage:
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   12G   38G  24% /

# Show memory usage:
$ free -h
              total   used   free
Mem:           7.7G   3.2G   4.5G
Swap:          2.0G   0.0B   2.0G

# Show folder/file sizes:
$ du -sh documents/
4.2M    documents/

# Show current date and time:
$ date
Wed Jul 17 10:30:00 NPT 2026

# Clear terminal screen:
$ clear
```

---

# SECTION 11 — Package Management

## 33. `apt` — Install Software (Ubuntu/Debian)
```bash
# Update package list:
$ sudo apt update

# Install a package:
$ sudo apt install gcc
$ sudo apt install vim
$ sudo apt install tree
$ sudo apt install net-tools

# Remove a package:
$ sudo apt remove packagename

# Search for a package:
$ apt search gcc
```

---

# SECTION 12 — Practical Lab Workflow

## Complete Workflow for Writing and Running C Programs

```bash
# Step 1: Create a working directory
$ mkdir ~/os_lab
$ cd ~/os_lab

# Step 2: Create your C file
$ nano scheduling.c
(write your code, Ctrl+O to save, Ctrl+X to exit)

# Step 3: Verify the file was created
$ ls -l
-rw-r--r-- 1 mukesh mukesh 1024 Jun 1 scheduling.c

# Step 4: Compile
$ gcc scheduling.c -o scheduling

# If no errors, you'll see the prompt again
# If errors appear, read the error message, fix in nano, recompile

# Step 5: Run
$ ./scheduling

# Step 6: If program needs input, type it when prompted

# Step 7: Save your output to a file for submission
$ ./scheduling > output.txt
$ cat output.txt    # Verify the output
```

---

## Common Mistakes and Fixes

| Mistake | Error Message | Fix |
|---|---|---|
| Typo in filename | `No such file or directory` | Check spelling with `ls` |
| Forgot `./` to run | `command not found` | Use `./programname` |
| No execute permission | `Permission denied` | Run `chmod +x programname` |
| Missing semicolon in C | `error: expected ';'` | Fix in nano/vim, recompile |
| Wrong directory | Files not found | Use `pwd` to check location |
| Infinite loop | Program never stops | Press `Ctrl+C` to stop |
| Missing `#include` | `implicit declaration` | Add correct `#include` at top |

---

## Quick Reference Card

```
NAVIGATION:
  pwd          → Where am I?
  ls           → What's here?
  ls -la       → Full details including hidden files
  cd dirname   → Go into dirname
  cd ..        → Go up one level
  cd ~         → Go to home directory
  cd -         → Go to previous directory

FILE OPERATIONS:
  touch file.txt        → Create empty file
  mkdir dirname         → Create directory
  cp src dst            → Copy
  mv src dst            → Move or rename
  rm file.txt           → Delete file
  rm -r dirname/        → Delete directory

VIEW FILES:
  cat file.txt          → Show full file
  less file.txt         → Page-by-page view
  head -n 20 file.txt   → First 20 lines
  tail -n 20 file.txt   → Last 20 lines

C PROGRAMMING:
  gcc file.c -o output  → Compile
  ./output              → Run
  gcc file.c -o out -lpthread  → Compile with threads

SEARCH:
  grep "word" file.txt  → Find text in file
  find . -name "*.c"    → Find files

PROCESS:
  ps aux                → List all processes
  top                   → Live process monitor
  kill PID              → Kill a process
  Ctrl+C                → Stop running program

PERMISSIONS:
  ls -l                 → View permissions
  chmod 755 file        → Set permissions
  chmod +x file         → Add execute permission

HELP:
  man command           → Full manual
  command --help        → Quick help
```

---

## References

- Linux man pages: `man <command>` in terminal
- GNU Bash Reference: https://www.gnu.org/software/bash/manual/
- The Linux Command Line (free book): https://linuxcommand.org/tlcl.php
- Ubuntu Documentation: https://help.ubuntu.com
