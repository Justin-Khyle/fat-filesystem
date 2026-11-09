# FAT File System in C

A working FAT-style file system built from scratch in C, running on a ~10 MB virtual disk volume. Built as a 5-person team for **CSC 415 – Operating Systems** at San Francisco State University (Summer 2026).

> **About this repo:** Course policy keeps the source code in a private class repository, so this repo documents the design, my contributions, and how we tested it. I'm happy to walk through the code in an interview.

---

## What it does

The file system formats a raw volume file, mounts it, and supports normal file operations through an interactive shell: creating and listing directories, moving and renaming files and folders, and reading and writing files through a buffered I/O layer. All data lives inside the volume file, so the file system survives being unmounted and mounted again.

## Volume layout

The volume is split into 512-byte blocks:

| Blocks | Contents |
|---|---|
| 0 | **Volume Control Block (VCB)**: magic number, block size, and where everything else starts |
| 1–153 | **File Allocation Table (FAT)**: one entry per block, linking each block to the next one in its file |
| 154–165 | **Root directory**: 54 directory entries |
| 166+ | **Data blocks**: file and directory contents |

On a fresh format, the volume has 19,531 blocks, with 19,365 of them free.

## Design decisions

**FAT instead of a bitmap for free space.** We weighed a bitmap against a FAT. A FAT tracks free space *and* links each file's blocks together in one structure, so a file doesn't need to sit in one contiguous run of blocks. Entries are `uint32_t` with sentinel values for free, end-of-chain, and reserved blocks.

**A running free-block counter.** The VCB keeps a `freeBlockCount`, so checking available space doesn't require scanning the whole FAT.

**Fixed-size directory entries.** Each entry is exactly 112 bytes, holding the name, location, size, timestamps, and a single flags byte. The flags byte stores "in use" and "is directory" as bit flags instead of separate booleans, which was feedback from our professor. A compile-time `_Static_assert` makes sure the struct never changes size by accident.

## My contributions

- **Directory entry format and root directory.** I wrote the directory entry and directory headers and `initRootDirectory()`, which sets up the root's `.` and `..` entries when the volume is formatted.
- **`mkdir`.** I implemented creating new directories. While reviewing my first version, I found a subtle bug: it assumed a directory's blocks were stored next to each other on disk. That's no longer true once any space has been freed and reused. I rewrote it to follow the FAT chain block by block, the same way directories are read back.
- **`seek` in the buffered I/O layer.** I implemented moving the read/write position within an open file, working with the file control block a teammate built.
- **`rename` / `mv`.** I implemented renaming files and moving them between directories. Moving a folder also updates its `..` entry, so it points to the new parent.

## Testing

We tested by formatting a fresh volume, mounting it again, and checking the raw bytes with a hex dump. For example, I confirmed that the VCB fields (FAT start, FAT size, root directory location, free block count) and the root directory's `.` and `..` entries were written exactly where the design said they should be.

## Teamwork

Five of us worked in one shared Git repository, committing straight to `main` as our professor required. That meant pulling and rebasing before every push and resolving merge conflicts when two people had edited the same file.

## Tech

C · gcc · Make · Linux (VM) · Git · hex dump inspection
