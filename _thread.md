# Blocking vs Non‑blocking I/O (Sockets)

This explains the difference between blocking and non‑blocking I/O at the socket level and how that affects threads. Blocking describes what happens to a thread when it calls an operation that cannot complete immediately (usually I/O).

Points of confusion
- Blocking means: the thread is suspended until the read or write completes.
- Thread ≠ connection: a single thread can handle many connections, and a single connection can be handled by different threads over time.
- Blocking vs non‑blocking is a property of the I/O/socket; the thread experiences the effect.

**Blocking**
- Behavior: The thread calls an I/O operation and pauses until data is ready or the operation finishes. The OS parks the thread and wakes it when the result is available.
- Pros: Simple mental model and code; good for one‑off or low‑concurrency tasks.
- Cons: One thread per in‑flight I/O; poor scalability when handling many connections; lots of idle threads consuming memory and context‑switch time.
- Analogy: One barista takes an order and waits by the machine until the drink is done; no other customers are served in the meantime.

**Non‑blocking**
- Behavior: The thread issues an I/O request and continues running. Completion is handled via readiness events, callbacks, futures/promises, or an event loop (e.g., epoll/kqueue/IOCP).
- Pros: High throughput with few threads; great for many concurrent connections; less context switching.
- Cons: More complex control flow; requires careful state management and error handling.
- Analogy: The barista gives a number, keeps serving others, and returns the drink when the machine signals it’s ready.

**Clarifications**
- Thread vs connection: They are different concepts. A process can use a thread‑per‑connection model (simple, blocking) or an event‑driven model (non‑blocking) where one thread manages many sockets.
- Are blocking sockets still used? Yes. They’re fine for simple tools, scripts, or services with limited concurrency. High‑scale servers generally prefer non‑blocking or async I/O.
- Hybrid approaches: Many systems mix models. For example, use non‑blocking sockets to multiplex connections, but perform certain stages (e.g., TLS handshake or CPU‑heavy work) in a thread pool.

**When to use which**
- Use blocking if simplicity is the priority and concurrency is small.
- Use non‑blocking/async for servers that must handle many simultaneous connections or maintain responsive UIs/event loops.

**Key takeaways**
- Blocking/non‑blocking is about how I/O operations interact with threads.
- Blocking is easier but scales poorly with many concurrent I/O operations.
- Non‑blocking scales better by keeping threads busy and reacting to readiness.
Therefore, the thread is the entity that gets blocked, and the socket is the entity that causes the blocking behavior when the thread makes an I/O call on it.
Java sockets I/O: blocking, non-blocking and asynchronous
Aug 26, 2020 — The POSIX definitions. In this article are used simplified definitions from the POSIX specification. Blocked thread - a thread that is waiting for some conditio...

Favicon
LinkedIn
How does non-blocking IO work under the hood? - Medium
Nov 20, 2019 — Calling an API that requests data from IO will cause the running thread to “block”, i.e. it is waiting until the requested data has returned to the caller. When...

Favicon
Medium
How socket mode affects thread behavior
Blocking Sockets
When a thread attempts to read from an empty blocking socket, the operating system suspends the thread's execution until data arrives.
The thread, and therefore your program's execution, is blocked. The thread is temporarily taken off the CPU, allowing other threads to run, but from the programmer's perspective, that thread is "stuck".
This is the simple, traditional model where a dedicated thread handles one connection at a time. The thread will spend much of its time idle, but waiting is handled transparently by the operating system.
Non-blocking Sockets
When a thread attempts to read from an empty non-blocking socket, the system call returns immediately with an error code (like EWOULDBLOCK or EAGAIN), instead of suspending the thread.
The thread is not blocked and can immediately move on to do other work.
The thread must then check repeatedly or wait for an event notification from the operating system to know when data is ready on the socket.
Why the thread perspective is relevant
By using the "blocking vs. non-blocking thread" phrasing, the explanation focuses on the most visible outcome from a developer's point of view: how your code's execution flow is impacted.
For example, a traditional multi-threaded server model uses blocking sockets and relies on the operating system to automatically manage the blocking threads. A modern non-blocking server (like Node.js) uses a single thread and non-blocking sockets, explicitly managing when to service connections using an event loop.
So, while the socket is the technical object with the "blocking" or "non-blocking" property, it is your thread's behavior that is fundamentally changed by that property. Both terms are correct depending on the context, but describing it from the thread's perspective often best explains the user-facing consequences.
AI responses may include mistakes. Learn more




undefined
undefined
undefined
15 sites
Windows Sockets: Blocking | Microsoft Learn
Aug 2, 2021 — A socket can be in "blocking mode" or "nonblocking mode." The functions of sockets in blocking (or synchronous) mode do not return until they can complete their...
favicon
Microsoft Learn
Leveraging Concurrency with Non-Blocking Sockets in Python | by ...
Apr 11, 2023 — In this article, we'll examine how Python leverages non-blocking sockets to enable concurrency on I/O tasks. * Types of Sockets. As mentioned earlier, there are...
favicon
Medium

Asynchronous Programming. Blocking I/O and non-blocking I/O
Apr 26, 2020 — So when we call the recv method, it will return to the main thread. The main mechanical difference is that send , recv , connect and accept can return without d...
favicon
luminousmen

Show all
I'm still confused.
A socket is: port + protocol + IP addres
A connection is: Connection = tuple (source IP, source port, destination IPm destination port, protocol)

What is a thread?
In computer programming, a thread is the smallest unit of execution that can be scheduled by an operating system. Multiple threads can exist within a single process and share that process's resources, like memory and open files. When you write a program, you define the actions that will be carried out within a thread.

To help clear up the confusion, let's establish the relationship between a process, a thread, a socket, and a connection.
The programming hierarchy
Process: A running instance of a program. An application can consist of one or more processes. A process has its own isolated memory space. Think of a process like a separate application you have open, such as your web browser.
Thread: The sequence of execution that performs work within a process. Every process has at least one main thread, but it can create multiple additional threads to handle different tasks simultaneously. A key difference is that all threads within the same process share the same memory, which allows them to communicate much more efficiently than separate processes.
Socket: An endpoint for communication. Sockets are used by a process to send and receive data across a network.
Connection: The established, logical pathway for communication between two specific sockets—one on a client and one on a server. A connection is defined by a tuple of network addresses (source IP, source port, destination IP, destination port, protocol).


How they work together
A process, through its threads, uses sockets to establish connections and communicate. The difference between blocking and non-blocking sockets is what determines how a thread behaves when an I/O operation is in progress.
Scenario: A simple web server
A web server needs to listen for incoming connections on a specific socket.
Start the process: You start the web server program, which becomes a process on the computer. This process contains at least one main thread.
Open a socket: The main thread opens a server socket on a specific port, say port 80, and calls the listen() function.
Handle connections with blocking I/O: The main thread (or a dedicated "listener" thread) enters a loop and calls the accept() function on the server socket.
accept() is a blocking call. The thread's execution is suspended and waits until a new client connection arrives.
When a connection arrives, accept() returns, providing a new socket dedicated to that connection.
To handle multiple clients, the server needs to create a new thread for each connection. This new thread then handles the client's request (e.g., reading a web page) through its dedicated socket. The original thread is freed up to go back and wait for the next connection.
Handle connections with non-blocking I/O: The main thread (or a listener thread) calls accept() on a non-blocking socket.
If there are no new connections, the accept() call returns immediately with an error indicating that it would block (EWOULDBLOCK). The thread is not suspended and can go on to do other things.
The thread can perform work for other clients or check for data on other sockets, all within the same thread. This is often done with a central "event loop" that monitors many non-blocking sockets at once.
