# Primer generic algorithms

### 10.1 Overview

- Most algorithm are part of the "algorithm" + "numeric" header
- Algorithm perform operations on a range of element, defined by iterators
- Generic algorithms can be applied to multiple types of container
- Algorithm are container independent, but depends on element-type operations
  - Most algorithms provide way to supply operation to replace default operator
- Algorithm does not use container operation
  - Cannot add/remove elements directly to container

### 10.2 A first look at the algorithm

- Input range: The range of elements the algorithm operates on
  - Always the first 2 parameters of the range
  - The first element + 1 past the last element to be processed

#### 10.2.1 Read-only algorithms

- Algorithm that read but not write to the container

##### Algorithms + element types

- The element in the container must match / convert to type of the third argument
- Best to use "cbegin()" + "cend()" for read-only algorithm

##### algorithms that operate on 2 sequences

- Takes 3 parameter, the start + end range iterator of the 1st sequence + the starting iterator of the 2nd sequence
  - Algorithm assumes that second sequence is at least the size of the range of the 1st iterator

#### 10.2.2 Algorithms that write container elements

- Needs to manually ensure the # of element being written =< # of element in the container
- Iterator arguments: When read elements from different container, the container type DOES NOT have to match + the element type DOES NOT have to match
  - As long as they can be compared against
  - Need to ensure that algorithms that takes 3 arguments, second container is at least same size to the first one


##### Algorithms do not check write operations

- Algorithm will assume that there's space + element to receive the write operations
  - Will not check if the algorithm exist or not

##### Introducing ("back_inserter")

- Insert iterator: An iterator that adds element to a container
  - Used to add new value to the container
- "back_inserter": Take a reference to the container + return a insert iterator
  - Also add a value to the container
- "back_inserter" can be used to append element + assign value to the new elements

##### Copy algorithms

- Write the element in the first container to the second container using destination iterator
  - After the algorithm, destination iterator will point 1 past the last elements
- Several algorithms will store result in new temporary sequence and return a "copy" sequence

#### 10.2.3 Algorithms that reorder container elements

##### Eliminating duplicates

- Use combination of "sort()", "unique()" + "erase()" to eliminating duplicate in a vector
  - "sort()" takes 2 iterators (start + end) + sort the elements in range

##### Using ("unique()")

- The "unique()" algorithm rearrange input container, puts all the unique elements at the front
  - Returns an iterator denotes the end of range of unique values (1 pass the last "unique" word)
  - Does not remove the element from the container

##### Using container operation to remove elements

- Use container operation to manipulate the size of the container, "erase()" function
  - Erase all elements from the unique iterator to the end of the container

### 10.3 Customizing operations

- Library defines ways to supply custom operation, to replace default operator

#### 10.3.1 Passing a function to an algorithm

- Some overloaded version of the algorithms take an extra arguments, "predicate"

##### Predicates

- Expression can be called + return a value for condition
  - Unary / binary predicates: Predicate with single / double parameters
- Needs to be able to convert element type to predicate's parameter type
- Needs to meet a set of requirements, described later

#### 10.3.2 Lambda expressions

- For processing more than 2 arguments

##### Introducing Lambdas

- Possible to pass any callable object to an algorithms

  - Any object/expression is callable is we can apply "(args)" to it

- Lambda expression is another form of callable object other than functions / function pointers

  - Can be considered as a unnamed, inline function

    - Needs parameter list, function body + return type

  - ```c++
    [capture](parameter_List) -> return_Type {function_Bodies}
    ```

    - Capture list: list of local variable defined for the function (often empty)
    - Lambda must use trailing return to specify the return type

  - Leaving the parentheses + parameter list indicate empty parameter list

  - Leaving out return type so it will depends on the code in the function body

    - If function body have more than single return statement + did not specify return type = return "void"

##### Passing arguments to a Lambda

- Argument + parameter type needs to match
- Lambda cannot have default function
  - Lambda always needs the exact amount of arguments
- The Lambda expression will be called when comparison are needed

##### Using the capture list

- Lambda can only use local variable if it's specified in the capture list
  - Local variable = variable part of the surrounding function

#### 10.3.3 Lambda captures + returns

- Compiler will generate a new class type for each Lambda defined
  - Define a new type + make a new object of the type
  - The "auto" keyword will create a new Lambda object
- Lambda class by default contain data members in the capture list

##### Capture by value

- Can capture a variable by value / reference
- Value of captured variable is stored when created in capture list, not called
  - Changes to the variable after Lambda creation has no effect on the return value of Lambda expression

##### Capture by reference

- Variable capture by reference just like other reference
  - Bound to the object, when the object changes, the Lambda expression reflect the change
- Developer need to make sure the referenced object still exist when calling Lambda expression
- Possible for function to return Lambda
  - Either return an callable object / an object that has callable object data member
  - Lambda cannot return reference captures (it will reference to a deleted local variable)
- Should always keep Lambda captures simple
  - Hard to ensure the referenced capture item still has the intended value when used by Lambda
  - Should avoid capturing reference + pointer if possible

##### Implicit captures ("&" / "=")

- "&": Capture by reference
- "=": Capture by value
- Can combine implicit + explicit captures
  - Separated by ",", the first item always needs to be "&" / "="
  - Explicitly capture must be in different type from implicit captures


##### Mutable lambdas

- Add "mutable" keyword to Lambda expression after parameter list
  - "mutable" Lambda expression cannot omit parameter list
- Depends on the const-ness of the variable, may not be able to change

##### Specifying the Lambda return type

- Lambda will return void if no return type specified

- Denote return type using trailing return type

  - ```c++
    []](T variable) -> return_T {...};
    ```

#### 10.3.4 Binding arguments

- Should be simple, rare operation
  - Use function otherwise
- For argument through parameter

##### The Library "bind" function

- Part of the "functional" header

- general-purpose function

  - Takes the current callable object, create a new one with all the local variable included

  - ```c++
    auto variable_Name = bing(callable, arg_index arg_list);
    ```

- Essentially creates a new function, wrap the original function + have certain arguments pre-filled

  - Calling this function in the future will use less arguments than before
  - The argument we needs to pass when we call the function


##### Using "placeholder" names

- "placeholder" is a special namespace in C++

  - ```c++
    using std::placeholders::_1; // for using _1 to represent a placeholder variable
    using namespace std::placeholder; // willl make all name defined by placeholders usable
    ```

- part of the "functional" header

##### Arguments to "bind"

- "bind" can bind / re-arrange parameter for any callable
  - The index (rather than position) determine the order in argument list

##### Using to "bind" to reorder parameters

- Useful to use "bind" to swap the order of the parameters + change the meaning of the function

##### Binding reference parameters

- It's essential to sometime pass reference + type that cannot be copied through bind to another function

- "bind" function does not support direct reference passing
  - However can use the "ref()" function, return an object contains the reference that a copyable
    - "cref()": return an const reference

### 10.4 Revisiting iterators

#### 10.4.1 Insert iterators

- An iterator adaptor

  - Takes container, return an iterator that can add element at specific location

- "back_inserter": Iterator that can perform "push_back"

- "front_inserter": Iterator that can perform "push_front"

- inserter: Iterator that can perform "insert", require second argument of an iterator of the container, element inserted in front of the iterator

  - ```c++
    auto it = std::inserter(vec); // is same as
    auto it = c.insert(it, val);
    ++it;
    ```

  - If element are inserted using the "front_inserter", since every element in inserted in front of the container, the list will be reversed

#### 10.4.2 "iostream" iterators

- "istream_iterator": Read the input stream
- "ostream_iterator": Read the output stream
- Use stream iterator to read / write data from stream object (usually cannot be copied)

##### Operations on "istream_iterators"

- "istream_iterators" default to "off-the-end" value

- When using an "istream_iterators", needs to assign a default type

- ```c++
  istream_iterator<T> in(cin), eof; // Read T from cin
  vector<T> vec (in, eof) // construct vec from the iterator range
  ```

  

##### Using stream iterators with the algorithms

- Support some iterator operations, i.e. "accumulate" to calculate the sum

##### "istream_iterators" are permitted to use lazy evaluation

- istream_iterator may not read the stream immediately
  - Guarantee to read before dereference the iterator
- However, might be issue if "istream_iterator" is destroyed without using / read from 2 different objects

##### Operations on "ostream_iterators"

- "ostream_iterators" cannot be empty / off-the-end character
- Can be defined for any type of output stream
- When using "ostream_iterators", can leave out dereference + increment
  - Do nothing to the iterator, but help the reader to understand the code better

#### 10.4.3 Reverse iterators

- Iterator that traverses a container backward
  - Last to first, incrementing move to the previous element, decrementing move to the next element
- All container (except "forward_list") have reverse iterator
  - "forward_list" is singly linked list
- Reverse iterator can be both const + non-const

##### Reverse iterators require decrement operators

- Reverse iterator must have access to increment + decrement
- Stream iterator cannot reverse, cannot create a reverse iterator from stream iterator

##### Relationship between reverse iterators + other iterators

- Ordinarily reverse iterator cannot be used for iterate through the container
  - Will iterate backwards
- Need to use the ("base()") member to get corresponding ordinary iterator to use tin algorithm + operations
  - The base iterator will be 1 pass the reverse iterator

### 10.5 Structure of generic algorithms

#### 10.5.1 The 5 iterator categories

- Iterators has a set of operations, some are common to all, some are specific to individual iterators
- Iterator can be categorized + form a hierarchy
  - Higher category iterator provide all operation of the lower categories
- Each generic + numeric algorithms has a minimum category of iterator
  - The iterator must be equal / higher to that category to be used
- Compiler often will not detect categories of iterator

##### The iterator categories

- Input iterators: Read elements in a sequence, contain the following operator
  - Equality / inequality "==", "!="
  - Prefix + postfix increment "++"
  - Dereference operator to read the element "*" 
    - Can also the arrow operator("->")
  - Can only be used sequentially, 1-pass operations
  - i.e. "istream_iterators"
- Output Iterators: Write elements in a sequence, contain the following operator
  - Prefix + postfix increment "++"
  - Dereference "*"
  - Can only be used for single-pass algorithm
  - i.e. "ostream_iterators"
- Forward iterators: Read + write to a sequence, can only move in 1 direction
  - Support all the operation in input + output iterators
  - Can read + write the same element multiple times
  - i.e. Iterators for "forward_list"
- Bi-directional iterators: Read + write a sequence in both direction
  - Support everything above + the prefix + postfix decrement operator "--" 
- Random-access iterators: Constant-time access to any elements in sequence
  - Support all the functionality and more
    - Relational operator "<, <=, >, >="
    - Additional + subtraction operator to advanced / retreated multiple elements in the sequence
      - Subtraction operator also shows the distance between 2 iterators
    - subscript operator "[]" to locate specify elements in the container
  - i.e. Iterator for "array, deque, string + vector"

#### 10.5.2 Algorithm parameter patterns

- Most algorithms follows a simple pattern of parameter conventions

  - ```c++
    alg(beg, end, other args);
    alg(beg, end, dest, other args);
    alg(beg, end, beg2, other args);
    alg(beg, end, beg2, end2, other args);
    ```

- All algorithm will take 2 iterators for range, depends on operation, some might require additaional information

##### Algorithms with a single destination iterator

- Algorithm which writes to an algorithm assume the destination is large enough to hold the write
- Destination is usually a insert iterators, creates new element while inserting

##### Algorithm with a second input sequence

- combine the 2 range of elements to perform computation
- Some algorithm only takes the starting iterators of the second container, assume second container is at least as big as range of input range

#### 10.5.3 Algorithm naming conventions

- Algorithms has a set of naming + overload conventions

##### Some algorithm use overloading to pass a predicate

- Predicate: Function / callable object used to evaluate elements in a container
- Function that allows predicate arguments are usually overloaded
  - Usually has a version that does not require predicate to provide context

##### Algorithms with "_if" versions

- If the algorithms takes predicate has the same # of arguments as other overloaded version, add "_if" to the end of the function to distinguish
  - To avoid any ambiguities in overloading function

##### Distinguishing versions that copy from those that do not

- Algorithm that writes a copy instead of back to the original container has "_copy" after the algorithm name
  - Usually take 1 more arguments too, to indicate the destination to copy the result to
- Algorithm with _copy can also have _if
  - Both calls a lambda, but requires different # of parameters

### 10.6 Container-specific algorithms

- "list" + "forward_list" has a set of own operations than generic algorithms
  - "sort, merge, remove, reverse + unique"
- node + linked based, cannot access randomly

##### The list-specific operations do change the containers

- Most operations are destructive to the container
  - Changes the container

