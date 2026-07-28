# Unit 8: Security Management
**Course:** CACS251 — Operating System | **Duration:** 3 Hours
**Year/Semester:** BCA II/IV | Tribhuvan University

---

## Syllabus Topics
Introduction, Security Problems, User Authentication: Passwords, Encrypted Password, One Time Password and Biometrics Password; User Authorization, Program Threats: Trojan Horse, Trap Door, Stack and Buffer Overflow; System Threats: Worms, Viruses, Denial of Services.

---

## 1. Introduction to Security Management

**Security** in an OS means protecting the computer system — its hardware, software, and data — from **unauthorized access, damage, or misuse**.

**OS Security has three main goals (CIA Triad):**

```
┌─────────────────────────────────────────────────────────┐
│                     CIA TRIAD                            │
│                                                          │
│   Confidentiality   Integrity    Availability            │
│        ↓                ↓              ↓                 │
│  Only authorized    Data is not    System is             │
│  users can access   modified       accessible when       │
│  data               without        needed                │
│                     authorization                        │
└─────────────────────────────────────────────────────────┘
```

**Confidentiality:** Information is accessible only to those authorized.
**Integrity:** Data can be modified only by authorized users in authorized ways.
**Availability:** System resources available to authorized users when needed.

> **Real-world example:** Your online banking system — only you (confidentiality) can see your balance, nobody should change your balance without your transaction (integrity), and the bank website should always be accessible (availability).

---

## 2. Security Problems

Security threats arise from multiple sources. The OS must defend against:

### 2.1 Accidental vs Intentional Threats

| Type | Description | Example |
|---|---|---|
| **Accidental** | Unintentional damage — no malicious intent | Accidental file deletion, power failure during write |
| **Intentional** | Deliberate attack — malicious intent | Hacking, virus infection, data theft |

### 2.2 Security Violation Categories

**Breach of Confidentiality:**
- Unauthorized reading of data.
- Example: Reading another user's private files.

**Breach of Integrity:**
- Unauthorized modification of data.
- Example: Changing exam results in a database.

**Breach of Availability:**
- Unauthorized destruction or making data unavailable.
- Example: Deleting critical system files, DoS attack.

**Theft of Service:**
- Unauthorized use of resources.
- Example: Using someone's computer for crypto mining.

**Denial of Service (DoS):**
- Preventing legitimate use of the system.
- Example: Flooding a server with requests so real users cannot connect.

### 2.3 Security Levels

Security must be maintained at multiple levels:

```
Physical Level     → Physical access to the machine
Human Level        → Social engineering, passwords
OS Level           → Authentication, access control
Network Level      → Firewalls, encryption
Application Level  → Input validation, secure coding
```

> **Real-world example:** A bank protects at all levels — physical vault (physical), teller verification (human), secure login (OS), HTTPS (network), and sanitized input fields (application).

---

## 3. User Authentication

**Authentication** = verifying that a user is who they claim to be.

It answers the question: **"Are you really who you say you are?"**

```
User claims identity → Authentication → Verified? → Access granted/denied
"I am mukesh"             ↓
                    Verify the claim
                    (password/biometric)
```

### Three Authentication Factors:
1. **Something you KNOW** — Password, PIN, security question.
2. **Something you HAVE** — Smart card, OTP token, phone.
3. **Something you ARE** — Fingerprint, face, iris (biometrics).

**Multi-Factor Authentication (MFA):** Uses 2 or more factors for stronger security.

> **Real-world example:** ATM = something you HAVE (card) + something you KNOW (PIN). Online banking with OTP = password + phone.

---

### 3.1 Passwords

The most common authentication method — user knows a secret string.

**Password Storage:**
- Passwords are **never stored in plain text**.
- OS stores the **hash** of the password.
- On login: hash(entered password) compared with stored hash.

```
Registration:
password "hello123" → hash function → "a665a45920..." stored in /etc/shadow

Login attempt:
password "hello123" → hash function → "a665a45920..." → matches! ✅
password "wrong"    → hash function → "ade46b0298..." → no match! ❌
```

**Common hash functions:** MD5 (outdated), SHA-256, bcrypt (recommended).

**Password Vulnerabilities:**
- **Brute force attack:** Try all possible combinations.
- **Dictionary attack:** Try common words and passwords.
- **Shoulder surfing:** Watching someone type.
- **Social engineering:** Tricking user to reveal password.
- **Phishing:** Fake login page captures password.

**Strong Password Guidelines:**
- Minimum 8–12 characters.
- Mix of uppercase, lowercase, numbers, symbols.
- No dictionary words.
- Different password for each account.
- Changed periodically.

> **Real-world example:** `password123` is the most common password and is cracked instantly. `P@$$w0rd#2024!` takes billions of years to brute force.

---

### 3.2 Encrypted Password

**Problem with plain hashing:** If two users have the same password, they have the same hash. An attacker can use **rainbow tables** (pre-computed hash tables) to reverse hashes.

**Solution: Salted Hashing**
- A random string called **salt** is added to the password before hashing.
- Salt is stored alongside the hash (not secret — just prevents rainbow tables).

```
Password:  "hello123"
Salt:      "xK9#mPq2"  (random, unique per user)
Salted:    "hello123xK9#mPq2"
Hash:      hash("hello123xK9#mPq2") = "f3a8b2..."

Stored in /etc/shadow:
mukesh:$6$xK9#mPq2$f3a8b2...:...
         ↑salt↑  ↑hash↑
```

**Why salt helps:**
- Two users with same password → different hashes (different salts).
- Rainbow tables useless — must recompute for each salt.
- Attacker must attack each account individually.

**Unix `/etc/shadow` file format:**
```
username:$algorithm$salt$hash:last_change:min:max:warn:inactive:expire
mukesh:$6$salt$hash:19500:0:99999:7:::
       ↑
       $6 = SHA-512 algorithm
```

---

### 3.3 One Time Password (OTP)

A password that is **valid for only one login session or transaction**. Even if intercepted, it cannot be reused.

**Types of OTP:**

**Time-Based OTP (TOTP):**
- Generated using current time + shared secret key.
- Changes every 30 seconds.
- Example: Google Authenticator app.

```
OTP = hash(secret_key + current_time_slot)
Valid only for 30-second window
```

**HMAC-Based OTP (HOTP):**
- Generated using a counter + secret key.
- Counter increments with each use.

**SMS OTP:**
- Server generates random number, sends via SMS.
- User enters code to verify.

**How OTP works:**
```
Step 1: User enters username + password (first factor)
Step 2: Server sends OTP to user's phone/app
Step 3: User enters OTP (second factor)
Step 4: Server verifies OTP is valid and not expired
Step 5: Access granted
```

**Advantages:**
- Even if OTP is intercepted, it expires immediately.
- Protects against password replay attacks.
- Easy to implement.

**Disadvantages:**
- Requires phone/device — not available if lost or no signal.
- Vulnerable to SIM-swapping attacks (for SMS OTP).

> **Real-world example:** Every time you log into your bank from a new device, a 6-digit OTP is sent to your registered mobile number. This OTP is valid for only 5 minutes.

---

### 3.4 Biometrics Password

Authentication using **physical characteristics** or **behavioral traits** that are unique to each person.

**Types of Biometrics:**

| Type | What it measures | Example |
|---|---|---|
| **Fingerprint** | Unique ridge patterns on fingertip | Smartphone unlock, laptop login |
| **Face Recognition** | Facial geometry, features | Face ID on iPhone, Windows Hello |
| **Iris Scan** | Unique patterns in the iris of the eye | High-security facilities |
| **Voice Recognition** | Voice pitch, tone, pattern | Voice assistants, phone banking |
| **Retina Scan** | Blood vessel pattern at back of eye | Government/military security |
| **Hand Geometry** | Shape and size of hand | Physical access control |

**How Biometric Authentication Works:**
```
Enrollment:
  User provides biometric → System creates template → Stored securely

Verification:
  User provides biometric → System creates template
       ↓
  Compare with stored template
       ↓
  Match within threshold? → Access granted
  No match?              → Access denied
```

**Key Terms:**
- **FAR (False Acceptance Rate):** How often an impostor is accepted. Lower = more secure.
- **FRR (False Rejection Rate):** How often a legitimate user is rejected. Lower = more usable.
- **EER (Equal Error Rate):** Point where FAR = FRR. Better systems have lower EER.

**Advantages:**
- Cannot be forgotten (unlike password) or stolen (unlike card).
- Unique to each person.
- Fast and convenient.

**Disadvantages:**
- Cannot be changed if compromised (you cannot change your fingerprint).
- Privacy concerns — biometric data is sensitive.
- Environmental factors (dirty fingers, glasses for face recognition).
- Hardware required (sensor).

> **Real-world example:** Fingerprint unlock on your smartphone — when you set it up (enrollment), the phone stores a mathematical model of your fingerprint. Every time you press your finger, it compares the new scan against the stored model.

---

## 4. User Authorization

**Authorization** = determining what an **authenticated user is allowed to do**.

Authentication answers "Who are you?" Authorization answers "What can you do?"

```
Authentication → Authorization → Access Control
"Are you mukesh?" → "What can mukesh do?" → "mukesh can read file1, write file2"
```

**Authorization mechanisms:**
- **Access Control Matrix** — table of users × resources × permissions.
- **Access Control List (ACL)** — per-resource list of (user, permission) pairs.
- **Role-Based Access Control (RBAC)** — permissions assigned to roles, users assigned to roles.

**RBAC Example:**
```
Roles:         Student, Teacher, Admin
Permissions:
  Student:  Read notes, Submit assignments
  Teacher:  Read/Write notes, Grade assignments, View student list
  Admin:    All permissions + manage users

User assignments:
  mukesh → Student role
  ram    → Teacher role
  sita   → Admin role
```

> **Real-world example:** In a hospital information system, doctors can view and update patient records, nurses can view records but have limited update rights, and billing staff can only see billing information — same system, different authorization levels.

---

## 5. Program Threats

**Program threats** are malicious programs or code that exploit software vulnerabilities.

### 5.1 Trojan Horse

A **Trojan Horse** is a program that appears **legitimate and useful** but contains **hidden malicious code** that executes secretly.

```
Appears to be:              Actually does:
┌─────────────────┐         ┌─────────────────────────────┐
│  Free Game      │         │  Free Game (works normally) │
│  Download!      │    +    │  + Secretly steals your     │
│                 │         │    passwords and sends to   │
└─────────────────┘         │    attacker                 │
                            └─────────────────────────────┘
```

**How it spreads:**
- Downloaded from untrusted websites.
- Email attachments.
- Bundled with legitimate software.
- Fake software updates.

**What Trojans can do:**
- Steal passwords and banking credentials.
- Give attacker remote control (Remote Access Trojan — RAT).
- Download additional malware.
- Encrypt files (ransomware).
- Use your computer to send spam or attack others.

**Named after:** Greek mythology — soldiers hidden inside a wooden horse gifted to Troy.

> **Real-world example:** A student downloads a "free" cracked version of Photoshop. It installs and works — but also silently installs a keylogger that captures every password typed. The student never knows until their accounts are hacked.

**Protection:**
- Download software only from official/trusted sources.
- Use antivirus software.
- Keep OS and software updated.
- Be suspicious of unexpected file attachments.

---

### 5.2 Trap Door (Back Door)

A **trap door** (back door) is a **secret, undocumented entry point** into a program or system, intentionally created by the developer.

```
Normal Login:                    Trap Door:
Username + Password              Special secret code
→ Authentication check           → Bypasses authentication
→ Access if correct              → Direct access regardless
```

**Origins:**
- Originally created by developers for **debugging and testing** during development.
- Forgotten or intentionally left in the final product.
- Can also be maliciously added by an attacker who gains code access.

**Example:**
```c
// Normal login check:
if (strcmp(entered_password, stored_password) == 0) {
    grant_access();
}

// With trap door (hidden by developer):
if (strcmp(entered_password, stored_password) == 0 ||
    strcmp(entered_password, "letmein_dev2024") == 0) {  // ← TRAP DOOR!
    grant_access();
}
```

**Real-world examples:**
- **NSA Backdoor:** Edward Snowden revealed that intelligence agencies pressured companies to include backdoors in encryption software.
- **Hardware backdoors:** Some network equipment manufacturers accused of including hidden remote access.

**Detection:** Difficult — requires complete source code audit.

**Protection:**
- Code reviews and security audits.
- Not shipping debug builds to production.
- Using open-source software where community can audit code.

---

### 5.3 Stack and Buffer Overflow

**Buffer:** A fixed-size memory region allocated for data.
**Buffer Overflow:** Writing more data than the buffer can hold, overwriting adjacent memory.

**How it works:**
```
Memory Layout (Stack):
┌──────────────────────────────┐ High address
│   Return Address             │ ← Attacker targets this!
├──────────────────────────────┤
│   Saved Frame Pointer        │
├──────────────────────────────┤
│   Local Variables            │
├──────────────────────────────┤
│   Buffer (allocated: 8 bytes)│ ← Only 8 bytes allocated
└──────────────────────────────┘ Low address

Attacker inputs 20 bytes → overwrites return address
→ When function returns, jumps to attacker's code!
```

**Simple C example of vulnerable code:**
```c
void vulnerable_function(char *input) {
    char buffer[8];           // Only 8 bytes allocated
    strcpy(buffer, input);    // No length check! UNSAFE!
    // If input is > 8 bytes, overflow occurs
}

int main() {
    char attack[] = "AAAAAAAAAAAAAAAAMALICIOUS_CODE";  // 30 bytes!
    vulnerable_function(attack);  // Buffer overflow!
}
```

**What attacker can achieve:**
- **Crash the program** (denial of service).
- **Execute arbitrary code** with the program's privileges.
- **Gain root/administrator access**.
- **Inject shellcode** — malicious machine code placed in the buffer.

**Stack Overflow vs Heap Overflow:**
- **Stack overflow:** Overflows a buffer on the stack (affects return addresses, local variables).
- **Heap overflow:** Overflows a buffer on the heap (affects dynamically allocated memory).

**Protection mechanisms:**
- **ASLR (Address Space Layout Randomization):** Randomizes memory addresses so attacker cannot predict where to jump.
- **Stack Canary:** Places a random value (canary) before return address — if changed, OS detects overflow.
- **DEP/NX bit (Data Execution Prevention):** Marks stack/heap as non-executable — attacker's code cannot run there.
- **Safe functions:** Use `strncpy` instead of `strcpy`, `fgets` instead of `gets`.
- **Bounds checking:** Always check input length before copying.

> **Real-world example:** The **Morris Worm (1988)** — first major internet worm — exploited a buffer overflow in the `fingerd` program on Unix. It infected thousands of computers (10% of the internet at the time) and caused millions in damage.

---

## 6. System Threats

**System threats** affect the entire operating system or network, rather than just individual programs.

### 6.1 Worms

A **worm** is a **self-replicating** malicious program that spreads across networks **without any user interaction** and without needing to attach to a host program.

```
Worm lifecycle:
Find vulnerable system → Exploit vulnerability → Copy itself
       ↑                                              ↓
       └────────────────── Repeat on new system ──────┘
```

**Difference from Virus:** A worm is **standalone** — does not need to attach to a file. It spreads on its own.

**How worms spread:**
- Network vulnerability exploitation.
- Email (self-sends itself to all contacts).
- Shared folders.
- Removable media.
- Instant messaging.

**Famous worm examples:**

| Worm | Year | Impact |
|---|---|---|
| **Morris Worm** | 1988 | First major worm — infected 6,000 computers (10% of internet) |
| **ILOVEYOU** | 2000 | Spread via email, overwrote files, $10 billion damage |
| **Code Red** | 2001 | Exploited IIS web server, 359,000 machines in 14 hours |
| **Slammer/Sapphire** | 2003 | Fastest spreading worm — doubled every 8.5 seconds |
| **WannaCry** | 2017 | Ransomware worm, infected 200,000 computers in 150 countries |

**Worm effects:**
- Consumes network bandwidth (slowing the entire network).
- Consumes CPU and memory (slowing infected system).
- Can carry payload (delete files, install ransomware, create botnet).
- Can cause complete network outages.

**Protection:**
- Keep OS and software patched/updated.
- Firewall to block unnecessary ports.
- Network intrusion detection systems.
- Disable unnecessary network services.

---

### 6.2 Viruses

A **virus** is malicious code that **attaches itself to legitimate programs or files** and spreads when those infected files are executed or shared. Unlike worms, viruses **need human action** to spread (running an infected file).

```
Virus lifecycle:
Attaches to host file → User runs infected file → Virus executes
→ Spreads to other files → Payload activates (damage)
```

**Types of Viruses:**

| Type | How it spreads | Example |
|---|---|---|
| **File Virus** | Infects executable files (.exe, .com) | Jerusalem virus |
| **Boot Sector Virus** | Infects MBR (Master Boot Record) | Michelangelo virus |
| **Macro Virus** | Infects document macros (Word, Excel) | Melissa virus |
| **Polymorphic Virus** | Changes its own code to evade detection | Polymorphic engines |
| **Stealth Virus** | Hides itself from antivirus | Various rootkits |
| **Multipartite Virus** | Infects both files and boot sector | Invader virus |

**Virus vs Worm:**

| Feature | Virus | Worm |
|---|---|---|
| Host needed | Yes (attaches to file) | No (standalone) |
| Spreads via | User action (running file) | Network automatically |
| Replication | Infects files | Copies itself across network |
| Example | ILOVEYOU (email attachment) | Morris Worm (network) |

**How antivirus works:**
- **Signature-based:** Compares file against database of known virus signatures.
- **Heuristic-based:** Detects suspicious behavior even without known signature.
- **Sandbox:** Runs suspicious file in isolated environment to observe behavior.

**Protection:**
- Keep antivirus software updated.
- Do not open suspicious email attachments.
- Download software only from official sources.
- Regular backups.
- Enable automatic OS updates.

> **Real-world example:** **ILOVEYOU worm/virus (2000)** — arrived as an email with subject "ILOVEYOU" and attachment "LOVE-LETTER-FOR-YOU.TXT.vbs". When opened, it overwrote image and audio files and emailed itself to all Outlook contacts. Infected 10 million PCs and caused $10 billion damage within 10 days.

---

### 6.3 Denial of Service (DoS)

A **Denial of Service (DoS)** attack attempts to make a system or network **unavailable to legitimate users** by overwhelming it with traffic or requests.

**Goal:** Not to steal data but to **disrupt service**.

```
Normal operation:               DoS Attack:
Legitimate users                Attacker sends
  → Server responds             millions of fake requests
  → Service available           → Server overwhelmed
                                → Cannot serve real users
                                → Service unavailable
```

**Types of DoS Attacks:**

**1. Volume-Based Attacks:**
- Flood the network with traffic.
- **UDP Flood, ICMP Flood (Ping of Death).**
- Measured in bits per second (bps).

**2. Protocol Attacks:**
- Exploit weaknesses in network protocols.
- **SYN Flood:** Sends thousands of TCP SYN packets but never completes the handshake. Server holds half-open connections until exhausted.
- Measured in packets per second (pps).

```
Normal TCP:           SYN Flood Attack:
Client → SYN →        Attacker → SYN → Server (× millions)
Server → SYN+ACK →   Server → SYN+ACK → (fake IP, no reply)
Client → ACK →        Server holds connections... exhausted!
Connection established
```

**3. Application Layer Attacks:**
- Target web applications.
- **HTTP Flood:** Sends millions of legitimate-looking HTTP GET requests.
- Hard to distinguish from real traffic.

**DDoS — Distributed Denial of Service:**
- Same as DoS but attack comes from **thousands or millions of compromised computers (botnet)** simultaneously.
- Much harder to block — traffic comes from many different IPs.

```
                    Attacker
                       ↓
              [Command & Control]
             /       |       \
        Bot1       Bot2      Bot3
        Bot4       Bot5      Bot6     ← (Compromised computers — BOTNET)
         \          |        /
          → → → Target Server ← ← ←
               (overwhelmed!)
```

**Famous DoS/DDoS attacks:**

| Attack | Year | Target | Impact |
|---|---|---|---|
| **GitHub DDoS** | 2018 | GitHub | 1.35 Tbps — largest at the time |
| **Dyn DNS DDoS** | 2016 | DNS provider | Twitter, Netflix, Reddit down for hours |
| **Estonia DDoS** | 2007 | Estonian gov | National infrastructure down for weeks |
| **AWS DDoS** | 2020 | Amazon AWS | 2.3 Tbps — current record |

**Protection against DoS/DDoS:**
- **Rate limiting:** Limit requests per IP per second.
- **Firewalls and ACLs:** Block suspicious IP ranges.
- **CDN (Content Delivery Network):** Distribute traffic across many servers.
- **Traffic analysis:** Detect and filter abnormal traffic patterns.
- **Black-holing:** Route attack traffic to null (discard).
- **DDoS protection services:** Cloudflare, AWS Shield.

---

## 7. Security Threats Summary

```
SECURITY THREATS
├── Program Threats (Software-based)
│   ├── Trojan Horse     → Disguised malicious program
│   ├── Trap Door        → Hidden entry point
│   └── Buffer Overflow  → Memory boundary violation
│
└── System Threats (Network/System-wide)
    ├── Worm             → Self-replicating, spreads via network
    ├── Virus            → Attaches to files, spreads via user action
    └── DoS/DDoS         → Overwhelms system to deny service
```

| Threat | Spreads By | Needs Host? | Target | Main Effect |
|---|---|---|---|---|
| Trojan Horse | Download/email | No (standalone) | User | Theft, remote access |
| Trap Door | Built into code | No | Developer installed | Unauthorized access |
| Buffer Overflow | Code exploit | No | Vulnerable program | Code execution, crash |
| Worm | Network (automatic) | No | Network | Bandwidth, payload |
| Virus | User action | Yes (file/MBR) | Files/system | Data destruction |
| DoS | Network flood | No | Server/network | Service disruption |

---

## 8. Exam Tips

Commonly asked questions for Unit 8:

1. **What is the CIA triad in security? Explain each component.**
2. **What is authentication? What are the three factors of authentication?**
3. **Explain password-based authentication. What are its weaknesses?**
4. **What is a salted hash? Why is salt used in password storage?**
5. **What is a One Time Password? How does it work? Give an example.**
6. **What is biometric authentication? List any three types with examples.**
7. **What is a Trojan Horse? How is it different from a virus?**
8. **What is a Trap Door (Back Door)? Give an example.**
9. **Explain Buffer Overflow attack with a diagram. How can it be prevented?**
10. **Differentiate between a Worm and a Virus.**
11. **What is a Denial of Service (DoS) attack? Explain SYN Flood.**
12. **What is a DDoS attack? How does a botnet work?**
13. **What is the difference between authentication and authorization?**

> **Key distinctions to remember:**
> - **Authentication** = WHO are you? | **Authorization** = WHAT can you do?
> - **Virus** needs a host file | **Worm** is standalone
> - **DoS** = one attacker | **DDoS** = many attackers (botnet)
> - **Trojan** = looks legitimate | **Virus** = attaches to files

---

## 9. References

- Andrew S. Tanenbaum, *Modern Operating System 6/e*, PHI, 2011/12 — Chapter 9
- Silberschatz, P.B. Galvin, G. Gagne, *Operating System Concepts 8/e*, Wiley India, 2014 — Chapter 15
