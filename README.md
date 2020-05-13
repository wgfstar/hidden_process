# hidden_process
所谓的进程隐藏，通俗地说指的是某个进程正常工作，不受任何影响，但是，我们使用类似任务管理器、Process Explorer 等进程查看软件查看进程，却看不到这个进程。适合秘密在计算机后台进行操作的程序，而不想让人发现。

本文讲解的就是实现这样的一个进程隐藏程序的原理和过程，当然，进程隐藏的方法有很多，例如 DLL 劫持、DLL注入、代码注入、进程内存替换、HOOK API 等等。本文要介绍的就是 HOOK API 函数 ZwQuerySystemInformation 实现的隐藏指定进程。
### 实现原理
首先，先来讲解下为什么 HOOK ZwQuerySystemInformation 函数就可以实现指定进程隐藏。是因为我们遍历进程通常是调用系统 WIN32 API 函数 EnumProcess 、CreateToolhelp32Snapshot 等函数来实现，这些 WIN32 API 它们内部最终是通过调用 ZwQuerySystemInformation 这个函数实现的获取进程列表信息。所以，我们只要 HOOK ZwQuerySystemInformation 函数，对它获取的进程列表信息进行修改，把有我们要隐藏的进程信息从中去掉，那么 ZwQuerySystemInformation 就返回了我们修改后的信息，其它程序获取这个被修的信息后，自然获取不到我们隐藏的进程，这样，指定进程就被隐藏起来了。

其中，我们将HOOK ZwQuerySystemInformation 函数的代码部分写在 DLL 工程中，原因是我们要实现的是隐藏指定进程，而不是单单在自己的进程内隐藏指定进程。写成 DLL 文件，可以方便我们将 DLL 文件注入到其它进程的空间，从而 HOOK 其它进程空间中的 ZwQuerySystemInformation 函数，这样，就实现了在其它进程空间中也看不到指定进程了。

我们选取 DLL 注入的方法是设置全局钩子，这样就可以快速简单地将指定 DLL 注入到所有的进程空间里了。

其中，HOOK API 使用的是自己写的 Inline Hook，即在 32 位程序下修改函数入口前 5 个字节，跳转到我们的新的替代函数；对于 64 位程序，修改函数入口前 12 字节，跳转到我们的新的替代函数。
