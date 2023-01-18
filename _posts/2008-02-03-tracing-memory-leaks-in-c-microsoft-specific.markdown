---
layout: post
title: Tracing memory leaks in C++ [Microsoft-specific]
date: '2008-02-03 23:20:35'
tags:
- cpp
- visual-studio
redirect_from:
- /tracing-memory-leaks-in-c-microsoft-specific
---

While working on a personal project, I ran across the ability to track memory leaks down to the exact line of source code in Visual C++. This works with Visual Studio 2005 (I was using the Express Edition), although it may work in more recent versions too.

## Enable allocation tracking

The first thing you need to do is tell the compiler that you want to track memory allocations, and then include the standard headers and the CRT debugging headers. These 3 lines accomplish this:

```cpp
#define _CRTDBG_MAP_ALLOC
#include <stdlib.h>
#include <crtdbg.h>
```

To be really effective, those lines need to be included in _every_ file, which can be a bit of a pain. As such, it’s actually easier to create dedicated header file (I called mine “debug.h”) and do _not_ add it to your VS solution. Rather, just let it sit idly by in the root folder of your project, and use the project properties to force the pre-processor to _automatically_ include it in every file.

Go to Project Properties, select the “Debug” configuration, then go to “Configuration Properties” -\> “C/C++” -\> “Advanced”. In the “Force Includes” line on the right, add the relative path to the header files. My “debug.h” file sits in the folder where the VS \*.sln file is, so I used a handy built-in macro to define the path, like this:

```
$(SolutionDir)debug.h
```

## Output the leak info

You need to actually provide a call somewhere in your program to dump the memory leak data. Obviously, that should usually be done right at the end of the program. There are apparently ways of making it happen automatically on program termination, but I just manually called it right at the end of the `WinMain()` function. It’s done using a single function call, like this:

```cpp
_CrtDumpMemoryLeaks();
```

Rebuild and run your project in Debug mode. After it finishes, bring up the “debug output” console, and you’ll see a report of any memory leaks that occurred. It will include their size, location in memory, and the name and line of source code file where the allocation was originally made.

## Hang on! Something isn’t right!

Contrary to the documentation, the allocation calls are _not_ traced automatically to your allocations. In fact, they are all traced back to that “crtdbg.h” file where the allocation/de-allocation functions are customized for debugging. So what do we do now? Well the MSDN documentation offered some suggestions, so I tinkered for a while.

The good news is that I managed to get it to trace all the allocations correctly. The bad news is that it requires you to replace all your `new` commands with a macro.

Go back to that header file you created (“debug.h” in my case), and add the following lines:

```cpp
#ifdef _DEBUG
#define DBG_NEW new( _NORMAL_BLOCK, __FILE__ , __LINE__ )
#else
#define DBG_NEW new
#endif
```

Now for the annoying bit. Go and replace all your `new` calls with “NEW\_DBG”. When in debug mode, they will get replaced by this handy call which correctly identifies the file and line number. When in release mode, they simply get replaced by a regular “new” call.

### Why not just call the macro “new”?

Replacing a keyword with a macro is generally a bad idea. It’s likely to cause several problems and conflicts.

## Outputting to file

There are various means of outputting CRT stuff to different places, and a useful one is to a log file. (I always prefer to open up the memory log in a good text editor so I can search it.) It’s not too complicated, but it’s not exactly intuitive either.

The first thing you need to do is to create/open an appropriate output file. Do that using the following code right at the top of your main function (i.e. `WinMain(..)` or equivalent):

```cpp
HANDLE hCrtLog = CreateFile(TEXT("crt.log"), GENERIC_WRITE, FILE_SHARE_WRITE, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
```

That just creates a handle called `hCrtLog` then uses it to create/open a file called “crt.log”. Before you tell the CRT where to send error data, you need to check that the file was opened successfully. If it was successful, then redirect CRT warnings and errors to it. The following lines do all that:

```cpp
if (hCrtLog != INVALID_HANDLE_VALUE)
{
    _CrtSetReportMode( _CRT_WARN, _CRTDBG_MODE_FILE );
    _CrtSetReportMode( _CRT_ERROR, _CRTDBG_MODE_FILE );
    _CrtSetReportFile( _CRT_WARN, hCrtLog );
    _CrtSetReportFile( _CRT_ERROR, hCrtLog );
}
```

Finally, you should close the file at the end of your program. Put the following bit of code just **after** your call to `_CrtDumpMemoryLeaks()`:

```cpp
if (hCrtLog != INVALID_HANDLE_VALUE)
    CloseHandle(hCrtLog);
```

Compile and run your program in debug mode, and after you exit, you should see a “crt.log” file. (It might be alongside the executable file or alongside the VS project file). Open it up, and you’ll see all your memory leak data. If you want to test it, just put a `new` command somewhere in your program, and don’t delete the object afterwards. Check if the source file and line reported in the log file are correct.
