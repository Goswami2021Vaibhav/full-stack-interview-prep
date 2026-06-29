# Child Processes

_Part of [Node.js](README.md) interview notes._

> Scaling across multiple processes/cores via the `cluster` module is covered in [Clustering & Scaling](clustering-and-scaling.md). This file covers spawning and managing individual child processes/threads.

## Table of Contents

**🟢 Easy**
- [1. What's the difference between `spawn()` and `fork()`?](#1-whats-the-difference-between-spawn-and-fork)
- [2. What is a daemon process?](#2-what-is-a-daemon-process)

**🟡 Medium**
- [3. How does Node.js handle child processes/threads?](#3-how-does-nodejs-handle-child-processesthreads)
- [4. What are Worker Threads, and how do they differ from child processes?](#4-what-are-worker-threads-and-how-do-they-differ-from-child-processes)
- [5. How do you execute an external program from within Node.js?](#5-how-do-you-execute-an-external-program-from-within-nodejs)

**🔴 Hard**
- [6. How do you kill child processes that themselves spawned their own child processes?](#6-how-do-you-kill-child-processes-that-themselves-spawned-their-own-child-processes)
- [7. How does Node.js support multi-processor platforms — does a single process use all cores?](#7-how-does-nodejs-support-multi-processor-platforms--does-a-single-process-use-all-cores)
- [8. When would you choose Worker Threads over the `cluster` module, or vice versa?](#8-when-would-you-choose-worker-threads-over-the-cluster-module-or-vice-versa)
- [9. What's the difference between `child_process.exec` and `execFile`?](#9-whats-the-difference-between-child_processexec-and-execfile)

---

### 1. What's the difference between `spawn()` and `fork()`? 🟢

- `spawn()`: launches **any** external command/executable as a new process, communicating via streams (stdout/stderr) — general-purpose.
- `fork()`: a specialized `spawn()` specifically for launching **another Node.js script** as a child process, automatically setting up an IPC (inter-process communication) channel for passing messages back and forth.

```js
const { spawn, fork } = require('child_process');

const ls = spawn('ls', ['-la']);
const child = fork('./worker.js');
child.send({ task: 'process' });
```

[↑ Back to top](#table-of-contents)

---

### 2. What is a daemon process? 🟢

- A background process that runs independently of any controlling terminal/parent session, typically started at boot and running indefinitely to provide a service (e.g. a database server, a logging agent) — not specific to Node, but a Node app can be run/managed as one (e.g. via PM2 or a systemd service).

[↑ Back to top](#table-of-contents)

---

### 3. How does Node.js handle child processes/threads? 🟡

- The `child_process` module (`spawn`/`exec`/`execFile`/`fork`) launches separate OS processes, each with their **own** memory space and V8 instance — communication happens via IPC messages or stdio streams, not shared memory.
- `worker_threads` (Q4) instead runs additional JS threads **within the same process**, optionally sharing memory directly.

[↑ Back to top](#table-of-contents)

---

### 4. What are Worker Threads, and how do they differ from child processes? 🟡

- `worker_threads` lets you run JS on a separate **thread**, inside the **same process** — lighter-weight to spawn than a full child process, and can share memory directly via `SharedArrayBuffer`, unlike child processes which must serialize messages over IPC.
- Best for CPU-bound work (heavy computation, image processing) that would otherwise block the main event loop, without the overhead of a whole new process.

```js
const { Worker } = require('worker_threads');
const worker = new Worker('./heavy-task.js');
worker.on('message', (result) => console.log(result));
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you execute an external program from within Node.js? 🟡

```js
const { exec } = require('child_process');

exec('ls -la', (err, stdout, stderr) => {
  if (err) return console.error(err);
  console.log(stdout);
});
```

[↑ Back to top](#table-of-contents)

---

### 6. How do you kill child processes that themselves spawned their own child processes? 🔴

- Killing the direct child (`child.kill()`) doesn't automatically kill **its** descendants — they can become orphaned, still-running processes. On Unix, spawn the child in its own **process group** (`detached: true`) and kill the entire group with a negative PID (`process.kill(-child.pid)`); on Windows, you typically need a tool like `tree-kill` since process groups work differently.

```js
const child = spawn('some-command', [], { detached: true });
process.kill(-child.pid); // kills the whole process group, including grandchildren
```

[↑ Back to top](#table-of-contents)

---

### 7. How does Node.js support multi-processor platforms — does a single process use all cores? 🔴

- A single Node.js process's JS execution runs on **one core** at a time (single main thread). To actually utilize multiple cores, you either spawn multiple processes (via `cluster` or manually with `child_process`/PM2) or use `worker_threads` for parallel CPU work within one process — Node doesn't automatically spread one process's JS execution across multiple cores on its own.

[↑ Back to top](#table-of-contents)

---

### 8. When would you choose Worker Threads over the `cluster` module, or vice versa? 🔴

- **Worker Threads**: best for offloading **CPU-intensive computation** (image processing, large data transformations) from the main thread, within a single process — lighter weight, can share memory.
- **`cluster`**: best for **scaling a network server** (like an HTTP API) across multiple cores, where each worker is an independent process handling its own share of incoming connections — not really meant for sharing in-memory state between workers.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between `child_process.exec` and `execFile`? 🔴

- `exec()`: runs a command through a **shell** — supports shell syntax (pipes, wildcards, env var expansion), but is riskier with untrusted input (shell injection) since the shell interprets the string.
- `execFile()`: runs an executable **directly**, without spawning a shell — faster, and safer against shell injection since arguments are passed as an array, not interpolated into a shell command string.

```js
// Risky if `userInput` is untrusted — shell interprets the whole string
exec(`ls ${userInput}`);

// Safer — arguments passed directly, no shell interpretation
execFile('ls', [userInput]);
```

> [!IMPORTANT]
> **Follow-up questions:**
> - Why specifically does `exec()` create a shell-injection risk that `execFile()`/`spawn()` (with an args array) avoids?

[↑ Back to top](#table-of-contents)

---
