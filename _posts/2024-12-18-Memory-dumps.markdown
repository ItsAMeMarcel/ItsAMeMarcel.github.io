---
layout: post
title:  "Creating Memory Dumps with gcore or gdb: A Practical Example"
date:   2025-01-18 17:22:15 +0100
categories: C++ Security
---

![Alt-Text](https://github.com/ItsAMeMarcel/blog-resources/blob/main/images/2024-12-18-Memory-dumps/title.png?raw=true)


Memory dumps are a crucial tool for analyzing the state of a program. With tools like **gcore** or **gdb**, developers can save the memory of a running process for debugging or to understand unexpected behavior. However, memory dumps can also expose sensitive information, such as passwords or cryptographic keys, if these are present in memory at the time of the dump. In this article, I’ll show you how to create memory dumps using gcore, analyze them, and highlight the importance of handling them securely.

# What is gcore?

`gcore` is a simple command-line tool that is part of the GNU Debugger (gdb). It allows you to create **core dumps**, which are snapshots of a process's memory, without terminating the process itself. These dumps contain:

- The state of the heap, stack, and other memory segments.
- Information about variables and data used by the program during runtime.

# Example: Creating a Dump with gcore

## **Step 1: Create a Test Program**

Let’s create a simple program that holds sensitive information, such as a password, in memory. The program will display its PID and remain active for a few minutes to allow analysis.

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <unistd.h> 
#include <chrono>

int main() {
    std::string password = "SuperSecret123";
    std::cout << "Program is running... PID: " << getpid() << std::endl;

    // Keep the program running
    std::this_thread::sleep_for(std::chrono::minutes(5));

    return 0;
}
```
Compile the program:

```bash
g++ -o memory_test memory_test.cpp
```
Run it:

```bash
./memory_test
```
The program will display the PID and remain active for 5 minutes.


## **Step 2: Create a Dump with gcore**

While the program is running, you can create a memory dump using `gcore`:

1. Find the process ID (PID) if you don’t already have it from the program’s output:

```bash
ps aux | grep memory_test
```
2. Create a dump with `gcore`:

```bash
gcore -o dumpfile <PID>
```
This generates a file named dumpfile.<PID>, which contains the entire memory content of the process.


## **Step 3: Analyze the Dump**

Use tools like strings to inspect the contents of the dump:

```bash
strings dumpfile.<PID> | grep "SuperSecret123"
```
If the password was not securely erased from memory, it will appear in the dump.


# Alternatively: Creating the dump with GDB

## **Step 1: Attach to a Running Process**

To analyze a specific process, you need to attach gdb to it using its process ID (PID). For example:

```bash
gdb -p <PID>
```
Replace <PID> with the actual process ID of the target process. Once attached, gdb will pause the process, allowing you to debug or inspect it.

## **Step 2: Focus on Specific Memory Regions**

Use the info proc mappings command to identify memory regions, such as the heap, stack, or code segments. For example:

```
(gdb) info proc mappings
```
This command displays memory regions with their start and end addresses, permissions, and associated paths. You can use this information to select the regions you want to analyze or dump.

## **Step 4: Combine Interactive Debugging with Memory Dumping**

After inspecting the memory, you can dump specific memory regions to a file. For example, to dump the heap region:

```
(gdb) dump memory heap_dump.bin 0xheap_start 0xheap_end
```
Replace 0xheap_start and 0xheap_end with the actual addresses of the heap region obtained from info proc mappings


# Why Does This Matter?

This example demonstrates how easily sensitive data like passwords or tokens can be extracted from memory if they are not securely cleared. In security-critical applications, developers should:

- Securely erase data immediately after use (see this [article](https://itsamemarcel.micro.blog/2024/12/06/securely-erasing-memory.html))
- Disable dumps when they’re not needed.

```bash
ulimit -c 0
```
# Conclusion

`gcore` and `gdb` are powerful tools for analyzing the state of a program, but it also highlights potential security risks if sensitive data remains in memory. By securely clearing data and disabling unnecessary dumps, developers can better protect their applications from data leaks.


# Further Resources

- [gcore Documentation](https://man7.org/linux/man-pages/man1/gcore.1.html)
- [GNU Debugger Documentation](https://linux.die.net/man/3/dbg)
- [Core Dumps on Linux (Linux Manual)](https://man7.org/linux/man-pages/man5/core.5.html)
