# Distributed File System (DFS)

## Overview
This project implements a robust and feature-rich **Distributed File System (DFS)** in C, designed to handle multiple clients and multiple storage servers concurrently. It achieves structured file distribution, access control, and advanced file manipulation features like streaming, checkpointing, and remote execution.

The architecture strictly follows a Client-Server and Server-Server communication model utilizing TCP sockets, managed centrally via a Naming Server.

## System Architecture
The system consists of three main components:
1. **Naming Server (NM)**: The central coordinator that keeps track of all active Storage Servers (SS) and the files they contain. It handles client requests, resolves file locations, and manages permissions and access control.
2. **Storage Servers (SS)**: Distributed endpoints that actually store the file chunks. Storage Servers dynamically connect to the Naming Server and serve file operations. 
3. **Clients**: The users interacting with the file system. Clients query the Naming Server for file locations/permissions, and then directly communicate with Storage Servers for heavy data operations like reading, writing, and streaming.

---

## Supported Features

### 1. Basic File and Folder Operations
- **File Management**: Create, delete, read, and write files. Write operations support sentence-level concurrency and locks (using a commit command like `ETIRW` to save).
- **Folder Management**: Create folders, view folder contents, and move files across the file system.
- **System Listing**: View available files via LIST or VIEW commands, with options to fetch detailed info (word counts, characters, last access time, and file owner).

### 2. Multi-user Concurrency and Access Control
- **User Discovery**: Track multiple clients dynamically.
- **Ownership & Permissions**: The user who creates a file becomes its owner.
- **Access Delegation**: 
  - Owners can grant read (`-R`) or write (`-W`) access uniquely to other target users.
  - Owners can revoke access seamlessly.
- **Concurrent Access**: Implements reader-writer semantics. If a file sentence is locked, other clients attempting to edit will receive an `ERR_SENTENCE_LOCKED` error.

### 3. Checkpointing and Revisions (Version Control)
- **Tags**: Safely track states of files by creating checkpoints with custom tags.
- **Version History**: List all checkpoints for a file or view a specific checkpoint tag's contents.
- **Reverting**: Rollback file states seamlessly to an older checkpoint tag or undo the most recent action directly.

### 4. Advanced Streaming and Execution
- **Streaming Files**: Avoid large chunk downloads by streaming files continuously block-by-block.
- **Remote Execution**: Process script executables securely through the filesystem execution requests.

---

## How to Build and Run

### 1. Building the Project
Simply compile the entire DFS codebase using the provided Makefile:
```bash
make clean
make
```

### 2. Testing System Setup (Automated)
You can simplify testing and setting up a Naming Server and two local Storage servers using the included Bash script:
```bash
chmod +x test_system.sh
./test_system.sh
```
*This will automatically spin up a Naming Server on port `8080` and Storage Servers on `9001` and `9002`.*

**Stopping Servers:** Run `bash stop_servers.sh` (generated automatically by the test script) to cleanly terminate all background servers.

### 3. Running Servers Manually
If you want to deploy servers manually on different machines:

**Start the Naming Server:**
```bash
./naming_server <PORT>
# Example: ./naming_server 8080
```

**Start a Storage Server:**
```bash
./storage_server <NM_IP> <NM_PORT> <SS_PORT> <STORAGE_DIR>
# Example: ./storage_server 127.0.0.1 8080 9001 storage1
```

### 4. Running the Client
To connect a client to the Naming Server:
```bash
./client <NM_IP> <NM_PORT>
# Example: ./client 127.0.0.1 8080
```
Upon running, you'll be prompted to input a `username` (e.g. `user1`). This username binds commands to your user role for accurate access control handling across the system. 
