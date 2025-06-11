# Recursive Mutex Implementation in C

This project provides an implementation of a recursive mutex (`rec_mutex_t`) in C using POSIX threads. A recursive mutex allows the same thread to acquire the lock multiple times without causing a deadlock, as long as it releases the lock the same number of times.

## File Structure

- [`rec_mutex.h`](rec_mutex.h): Header file defining the `rec_mutex_t` structure and function prototypes.
- [`rec_mutex.c`](rec_mutex.c): Implementation of the recursive mutex functions.

## Data Structure

The `rec_mutex_t` structure contains:
- `n`: Number of times the lock has been acquired by the owning thread.
- `locking_thread`: The thread currently holding the lock.
- `cv`: Condition variable for blocking/waking up waiting threads.
- `state_mutex`: Mutex to protect the internal state.
- `n_waited`: Number of threads waiting for the lock.

## Function Logic

### `rec_mutex_init(rec_mutex_t *rec_mutex)`

Initializes the recursive mutex:
- Sets the lock count (`n`) to 0.
- Sets the owner thread to 0 (no owner).
- Initializes the condition variable and state mutex.
- Sets the waiting thread count to 0.

### `rec_mutex_lock(rec_mutex_t *rec_mutex)`

Acquires the recursive mutex:
1. Locks the internal state mutex.
2. **If the mutex is not locked** (`n == 0`):  
   - Sets the owner to the current thread.
   - Increments the lock count.
3. **If the current thread already owns the mutex**:  
   - Increments the lock count.
4. **If another thread owns the mutex**:  
   - Increments the waiting count.
   - Waits on the condition variable until the mutex becomes available.
   - Decrements the waiting count after being signaled.
5. Sets the owner and lock count after acquiring.
6. Unlocks the internal state mutex.

### `rec_mutex_unlock(rec_mutex_t *rec_mutex)`

Releases the recursive mutex:
1. Locks the internal state mutex.
2. **If the mutex is not locked**:  
   - Asserts failure (should not happen).
3. **If the current thread owns the mutex**:  
   - Decrements the lock count.
   - If the lock count is still above 0, simply unlocks and returns.
   - If the lock count reaches 0 and there are waiting threads, signals one of them.
   - Resets the owner to 0.
4. **If another thread tries to unlock**:  
   - Asserts failure (illegal operation).
5. Unlocks the internal state mutex.

### `rec_mutex_destroy(rec_mutex_t *rec_mutex)`

Destroys the recursive mutex:
- Asserts that the mutex is not locked, has no owner, and no waiting threads.
- Destroys the internal state mutex and condition variable.

## Usage

Include `rec_mutex.h` in your project and use the provided functions to create and manage recursive mutexes in your multithreaded applications.
