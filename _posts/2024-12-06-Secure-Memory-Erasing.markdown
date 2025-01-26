---
layout: post
title:  "Securely Erasing Memory in C/C++: Why It’s Important and How to Do It Right"
date:   2024-12-06 20:30:45 +0100
categories: C++ Security
---


<img src="https://raw.githubusercontent.com/ItsAMeMarcel/blog-resources/refs/heads/main/images/2024-12-06-Secure-Memory-Erasing/title.webp" width="600" height="600" alt="">

In software development, it is often necessary to securely erase sensitive data from memory as soon as it is no longer needed. This is especially true for passwords, tokens, or other confidential information in security-critical applications. While the concept sounds straightforward, there are some important nuances to consider. In this article, I’ll explain why a simple `memset` is not sufficient to securely erase memory and provide an overview of alternatives.

---

# The Problem: Why Erasing Isn’t Always Secure

A developer might assume that overwriting memory with a function like `memset` is enough to erase sensitive data. 
A naive implementation might look like this:

```cpp
void insecure_clean(void* ptr, size_t size) {
    memset(ptr, 0, size);
}
```
However, the issue lies in compiler optimizations. Modern compilers such as `gcc` or `clang` analyze code and may detect that the memory cleared by `memset` is no longer being used. Based on this analysis, the compiler could remove the `memset` operation entirely to optimize runtime performance.

---

# Attack Vectors

Sensitive data that is not properly erased can be compromised in several ways:

1. **Memory Dumps:** An attacker could create a core dump or analyze the memory using debugging tools like `gdb`.
2. **Unsafe Memory Deallocation:** If memory is freed without being cleared, other programs or threads may access the leftover data.

For example, a password that remains in memory after authentication could be discovered and exploited by an attacker.

---

# The Solution: How to Securely Erase Memory

There are various approaches to securely clear memory. These methods can be general-purpose, platform-specific, or depend on the compiler version. A comprehensive and robust solution is also offered by the external library **Libsodium**.

---

## 1. Using `volatile`

The `volatile` keyword prevents the compiler from optimizing out memory-clearing operations. By marking the memory pointer as `volatile`, the compiler is forced to perform each write operation, even if it considers them redundant. Here’s how it can be used:

```cpp
void secure_clean(volatile void* ptr, size_t size) {
    volatile unsigned char* p = (volatile unsigned char*)ptr;
    while (size--) {
        *p++ = 0; 
    }
}
```
---

## 2. Using Specialized Platform Functions

Many operating systems or standard libraries provide specialized functions for securely erasing memory. These functions use internal mechanisms to ensure that neither the compiler nor the hardware skips the clearing operation.

### Unix

The explicit_bzero function securely erases memory and guarantees that the operation will not be removed by the compiler.

```cpp
#include <string.h>

explicit_bzero(void *b, size_t len);
```
### Windows
The SecureZeroMemory function ensures that memory is securely cleared.

```cpp
#include <windows.h>

PVOID SecureZeroMemory(_In_ PVOID  ptr, _In_ SIZE_T cnt);
```
### C23
The new memset_explicit function can also be used to securely overwrite memory.

```cpp
#include <string.h>

memset_explicit( void *dest, int ch, size_t count );
```
---

## 3. Leveraging External Libraries

Libraries like **Libsodium** provide robust and platform-independent methods for secure memory erasure. These libraries are designed to securely remove sensitive data from memory while leveraging platform-specific optimizations.

```cpp
#include <sodium.h>

sodium_memzero(void * const pnt, const size_t len);
```
---

## 4. Additional Security Measures

In addition to securely erasing memory, consider these practices to further protect sensitive data:

- **Erase memory immediately:** Clear data as soon as it’s no longer needed, rather than keeping it in memory longer than necessary.
- **Avoid unnecessary copies:** Minimize copies of sensitive data to reduce the number of locations that need to be cleared.
- **Prevent memory dumps:** Disable core dumps in security-critical applications. For example on Unix:
```bash
ulimit -c 0
```
---

# Best Practices

1. Prefer platform-specific or library-specific functions like `explicit_bzero` or `sodium_memzero`.
2. Use `volatile` if no specialized function is available.
3. Erase sensitive data immediately once it’s no longer needed.
4. Implement platform-specific mechanisms to protect memory from unauthorized access.

---

# Conclusion

Securely erasing memory is an often-overlooked but critical aspect of software development. Attackers can exploit leftover data in memory if it is not properly erased. Fortunately, there are a variety of proven methods to address this issue. By combining secure memory-clearing functions with best practices, you can ensure that sensitive data is reliably removed from memory.

---

# Further Resources

- **[Libsodium Documentation](https://libsodium.gitbook.io/doc/)**
- **[explicit_bzero(3) - Linux Manual Page](https://man7.org/linux/man-pages/man3/explicit_bzero.3.html)**
- **[SecureZeroMemory (Microsoft Docs)](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/aa366877(v=vs.85))**
- **[memset_explicit Documentation](https://en.cppreference.com/w/c/string/byte/memset)**