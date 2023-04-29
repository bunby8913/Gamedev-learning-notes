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





### 6.4 Overloaded functions

### 6.5 Features for specialized use

### 6.6 Function matching

### 6.7 Pointers to functions
