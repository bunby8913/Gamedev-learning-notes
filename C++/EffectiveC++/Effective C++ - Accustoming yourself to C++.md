# Effective C++ - Accustoming yourself to C++

### C++ is a federation of languages

- Start as C with classes, evolved to include many more ideas, features + programming strategies
	- Became a multiparadigm programming language
- Should view C++ as a collection of language, and rules of C++ are applied within each sub-language
	- C: The basic syntax of the language is derived from C
	- Object-oriented C++: Classes, encapsulation, inheritance + polymorphism, virtual function, most OOP rules applies
	- Template C++: Generic + metaprogramming portion, rules are separate and does not interact with normal C++ programming
	- STL: template library, containers, iterators, algorithms + function objects. Has a set of special conventions needs to follow
- Strategies of coding changes depends on the sub-languages



#### Prefer `const`, `enums` + `inline` > `#define`

- Prefer compiler over pre-processor

- Pre-processer defined symbolic name may never been seen by compiler / enter the symbol table

  - Remove + replaced by the pre-processor before compiled
  	- Result in hard-to-decode error messages

- Using `const` variable could result in smaller code

  - pre-processer macro will replace every name to its represented value, result in multiple copies
  - `const` variable will always 1 copy

- To ensure the `const` object is limited to a class + only have a single copy, use the `static` keyword + add it as a member of the class

- When using a `const` pointer in C++ in replacing `#define` variables, needs to ensure both top + bottom level of `const`

- Exceptions for class-specific static `const` that is a integral type, can be declared without proving definition

  - The value of the variable must be declared at declaration (cannot be changed otherwise)
  - To reference the variable external / copy by reference, the variable require a separate definition
  	- If Defined as `inline` in the declaration, it can be declared + defined at the same time
  - If in-class initialization is not supported, initialize the value during definition

- `#define` does not respect scope, variable cannot be class-specific + cannot be invalidate until `#undef`

  - Cannot provide any encapsulation either (Being a private member of the class)

- The `enum` hack: Used when the value of a class is needed during compilation

  - i.e. For declaring array size during compilation

  - A fundamental technique in template metaprogramming

  - `enum` type can be used in replace integer

    - ```c++
    	enum {va1 = 5}; // the enum hack
    	```

  - `enum` address cannot be passed, like `#define` variables,  good way to avoid the reference of the variable being passed / accessed

  - Good compiler often replace the const integral type with its represented value during compilation (unless the object is referenced / pointed to)

    - Using `enum` can avoid unnecessary memory allocation

- Using `#define` to implement macros that looks like functions is extremely bad practice

	- Could lead to unexpected side-effect when calling as an expression
		- Increment could run multiple times / increment could run different amount of times then it appear to
		- Also usually a lot of parenthesized parameters to set the precedence right, often offload the burden to the user 

- Should always use a template with inline function instead

	- Access + scope rules can also be applied

- Other pre-processor is still important

	- `#include` to insert content of another to the current file
	- `#ifdef/#ifndef`: Compile code base on if a specific macros (preprocessor) is present or not



### Use "const" whenever possible

- `const` specify a compiler enforceable semantic constraint to compiler + user, the object should not be modified
- `const` can be applied to ordinary + static object at any scope
- Top-level vs. low-level `const`
  - Top-level `const` (Constant pointer): `char* const p`, const pointer, non-const data
  - Low-level `const` (Pointer to a constant): `const char* p`, const data, non-const pointer
  	- Low-level `const` could also be in the form of `char const* p`, they are equivalent
  - top + low level `const`: `const char* const p`: const data + const pointer
- STL's iterator is based on pointers, so top + low level of `const` also applies
  - A `const` iterator is a low-level `const`, cannot points to other object, but the can change the object's data
  - A `const_iterator` is a top-level `const`, cannot change the pointed object's data, but can points to other object
- Using `const` in function declarations, `const` can be applied to return value, parameters or assign the entire member function to be `const`
  - Return `const` return value is generally inappropriate, however, can reduce client-side error rate
    - By assigning a return value as `const` it ensures values can't be assigned to the return value

- Well-designed user-defined type should behave similar to built-in type to avoid potential confusion / misuse

- Functions parameter should always be const unless it needs to be modified / local object

- `const` member functions identify which function should be used on a `const` object

	- The `const` keyword is added at the end of the function declaration (after the parameter list)
	- Make class interface easier to understand, a good indication of the purpose of the function (modify the object or not)
	- `const` object can only use `const` functions
		- Side note, `non-const` object can use both `non-const` + `const` functions
		- C++ prefers pass object by reference to const
			- Pass object as reference to improve performance, pass object const as `const` object to ensure safety (ensure the object state remain unchanged)

- Functions can be overloaded if they are differ in `constness` 

	- Overloaded functions can handle `const`  + `non-const` object differently
	- Usually the compiler determine which version of the function to call base on the variable type
	- When pass by pointer / reference to const, the compiler will automatically use the `const` version of the function

- C++ function return object by value, returns R-value, immutable therefore, it's illegal to modify return value of a function if it's a built-in type

	- Only object returned by reference are modifiable

- Bit-wise constness vs. logical constness

	- Bit-wise: If the function does not modify any object's data member

		- Easy to detect violation
		- The definition used by C++

	- However, `const` function can still modify pointer referenced object even on a const object, and the compiler will not raise error

	- In another case, the function could be returning a reference to a pointer / reference with top-level const, which follows bit-wise const rules, but reference can be used to modify the data within after

	- Logical constness: Const member function can modify come portion of the object without the user detecting them

		- The object should appear const, but internal function should be able to modify

		- The object that are internally editable should be labelled with the `mutable` keyword

			- ```c++
				class Some_Class {
				    private:
				    	mutable std::size_t textLength;
				    	mutable bool lengthIsValid;
				};
				std::size_t getLength() const
				{
				    if (!lengthIsValid)
				    {
				        textLength = std::strlen(someString);
				        lengthValid = true;
				    }
				    return textLength;
				}
				```

- To avoid code duplication between const + non-const version of the overloaded function, the `non-const` version of the function can call the `const` internally

	- Note: Generally casting is a bad idea, This should only be performed if both function does the exact same thing

		- Cast away the const return value is safe, since only `non-const` object can call the `non-const` function

	- Requires 2 casts

		- First needs to cast away the `const` on the return value from the `const` version of the function using `const_cast`

		- Then, needs to convert the calling object to `const` to use the `const` version of the function using `static_cast`

			- Otherwise the function would enter infinite incursion within the function

		- ```c++
			return const_cast<char&>(static_cast<const TextBlock&>(*this)[position]);
			```

- The opposite (calling the `non-const` version of the function to use in the `const` version) should be strictly avoided

	- The `non-const` overloaded function are intended to break logical constness



### Make sure that objects are initialized before they're used

- Reading uninitialized values yields undefined behavior

	- In some case the program will die, in other, random values are being read, lead to hard to debug errors

- Guaranteed object initialization does not always happens

	- In most case, the C portion that requires run-time cost on initialization will not guarantee initialization but some new C++ members will

- The easiest solution is always to initialize object before usage

	- Manually with built-in objects
	- With the rest, make sure the constructor initialize every member of the object

- Initialization vs. assignment: initialization are called ahead of assignment, using default constructor, before the constructor body of the current object are called

	- In C++, member initialization list should be preferred for assignment (Better efficiency)

		- ```c++
			ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones): theName(name), theAddress(address),
			thePhones(phones),
			numTimesConsulted(0)
			```

		- This way, the value are passed to constructor (copy-constructed), instead of default construction + value assignment

			- Single call of constructor < default constructor + copy assignment operator

	- Member initialization can still call for default constructor on the member, by leaving parentheses empty

		- Better for consistency to have all member initialized using initialization list
		- Const member + reference also can only be initialized using initialization list

- When data needs to be looked up for assignment / too many data member needs to be initialized + assignment, the initialization + assignment is preferred

- In C++, base class initialized before derived class, within a class, the declared order determines the order of initialization

	- Although not required, the initialization list should be in the same order as the declaration

- Order of initialization of non-local static object

	- Static object exist from construction to the end of the program

	- Any static object that are not part of a function is non-local static object

	- Translation unit: Single source file with all the included files

	- If 1 non-local static object from 1 translation unit uses another non-local static from elsewhere, then the other static object might not be initialized, relative order of initialization of non-local static object in different translation is undefined

	- It's impossible to determine order of initialization non-local static object

		- However, the order problem can be solved by replacing non-local static object with local static object

			- ```c++
				inline FileSystem& tfs() // only 2 lines of code, great candidate for inline
				{
				    static FileSystem fs; //Replace the non-local static object
				    return fs; // return the reference to the local static object
				}
				```

				

		- Local static object are guaranteed to be initialized during first encounter of a call to that function

		- The local static object is only created if used

	- Use the function returning reference instead of the actual static object

	- Problematic in multi-threading system

		- Manually create all the local static object using function at single-threaded startup

- Ultimately, design around initialization order uncertainty 