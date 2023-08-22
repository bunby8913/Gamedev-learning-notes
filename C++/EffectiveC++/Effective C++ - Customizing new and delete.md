# Effective C++ - Customizing `new` and `delete`

- Manual memory management is an advantage in C++, allow programmer to tailor memory allocation + deallocation for the best possible performance

### Understand the behavior of the new-handler

- The `new` operations calls a new-handler when the operations fails / cannot be satisfied
  - At its core, new-handler is a typedef of a function pointer, a function does not return any value + no arguments
  - The `set_new_handler` function to take a new-handler (A function with no return value + parameters) and return a new-handler
  	- Keep in mind that this function should not throw any exception

- In production code, use set_new_handler function to determine the behavior when the new operations fails

- The `new` operator will continuously call the new handler until enough memory can be allocated to the operation

- The new-handler could handles the memory allocation in several ways
  - Make more memory available: Allocate some memory space when the program starts, release some of the memory when the new-handler is called
  - Install a different new-handler: If the current handler cannot allocate more memory, hand the operation to a different new-handler (using the `set_new_handler` function)
  - Deinstall the new-handler: If all the new-handler could not allocate enough memory for the operation, return a null pointer, indicate to the `new` operation that allocate is unsuccessful
  - Throw an exception: Throw a `bad_alloc` exception for the program that invoked the `new` operator to handle the error
  - Not return: Calling abort or quit the program

- Possible to create class-specific new-handlers + `new` operator definition
  - Declared them `static` function

  	- ```c++
  		static std::new_handler set_new_handler(std::new_handler p) noexcept;
  		static void* operator new(std::size_t size)throw(bad_alloc);
  		static std::new_handler currentHandler;
  		```

  - Make sure that the `static` function is defined outside the class definition

  - The new operator (The class-specific) should do the following

  	- First use the standard `set_new_handler` to change the new-handler to the class-specific one, replace the global new-handler
  	- Then, use the global `new` operator to attempt to allocate memory
  		- If allocation fails, call the class-specific new-handler. If the new-handler fails, throw an `bad_alloc` exception + restore the original new-handler
  	- If the global `new` operator is successful (allocated the correct amount of memory), the global `new` operator returns the pointer to memory to the class specific `new` operator, run the destructor which `set_new_handler` back to the global one

- The underlying code of the implementation stays the same, the only changes being the class-specific new-handler

	- Possible to make the new-handler + the overloaded function into mixin-style base class + templates
		- A base template class (does not use the type name T in the code) that class wants to implement custom new-handler can inherit from
		- A set of template functions that can be used to implement the new operators + new-handlers + functions to set the new handlers
	- Known as the curiously recurring template pattern (Do it for me)

- `nothrow` is an alternative that returns null pointer to a failed allocated new operator

	- Called before the usual `new` operator, if failed, return a null pointer
		- However, is succussed, the class constructor gets called + which might use some `new` operator that are not `nothrow` defined, exceptions will be used instead

### Understand when it makes sense to replace `new` + `delete`

- Detect usage errors: Failure to delete memory leads to memory leaks
  - `delete` the same memory multiple times lead to undefined behavior
  	- Custom `new` operator can keep track of list of allocated memory (a list) which then can be used by `delete`
  - Detect data overrun (writing beyond the end of the allocated block) + data underruns (writing before the start of the allocated block)
  	- Custom `new` operator can allocated additional block for "signature" before + after the allocated block, use signature to detect if overrun / underrun occurred
- Improve efficiency: The `new` + `delete` operator for compiler have to be for general-purpose, design decision full of middle-of-the-road strategy
  - `new` + `delete` strategy that are more optimized for some usage are more likely to be more efficient (up to 50%)
- Collect usage statistics / detect usage errors: Determines how the software uses dynamic memory
  - FIFO / LIFO / random order
  - Different allocation pattern at different stages of the software
  - Maximum amount of memory needed for the software (The high water mark)
  - Many computer architecture requires particular type of data placed in specific types of addresses
  	- i.e. Double can only be placed in addresses divisible by 8 + pointer can only be placed in address divisible by 4
  	- Not followed properly could lead to hardware exceptions
  	- Some architecture are more lenient, data can be stored anywhere, but access time will be significant slower if not placed according to the rule
  - Manually add extra data in memory could lead to mis-alignment / unsafe code, could lead to code crashing / the software runs significantly slower
  - Most compiler has implemented feature to debug + log functionality of the memory management functions
  	- Alternatively, use an open source memory managers / at least reference the implementation code
- Increase the speed of allocation + deallocation: Allocation + deallocation for specific types could be more efficient (fixed-size allocator)
- Reduce the space overhead of default memory management: General-purpose allocator uses more memory from overhead for each allocated block, which can be eliminated
- Compensate for suboptimal alignment in the default allocator: Some compiler does not perform the proper memory alignment for different type of data to achieve memory optimization, Replace the default new + delete operator can improve memory performance
- Cluster related objects near one another: To ensure related object is clustered on the same page for faster access + better hit rate
	- Create a separate help for the data structure to clustered multiple data together
- Obtain unconventional behavior: When the `new` + `delete` operator needs to perform some specific function that are normally not provided

### Adhere to convention when writing `new` + `delete`

- `new` operator conventions
	- Return the correct return value: Return the pointer to the allocated memory
		- Throw exception of `bad_alloc` is there are insufficient memory
			- More specifically, bad memory allocation will call the new-handling function repeatedly, until the function returns null, in this case, throw the exception  
	- Call the new-handling function when there are insufficient memory
	- Handles the special case where client request no memory
		- Even if the client request to allocate no memory, C++ requires the operator to return a legit pointer (Always request at least 1 byte of memory)
		- To handles any issues with the allocation, we must first the new-handling function pointer to null, the reset it to the original new handling
			- No way to get new-handling function pointer directly
		- Safe for single threaded code, might require some form of thread lock to ensure thread safety
		- Using a infinite loop until either the memory has been successfully allocated / new-handling functions fails and return a null, throw a `bad_alloc` expectation
	- Although the `new` operator is inherited by the derived class, the `new` operator is tuned for object of specific sizes, the `new` operator does not work on inherited / parent class
		- Solution: Use a condition to check the size of the object and the size of the base object, only run the `new` operator if they are the same
			- Since no object in C++ has the size of 0, it's not necessary to write a special case condition check for the case the object is 0
	- Use `new[]` (Array new) for class specific memory allocation
		- Simply allocate the necessary memory, does not form the object
		- `new[]` can be called by the derived class, which might be different size than the base class, cannot assume the size of the object in `new[]`
			- By extension, cannot determine the # of objects in the `new[]`
- `delete` operator convention
	- C++ guarantees its always safe to delete a null pointer
		- The delete operation don't have to do anything when deleting a null pointer
	- Needs to check for the size of the object / pointer being deleted
		- Needs to make sure the correct object is used to calculate the size of the object / memory used, especially in the case the virtual destructor is omitted

### Write placement `delete` if you write placement `new`

- When the memory allocation succeed but object construction failed, the program must deallocated the memory to avoid a memory leak
	- Client can't unallocated manually, they don't have the pointer to the memory
	- The system needs to know which version of `delete` to call on the pointer
		- Only an issue when custom, non-normal form of `new` operator are being used (takes additional parameter)
- "Placement new": a `new` operator function that takes extra parameter other than the mandatory `size_t` argument (usually a pointer)
	- A common version of placement new includes a pointer indicating the location the object should be constructed
		- Part of the standard library, used by Vector to create object in unused capacity
	- Placement new can also be used to describe overloaded `new` operator that takes additional arguments that are not a pointer
- The run-time system will look for `delele` that has the same # + type of extra arguments as the `new` operator
	- If not found, the proper `delete` action can't be completed, cause memory leakage
	- The "placement delete" needs to be used to accompany placement new
- The placement delete is only called when the constructor throws an exception, the normal `delete` occurs in the client code uses the normal `delete` operator
	- To avoid memory completely, the normal `delete` operator should response to the placement version of `new` as well
- Be aware of name scope in C++, inner scope will always hide the name from the outer scope
	- They need to be made available manually if a placement `new` is declared within the scope
	- For any `new` operator made available, a `delele` operator needs to be implemented too
	- Solution: the base class should contain all the normal form of `new` + ` delete`



