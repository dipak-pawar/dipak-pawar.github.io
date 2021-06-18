---
author: "Dipak Pawar"
title: "Sparse File"
date: "2021-05-22"
tags: ["TIL", "linux", "storage"]
ShowToc: true
TocOpen: true
weight: 2
---

*Do you know that we can fake the size of file?* Yes, you read it correctly. We can fake the size of file. In this article, will focus on *sparse files* & more details about it.

### What's a sparse file?
From Wikipedia:

 *In computer science, a sparse file is a type of computer file that attempts to use file system space more efficiently when the file itself is partially empty. This is achieved by writing brief information (metadata) representing the empty blocks to disk instead of the actual "empty" space which makes up the block, using less disk space. The full block size is written to disk as the actual size only when the block contains "real" (non-empty) data.*

Files having blocks which contains zero, but not stored on disk are sparse files. So If we have 10 GB of sparse file, it's not going to take less space on filesystem assuming it has zero blocks.

### Why
When, we have limited disk space & need to allocate space for multiple users/programs which never fills that space completely e.g. virtual machine.

If virtual machine never fills the disk, then certain amount of disk will have zeros in it. Sparse files permits us to save disk space in this.

### How
Sparse files doesn't store the zeros on disk, instead it stores metadata describing zeros which can be generated.

When reading sparse files, the filesystem converts metadata representing zeros into real blocks filled with null bytes at runtime.

#### Create Sparse File
```
# create sparse file using truncate
[dipak@localhost ~]$ truncate -s 100MiB /tmp/sparse-file

or

# create sparse file using dd
[dipak@localhost ~]$ dd of=/tmp/sparse-file bs=10M seek=10 count=0
0+0 records in
0+0 records out
0 bytes copied, 0.000121888 s, 0.0 kB/s

# check occupied space
[dipak@localhost ~]$ ls -als /tmp/sparse-file
0 -rw-rw-r--. 1 dipak dipak 104857600 Jun 18 22:46 /tmp/sparse-file

or

[dipak@localhost ~]$ du -hs /tmp/sparse-file
0	/tmp/sparse-file

```

#### Convert existing file into sparse
On Linux, an existing file can be converted to sparse by: `fallocate -d <filename>`

```
# create file using dd
[dipak@localhost ~]$ dd if=/dev/zero of=/tmp/output bs=100M count=4
4+0 records in
4+0 records out
419430400 bytes (419 MB, 400 MiB) copied, 0.193488 s, 2.2 GB/s

# check occupied space
[dipak@localhost ~]$ stat /tmp/output
  File: /tmp/output
  Size: 419430400 	Blocks: 819200     IO Block: 4096   regular file
Device: 31h/49d	Inode: 2957        Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/   dipak)   Gid: ( 1000/   dipak)
Context: unconfined_u:object_r:user_tmp_t:s0
Access: 2021-06-18 23:03:20.745563236 +0530
Modify: 2021-06-18 23:03:20.008545895 +0530
Change: 2021-06-18 23:03:20.008545895 +0530
 Birth: -

# covert to sparse files
[dipak@localhost ~]$ fallocate -d /tmp/output

# check occupied space
[dipak@localhost ~]$ fallocate -d /tmp/output
[dipak@localhost ~]$ stat /tmp/output
  File: /tmp/output
  Size: 419430400 	Blocks: 0          IO Block: 4096   regular file
Device: 31h/49d	Inode: 2957        Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/   dipak)   Gid: ( 1000/   dipak)
Context: unconfined_u:object_r:user_tmp_t:s0
Access: 2021-06-18 23:04:22.349012691 +0530
Modify: 2021-06-18 23:04:22.284011162 +0530
Change: 2021-06-18 23:04:22.284011162 +0530
 Birth: -
```

The the number of blocks now is 0 as the blocks with empty bytes were de-allocated. Note that size remains the same which is logical size of file, not it's size on disk.
