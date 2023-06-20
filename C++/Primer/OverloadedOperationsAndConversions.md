# Overloaded operations + conversions

- Define the meaning of operator when applied to a specific class
  - Make the program easy to read + easy to write 

### 14.1 Basic concepts ("operator")

- Use the "operator" keyword, followed by the overloaded operator symbol
  - Function uses the same # of parameter as the operand
  - left first, right second
- Apart from "operator()", not allowed to have default argument
  - if a member function, the first variable is omitted to "*this", binary operator function will only take 1 parameter
- Operator function must either be part of a class member / at least 1 parameters being class type
  - Prevent from overriding built-in operator operation
- Not all operators are overload-able
- Can only use the available operator, cannot invent new ones
- Some symbols can be used as unary + binary operators
  - Operator being defined determines by the # of parameters
- Operators cannot be overloaded
  - "::", ".*", ".", "?:"

##### Calling an overloaded operator function directly'

- Call the function by using the operator on the defined type
  - Alternatively, call the function by the function name "operator+(or other operators)(argument list)"
- Call the member function using an object to run the function
  - Use the dot operator "." to get the function + pass in argument

##### Some operators shouldn't be overloaded

- Operators which guarantee orders should be not be overloaded
  - Function call do not guarantee order of evaluation
  - "&&", "||", "," should not be overloaded
- Overloaded logical AND + OR does not preserve circuit properties, both operands are always evaluated
- "," have built in meaning + should not be overloaded

##### Use definition that are consistent with the built-in meaning

- Operations with logical mapping should be defined as overloaded operators
  - If the class use IO, The shift operator should be consistent with built-in IOs
  - If the class requires equality check, define equal to "==" + not equal to operator "!="
  - If class needs to be sorted in orders, should define all the relational operators
  - Return type should be consistent with the built-in definition
    - Comparison should return "Bool"
    - Arithmetic should return class type
    - Assignment + compound assignment should return reference to left-hand operand

##### Assignment + compound assignment operators

- Should behave close to synthesized operators
  - Left + right operand should have the same value
  - Should always return the left-hand operand reference
- Should avoid over-use of overloaded operators
  - Each Operator should only have 1 interpretation
- If an arithmetic / bitwise operator is defined, should also define the compound assignment version of the operator

##### Choosing member / nonmember implementation

- Depends on the situation, overloaded operator should be member function / ordinary nonmember function
  - Symmetric operators (arithmetic, equality, relational, bitwise  + compound operator) should be non-member functions
  - Assignment + access + call + tightly type related operators should be member function
- Overloaded operator must be a non-member function if the types of object does not match
  - Member function can only operate on the member type
  - Ensure at least 1 operands is a class type

### 14.2 Input + output operators

#### 14.2.1 Overloading the output operator "<<"

- First parameter should be a non onset reference to a “ostream” object
	- Reference since “ostream” object cannot be copied
- Second parameter should be a const reference to a class type
	- Reference to avoid copying the object
- The operator should return a reference to the “ostream” object of the first parameter

##### Output operators usually do minimal formatting

- Text formatting should be handled by user / use a specific formatting function
	- Output operators especially should not print new line between operands

##### IO Operators must be non-member function

- “iostream” must be non-member ordinary function
	- If the operators belong to a class, then the class must be a member of “iostream” class
		- Cannot add new member to a library class
- To read non-public data members, IO operator should be defined as “fried” to the class type

#### 14.2.2 Overloading the input operator “>>”

- Similar to the parameters of the output operators
	- Except the second parameter should be a non-const, needs to read the input operator’s data to the object
- Input for each object is separated by white space “ “ in the input stream object

##### Errors during input

- Error during input include stream contain incompatible type from the right hand operand + reaches the end-of-file character + other errors
- Only check if the input stream is valid when we have to use the captured data
	- Depends on class, decide how to error recovery
	- If not valid, reset the object to default state
	- The object should remain valid in case the input error is ignored

##### Indicating errors

- Consider using the fail bit to apply data verification when the data failed the data check
	- Bad bit indicate the stream was corrupted

### 14.3 Arithmetic + relational operators

- Arithmetic + relational operator should usually be non-member function, to allow type conversions
	- Operands shouldn’t need to change state, should be const
- Arithmetic operator should store result as a local variable + return the local variable at the end
	- If defines a arithmetic operator, should also define the equivalent compound assignment operator too
		- Usually more efficient to use the compound assignment operator to define the arithmetic operator

#### 14.3.1 Equality operators (“==“, “!=“)

- If all data member of 2 objects are equal, then the object are equal
- Should use the equality operators to define the NOT equality operators
	- Delegate the work to the other operator
- If a class needs an operation to compare 2 objects, should overloaded the equality operators instead of a named function
	- More intuitive
- Equality operator should be transitive

#### 14.3.2 Relational operators

- Since some associative container + algorithms uses the “<“ operator, useful to overloaded for a class
	- Define a order relation for associative container + order between objects that are not equivalent
- Important to consider relational operators is necessary
	- In many cases, not equivalent objects might not be related to each other
- If there are no single logical definition of relational operator, should not define 1

### 14.4 Assignment operators (“=“)

- Class can overload assignment operators that takes other type as right-hand operand
- Overloaded function should always return a reference to the left-hand operand
	- Does not need to check for self-assignment, since the right-hand operand is a different type
- Must be defined as member function

##### Compound-assignment operators

- Not requires to be a member function, but will increase readability is included with the class
- Return reference to the left-hand operand

### 14.5 Subscript operator (“[]”)

- Must be a member function
- Usually return an reference to the element being fetched
	- L-value reference can exist on both side of assignment
- Good Idea to define both const + non-const version of operator, Return const object if the content of the return object should not be changed
	- Define both the function + return type to be const

### 14.6 Increment + decrement operators (“++”, “—“,)

- Implement ed for iterator, move between element in a sequence
- Not required to be member function, but they modify the most likely modify the state of the object in the sequence, best to keep them in class
- Should define both prefix + postfix increment / decrement in class

##### Defining prefix increment / decrement operators

- Define the prefix increment/decrement operators as follow
	- ```C++
	   T& operator++();
		T& operator—();
	```
- Both should return a reference to the incremented/decremented object
- The function should first check if the object is still valid
	- Check will throw if the iterator we pass is not valid / at the end of the container
- Otherwise, prefix increment/decrement + return the object

##### Differentiating prefix + postfix operators

- Postfix overload function takes an extra un-used parameter of (int)
	- ```C++
		T operator++(int);
		T operator—(int);
	```
- The int variable is not used, therefore, does not require a name
- Postfix operator should return the old value, not a reference
- The function needs to create a new object to holds the current value, increment the object, return the stored value
- Postfix operator can call the pre-fix operator to handle the increment
	- This will avoid us writing the check function twice

##### Calling the postfix operators explicitly

- Can call prefix + postfix operator as a function call
	- For post-fix operator, add a integer into the variable list to distinguish

### 14.7 Member access operators (“*”, “->”)

- Can be used by classes act as iterators + smart pointer classes
- Operator must be a member, dereference operator does not have to be a member function + but are encourage to be
- Dereference operator should perform check first, if valid, return the address of the element by the operator
- The arrow operator calls the dereference operator to perform check + dereference, return the object being referenced by the address
- Everything should be non-const, so return value can be modified again

##### Constraints on the return from operator arrow

- Dereference operator can be coded to do whatever
- Not the case for overloaded arrow
	- Cannot change the definition of arrow (fetching specified member)
	- Left operand must be a pointer to class object / with overloaded arrow operator
- If left operand is a object to class type with overloaded operator arrow, the return value is used to access the data member
	- If return a pointer, it can be used for direct access
	- If return a object with arrow operator overloaded, continue to run the function until it returns a pointers

### 14.8 Function-call operator (“()”)

- Must be a member function
- Overloading the call operator allow objects to be used as function
	- More flexible than ordinary function
	- ```C++	
		T operator()(T variables) const
	```
- Once the functional call operator loaded, “calling” the object as function will run the overloaded call operator
- Known as the “function objects”

##### Function-object classse with state

- Function object can use in-class members, class constructor to customize operations
	- When calling the function object, can use default argument / provide arguments
- Function object are usually used as argument for generic algorithms

#### 14.8.1 Lambdas are function objects

- Compiler translate lambda expression into unnamed object of unnamed class
	- Single class member, single function-call operator
		- Normally, lambda cannot change function member, const function member
		- Mutable lambda expression defines function as non-const
	- In essence, lambda expression construct a un-named object, calls the name function, execute the function-call operator function

##### Classes representing lambdas with captures

- Lambda can use variable by reference directly, without storing data locally
- Lambda that captures variable by value will store value locally, as a data member in class
	- Constructor will initialize the constructor member
- Lambda’s deleted copy + move constructor depends on the data type

#### 14.8.2 Library-defined function objects

- Each arithmetic, relational + logical operator can define function-call operator
	- A template that takes a single type
	- Use the template to create function object
	- ```C++	
		plus<T> plusFunction;
		T var . plusFunction(var1, var2);
	```

##### Using a library function object with the algorithms

- Function object used to override default operator behaviour
- Alter the default behaviour of generic algorithm by supplying different operator
- Function object guarantee to work with pointer
	- Pointers are ensured to be the same type
- Associative container uses library function to compare between keys
	- Able to use pointers as key in “map” structure

#### 14.8.3 Callable objects + function

- Multiple callable object could share the same call signature
	- Defined by the return type + arguments type of the function
	- ```C++	
		T1(T2, T3);
	```

##### Different types can have the same call signature

- When different function have the same call signature + perform different tasks, best to put them in a function table
	- Store a function pointer, depends on the situation, provide the same argument variables to different functions
- However, some callable are different class type, cannot be stored in the same function table
	- “lambda” is a different class type from other callable

##### The library “function” type

- Part of the “functional” header
- A template type, takes the call signature as the type
- Now possible to  assign a Lambda expression directly to a function type variable
	- Can use the “function” type define a function table
	- Using a “key” to retrieve “function” type, which returns the callable object to pass the arguments to perform tasks

##### Overloaded functions + “function”

- If a function is overloaded + wants to store the function using the “function” type, needs to resolve ambiguity
	- Store the correct version of the function to a function pointer first
	- Then, use the function pointer to Insert the function to the function table

### 14.9 Overloading, Conversions + operators

- Possible to define a “class-type conversions” using a conversion operator

#### 14.9.1 Conversion operators

- ```C++	
		operator T() const;
	```
- A special member function, convert 1 class type to another type
	- Can convert to any return type other than void, array + function types
	- Can be converted to any pointer + reference type
- Conversion should not change the state of the object, function should be const

##### Defining a class with a conversion operator

- Implicit user-defined conversion can be applied before / after built-in conversion
  - The object can be converted using convert operator + another built-in conversion
- Implicitly applied, cannot pass any argument to the function
  - Return type determined by the conversion
- Conversion function can be misleading when there are more than 1 one-to-one mapping options available

##### Conversion operator can yield surprising results

- Common for class to determine conversions to "bool"
  - Conversion could happen at unexcepted places
    - i.e. Bits manipulation

##### "explicit" conversion operators

- If apply "explicit" conversion to conversion operator
  - Will still apply the conversion, but must be called through a "cast"
  - Conversion are used implicitly to convert expression for conditional operations

##### Conversion to "bool"

- If the converting to "bool", use "explicit" conversion operator
  - Since conversion to "bool" is usually for conditional evaluation

#### 14.9.2 Avoiding ambiguous conversions

- There should only be 1 way for a class type to convert to target type
  - Either direct mutual conversions / multiple conversion to related arithmetic type
    - Should avoid mutual conversions + conversion > 2 arithmetic types

##### Argument matching + mutual conversions

- If multiple call are equally good, it's ambiguous + error
- Can't be solved with cast, suffer from the same ambiguity

##### Ambiguities + multiple conversions to built-in types

- Class cannot define multiple conversion that are related to each other
  - i.e. Class define constructor from / conversion to multiple arithmetic type
- If the conversion process is indistinguishable (all takes the same steps) call is ambiguous + will cause errors
- If there are multiple user-defined conversion, the rank of standard conversion determine which conversion operator to use

##### Overloaded functions + converting constructors

- overloaded function takes different class type with same converting constructors will cause errors
  - Ambiguous, could call any of the class type

##### Overloaded function + user-defined conversion

- When calling overloaded function, if multiple user-defined conversion can be used, they are equally good
  - Rank of standard conversion does not matter between different class
    - Only matter same conversion function can be matched with built-in conversion

#### 14.9.3 Function matching + overloaded operators

- Operator can be overloaded, same concept as overloaded functions
  - Using function matching to determine which overloaded version of operator to use
- Overloaded function pool include Non-member + built-in version of the function
- Usually ok if perform operator operation between same class, however, could be ambiguous when perform mixed-mode arithmetic
