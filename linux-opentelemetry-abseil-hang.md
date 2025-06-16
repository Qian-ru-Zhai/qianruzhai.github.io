---

## 🔧 Background

While building **OpenTelemetry** using **Conan**, everything compiled successfully. However, when running the **example application**, the process **hung at startup**—with no output or error messages.

---

## 🧪 Reproducing the Problem

- OpenTelemetry built using Conan.
- Abseil is pulled in as a Conan dependency.
- Running the example results in a **hang on startup**.

---

## 📟 Call Stack (via GDB)
```
Breakpoint 1, 0x00005555564c9116 in absl::lts_20240116::flags_internal::FlagRegistry::RegisterFlag(absl::lts_20240116::CommandLineFlag&, char const*) ()
(gdb) bt
#0  0x00005555564c9116 in absl::lts_20240116::flags_internal::FlagRegistry::RegisterFlag(absl::lts_20240116::CommandLineFlag&, char const*) ()
#1  0x00005555564ca037 in absl::lts_20240116::flags_internal::RegisterCommandLineFlag(absl::lts_20240116::CommandLineFlag&, char const*) ()
#2  0x000055555561b094 in absl::lts_20240116::flags_internal::FlagRegistrar<unsigned short, true>::FlagRegistrar(absl::lts_20240116::flags_internal::Flag<unsigned short>&, char const*) ()
#3  0x000055555561a77b in __static_initialization_and_destruction_0(int, int) ()
#4  0x000055555561a7b5 in _GLOBAL__sub_I_FLAGS_port ()
#5  0x0000555556573c2d in __libc_csu_init ()
#6  0x00007ffff7a7b010 in __libc_start_main (main=0x55555561a694 <main>, argc=1, argv=0x7fffffffe218, init=0x555556573be0 <__libc_csu_init>, fini=<optimized out>,
    rtld_fini=<optimized out>, stack_end=0x7fffffffe208) at ../csu/libc-start.c:264
#7  0x000055555561a3ae in _start ()
```
---
## 🔍 Root Cause

This was initially suspected to be a bug in the OpenTelemetry library, but after deeper investigation, the real issue was:

> **Abseil was compiled by Conan using `-O3` optimization, which caused static initialization of flags to behave incorrectly**, leading to a hang during startup.

---

## ✅ Solution

To fix the issue, Abseil should **not be compiled with `-O3`**. Instead, we can override Conan’s default release optimization flags by using a **custom toolchain file**.

---

## 🛠 Create a Custom Toolchain

Create a file called `mytoolchain.cmake`:

```cmake
set(CMAKE_CXX_FLAGS_RELEASE "-msse2")
set(CMAKE_C_FLAGS_RELEASE "-msse2")
```

## 🧪 Rebuild Abseil Using Conan + Toolchain
Use this Conan command to apply your custom toolchain:
```
conan install . -s arch=x86 --build=missing -c tools.cmake.cmaketoolchain:user_toolchain="['/full/path/to/mytoolchain.cmake']"
```
