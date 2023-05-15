# Primer Functions

- Function: A block of code with a name, execute by calling the name
  - 0 or more arguments
  - Can be overloaded, same name refer to different functions

### 6.1 Function basics

- Return type, name, 0 or more parameter, body
- Call operator: pair of parentheses
  - Takes expression of function / pointer of a function, add arguments inside the parentheses
- Calling function initialize parameter + transfer control to the function
- End function execution with the return statement
  - Transfer control back to the calling function

##### Parameters + arguments

- Does not guarantee the order arguments are evaluated
- Each argument must match corresponding parameters' type
  - Or can be converted to the parameters' type by compiler
- Must pass same # of parameters, make sure parameters are always initialized

##### Function parameter list

- Parameter can be left empty / filled with "void" keyword
- Each parameter must be separately declare its type
- Parameters cannot have the same name, they can have no name, but cannot be used, unless they are un-used

##### Function return type

- Cannot return array type + function type, but can return pointer of them

#### 6.1.1 Local objects

- Scope of name: Part of the program's text the name is visible
  - Hide any variable declared outside the scope with the same name
- Lifetime of object: The time program's execution the object exist

##### Automatic objects

- Ordinary object, created when variable's definition executed, destroyed at the end of the block
- Parameters + local variables
- After exits the block, automatic objects within scope become undefined
- Allocate memory to parameters when function starts
- Variables always requires a initializer, otherwise it will be undefined

##### Local static objects (static keyword)

- Local variable which continue to exist
- Initialized when the function is first called, before the execution of object's definition
  - Without explicit initializer for built in type, value will eb initialized to 0
- Not destroyed when the function ends, persist its value + destroyed when the program terminate
- Potentially could be issue with multi-threading, will require symbolization between threads

#### 6.1.2 Function declarations

- Declare function name before define + using it
  - Declare function in header file
    - Reduce error, make code less tedious
    - Source file that defines the function needs to include the header file with the declaration
  - Declaration of function is same as function definition, except no function body, replace with null statement;
  - Do not need to include parameters names

#### 6.1.3 Separate compilation

- Split program into separate files, compiled independently

##### Compiling + linking multiple source files

- Tell compiler all the files used
- Most compiler provide way to compile each file individually
  - Compile to .obj on window + .o on UNIX

### 6.2 Argument passing

- Unless parameter is a reference, only value of parameter are passed

#### 6.2.1 Passing by value

- Change made on the variable has no effect on the initializer

##### Pointer parameters

- Behave like non-reference type, the value of the pointer (memory address) is copied, pointers are not associated
  - However, can use pointer to change value of the object originally points to
- Should always try to use reference to pass object from outside the function

#### 6.2.2 Passing by reference

- Bound the reference parameter directly to the object when initialized
	- No longer need the address of the object

##### Using references to avoid copies

- Inefficient to copy objects to large class types
- For class types that cannot be copied, can only use reference to pass
- For reference that should not be changed in the function, references them as constant

##### Using reference parameters to return additional information

- For function to return more than 1 values
- Pass additional arguments to hold the other return values

#### 6.2.3 Const parameters + arguments

- Top vs. low level const: Describe the different ways to apply const qualifiers to printers + objects
	- Top-level const: The address points to are constant, but the content can be changed (const keyword after the type)
	- Low-level const: Object being pointed to cannot be changed, but pointer can point to different objects at different memory address (const keyword before the type)
	- Possible to have both top + low level const 
- Top-level const are ignored when we copy an argument to initialize parameter
	- Low-level const needs to match
- Can pass both const + non-const objects to const parameter
- Cannot overload the function by change the const qualifier on a parameter

##### Pointer or reference parameter + const

- General initialization rules apply

##### Use reference to const when possible

- Define unchanged parameter as plain reference creates misleading impression that function might change it’s value
- Using reference to const lead to more types of arguments available to the function

#### 6.2.4 Array parameters

- Cannot copy array, to use array in function, convert the array into a pointer
	- Cannot pass array by value
- Compiler can convert array to pointer of the first element
	- Function won’t know the size of array, need to pass that separately to avoid array out-of-bound errors

##### Using a marker to specify the extent of an array

- Can manage array by adding end marker as the last element
	- Cannot be part of ordinary data
	- Work well with c-style strings, but not well with other data type, where every value is in legitimate range

##### Using the standard library conventions

- Pass pointer to first elements + end of the array (1 pass the last element in an array)
	- Safe as long the pointers are correctly calculated

##### Explicitly passing a size parameter

- Defines a second parameter, passing the size of the array
	- Common in C + older C++ codes
- Safe to execute as long the size parameter <= actual size of the array

##### Array parameters + const

- Similar concept applies to reference as to pointer

##### Array reference parameters

- Can define a parameter as a reference to an array
- ```
	void print (int (&arr)[10]);
	// The Parentheses are necessary, otherwise, we are declaring an array of int reference
	```
- Size of the array is part of the initialization, meaning we can only pass array with the exact same size
	- argument must be an array of 10 ints

##### Passing a multidimensional array

- Pointer is a a pointer to an array
	- Need specify the size of the array the pointer points to (must be specified)
- ```
	void print (int (*matrix)[10], int rowSize) // make sure to include parentheses, otherwise it would be array of ten pointer instead of a pointer to an array with 10 ints
	```
- Can also use array syntax to define 2-D array
- ```
	void print (int matrix[][10], int rowSize)
	```

#### 6.2.5 main: handling command-line options

- Pass parameter to the program using command line options
	- ```
		prog -d -o file data0
		```
- For the main function, first argument points to the name of the program, from 1 is the command-line options
	- argc container the # of options added, argv is an pointer to an array of pointer of string to each input string

#### 6.2.6 Functions with varying parameters

- When we don’t know how many arguments to pass to a function
- Either use a library type initializer_list, or write special functions called variations template
- C++ have special parameter type, ellipsis, but should only be used when interface with C functions

##### Initializer_list parameters

- Represent an array of values of specified type
- ```
	Initializer_list<T> lst;
	```
- A template type, must declare the type of the element it contains
- Properties
	- Can have elements in initializer
	- Pass value to an initialize list must be enclosed in curly braces
	- Can share elements between initializer lists
	- Can show # of elements in the list
	- Can return the begin + end of the array
		- Can use ranged for on initializer list
	- All the elements are constant value, cannot be modified

##### Ellipsis parameters

- Allow program to interface to C code using varargs library
	- Should only use on types that are common for both C + C++
- ```
	void foo(param_list, …);
	```
- Does not perform type check on ellipsis parameter

### 6.3 Return types + the return statements

#### 6.3.1 Functions with no return value

- Void return type only, implicit return statement at the end of the function
- Can use return statement to call other functions that also returns void

#### 6.3.2 Functions that return a value

- Any return type other than void must return a value of the type (or able to implicit convert to the type)
- Compiler can guarantee every return include the appropriate type
- Error to not include return statement after loop, but not all compiler might detect them

##### How value are returned

- Return object create a temporary object where the function is called, the temporary object stores the return value of the function
- Can return reference to object in return, the object itself will not be returned

##### Never return a reference or pointer to a local object

- The memory of local object after function terminate are no longer valid
- In local function, string literal is converted to a local temporary string object, freed at the end of the function

##### Functions that return class type + the call operator

- Function has same precedence as "." + "->"
  - Left associative
  - Functions result can be used as call member of resulting object

##### Reference returns are l-values

- Return type determine the value of the function call
  - Return references = LValue
  - Return else = RValue
- Constant rules still applies, cannot assign value to constant variable

##### List initializing the return value

- Can return a braced list of value
- Return a empty / filled vector
  - If the list is empty, the temporary will be value initialized

- Braced list can only contain 1 value if it's built-in type
  - If return a custom class, then the class define how the initializer are used


##### Return from main

- Main function allows terminate without return

  - Compiler will automatically add return 0 at the end if not found

- C++ have special header to return machine independent error code

  - cstdlib

  - ```c++
    return EXIT_FAILURE;
    return EXIT_SuCCESS;
    ```

  - Not part of the standard library

##### Recursion

- Function that calls itself, directly / indirectly
- Recursive function must always have path that does not involve recursive calls (the base case)
  - Otherwise we enter an infinite loop (recursion loop)

#### 6.3.3 Returning a pointer to an array

- Use type alias to define data type, and use the synonym to define return value of functions

  - ```c++
    typedef int arrT[10]; // Create new name for an array of 10 ints
    using arrT = int [10]; // Equivalence, also assign alias for the array of 10 int
    arrT* func(int i);
    ```

##### Declaring a function that returns a pointer to an array

- Array must be declared with dimension

- Without type alias, we must define an array of fixed length + array pointer same length, and ask the pointer array to points to the first array

  - ```c++
    // Format of declaring function returns a pointer to an array
    Type (*function(parameter_list))[dimension];
    ```

  - The parameter_list defines the type of argument we use to call the function

  - the "*" defines that we can dereference the result of the call

  - [dimension] defines that the dereference result is an array of the size of the dimension

  - Type defines the type of element in the array

##### Using a trailing return type

- Useful when the function has complicated return types (i.e. pointer to array)
- Use the "auto" keyword replace the return type

##### Using decltype

- We can use decltype to define the return type base on the data structure

  - Since decltype will not automatically convert array to pointer, need to manually indicate that the function returns a pointer

- ```c++
  // function takes an int argument, return a pointer to an array of 10 int
  auto func (int i) -> int(*)[10];
  // extra note, can use declval function to get the decltype specifier wihtout constructing the object
  std::declval<int>
  ```

### 6.4 Overloaded functions

- Functions can have the same name but different parameters (overloaded)
  - Perform same general action, but takes different parameter
  - Main function cannot be overloaded

##### Defining overloaded functions

- Compiler use the parameter to figure which function to use
  - Number of parameter / type of parameter must be difference
- Error to only have different return type but same parameter

##### Determining whether 2 parameter types differ

- Parameter name are only for documentation, do not change the type of parameter
- Type alias does not affect the parameter type

##### Overloading + const parameters

- Top-level constant does not affect the object passed to the parameter, do not change the type of parameter
- Low-level constant creates different parameter types, Difference in pointer / reference in const / non-const version

##### Const_cast + overloading

- Use const_cast to remove / add the const-ness to variable to use pre-defined functions + change the const-ness of the return value

##### Calling an overloaded function

- Compiler compares parameter with function parameter, determine which overloaded to call
- 3 possible outcomes of calling overloaded functions
  1. Complier find exact (best) matching for actual arguments + generate code to call the function
  2. Compiler find no matching functions, compiler issue error message
  3. More than 1 functions could matches, no clear best, compiler throws an error of ambiguous call

#### 6.4.1 Overloading + scope

- Bad idea to declare function locally
- Names do not overload across scopes
- Name lookup happens before type checking
- Any re-declaration of functions inside the scope will override the declaration on the outside

### 6.5 Features for specialized use

#### 6.5.1 Default arguments

- Declare common value as default argument, allow function to be called without argument
  - Specify as an initialize for parameter in the parameter list
  - If a parameter has a default value, all parameters needs to have default arguments

##### Calling functions with default arguments

- When calling function with default arguments, can provide 0 to all arguments
  - Resolved by position
- Need to design the function to put the value least likely to use default value at the front

##### Default argument declarations

- Legal to re-declare a function multiple times
  - However, can only provide default argument once
- Default can only be specified if all parameters on the right has default value

##### Default argument initializers

- Apart from local variable, default argument can be initialized by any types of expression (convertible)
  - Value of the expression are evaluated when called

#### 6.5.2 Inline + constexpr functions

- Sometime functions runs significantly slower for some operations

##### Inline functions avoid function call overhead

- Specific the function as inline instead of calling function name

- Remove the overhead time of the function

- Define the function with the inline keyword (in front of the return type)

  - ```c++
    inline const string &shorterString();
    ```

- Compiler dependent, some compiler might ignore the inline request

##### constexpr functions

- Function can be used in a constant expression
  - Implicitly inline
- Return type + parameter types must all be a literal type
  - Function body can only have 1 return statement
- We can use constexpr functions to assign values to constant expression variable
  - Replace the function call with the returning value
- Can contain null statement, type aliases + using declaration
- Constexpr function is not required to return constant expression
  - If provide constexpr function a not constant expression argument, the function will return non-constant expression value
  - Compiler will throw an error

##### Put inline + constexpr function in header files

- All the inline + constexpr must match up in the code, therefore, best to put inline + constexpr functions in the header file

##### 6.5.3 Aids for debugging

- Provide program debugging code only executed during development
  - Can turn debug code off once finished
- 2 approaches, Assert + NDEBUG

##### The assert preprocessor Macro

- Preprocessor macro: Runs before the main compilation process, edit the source code before compiling
  - Acts somewhat like inline function
- Part of the "cassert" header, not apart of the standard library (std)
  - If evaluation is false, assert writes a message + turn off, is true, does nothing
- Usually used to check conditions that cannot happen

##### The NDEBUG preprocessor variable

- if NEDBUG if defined, assert does nothing

- By default NEDBUG is not defined -> assert perform run-time check

- Turn off NDEBUG by provide definition through command-line option

  - Defined NEDBUG avoid run-time overhead, but also no run-time check
  - Ensure assert macro only checks truly impossible condition

- Can insert conditional debugging code using NEDBUG in function

  - if NDEBUG is not defined, run the debugging code

- Available variable names

  - ```c++
    __func__; // print the name of the function
    __FILE__; // print the name of the file containg the function
    __LINE__; // print the current line number
    __TIME__; //print the compile time of the function
    __DATE__; //prin the compile date of the function
    ```

  - Can combine them together to generate run-time error messages

### 6.6 Function matching

- Sometimes, function will have the same # of parameters + parameters in different functions are related by conversions

##### Determining the candidate + and viable functions

- candidate functions: function with the same name of the called function + has visible declarations
  - First, identify all possible candidate functions
- viable functions: function with the same # arguments as the called arguments
  - Second, identify all possible viable functions within candidate functions
    - Compiler will throw error if there are not matching function
- Third, determine the capability of the functions with the called function
  - Argument matches the type exactly, arguments types that requires conversion

##### Finding the best match, if any

- Select from viable functions which parameters best matches the argument

##### Functions matching with multiple parameters

- The match of each argument is not worse than any other viable functions
- There's at least 1 argument that are better matched than any other viable functions
- Otherwise, return an error
- Matching can be forced with explicit casting, but not recommended + can be avoided in well-designed system

#### 6.6.1 Argument type conversions

- Compiler ranks all the conversions to corresponding parameters
  1. Exact match:
     - Argument type = parameter type
     - Conversion from an array type to the same pointer type
     - top-level const difference between the 2 types
  2. Matches through "const" conversions
  3. Matches through promotion
  4. Matches through arithmetic / pointer conversion
  5. Matches through class-type conversion

##### Matches requiring promotion / arithmetic conversion

- Well-designed function design should have parameters as closely related, to avoid surprising result from conversion
- All small integral will be converted to int or larger type
- All arithmetic conversions are equivalent to each other, does not take precedence over others + could cause ambiguous call

##### Function matching + constant arguments

- The compiler can use the const-ness of parameter / function call parameters
	- Cannot bind a plain reference to const object
- Pointers parameters works the same way
	- The compiler can decide which function to use base on the const-ness of the calling argument


### 6.7 Pointers to functions

- Function pointer: a pointer that points to an functions
	- Points to a particular type, determined by the return type of the function + parameters
- To create a pointer to function
	- ``` C++
	  type (*pointer_name) (parameter_Type_1, parameter_Type_2, …);
    ```
- The parentheses around the pointer name emphasis that it’s a pointer to function, not a function that returns the pointer version of the return type

##### Using function pointers

- Can assign address to functions pointer by assigning them with function names
	- ``` C++
	  pointer_name = function_name;
	  pointer_name = &function_name; // equivalent assignment
    ```
- Can use pointer to function call to call the function pointes to
	- ``` C++
	  return_Type variable = pointer_Name(parameter1, parameter2…);
		return_Type variable = (*pointer_Name)(parameter1, parameter2…); // equivalent calls
		```
- Different pointer with different return type / parameters cannot covert to each other
	- However, can assign pointers to function to a null pointer or 0

##### Pointers to overloaded functions

- Since we have to declare the parameter + return type to create pointer to function, compiler use those to determine which overloaded function to assign to the pointer

##### Function pointer parameters

- Functions can be used in the parameter of another function, and will be automatically covert to a pointers to function

  - ```c++
    return_Type function_Name (parameter1, return_Type function_Name(parameter1, parameter2, ...));
    return_Type function_Name (parameter1, return_Type (*function_Name)(parameter1, parameter2, ...));
    ```

- It's also possible to use "decltype" + "typedef" to create pointers to function

  - To create the pointers to function first, then use it as a parameters later

##### Returning a pointer to function

- Can't return a function type, but can return a pointer to a function

  - Use type alias to declare function that returns a pointers to function

    - ```c++
      using pointer_Name = return_Type(*) (parameter1, parameter2);
      ```

    - Need to explicitly specify the return type is a pointers to function

##### Using "auto" / "decltype" for function pointer type

- Use "decltype" if we know which function we want the pointer points to
  - Simplify writing a function pointer return type
- Notice "decltype" return the type of the function, not in pointer
