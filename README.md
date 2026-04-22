# 🔍 Process Inspector

A lightweight Linux command-line tool written in C that inspects any running process by reading directly from the `/proc` virtual filesystem. Given a PID, it displays process metadata and a categorized, human-readable memory layout map.

---

## 📋 Features

- **Process Information** — Displays the process name, PID, current state, and thread count from `/proc/<pid>/status`
- **Memory Layout Map** — Parses `/proc/<pid>/maps` and categorizes every memory region into:

| Section | Description |
|---------|-------------|
| `TEXT`  | Executable code / binary instructions |
| `DATA`  | Global and static variables |
| `HEAP`  | Dynamically allocated memory (`malloc`/`new`) |
| `STACK` | Local variables and call frames |
| `VDSO`  | Virtual Dynamic Shared Object |
| `VVAR`  | Virtual Variables (kernel-exported read-only data) |
| `OTHER` | Shared libraries, memory-mapped files, anonymous regions |

---

## 🖥️ Requirements

- **OS:** Linux only (relies on `/proc` filesystem)
- **Compiler:** GCC or any C99-compatible compiler
- **Privileges:** You can inspect your own processes freely; inspecting other users' processes may require `sudo`

---

## 🚀 Build & Run

```bash
# Compile
gcc -Wall -o process_inspector process_inspector.c

# Run
./process_inspector
```

You will be prompted to enter the PID of the process to inspect:

```
--------------------------------------------------------------
---------------- Marvellous Process Inspector ----------------
--------------------------------------------------------------
Enter the pid of process that you want to inspect:
1234
```

To find the PID of a running process:
```bash
pidof <process_name>   # e.g., pidof bash
pgrep <process_name>
ps aux | grep <process_name>
```

---

## 📄 Example Output

```
--------------------------------------------------------------
-------------------- Process Information ---------------------
--------------------------------------------------------------
Name:   bash

Pid:    1234

State:  S (sleeping)

Threads:        1

--------------------------------------------------------------
Accessing the file: /proc/1234/maps
--------------------------------------------------------------
----------------------- Memory Layout ------------------------
--------------------------------------------------------------
55a1b2c3d000 55a1b2c3f000 8 r--p TEXT /usr/bin/bash
55a1b2c3f000 55a1b2d6e000 1212 r-xp TEXT /usr/bin/bash
7f3c4d500000 7f3c4d528000 160 r--p OTHER /usr/lib/x86_64-linux-gnu/libc.so.6
7ffd12345000 7ffd12366000 132 rw-p STACK [stack]
--------------------------------------------------------------
```

---

## 🛠️ How It Works

1. The user provides a PID.
2. **`ShowProcessInformation()`** opens `/proc/<pid>/status` and filters key fields (Name, Pid, State, Threads).
3. **`showMemoryLayout()`** opens `/proc/<pid>/maps`, parses each line with `sscanf`, and passes the permission flags and region name to **`GetSectionType()`** which classifies the region.
4. Each region is printed with its start/end address, size in KB, permissions, section type, and mapped file/name.

---

## 🐛 Known Limitations

- Line buffer in `showMemoryLayout` is fixed at 80 bytes; very long map entries (e.g., long file paths) will be truncated. Increase `line[80]` to `line[512]` for safer parsing.
- Only tested on Linux kernel 4.x and above.
- Does not display `vsyscall` region (kernel-mapped, not always visible in `/proc/maps`).

---

## 📁 File Structure

```
.
├── process_inspector.c   # Main source file
└── README.md             # This file
```

---

## 📜 License

This project is released under the [MIT License](https://opensource.org/licenses/MIT). Feel free to use, modify, and distribute.
