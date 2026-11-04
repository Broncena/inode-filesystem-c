# Inode Filesystem in C

A filesystem implemented in C that stores files and directories
inside a disk-image file.

The project explores how directory entries, inodes, block allocation,
and file descriptors work together to organize persistent data.

Originally developed for an operating systems course using supplied
support libraries and a Google Test suite.

## Features

- Format, mount, and unmount filesystem images.
- Create regular files and nested directories.
- Open and close files using file descriptors.
- Read, write, and seek within files.
- List directory contents.
- Move files and directories.
- Create hard links with inode reference counting.
- Remove files and empty directories, reclaiming allocated blocks.

## Storage Design

The filesystem uses 4 KiB blocks within a 256 MiB backing store.

Each inode records file metadata and references its data through:

- Six direct block pointers.
- One single-indirect block pointer.
- One double-indirect block pointer.

Small files access their blocks directly. Larger files use additional
pointer blocks to address more data.

Directory entries map names to inode numbers. File descriptors track
the current position for read, write, and seek operations.

## Build and Test

The bundled support libraries target Linux x86-64.

Requirements:

- C and C++ compilers
- CMake 3.25 or newer
- Google Test development libraries

On Ubuntu 24.04, install the build dependencies:

```sh
sudo apt-get update
sudo apt-get install -y cmake g++ make libgtest-dev
```

From the repository root:

```sh
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

GitHub Actions runs the same build and tests on Ubuntu 24.04.
The tests run together because later cases reuse images created by
earlier cases. Generated images stay in `build/test-images/`.

## Test Coverage

The provided Google Test suite exercises:

- Formatting, mounting, and unmounting.
- File and directory creation.
- File descriptor allocation and closing.
- Directory listings.
- Writes across direct and indirect block boundaries.
- Reads, seeking, and storage exhaustion.
- File removal, moves, and hard links.
- Invalid paths and arguments.

## Project Structure

```text
src/FS.c          Filesystem implementation
include/FS.h      Public API and storage structures
include/         Support-library interfaces
lib/             Supplied Linux support libraries
test/            Google Test suite
CMakeLists.txt   Build and test configuration
```

## Core Concepts

- Mapping byte offsets to data blocks.
- Managing free space and inode allocation.
- Separating directory names from file identity.
- Tracking open-file positions.
- Reclaiming storage when the final link is removed.
