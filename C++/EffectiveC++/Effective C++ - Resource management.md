# Effective C++ - Resource management

- Memory leak: When a segment of memory has been allocated, but never deallocated
- There are many other resources that require management

### Use objects to manage resources

- With manual memory management, we cannot guarantee the code to deallocate memory will run
  - Control might skip the code / a exception raised before the deallocation
  - Bad practice to rely on the calling function being responsible to deallocated used memory
- Solution: wrap the dynamic allocated object in an object, the destructor will automatically clean up the allocated memory
- Use smart pointer to points allocate resource on the heap + release them when controls leave the block + functions
	- Used to be `auto_ptr`, replaced with `unique_ptr` + `shared_ptr`
	- The resource should be created + immediately assigned to the smart pointer, Resource acquisition is initialization (RAII)
	- Resource-managing object uses destructor to release any allocated resource
	- Multiple `shared_ptr` can be assign to the same object, the object memory will only be released when the last pointer stop pointing at the location
		- A reference counting smart pointer (RCSP)
	- Trying to copy a `unique_ptr` will result in compiler error, only move operation is allowed
- Should avoid using smart pointer with C-style array
- One should never release resource manually, if smart pointer does not behave properly, craft a custom resource managing classes

#### Think carefully about copying behavior in resource-managing classes

- Not all resources are heap based, smart pointer is not suitable for all situations
- What should RAII object do when copied?
	- Deny the copy operation: If copying the object makes no sense, the copy operation should be prohibited, `delete` the copy construction
		- i.e. Lock object
	- Increase the reference counter for the resources: Implemented with `shared_ptr`
		- Allow specification for the "deleter", determine the operation when the counter goes to 0
		- The deleter function must take the same argument as the type pointed by the pointer
	- Copy the underlying resource: Perform a "deep copy", copy the resource + the resource managing object
	- Transfer ownership of the resources: Operation should be performed with `std::move` 
- Unless value copy is required, one should always define custom copy constructor + copy assignment operator

### Provide access to raw resources in resource-managing classes

- Resource-managing classes are powerful and should be as much as possible
	- However, in real life situations is often required to bypass resource managing object + deal with raw resources (i.e. APIs)
- Often requires to convert resource managed object into raw resource, with 2 approaches
	- Explicit conversion: Smart pointers has a `get()` member function, gets the raw pointer (stored pointer) in the smart pointer object
		- Usually the preferred way, minimizes the chances of unintended type conversion
	- Implicit conversion: Smart pointer has overloaded the dereferencing operator, allow implicit conversion to the raw pointer
		- Access the value of the object directly either through pointer / dereferenced pointer's object
		- Implicit conversion makes the program easy + natural to read + handle
			- However, could run into issues with accidental implicit conversion, could result in unexpected behavior
- Interface should be easy to use + hard to use incorrectly
- Encapsulation does not mean hide all the data, just hide the stuff the client don't needs to see

### Use the same form in corresponding uses of `new` + `delete`

- The # of object in the allocated memory determines # of destructors to be called
	- Memory layout for single object is different from layout for arrays
		- The `delete` function could use the size of array to determine how many destructor to run
	- `delete[]` should be used to remove array from memory, `delete` assume pointer only points to a single object
- Use the array deleter (`delete[]`) on a single object is undefined
	- Wrong type of destructor + wrong # of destructors will run
- Use the single object deleter (`delete`) on a pointer to array of object is also un-defined
	- Too few destructor will be called
- Square bracket should match in `new` + `delete`
	- Important when the object has multiple constructors to use the same form of `new` to initialize pointers
- `typedef` author needs to specify which type of `deleter` to use `typedef` type object
	- Should always avoid using array with `typedef`

### Store `new`ed objects ins smart pointers in standalone statements

- If the same statement needs to execute multiple things, the order of the execution of each things are random
	- In some case (especially with explicit conversion), some other action might happen between creating a new dynamic object + dynamic object assigned to a smart pointer
		- The code in between could raise an exception, leave the scope before assigning the allocated memory to a smart pointer, creating a memory leak
- Therefore, the RAII should always happen in a standalone statement
	- Pass the created smart object after

