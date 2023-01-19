---
layout: post
title: Workaround for nvcc pthread issue
date: '2018-10-23 14:03:04'
tags:
- cmake
- cpp
redirect_from:
- /workaround-nvcc-pthread-issue
- /workaround-nvcc-pthread-issue/
---

I recently upgraded various pieces of software on my work PC. Afterwards, I found that our C++/CUDA projects wouldn’t build. The following error was reported:

```
nvcc fatal : Unknown option 'pthread'
```

I was attempting to build it in CLion 2018.2.5 using the bundled CMake 3.12.2. My OS is Ubuntu 18.04 and I’m using CUDA 9.1.

So far, I haven’t been able to find a proper solution. However, I have found a simple workaround. If you know of a better solution then please let me know in the comments.

## The problem

This issue seems to occur when using the FindThreads module in the CMake file. It adds a “-pthread” configuration flag which the CUDA compiler (nvcc) doesn’t understand.

I haven’t had time to dig into the underlying cause so I don’t know why this has started occurring relatively recently. However, it is a known problem and there have been some attempts to resolve it.

## The workaround

For now, downgrading CMake seems to avoid the problem. I downgraded to CMake 3.10.2, although more recent versions may also work.

If you’re using Ubuntu 18.04, then you can install CMake using apt. This currently installs version 3.10.2:

```console
sudo apt install cmake
```

Other distributions/repositories may have other versions available. If it doubt, consult the [CMake website](https://cmake.org/).

If you build your projects from within CLion then you will need to instruct it to use the correct CMake instance. You can do this by going to “File &rarr; Settings &rarr; Build, Execution, Deployment &rarr; Toolchains”. For CMake, select “Custom CMake executable”, and then give it the path to your manually installed version of CMake. On Linux, this is likely to be **`/usr/bin/cmake`**. You may want to do this in a new toolchain so that you don’t affect other projects.

## See also

- [https://gitlab.kitware.com/cmake/cmake/issues/18008](https://gitlab.kitware.com/cmake/cmake/issues/18008)
- [https://gitlab.kitware.com/cmake/cmake/merge\_requests/2092](https://gitlab.kitware.com/cmake/cmake/merge_requests/2092)
- [https://github.com/ginkgo-project/ginkgo/issues/86](https://github.com/ginkgo-project/ginkgo/issues/86)
- [https://stackoverflow.com/questions/43911802/does-nvcc-support-pthread-option-internally](https://stackoverflow.com/questions/43911802/does-nvcc-support-pthread-option-internally)
- [https://devtalk.nvidia.com/default/topic/525352/nvcc-compiler-pthreads-linux](https://devtalk.nvidia.com/default/topic/525352/nvcc-compiler-pthreads-linux)
