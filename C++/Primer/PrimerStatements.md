# Primer Statements

### 5.1 simple statements

- Evaluate the expression, discard the result

##### Null statements

- When language need statement, but don't need logic, make sure to comment them
- Double semicolons just adds a null statement at the end
  - Semicolon after while/for/if changes the logic, will loop null statement instead

##### Compound statements (blocks)

- Names / variables within blocks are local
- Block is not affected by semicolon

### 5.3 Conditional statements

#### The if statement

- Can be expression (evaluation) or variable declaration (which will be true)
- Braces control the flow, code are outside the scope of conditional statement, and will always execute
- Dangling else: Each else is matched with the closet preceding unmatched if
  - Control execution with braces (reason conditional statement should always use braces)

#### The switch statement

- Selecting from a # of fixed alternatives

- ```c++
  switch(variable)
  {
      case integralConstant:
          statements;
  }
  ```

- Evaluating parenthesized expression follows the keyword 'switch'

  - Once the correct case found, execute until the end of switch / break statement, exit the switch statement block

- If no match found, exit the switch and execute the next statement

- Case label: case keyword must be followed with integral constant expression

- Error if any case label has the same value

- Start from the top label, execute every cases until interrupts / finishes

  - When we have 2 or more values shares a common set of actions, don't need break the flow of switch
    - In this case, can also put all the case on the same line, improve readability (Use comment to explain logic)

- Without break, any statements under the found case will be executed

  - Good practice to include break at the end

##### The default label

- The default label executed when no case label matches, cannot be empty

##### Variable definition inside the body of a switch

- Illegal to initialize variable in 1 case, and use it in another case
  - Initialize out of scope
- Define + initialize for particular case inside a block

### 5.4 Iterative statements (loops)

#### The for statement

- init-statement must be an expression / null statement
  - Initialize / assign starting value being modified
  - Init-statement can define multiple object, all the object initialized by init-statement must be the same type

##### Omitting parts of the for header

- Omitting conditions means it will always be true, need to the body block to contain statement to break the loop
- Omitting expression, the body must push the code to advance

#### The range for statement

- ```c++
  for (declaration: expression)
      statment;
  ```

- Expression must be a list, array, vector, string

  - Has begin + end member, return iterators

- Must be able to covert elements in expression to declaration type

  - Use auto type specifier
  - Variable defined + initialized by the next sequence in expression, executed statement on the element
  - To change the value of the elements, declaration should be a reference (instead of value)

#### The do while statement

- Similar to while, but conditions are tested after execution

  - The loop will execute at least once

- ```c++
  do{
      statement;
  }
  while (condition);
  ```

- Condition cannot be empty

  - If condition evaluate to false, loop stops

- Cannot declare variable in condition and use it in the statement

### 5.5 Jump statements

- Interrupt the flow of execution

#### The break statement

- Terminate the nearest while, do while, for or switch statements
  - Does not terminate the code after them

#### The continue statement

- Terminate the current iteration, and immediately begin the next iteration
  - Only in for, while and do while loops
  - Only applied to the nearest loops
  - Can only be inside a switch block if the switch statement is in a iterative statement

##### The goto statements (Legacy)

- Should not be used, make code hard to understand + modify

- ```c++
  goto label;
  
  label: return;
  ```

- Cannot initialize variable outside goto and use it in the goto section

- Can jump backward to earlier code, but will destroy anything the goto statements created earlier

### 5.6 Try block and exception handling

- ```c++
  throw runtime_error("Additional information");
  ```

  

- Dealing with anomalous behavior, detect part of the sysemt that cannot continue

  - Need to signal the program without knowing which part of the system will deal with the exception

#### Throw expression

- Indicate something that system cannot handle
- Throw expression to raise an exception
  - Terminate the current function, transfer control over to error handler
    - Need to initialize handler with string to provide additional information

#### Try block

- ```c++
  try {
      statements;
  }
  catch (exceptation-declaration)
  {
      handler-statements;
  }
  ```

- Try block followed by a list of catch clauses

- Catch does not need named declaration

##### Writing a handler

- Each error class have a member function what(), return c-style character string, copy of the string initialize the particular object

##### Function are exited during the search for a handler

- In a chain of try blocks, reverse the call chain when an exception is thrown
  - Keep searching until a appropriate catch is found, if not, terminate the program
- If unexpected error occurred without try block, also terminate  

#### Standard exception

- 4 types of header defined in the standard library
  - Exception: general kind of exception, does not provide extra information
  - Stdexcept: 
  - new: define bad allocation exception type
  - type_info: defines bad_cast exception type
- Can create, copy and assign object to be any types of exception
  - Default initialize new, type_info and exception type of exception
  - stdexpect requires an initializer, string to provide additional information
    - Single function what() return the information

