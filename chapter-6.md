### Chapter 6 - Concurrency

Chapter 6 tackles the complex topic of **concurrency**, which is about having your program do more than one thing at the same time. The authors compare it to juggling: it's powerful when done right, but it's very easy to drop the balls.

The main message is that traditional ways of handling concurrency (like using threads that share memory) are incredibly difficult to get right and are a common source of bugs. Pragmatic programmers should look for safer, simpler models.

Here are the key principles and ideas from the chapter:

#### 1. Concurrency vs. Parallelism
It's important to understand the difference:
*   **Concurrency** is about the *structure* of your program. It's about dealing with many tasks at once, even if you only have one processor core. For example, a web server handles multiple user requests concurrently, switching between them as needed.
*   **Parallelism** is about the *execution*. It's about doing many tasks at the exact same time, which requires multiple processor cores.

You can have concurrency without parallelism, but you can't have parallelism without a concurrent design.

#### 2. The Problem: Shared State is Dangerous
The biggest danger in concurrent programming is **shared, mutable state**. This means having multiple threads or processes that can all read and write to the same piece of memory (like a variable or an object).

This leads to a classic bug called a **race condition**. This happens when the final result of an operation depends on the unpredictable timing of which thread runs when. The code might work 99% of the time but fail randomly under heavy load, making these bugs a nightmare to find and fix.

#### 3. Safer Models for Concurrency
Instead of fighting with shared state, the book recommends using concurrency models that are inherently safer because they *avoid* shared state.

*   **The Actor Model (or Communicating Sequential Processes):** This is the book's preferred approach. Instead of sharing memory, your concurrent tasks are completely isolated, like little independent programs (called "actors" or "processes"). They don't touch each other's data. If they need to coordinate, they send each other messages through a channel (like a queue).
    *   **The Rule:** "Don't share memory by communicating; instead, communicate by sharing memory" is a typo in the original book. The correct principle is: **"Share memory by communicating, not by sharing memory."** (This is a famous typo they corrected later). The core idea is to pass messages instead of letting threads access the same data.

*   **Immutable Data:** If the data you share between threads cannot be changed (it's immutable), then there's no risk of a race condition. Many threads can read the same data at the same time without any conflict.

The goal is to design your system so that you don't need low-level tools like locks and semaphores if you can avoid them.

---

### Python Example: The Danger of a Race Condition

Let's demonstrate a race condition with a simple counter. We want two threads to increment a counter 1,000,000 times each. The final result should be 2,000,000.

#### The "Broken" Code (with a Race Condition)

This code uses threads that share a mutable `value`.

```python
import threading
import time

class SharedCounter:
    def __init__(self):
        self.value = 0

    def increment(self):
        # This operation is NOT atomic and causes a race condition
        current_value = self.value  # 1. Read the value
        time.sleep(0) # A tiny sleep to make the race condition more likely
        self.value = current_value + 1 # 2. Write the new value

def worker(counter, times):
    """A worker function that calls increment multiple times."""
    for _ in range(times):
        counter.increment()

# --- Setup and Run ---
counter = SharedCounter()
iterations = 1_000_000

# Create two threads that both work on the SAME counter object
thread1 = threading.Thread(target=worker, args=(counter, iterations))
thread2 = threading.Thread(target=worker, args=(counter, iterations))

thread1.start()
thread2.start()

thread1.join() # Wait for thread 1 to finish
thread2.join() # Wait for thread 2 to finish

print(f"Expected final value: {2 * iterations}")
print(f"Actual final value:   {counter.value}")
```

**Why it breaks:** When you run this, the "Actual final value" will be much less than 2,000,000. This is the **race condition**:
1.  Thread 1 reads `value` (e.g., it's 50).
2.  The operating system pauses Thread 1 and runs Thread 2.
3.  Thread 2 reads `value` (it's still 50).
4.  Thread 2 calculates `50 + 1` and writes `51` back to `value`.
5.  Thread 1 wakes up. It doesn't know Thread 2 did anything. It just remembers reading 50, so it calculates `50 + 1` and also writes `51` back to `value`.

Both threads ran, but the counter only increased by one. An increment was lost!

#### The "Pragmatic" Solution (Using a Lock)

The classic way to fix this specific problem is with a **lock**, which ensures that only one thread can execute a piece of code at a time.

```python
import threading

class LockedCounter:
    def __init__(self):
        self.value = 0
        self._lock = threading.Lock() # Add a lock to the object

    def increment(self):
        # The 'with' statement automatically acquires and releases the lock
        with self._lock:
            # Everything inside this block is safe from race conditions.
            # Only one thread can be in here at a time.
            current_value = self.value
            # The sleep is no longer needed, but even if it were here,
            # the code would still be correct.
            self.value = current_value + 1

# --- Setup and Run (the worker function is the same) ---
locked_counter = LockedCounter()
iterations = 1_000_000

thread1 = threading.Thread(target=worker, args=(locked_counter, iterations))
thread2 = threading.Thread(target=worker, args=(locked_counter, iterations))

thread1.start()
thread2.start()

thread1.join()
thread2.join()

print(f"\n--- With a Lock ---")
print(f"Expected final value: {2 * iterations}")
print(f"Actual final value:   {locked_counter.value}")
```

**Why this works:** The `with self._lock:` statement protects the critical section of code. When Thread 1 enters this block, it "acquires the lock." If Thread 2 tries to enter, it is forced to wait until Thread 1 finishes and "releases the lock." This makes the read-and-write operation **atomic** (un-interruptible), fixing the race condition.

While locks work, the book's main point is that designing your system with actors or immutable data is often an even better, higher-level solution that can prevent you from needing locks in the first place.
