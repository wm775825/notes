# Google File System

## 1. Introduction

Feature:

- scalable, distributed

- fault tolerance, unreliable inexpensive commodity hardware

- high aggregate performance to a large number of clients

## 2. Design Overview

### 1. Some Points

- Components failures are norm rather than exception. => constant monitoring, error detection, fault tolerance, automatic recovery.

- Files are huge by traditional standards. => revisit I/O operations and block sizes.

- For write, most append rather than overwrite, random writes within a file are non-existent.

  For read, only read once written. Large streaming reads and small random reads.

  ​				=> append and atomicity guarantees are focus.

- co-design the file system API and applications.

  Most target apps require processing data in bulk at high rates rather than quick response for an individual read/write operation. => high bandwidth is more important than low latency.

### 2.  