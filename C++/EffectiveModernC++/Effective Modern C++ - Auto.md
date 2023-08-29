# Effective Modern C++ - Auto

### Prefer `auto` to explicit type declarations

- `auto` makes code easier to read + constraint the coder to follow standards

	- `auto` deduced initializer's type, variables must be initialized

- Lambda expression can use `auto` to deduce the type of the expression

	- In C++ 14, the Lambda parameter can also be `auto`, allow compiler to deduce the type

		- ```c++
			auto SomeCompFunction = [](const auto & p1, const auto& p2)
			{return *p1 < *p2};
			```

		- Side note: the captured variable (variables in the"[]") are data members for the lambda expression, they are local to the scope of the lambda definition, used to control the execution flow of the lambda expression.

- `std::function` object: A template in C++ 11, a generalized function pointer

	- Can be used to refer to any callable object (Function, lambda, classes with `()` operator overloaded, function pointer, etc.)

	- The specific type of function is required to create a `std::function` object

		- ```c++
			std::function<bool(const std::unique_ptr<Widget>&) func; // Creating a std::function that return a bool and takes a single const unique pointer points to a widget object called func
			```

- Using `std::function` is not the same as using `auto`

	- `auto` declared variables has the exact same type being deduced, store the variable as the type
	- `std::function` holds an instigation of `std::function`, fixed size given any signature
		- If the closure is too big, might allocate heap memory to store (Could yield out-of-memory exception)
		- Generally use more memory than `auto` + slower than `auto`
	- `auto` should always be preferred over `std::function`

- `auto` can help to avoid `type shortcuts`

	- i.e. Declare size of vector as an unsigned integer when size is actually a special type (`size_type`)
	- Could cause compatibility issue with 64-bit operating system
		- Unsigned is 32-bits + size_type is 64-bits in 64 bits OS
	- Using `auto` ensures the correct type is being used (`size_type`)
	- i.e. A more critical example, mismatch types could result in compiler creating a temporary object to match the user declared type
		- Could result in reference bind to local, soon to be deleted variables
	- Explicitly specifying types -> implicit conversions -> unexpected result

- The flaws of `auto`: The readability of the source code

	- Explicit type declarations provide better readability
	- However, type interface is common in many other language as result of the popularity of dynamically typed language (variable rarely explicitly typed)
	- Explicitly specifying type could also introduce subtle, hard to spot errors
	- Also make design decision easier to implement
		- Instead of finding every explicitly type declarations, variables type can be changed automatically

### Use the explicitly type initializer idiom when `auto` deduces undesired types

- Proxy class: Class for emulating + augmenting behavior of some other type

	- i.e. `std::vector<bool>`, `std::shared_ptr`, `std::unique_ptr`
	- A standard design pattern utilization
	- Some proxy class are visible, some are less well known + could cause compilation issues

- Many operations in C++ requires the conversion form a proxy class to another type through implicit conversion (Constructor / assignment operator overload)

	- Auto will often deduce the proxy type instead of the type the client might want
		- i.e. `std::vector<bool>[]` will return a `std::vector<bool>::reference` type instead, which can be implicit converted to bool
		- `std::vector<bool>::reference` essentially is a pointer to an address of the temporary `std::vector<bool>` object, when the bool vector gets destroyed at the end of the statement, the pointer becomes a dangling pointer, cause errors

- Invisible proxy classes does not mix well with `auto`

	- Hard to spot, requires deep understanding of the library + implementation

		- Usually discovered when compilation problem + incorrect unit test result shows up

	- Solution: force a different type of deduction for `auto`, use explicitly type initializer idiom

		- ```c++
			auto SomeVar = static_cast<bool>(features(w)[5]); // In this case, features return a std::vector<bool> and the [] operator converts is into a std::vector<bool>::reference
			// Use the static cast to explicitly cast the result fo a bool for auto to deduce
			```

		- In this case, the pointer is still valid, dereference and converted to a bool object, can be stored by `someVar`

- Explicitly type initializer idiom is also useful for creating a variable that different from the generated initializing expression

	- ```c++
		double SomeFuncForDouble();
		float ep = SomeFuncForDouble(); // implicitly convert the result of function to a float
		// using explicitly type initializer idiom
		auto ep = static_cast<float> SomeFuncForDouble();
		```

	- Also useful to cast an float / double variable to integer using `static_cast<int>`