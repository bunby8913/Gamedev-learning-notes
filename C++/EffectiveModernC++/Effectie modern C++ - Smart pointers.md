# Effective modern C++ - Smart pointers

- Why raw pointer are bad
	1. Cannot tell if the pointer points to object / array
	2. When deleting a pointer, cannot tell if we should delete as object / array
	3. Does not know if the memory address has 1 or multiple pointers points to it
	4. Does not know if `delete` or alternative deleting methos should be used
	5. Cannot delete the pointer more than once / does not delete the pointer (undefined behavior / memory leak)
	6. Does not know the content the pointer points to still exist
- Should always use smart pointers > raw pointers
- 4 types of smart pointer in C++: `std::auto_ptr`, `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`
	- `auto_ptr` is result of earlier version of C++ and should not be used unless necessary (lead to surprising code + usage restriction)
		- `unique_ptr` is the better version of `auto_ptr`

### Use `std::unique_ptr` for exclusive-ownership resource management

- Same size as raw pointer, support most operation of raw pointers

- Each `std::unique_ptr` owns the object it points to

	- Ownership can only be transfer, but not copied (Move only type)

- delete a `std::unique_ptr` by apply `delete` to the raw pointer in it

	- Alternative, the pointer underneath is deleted when `std::unique_ptr` is destroyed

- Commonly used in the factory design pattern

	- Allocate an object on the heap + return the pointers to it, the client is responsible to manage the resource
	- A common scenario would to have the `std::unique_ptr` returned to a container (component) that is a data member of an object, when the object gets deleted, the destructor of the `std::unique_prt` will be called, safely deallocate resources

- Able to pass in and use custom `delete` function

	- Passed in as the second type argument for `std::unique_ptr`, able to determine the type of the function using `decltype`
	- In C++14, the delete function can be a part of the factory function that construct the `std::unique_ptr` (using return type deduction)

- Cannot directly assign a raw pointer to `std::unique_ptr`, implicit conversion of such are problematic + prevented by the compiler

- Use `std::forwarad` to forward arguments from the calling function to the constructor

- Preferably, the custom `delete` function should be written in Lambda to reduce performance penalty

	- Depends on the # of local variable + states, using functions could have significant size penalty

- `std::unique_ptr<T>`/`std::unique_ptr<T[]>` could be a pointer to object / pointer to array, each use different signature

	- APIs for each are designed to match the form
		- i.e. Cannot use indexing operator on object pointer + cannot dereference array unique pointer

- Easy to convert `std::unique_ptr` to `std::shared_ptr`

	- ```c++
		std::shared_ptr<MyClass> mc = MakeMyClass(arguments); // Assume that function Make my class returns a std::unique_ptr
		```

- Makes `std:;unique_ptr` the best candidate to be returned by factory function

### Use `std::shared_ptr` for shared-ownership resource management

- `std::shared_ptr`: best of both world, able to automatic garbage collection + applied to all resources + predictable timing

	- Managed through shared ownership, when the last `std::shared_ptr` stop pointing at the object, the pointer destroys the object + reclaim the memory
		- Other garbage collection language are conditional driven, where `std::shared_ptr` are event driven, provide programmer with more fine control when the object will be deleted + memory deallocated

- Resource's reference count: How `std::shared_ptr` know if it's the last pointer to the object

	- Constructor increment count (with exception of move construction), destructor decrement count, copy constructor increment + decrement (Increment to the pointer being assigned to, decrement to the pointer being assigned)
	- Performance implication
		- Twice the size as raw pointer (Store a raw pointer to the reference count) (A control block to be more specific)
		- The reference count is independent from object being pointed to, reference count are stored as dynamically allocated data
		- Increment + decrement reference count needs to be thread safe (use atomic)
			- Slower than non-atomic actions

- `std::shared_ptr` move operation

	- Move constructing a `std::shared_ptr` sets the source to point to null, reference count does not change
		- Therefore, more efficient to move than to copy (also avoids the costly atomic increment + decrement actions)

- Custom deleters

	- Uses `delete` as the default resources-destruction method, but also support custom deleters

	- With `std::shared_ptr`, deleters does not have to be part of the pointer

		- Much more flexible than unique pointer

		- Is possible to create `std::shared_ptr` of the same type that uses different delete operation + place them in the same container

			- ```c++
				std::vector<std::shared_ptr<MyClass>> MCContainer {mc1, mc2};
				```

	- Custom deleters does not take additional space on the despite the size of the deleters function, function reference managed by the control block

- `std::shared_ptr`'s control block

	- A data structure that are dynamically allocated, contains reference count, weak pointer count, custom deleter, allocator, etc.
	- Rules to create control block
		- Always creates a control block with `std::make_shared` making a new object for the pointer to points to, a new control block is necessary
		- Always creates a control block when `std::shared_ptr` is constructed from unique ownership pointer (`std::unique_ptr`)
			- Unique pointer does not use control block + shared pointer will take over ownership
		- Always creates a control block when constructing `std::shared_ptr` with raw pointer
			- IF using `std::shared_ptr` + `std::weak_ptr`  as argument, will not create a new control block, use the control block that already exist (increment reference count)
	- Consequence: Constructing multiple `std::shared_ptr` from a single raw pointer will result in undefined behavior, the raw pointer will have multiple control block
		- Eventually will lead to the same pointed to object being deleted multiple times -> undefined behavior
	- Lesson: should always use `std::make_shared` if possible, if passing custom deleters, should pass the result of `new` directly to `std::shared_ptr` constructor (do not save any raw pointer variables)
		- Prevent using the same raw pointer multiple times, using shared pointer's copy constructor instead
			- Especially be aware of the `this` keyword, its a raw pointer + should not be used to construct shared pointer
				- Solution: `std::enable_shared_from_this`: A base class template to be inherited, used to manage `std::shared_ptr` to safely make shared pointer form the `this` pointer
					- Using the The curiously recurring template pattern (CRTP)
					- Offers a `shared_from_this` member function, should be used to create a shared pointer pointing to `this`
					- Note: this requires a control block associated with `this` already exist, otherwise it will also produce undefined behavior
				- The solution should be hidden away from client, only available through factory function
	- Control block can get quite big with deleters, allocators, virtual function + atomics
		- Not suppose to be the best solution to every resource management problem
		- But for basic scenario (Default deleters, no allocators), control are relatively small, dereferencing shard pointer is as fast as raw pointer + reference count increment / decrement are optimized at machine code level
		- The benefit definitely outweigh the cost
		- However, if `std::unique_ptr` will do, it should always be preferred (easy to convert to `std::shared_ptr` down the line anyway)
			- Cannot convert `std::shared_ptr` back to unique pointer

- `std::sharded_ptr` does not work with array, designed only to operate on single object

	- Should not try to implement one either, no index operator (require awkward expression instead)
	- Creates issue with type matching between single object + array shared pointer

### Use `std::weak_ptr` for `std::shared_ptr` like pointers that can dangle

- `std::weak_ptr`: A smart pointer that can track if it dangles

	- A pointer pointing to a shared ownership object without affect object's reference count
	- Cannot be dereferenced, cannot test for nullness, more like a extension of `std::shared_ptr`
		- Even created from `std::shared_ptr`

- Is rather difficult to use to check if the object has "expired" + to access the object through `std::weak_ptr` if its not expired

	- Have to careful with multi-thread race condition from separating check + dereferencing

		- Solution: An atomic operation to perform both operations at once

	- 2 ways to create `std::shared_ptr` from `std::weak_ptr` to access the object it points to (Depends on what happens if `weak_ptr` is expired when creating the shared pointer)

		- `weak_ptr::lock`

			- ```c++
				std::shared_ptr<MyClass> smc1 = wmc1.lock();
				auto smc2 = wmc2.lock(); // Same as above
				```

			- If `std::weak_ptr` is expired, the shared pointer will be set to `null`

		- Using weak pointer to construct shared pointer

			- ```c++
				std::shared_ptr<MyClass> smc1(wmc1);
				```

			- Will throw exception if `std::weak_ptr` has expired

- Using `std::weak_ptr` in caching result of the factory function

	- When the factory function is too expensive to construct the `shared_ptr` from source information (file read /  Request over internet) + the same parameter could be used multiple time

		- Use a static `std::werk_ptr` cache to store pointers that has been created

	- The cache pointer should be `std::weak_ptr` able to detect if the pointer is dangling + without affecting the reference count of the `shared_ptr` pointing to the object

	- ```c++
		stataic std::unordered_map<ClassID, std::weak_ptr<const MyClass> cache;
		auto objPtr = cache[id].lock();
		if (!objPtr)
		{
		    // If not in cache, load + add the pointer to cache
		}
		```

		

- Using `std::weak_ptr` to implement observer design pattern

	- Object being observed should store a reference to the observer in order to pass changes of state + events to the observer
	- Hold the observer with `std::weak_ptr`, can check if the observer will exist before using it to pass events

- Using `std::weak_ptr` to hold reference to the pointer of the owner/ parent

	- `std::weak_ptr` should be used to hold reference (soft reference) to parent if necessary
		- Using `std::sharde_ptr` will create circular dependencies, both parent + children will never be automatically deallocated
			- Child should never have longer lifetime than parent, childing won't dereferencing dangling parent pointer

- `std::weak_ptr` takes same amount of resources as `std::shared_ptr`

	- The same size in memory
	- Both uses a control block + reference
		- `std::weak_ptr` will not increase reference count of the `std::shared_ptr`, but it has a separate reference count used by itself

### Prefer `std::make_unique` + `std::make_shared` to direct use of `new`

- `std::make_unique` is not available until C++ 14, but pretty easy to implement

	- ```c++
		template <typename T, typename ...Ts> // This should not be implemetned within the std name space, could clash with the compiler version in the future
		std::unique_ptr<T> make_unique(TS&... params)
		{
		    return std::unique_ptr<T>(new T(std::forward(params...)); // construct a unique smart pointer using the raw pointer passed in
		}
		```

- Make functions: functions that forward arguments to constructor to dynamically allocate object, return a smart pointer to the object

	- `std::allocate_shared`: Same as `std::make_shared`, but first argument is an allocator to be used for memory allocation

- Using make functions could reduce code duplication

	- without the make function, the type name will be repeated twice in the code
	- prone to error

- Using make function can help with exception safety

	- A single line of source code can be converted into multiple lines of machine code, but compiler are not required to execute generated code in order

	- If any code runs between allocating memory for object + assign object to a smart pointer raise an exception, could cause memory leak (The exception cannot deallocate the memory already been assigned)

		- ```c++
			/* 
			When the function computePriority will run is not determinstic, if it runs between creating the object on the heap + assign the object to a smart pointer and raise an exception, resource leak
			*/
			ProcessMyClass(std::shared_ptr<MyClass>(new myClass), computePriority());
			```

		- The assigned memory never gets managed

	- Solution: Using `std::make_shared`, combine creating + assigning object to smart pointer in single operation, exception from the other function won't affect the pointer

		- same with `std::make_unique`

- using `std::make_shared` improves efficiency, generate smaller code + faster code

	- Using `new` would cause 2 memory allocation, for the pointed object + the control block
	- Using `std::make_shared` combines them into 1
	- Reduce overall memory footprint, faster allocation time + reduce the static size of the program
	- The same can be applied to `std::allocated_shared`

- Circumstance where make function should not be used

	- Make function does not allow custom deleters to be added, can only be done with smart pointer constructor
	- Make function uses parentheses when perfect forwarding parameters to the pointer constructor
		- Will have to use `new` directly to construct pointed to object using braced initializer
		- `std::forward` does not support passing braced initializer
			- A workaround will be introduced later

- Circumstance unique to `std::shared_ptr`'s make function

	- `std:make_shared` does not work well with custom new + delete operator
		- Make function will use the global new function + allocate memory for the control block
	- `std::shared_ptr`'s control block also keep track of how many weak pointer references it
		- The control block will only be deallocated if both shared reference count + weak reference count are both 0
		- Make function combines pointed to object + control block into one (have to be deallocated together)
			- Even if the pointed to object is no longer needed, the program still have to wait until all weak reference has been removed too
	- Directly using `new` creates a separation between control block + the pointed to object
		- Pointed to object can be deallocated as soon shared reference count goes to 0

- Tricks to keep `std::shared_ptr` exception safe using `new` 

	- Creates the `std::shared_ptr` in a separate statement, then apply the shared pointer to any functions that may raise an exception

	- Additional note: Creating raw pointer on the spot will yield a R-value, perfect for move operation for optimization (Can avoid the atomic increment / decrement of the reference counter), but putting creation of the shared pointer on a separate line yields a L-value

		- Solution: use `std::move()` to convert the L-value to R-value so move operation can be applied (Both more efficient + exception safe)

	- ```c++
		std::shared_ptr<MyClass> smc (new MyClass, customDeleter);
		ProcessMyClass(std::move(smc), computePriority()); // convret the smart pointer to R-value to use move operation for optimization
		```

### When using the Pimpl Idiom, define special member function in the implementation file

- Pimpl Idiom (Pointer to implementation): replace data members with a separate class, use a pointer pointing to the implementation class for access

	- Reduce the class compilation time by hiding details of the implementation class

- Incomplete type: A type that has been declared, but not defined

	- Usually introduced by forward declaration
	- Only has limited functionality
		- Declare pointer / reference / smart pointer to it
		- Declare function that accept the class as parameter / return value

- Dynamically allocate + deallocate the object holds the data member in the source file

	- Moved dependencies to be only visible + used by implementation

- In modern C++, smart pointer + make smart pointer should be used instead

	- In theory, no class destructor required since smart pointer will delete itself automatically

	- However, the code might not compile due to issues with destructor

		- In particular, when the destructor is not declared, compiler will auto generate code for the destructor
		- Destructor will call `delete` function on the raw pointer in the `std::unique_ptr` using `static_assert` that raw pointer is not pointing to a incomplete type
			- If the compiler generate destructor code in header file, then the raw pointer will be a incomplete type

	- Solution: Change when the code for destructor is generated (Inside the source file instead of header file)

		- ```c++
			class MyClass
			{
				public:
			    MyClass();
			    ~MyClass();
			    private:
			    struct Impl;
			    std::unique_ptr<Impl> pimpl;
			}
			// in source file after the definition of Impl
			MyClass::~MyClass() // Defined destructor, now destructor have access to the definition of Impl + no longer a incomplete type
			{}
			// Alternatively
			MyClass::~MyClass() = default;
			```

- Pimpl move operation support

	- Since we declared destructor, the compiler will not generate move operation automatically

	- Be aware the same issue with destructor, implementation of the move assignment operator + move constructor should be done after the definition of implementation class / struct is clear (In the source file)

		- ```c++
			// In the header file
			MyClass(MyClass&& rhs);
			MyClass& MyClass::operator=(MyClass &&rhs);
			
			// In the source file
			MyClass::MyClass(MyClass &&rhs) = default;
			MyClass& MyClass::operator=(MyClass &&rhs) = default;
			```

- Pimpl copy operation support

	- Copy operations will not be automatically generated

		- `std::unique_ptr` is a mote-only type + should implement custom copy logic

	- Also needs to add definition of the copy operations after the implementation class details is clear (in Source file)

	- The data struct / class will have automatically generated copy constructor, to copy each elements individually

		- ```c++
			// in source file
			MyClass::MyClass(const MyClass& rhs):Pimpl(std::make_unique<Impl>(*rhs.pImpl)); // assign the current unique pointer with the copied pointer
			MyClass& MyClass::operator=(const MyClass& rhs)
			{
			    *pImpl = *rhs.pImpl;
			    return *this;
			}
			```

- `std::unique_ptr` + `std::shared_ptr` supports custom deleters differently

	- To make `std::unique_ptr` space efficient, deleter is part of the smart pointer, allow for smaller data structure + faster code
		- But when compiler generate special function, the pointer must points to a type-complete type
	- `std::shared_ptr` uses a a separate, independent deleters, pointed object does not needs to be complete when the complier generate special function





