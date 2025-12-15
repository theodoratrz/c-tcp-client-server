# C Client-Server Application

A project demonstrating a **TCP client-server architecture in C** with
multithreading and socket communication.

This application illustrates how to handle multiple client connections and command
dispatch on the server side.

## Features

- **TCP communication** between client and server
- **Multithreaded server** using a thread pool
- Request parsing and command execution
- Dynamic buffer handling for variable-length responses

## Compile and Run
use `make` to compile all files. And `make clean` when finish to clean it.

Running the Server:
`./bin/Server [port] [buffersize] [threadpoolsize]`

Running the Client:
`./bin/Client [serverName] [port] [command]`

## Project Structure  
|--> bin (for executables)  
|--> build (for files created in runtime)  
|--> include (header files)  
|--> src (source files) 
makefile  
readme

## How it Works

The client connects to the server using TCP and sends commands.
The server accepts incoming connections and assigns each one to a
controller thread.

Controller threads parse incoming requests and enqueue jobs into a
shared buffer. Worker threads retrieve jobs from this buffer, execute
them using `fork()` and `exec()`, and send the results back to the client.

Synchronization between controller and worker threads is handled using
mutexes and condition variables to protect shared buffers and ensure
correct concurrent execution.

## Architecture Overview

### Client
The client establishes a TCP connection with the server, submits
commands for execution, and waits for responses.

### Server
The server is responsible for accepting client connections and
executing commands concurrently. It consists of three types of threads:

- **Main thread**
  - Listens for incoming connections using `accept()`
  - Spawns one controller thread per client
  - Initializes a pool of worker threads

- **Controller threads**
  - Read commands from client sockets
  - Enqueue execution requests into a shared buffer
  - Return immediate responses when applicable

- **Worker threads**
  - Dequeue jobs from the shared buffer
  - Execute commands using `fork()` and `exec()`
  - Capture command output and send it back to the client
 
## Implementation Details

Client–server communication is performed using `send()` and `read()`
on fixed-size buffers. When command output exceeds the buffer size,
the server first sends the size of the response, followed by the actual
data using a dynamically allocated buffer.

Synchronization is implemented using mutexes and condition variables.
Controller and worker threads coordinate through a shared job buffer:
- Workers block when the buffer is empty
- Controllers block when the buffer is full

## Data Structures (ADTs)

To preserve FIFO ordering while supporting job cancellation
(e.g. `stop <jobID>`), a custom queue-like data structure was implemented.

The `QueueList` supports:
- `push()` — enqueue job
- `pop()` — dequeue job
- `find()` — locate job by ID
- `removeValue()` — remove arbitrary job

A custom comparison function is used due to the complexity of the job
structure.
