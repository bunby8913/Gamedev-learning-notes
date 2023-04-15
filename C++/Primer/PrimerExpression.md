# chapter 4
## Expression
### 4.1  Fundamentals

- Unary operators, single operands
	- *, &  
- Binary operators, act on both operands
	- +, ==
- Some can be both binary + unary operator, depends on the situation
- Follows the same precedence + associativity rules
- conversions are done automatically
#### Overloaded operators
- The definition of operators can be re-defined for individual class

##### R-value vs. L-Value

- L-value usually represent the identify of the value (memory location)
  - Cannot be replaced by the R-Value
- R-value usually represent the value associated (content)

##### Decltype type refresher

- define an variable using expression
  - The compiler analyze the expression and determine the type
  - Will not actually call the function, but will use the return type of the function
- Different from auto assignment
  - Will include type level const + reference
    - If expression is a reference, the new variable will be the same reference
    - Will return reference to L-value (memory location)
- decltype with a parenthesized variable will always be a reference

#### 4.1.2 Precedence + associativity

- Compound expression: expression with more than 2 operators
- Higher precedence group operates first, within the same precedence group, use associativity determine the regroup

#### 4.1.3 Order of evaluation

- Usually no way knowing the order of evaluation
  - Error for expression to refer + change the same object
    - Un-defined behavior error
- Only logical operators will operate in order (evaluate first part then second part)
  - and, or, conditional, comma operator

##### Advice on managing compound expressions

- When in doubt, use parentheses
- Don't change + use the same operands in the same expression

### 4.2 Arithmetic operators

- Unary operator has the highest precedence
- Multiplication + division > addition + subtraction
- Result of arithmetic operators are r-values
  - Smaller data type will be promoted to larger data type
  - Convert to common type before operations
    - Bool will be promoted to Int
- Careful + watch out for overflow of data type

### 4.3 Logical + relational operators

- Return bool values
- In relational operators, chaining up operators can cause unexpected result
  - Best to split into multiple statements
- To test truth value from arithmetic/pointer type, use value in a condition
  - When the value is not a bool, comparing values does not work
  - Bad practice to use Boolean literals as operands

### 4.4 Assignment operators

- Left of assignment must always be modifiable lvalue
- right of assignment will be converted to the type on the left
- Assignment is right associative
- Operations is applied from right to left
- Low precedence, but assignment has even lower precedence, needs to add parentheses to the the assignment

##### Equality $\neq$ assignment operators

##### Compound assignment operators

- Assign result to the same object

- Arithmetic operators + bitwise operators

  - ```c++
    a = a op b (a = a << b)
    ```

- Compound assignment, left hand operand only evaluate once

### 4.5 Increment + decrement operators

- Before the variable: Yield the incremented value
- After the variable: Yield the un-incremented value
- Should always use prefix operators unless for a reason
  - Avoid unnecessary work

##### Dereference + increment in a single expression

- Increment can also work on getting variable that a single compound expression
  
  - ```c++
    auto pbeg = v.begin();
    // if the current element is not the last and not negative
    while (pbeg != v.end() && *pbeg >= 0)
        // Deference + print the value + increment pointer to next element
        cout << *pbeg++ << endl:
    ```
  - Use in for loop to get all the value in a vector
  - Post fix increment has higher precedence than deference
    - Will increase the to next elements, but dereference the previous unchanged pointer's reference
  

##### Operands can be evaluated in any order

- If left + right hand operants use the same variable + right-hand operant changes the variable, the assignment becomes undefined
- ``` c++
	*beg = toupper(*beg++);
	// could be interpreted either as
	*beg = toupper(*beg); // left-hand evaluate first
	*(beg + 1) = toupper(*beg); // right-hand evaluate first
  ```

### 4.6 The member access operators

- Dots (.) + arrow (->)
- ``` c++
	ptr -> mem = (*ptr).mem
	```
- Dereference has lower precedence than dot, Dereference operation must have parentheses
- Arrow operator can only yield l-value, dot operators depends on the left operands

### 4.7 The conditional operator

- ?: conditional operator, if-else logic as a expression, guarantee only 1 expression will be evaluated
- ``` c++
	cond ? expr1(true) : expr2(false);
	```
- expression 1 + 2 need to be able to convert to common type / same type
- Result is l-value if both expression type are / can convert to L-value type, else, r-value

##### Using conditional operator as output expression

- Very low Precedence, good practice to always include parentheses around them

### 4.8 The bitwise operators
- Take (signed/unsigned)integer and use them as collection of bits
	- Could also operate on library
	- Does not have special logic to handle signed bit, strongly recommend to use unsigned int
	- Adds higher bit as 0 during conversion
- “~”: bitwise NOT
- “&”: bitwise AND
- “^”: bitwise XOR
- “|”: bitwise OR
- “<< + >>” : left/right shift
	- Right operant needs to > -1, less than the # of bits in the result
	- Always insert 0-valued bit

##### Shift operators are Left associative 
- ``` c++
	cout << 10 < 42 = (cout << 10) < 42 // error, compare cout with 42
	```

### 4.9 The sizeOf operator

- return the size in bytes of expression / type name
	- Right associative
	- Produce constant expression of size_t
	- Does not evaluate the operand
- Can contain parentheses or not
- ``` c++
	Some_data data, *p;
	sizeOf(Some_data) = sizeOf(*p) = sizeOf(data); // all return the size required to hold the object
	sizeOf(p); //size of a pointer
	sizeOf(data.revenue) = sizeOf(Some_data::revenue); //all getting the size of the revenue type
	```
- Safe to evaluate / referencing size of invalid pointer, does not actually use the pointer
- SizeOf can directly operate on class to get size of a member of a class type
- SizeOf does not covert array to pointer
- sizeOf of vector return the size of pointer holding the elements, does not consider size of each element

### 4.10 Comma operator

- 2 operands, left associative 
	- Guarantee order of evaluation
- Lowest precedence
- evaluate and discard result, return the result of the last operands

### 4.11 Type conversions

- 2 types are related if they can be converted to one and another
- Implicit conversions: Automatic conversions, without programmer intervention
  - Preserve precision if possible (int -> double)
- Type of object initializing dominated the type

##### When implicit conversions occur

- Smaller integral type usually convert to larger integral type
- non-bool convert to bool
- Right hand operand convert to left hand operand
- During function calls

#### 4.11.1 Arithmetic conversions

- Always the larger integral type
- Always float > integral
- bool, (signed / unsigned) char, (unsigned) short -> int if fit, otherwise -> unsigned int
- In relational expressions / operator, convert to a common type
  - Integral promotions happens first, if both have the same signedness, smaller type convert to larger type
  - If signedness different, signed operand convert to unsigned if unsigned is the same or larger type
  - Special case: signer operand is a larger type then the unsigned operand, machine dependent

#### 4.11.2 Other implicit conversions

- Array to pointer conversion: Array automatically convert to pointer, point to the first element in array
  - Does not work with decltype, won't convert for sizeOf, typeid
- Pointer conversions: constant int 0 + nullptr can convert to any pointer type
  - Non const type can convert to void *, any pointer can convert to const void *
- If pointer is 0 / arithmetic value = 0, convert to false, else convert to true
- Conversion to const: any non const pointer + reference can convert to const pointer + reference
- Conversion defined by class type

#### 4.11.3 Explicit conversions

- Use cast to explicit require conversions (dangerous operation)

##### Named casts

- ```c++
  cast-name<type>(expression);//type = target type, expression = value to cast
  ```

###### Static_cast

- Apart from low-level const, can convert any type conversion

- ```c++
  double answer = static_cast<double>(j) / n;
  ```

- Establish we are not concerned with loss of precession

  - Turn off warning message of convert from large type to small type

- Guarantee conversion: Covert any pointer to void * type, and convert back to original type later (What Unreal cast to use?)

###### Const_cast

- Change only low-level const, sed to modify the const-ness of an object
  - Immutable object to be modified
  - However, cast away the const and then modify the object might be undefined, compiler already decided the object to be const
- Should only be used when modifying the object is safe
- Used to overload function

###### Reinterpret_cast

- reinterpretation of the bit pattern of its operand, ask compiler to read the bit stored as different type
- Used when working with low-level programming, interacting with hardware, raw memory or C libraries
- Should not treat the casted pointer as the new type
  - Machine dependent

###### Old-style casts (legacy)

- ```c++
  type (expr);
  (type) expr;
  ```

- Should not use, can be any types of cast above mentioned
