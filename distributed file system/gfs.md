#                                                    Google File System

## 1. Introduction

Feature:

- scalable, distributed

- fault tolerance, unreliable inexpensive commodity hardware

- high aggregate performance to a large number of clients

## 2. Design Overview

### 2.1. Some Points

- Components failures are norm rather than exception. => constant monitoring, error detection, fault tolerance, automatic recovery.

- Files are huge by traditional standards. => revisit I/O operations and block sizes.

- For write, most append rather than overwrite, random writes within a file are non-existent.

  For read, only read once written. Large streaming reads and small random reads.

  ​				=> append and atomicity guarantees are focus.

- co-design the file system API and applications.

  Most target apps require processing data in bulk at high rates rather than quick response for an individual read/write operation. => high bandwidth is more important than low latency.

### 2.2.  Architecture

single master + multiple chunk servers + multiple clients.

no cache on server side, cache for metadata on client side.

#### 2.2.1. single master

It maintains all file system metadata: namespace, access control info, mapping from files to chunks(i.e. filename -> list of chunk global index?), the current locations of chunks(i.e. chunk global index -> ip : port?).

 A read example:

> client --> master server:  filename, offset
>
> client <-- master server:  chunk handle, chunk replicas locations
>
> client --> chunk server: chunk handle, byte range
>
> client <-- chunk server: chunk data

#### 2.2.2. chunk size

Large chunk size, default: 64 MB. Lazy space allocation.

pros:

- multiple access to a big file only one interaction between client and master.
- multiple access to a big file only one persistent TCP connection to reduce network overhead.
- reduce the size of metadata so that:
  - reduce overhead of storage in memory.
  - master can keep it in the memory.
  - clients can cache the metadata.

cons:

- a chunk per small file => internal fragments.
- multiple accesses to a small file(maybe only one chunk) make it a hotspot. Measures:
  - store it with a higher replication factor
  - stagger access times.

#### 2.2.3. Metadata

Three types of metadata:

- the file and chunk namespace;
- the mapping from files to chunks;
- the locations of each chunk's replicas.

A Write-Ahead-Log for the first two types of metadata to keep persistent.

The third type not persistent. Master asks each chunk server when it(master) starts and when a chunk server join.

All metadata is kept in the memory of the master.

Less than 64 bytes of metadata for each 64 MB chunk. Less than 64 bytes of file namespace for each file.

**The cost of adding extra memory to the master is a small price to pay for simplicity, reliability, performance and flexibility gained by storing metadata in memory.**



















