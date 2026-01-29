## abort

```python
def abort(handle: ControllerFork[T, X, M], error: Exception) -> Task
```
Aborts a given task by throwing an error.
This function terminates the task with an error by calling conclude.
If the result is a failure, an exception is injected into the generator and the task behaves as if it raised the error itself.
The task may still remain active temporarily because:
it has a finally block
cleanup logic yielded control during termination

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
