# A Kqueue-based event loop for asynchronous computations on macOS

This library provides an [async](https://github.com/stefan-hoeck/idris2-async) compatible, [Kqueue](https://en.wikipedia.org/wiki/Kqueue)-based event loop for asynchronous computations on macOS.

> [!NOTE]
> This library currently only provides macOS support due to incompatibilities in the [posix](https://github.com/stefan-hoeck/idris2-linux/tree/main/posix) library (dependency of the [async-posix](https://github.com/stefan-hoeck/idris2-async/tree/main/async-posix) library).
> You can see the errors in [this](https://github.com/Matthew-Mosior/idris2-async-kqueue/actions/runs/35418830057/job/105832538925) failed FreeBSD GitHub action job.

## Kqueue vs. epoll

Kqueue and [epoll](https://en.wikipedia.org/wiki/Epoll) are high-performance both kernel event notifications mechanisms designed to monitor thousands of file descriptors simultaneously in O(1) time complexity.
They both work on a simple premise, which is that you register your interest with the kernel once, and the kernel notifies you exactly which resources are ready.

### Key workflow differences

#### The epoll Workflow (Linux)

To manage events in Linux, your application must jump back and forth between user space and kernel space using specialized commands:
- `epoll_create()` -> Allocates an epoll instance inside the kernel.
- `epoll_ctl()` -> Modifies the interest list (`EPOLL_CTL_ADD`, `DEL`, `MOD`).
  - If you want to change multiple file descriptors, you must invoke this system call repeatedly.
- `epoll_wait()` -> Blocks your application thread until an event occurs, returning only the ready file descriptors.

#### The Kqueue Workflow (BSD/macOS)

kqueue is widely considered by developers to be the cleaner API because of its batching capability:
- `kqueue()` -> Allocates the event queue in the kernel.
- `kevent()` -> This single powerhouse system call handles everything else.
  - You pass it a changelist (an array of events you want to add, remove, or modify) and an empty eventlist. The kernel applies all of your modifications first, checks for ready events, and fills the eventlist before returning to user space.

### Summary

| Feature                 | epoll                                                                                           | Kqueue                                                                                                 |
| ----------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| System calls            | `epoll_create()`, `epoll_ctl()`, and `epoll_wait()`                                             | `kqueue()` and `kevent()`                                                                              |
| Batching modifications  | Modifying 10 sockets requires calling `epoll_ctl()` 10 separate times                           | Can register new events and fetch active events simultaneously in one `kevent()` call                  |
| Scope of events         | Designed strictly around file descriptors (sockets, pipes)                                      | Uses an abstraction layer ("filters") to watch sockets, filesystem changes, timers, and process states |
| Internal data structure | Uses a Red-Black Tree to organize monitored file descriptors and a linked list for ready events | Uses a hash table / array system mapped to specific event filters inside the kernel                    |

## API

The `kqueueApp` function is the main interface this library provides to users.
