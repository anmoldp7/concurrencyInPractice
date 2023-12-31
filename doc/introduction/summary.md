# INTRODUCTION

- Before OS, single program executed from beginning to end, this program had access to all resources on machine.
- OS evolved to allow more than one program to run at once, running individual programs in processes.
  - Process: Isolated, independently executing programs.
  - OS allocates resources like memory, file handles & security credentials to processes.
  - Processes can communicate with each other through a variety of coarse grained communication mechanisms:
    - Sockets
    - Signal Handlers
    - Shared Memory
    - Semaphores
    - Files
- Motivating factors behind OS & multiprocessing:
  - Resource Utilization:
    - Programs have to wait for external operations such as input or output, while waiting can do no useful work.
    - More efficient to let another program use the wait time for execution.
  - Fairness:
    - Multiple users and programs may have equal claims on machine's resources.
    - More preferable to let programs share the computer via fine grained time slicing than to let one program run to completion and then start another.
  - Convenience:
    - Easier or more desirable to write several programs that each perform a single task and have them coordinate with each other as necessary than to write a single program that performs all the tasks.
- Von Neumann Computer:
  - Comprises of Control Unit, Arithmetic & Logical Memory Unit, Registers, I/O.
  - Uses a single processor.
  - Uses one memory for both instructions and data.
  - Executes programs following fetch-decode-execute cycle.
  - Components of Von Neumann Model:
    - CPU
      - Performs bulk of data processing and responsible for executing instructions of computer program.
      - Components:
        - ALU:
          - Arithmetic & Logic Unit.
          - Performs required micro-operations for executing instructions.
          - Arithmetic (+, -, *, / etc) and Logic operations (AND, OR, NOT etc).
        - Control Unit:
          - Controls operations of components like ALU, memory, I/O devices.
          - Consists of program counter that contains address of instructions to be fetched and instruction register into which instructions are fetched from memory for execution.
        - Registers:
          - High speed storage areas in CPU.
          - Data processes by CPU are fetched from registers. Types of registers:
            - MAR (Memory Address Register): Holds memory location of data that needs to be accessed.
            - MDR (Memory Data Register): Holds data that is being transferred to or from memory.
            - AC (Accumulator): Holds intermediate arithmetic and logic results.
            - PC (Program Counter): Register contains address of next instruction that needs to be executed.
            - CIR (Current Instruction Register): Contains current instruction during processing.
    - Buses
      - Means by which information is shared between registers in multi-register configuration system.
      - Consists of set of common lines, one for each bit of register, through which binary information is transferred one at a time.
      - Control signals determine which register is selected by bus during each particular register transfer.
      - Comprises of 3 major bus systems for data transfer:
        - Address Bus: Carries address of data between processor and memory.
        - Data Bus: Carries data between processor, memory unit and I/O devices.
        - Control Bus: Carries signals/commands from CPU.
    - Memory Unit
      - Collection of storage cells together with associated circuits needed to transfer information in and out of storage.
      - Stores binary information in groups of bits called words.
      - Internal structure of memory unit is specified by number of words it contains and number of bits in each word.
        - RAM
        - ROM
- In early timesharing systems, each process was a virtual Von Neumann computer:
  - It had memory space storing both instructions and data.
  - It executed instructions sequentially according to semantics of machine language.
  - It interacted with outside world via OS through a set of I/O primitives.
  - For each instruction executed, there was clearly defined "next instruction".
- Finding the right balance of sequentiality and asynchrony is characteristic of efficient people and programs.
  - This is the motivation behind development of threads.
  - Threads allow multiple streams of program control flow to coexist within process.
  - Threads share process wide resources such as memory and file handles.
  - However each thread has its own program counter, stack and local variables.
  - Threads also provide natural decomposition for exploiting hardware parallelism on multiprocessor systems.
  - Multiple threads of same program can be scheduled simultaneously on multiple CPUs.
  - In absence of explicit coordination, threads execute asynchronously and simultaneously wrt each other.
  - Threads share memory address space of owning process, thus they share:
    - Same variables
    - Allocated objects in heap of owning process. This allows finer grained data sharing than inter-process communication.
  - Without explicit synchronization, thread may modify variables that other thread is in middle of using, with unpredictable results.
- Benefits of threads:
  - Major benefits:
    - Exploiting multiple processors.
  - Minor benefits:
    - Reduce development and maintenance costs, improve performance of complex applications.
    - Threads make it easier to model how humans work and interact, by turning asynchronous workflows to mostly sequential ones.
    - Turn convoluted code to straight line code that is easier to write, read and maintain.
    - Useful in GUI applications for improving responsiveness of UI.
    - Useful in server applications for improving resource utilization and throughput.
    - Simplifies implementation of JVM, garbage collector runs on one or more dedicated threads.
  - Major benefits:
    - Exploiting multiple processors:
      - Multiprocessor trends will only accelerate as it gets harder to scale up clock rates. Manufacturers would put more processor cores on a single chip.
    - Simplicity of modelling:
      - Enforce separation of concerns.
      - Program that processes one type of task sequentially is easier to write, less error prone and easier to test than one managing multiple different types of task at once.
      - Assigning thread to each type of task or to each element in simulation affords illusion of sequentiality and insulates domain logic from details of scheduling, interleaved operations, asynchronous I/O and resource waits.
      - Complicated and asynchronous workflow can be decomposed into number of simpler, synchronous workflows each running in a separate thread, interacting with each other only at specific synchronization points.
      - This is exploited by frameworks such as servlets or RMI (remote method invocation).
      - Framework handles details of request management, thread creation, load balancing, dispatching portions of request handling to appropriate application component at appropriate point in workflow.
    - Simplified handling of asynchronous events:
      - Server application that accepts socket connections from multiple remote clients may be easier to develop when each connection is allocated its own thread and allowed to use synchronous I/O.
      - If an application goes to read from a socket when no data is available, read blocks until some data is available.
        - In single threaded application, this means that not only does processing the corresponding request stall, but processing of all requests stall while single thread is blocked.
        - To avoid this problem, single threaded server applications are forced to use non-blocking I/O, which is more complicated and error prone than synchronous I/O.
      - Historically, OS placed relatively low limits on number of threads that a process could create, as few as several hundred (or even less).
      - As a result, OS developed efficient facilities for multiplexed I/O, such as Unix *select* and *poll* system calls.
      - To access these facilities, Java class libraries acquired a set of packages (java.nio) for non-blocking I/O.
      - NPTL threads package was designed to support hundreds of thousands of threads.
    - More responsive user interfaces:
      - GUI applications used to be single threaded, which meant you had to either frequently poll throughout code for input events (messy or intrusive) or execute application code indirectly through a "main event loop".
      - If code called from main event loop takes too long to execute, UI appears to freeze until that code finishes because subsequent UI events can't be processed until control is returned to main event loop.
      - Modern GUI frameworks such as AWT and Swing toolkits replace main event loop with an event dispatch thread (EDT).
      - When a user interface event such as button press occurs, application defined event handlers are called in event thread.
      - If only short lived tasks execute in event thread, interface remains responsive since event thread is always able to process user actions reasonably quickly.
- Risks of threads:
  - Thread safety can be unexpectedly subtle because in absence of sufficient synchronization, ordering of operations on multiple threads is unpredictable and surprising (race condition).
  - In absence of synchronization, compiler, hardware & runtime are allowed to take significant liberties with timing and ordering of actions such as caching of variables and ordering of actions, such as caching variables in registers or processor local caches where they are temporarily (or permanently) invisible to other threads.
  - These tricks are in aid of better performance and are generally desirable but place burden on developer to clearly identify where data is being shared across threads so that these optimizations don't undermine safety.
- Liveness hazards of threads:
  - Safety means "nothing bad happens". Liveness concers complimentary goal that "something good eventually happens".
  - A liveness failure occurs when an activity goes in a state such that it is permanently unable to make forward progress.
  - One form of liveness failure that can occur in sequential programs is an inadvertent infinite loop.
  - Bugs causing liveness failure can be elusive because they depend on relative timing of events in different threads, thus they don't always manifest in development or testing.
- Performance hazards of threads:
  - In liveness, something good eventually happens. For performance, *eventually* won't be good enough.
  - Subsume broad range of problems: poor service time, responsiveness, throughput, resource consumption, scalability.
  - In well defined concurrent applications, threads lead to performance gain but still carry some runtime overhead.
  - Context switches, where scheduler suspends active thread temporarily so another thread can run are more frequent in applications with many threads and have significant costs: saving and restoring execution context,loss of locality, CPU time spent scheduling threads instead of executing them.
  - Even if your program never creates thread, frameworks may create threads on your behalf, code called from these threads have to be thread safe.
  - **Every java application** uses threads. When JVM starts, it creates threads for JVM housekeeping tasks (garbage collection, finalization) and a main thread for running main method.
  - AWT and Swing UI frameworks create threads for managing user interface events.
  - Timer creates threads for executing deferred tasks.
  - Component frameworks such as servlets and RMI create pools of threads and invoke component methods in these threads.
  - Frameworks introduce concurrency in applications by calling application components from framework threads. Components invariably access application state, thus requiring all code paths accessing that state be thread safe.
