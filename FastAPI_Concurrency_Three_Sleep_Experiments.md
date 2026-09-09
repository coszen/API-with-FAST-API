# FastAPI Concurrency Experiment: `def`, `async def`, `time.sleep()` and `asyncio.sleep()`

## Overview

This document compares three FastAPI endpoint patterns:

1. `def` + `time.sleep(5)`
2. `async def` + `await asyncio.sleep(5)`
3. `async def` + `time.sleep(5)`

The purpose is to understand blocking vs non-blocking behavior, FastAPI/Starlette's thread-pool execution for synchronous endpoints, the asyncio event loop, and why the same 5-second sleep can produce approximately 5, 15, or 500 seconds of total test time depending on how it is used.

---

# 1. The Three Endpoints

## 1.1 `def` + `time.sleep(5)`

```python
@app.get("/sleep-sys-time")
def time_sys_sleep():
    time.sleep(5)
    return {"message": "time.sleep(5) is used with no function as async"}
```

`time.sleep(5)` is a **blocking operation**.

For a normal synchronous FastAPI route (`def`), FastAPI/Starlette executes the endpoint using its thread-pool mechanism rather than directly running the synchronous function on the event-loop thread.

The sleep therefore blocks a worker thread, but does not directly block the event loop.

---

## 1.2 `async def` + `await asyncio.sleep(5)`

```python
@app.get("/sleep-sys-asyncio")
async def asyncio_sleep():
    await asyncio.sleep(5)
    return {"message": "await asyncio.sleep(5) used"}
```

This is an asynchronous endpoint.

`asyncio.sleep(5)` is an asynchronous waiting operation. At:

```python
await asyncio.sleep(5)
```

the coroutine pauses while the event loop remains available to work on other asynchronous tasks.

This is **non-blocking waiting**.

---

## 1.3 `async def` + `time.sleep(5)`

```python
@app.get("/sleep-async-time")
async def async_time_sleep():
    time.sleep(5)
    return {"message": "async def with blocking time.sleep(5)"}
```

This is the problematic combination.

Although the function uses `async def`, `time.sleep(5)` is still blocking.

Because an `async def` route normally runs on the event-loop thread, `time.sleep(5)` blocks that event loop.

The `async` keyword does not magically make blocking code non-blocking.

---

# 2. `async def` Does Not Automatically Mean Non-Blocking

A common misconception is:

```text
async def
    ↓
non-blocking
```

That is incomplete.

The more accurate model is:

```text
async def + asynchronous operation
    ↓
can yield control to the event loop
```

Whereas:

```text
async def + time.sleep()
    ↓
still blocking
```

For example:

```python
async def bad_endpoint():
    time.sleep(5)
```

is syntactically asynchronous, but it contains a blocking operation.

---

# 3. What Does "Blocking" Mean?

Blocking means the current execution resource cannot continue doing other work while the operation is waiting.

For:

```python
time.sleep(5)
```

the thread executing the call is unavailable for approximately five seconds.

Conceptually:

```text
Execution thread
      |
      v
time.sleep(5)
      |
      |  BLOCKED
      |
      v
continue execution
```

---

# 4. What Does `await` Do?

Consider:

```python
await asyncio.sleep(5)
```

The coroutine effectively tells the event loop:

> "I am waiting. I do not need to continue right now. You can work on another coroutine and return to me when this operation is ready."

Conceptually:

```text
Request A
    |
    v
await asyncio.sleep(5)
    |
    v
Coroutine A pauses
    |
    +----------------------+
                           |
                    Event loop works
                    on other tasks
                           |
                           v
                    Request B
                    Request C
                    Request D
                           |
                           v
                    5 seconds pass
                           |
                           v
                    Coroutine A resumes
```

This is the central idea behind asynchronous I/O.

---

# 5. Where Is the Thread Pool?

You did not create a thread pool in your endpoint.

FastAPI is built on Starlette, which uses a thread-pool mechanism for synchronous work such as normal `def` path-operation functions.

The important point is:

> A new thread pool is not created for every API request.

Instead, synchronous work can use a shared pool/limiter with a finite number of concurrent thread-pool execution slots.

Conceptually:

```text
                    FastAPI
                       |
                       v
                Thread Pool
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
   Thread 1        Thread 2        Thread 3 ...
       |               |               |
   Request A        Request B        Request C
       |               |               |
 time.sleep(5)    time.sleep(5)    time.sleep(5)
```

In the Starlette configuration relevant to this experiment, the default thread-pool limiter is 40 tokens. Your 100-request experiment strongly demonstrates this limit.

---

# 6. Why the 100-Request Test Revealed the Thread Pool

You ran:

```zsh
python concurrent_runner.py http://127.0.0.1:8000/sleep-sys-time 100
```

The important result was:

```text
Requests 1-40   → approximately 5 seconds
Requests 41-80  → approximately 10 seconds
Requests 81-100 → approximately 15 seconds

Total time: 15.1079 sec
```

This is consistent with approximately 40 synchronous endpoint executions being able to occupy the available thread-pool slots at once.

The pattern is:

```text
100 requests
     |
     v
~40 requests execute
     |
     v
time.sleep(5)
     |
     v
first ~40 complete
     |
     v
next ~40 execute
     |
     v
another 5 seconds
     |
     v
remaining ~20 execute
     |
     v
another 5 seconds
```

Therefore:

```text
~5 sec
+ ~5 sec
+ ~5 sec
≈ ~15 sec
```

---

# 7. Your Actual `def + time.sleep(5)` Experiment

Command:

```zsh
python concurrent_runner.py http://127.0.0.1:8000/sleep-sys-time 100
```

Important portion of the terminal output:

```text
Request 1: 5.0699 sec | Status: 200
Request 2: 5.0559 sec | Status: 200
...
Request 40: 5.0524 sec | Status: 200

Request 41: 10.0569 sec | Status: 200
Request 42: 10.0490 sec | Status: 200
...
Request 80: 10.0523 sec | Status: 200

Request 81: 15.0555 sec | Status: 200
Request 82: 15.0551 sec | Status: 200
...
Request 100: 15.0557 sec | Status: 200
```

Summary:

```text
Total requests    : 100
Successful        : 100
Failed            : 0
Total time        : 15.1079 sec
Average time      : 9.0526 sec
Minimum time      : 5.0481 sec
Maximum time      : 15.0563 sec
Throughput        : 6.62 requests/sec
```

### Interpretation

The individual latency is different from total test time.

For example, a request completing at 10 seconds may have spent approximately 5 seconds waiting for an execution slot and another 5 seconds inside `time.sleep(5)`.

So:

```text
request latency
=
queue/wait time
+
actual endpoint execution time
```

---

# 8. Your Actual `async def + await asyncio.sleep(5)` Experiment

Command:

```zsh
python concurrent_runner.py http://127.0.0.1:8000/sleep-sys-asyncio 100
```

The requests all completed at approximately 5 seconds:

```text
Request 1: 5.0584 sec | Status: 200
Request 2: 5.0496 sec | Status: 200
Request 3: 5.0497 sec | Status: 200
...
Request 100: 5.0581 sec | Status: 200
```

Summary:

```text
Total requests    : 100
Successful        : 100
Failed            : 0
Total time        : 5.0928 sec
Average time      : 5.0550 sec
Minimum time      : 5.0402 sec
Maximum time      : 5.0645 sec
Throughput        : 19.64 requests/sec
```

### Interpretation

All 100 requests can enter the asynchronous waiting state.

Conceptually:

```text
R1 → await sleep ─┐
R2 → await sleep ─┤
R3 → await sleep ─┤
...               ├── approximately 5 seconds
R100 → await sleep┘
```

The event loop does not need one dedicated sleeping thread per request.

---

# 9. Your Actual `async def + time.sleep(5)` Experiment

Command:

```zsh
python concurrent_runner.py http://127.0.0.1:8000/sleep-async-time 100
```

The result was:

```text
Request 1: 5.0383 sec | Status: 200
Request 2: 500.5799 sec | Status: 200
Request 3: 500.5798 sec | Status: 200
...
Request 100: 500.5696 sec | Status: 200
```

Summary:

```text
Total requests    : 100
Successful        : 100
Failed            : 0
Total time        : 500.6193 sec
Average time      : 495.6196 sec
Minimum time      : 5.0383 sec
Maximum time      : 500.5815 sec
Throughput        : 0.20 requests/sec
```

This is the clearest demonstration of the danger of putting blocking code inside an asynchronous endpoint.

---

# 10. Why Did the Third Experiment Take Approximately 500 Seconds?

The endpoint effectively does:

```python
async def async_time_sleep():
    time.sleep(5)
    return {"message": "done"}
```

The request is handled by the event loop.

Then it reaches:

```python
time.sleep(5)
```

The event-loop thread is blocked for five seconds.

Conceptually:

```text
Event Loop
    |
    v
Request 1
    |
time.sleep(5)
    |
EVENT LOOP BLOCKED
    |
    X Request 2
    X Request 3
    X Request 4
    X ...
```

After approximately five seconds, Request 1 can complete and the event loop can move on.

Then another request blocks it for another five seconds.

So the pattern is approximately:

```text
Request 1    → 5 sec
Request 2    → 10 sec
Request 3    → 15 sec
Request 4    → 20 sec
...
Request 100  → 500 sec
```

Therefore:

```text
100 requests × 5 seconds
≈ 500 seconds
```

Your actual result:

```text
Total time: 500.6193 sec
```

matches this behavior very closely.

---

# 11. Why Did Request 1 Take 5 Seconds but the Others Take ~500 Seconds?

This is an important detail in your output.

You saw:

```text
Request 1: 5.0383 sec
Request 2: 500.5799 sec
Request 3: 500.5798 sec
...
```

The endpoint did not change from `time.sleep(5)` to `time.sleep(500)`.

The approximately 500-second value is the **client-observed end-to-end request latency**.

The client started many HTTP requests concurrently, but the server's event loop was repeatedly blocked by `time.sleep(5)`.

Those requests therefore remained outstanding while the server could not process them normally.

So the measured time includes the waiting/queueing caused by the blocked event loop.

---

# 12. The Three Experiments Compared

| Endpoint | Main execution model | 100 requests | Observed total |
|---|---|---:|---:|
| `def + time.sleep(5)` | Thread-pool execution | ~40 + ~40 + ~20 | **~15 sec** |
| `async def + await asyncio.sleep(5)` | Async event loop | Async waiting | **~5 sec** |
| `async def + time.sleep(5)` | Event loop blocked | Effectively serialized | **~500 sec** |

This is the central result of the experiment.

---

# 13. Visual Comparison

## 13.1 `def + time.sleep(5)`

```text
                    FastAPI
                       |
                       v
                Thread Pool
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
      R1              R2              R3
    sleep 5         sleep 5         sleep 5
       |               |               |
       +---------------+---------------+
                       |
                  ~5 seconds

Approximately 40 execution slots are available in the
thread-pool mechanism relevant to this experiment.

Next group:
R41-R80 → ~10 seconds

Final group:
R81-R100 → ~15 seconds
```

---

## 13.2 `async def + await asyncio.sleep(5)`

```text
                  Event Loop
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
      R1             R2             R3
      |              |              |
    await           await          await
   sleep(5)        sleep(5)       sleep(5)
      |              |              |
      +--------------+--------------+
                     |
               Event loop remains
                   available
                     |
                ~5 seconds
                     |
                     v
              Requests resume
```

---

## 13.3 `async def + time.sleep(5)`

```text
                  Event Loop
                      |
                      v
                     R1
                      |
                time.sleep(5)
                      |
              EVENT LOOP BLOCKED
                      |
             R2, R3, ... R100
                    WAIT
                      |
                  5 seconds
                      |
                     R2
                      |
                time.sleep(5)
                      |
              EVENT LOOP BLOCKED
                      |
                  5 seconds
                      |
                     R3
                      |
                     ...
                      |
                    R100
                      |
                time.sleep(5)
                      |
                      v
                ~500 seconds
```

---

# 14. Why the 10-Request Test Was Misleading

With only 10 requests, your synchronous endpoint was below the approximately 40-token thread-pool limit.

Therefore:

```text
def + time.sleep(5)
        ↓
10 requests fit into available thread-pool capacity
        ↓
~5 seconds
```

And:

```text
async def + await asyncio.sleep(5)
        ↓
10 asynchronous waits
        ↓
~5 seconds
```

They therefore looked identical from the client's total-time measurement.

Increasing the test to 100 requests exposed the difference:

```text
10 requests

def + time.sleep          → ~5 sec
async + asyncio.sleep     → ~5 sec


100 requests

def + time.sleep          → ~15 sec
async + asyncio.sleep     → ~5 sec
async + time.sleep        → ~500 sec
```

---

# 15. Important Distinction: Thread Pool vs Event Loop

Think of them as two different execution mechanisms.

```text
                         FastAPI
                            |
              +-------------+-------------+
              |                           |
            def                       async def
              |                           |
         Thread Pool                  Event Loop
              |                           |
       blocking work                async work
              |                           |
       time.sleep()              await asyncio.sleep()
              |                           |
    worker thread blocked         coroutine pauses
              |                           |
     other threads can work       event loop can work
```

The thread-pool approach uses multiple threads.

The async approach does not require one dedicated thread for every coroutine that is waiting asynchronously.

---

# 16. The Most Important Mental Model

### `time.sleep()`

```text
"I am waiting.
The execution resource running me is blocked."
```

### `await asyncio.sleep()`

```text
"I am waiting.
The event loop can work on something else."
```

This is the simplest mental model to retain.

---

# 17. `await` Is Not the Same Thing as `asyncio.sleep()`

These are different concepts.

```text
await
  |
  +--> tells the coroutine to pause until an awaitable completes

asyncio.sleep()
  |
  +--> provides an asynchronous timer/wait operation
```

Therefore:

```python
await asyncio.sleep(5)
```

means, conceptually:

```text
Wait asynchronously for 5 seconds.
While waiting, allow the event loop to handle other work.
```

---

# 18. Why This Matters for Real APIs

The same concept applies to real I/O.

For example:

```python
async def get_data():
    response = await call_external_api()
    return response
```

If the external API takes time to respond, the coroutine can wait asynchronously.

Other requests can make progress during that waiting period.

This is why async programming is particularly useful for I/O-bound work such as:

- HTTP/API calls
- Async database queries
- Network operations
- Other operations that spend substantial time waiting

---

# 19. What Async Does Not Automatically Solve

Async is not magic.

This:

```python
async def endpoint():
    time.sleep(5)
```

is still blocking.

Similarly, CPU-heavy work can block an event loop if it runs directly inside an async endpoint.

For CPU-heavy work, a different execution strategy may be appropriate.

The key is to identify what the operation is doing while it waits.

---

# 20. Final Takeaway

Your three experiments demonstrate three different behaviors:

```text
def + time.sleep(5)
        ↓
blocking operation
        ↓
FastAPI/Starlette can execute sync endpoint work
using its thread-pool mechanism
        ↓
limited by thread-pool capacity
        ↓
100 requests ≈ 15 seconds in your test
```

```text
async def + await asyncio.sleep(5)
        ↓
non-blocking asynchronous wait
        ↓
coroutine pauses
        ↓
event loop remains available
        ↓
100 requests ≈ 5 seconds
```

```text
async def + time.sleep(5)
        ↓
blocking operation inside async endpoint
        ↓
event loop is blocked
        ↓
other async requests cannot make progress normally
        ↓
100 requests ≈ 500 seconds
```

## Rule to remember

```text
Inside async def:

    ❌ time.sleep()
       blocks the event loop

    ✅ await asyncio.sleep()
       gives control back to the event loop
```

The deeper rule is:

> **The important distinction is not simply `def` versus `async def`. The important distinction is whether the operation blocks the execution resource that needs to handle other requests.**

---

# 21. Experimental Summary

Your actual measurements:

```text
┌──────────────────────────────────────┬───────────────┐
│ Experiment                            │ Total Time    │
├──────────────────────────────────────┼───────────────┤
│ def + time.sleep(5)                   │ ~15.11 sec    │
│ async def + await asyncio.sleep(5)    │ ~5.09 sec     │
│ async def + time.sleep(5)             │ ~500.62 sec   │
└──────────────────────────────────────┴───────────────┘
```

These results provide a practical demonstration of:

- Thread-pool concurrency
- Thread-pool capacity
- Event-loop concurrency
- Blocking operations
- Non-blocking asynchronous waits
- Request queueing
- Why blocking code inside an `async def` endpoint should be avoided

