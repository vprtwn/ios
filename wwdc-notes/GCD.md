### Quality of Service
* New in iOS8
    * UserInteractive = main thread animations
    * UserInitiated = immediate results (user blocking?)
    * Utility = long running tasks (is user aware of progress?)
    * Background = not user visible
* Complex resource controls
    * CPU scheduling priority
    * I/O priority
    * CPU throughput vs efficiency

### GCD Design Patterns with QoS
#### Inferred QoS
* UserInteractive is automatically translated to UserInitiated when dispatching from main to background
* Used if no QoS specified
* Will never lower QoS

#### Block QoS
* `dispatch_block_create_with_qos_class(0, QOS_CLASS_Utility, 0, ^{_})`
* `DISPATCH_BLOCK_ASSIGN_CURRENT`
    * captures current QoS
    * can store a callback block for later submission
* `DISPATCH_BLOCK_DETACHED`
    * way to opt-out of QoS
    * "this block has nothing to do with flow of execution"
    * doesn't capture properties of execution context
* `DISPATCH_BLOCK_ENFORCE_QOS_CLASS`
    * want value in block, override value in queue

#### Asynchronous Priority Inversion
* Didn't really get this

#### Queues as locks
* `DISPATCH_QUEUE_SERIAL`
* use `dispatch_sync(data.q, ^{_})` to execute critical section block

#### Synchronous Priority Inversion
* High QoS thread waiting on lower QoS work
    * QoS of waited on work is raised for
        * `dispatch_sync()` and `dispatch_block_wait()` of blocks on serial queues
        * `pthread_mutex_lock()`

### Queues, Threads, and Run Loops
* GCD has an ephemeral thread pool

#### Run loop vs serial queue
* Run loop
    * bound to a thread
    * get delegate method callbooks
    * autorelease pool pops after each interation
    * can be used reentrantly
* Serial queue
    * Use ephemeral threads
    * Block callbacks
    * Autorelease pool pops when thread idle
        * could never happen if app is busy
    * Will deadlock if used reentrantly
* Timer APIs
    * RunLoop
        * `[NSObject performSelector:withObject:afterDelay:]`
        * `[NSTimer scheduledTimerWithTimeInterval:]`
    * Queue
        * `dispatch_after()`
        * `dispatch_source_set_timer()`

#### Thread creation and pooling
* Waiting
    * A thread waits (blocks) when it needs to wait for a resource such as I/O or locks
    * When a thread waits, GCD may spin up a new thread to ensure one thread per core
    * Thread explosion - lots of waiting threads
        * exceeding thread limit can result in deadlock

#### Avoid Thread explosion
* Use async APIs
* Use serial queues
* Use NSOperationQueues with concurrency limits
* Don't generate unlimited work
* Be careful about mixing sync and async from the main thread

```objc
// fast, just a lock
dispatch_sync(q, ^{_});

// fast, just an atomic enqueue
dispatch_async(q, ^{_});

// slow, has to wait for a thread to complete above block
dispatch_sync(q, ^{_});
```

* Use `dispatch_apply` to allow GCD to manage parallelism

```objc
// DANGEROUS
for (int i=0; i<999; i++) {
    dispatch_async(q, ^{...});
}
dispatch_barrier_sync(q, ^{});

// GOOD
dispatch_apply(999, q, ^(size_t i){...})
```

* Use `dispatch_semaphore` to limit number of concurrent tasks
    * Useful if submitting blocks from different parts of app, so `dispatch_apply` doesn't make sense

```objc
#define CONCURRENT_TASKS 4
sema = dispatch_semaphore_create(CONCURRENT_TASKS);
for (int i=0; i<999; i++) {
    dispatch_async(q, ^{
        // do work
        dispatch_semaphore_signal(sema);
    });
    dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
}
```

### GCD and crash reports
* `_dispatch_mgr_thread`
    * helps process dispatch sources
* Idle GCD thread
    * `start_wqthread` 
    * `_workq_kernreturn`
* Active GC thread
    * `start_wqthread` 
    * `_dispatch_client_callout`
    * `dispatch_call_block_and_release`
    * your code 
    * will also see queue name
* Idle main thread
    * queue name = com.apple.main-thread
    * `__CFRunLoopRun`
    * `__CFRunLoopServiceMachPort`
    * `mach_msg_trap`
* Active main thread
    * `__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__`
    * `__NSBLOCKOPERATION_IS_CALLING_OUT_TO_A_BLOCK`

