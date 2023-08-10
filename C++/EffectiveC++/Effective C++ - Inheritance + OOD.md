# Effective C++ - Inheritance + OOD

### Make sure public inheritance model "is-a"

- Public inheritance means "is-a"
- The derived class is a more specialized concept of the base class (With public inheritance)
	- Anywhere the base class is needed, it can be replaced by the derived class
		- Not vice versa
	- If an argument requires a base class object, it can be filled with a derived object of the base class
- Public inheritance requires careful consideration, often require precise categorization + depends on the current + future expected function of the system
	- Alternative: Override functions that are not suitable in the derived class with error messages
		- Not a constraint enforced by compiler, rather a run-time logic
		- Not recommended, good interface should prevent bad code, not able to call the function at all is the better choice

### Avoiding hiding inherited names

- Names in the inner scope hides the name in the outer scope

  - ```c++
  	int x;
  	void SomeFunc()
  	{
  	    doudle x;
  	    std::cout << x << endl; // the double x will be used
  	}
  	```

- If name lookup couldn't found the name in inner scope, then look in the outer + global scope

- Scope of derived class is within the base class

  - If the base class overloaded function (have multiple function with the same name but different parameter), then if derived have a function with the same name, all base class functions are no longer visible in derived class

    - Even with different parameter types + virtual / non-virtual function

  - Prevent accidental inheriting overloads, however, is a violation of the "is-a" relationship between hierarchy

  - Solution: Using the `using` declarations, manually add base class functions to the derived class's scope

    - ```c++
      using Base::mf1;
      ```

  - Solution: If the derived class only wants to use a specific version of the function, use a `inline` forwarding function

    - ```c++
      virtual void mf1()
      {Base::mf1();}
      ```

### Differentiate between inheritance of interface + inheritance of implementation

- Public inheritance = inheritance of function interface + inheritance of function implementation (Function declaration vs. function definition)
- For derived class to only inherit function interface, use a pure virtual function
	- For functions that are used by all derived class + have no default implementation
	- Side note: Is possible for pure virtual function to have a default implementation + to use it, call the function with the base class scope operator
- For derived class to inherit interface + default implementation, but may wants to change the implementation in derived class
	- Dangerous to specify function interface + implementation with simple virtual function
		- An derived class might require a different implementation to the virtual function might forget to implement + will use the version of function not suited for it
		- Solution: Separate the function of simple virtual function to a normal function (to implement default behavior) + pure virtual function (to allow derived object to implement their own logic)
			- Default behavior is implemented as `non-virtual` function since it implementation should not be re-defined
		- Alternative solution (avoid namespace pollution): Taking advantage that pure virtual function can still have default implementation
			- For any derived class that wants to use the default behavior, use the base class scope operator to access
			- Easy for derived class to implement new behavior too
			- However, can no longer different access specifier to the function
- For derived class to inherit interface + mandatory implementation, use `non-virtual` function
- The 2 common mistakes to avoid
	- Should not declare all function as `non-virtual`: Limit the customizability of the derived class, any class that are intended to be used as base class should contain virtual class
		- 80-20 rule: 80% of the runtime will be spent executing 20% of the code, majority of the virtual function call will not have a performance impact
	- Should not declare all member functions `virtual`: Some function should not be re-defined + define the function as `non-virtual` shows that, better readability + reduce unnecessary variant in the derived class

### Consider alternatives to virtual functions

- Sometimes using virtual functions might be the most obvious approach, but may not be the best approach

##### The template method pattern via the non-virtual interface idiom

- Virtual function should always be `private`, called by a public, `non-virtual` member function
	- The non-virtual interface (NVI) idiom, template method (not associated with C++ template)
	- The normal member function is the wrapper for the virtual function
		- The wrapper function also handles preparation + clean up before + after the virtual function being called
- The derived class cannot directly access the base virtual function (as its private), but can still override the function + the public normal member function are inherited normally, will still able to access the derived class's virtual function implementation
- Base class non-virtual member function determines how the virtual function are being called (i.e. mutex, log entry, other pre/post function call work)
- If the base virtual function have logic the derived virtual function needs to use, the base virtual function needs to be `protected` / `public`

##### The strategy pattern via function pointers

- Separate the function from the class

	- Pass the function in as function pointer during the class construction

- Use `typedef` (alternatively `using` keyword) to define a function pointer in the following format

	- ```c++
		typedef Return_Type(*FPointer_Name)(Argument_list); // the argument list likely needs forward declartaion
		```

- Utilizing the strategy design pattern

	- Have a set of algorithms / functions that can be applied to multiple objects
	- This way, different instance of the same class can have different functions
	- Different functions can be switched to at run time

- Common issue with the strategy design pattern:

	- The function does not have special access to non-public members of the class
		- Can only access class's public interface
	- Common issue when member function is replaced with non-member functions
	- Solution: weaken class's encapsulation, declare the strategy class as friend to the utilizing class

- Tradeoff between per-object function implementation + access to non-public members vs. single implementation for multiple classes + change of implementation during run-time needs to be evaluated case-by-case

##### The strategy pattern via `std::function`

- Use `std::function` in replace of function pointer

- `std::function`: A general purpose, polymorphic function wrapper

	- Can hold function pointers, function object, lambda expression, bind expressions + pointers to member functions

	- ```c++
		type def std::function<ReturnType(param1, param2, ...)> FuncName; //Define FuncName as a type alias for a std::function with ReturnType as return type and takes a series of param
		```

		- A generalized function pointer type, can hold any function with a matching calling signature (The same return type + parameters) + or any function with calling types that can implicitly converted to the calling signature

- Provide significantly flexibility for client in what functions can be used

	- The client can use any functions as long its return value + parameters can be converted to the `std::function` calling signature

- `std::bind`: Produce new callable object by adding 1 or more arguments to the function / callable object

	- Adapt the function take more arguments

	- ```c++
		std::bind(&Gamelevel::health, currentLevel, _1); // The "_1" is a generic placeholder variable in C++
		```

##### The "classic" strategy pattern

- Solve the problem through design pattern
	- Making a separate calculation class with its own calculation hierarchy, add the calculation object as a component to the class
- The standard strategy pattern implementation

### Never re-define an inherited non-virtual function

- Non-virtual function are statically bound + virtual function are dynamically bound
	- Re-define non-virtual function could result in inconsistent behavior
		- The function being called is determined by the pointer type, not the type of the underlying object
- Is derived class have to change the implementation of a function in the base class, the function should be `virtual`
	- Hence the reason why destructor has to be `virtual`, allows different implementation to the same function

### Never re-define a function's inherited default parameter value

- Only 2 types of function can be inherited, virtual + non-virtual functions
	- Virtual function are dynamically bound, but the default parameter are statically bound
- Static type: The type of object described in the code
- Dynamic type: The type of object being referred (Usually the derived class as result of polymorphism)
	- Virtual function are dynamically bound, determined at run-time by the type of object which functions to call
- However, the default parameter of the virtual function are static bound to the base class, for run-time efficiency sake
- Both base class + derived class needs to have the same default parameter set
	- Change of base class default parameter requires manual changes of all the derived class's default parameter value
	- In that case, it will more efficient to use alternative to virtual function (NVI)
		- A public non-virtual member function calling a private virtual function that derived class can re-define

### Model "has-a" or "is-implemented-in-terms-of" through composition

- Composition: When an object of 1 type contains object of another type
- In many case, composition is preferred over inheritance
- Composition is dealing with both application domain (Representing real life object) + implementation domain (Represent software implementation artifacts)
	- has-a relationship vs. is-implemented-in-term-of relationship
- is implemented in term of relationship is particular useful when we want to use an object that contain majority of the properties and functionality we need, but is not a exact "is-a" relationship
	- Quick reminder, the "is-a" relationship means that everything true for the base class is true for the derived class

### Use `private` inheritance judiciously

- Private inheritance behavior
	- Compiler will not convert derived object to privately inherited base object
	- All data member private inherited from the base object will have the `private` access specifiers
- Private inheritance = is-implemented-in-term-of
	- to take advantage of the some of the features available in the base class, private inheritance does not indicate any hierarchy relationship (An implementation technique)
	- Only inherit the implementation + ignore the interface
- Use composition as much as possible + use private inheritance only when necessary
	- i.e. When the class provide functionality but requires override of some of the virtual function
		- This way, we an inherit the virtual function privately, implement custom functionality + use the function within the class
	- However, private inheritance is not necessary in this case, Possible to create a nested class, public inherit from the base class, add functionality to the virtual function + add the inherited class as composition
- Public inheritance + composition > private inheritance for 2 reasons
	- Derived class may re-defined virtual function even if they are private, however, the derived class can re-define the function we privately inherited
		- We can define the nested class's object to be private, deny any access from the derived class
		- Alternatively, use the `final` keyword offered after C++11
	- Use public inheritance + composition could minimize compilation dependencies
		- We can use forward declaration instead of including the file in header class for inheritance
- Space optimization of private inheritance over composition
	- When inheriting from class with no data, no virtual function + no virtual base classes (in other word, the object in theory will take 0 spaces)
		- In most C++ compiler, an empty object will still take some memory space, most do not allow zero-size freestanding object
		- If the object is included as composition to another object, it will take a non-zero amount of memory space
	- However, if we inherit from the "empty" class, the base class will not take any space (empty base optimization) (EBO)
		- This is only viable under single inheritance
- Private inheritance is most likely to deal with 2 classes not related to each other, but require access of protected member of the other class / re-define some virtual classes of the other

### Use multiple inheritance judiciously

- 2 school of thought on multiple inheritance
	- Multiple inheritance is better than single inheritance
	- Multiple inheritance is not worth the trouble
- Multiple inheritance creates ambiguity, able to inherit the same name from multiple base class
	- C++ will perform function lookup for overloaded function names first before determining if the found function has the correct availability
	- To resolve ambiguity, use scope operator to specify which base class the function is from
- Not Un-common for multiple inherited base class to have a common higher-level base class (A deadly MI diamond)
	- With MI, some data member will be inherited multiple times, unclear how many copies of the same data member should exist in the MI derived class
		- The compiler default to perform replication 
	- Solution: Use virtual public inheritance, ensure only 1 copy of each data member is inherited by the MI derived class
		- However, virtual inheritance comes with performance overhead + uses more storage then classes without virtual inheritance
		- Classes that derive from virtual base inheritance must take initialization responsibilities for the virtual base
- Virtual inheritance should be avoided if possible. If necessary to use virtual inheritance, avoid putting data in them
	- Other language has strict virtual base class restriction to prevent data being added to them
	- Abstract class cannot be instantiated, does not have a constructor