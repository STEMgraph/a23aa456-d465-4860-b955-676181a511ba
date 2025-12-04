<!---
{
"id": "a23aa456-d465-4860-b955-676181a511ba",
  "depends_on": ["2ad3a473-2965-40ad-ab86-891dcfeed508"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-12-04",
  "keywords": ["C++", "Compiled library", "CMake", "Static linking", "Project structure"]
}
--->

# Linking a Self-Written Static Library (`.cpp` + `.h`) Using CMake

> In this exercise you will learn how to create and link a self-written compiled C++ library consisting of a `.h` and `.cpp` file. Furthermore we will explore how CMake defines build targets for such libraries and how to integrate them cleanly into your project structure.

## Introduction

Unlike header-only libraries, which are fully defined in headers and included directly, **compiled libraries** split their interface and implementation into a header (`.h`) and a corresponding source (`.cpp`) file. This separation improves compilation times, limits symbol exposure, and keeps object code reusable. In CMake, such a library is declared using `add_library()` with its `.cpp` sources and optionally linked into executable targets using `target_link_libraries()`.

In this exercise, you’ll extend your project by adding a statically compiled library. You'll structure the code cleanly, define the library target, and then link it into your main application. This process introduces more advanced CMake concepts, such as visibility (`PUBLIC`, `PRIVATE`) and linking targets properly. Understanding these fundamentals is essential for building scalable software and for integrating third-party C++ libraries later on.

By writing both the header and implementation yourself, you'll practice the full lifecycle of static library integration within a modern CMake project.

### Further Readings and Other Sources

* CMake `add_library()`: [https://cmake.org/cmake/help/latest/command/add_library.html](https://cmake.org/cmake/help/latest/command/add_library.html)
* Modern CMake Patterns: [https://cliutils.gitlab.io/modern-cmake/](https://cliutils.gitlab.io/modern-cmake/)
* YouTube: *CMake Linking Explained* by The Codeholic
* ISO C++: [https://isocpp.org/wiki/faq/coding-standards](https://isocpp.org/wiki/faq/coding-standards)

---

## Tasks

### 1. Set up your directory structure

Create the following layout:

```bash
mkdir cpp_staticlib
cd cpp_staticlib
mkdir -p src libs/arithlib
```

* `src/`: main program
* `libs/arithlib/`: the compiled library
* `build/`: separate build directory

### 2. Write the header file

Create `libs/arithlib/arith.h`:

```cpp
#pragma once

namespace arith {
    int add(int a, int b);
    int subtract(int a, int b);
}
```

### 3. Write the implementation file

Create `libs/arithlib/arith.cpp`:

```cpp
#include "arith.h"

namespace arith {
    int add(int a, int b) {
        return a + b;
    }

    int subtract(int a, int b) {
        return a - b;
    }
}
```

### 4. Write the main program

Create `src/main.cpp`:

```cpp
#include <iostream>
#include "../libs/arithlib/arith.h"

int main() {
    std::cout << "add(4, 2) = " << arith::add(4, 2) << std::endl;
    std::cout << "subtract(4, 2) = " << arith::subtract(4, 2) << std::endl;
    return 0;
}
```

### 5. Create the root `CMakeLists.txt`

In the project root, add the following:

```cmake
cmake_minimum_required(VERSION 3.10)
project(StaticLibDemo VERSION 1.0 LANGUAGES CXX)

add_library(arithlib
    libs/arithlib/arith.cpp
)
target_include_directories(arithlib PUBLIC libs/arithlib)

add_executable(statlibapp src/main.cpp)
target_link_libraries(statlibapp PRIVATE arithlib)
```

Explanation:

* `add_library()` creates a static library from your `.cpp` file
* `target_include_directories()` makes the headers available to dependents
* `target_link_libraries()` links the library into the main executable

### 6. Build and run

Configure and build the project:

```bash
cmake -B build
time cmake --build build
```

**Note:** Pay attention to the build time displayed by the `time` command - you'll compare this with the rebuild time in the next step.

Run the program:

```bash
./build/statlibapp
```

Expected output:

```
add(4, 2) = 6
subtract(4, 2) = 2
```

### 7. Demonstrate faster build times with library separation

Now let's see how the separation of library and main code improves build performance. Make a change to `src/main.cpp`:

```cpp
#include <iostream>
#include "../libs/arithlib/arith.h"

int main() {
    std::cout << "Testing arithmetic operations:" << std::endl;
    std::cout << "add(4, 2) = " << arith::add(4, 2) << std::endl;
    std::cout << "subtract(4, 2) = " << arith::subtract(4, 2) << std::endl;
    std::cout << "add(10, 5) = " << arith::add(10, 5) << std::endl;
    std::cout << "subtract(10, 5) = " << arith::subtract(10, 5) << std::endl;
    return 0;
}
```

Rebuild and time the process:

```bash
time cmake --build build
```

You'll notice that:
- Only the `main.cpp` file gets recompiled
- The `arithlib` library is **not** recompiled since it hasn't changed
- Build time is significantly faster than the initial full build

This demonstrates the key benefit of library separation: **incremental compilation**. When you modify only the main application code, the pre-compiled library objects are reused, leading to much faster build times in larger projects.

---

## Questions

**1. Why do we use both a `.h` and `.cpp` file instead of just defining everything in the header?**

<details>
<summary>Click to reveal answer</summary>

Separating interface from implementation reduces compile-time dependencies and hides implementation details, which is better for modularity and binary reuse.

</details>

**2. What is the role of `target_include_directories()` here?**

<details>
<summary>Click to reveal answer</summary>

It tells CMake where to find the headers for the `arithlib` library so that dependent targets like `statlibapp` can compile successfully.

</details>

**3. What happens if you remove `target_link_libraries(statlibapp PRIVATE arithlib)`?**

<details>
<summary>Click to reveal answer</summary>

The main program will compile but fail to link, because it references symbols (functions) that were defined in `arith.cpp` but never linked in.

</details>

---

## Advice

Working with compiled libraries is a core part of effective C++ software development. This exercise demonstrates the correct way to structure and link such libraries using CMake. As projects grow, breaking them into libraries makes them easier to test, reuse, and maintain. Prefer explicit linking with `target_link_libraries()` and keep your include paths modular using `target_include_directories()`. In larger codebases, you’ll often move these library declarations into their own `CMakeLists.txt` files for better scalability. If you understand how to separate implementation and interface cleanly now, you'll be well prepared for upcoming exercises that involve dependency injection, testing, and packaging.
