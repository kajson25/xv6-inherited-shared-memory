# xv6 Inherited Shared Memory

This project extends the **xv6 Operating System** to support shared memory inheritance between parent and child processes. It introduces a mechanism where a parent process can define regions of its memory to be shared, and these regions remain accessible and synchronized across its descendants even after an `exec()` call.

## 🚀 Key Features

- **Inherited Memory Regions**: Unlike standard IPC, this implementation allows memory mapping to persist through the `fork()` and `exec()` chain.
- **Virtual Memory Mapping (>1GB)**: Modified the xv6 memory layout to reserve the space starting at 1GB in the virtual address space for shared objects, preventing collisions with the standard process stack and heap.
- **Kernel-Level Synchronization Data**: The `proc` structure was modified to track up to 10 shared memory metadata structures (Name, Virtual Address, Size).
- **Page Table Sharing**: Optimized `exec()` and `fork()` to ensure child processes map to the same physical frames as the parent for designated "shared" blocks.

## 🛠 New System Calls

| System Call | Description |
| :--- | :--- |
| `int share_data(char *name, void *addr, int size)` | Called by the parent to register a memory region for sharing. Returns a unique ID (0-9). |
| `int get_data(char *name, void **addr)` | Called by a child to retrieve the virtual address pointer of a shared region registered by its parent. |

## 💻 Included Applications (The Dalle-coMMa-liSa Suite)

To demonstrate the shared memory capabilities, a text analysis suite was implemented:
- **`dalle` (The Coordinator)**: The parent process. It initializes 9 different shared memory structures (storing paths, word counts, longest/shortest words, and command indicators) and spawns its children.
- **`liSa` (The Worker)**: A background process that parses a text file, finds the longest/shortest words in each sentence, and updates the shared memory buffer in real-time.
- **`coMMa` (The Interface)**: An interactive command-line tool that reads from the shared memory updated by `liSa` to show the user "latest" results and "global extrema" without re-reading the file.

## 🔧 Technical Implementation Details

1. **`fork()` Modification**: Updated to copy the shared structure metadata from parent to child. It also saves a reference to the parent's page directory.
2. **`exec()` Modification**: When a process executes a new binary, the system identifies inherited shared regions and re-maps those virtual addresses (starting at 1GB) to the same physical memory frames used by the parent.
3. **Memory Cleanup**: Enhanced the process termination logic to ensure that physical frames are only freed when the last process in the parent-child tree referencing them is destroyed.

## 📦 Setup and Execution

1. **Clone and Build**:
   ```bash
   git clone https://github.com/YourUsername/xv6-inherited-shared-memory.git
   cd xv6-inherited-shared-memory
   make qemu
