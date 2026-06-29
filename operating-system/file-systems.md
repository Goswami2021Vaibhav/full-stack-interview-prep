# File Systems

_Part of [Operating System](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. What is a file system?](#1-what-is-a-file-system)
- [2. What is a file descriptor?](#2-what-is-a-file-descriptor)
- [3. What is an inode?](#3-what-is-an-inode)

**🟡 Medium**
- [4. What's the difference between hard links and symbolic links?](#4-whats-the-difference-between-hard-links-and-symbolic-links)
- [5. What are common file allocation methods?](#5-what-are-common-file-allocation-methods)
- [6. What is a journaling file system?](#6-what-is-a-journaling-file-system)
- [7. What's the difference between sequential and direct (random) file access?](#7-whats-the-difference-between-sequential-and-direct-random-file-access)
- [8. What is file system fragmentation?](#8-what-is-file-system-fragmentation)

**🔴 Hard**
- [9. How does the OS keep track of free disk space?](#9-how-does-the-os-keep-track-of-free-disk-space)
- [10. What's the conceptual difference between FAT and a modern journaling file system like ext4/NTFS?](#10-whats-the-conceptual-difference-between-fat-and-a-modern-journaling-file-system-like-ext4ntfs)
- [11. How does a journaling file system recover from a crash?](#11-how-does-a-journaling-file-system-recover-from-a-crash)
- [12. What's the relationship between an inode and a directory entry?](#12-whats-the-relationship-between-an-inode-and-a-directory-entry)

---

### 1. What is a file system? 🟢

- The OS component that organizes how data is **named, stored, and retrieved** on a storage device — provides the abstraction of files and directories on top of raw disk blocks/sectors.

[↑ Back to top](#table-of-contents)

---

### 2. What is a file descriptor? 🟢

- A small integer handle a process uses to refer to an **open** file (or socket, pipe) — returned by the OS when a file is opened, and used in subsequent read/write/close calls instead of the file path itself.

[↑ Back to top](#table-of-contents)

---

### 3. What is an inode? 🟢

- A data structure storing a file's **metadata** (size, permissions, timestamps, and pointers to where its actual data blocks live on disk) — notably, the inode does **not** store the file's name (that lives in a directory entry, Q12).

[↑ Back to top](#table-of-contents)

---

### 4. What's the difference between hard links and symbolic links? 🟡

- **Hard link**: a second directory entry pointing to the **same inode** — both names are equally "real," and the file's data is only actually deleted once **every** hard link to it is removed.
- **Symbolic link**: a separate small file that just **stores the path** to another file — if the original is deleted/moved, the symlink becomes a "dangling" broken reference, since it doesn't share the same inode at all.

[↑ Back to top](#table-of-contents)

---

### 5. What are common file allocation methods? 🟡

- **Contiguous**: a file's blocks are stored in one continuous run on disk — fast sequential access, but suffers from external fragmentation and requires knowing the file's final size upfront.
- **Linked**: each block holds a pointer to the next block — no fragmentation issue, but poor for random access (must follow the chain from the start) and a broken pointer corrupts everything after it.
- **Indexed**: a dedicated index block holds pointers to all of a file's data blocks — supports efficient random access without contiguous storage, the basis for how inodes typically organize block pointers.

[↑ Back to top](#table-of-contents)

---

### 6. What is a journaling file system? 🟡

- Keeps a **log (journal)** of intended changes **before** actually committing them to the main file system structures — if the system crashes mid-write, the journal lets the file system replay or roll back incomplete operations on next boot, rather than leaving the file system in a corrupted, inconsistent state (Q11).

[↑ Back to top](#table-of-contents)

---

### 7. What's the difference between sequential and direct (random) file access? 🟡

- **Sequential access**: reading/writing a file strictly in order, from beginning to end — simple, and well-suited to allocation methods like linked allocation.
- **Direct/random access**: jumping directly to any specific position/block in a file without reading everything before it — needed for things like database files, which requires an allocation method (indexed) that supports efficient arbitrary lookups.

[↑ Back to top](#table-of-contents)

---

### 8. What is file system fragmentation? 🟡

- Over time, as files are created/deleted/resized, a file's blocks can end up **scattered** non-contiguously across the disk — slowing down access (especially on spinning hard drives, where seeking between scattered locations is expensive) since reading the "whole" file means jumping around rather than reading one continuous stretch. Less of a performance concern on SSDs, which don't have the same mechanical seek penalty.

[↑ Back to top](#table-of-contents)

---

### 9. How does the OS keep track of free disk space? 🔴

- Common approaches: a **free space bitmap** (one bit per block, marking it free/used — compact and fast to scan for contiguous free runs), or a **linked list of free blocks** (each free block points to the next free one — simple, but slower to find a large contiguous run). Most modern file systems use bitmap-based approaches, often combined with additional structures to track free space efficiently at scale.

[↑ Back to top](#table-of-contents)

---

### 10. What's the conceptual difference between FAT and a modern journaling file system like ext4/NTFS? 🔴

- **FAT** (File Allocation Table): uses a simple linked-list-style table mapping each block to the next block in a file's chain — no journaling, so an unexpected crash/power loss can leave the file system in an inconsistent state requiring a full scan (`chkdsk`/`fsck`) to repair.
- **ext4/NTFS**: journaling file systems (Q6) — track pending changes in a log first, enabling fast, reliable crash recovery without needing a full disk scan, plus generally richer metadata (permissions, larger file/volume size limits) than FAT supports.

[↑ Back to top](#table-of-contents)

---

### 11. How does a journaling file system recover from a crash? 🔴

- On reboot, it checks the journal for any **incomplete** transactions that were logged but never fully applied — it can either **replay** them forward to completion (if the journal recorded enough information to finish safely) or **discard** them entirely, restoring the file system to its last fully-consistent state, all without needing to scan the entire disk to find inconsistencies.

[↑ Back to top](#table-of-contents)

---

### 12. What's the relationship between an inode and a directory entry? 🔴

- A **directory** is itself just a special file containing a list of **(filename, inode number)** pairs — the directory entry maps a human-readable name to the inode holding the actual metadata/data pointers. This separation is exactly why multiple names (hard links, Q4) can point to the same underlying file: multiple directory entries, in the same or different directories, can reference the **same** inode number.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why can a hard link not cross file system/partition boundaries, while a symbolic link can?

[↑ Back to top](#table-of-contents)

---
