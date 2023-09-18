# Effective modern C++ - R-value references, move semantics + perfect forwarding

### Understand `std::move` + `std::forward`

- Both functions doesn't do what they appear to do

	- More precisely, they are function that perform casts

		- `std::move`: Cast the object unconditionally to R-value reference, set up the object ready for move

			- Essentially a template function, takes T, remove any reference the type may have + create a R-value reference type, use that type to convert the object to its R-value reference

				- ```c++
					template <typename T>
					decltype(auto) move(T&& param)
					{
					    // Create a type of the R-value reference of the type T
					    using return_Type = remove_reference_t<T>&&;
					    return static_cast<return_Type>(param);
					}
					```

					

		- `std::forward`: Cast the object to R-value when the condition has been met, passes object to another function, retains the original L-value / R-value type

- However, using R-value does not guarantee for moving

	- i.e. If `std::move` is applied to a const type, the constness will remain
		- Moving object generally indicate change the in both source + target object, right hand object should not be `const`
		- In this case, usually, the const to L-value reference copy constructor would be used

- `std::forward` is similar to `std::move`, but will only cast the move if the argument was initialized with R-value

	- The type of reference is encoded in the parameter `T`, used by `std::forward` if cast is necessary

- In theory, `std::move` + `std::forward` are unnecessary, everything it does can be replaced by casting, but extremely bad idea

### Distinguish universal reference from R-value references

- `T&&` !== R-value reference, 2 different meanings

	- R-value reference, The reference can only strictly bind to R-value
	- Represent both L-value / R-value reference
		- Can bind to `const` + `volatile` object, almost any objects
		- More like a universal references (Forwarding reference)

- The context where universal references exist

	- Part of the template function parameter

		- ```c++
			template <typename T>
			void f(T&& param);
			```

	- When using `auto` declarations

		- ```c++
			auto&& v2 = v1;
			```

	- Essentially, when the type of reference are type deduced, the R-value reference into a universal reference

		- ```c++
			void f(MyClass&& param); // no type deduction, param is a R-value reference
			```

		- The type deduction must be in the exact same type as `T&&`, any other type deduction will result in R-value reference

			- Object qualifiers will also make the type strictly R-value reference

				- ```c++
					template <typename T>
					void f(const T&& param); // param is a R-value reference
					```

					

- The initializer of the universal reference determines the type of the reference it stores

- Depends on the set up of the function `T&&` does not necessarily indicate universal reference

	- If type deduction is performed elsewhere (with instantiation), then `T&&` will still only represent R-value reference

- `auto` variables to universal reference is less common in C++11, more common in C++14's lambda expression, declare `auto&&` parameters

### Use `std::move` on R-value references, `std::forward` on universal references

- R-value reference should only be bind to object that are eligible to move
  - Use `std::move` to cast parameter to R-values unconditionally

- Universal reference are bound to object that might be able to move
  - Should only cast to R-value reference if it's initialized with R-value (Using `std::forward`)

- Should not mix use `std::forward` with `std::move` or vice versa

	- Error-prone, creates unexpected result, hard to understand

	- Move operation might be performed at unexpected places, leaving the object with a unspecified value

	- Possible solution: Instead of passing in a universal reference, overload the function with `const` L-values + R-values alternatives

		- ```c++
			void SetName(T&& newName); // Using universal reference
			//Alternative
			void SetName(const std::string& newName); // const L-value reference overload
			void SetName(std::string&& newName); // R-value reference overload
			```

		- Many issues with this implementation

			- A lot of more code to write + maintain
			- A lot less efficient
				- With universal reference, the string literal can be directly assigned to `name`
				- With overloaded method, temporary object of `std::string` needs to be created + assigned to `name`
					- Will use constructor, object assignment operator + destructor for a single operation

				- Depends on the type, could have huge runtime cost

			- Really bad for scalability
				- The parameter the function have, the significantly more overloaded function required
				- Impossible with function that could take unlimited amount of parameters
					- Universal reference is the way to go


- If the parameter object needs to be used multiple times, the possible conversion to R-value reference should always be done at the end

	- This is to avoid the object being moved prematurely

	- If the object reference is needed before, we should always pass it as a L-value reference

		- ```c++
			template<typename T>
			void setObjText(T&& name)
			{
				obj.SetObjName(name); //pass L-value reference to any function, avoid moving
			    obj.addHIstory(now, std::forward<T>(text)); // only perform the operation that could move the object at the end   
			}
			```

- Function return by value of object bound to reference should always use `std::move` (for R-value reference) / `std::forward` (for universal reference)

	- The value of the object bound to the reference will have to be moved to the function caller's memory location + move operation is cheaper than copy operation + cheaper than constructing a new object on the spot

		- ```c++
			return std::move(lhs); // retunrs R-value for faster move opertaions
			return std::forward<T>(lhs);
			```

	- Even if the object does not support move operation yet, copy operation will be used instead (no performance penalty for using it)

		- And once move supports get added, code only requires a single compilation to support move operation with the return value

- Should be aware of overusing `std::move` by returning local variable with it (Return value optimization)

	- The compiler already have decent RVO in place
		- Performed when the local object has the same type as the return type + the local is being returned
			- Returning with `std::move` will return a reference to the local object, does not fit the rule
	- Can't really help the compiler

### Avoid overloading on universal references

- Using universal reference + `std::forward` to determine the type of reference is more efficient and could save on unnecessary copy operations
- Problem raises when we need to overload functions that tasks universal reference
	- If the argument parameter is not an exact match with the overloaded function (other than the universal reference), the universal reference function will always win
	- Functions with universal references parameter will exact match with almost all types of arguments, but there won't be suitable conversions between the type we pass + the type the function expect (Universal reference are extremely greedy)
- Should never overload functions that uses universal references
- Perfect forwarding constructor is even more risky
	- Similar to the problem with overloaded functions with universal references, any parameter that is not an exact match with the overloaded parameter will be a using the universal reference version instead
	- Additionally, C++ will generate copy + move constructors automatically, create even more problems
		- Instead of the copy constructor, the perfect forwarding constructor will be called
			- Constructor with universal reference will be a better match then copy constructor, program will attempt to use a constructed object to construct another object of the same type
			- Since the copy constructor requires adding `const` in front of the object, universal references is less resistance
				- If passing a `const` object instead, the copy constructor will be used instead of the constructor
	- It gets even worse with inheritance into play
		- Derived copy + move constructor will most likely call the base class constructor instead of base class copy + move constructor

### Familiarize yourself with alternatives to overloading on universal references

##### Abandon overloading

- Instead of using overloading, create new functions with different name + takes different parameters
	- Con: won't work for constructors, constructors names are fixed

##### Pass by const `T&`

- Replace pass by universal reference with pass by L-value reference to const
	- Not an efficient design + abandon modern C++ approach

##### Pass by value

- Replace pass-by-reference parameter with pass by value.
	- Good when you the object will be copied anyway

##### Use tag dispatch

- If perfect forwarding is necessary, then universal reference is necessary
- Solution: Add additional parameter to the functions + intentionally mismatch the parameter type for the universal reference when we want to use non-universal reference overloaded version

  - Use a single un-overloaded function to initiate the dispatch, which will construct the tag + call the correct version of the overloaded function

  - We can pass an additional argument indicate the type of parameter being passed in, use additional argument to determine if universal reference should be used or not

  	- Use `std::remove_reference` to remove any reference qualifiers from the type as the type check does not support reference

  		- ```c++
  			// Code example if we wnats to determine if the parameter is integral
  			template <typename T>
  			void LogAndAdd(T&& name)
  			{
  			    logAndAddImpl(std::forward<T>(name), std::is_integral<typename std::remove_reference<T>::type>());
  			}
  			```

  - Overload the function with 1 taking `std::true_type` + the other taking `std::false_type`

  	- Forces to use the version of overload we want

- Tag dispatch is essential to template metaprogramming

##### Constraining template that take universal references

- Tag dispatch requires a single un-overloaded function to initiate the process
  - Still does not solve the issue with constructors
  - Even if the constructor is using the tag dispatch system, the compiler generated function will not + still will take over controls

- Consider using `std::enable_if`

	- Force compiler to behave as if particular template didn't exist

	- Enable the template only if the `std::enable_if` conditions has been met

	- using `std::enable_if` has no effect on function implementation (remain the same)

		- ```c++
			template <typename T, typename = typename std::enable_if<condition>::type>
			explicit MyClass(T&& n);
			```

	- In order to compare types in conditions, we need to use `std::is_same` 

		- ```c++
			!std::is_same<Person, T>::value;
			```

		- In addition, we need to strip reference + const + volatile from the object to properly compare using `std::decay`

	- ```c++
		!std::is_same<Person, typename std::decay<T>::type>::value // remove any reference + qualifirers from T and compare is Person + T are the same type, return the value of the comparison
		```

- This should be the last resort when previous method cannot deal with the situation

- This still does not solve the issue for derived class's move + copy constructor calling for their ancestor

	- The universal reference will provide a better match (Derived class is not the same as base class)

		- Need to expand the `std::enable_if` conditions further to include matches with base type + any derived type (using `std::is_base_of`)

			- `std::is_base_of` can in theory replace `std::is_same` , since it will be true for the base class too

				- ```c++
					template <typename T, typename = typename std::enable_if<!std::is_base_of<MyClass, std::decay<T>::type>::value>::type
					```

					- In C++14, everything can be shortened further

- To put everything together in C++14

	- ```c++
		template <typename T, typename = std::enable_if_t<!std::is_base_of<MyClass, std::decay_t<T>>::value && !std;:is_integral<std::remove_reference_t<T>>::value>>
		```

- This template function will offer maximum efficiency

##### Trade-offs

- Specify the type of parameter vs. using perfect forwarding has significant consequences
	- Perfect forwarding is more efficient, avoid creating temporary object to match the type specific parameters
	- Perfect forwarding also has drawbacks, Some arguments cannot be perfect forwarded + but can be passed to function taking specific types
		- Also hard to detect type errors / invalid arguments since universal references accepts almost everything
			- Creates hard to understand complier error messages + hard to trace
			- Solution: use `static_assert` to verify the passed in arguments when we know the type required
				- However, the error will come after the universal reference mismatch error messages (could be 100+ lines long)

### Understand reference collapsing

- With universal reference, if L-value is passed as argument, T is L-value, if R-value is passed, T is the object without any references
- Reference to reference are illegal in C++ (compiler error)
	- When the universal reference takes a L-value reference parameter, the compiler reference collapsing kicks in
- 4 possible reference collapsing combinations (reference to a reference), all follow the same rule
	- If either reference is an L-value reference, then collapsed to L-value reference, otherwise, collapsed to R-value reference
	- Key part of how `std::forward` work
		- `std::forward` can use reference collapsing to determine if input is L-value reference / R-value reference + use `remove_reference` to make sure we have a L-value reference parameter, cast to parameter to that type using static cast
	- C++14 implementation will be concise
- 4 context where reference collapsing exist
	- Template instantiation
	- Type generation from `auto` keyword: Essentially the same to template instantiation
	- Generation + use `typedef` + `using` type aliases: When reference of reference is created, its automatically solved using reference collapsing
	- Used in `decltype`: If reference of references gets created, eliminate it using reference collapsing
- Universal reference is actually R-value reference with 2 conditions satisfied
	- Type deduction able to distinguish L-value + R-value, each deduce to L-value reference + object type
	- Reference collapsing can determine which type of reference the universal reference will become

### Assume that move operations are not present, not cheap and not used

- Move semantics is one of the premier feature of modern C++
	- Replace expensive copy operation with cheap move operation
	- However, its effect are generally exaggerated
- Many types fails to support move semantics
	- Without explicit declarations, compiler will only generate move operation if there are no existing copy operations, move operations + destructors
- Explicit move support may not be as effective as one imagines
	- Should not assume that moving all containers will be cheaper
		- Some container does not have have any cheap ways to move contents
		- Some cheap move operation of containers requires specific types of elements
	- i.e. When elements are stored as object within the container, move operation will have to manually move every elements in the container (linear time complexity)
	- i.e. `std::string` utilize SSO (Small string optimization), store strings with shorter length directly with the object (instead of pointer pointing to the address with the string)
		- When perform move operation with string stored with object, is the same efficiency as copy operation
- The move operations will only replace the copy operation if the operations is known to not throw an exception
- Should always assume that the move operation does not exist, is not faster + cannot be used until proven otherwise (Could create unstable code)

### Familiarize yourself with perfect forwarding failure cases

- Perfect forwarding is not "perfect" with all cases

	- Few epsilon needs to be aware of

- Forwarding: 1 functions passes parameter to another function

	- For functions to receive the exact same object
		- Cannot be used on function with value parameter + function with pointer parameter
		- Only possible with function with reference parameter
	- Passes the type of the reference (L-value / R-value) + object qualifiers (const / volatile)
		- The parameter will have to be a universal reference (to be able to take all form of references)

- There are several kinds of arguments that will not be perfectly forwarded to another function

	- Assume the following function structure for the following cases

		- ```c++
			// Template function that forward a single parameter
			template<typenatme T>
			void fwd(T&& param)
			{
			    f(std::forward<T>(param));
			}
			// Template function that forward any numbers of parameters
			template<typename... Ts>
			void fwd(Ts&&... params)
			{
			    f(std::forward<Ts>(param)...); // forward any number of parameter
			}
			```

##### Braced initializers

- Use of braced initializer is a perfect forwarding failure case

	- ```c++
		void f(const std::vector<int>& v);
		f({1,2,3}) != fwd({1,2,3}); // the fwd function will not compile
		```

	- Normally, compiler compare the arguments at the call site with the type of argument declared by the function

		- Perform implicit conversion to make the call success

	- With perfect forwarding, the first function deduce the type of the parameter + compare the argument in forwarding function withe the parameter declared in the second function

	- Perfect forwarding will fail in the following case

		- Compiler unable to deduce the type in the forwarding function
		- Compiler deduce the wrong type
			- the type is different from directly calling the function with the same parameter / the function will behave differently due to overloading the function

	- Passing braced initializer to function template will not be deduced to `std::initializer_list` , the wrong type is passed to the second arguments

	- Solution: use auto to store the braced initializers as `std::initializer_list`

		- ```c++
			auto li = {1,2,3}; // li will be deduced as std::initializer_list in this case
			fwd(li); // now it can be passed to the second function safely
			```

##### 0 or NULL as null pointers

- 0 and NULL when passed as parameter will be deduced as an integer (0), instead of a null pointer
- Solution: should always use `nullptr` over 0 and NULL when dealing with pointers

##### Declaration-only integral `static const` data members

- The compiler can use const propagation, which replace the static const variable with the value it represent
	- The `static const` data member has no memory address -> no definition required unless address for the data member is required (a pointer towards the data member)
- However, when passing `static const` data member to a forwarding function, the function takes universal reference (Pointer like), the object passed in requires de-referenced, but with const propagation, the `static const` data member does not have a memory address, will cause linker error
- Solution: define the `static const` data member to provide a memory address to it
	- Note: experiences may vary depends on the type of compiler + linker being used

##### Overloaded function names + template names

- When the second function can receive function pointers, to use the function passed in for additional processing + the function being passed in has been overloaded
	- However, perfect forwarding will only pass the function name, does not provide additional detail to determine which overloaded version to use
		- Same with passing a template function
- Solution: specify the needed signature for the function + static cast the function to the specific type of function with the return type + parameter list

##### Bitfields

- C++ does not allow pointer + reference to be bound a specific bit-field
	- Cannot directly address individual bits
- Bit field can only be passed by value / by reference-to-const
	- Reference-to-const convert the bitfield to value + bind to the object that stores the value
