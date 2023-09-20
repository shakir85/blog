---
title: "Understanding inodes"
description: Exlpore the fundamental concept of inodes and their role in file system management
# slug: hello-world
date: 2023-08-16T23:17:40-07:00
# image: cover.jpg
categories: ["linux"]
tags: ["good to know"]
# weight: 1
draft: false
---

In Linux, and other Unix-like operating systems, an inode (short for "index node") is a data structure that stores metadata about a file or directory. Each file or directory on a filesystem is associated with a unique inode, which contains information such as:

- File type (regular file, directory, symbolic link, device file, etc.).
- Permissions (read, write, execute) for the owner, group, and others.
- Ownership (user and group) of the file.
- File size in bytes.
- Timestamps indicate when the file was accessed and modified and when the inode itself was last modified.
- The number of hard links to the file.
- Disk block pointers point to the data blocks on the storage device.

!["Example: inode of an Empty File"](example.png)

In this post, I'll cover the essentials of inodes – how to identify issues tied to them, and ways to troubleshoot. Keep in mind that inodes can become quite complex, touching various aspects of the filesystem and storage structure – though, for the sake of this blog, we won't delve into all those complexities.

## Why do we need inodes?

Inodes are crucial in managing file metadata and data storage on a filesystem. When you create a file or directory, an inode is allocated to store its metadata, and data blocks are allocated to hold the actual content. Inodes allow the filesystem to manage file access and storage efficiently and simplify operations like finding files by name, checking permissions, and tracking file sizes.

## Inode capacity

For a filesystem, there is a quota for its inodes. The number of inodes available on a filesystem is determined during the filesystem creation (formatting) and is usually fixed. This means that if a filesystem runs out of available inodes, you may be unable to create new files or directories, even if there is a free disk space.

### In inode, we trust! 

So, there is a capacity for inodes—great. But what happens if we run out of them?

Running out of inodes, often called "inode exhaustion," can lead to weird system failures. You might typically suspect that the system has run out of available inodes when a program or process fails to create a file or directory due to insufficient storage, as an error message indicates.

Overall, it is unlikely to run out of inodes before running out of storage. But this is not impossible, especially when there are a lot of processes writing small files constantly for a long time.

## Reasons for inodes exhaustion

Inode exhaustion can be caused by (but not limited to) the following factors:

### Small Files and Directories

If your filesystem is primarily composed of a ton of very small files and directories, each of these entries will consume an inode. This can quickly deplete the available inode pool, even if there is plenty of free space on the disk.

### Lots of Small Writes

Frequent small writes, such as those caused by a lot of logging or temporary file creation, can contribute to inode exhaustion. These small writes create new inodes each time, and over time, this can lead to a depletion of available inodes.

### Temporary Files

Applications that generate many temporary files without cleanup can exhaust inodes. These temporary files accumulate and consume inodes if they are not managed or removed.

### Software Builds and Compilation

Software development processes involving frequent compilation and building of code can create many temporary files and directories. For example, building node apps is known for creating a ton of small files and cache. Over time, these files can contribute to inode exhaustion.

### Excessive Filesystem Operations

Certain applications or scripts that frequently create, modify, or delete files and directories can lead to inode exhaustion if not properly managed.

### Other Factors

Other factors such as mail servers, content management systems (e.g., WordPress server), backup servers can also be reasons for running out of inodes.

## How to fix inode exhaustion?

First, ensure whether you’re running out of inodes or not:
!["Check available inodes"](check_available_inodes.png)

You might think that just restarting the server would fix the problem, but that's not entirely right. The issue is more about the filesystem itself and not the operating system processes.

The filesystem is persisted on disk, so a reboot does not modify or alter the underlying filesystem structure or its properties. If the filesystem was already experiencing inode exhaustion before the reboot, the same limitation will persist after the reboot.

A reboot *might* be helpful only to clear out any stale or stuck processes that keep trying to write tiny files. But the reboot itself does not directly fix the underlying filesystem issue.

### Identify the issue

- Monitor filesystem usage and identify when inode exhaustion occurs.
- Review error messages, logs, and filesystem reports to determine the cause of the issue.
- If you encounter an error message indicating insufficient storage capacity, despite being certain that there is available space, it may suggest an inode issue.

### Cleanup and delete unnecessary files

- Identify and delete unnecessary files, especially small and temporary files.
- Check out the `/tpm` directory; applications usually use this location for scratch/cache data.
- Clear the cache of package managers by using their command line options or manually locate and remove their cache directory.
- Use tools like `find` to locate and delete no longer needed files.

### Storage optimization

- Consider compressing unused files.
- Remove redundant files.
- Avoid excessive nested directories.
- Consider using alternative storage like a NAS or other distributed/network filesystem.

### Prevent it from happening again

- Implement regular cleanup and maintenance tasks.
- If it is a recurring issue, it might be a good idea to get a storage expert involved. They can dig in deeper and think about taking stronger actions, like
reorganizing and tidying up the whole filesystem.
- If you're running into this problem often while compiling software in your CI/CD pipeline, you might want to think about using a Docker container to build the app and then stash away the build artifacts/stuff using Docker bind mounts or [docker exporter](https://docs.docker.com/build/exporters/). Then delete the build container (which contains the filesystem).
