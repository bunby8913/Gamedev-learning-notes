# Effective Modern C++ - Moving to Modern C++

### Distinguish between `()` + `{}` when creating objects

- Usually 3 ways to initialize value

	- ```c++
		int x(0);
		int x = 0; // This is not assignment
		int x {0};
		```

- uniform initialization: A single initialization syntax can be used anywhere + express anything

	- Based on braces, braced initialization

	- ```c++
		std::vecto<int> vec{1,3,5}; // Initialize a container with multiple values
		```

	- Can perform non-static data member default initialization

		- ```c++
			class SomeClass {
			    ...
			    private:
			    	int x{0}; // int y = 0; is also valid
			    	int z(0); // error, not valid
			}
			```

			 

	- Un-copyable object can be initialized with braces + parentheses, but not `=`

- Braced initialization prohibits implicit narrowing conversion

	- i.e. Cannot convert double -> int
	- Possible with parentheses + `=`

- Braced initialization can avoid most vexing parse: Anything can be parsed as a declaration must interpreted as one

	- Accidently declaring a function instead of default construct an object
		- Calling constructor with 0 argument will declare a function instead

- Drawback of the braced initialization: unexpected behavior

	- Compiler might mix up the braced initialization with `std::initializer_lists`

	- When using `auto`-declared variable with braced initialization, type deduced will always be `std::initializer_lists`

	- In constructor, if a constructor takes `std::initializer_lists`, then any initialization with braced initialization will use the that constructors

		- Even able to hijack move + copy constructors

	- Only if there are no ways to convert argument type to `std::initializer_lists` will alternative constructor be used

	- Edge case: Empty braced initialization means no argument, not a `std::initializer_listst` with no elements

		- To construct an object with an empty `std::initializer_lists`

			- ```c++
				SomeClass sc({});
				SomeClass sc2{{}};
				```

- e.g. `std::vector` has constructors for non-`std::initializer_list` + `std::initializer_list`

	- Declaring a 2 elements vector using braces vs. parentheses makes a huge different
		- Braces(`std::vector<int> v1{x,y})`: Using the `std::initializer_list` version of the constructor, creates 2 elements
		- Parentheses (`std::vecror<int> v2(x,y)`: using the non-`std::initializer_list` version of the constructor, creates `x` number of elements, all have the value `y`

- When designing a class, should avoid overload constructor with `std::initializer_lists` parameter

	- Client should not have to worry about using parentheses / braces when constructing
	- Add new overladed constructor with `std::initializer_lists` with great cautions

- Important to choose a default way to initialize value (parentheses / braces)

	- Only use the other one if necessary
	- Both side has its benefit + limitation

- Hard for template to determine which type of initialization to use

	- Precise documentation of design decision + clear communication to the user required

### Prefer `nullptr` to 0 + NULL

- Ordinarily, 0 is a integer, if used in place as pointer, the compiler will translate it to a `nullptr`
	- Same goes with NULL, not a pointer type, but can converted to a null pointers
		- Causes conflict between the apparent meaning vs. the actual meaning of the code
	- Prior to modern C++, it's forbidden to overload with pointer + integral types
- Solution: Use `nullptr` : not a integral types, a pointer of all types
	- `nullptr` is a `std::nullptr_t`, which can be converted to all raw pointer type
	- `f(nullptr)` calls for `f(*void)`
- `nullptr` improves code clarity
	- Indicate we are dealing with pointers, not integral types
- `nullptr` is especially helpful in templates
	- If using `0` / NULL, template will be deduce them as integral, when in reality, a pointer type is needed

### Prefer alias declarations to `typedef`s

- `typedef` is very useful when we don't want to write long types multiple times
	- In C++ 11, functionality replaced with alias declarations (`using`)

- Differences: alias declarations can be templatized, `typedef`s cannot
	- `typedef` relies on creating a struct nested inside the template
		- if `typedef` uses the type specified by the template parameter, is defining a dependent type nested within the template, `typename` keyword is necessary if this types wants to be used
			- The definition of the `typedef`'d name dependent on the type of template parameter, therefore, `typename` must be used to indicate to compiler that it should be treated as type

		- Using type aliasing, the name is an alias template, must be a type, therefore, is a non-dependent type, does not require  `typename`

- Historically in TMP, to change the type of the template type parameter, a special `<type_traits>` should be used to apply transformation to the type
	- Uses type synonym for historical reason in C++11
	- In C++14, it has been replaced with alias template


### Prefer scope `enum` to un-scoped `enum`s

- Normally in C++, Name declared within curly braces stays within the curly braces

	- Limited visibility

	- However, not the case with `enum`, `enum` are un-scoped

		- ```c++
			enum Color {black, white red};
			```

			

- C++11 introduce the `enum class`, scoped `enum`, name does not leak

	- ```c++
		enum class Color {black, white, red};
		```

		

	- Enumerators are also much strongly typed, no implicit conversions between scoped `enum` to any other type

		- Explicit cast must be used to convert type

- Both scoped + un-scoped `enum` can be forward declared (un-scope `enum` will require extra works)

	- Forward declarations helps to solve the issue with dependencies, especially when things that are included by majority part of the system and changes have to be made to it constantly

	- By default, un-scoped cannot be declared, but only defined, so the `enum` content can be known for storage type optimization

	- When forward declare an un-scoped `enum`, declare the underlying type, result may be forward-declared

		- ```c++
			enum Color: std::uint8_t;
			```

	- forward declare scoped `enum` is much easier

		- ```c++
			enum class Color: std::uint32_t;
			```

		- By default, scoped `enum` uses `int` as underlying type

- In some situation, un-scoped `enum` might be preferred

	- When we want to use `enum` to represent relationship between index and the type of data + when the function requires a implicit conversion between `enum` type to other data type (i.e. `size_t`)
		- To achieve the same with scoped `enum`, we will have to statically cast the `enum` to appropriate type (no implicit conversion available)
			- However, it does avoid name space pollution + unexpected conversion of enumerators

### Prefer deleted functions to private undefined ones

- Tricky to prevent client from calling function that are declared by the compiler

	- Common for C++ to synthesizes a series of functions what's needed

- Old C++ approach: Declare function as `priavte`, do not define them

- Modern C++ approach: Use `= delete` to mark them as deleted functions

	- Prevents member + friend functions to access deleted function
	- Deleted function should be all `public`, C++ requires to check accessibility of the function before the deleted status
		- Some compiler will complain that the function cannot be found as `private` member function
		- Making legacy code function generally improve error messages

- Any functions may be `deleted`, but only member function can be `private` (prevent access)

	- Particular useful when we wants to prevent implicit conversion of parameter of functions
		- Solution: Overload the function, delete any functions with parameter types that are not needed

- Deleted function could be used to prevent template instantiations that should be disabled

	- When we want to prevent the template to instantiate for any particular type

		- ```c++
			// Example of prevent insttiate the template class with a void* pointer
			template<>
			void ProcessPointer<void>(void*) = delete;
			void ProcessPointer<const void>(const void*) = delete;
			```

	- The same cannot be achieved with moving certain instantiation to `private` access level

		- Cannot provide different access level of function template specialization from main template
			- Will not compile, template specialization are on namespace scope, not class scope

### Declare overriding functions `override`

- OOP fundamental concept: Declare + implement basic functionality in virtual function in the base class, implemented override functionality in derived class
	- Overriding makes it possible to call derived class function using base class interface
- Requirements of overriding
	- Base function needs to be virtual
	- Both function (base + derived) needs to have the same name + parameter list + same constness + same return type (or at least compatible)
	- (Modern requirement): Function's reference qualifiers must be the same
	- Most requirements error cannot be spotted by the compiler, they will still execute as valid code, but not as overrided function
- Requirements of overriding is hard to keep track + important to get right
	- Solution: Use the explicit keyword `override` after each override function, force the compiler to catch all the overriding-related problem
	- Note: The keyword `virtual` in front of the override function is not necessary, but does not improve readability + clarity
- Using `override` is also a good indication if signature change in the base class is worth the pain
	- `override` error message from derived class can report how many override functions also needs to change the call signature
- `override` is a contextual keywords, depends on where it is in the source code, it has different meaning
	- Declare the function a `override` function from the base class only when its' placed at the end of the member function declarations
		- Its original meaning unaffected
- Side note: Functions that accept parameter of the same name, different L-value / R-value reference parameter are considered different function
	- Member function reference qualifier make treading L-value vs. R-value differently

### Prefer `const_iretator` > `iterator`

- `const_iterator`: A pointer to const, low-level constness, the value being pointed to cannot be changed
	- Although existed in old C++, was not fully implemented
		- `const_iterator` either has to be created by a normal iterator / reference-to-const (Requires extra steps)
		- Old C++ does not support using `const_iterator` directly, require cast it back to normal iterator, which are not supported by many cast methods
			- Not practical to use overall
	- Now easy to get + use, even used by some internal STL member function
- To write generalized code to be used as library, used non-member `cbegin` + `cend`
	- Some container does not have those function
	- Or we can always write custom `cbegin` + `cend` template function
		- Add constness to the container + using the standard `begin` function to get a iterator of a constant container = a const iterator

### Declare functions `noexcept` if they won't emit exceptions

- Old school C++ does not have good support for exceptions
  - Each function needs to specify exception manually
  	- Change of function might require change of exception specification (which might break client code)
  	- The compiler are not helpful in spotting + dealing with exception issues
- C++ 11 overhauled the exception design, now provide a clear indication if a function might throw exception or not
  - Using the unconditional `noexcept` to guarantee the function won't throw any exceptions
- Using `noexcept` indicate important information to the client + could help the compiler generate better object code
  - Using `throw()` will guarantee throwing unwound the caller stack, in attempt to solve the exception at the caller level, but using `noexcept`, the call stack will only possibly unwound before program termination
  	- Does not have to use the stack to destroy constructed object in inverse order since the program will be terminated anyway
  	- `throw()` invokes `std::unexpected`, `noexcept` invokes `std::terminate

- `noexcept` is especially useful for `move`, `swap` and many other container operations

	- Useful when we want to take advantage of move semantics but also ensure strong exception safety guarantee

		- Replace the use of copy operation with move operation if the move operation is declared as `noexcept`

	- using `noexcept` with `swap` function is also desirable

		- The exception state of the `swap` function depends on the state of the user-defined swap operation

			- ```c++
				// Compiler implementation of the swap function
				template <class T, size_t N>
				void swap(T (&a)[N], T(&b)[N]) noexcept(noexcept(swap(*a, *b))); // use the user defined swap's exceptation settings
				```

		- Conditionally no-except: The `noexcept` depends on the expression inside the `noexcept` parameter

			- If swap of the object is `noexcept`, then swap containers with those type of elements will also be `noexcept`

- Functions should be `noexcept` only if they can be implemented as `noexcept` over long term

	- Removing `noexcept` risk breaking client code

- Most functions are exception-neutral

	- The function might not throw any exception, but its function call might
		- They should not be `noexcept`, they needs to pass the exception from the function they called to higher level

- Is essential for some function to be `noexcept`

	- Swap + move operation mentioned earlier
	- Destructor should never throw exception

- Wide contracts vs. narrow contracts

	- Wide contracts: no preconditions, called regardless the state of the program
		- If for sure the function won't emit exception, setting it as `noexcept`
	- Narrow contracts: have pre-conditions, if violated, result are undefined
		- Should avoid from using `noexcept`, exception are useful in terminate if pre-conditions has been met

### Use `constexpr` whenever possible

- Different meaning depends on what it applied to
	- `constexpr` Object: A beefed-up `const`, value of the object known at compilation
		- Value stored in read-only memory, useful for embedded system
		- Useful when the integral constant expression are required (i.e. array size, integral template arguments, enum, etc.)
	- `constexpr` functions: Produce compile-time constant when all parameter are compile-time constant, act like normal function if passes normal parameters
		- The functions will return `constexpr` type if all of its parameter are known as compile time, if not, the code will reject
			- Result can be assigned to a `constexpr` object
		- When the function has parameter not known as compile-time, act like a normal function, return normal type
- Useful when we needs to use `std::array` as the data structure for storage + wants to allocate enough space base on variables known before compile time
	- Create a function that can both used during compile-time-computing + called during run-time
- `constexpr` function has different limitation between C++11 + C++14
	- In C++11, only a single return statement is allowed
		- However, functionality can be expanded using conditional `?` operator + using recursion
		- `constexpr` member are implicitly `const` + `void` return type
			- Impossible to have `setter` function to be `constexpr`
	- In C++14, all limitation removed
- `constexpr` functions can only take + return literal types, types known at compile-time
	- All built-in types + any user-defined type that has `constexpr` constructor + member functions
	- This allows values to be calculated through constructor + getter functions and store the result in read-only memory during compile-time
		- The more code / calculation can be done during compilation, the faster than code will run (The slower the compilation process will be)
- However, needs to be aware that if `constexpr` if removed for whatever reason, large portion of client code might stop working / broken

### Make `const` member functions thread safe

- Using `mutable` data members allows the member to be modified even when the object is `const`

	- Is useful when we want to modify the object without affecting how the object operates
		- Used to cache calculated values to be re-used

- However, when multiple threads calls the same function to calculate + return the result from a `const` object, multiple threads are reading + writing from the same memory location (data race)

	- Undefined behavior

- Root of the problem: Object are declares as `const` but not thread safe

	- Easiest solution: Using `mutex`

		- ```c++
			std::lock_guard<std::mutex> g(m); // Lock mutex at the start of the function
			
			private:
				mutable std::mutex m; // Declare the mutex object mutable
			```

			- Side effect: `mutex` is a move-only type (cannot be copied), this will in extension make the `const` object not copiable, only movable too

	- Alternative solution when `mutex` is an overkill: `std::atomic`

		- ```c++
			++callCount; // Increase the atomic counter
			
			private:
				mutable std::atomic<unsigned> callCount {0}; // Initialize the call count atomic objec to 0
			```

			- Same side effect: make the object deployed `std::atomic` movable only
			- More side effect when multiple variables requires synchronization
				- It can either cause other thread to read pre-calculated result / perform multiple calculation even though a thread is already working on it (Reduce efficiency significantly)

	- Tip: For single variable synchronization, use `std::atomic`, for multiple, use `mutex` to avoid side effects

- If we can guarantee the code will only execute in single-thread environment, thread safety is immaterial + can avoid the cost of thread-safety measures (rare)

### Understand special member function generation

- C++ had 4 special functions that will be synthesized/generated if not created by the user

	- Constructor, Destructor, Copy assignment operator + copy constructor
	- Generated only if needed
	- Implicitly public + `inline`
	- All non-virtual except if the parent class has a virtual destructor, then the generated destructor will also be `virtual`

- In C++11, 2 new special member function has been added

	- Move constructor + move assignment operator

		- ```c++
			MyClass(MyClass&& rhs); // move constructor
			MyClass& operator=(MyClass&& rhs); // move assignment operator 
			```

	- Generated only if needed

	- Perform member wise moves on all non-static member of the class

	- Move construct each non-static member + move assign each member with parameter

		- Also operates on the base class parts

	- No guarantee that move operation will take place (more like move request)

		- If move operation is not supported, types will be moved with copy operation
		- Move data member if it support move operation, use copy operation if not

- Copy operations are independent, declaring 1 (copy constructor / copy operation) does not stop the compiler from generating the other (if missing)

- Move operation are not independent, Declaring 1 of them requires us to declare the other (move constructor + move operation)

- If copy operations has been declared, move operation will not be generated

	- Custom copy operation indicate the normal approach is to copy the object

- If move operation has been declared, copy operation will not be generated

- The rule of 3: If declaring 1 of copy constructor, copy assignment operator / destructor, then all 3 of them should be declared

	- All 3 special functions indicate special resource management is required for the class
		- Destructor: Ensure resources gets properly released
		- Copy constructor: Ensure that resources gets correctly duplicated
		- Copy assignment operator: Ensure existing resources are released so new resources can be acquired by the copying object
	- An important that was not fully appreciated back in the day, and now enforcing the rule will likely break too much legacy code
		- An important reason why compiler do not generate code for move operation if destructor has been declared

- C++ is shifting to deprecates automatically generated copy operation if destructor has been declared

	- So maintain legacy code, Declare copy operations, and assign them to default

		- ```c++
			MyClass(const MyClass&) = default; // default copy constructor behavior
			MyClass& operator=(const MyClass&) = default; // default copy assignment behavior
			```

	- Since declared copy operation blocks generation of move operation, move operation should also be explicitly declared with `default

		- ```c++
			MyClass(MyClass&&) = default; // default move constructor
			MyClass& operator=(MyClass&&) = default; // default move assignment operator
			```

- Declare special function is good practice for ore clarity

	- Could prevent many issues in the long run, i.e. when we want to use the destructor by declaring it ourself, we won't break the code since move + copy operation won't be generate anymore

- Using member template function for copy + move operations are fine, but could have important consequence

