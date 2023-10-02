# Effective modern C++ - The concurrency API

### Prefer task-based programming to thread_based

- 2 approach to run a function asynchronously

	- Thread-based : using `std::thread`

		- ```c++
			std::thread t(DoSomeWork);
			```

			

	- Task-based: using `std::async`

		- ```c++
			auto fu = std::async(DoSomeWork); // The DoSomeWork function is considered as a task
			```

		- Should be preferred over using the thread approach

			- The return value of the task can be captured
			- `std::async` offers a get function that can even get exceptions from the task
			- Omit the details of thread management

- Thread in concurrent C++ software

	- Hardware thread: the physical thread that performs the computation. multi-thread per CPU core
	- Software thread: Operating system level thread, managed across all processes + the OS is responsible to manage which gets executed, use a schedular to determine when the thread will be executed 
		- Usually more software thread than hardware thread
	- `std::thread`: Handles on the software threads
		- Objects could be in null state, moved to a software thread for processing + joined back when the thread process is finished + detached when the connection with the software thread is terminated

- Limit amount of software thread, will throw exception even is `noexcept` is used

	- If the software uses a single thread, could lead to unbalanced load + slow GUI response time
	- Hard to set a running time table for multi-thread environments

- Oversubscription will lead to thread schedular time slice hardware slice

	- Could lead to overhead to manage the system
	- Costly when a software thread has switch to different hardware thread, could result invalid all previous result + the thread has to start over again
	- Difficult to avoid oversubscription: Platform dependent, machine dependent

- `std::async` deals with all the multi-threading problems for you

	- Does not guarantee to create a new software thread, schedule the task to be run (does not have to run immediately)
	- Avoid the load-balancing issue, the system is better at handling it anyway
	- Using `std::launch::async` will ensure that the function is ran on a different thread

- Task-based design is easier to manage + provide direct way to access the result from the asynchronously function

- Should still use `std::thread` in the following situation

	- Need access to the API's threading implementation
	- Optimize thread specifically for the application
	- Implement thread technology beyond C++ concurrency

### Specify `std::launch::async` if synchronicity is essential

- When running a function with `std::async`, the program really request the function to be run accord to `std::aynsc` launch policy

- 2 standard policy (Represented by scoped `enum`)

	- `std::launch::async`: The function must be run on a different thread
	- `std::launch::deferred`: The function is only run when the result is needed + until a call to the function has been made
		- Call will execute synchronously (on the same thread)

- The default launch policy of `std::async` is combinational of the 2 standard policies

	- Function can run either asynchronously / synchronously
	- Depends on the load balancing + # of available threads, the system can choose which 1 to use

- Implication of the default launch policy

	- Cannot predict which policy will be used
	- Cannot predict which thread the function will ran on
	- Cannot predict if the function will run at all
		- Do not guarantee that a get / wait will be called for every `std::async` object
	- Cannot predict which thread-local storage will be used
	- Wait-based loop using timeout might never exit the loop if deferred launch policy has been used
		- `std::async` will always return deferred instead of ready

- Implication of default policy is hard to catch in production, usually only shows in heavy load testing + use case

	- Due to oversubscription + thread exhaustion

- Solution: Run a 0 second timeout to determine if the task is deferred / run asynchronously

	- If it is not deferred, use it in a time-based loop to avoid the infinite looping

- To avoid async policy implication all together, consider pass `std::launch::async` as argument in the call

	- ```c++
		auto fut = std::async(std::launch::async, f); // make sure the function is launched asynchronously
		```

	- Can create a template function that perfect forward its function name + arguments along with the policy, to avoid writing the argument every time

		- ```c++
			template <typename T, typename ...As>
			inline std::future<typename std::result_of<F(As...)>::type> ReallyAsync(T&& T, As&& ... params)
			{
			    return std::async(std::launch::async, std::forward<T>(T), std::forward<As>(params)...); // Return the async object that has the launch policy + the function + parameters forwarded
			}
			```

### Make `std::thread` un-joinable on all paths

- Every `std::thread` can either be joinable / un-joinable
  - Joinable: Thread of execution is active, currently executing / executed but no joined yet
  	- Oversubscribed thread (blocked / waiting to be scheduled) or also joinable
  	- Thread that have completed the execution is also joinable
  - Un-joinable: 
    - Default-constructed `std::thread`, no associated function, not joinable
    - Moved `std::thread`: Like source object after a move operation, source `std::thread` is left in a un-specified state, un-joinable
    - `std::thread` that has been joined: Once joined, the thread can not join twice
    - `std::thread` that has been detached: Detaching the `std::thread` object from the executing thread, no longer needs the thread to execute
    	- Thread can be cleaned up after execution
    	- A thread must either be joined + detached before exiting the scope
    	- Could leave to issue where the main thread is detached but sub-thread are still running (dangling pointer but worse) 
- If destructor for a joinable thread is called, the program is terminated
	- Like `noexcept` function raise an exception (termination)
	- However, implicit `join` / `detach` is generally a worse option
		- Implicit `join`: Wait for the asynchronous thread to finish executing
			- Could cause performance anomalies + counterintuitive if we set conditions that needs to be met in order to join the thread
		- Implicit `detach`: Would destroy the connection between `std::thread` + the thread of execution
			- But the thread would continue to run + will still modify the memory block it once occupied
			- The calling function would be terminated due to detach, but the thread will continue to run in the stack frame + memory of the calling function's location
				- Thread function will take over stack frame + some / all memory it had
				- Result in changing values in memory unexpectedly
- Destroying a joinable could lead to catastrophe, hence when std::thread is first defined, is in a un-joinable state
	- But hard to consider every program flow, especially with jump conditions
- RAII: Resource accquisition is initialization, common in the standard library + smart pointer
	- However, not implemented for `std::thread`, doesn't really have use for it if the destructor is going to terminated the program anyway
- It's easy to implement a custom thread RAII class that takes a `std::thread` (R-value for move operation) to create custom destructor to determine if join / detach is appropriate
	- Good practice to declare the `std::thread` object last in parameter
		- In case they run immediately, all other parameters will be initialized first
- `std::thread` object can only change state through member function call
	- Simultaneous member function call is only safe if the member is a `const` object
- C++11 does not have native support for interruptible threads, could lead to `std::thread` to hung
- If we have custom destructor for a special wrapping class for `std::thread`, we also needs to implement explicitly declare move operation

### Be aware of varying thread handle destructor behavior

- `std::thread ` + future object are essentially handles for software threads
	- But they handle different when being destructed
		- `std::thread` will terminate the program
		- future object will sometime behave like implicit join, some like implicit detach + some time neither
- `std::future` creates a communication channel between the main thread (where future object is created, caller) + the async thread (software thread, callee)
	- The result from the callee should not be stored on caller / callee, instead a middle ground (shared state) should be used
		- Part of heap memory, but does not have be a specific object type / specific set of interfaces
- The behavior of the `std::future` destructor depends on the state of the associated shared state
	- If the future is the last future to the shared state + being destroyed has a shared state for non-deferred task launched using `std::async` (in other word, the thread is currently being executed), the destructor will be blocked until the async task is completed
		- Perform a implicit `join`
	- If the task has not started yet (until when the result are needed), then the thread will be `detached` and never executed
	- Default behavior: Future's destructor destroys the future object
		- Also decrement the reference counter on the shared state
		- To make exception from the default behavior, all conditions must meet
			- Shared state was created due to `std::async`
			- Task's launch policy must be `std::launch::async`
			- The future object must be the last future referring to the shared state
	- In short, future from `std::async` block the destructors
		- Avoid the problem with implicit detach
- There are other ways to create shared states (`std::packaged_task`)
	- Wrap a function object + prepare it for async execution

### Consider `void` futures for one-shot event communication

- It's necessary to create inter-thread communication for one thread to notify a second async running task that certain event occurred

- Obvious approach: condition variable
  - Create a task to detect the condition (Detection task)
  - The detection will change the condition variable when event occur, the reaction task waits on the condition variable to meet the condition

  - The reacting task will first lock the mutex using `std::unique:lock` 
  	- Then the task will enter the waiting state, waiting for events
  	- Once conditions variables changes + conditions been met, react to event in critical conditions first + apply the rest of the reaction after leaving the critical section
  - Issues with this approach:
  	- The mutex lock might not always be necessary if each events operates on different data
  	- If the detecting task notify the Condover before the reacting task waits, reacting task will hung
  		- Since it's a one off event, if the reaction task is not ready, then it will miss the notification + won't get another chance
  	- The wait statement fails to account for spurious wakeups
  		- The waited events might be triggered even when the Condvar wasn't notified (spurious wakeups)
  			- In C++, one should always use lambda function to make sure that the conditions has been met when starting the event

- Next approach: Shared Boolean flag

	- Flag initially false, flip when detecting corelated event
	- Advantage:
		- No need for mutex
		- Can detect task set the flag before the reacting task is activated
		- will not suffer from spurious wakeup
	- Issue:
		- The cost of polling: While waiting for the flag, the task is blocked but still running
			- Less power efficiency, taking up hardware resources that could be allocated otherwise, take up extra allocation resources (especially in round robin sinuation)

- Middle ground: combine condvar + flag-based design

	- Access to the flag is synchronized by mutex
	- The flag change will trigger the event on the reacting task
	- The detecting task will set the lock guard for the mutex when detected change + set the flag
	- The reacting task will lock the mutex, wait for the event to be triggered, when triggered use lambda to check the flag bit
	- However, this approach requires a double safe, the task can only start with a 2 conditions being met, not quite clean + efficient

- Alternative solution to completely avoid variables, mutex + flag

	- The reacting task can wait for the change on the future object, that can be changed by the detecting task
		- Future is a communication channel between the callee and the caller
	- The detecting task has the `std::promise`, reacting task has the future to receive the information
		- Detecting task sent event information using `std::promise`, the reacting task blocks the reaction until `std::promise` is set
	- If no data needs to be transmitted but only signal, use `void` as the type parameter
		- `std::promise<void>`, it will still be able to detect the writing without transmitting any data
	- Flaw of the alternative solution:
		- Shared state are dynamically allocated, additional cost of heap allocation + deallocation
		- `std::promise` can only be set up once, can only communicate once before being destroyed
			- Both condvar + flag can be used multiple times

### Use `std::atomic` for concurrency, `volatile`for special memory

- `volatile` in C++ is not directly associated with concurrency
- `std::atomic` template, can be instantiated, offer operations to guarantee tot be atomic by other thread
	- Using special machine instruction to behave as if it were in a mutex-protected critical section
	- More efficient than the traditional mutex
	- Once `std::atomic` object is constructed, all member function are guaranteed to be atomic
- `volatile` does not guarantee anything in multi-thread environment
	- Could be multiple thread reading + writing from the same `volatile` address, data race
- When data races happens, the result are usually un-predictable
	- Data race will cause undefined behavior, the compiler will generate code to avoid data race
- When the compiler determines that the assignment are in random order, it will optimize the code by re-ordering the assignment
	- If not compiler, the same optimization will be performed at hardware level
- `std::atomic` imposes restriction to maintain the code order
	- Order of codes surrounding `std::atomic` write operation must stay consistent
	- The compiler needs to generate extra code to ensure that at hardware level
- `votatile`: tells the compiler that the memory will have special behaviour
	- Normal memory behavior
		- If a value is written to a memory address, the value will stay the same until it's changed (overwritten) again
		- If a memory address is been overwritten multiple time, only the last overwritten is used
	- Quite common after template instantiation, inlining + re-ordering optimizations
	- Sometimes, the memory needs to behave differently
		- i.e. memory-mapped I/O
		- Need a way to tell the compiler to not perform any optimization on memory operation (`volatlie`)
	- `std::atomic` is not suited for this situation
		- Copy operation are deleted for `std::atomic`
			- In order to perform copy operation, hardware needs to perform reading one `std::atomic` and writing to the other, usually not supported by the hardware
