---
layout: post
title: Automatically output the callstack on a breakpoint in Visual Studio
date: '2013-09-18 18:00:22'
tags:
- visual-studio
- cpp
- debugging
---

When you’re dealing with a large program and multiple developers, it’s not always obvious how and when certain things get executed. One of the very useful ways to debug unexpected behaviour is to set a [breakpoint](https://en.wikipedia.org/wiki/Breakpoint) on a suspect line of code and examine the [callstack](https://en.wikipedia.org/wiki/Call_stack) (or [stack trace](https://en.wikipedia.org/wiki/Stack_trace)) when it gets hit to see the execution path.

For infrequent events, it’s not always desirable to halt the entire program while you do that though. Instead, you can tell Visual Studio to write the callstack to the output when the breakpoint gets hit, and immediately continue execution.

## Supported versions of Visual Studio

I’m working with C++ in Visual Studio 2010 Professional. I believe the procedure is roughly the same on VS2012.

## Step-by-step

To set your breakpoint, do the following:

1. Create a normal breakpoint (if you don’t have one already)
2. Right-click the breakpoint marker
3. On the context menu, click “When Hit...”
4. Tick the checkbox labelled “Print a message”
5. In the text box below it, enter `$CALLSTACK`
6. Optionally add some other text to identify the breakpoint
7. Make sure “Continue execution” is ticked
8. Click OK

The “When Hit...” dialog should look like this:

![Screenshot of "When Hit..." dialog]( __GHOST_URL__ /content/images/2019/09/OutputCallstackOnBreakpoint.jpg)

Run your program in Debug mode. When the breakpoint is hit, it should output the callstack to the Output pane in Visual Studio. If the Output pane isn’t visible, you can usually enable it from the View menu. Below is an example of a call stack which was outputted using this technique:

    MessageServer.exe!Protocol::clear() 
    MessageServer.exe!Protocol::~Protocol() 
    MessageServer.exe!Interface::~Interface() 
    MessageServer.exe!Interface::`scalar deleting destructor'() 
    MessageServer.exe!std::_Destroy<Interface>() 
    MessageServer.exe!std::allocator<Interface>::destroy() 
    MessageServer.exe!std::_Dest_val<std::allocator<Interface>,Interface>() 
    MessageServer.exe!std::vector<Interface,std::allocator<Interface> >::pop_back() 
    MessageServer.exe!Connection::negotiateHandler() 
    MessageServer.exe!Connection::send() 
    MessageServer.exe!ConnectionManager::broadcastToAll() 
    MessageServer.exe!main() 
    MessageServer.exe!__tmainCRTStartup() 
    MessageServer.exe!mainCRTStartup() 
    kernel32.dll!7632ed5c() 
    [Frames below may be incorrect and/or missing, no symbols loaded for kernel32.dll]
    ntdll.dll!777137eb() 
    ntdll.dll!777137be()

## Advanced breakpoints

There’s a lot more you can do with breakpoints, including breaking after a certain number of hits, or when other conditions are met. For more information, check out this article on MSDN:

- [http://msdn.microsoft.com/en-us/library/vstudio/5557y8b4.aspx](http://msdn.microsoft.com/en-us/library/vstudio/5557y8b4.aspx)
<!--kg-card-end: markdown-->