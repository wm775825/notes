#                          Google File System

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

### 2.  Architecture

single master + multiple chunk servers + multiple clients.

no cache on server side, cache for metadata on client side.

#### 1. single master

It maintains all file system metadata: namespace, access control info, mapping from files to chunks(i.e. filename -> list of chunk global index?), the current locations of chunks(i.e. chunk global index -> ip : port?).

 A read example:

> client -> master server:  filename, offset
>
> client <- master server:  chunk handle, chunk replicas locations
>
> client -> chunk server: chunk handle, byte range
>
> client  <- chunk server: chunk data

#### 2. chunk size

default: 64 MB.

























