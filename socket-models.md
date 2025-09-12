Event Loops:
- A single-threaded loop (or a small number of threads) waits for events or messages to arrive.
- When an asynchronous I/O operation (like a network request or file read) is needed, the request is offloaded to a non-blocking API (often managed by the OS or a library like libuv).
- The event loop immediately becomes free to handle the next request.
- The event loop later picks up the callback and executes it, continuing the request's processing.
