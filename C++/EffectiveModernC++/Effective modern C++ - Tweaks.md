# Effective modern C++ - Tweaks

### Consider pass by value for copyable parameters that are cheap to move + always copied

- Sometime for efficacy, we want functions to copy L-value + move R-value parameter
  - However, that would mean 2 function to declare + implement
  - Alternative solution: use universal reference with template functions
  	- Universal reference could also be hard to deal with
- Alternative approach: ditch the rule to avoid passing object of user-defined types by value
  - void code duplication, not using universal reference to create bloated header file
- In Modern C++ (11 and beyond), passing object by value are copy constructed for L-value, move constructed with R-value
- Compare the 3 different approach
  - Overloading: Pass by reference, no cost to copy / move construct temporary object
  - Using a universal reference: Pass by reference, no cost to copy / move construct temporary object, same overall cost as overloading
  - Passing by value: Copy construct (L-value) + move construct (R-value) + whatever operation in the object
  	- One extra move for both L / R-values
- Pass by value should only be considered as viable alternative in the following situation
	1. If the higher cost of using pass by value is acceptable in the application
		- Side note, it's possible for function that are passing by value calls a chain of other functions that are also passing by value, this can slow the performance down significantly
	2. The parameter should be copyable, must not be move-only type
		- If it's move-only, we no longer have to overload the function to provide support for L-value reference
		- Since passing by value would make an extra move operation, move-only type pass by value will be twice less efficient
	3. The parameter are cheap move, since an extra move operation will be added regardless of L-value / R-value reference being used
	4. The parameter are always copied, if there are no guarantee that the parameter will be used every time (maybe the function has some form pre-condition must be met before executing the rest of the instructions)
- Be aware of parameter copied using assignment
	- Usually will cause an extra pair of allocation + deallocation of memory
		- There are situations where de-allocation + re-allocation additional memory is also mandatory when passing by reference
- Passing by value could also encounter the slicing problem (originated in C++98)
	- With a base class + derived class, if passing by value pass the base class type, the derived class portion (additional functions + data member) will not be passed
- Overall, to avoid writing additional operation to take advantage of move semantics, for copyable, cheap to move type when slicing is not a concern, pass by value is a good alternative that is nearly as efficient as pass-by-reference

### Consider emplacement instead of insertion

- Insertion (`push_back`) in many cases are not as efficient as one might think
	- If there is a mismatch between type of argument + type of parameter, compiler will generate code to construct a temporary object for insertion
	- Huge room for efficiency improvement
- In total, insertion uses 2 constructors + 1 destructor
	1. If the type of argument + type of parameter mismatch, the parameter must be constructed from the argument
	2. Within the container, memory must be allocated + constructed for the object to be inserted
	3. Right after the insertion, the temporary object (Constructed form the arguments) will be destroyed, calling the destructor
- Should be possible to direct construct the object inside the container
	- Avoid construction + destruction of the temp object from argument
- Solution: Use `emplace_back`
	- Uses the argument passed, construct the object within the container without temporary object
		- Uses perfect forwarding (needs to avoid perfect forwarding's limitation)
		- Supported by all containers (some are achieved with different function name)
		- More flexible interface, constructor argument for objects to be inserted
	- In theory, `emplace_back` should always be faster than insertion, but not really the case in practice
- In the following situation, emplacement should outperform insertion
	- value added is constructed in containers, not assigned
		- When the object is assigned to an already occupied memory in container, emplacement is the same performance level as insertion
	- The argument type if different from the type held by the container
		- Emplacement is only faster when temporary object needs to be constructed + destructed
	- The container is unlikely to reject the new value as a duplicate
		- The new object should be added to the container regardless of duplication, if we are detecting duplication, performance penalty for insertion is not as pronounced
- 2 additional issue worth of considering
	- Resource management: When we are dealing with pointer + dynamically allocated memory
		- With custom deleter for smart pointer, the pointer must be created directly with `new`
		- Use insertion creates a temporary object referring to the pointer, and in the case of exception, the pointer won't be lost (avoid dangling pointer)
		- Using emplacement will delay the creation of the resource-managing object until it's constructed in the container's memory
	- Regular expression object + emplace could lead to the compiler having difficulty in detecting errors
		- Does not require implicit conversion between pointer to regular expression
		- Use null pointer to construct a regular expression will compile, but result in undefined behavior
	- Copy initializing is not allowed to use explicit constructors, but direct initializing is (The complier is allowed to use a single parameter to construct the object)
- When using emplacement function, we need to be extra careful with passing the correct arguments