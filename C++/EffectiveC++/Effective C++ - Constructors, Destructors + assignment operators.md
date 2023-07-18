# Effective C++ - Constructors, Destructors + assignment operators 

- Backbone of well-formed classes, huge unpleasant repercussion if getting them wrong

### Know what function C++ silently writes + calls

- Compiler will always synthesize constructor + copy constructor + copy assignment operator + destructor if not explicitly created
  - `public` + `inline` functions
- Default constructor + destructor: "behind the scene" code, invocation of constructor + destructors of base classes + non-static member
	- Destructor are non-virtual unless inherited from a base class with virtual destructors
- Copy constructor + copy assignment operator determine how data member are copied between source + target object
	- Copy constructor will use the copy constructor of each data member, synthesized if not found
	- By default, copy constructor can only perform copy by value, will not compile if reference / const needs to be copied
		- Must define custom copy assignment operator
	- If the base-class's copy assignment operator are private, the derived class cannot synthesized generate copy assignment operator
		- Derived class needs to invoke member function that it cannot access

### Explicitly disallow the use of compiler generated function you do not want

- Usually, if a function is not declared, it will not be called
	- However, with copy constructor + copy assignment operator, if not declared, the compiler will synthesized one
- The traditional approach is to set the copy constructor + copy assignment operator as `private` + declare them without define
	- By declaring copy constructor + copy assignment operator as private in base class, any derived class can no longer access them (Will throw compiler error if attempt so)
- However, the modern approach is to declare the function + set them to be `deleted`

### Declare destructors virtual in polymorphic base classes

- Factory function: A function that return a base class pointer to a derived class object

	- Pointer will be dynamically allocated, will needs to be removed to avoid memory leak
	- Or just use smart pointer

- Factory function has a huge native risk if the base class has a non-virtual destructor

	- Un-defined behavior when deleting through base-class pointer with a non-virtual destructors
	- Only the base class portion of the object will be deleted, the derived portion will be memory leak
	- The derived class's destructor will not run
		- Partially destroyed object

- Easy solution by give the base class a virtual destructors

- Virtual class allow customization the implementation base on the derived class

	- Function with virtual function should always have virtual destructor

- If the class is not a base class, using virtual destructor is a bad practice

- Virtual function object contains a `vptr` (virtual table pointer) + `vtbl` (Virtual table), contains a table of function pointer, used to determine which function call to use

	- Virtual table + pointer will increase the size of the object by 50-100%

- Declare virtual destructor if only the class contain at least 1 virtual functions

- Note many built-in class does not have virtual destructor, should not be used as base class

	- i.e. `std::string`, all the STL container type

- C++ classes can use the `final` keyword to indicate that it cannot be derived from

- If a class needs to be abstract but does not have any natural pure virtual function, consider define the virtual destructor as pure virtual (Pure virtual destructor)

	- ```c++
		virtual ~SomeClass() = 0; // Pure virtual destructor
		```

	- Must provide definition in the abstract base class

		- Compiler will generate call to base class destructor from derived class destructors

- Only applies to polymorphic base class: Allow manipulation of derived class through base class interface

	- Manipulate derived class using only the base class pointer pointing to those objects

### Prevent exceptions from leaving destructors

- C++ allow destructors to throw exceptions, but it could potentially lead to memory leak
	- C++ does not destructor that throws exceptions, if C++ encounter more than 1 exceptions, its undefined behavior + program likely to terminate
- 2 ways to avoid destructors exception leaving the destructors
	1. Terminate the program: If the destructor encounters any issues, throw an exceptions
	2. Swallow the exception: Generally a bad idea, however, sometime it's the better option than premature termination / undefined behavior
		- Allow the program to continue executing + ignore the error
- A better approach by re-design the interface that handles the destructor with potential exceptions
	- Offer the opportunity (exposed function) for client to deal with exceptions themself
	- The destructor can determine if the exceptions has been solved, only attempt to handles the part that could raise exceptions if it's not dealt by the client (A backup call)
		- Deal with part with termination / swallowing
- Give the client opportunity to deal with any errors over impose burden on them
	- Client can always ignore the opportunity + let the system handle it

### Never call virtual function during construction / destruction

- Calls of virtual function in construction + destruction will not call the correct function (It will always call the base class virtual function)
  - Compiler will utilize the base class constructor to create the base class portion of the object, which will create a base object first, utilize the base class virtual function
  - Base class constructor runs before derived class constructor, derived class data member initialized after base object created
  	- Otherwise, will result in undefined behavior by calling members that doesn't exist yet
- The same applies in destruction, derived  class portion are destroyed first, leaving base class destructor behind
- Not always obvious to detect the use of virtual function on construction + destruction
  - Good practice to wrap the call to virtual function in a non-virtual member function, used in constructor
- One way to avoid the virtual call confusion is to change the virtual function into a non-virtual function + add the function to constructor
	- The constructor of the base object will pass any parameters to the function
	- Any derived class will member initialize base class's constructor, to pass any parameters to build the base portion of the object
	- The derived class constructor should use helper function to create the value to base class as "static" function
		- This way, helper is always available, before derived object is created

### Have assignment return a reference to *this

- Assignment are continuous + right associative (Right assign to left)
	- Assignment returns reference to the left hand side, same principle to implement assignment operators
		- Applies to all assignment operators
- Universal rule followed by everything in C++, should maintain the rule, should always return the pointer of the current object, the 1 being assigned, with `*this`

### Handle assignment to self in operator=

- pointer to the same thing -> aliasing: when more than variable (pointer / reference) refer to the same object
- Self assignment is an issue especially with aliased object
	- Hence why to use resource management object
	- Is possible for resource to be deleted while still needed
- The traditional solution is to check if both object are equal, if the same, do nothing
	- However, exception-unsafe
	- If exception occurred during the copying, portions of the object might not be constructed properly, leading to pointers to deleted object
		- Extremely unsafe
- A simpler, thread-safe solution is to order the code in a exception + self-assignment safe way
	- Points the left hand pointer to a copy of the right hand pointed object + delete the right hand pointer
- Fine balance between the cost of self-assignment test vs. identity test efficiency

- Copy + swap: Base on 2 facts
	- Copy assignment operator can take arguments by value
	- Passing value makes a copy
	- However, passing by reference is generally more efficient

### Copy all parts of an object

- Only 2 functions should copy object, copy assignment operator + copy constructor (Copying functionts)

- Compiler will not determine the correctness of implemented copy operations

- Compiler will not warn the user if a data member is not copied

  - When a new member is added to a class, need to make sure its copied to

  - Derived class need to make sure base class's data member is also copied in derived class' copy constructor, using base class' copy constructor as member initialization + Call base class' copy assignment operator

    - ```c++
    	VIP::VIP(const VIP &rhs): customer (rhs), priority(rhs.priority)// call the base class copy constructor
    	{}
    	VIP& VIP::operator=(const VIP& rhs)
    	{
    	    customer::operator=(rhs);
    	    priority = rhs.priority;
    	    return *this;
    	}
    	    
    	```

    - Otherwise, base class' data member will be default initialized

- Should always copy all local data member + call copying function from base class

- Should not try to use copy constructor to call copy assignment operator or vice versa

	- Copy constructor initialize a new object + copy assignment operator only applies to object that are initialized (Should not be mixed together)
	- If copy constructor + copy assignment operator uses similar code, put them into a member function, called by both copy function