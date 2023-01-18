---
layout: post
title: Percent-sign in preprocessor definition breaks CMake Visual Studio generator
date: '2020-01-07 18:47:00'
tags:
- cmake
- cpp
- visual-studio
redirect_from:
- /percent-sign-in-preprocessor-definition-breaks-cmake-visual-studio-generator
---

I recently found a minor but frustrating problem while working on a cross-platform C++ project. In my CMake configuration file, I was trying to declare a preprocessor definition containing a date format string. The build worked on macOS using apple-clang/Xcode but it failed on Windows using MSVC/Visual Studio.

This is the CMake command which caused the problem:

```cmake
target_compile_definitions(MyProject PRIVATE
    LOG_FILE_NAME="MyProject-%Y-%m-%d.log")
```

It resulted in error messages like this when building the program on Windows:

```
cl : Command line error D8038: invalid argument
    'LOG_FILE_NAME="MyProject-%Y-%m-%d.log"'
    [C:\Users\peter.bloomfield\source\repos\my-project\build\MyProject.vcxproj]
cl : Command line error D8040: error creating or communicating with child process
    [C:\Users\peter.bloomfield\source\repos\my-project\build\MyProject.vcxproj]
```

Through trial and error, I quickly determined that it was the percent-signs (%) which were causing the failure but it wasn't immediately clear why. It isn't a special character in CMake strings or the C/C++ preprocessor.

## The problem

As you may know, CMake doesn't build programs itself. It generates build files for use by the selected toolchain. In this case, it generated a Visual Studio project file called "MyProject.vcxproj". I opened the file in a text editor and everything _looked_ fine to me. I saw the expected preprocessor definition in XML like this:

```xml
<PreprocessorDefinitions>
    ...;LOG_FILE_NAME="MyProject-%Y-%m-%d.log";...
</PreprocessorDefinitions>
```

However, the project file is only used to generate build commands based on command line programs like `cl.exe`. In the Windows shell, the percent-sign can be a special character. For example, [a batch script variable](https://www.tutorialspoint.com/batch_script/batch_script_variables.htm)can be referenced like this: `%MY_VARIABLE%`.

Indeed, I found that [Microsoft's documentation for cl.exe](https://docs.microsoft.com/en-us/cpp/build/reference/d-preprocessor-definitions) warns about this issue when declaring preprocessor definitions:

> "When you define a preprocessing symbol at the command prompt, consider both compiler parsing rules and shell parsing rules. For example, to define a percent-sign preprocessing symbol (`%`) in your program, specify two percent-sign characters (`%%`) at the command prompt. If you specify only one, a parsing error is emitted."

To confirm that this was the problem, I boiled it down to the simplest possible example. I created an empty C program in a file called `main.c`:

```c
int main() { return 0; }
```

I then tried to compile it on in the Windows shell using MSVC (note that you typically need to run the relevant `vcvarsall.bat` script first to prepare the environment):

```console
cl main.c
```

That worked as expected. It compiled the program and produced an executable.

I tried it again with a simple preprocessor definition, and again it worked as expected:

```console
cl /Dfoo=bar main.c
```

However, I then tried adding a percent-sign into the preprocessor symbol:

```console
cl /Dfoo=bar% main.c
```

It failed with this error message:

```
cl : Command line error D8038 : invalid argument 'foo=bar%'
```

I doubled the percent-sign as suggested in the documentation, and this time the build worked:

```console
cl /Dfoo=bar%% main.c
```

For comparison, I tried the equivalent commands to build the software using clang:

```console
clang ./main.c

clang -Dfoo=bar% ./main.c

clang -Dfoo=bar%% ./main.c
```

All those commands worked on both macOS and Windows (I have clang installed on Windows too). It even worked from the Windows shell, which surprised me. This means it wasn't just the shell that was causing the problem. Part of the compiler was treating the percent-sign as a special character too.

Percent-signs could still cause problems for clang (or other compilers) from the Windows command line though. For example:

```console
set bar=hello world
clang -Dfoo=%bar% ./main.c
```

That will fail because `%bar%` gets replaced with `hello world` before the command is invoked. The compiler sees `world` as a positional argument and thinks you are specifying an object file to be linked.

## The workaround

If you're only compiling for Windows then you can simply double the percent-signs in your preprocessor definitions. Nothing else needs to be done. For example, changing my CMake command to this worked on Windows:

```cmake
target_compile_definitions(MyProject PRIVATE
    LOG_FILE_NAME="MyProject-%%Y-%%m-%%d.log")
cmake
```

However, if you need to generate build files for other platforms too (or indeed other compilers on Windows) then it's not quite that simple. The double percentage sign will be taken literally by other compilers. In my case, that means the date format string would no longer work.

One approach to handle multiple platforms is to define the preprocessor definitions separately for each one, like this:

```cmake
if(WIN32)
    target_compile_definitions(MyProject PRIVATE
        LOG_FILE_NAME="MyProject-%%Y-%%m-%%d.log")
else()
    target_compile_definitions(MyProject PRIVATE
        LOG_FILE_NAME="MyProject-%Y-%m-%d.log")
endif()
```

That's simple and it's clear what it's doing, but I'm not keen on that solution because the two definitions could accidentally diverge. That could result in different behaviour on different platforms which could be confusing.

Instead, I opted to define the original format string in a CMake variable, and then automatically escape it on Windows before using it:

```cmake
set(LOG_FILE_NAME "MyProject-%Y-%m-%d.log")
if(WIN32)
    string(REPLACE "%" "%%" LOG_FILE_NAME ${LOG_FILE_NAME})
endif()
target_compile_definitions(MyProject PRIVATE
    LOG_FILE_NAME="${LOG_FILE_NAME}")
```

That has the downside of perhaps being a little less readable. That can hopefully be mitigated by adding suitable comments. It may be worth making a CMake function to escape the format string, especially if it will be done in multiple places.

Side note: Ideally, you should check which build toolchain is being used instead of just checking the platform. For simplicity, the workaround examples above assume that Visual Studio is the only toolchain being used when the target platform is Windows.

## A better solution?

As it stands, CMake's Visual Studio generator is creating project files which cannot be used. It therefore might be helpful for the generator to escape the percentage symbols automatically so that we don't need to workaround the problem in our build configuration. However, that may be more complex than it first appears. Perhaps there are more nuances than I'm aware of, such as variations in command line configuration.

