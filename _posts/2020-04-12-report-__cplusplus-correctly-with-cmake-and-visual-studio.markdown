---
layout: post
title: Report __cplusplus correctly with CMake and Visual Studio
date: '2020-04-12 11:05:30'
tags:
- cmake
- cpp
- visual-studio
redirect_from:
- /report-__cplusplus-correctly-with-cmake-and-visual-studio
---

Some C++ projects use the [`_cplusplus` predefined macro](https://docs.microsoft.com/en-us/cpp/preprocessor/predefined-macros) to determine which language features the compiler supports. However, by default, the Microsoft Visual C++ compiler still reports a very old value, even if you're explicitly using a modern C++ standard. This can occasionally cause some problems, such as libraries omitting certain API features.

Microsoft introduced the [`/Zc:__cplusplus` compiler option](https://docs.microsoft.com/en-us/cpp/build/reference/zc-cplusplus) to fix this in Visual Studio 2017 version 15.7 (also see [this blog post about it](https://devblogs.microsoft.com/cppblog/msvc-now-correctly-reports-__cplusplus/)). It's easy to enable this in your project properties if you're working directly with the Visual Studio IDE.

This post will explain how to enable this option if you're managing your build using CMake.

## Single target

To fix the `__cplusplus` macro for a single library or executable, add this to your CMake file:

```cmake
if(MSVC)
    target_compile_options(mytarget PUBLIC "/Zc:__cplusplus")
endif()
```

You should typically put it just after your target has been defined; i.e. `add_executable()` or `add_library()`. Replace `mytarget` with the name of the CMake target you want to affect.

Note that you will also need to ensure you've specified which version of the C++ standard you're using. For example, if you're using C++14:

```cmake
target_compile_features(mytarget PUBLIC cxx_std_14)
```

Again, replace `mytarget` with the name of your CMake target. You may already have specified the C++ standard somewhere else, in which case you don't need to do it again.

## Multiple targets

If you want to fix the macro for multiple targets at the same time then you can do this instead:

```cmake
if(MSVC)
    string(APPEND CMAKE_CXX_FLAGS " /Zc:__cplusplus")
endif()
```

This should affect any CMake target which is specified in the same CMake file or a sub-directory, or which includes the CMake file. You may find it helpful to put it in the root CMakeLists.txt file for your entire project.

As above, you will need to ensure the C++ standard has been specified. If that's not already been done elsewhere, you can specify C++14 for multiple targets like this:

```cmake
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

## Example project

To see the macro in action, create a folder for a simple test project. Within it, create two files: `main.cpp` and `CMakeLists.txt`.

Put this in `main.cpp`:

```cpp
#include <iostream>
int main()
{
    std::cout << " __cplusplus=" <<__ cplusplus << std::endl;
}
```

Put this in `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.1)
project(demo CXX)
add_executable(demo main.cpp)
target_compile_features(demo PUBLIC cxx_std_14)
if(MSVC)
    target_compile_options(demo PUBLIC "/Zc:__cplusplus")
endif()
```

Open a Windows Command Prompt and navigate to the project folder you just created. We're going to do an out-of-source build, so create and navigate into a sub-folder:

```console
mkdir build
cd build
```

From there, run these commands to build and run the example program:

```console
cmake ..
cmake --build .
Debug/demo.exe
```

After the last line, you should see this:

```
__cplusplus=201402
```

In `CMakeLists.txt`, try changing `cxx_std_14` to `cxx_std_17`, and the output should change accordingly when you build and run the project. You could also try commenting-out the `target_compile_options` line to see the default macro value.

## Conan package

If you're creating a library to be packaged by [Conan](https://conan.io/) then you may also want to specify the compiler option in your package info. This will ensure that any Conan project depending on your library automatically sets the `__cplusplus` macro correctly as well when building with MSVC.

Open your Conan recipe file (typically conanfile.py), and add the following to the `package_info()` method:

```python
if self.settings.compiler == "Visual Studio":
    self.cpp_info.cxxflags.append("/Zc:__cplusplus")
```

