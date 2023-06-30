# THREAD SAFETY

- Writing thread safe code is basically about managing access to state, particularly shared mutable state.
- Making object thread safe will depend on whether it will be accessed by multiple threads.
  - Making object thread safe requires using synchronization to coordinate access to its mutable state.
- Primary method of synchronization in Java is using *synchronized* keyword which provides exclusive locking.
  - Synchronization also involves using *volatile* variables, explicit locks and atomic variables.
- Program that omits needed synchronization might appear to work, passing tests and performing well for years, but it's still broken and may fail at any moment.
- If multiple threads access the same mutable state variable without appropriate synchronization, it is broken. 3 ways to fix it:
  - Don't share state variable across threads.
  - Make the state variable immutable.
  - Use synchronization whenever accessing state variable.
- **It is far easier to design a class to be thread safe instead of retro-fitting it for thread safety later.**
- OOP which helps write maintainable, well organized classes using data hiding and encapsulation can also help create thread safe classes.
- The less code that has access to particular variable, easier it is to ensure all of it uses proper synchronization, easier it is to reason about conditions under which a given variable might be accessed.
- The better encapsulated the program state, the easier it is to make the program thread safe and to help maintainers keep it that way.
- When designing thread safe classes, good OOP techniques like encapsulation, immutability and clear specification of invariants are indispensable.
- Tradeoffs with concurrent code:
  - Compromising design guidelines for performance improvements or backward compatibility with legacy code.
  - Abstraction and encapsulation vs performance.
- General guideline: Make your code right, then make it fast.
  - Even then, pursue optimization only if performance measurements and requirements make it necessary.
- If encapsulation is compromised, making program thread safe is harder.
  - Thread safety is more fragile without encapsulation.
  - Increases development cost.
  - Increases maintenance cost and risk.
- Thread safe program might not be entirely comprised of thread safe classes.
- Correctness means a class conforms to its specification.
- Class is considered thread safe when **it continues to behave correctly when accessed from multiple threads, regardless of scheduling or interleaving of execution of those threads by runtime environment and with no additional synchronization or other coordination on part of calling code.**
- Any incorrect single threaded program is not thread safe by extension.
- **No set of operations performed sequentially or concurrently on instances of a thread safe class can cause an instance to be in an invalid state.**
- **Thread safe classes encapsulate any needed synchronization so that clients need not provide their own.**

## Stateless Servlet

```
@ThreadSafe
public class StatelessFactorizer implements Servlet {
  public void service(ServletRequest req, ServletResponse res) {
    BigInteger i = extractFromRequest(req);
    BigInteger[] factors = factor(i);
    encodeIntoResponse(resp, factors);
  }
}
```
- This is like most servlets stateless.
- Transient state for particular computation exists solely in local variables that are stored on thread stack and are accessed only when thread is executed.
- Actions of thread accessing stateless object can't affect correctness of operations in other thread.
- **Stateless objects are always thread safe.**

# Atomicity

Servlet with state (not thread safe, susceptible to lost updates):

```
@NotThreadSafe
public class UnsafeCountingFactorizer implements Servlet {
  private long count = 0;
  
  public long getCount() { return count; }
  
  public void service(ServletRequest req, ServletResponse resp) {
    BigInteger i = extractFromRequest(req);
    BigInteger[] factors = factor(i);
    ++count;
    encodeIntoResponse(resp, factors); 
  }
}
```

- **Race Condition:** Occurs when correctness of computation depends on relative timing or interleaving of multiple threads by runtime (getting to right answer relies on lucky timing).
  - Most common type of race condition: check-then-act.
  - Example of check-then-act: Lazy initialization.
  - Race conditions don't always result in failure, some unlucky timing is also required.

- **Atomicity:** Operations A & B are atomic wrt each other if, from perspective of thread executing A, when another thread executes B, either all of B has executed or none of it has. Atomic operation is one that is atomic wrt all operations, including itself, that operate on same state.
  - Check-then-act and read-modify-write operations are collectively referred to as compound actions: sequences of operations that must be executed atomically to remain thread safe.

```
@ThreadSafe
public class CountingFactorizer implements Servlet {
  private final AtomicLong count = new AtomicLong(0);

  public long getCount() { return count.get(); }

  public void service(ServletRequest req, ServletResponse resp) {
    BigInteger i = extractFromRequest(req);
    BigInteger[] factors = factor(i);
    count.incrementAndGet();
    encodeIntoResponse(resp, factors);
  }
}
```
  - *java.util.concurrent.atomic* package contains atomic variable classes for effecting atomic state transitions on numbers and object refernces. Used to create thread safe objects.

- **Locking:**

```
/*
 * Servlet that attempts to cache last result without adequate atomicity. Not thread safe.
 */

@NotThreadSafe
public class UnsafeCachingFactorizer implements Servlet {
  private final AtomicReference<BigInteger> lastNumber
    = new AtomicReference<BigInteger>();
  private final AtomicReference<BigInteger[]> lastFactors
    = new AtomicReference<BigInteger[]>();

  public void service(ServletRequest req, ServletResponse resp) {
    BigInteger i = extractFromRequest(req);
    if(i.equals(lastNumber.get())) {
      encodeIntoResponse(resp, lastFactors.get());
    } else {
      BigInteger[] factors = factor(i);
      lastNumber.set(i);
      lastFactors.set(factors);
      encodeIntoResponse(resp, factors);
    }
  }
}
```

  - Example of incorrect operation, lastNumber has been set bug lastFactors has not been set for another response.
  - When multiple variables participate in an invariant, they are not independent, value of one constains the allowed values of others.
  - To preserve consistency, update related variables in a single atomic operation.
  - **Intrinsic Locks**:
    - Has 2 parts:
      - Reference to object that will serve as lock.
      - Block of code guarded by lock.
    - Synchronized method is shorthand for synchronized block that spans entire method body and whose lock is object on which method is being invoked.
    - Every java object can act as lock for purposes of synchronization, these built-in locks are called intrinsic locks or monitor locks.
    - Lock is automatically acquired by executing thread before entering *synchronized* block and automatically released when control exits *synchronized* block whether by normal control path or by throwing exception out of block.
    - Intrinsic locks act as mutexes, at most one thread may own the lock.
      - If a thread never releases a mutex, another thread must wait forever to acquire it.
      - No thread executing a synchronized block can observe another thread to be in middle of synchronized block guarded by same lock.

```
/*
 * Servlet that caches last result but with unacceptable responsiveness, concurrency and performance.
 * Fairly extreme, as it inhibits multiple clients from using servlet simultaneously.
 */
@ThreadSafe
public class SynchronizedFactorizer implements Servlet {
  @GuardedBy("this") private BigInteger lastNumber;
  @GuardedBy("this") private BigInteger[] lastFactors;

  public synchronized void service(ServletRequest req, ServletResponse resp) {
    BigInteger i = extractFromRequest(req);
    if (i.equals(lastNumber)) {
      encodeIntoResponse(resp, lastFactors);
    } else {
      BigInteger[] factors = factor(i);
      lastNumber = i;
      lastFactors = factors;
      encodeIntoResponse(resp, factors);
    }
  }
}
``` 
  - **Reentrancy:**
    - Intrinsic locks are reentrant, if a thread tries to acquire a lock it already holds, request succeeds.
    - Reentrancy means that locks are acquired on per-thread instead of per-invocation basis.
      - Different from default locking behavior for pthreads mutexes which are granted on per-invocation basis.
    - Implemented by associating with each lock an acquisition count and an owning thread.
      - When count is 0, lock is considered unheld.
      - When thread acquires unheld lock, JVM records owner and sets acquisition count to one.
      - When same thread acquires lock again, count is incremented, when thread relieves block, count is reduced.
      - When count reaches 0, lock is released.
    - Reentrancy facilitates encapsulation of locking behavior and simplifies development of objec oriented concurrent code.
    - Without reentrancy, a call from synchronized method to another synchronized method would result in deadlock.
  - **Guarding State With Locks:**
    - Constructing protocols for guaranteeing exclusive access to shared state.
      - Object Serialization: Turn object into byte stream.
      - Serialization: Taking turns to access exclusively, rather than doing so concurrently.
    - Holding lock for entire duration of compound action can make it *atomic*.
    - If synchronization is used to coordinate access to variable, it is needed everywhere that variable is accessed.
    - When using locks to access variable, same lock has to be acquired. This way, variable is *guarded* by a lock.
    - **Common misconception: Synchronization required only when writing to shared variables.**
    - The fact that every object has a built-in lock is just a convenience so that needn't explicitly create lock objects.
      - Bad design, it can be confusing and forces JVM implementors to make tradeoffs between object size and performance.
    - Every shared, mutable variable should be guarded by exactly one lock. Make it clear to maintainers which lock that is.
    - Common locking convention is to encapsulate all mutable state within an object and protect it from concurrent access by synchronizing any code path that accesses mutable state using object's intrinsic lock.
      - Used by many thread safe classes such as vector and other synchronized collection classes.
      - All variables in object's state guarded by object's intrinsic lock.
    - Not all data needs to be guarded by locks: only mutable data that will be accessed by multiple threads.
      - Adding a simple asynchronous event can create thread safety requirements that ripple throughout program.
    - When variable is guarded by lock, only one thread can access it at a time.
    - When class has invariants that involve more than one state variable, there is an additional requirement: each variable participating in the invariant must be guarded by same lock.
    - Synchronizing every method in class can be indiscriminate, with too little or too much synchronization. May not render compound actions atomic.
    - Synchronizing every method can also lead to liveness or performance problems.

```
// this code has race condition even if contains & add are made atomic.
if (!vector.contains(element)) {
  vector.add(element);
}
```

  - **Liveness & Performance:**
    - Excessive & indiscriminate synchronization may lead to:
      - Compromise on performance & responsiveness.
      - Insufficient CPU utilization, having multiple CPUs available and still keeping most idle.
    - Poor concurrency: Number of simultaneous invocations is limited not by availability of processing resources but by structure of application itself.
    - This can be avoided by narrowing scope of *synchronized* block.
    - Acquiring and releasing a lock has some overhead too, so it is undesirable to make the scope too small without compromising atomicity.
    - Deciding how big or small *synchronized* blocks may require tradeoffs among competing design forces, including safety, simplicity & performance, however a reasonable balance can be found.
    - While using locks, be aware of what code in block is doing and how likely it is to take a long time to execute.
    - Holding lock for long time (either due to compute intensive task or blocking operation such as network, console I/O etc) introduces risk of liveliness or performance problems.
    - There is frequently tension between simplicity and performance. While implementing synchronization policy, resist the temptation to premature sacrifice simplicity for the sake of performance.

```
/*
 * Ideal design for factorizer.
 */
@ThreadSafe
public class CachedFactorizer implements Servlet {
  @GuardedBy("this") private BigInteger lastNumber;
  @GuardedBy("this") private BigInteger[] lastFactors;
  @GuardedBy("this") private long hits;
  @GuardedBy("this") private long cacheHits;

  public synchronized long getHits() { return hits; }
  public synchronized double getCacheHitRatio() {
    return (double) cacheHits / (double) hits;
  }

  public void service(ServletRequest req, ServletResponse resp) {
    BigInteger i = extractFromRequest(req);
    BigInteger[] factors = null;
    synchronized (this) {
      ++hits;
      if (i.equals(lastNumber)) {
        ++cacheHits
        factors = lastFactors.clone();
      }
    }
    if (factors == null) {
      factors = factor(i);
      synchronized (this) {
        lastNumber = i;
        lastFactors = factors.clone()
      }
    }
    encodeIntoResponse(resp, factors);
  }
}
```
