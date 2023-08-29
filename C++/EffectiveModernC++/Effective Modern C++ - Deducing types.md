# Effective Modern C++ - Deducing types

- Modern C++ extends on the deducing types functionality to 2 new keywords
	- `auto` + `decltype`
	- Makes the mode more adaptable, changing type once, apply everywhere
	- Downside: Makes code harder to read, the type of the member might be unclear

### Understand template type deduction

- Type deduction for template is the basis of `auto`

	- However, seems less intuitive than how its applied in template

- (Function) Template type deduction are performed during compilation

	- Function arguments are used to deduce 2 types, `typename` type + `parameter` type
		- Frequently different due to adornment (const, reference, pointer, etc.)

- Type deduction for `typename T` not only depends on the type of argument + but the form of `parameter`

- 3 cases to consider for the following pseudocode

	- ```c++
		template <typename T>
		void f(ParamType param);
		f(expr); // use expr to deduce T + param
		```

##### `paramType` is a reference / pointer, but not a universal reference

- Universal reference: A R-value reference with type parameter, they can represent both L-value + R-value reference depends on the deduction of the type
- The simplest situation
	- If `expr` is a reference / pointer, ignore the reference / pointer part
	- Pattern match `expr` against `ParamType` use that to determine `T`
- The `const`ness of the object becomes part of the type `T` + `ParamType`
	- If the `ParamType` is already a `const` type, all `paramType` will be `const`
		- In this case, `const`ness no lo-nger have to be deduced from `T`, T can be regular type
- R-value reference work the same way as L-value reference, requires R-value reference parameter to be able to pass R-value reference

##### `ParamType` is a universal reference

- Parameters declare like R-value reference, behave differently when a L-value reference is passed in
	- If `expr` is an L-value, then both `T` + `ParamType` are L-value reference (The only situation where `T` is deduced to be reference)
	- If `expr` is an R-value, `T` is regular type + `ParamType` is R-value reference + modifiers

##### `ParamType` is neither a pointer / reference

- Pass-by-value, the passed in object will be a copy of the original object
- Ignore reference, `const` + `volatile` modifiers
	- Just because the original object cannot be modified doesn't mean the copied object cannot be modified either
	- `ParamType` + `T` most likely to be the same type
- For pointer of both top + low level const, only top level const will be ignored, result in a modifiable pointer to a const object
	- The `const`ness of the pointer is ignored, the property of the object stays

##### Array arguments

- Array-to-pointer rule can convert a const char array to a pointer (points to the first char in the array)
- Array cannot be used as function parameter, it will automatically converted to a pointer of the same type (but array + pointer types are not the same)
- So if pass by value, array will be converted to pointer and be deducted as such
- However, if pass the array by reference, type deduction will deduce the type as actual array (include the size of the array)
	- `T` is deduced as `type[array_Size]` + `ParamType` is deduced as `type(&)[array_Size]`
	- Declare reference to array enable template that can determine the # of element in an array

##### Function arguments

- Function types can decay into function pointers
	- Same rules as array -> pointers
		- If copy the function pointer by value, type will be deduced as pointer to function
		- If pass the function pointer by reference, type will be deduced as reference to function

### Understand `auto` type deduction

- `auto` type deduction is same as template type deduction (with 1 exception)

	- Algorithmic transformation between the 2 type reduction

- `auto` essentially plays the role of `T` in function template type deduction

	- The type specifiers for the variable plays the role of `ParamType`

-  Compiler pretends each `auto` declaration as a function template declaration + call to the template with initializing expression

	- ```c++
		auto x = 27;
		template <typename T>
		void func_for_x(T param);
		func_for_x(27); // deduce the type of x
		
		const auto cx = x;
		template <typename T>
		void funct_for_cx(T param)
		func_for_cx(x); // deduce the type of cx
		```

- Same as function template type deduction, there are 3 cases of type specifier

	- All 3 cases work the same way as function template
	- Array + function pointers work the same as function template

- The subtle difference between function template type deduction vs. `auto` type deduction: `std::initializer_list` + braced initializer ({})

	- 4 ways to declare an int variable with an initial value

		- ```c++
			int x1 = 27;
			int x2(27);
			int x3{27};
			int x4 = {27};
			// All valid form of declaring an int variable
			// The int variables can be replaced with `auto` keyword
			```

	- Declarations have different meanings

		- If the `auto` variable is declared with braced initialize, `auto` will deduced the type as `std::initializer-list<int>` instead
			- Contain single element with value of 27

	- When the initializer for auto-declared variable is wrapped in braces ({}), the deduced type are always `std::initializer-list<type>`

		- If such type cannot be deduced, the code will throw error

	- A 2 steps deduction, `auto` first deduce the type to be `std::initializer-list<T>`, then the template class deduce the `T` type when instigate (template type deduction)

	- Deduced type of `auto` with braces are a instantiation of `std::initializer_list`

		- No apparent design advantage + obvious reason for this design decision

	- If the initializer is passed plainly to a template `T` param, the code will not compile

		- ```c++
			template <typename T>
			void f(T param);
			f({11,23}); // code will not compile, cannot deduce the type T
			```

	- However, if param is specified as a `std::initializer_list<T>`, then the code will run fine

		- ```c++
			template <typename T>
			void f(std::initializer_List<T> param);
			f({11,23}); // The code will run fine, T deduced as int
			```

- A common mistake in modern C++, declare an variable as `std::initializer_list<type>` instead of the actual variable type

	- Good practice to only put braces around initializers when needed

- In C++ 14, return type + lambda parameter can use `auto` for compiler to determine the type of the variable, however, they use template type deduction instead of `auto` deduction


### Understand `decltype`

- Given a name / expression, `decltype` returns its type

	- Usually correct, occasionally provides unexpected result

- Used to declare function template return type depends on the parameter type

	- i.e., return elements of the container depends on the type of variable the container stores

	- Trailing return type syntax should be used

		- ```c++
			template <typename Container, typename index>
			auto access(Conatiner &c, index i) -> decltype (c[i]) // auto indicates trailing return type is being used 
			// Return type declared after the "->" (Using decltype)
			{
			    validContainer();
			    return c[i];
			}
			```

		- In C++ 14, the trailing return type can be omitted, `auto` can automatically figure out the return type (Type deduction will still take place)

	- However, the type deduction by compiler will ignore the reference-ness of the type, function will return plain type, a R-value that cannot be assigned to

		- Solution: Use `decltype` to deduced the exact type

			- ```c++
				template <typename Container, typename index>
				decltype(auto) access(Conatiner &c, index i)
				{
				    return c[i];
				}
				```

			- `auto`: The type should be deduced

			- `decltype`: `decltype` rules should be used to deduce the type

- `decltype(auto)` can be used in initializing expression to retain the reference-ness normally removed by `auto`

- For a function to accept R-value parameter, use the universal reference

	- Depends on the functionality, it make sense for some function to accept R-value parameter

	- ```c++
		template <typename Container, typename index>
		decltype(auto) access(Conatiner&& c, index i)
		// Correct for C++ 14, for C++ 11, make sure to include the trailing return type mentioned earlier
		{
		    return std::forward<Container>(c)[i];// Since we are using universal reference, return type (as L-value / R-value) needs to be determined base on the function parameter
		}
		```

- Putting parentheses "()" can change the type that `decltype` reports

	- If L-value expression other than name has type T, `decltype` is going to return `T&`, since most L-value expression include an L-value reference qualifier
	- Beware of returning reference to local variable (deleted at the end of the scope)

- Overall, `decltype` is essentially correct when applied to name, should be used worry free

### Know how to view deduced types

- 3 possible ways

##### IDE editors

- Most IDE when hovering over the name would show its type
	- The IDE running a complier in the background, compiling the code to determine the type
	- Might not be helpful with complicated types

##### Compiler diagnostics

- Compiler error message usually include the type causing the compilation problem
	- i.e., Using a template class (Declared but not defined, no way to instantiate), and pass the variable we would like to check type would generate messages that contains the variable's type

##### Runtime output

- Type information can be outputted by the program during run-time

	- A common approach: use `typeid()` to get the `std::type_info` object, use the `name` member of the object to determine the type
		- Helpfulness varies depending on the compiler

- Result of `std::type_info::name` are not reliable (Required to be incorrect)

	- Treats the type as pass-by-value parameter in a template function
		- `const` ignored, reference-ness omitted, etc.

- Result from the IDE editor is also not reliable

- For reliable result, use boost TypeIndex library, being cross-platform + open source, just as good as relying on the standard library

	- ```c++
		#include <boost/type_index.hpp>
		...
		using std::cout;
		using std::endl;
		using boost::typeindex::type_id_with_cvr;
		...
		cout << type_id_with_cvr<T>().pretty_name() << endl;
		```

		