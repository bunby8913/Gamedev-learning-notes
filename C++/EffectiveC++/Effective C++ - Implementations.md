# Effective C++ - Implementations

### Postpone variable definitions as long as possible

- Variable adds to the cost of construction + destruction, un-used variable should be avoided at all time
	- Variable should be declared + defined right before they are being used
	- To maximize the performance, variable should avoid default constructor + assign value on definition (i.e. Initialize + define using copy constructor)
- Avoid constructing + destructing unnecessary objects
- When in a loop, object should be initialized in/out the loop depends on the speed of constructor / destructor vs. assignment
	- If assignment cost < constructor + destructor, initialize variable out the loop
		- However, have variable initialization outside the loop break program comprehensibility
	- Otherwise, variable initialization should be inside the loop by default

### Minimize casting

- C++ compiler usually guarantee object safety
  - However, guarantee could be affected by casting (Should approach with care)
- C++ offer 2 old, C-style cast + 4 C++ style casts
  - Old C-style cast should be avoided + should only be used for explicit constructor to pass object to function (Can be replaced with `static_cast` anyway)
    - Harder to identify + more ambiguous
  - `const_cast<T>()`: Used to remove the const-ness off an object
  - `dynamic_cast<T>()`: Used to cast a base class object to a derived object
    - Could have significant run-time cost
  - `reinterpret_cast<T>()`: Low-level cast, implementation dependent, rarely used outside low-level code, i.e. convert pointer to int  
  - `static_cast<T>()`: Force implicit conversion, used to perform any reverse conversion
    - Can pretty much perform any kind of cast apart from `const_cast`, should only be used if the cast is certain to not cause any issues

- Often the pointer value of base class pointer to derived class object is not the same to derived class object

  - Especially true when the derived object inherit from multiple base class
  - Derived object can be represented by multiple base pointer object
  - However, the order + position of base object in derived object's memory location is not guarantee + should not make assumption of (compiler dependent)

- Casting might not always be necessary

  - i.e. When a derived, virtual function wants to invoke its base class function  before running the new code, it's not necessary to cast the current object to base object + call the function

  	- ```c++
  		public:
  			virtual void onResize(){
  		        static_cast<Window>(*this).onResize() // Incorect
  		    }
  		```

  		

  	- This will create a new temporary base object which calls the function

  	- Solution: Simply use the scope operator to locate the base class function to call

  		- ```c++
  			public:
  				virtual void onResize(){
  			        Window::onResize();
  			        // more codes
  			    }
  			```

- Dynamic cast implementation could be slow

  - When dynamically cast an object, every level of inheritance needs to be checked to ensure type safe + support dynamic linking

- `dynamic_cast` are used when a derived object is pointed/ referenced by base object + we want to perform derived object's operation on the object, there are 2 alternative solutions to avoid `dynamic_cast`

	- Use container to store pointer to derived object directly
		- Drawback: Might need a different container for every type of derived class
	- Declare any function used by the derived object as virtual function in the base class
		- Create a pure virtual for the derived class to implement

- Cascade `dynamic_cast` is a bad idea

	- Bulky + slow code, requires update every time the class changes
	- Should always be replaced with virtual function call

- Cast should be hidden behind interface, isolated AMAP

### Avoid returning "handles" to object internals

- The return of object internal "handles" -> bypass the `const`-ness of the object + able to modify the internal data member
	- data member encapsulation depends on the most accessible function return
	- A member function should never return pointer (handles) to less accessible members
- Simple solution: declare the return type to be `const`, eliminate write access to the return handle
- Should be aware of dangling returned handle
	- The object pointed by the handle could be deleted after the handles has been returned
	- Any function that returns internal handles could be dangerous + should be avoided (With some exception, i.e. STL element access)

### Strive for exception-safe code

- 2 requirements for exception safety

	- Leak no resources: Allocated resources needs to deallocated (i.e. Locked thread needs to be unlocked) + cannot be affected by exceptions
		- Solution: Using resource managing class, let the system automatically handle the unlock + release of resources
	- Don't allow data structure to be corrupted: If an object creation throws an exception, any its pointer will be pointing to deleted data + data should reflect the exceptions (i.e. counter should not increment)

- The 3 exception-safe function guarantees

	- Basic guarantee: Everything in the code remain valid when an exception occur
		- Result of the guarantee might not be predictable (From client's perspective)
	- Strong guarantee: The state of the program remain unchanged when exception occurs
		- State of the program only changes if the function completely succeed
		- Behavior is easier to predict, 
		- `nothrow` guarantee: The function will never throw a exception
			- All built-in operations has `nothrow` guarantee

- Exception-safe codes are determined by implementation, not declarations

	- ```c++
		int SomeFunction() noexcept; //declare a function to not throw exception, if exception occurs, exit the program
		```

	- the `noexcept` is more for communicating between programmers, does not guarantee the function to not throw any exceptions

- Offers the strongest exceptions-safe guarantee as possible

	- Easy to implement C part of the C++ with `nothrow` guarantee
		- But dynamically allocated memory are exception prone (`bad_alloc` exception)

- Some general design strategy for strong guarantee

	- Use object (smart pointer) to manage resources
	- Move the change of status of the object after the changes has been performed
	- Replace content of the pointer > delete + create a new pointer
	- Copy + swap: Copy the object to modify, apply modification, if modification is successful, swap the original element with the copied element (non-throw swap should be used)
		- If modification raised exception, original object will remain unchanged

- Pimpl Idiom: All the object's data is stored in a separate implementation object. The object keeps a pointer (Usually smart pointer) to the implementation object

	- The implementation object can be both `struct` / `class`, its access restriction can be controlled by the "real" object

- Side-effect of strong exception-safe: Hard to guarantee overall function exception safety, especially when the function invoke other function calls

	- If the invoked function call only offers basic guarantee, it's up to the calling function to implementation strong guarantee, catch any exceptions + re-store original state
	- When the function invoke multiple function calls, hard to determine the original state after each function calls, state of the program changes after every function call

- Basic guarantee does not have the same side effect + usually more efficient to run (Copy + modify object takes resources)

	- It's not practical to offer strong guarantee to every functions

- Unless there is a really good reason, a function should never have no exception guarantee at all

	- Could lead the program into un-predictable, un-reversable state
	- Any functions that calls the no exception guarantee cannot offer any safety guarantee

- No such thing as partially exception-safe system

	- Use object to manage resources to prevent resource leak
	- Determine which level of safety guarantee is appropriate + Document decision

### Understand the ins + outs of inlining

- `inline` can not only avoid the overhead of function calls, but enable compiler context-specific optimization to the code (Only available to long piece of code without function calls)
- `inline` function: Replace the function call with the code body
  - If `inline` is too big, Will increase the size of object code, take up more memory usage, lead to unnecessary, additional paging, reduce hit rate slow memory performance down
    - However, smaller `inline` code use less resources than function call, lead to smaller object code, increase memory hit rate
- Implicit + explicit `inline` function
  - Implicit: Define the function inside the class definition (header file)
  	- Usually member function
  - Explicit: Define functions with the `inline` keyword
- Function templates != `inline` functions
	- `inline` functions should be in header file, to replace function call during compile time (With some exception)
	- Template functions should also be in header file, so the compiler can know how to instantiate the function when it's used
	- Template functions should only be defined as `inline` after careful consideration, `inline` has many additional cost to the function
- Compiler have the choice to ignore the `inline` request (when function are too complicated)
	- Virtual function cannot be `inline`, compiler needs to determine which function to call at run time
- the `inline`-ness of a function depends on the compiler + computer hardware
	- Will warn the developer is `inline` is not being properly applied
- Usually bad practice to make constructor + destructor `inline`
	- Compiler will generate codes for creation + destruction of data member in construction + destruction
	- `inline` constructor + destructor can get bulky real fast
- To change `inline`-ness of functions in library is significantly more difficulty, often require re-compilation from the client
	- Easier to change a non-`inline` function to `inline`
- `inline` is hard to debug, cannot insert break point to codes that are not local
- Functions should not be `inline` unless functions are trivial + necessity

### Minimize compilation dependencies between files

- C++ class does not separate interface + implementation well
	- Class will require definition of all its data member during compilation
	- Can create a long chain of compilation dependencies, increase compile time drastically
- Forward declaration could omit the class definition, but does not solve the problem, compiler still need class definition to allocate enough space for the object
	- Solution: Pimpl Idiom (Pointer to implementation), Separate the interface + implementation of the interface into 2 different classes (True separations between interface + implementation)
- Replace dependencies on definition -> dependencies on declarations
	- Use pointer / reference to object > actual object
		- Reference / pointer can be created using only declaration, actual object requires class definition to construct
	- Use class declarations > class definition
		- Class declarations is sufficient to use the type in function
- By including the header file in source file + use forward declaration in header file, only the essential dependencies will be applied through headers
	- To use specific functions utilizing forward declared classes, include the definition for those classes in sources file as needed, Eliminate artificial client dependencies
- Separate header file for declarations + definitions
	- Less common in modern C++ development, compile time has improved significantly + the separation of multiple headers is too complex + prone to error
	- Any changes to one headers needs to be applied to the other header too
- Utilizing interface classes (Common in Java)
	- Specify interface for derived classes, only contain a set of pure virtual function
	- Use factory functions to construct derived classes, return smart pointer to dynamically created object
	- Implementation are achieved in the derived classes, inherit interface from the interface class + implement the virtual function in the interface
	- Additional memory cost + run time slow down

