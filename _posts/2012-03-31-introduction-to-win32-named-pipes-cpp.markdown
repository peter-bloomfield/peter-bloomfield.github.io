---
layout: post
title: Introduction to Win32 Named Pipes (C++)
date: '2012-03-31 22:01:52'
tags:
- cpp
- windows
redirect_from:
- /introduction-to-win32-named-pipes-cpp
- /introduction-to-win32-named-pipes-cpp/
- /introduction-to-win32-named-pips-cpp
- /introduction-to-win32-named-pips-cpp/
---

There are times when it’s extremely useful to be able to pass some data between different programs running on the same system. For example, you might have multiple programs forming part of the same package, and they need to share some important information or work together to process something.

There are several ways to do it, but my choice in a recent C++ project was to use named pipes in the Win32 API. Note that pipes on other operating systems are a little different so not all of this information is portable.

> You can find the [Win32 named pipe example programs on GitHub](https://github.com/peter-bloomfield/win32-named-pipes-example).
{: .prompt-tip }

# What are named pipes?

The “pipe” metaphor is fairly simple. Just like a literal pipe can carry water from one place to another, the metaphorical pipe carries data from one program to another. However, unlike most literal pipes around your home, the metaphorical pipes can support two-way flow.

In practical terms, the pipe is accessed very much like a file. Some of the behaviour is a little different though (it is more like a client-server architecture), and there are various other commands to be aware of. It is especially important to learn where things can go wrong, and what error codes to look out for.

As a side note, unnamed pipes (or anonymous pipes) also exist on various operating systems, but usually work in a different way and are used for slightly different purposes. They are beyond the scope of this post.

# A simple example

Here’s a quick overview of the steps required to create and use a simple named pipe to send data from a server program to a client program.

Server program:

1. Call [CreateNamedPipe(..)](https://docs.microsoft.com/en-gb/windows/win32/api/winbase/nf-winbase-createnamedpipea) to create an instance of a named pipe.
2. Call [ConnectNamedPipe(..)](https://docs.microsoft.com/en-gb/windows/win32/api/namedpipeapi/nf-namedpipeapi-connectnamedpipe) to wait for the client program to connect.
3. Call [WriteFile(..)](https://docs.microsoft.com/en-gb/windows/win32/api/fileapi/nf-fileapi-writefile) to send data down the pipe.
4. Call [CloseHandle(..)](https://docs.microsoft.com/en-gb/windows/win32/api/handleapi/nf-handleapi-closehandle) to disconnect and close the pipe instance.

Client program:

1. Call [CreateFile(..)](https://docs.microsoft.com/en-gb/windows/win32/api/fileapi/nf-fileapi-createfilea) to connect to the pipe.
2. Call [ReadFile(..)](https://docs.microsoft.com/en-gb/windows/win32/api/fileapi/nf-fileapi-readfile) to get data from the pipe.
3. Process or output the data.
4. Call [CloseHandle(..)](https://docs.microsoft.com/en-gb/windows/win32/api/handleapi/nf-handleapi-closehandle) to disconnect from the pipe.

I’ve included full source code for each program at the bottom of this article, and you can also [find them on Github](https://github.com/peter-bloomfield/win32-named-pipes-example). This is a very simple example though and there’s lots more you can do with pipes on Win32. Take a look at the MSDN article on [Named Pipe Operations](https://docs.microsoft.com/en-gb/windows/win32/ipc/named-pipe-operations) for more information on other useful functions.

# Pipe names

You can name Win32 pipes almost anything you like, but they must start with the prefix `\\.pipe\`. In practice, the prefix will usually be `\\\\.pipe\\` because you have to escape backslashes in C/C++ strings. Everything after that in the name is up to you, as long as you don’t use backslashes and don’t exceed 256 characters in total.

# Read/write modes

There are two main modes of read/write operation available on pipes: byte stream and message. The difference is fairly small, but can be significant depending on your application.

Message mode simply makes a distinction between each set of data sent down the pipe. If a program sends 50 bytes, then 100 bytes, then 40 bytes, the receiving program will receive it in these separate blocks. It will therefore need to read the pipe at least 3 times to receive everything.

On the other hand, byte stream mode lets all the sent data flow continuously. In our example of 50, 100, then 40 bytes, the client could receive everything in a single 190-byte chunk. Which mode you choose depends on what your programs need to do.

# Overlapped pipe IO

By default, pipe operations in Win32 are synchronous (aka blocking). That means your program (or specifically the thread which handles the pipe operations) will need to wait for each operation to complete before it can continue. This can seem frustrating, but it makes programming much simpler. When one of the pipe functions returns, it means you know it has either been successful or it has failed.

Using [overlapped pipe IO](https://docs.microsoft.com/en-gb/windows/win32/ipc/synchronous-and-overlapped-input-and-output) means that pipe operations can process in the background while your program continues to do other things, including running other pipe operations in some cases. This can be very helpful, but it means you have to keep track of which operations are in progress and monitor them for completion.

An alternative to overlapped operation is to run synchronous pipe operations in a separate thread. If your pipe IO needs are fairly simple then this may be a simpler option. However, make sure your thread can terminate cleanly when needed.

# Buffered input/output

When calling `CreateNamedPipe(..)`, you can choose to specify buffer sizes for outbound and inbound data. These can be very helpful for program performance, particularly in synchronous operation. If your buffer size is 0 (which is entirely valid) then every byte of data must be read from the other end of the pipe before the write operation can be completed.

However, if a buffer is specified then a certain amount of data can linger in the pipe before it gets read. This can allow the sending program to carry on with other tasks without needing to use overlapped pipe IO.

# The “Hello Pipe World” named pipe example

To see synchronous named pipes in action, have a look at the [Win32 named pipe example programs on GitHub](https://github.com/peter-bloomfield/win32-named-pipes-example). It includes project/solution files for Visual Studio 2015.

Alternatively, I’ve included the source code below if you just want to browse it. It should be possible to compile it using any version of Visual Studio. Simply add the code for each program to a Win32 console application, and make sure you are linking against the Windows libraries.

**Important note:** When running the programs, run the server first! The client program fails if the pipe is not available.

## Server program

```cpp
///// SERVER PROGRAM /////
#include <iostream>
#include <windows.h>
using namespace std;
int main(int argc, const char **argv)
{
    wcout << "Creating an instance of a named pipe..." << endl;
    // Create a pipe to send data
    HANDLE pipe = CreateNamedPipe(
        L"\\\\.\\pipe\\my_pipe", // name of the pipe
        PIPE_ACCESS_OUTBOUND, // 1-way pipe -- send only
        PIPE_TYPE_BYTE, // send data as a byte stream
        1, // only allow 1 instance of this pipe
        0, // no outbound buffer
        0, // no inbound buffer
        0, // use default wait time
        NULL // use default security attributes
    );
    if (pipe == NULL || pipe == INVALID_HANDLE_VALUE) {
        wcout << "Failed to create outbound pipe instance.";
        // look up error code here using GetLastError()
        system("pause");
        return 1;
    }
    wcout << "Waiting for a client to connect to the pipe..." << endl;
    // This call blocks until a client process connects to the pipe
    BOOL result = ConnectNamedPipe(pipe, NULL);
    if (!result) {
        wcout << "Failed to make connection on named pipe." << endl;
        // look up error code here using GetLastError()
        CloseHandle(pipe); // close the pipe
        system("pause");
        return 1;
    }
    wcout << "Sending data to pipe..." << endl;
    // This call blocks until a client process reads all the data
    const wchar_t *data = L" ***Hello Pipe World***";
    DWORD numBytesWritten = 0;
    result = WriteFile(
        pipe, // handle to our outbound pipe
        data, // data to send
        wcslen(data) * sizeof(wchar_t), // length of data to send (bytes)
        &numBytesWritten, // will store actual amount of data sent
        NULL // not using overlapped IO
    );
    if (result) {
        wcout << "Number of bytes sent: " << numBytesWritten << endl;
    } else {
        wcout << "Failed to send data." << endl;
        // look up error code here using GetLastError()
    }
    // Close the pipe (automatically disconnects client too)
    CloseHandle(pipe);
    wcout << "Done." << endl;
    system("pause");
    return 0;
}
```

## Client program

```cpp
///// CLIENT PROGRAM /////
#include <iostream>
#include <windows.h>
using namespace std;
int main(int argc, const char **argv)
{
    wcout << "Connecting to pipe..." << endl;
    // Open the named pipe
    // Most of these parameters aren't very relevant for pipes.
    HANDLE pipe = CreateFile(
        L"\\\\.\\pipe\\my_pipe",
        GENERIC_READ, // only need read access
        FILE_SHARE_READ | FILE_SHARE_WRITE,
        NULL,
        OPEN_EXISTING,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );
    if (pipe == INVALID_HANDLE_VALUE) {
        wcout << "Failed to connect to pipe." << endl;
        // look up error code here using GetLastError()
        system("pause");
        return 1;
    }
    wcout << "Reading data from pipe..." << endl;
    // The read operation will block until there is data to read
    wchar_t buffer[128];
    DWORD numBytesRead = 0;
    BOOL result = ReadFile(
        pipe,
        buffer, // the data from the pipe will be put here
        127 * sizeof(wchar_t), // number of bytes allocated
        &numBytesRead, // this will store number of bytes actually read
        NULL // not using overlapped IO
    );
    if (result) {
        buffer[numBytesRead / sizeof(wchar_t)] = '\0'; // null terminate the string
        wcout << "Number of bytes read: " << numBytesRead << endl;
        wcout << "Message: " << buffer << endl;
    } else {
        wcout << "Failed to read data from the pipe." << endl;
    }
    // Close our pipe handle
    CloseHandle(pipe);
    wcout << "Done." << endl;
    system("pause");
    return 0;
}
```