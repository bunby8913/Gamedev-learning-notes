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

