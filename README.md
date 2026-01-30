# Task Control Functions

## abort

```python
def abort(handle: ControllerFork[T, X, M], error: Exception) -> Task
```

Aborts a given task by throwing an error.

This function terminates the task with an error by calling conclude. If the result is a failure, an exception is injected into the generator and the task behaves as if it raised the error itself.

The task may still remain active temporarily because:
- it has a finally block
- cleanup logic yielded control during termination

## all

```python
def all_(tasks: Iterable[Task[M, T]]) -> Task[Any, list[T]]
```

Runs multiple tasks concurrently and returns their results in order. Equivalent to Promise.all but that does not have cancellation.

## batch

```python
def batch(effects: list[Effect[T]]) -> Effect[T]
```

Takes several effects and combines them into one effect.

## effects

```python
def effect(task: Task[None, T]) -> Effect[T]
```

Converts a task (that never fails or sends messages) into an effect that produces its result as a message. Useful for converting task results into the message stream.

## enqueue

```python
def enqueue(task: ControllerFork[T, X, M]) -> None:
```

Every task belongs to exactly one task group, then it marks the task as runnable and removed from idle if the task was previously blocked, then the code inspect parent group scheduler queues and locate the driver (a child group may have a driver but MAIN will have no driver as it will be root of the tree and then we unblock the driver if it is idle and stop if already unblocked and move upwards in the tree and scheduler loop executes tasks until no idle tasks remain, if some task crashes crashing task is removed and unrelated tasks take place.

## exit

```python
def exit_(handle: ControllerFork[T, X, M], value: Any) -> Task[None, None]
```

Concludes the task with a Success result. Executes any finally blocks in the task (calls conclude with return value). Task handlers are called with the success value.

## fork

```python
def fork(task: Task[M, T], options: ForkOptions | None = None) -> Fork[T, X, M]
```

Creates a Fork wrapper around the task but does not start execution immediately (lazy).

**Usage Patterns:**
- Concurrent execution: `fk = yield from fork(work())`
- Deferred joining: `fk = fork(work()); ...; result = yield from fk.join()`
- Async integration: `result = await fork(work())`

## group

```python
def group(forks: list[Fork[T, X, M]]) -> Task[Optional[Instruction[M]], None]
```

It groups multiple forked tasks together and joins them with the current task so that all group tasks either complete successfully or the first failure should abort the entire group.

**Working:**
- A new task group is created and current task acts as a driver of the group
- Each fork is checked if they have already finished, it is not moved into the group and failure is recorded and continues checking the next fork
- Unfinished forks are moved into the group
- If grouped tasks are blocked (idle):
  - The driver suspends itself
  - It will be resumed automatically when a child task wakes up (because enqueue() unblocks the driver)

**Abort handling:**
```python
# Abort all idle tasks and re-enqueue them
for task in group.stack.idle:
    yield from abort(task, error)
    enqueue(task)
```

Idle tasks may still have cleanup code (finally blocks), so:
- They are aborted
- Then re-enqueued so the abort fully completes

## join

```python
def join(fork: Fork[T, X, M]) -> Task[Optional[Instruction[M]], T]
```

- If fork is idle, activates it
- If fork hasn't completed, groups it and waits for completion
- Returns the fork's success value
- Throws the fork's error if it failed


