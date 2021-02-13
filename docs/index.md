# Design for Project 0

## Part 1: Scheduler Class

A FIFO scheduler needs to maintain a FIFO queue of threads. Add a private member to Scheduler,

```cpp
    List<Thread*>* readyList;
```

The Scheduler constructor initializes readyList to an empty `List<Thread*>`, and the Scheduler destructor deletes it.

Scheduler::ReadyToRun simply enqueues a thread by appending it to the end of the ready list:

```cpp
    Scheduler::ReadyToRun(Thread* thread)
        append thread to readyList
```

Scheduler::FindNextToRun dequeues a thread by removing a thread from the front of the ready list. If no threads are on the ready list, returns null.

```cpp
    Scheduler::FindNextToRun()
        if readyList is empty
            return null
        else
            remove thread from front of readyList
            return thread
```

Lastly, Scheduler::Run switches from the current thread to the specified thread. At this point, not much needs to be done: just set kernel->currentThread to the next thread, then call SWITCH. Note that if you don't set kernel->currentThread before calling SWITCH, it's really hard for the next thread to figure out who it is.

```cpp
    Scheduler::Run(Thread* nextThread)
        set oldThread to kernel->currentThread
        set kernel->currentThread to nextThread
        SWITCH(oldThread, nextThread)
```

## Part 2: Thread Class

There are only two pieces of important information that a TCB must store: (1) the thread's register set, and (2) information about the stack (used on thread termination to delete the stack). Both of these pieces are already contained in the Thread class, so no additional state will be added.

ThreadRoot calls Thread::Begin right before it calls the function being forked to. Thread::Begin must complete all Thread initialization that couldn't be done while the thread wasn't yet running. For now, nothing falls in this category, so Thread::Begin can remain empty.

Thread::Yield tries to find another thread to run by calling Scheduler::FindNextToRun(). If it is successful, it moves the current thread to the ready list, and runs the other thread.

```cpp
    Thread::Yield()
        set nextThread to kernel->scheduler->FindNextToRun()
        if nextThread is non-null
            kernel->scheduler->ReadyToRun(this)
            kernel->scheduler->Run(nextThread)
```

Thread::Sleep also tries to find another thread to run by calling Scheduler::FindNextToRun(). If it is successful, it runs the other thread. Otherwise, it calls Interrupt::Idle to busy wait (effectively) for an interrupt. Since no threads are scheduled to run, only an interrupt can give us anything to do.

```cpp
    Thread::Sleep()
        set nextThread to kernel->scheduler->FindNextToRun()
        while nextThread is null
            kernel->interrupt->Idle()
            set nextThread to kernel->scheduler->FindNextToRun()
        kernel->scheduler->Run(nextThread)
```

Thread::Fork prepares a thread to be run, and then places the thread on the ready list. For now, this preparation consists only of calling StackAllocate() to prepare the thread's stack and register set.

```cpp
    Thread::Fork(VoidFunctionPtr func, void* arg)
        StackAllocate(func, arg)
        kernel->scheduler->ReadyToRun(this)
```

Thread::Finish is called when the thread is voluntarily terminating. This is either because the forked function explicitly called Thread::Finish, or because the forked function returned to ThreadRoot, which calls Thread::Finish as a last step. This thread must be destroyed, but it can't be destroyed right now because it's the current thread. Instead, the current thread is scheduled to be destroyed later and put to sleep. To accomplish this, add another private member to the Scheduler class, initialized to null by the Scheduler constructor:

```cpp
    Thread* toBeDestroyed;
```

Add a public method, Scheduler::CheckToBeDestroyed(), that checks toBeDestroyed and destroys it if it is non-null:

```cpp
    Scheduler::CheckToBeDestroyed()
        if toBeDestroyed is non-null
            delete toBeDestroyed
            set toBeDestroyed to null
```

Add a flag to Thread::Sleep() and Scheduler::Run(), bool finishing, that is TRUE if the thread giving up the processor is finishing. Thread::Sleep() passes this flag on to Scheduler::Run() after it finds another thread to run:

```cpp
    Thread::Sleep(bool finishing)
        ...
        kernel->scheduler->Run(nextThread, finishing)
```

Scheduler::Run checks this flag, and if it is set, sets toBeDestroyed to the current thread:

```cpp
    Scheduler::Run(Thread* nextThread, bool finishing)
        set oldThread to kernel->currentThread
        if finishing is TRUE
            set toBeDestroyed to oldThread
        set kernel->currentThread to nextThread
        SWITCH(oldThread, nextThread)
```

Therefore Thread::Finish() is simply a call to Thread::Sleep():

```cpp
    Thread::Finish()
        Sleep(TRUE)
```

Scheduler::CheckToBeDestroyed() must be called immediately after every context switch (i.e. every time SWITCH returns). There are two places that SWITCH can return to. The first, more obvious one is Scheduler::Run():

```cpp
    Scheduler::Run(Thread* nextThread, bool finishing)
        ...
        SWITCH(oldThread, nextThread)
        CheckToBeDestroyed()
```

The second place is less obvious. When a thread is forked, SWITCH actually returns to ThreadRoot, not to Scheduler::Run. ThreadRoot first calls Thread::Begin, which we now have a use for:

```cpp
    Thread::Begin()
        kernel->scheduler->CheckToBeDestroyed()
```

## Part 3: Preemption

Preemption significantly complicates things because it requires that the timer interrupt handler be allowed to call Thread::Yield(), which in turn accesses and modifies the ready list. We therefore require that interrupts already be disabled whenever anybody calls Scheduler::ReadyToRun, Scheduler::FindNextToRun, Scheduler::Run, or Scheduler::CheckToBeDestroyed. The direct implications of this are that Thread::Yield, Thread::Sleep, and Thread::Fork must ensure interrupts are disabled before calling Scheduler methods. For simplicity we require that interrupts already be disabled whenever Thread::Sleep is called. This requires a change to Thread::Finish():

```cpp
    Thread::Finish()
        kernel->interrupt->setLevel(IntOff)
        Sleep(TRUE)
```

Thread::Yield and Thread::Fork will disable interrupts on entry and restore them on exit:

```cpp
    Thread::Yield()
        DisableInterrupts()
        ...
        RestoreInterrupts()

    Thread::Fork(VoidFunctionPtr func, void* arg)
        StackAllocate(func, arg)
        DisableInterrupts()
        kernel->scheduler->ReadyToRun(this)
        RestoreInterrupts()
```

Since interrupts are disabled when Scheduler::Run() is called, interrupts are also disabled when SWITCH is called, and when it returns. Since SWITCH returns to ThreadRoot when a thread runs for the first time, Thread::Begin is called with interrupts disabled. However, we want the forked function to be called with interrupts enabled. So Thread::Begin should enable interrupts after calling Scheduler::CheckToBeDestroyed():

```cpp
    Thread::Begin()
        kernel->scheduler->CheckToBeDestroyed()
        kernel->interrupt->SetLevel(IntOn)
```

Now it is safe to call Thread::Yield() from within the timer interrupt handler, so we can move on to the Alarm class.

Create an Alarm class, derived from CallBackObj. Alarm has one private member:

```cpp
    Timer* timer;
```

The Alarm constructor points timer at a new Timer, using the alarm object as the callback object, and not using random time-slicing for now:

```cpp
    Alarm::Alarm()
        timer = new Timer(FALSE, this)
```

The Alarm destructor destroys the timer. The Alarm callback tells the Interrupt object to yield on return. This will cause Thread::Yield() to be called on the current thread before the interrupt handler returns.

```cpp
    Alarm::CallBack()
        kernel->interrupt->YieldOnReturn()
```

[Next](./part-2.md)
