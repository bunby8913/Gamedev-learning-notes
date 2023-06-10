# Primer Copy control

- Class contain special member function to control "copy constructor", "copy assignment operator", "move constructor", "move-assignment operator" + "destroy"
  - Compiler will auto generate missing operation if not defined

### 13.1 Copy, assign + destroy

#### 13.1.1 The copy constructor

- Copy constructor's first parameter is a reference to the class, and have no other parameters

  - Additional parameter are default value

  - ```c++
    class_Name(const class_Name&);
    ```

  - Almost always a const reference to the class

- Usually the copy constructor should not have the "explicit" keyword

  - "explicit": Used by single argument constructors, avoid accidental conversion from 1 type to the class type, lead to unexpected behavior 
  - Use "explicit" will prevent compiler using copy constructor

##### The synthesized copy constructor

- Synthesized unless we explicitly define a copy constructor
- Synthesized copy constructor perform member-wise copies
  - Class type are copied with copy constructor of the class
  - Built-in type are copied directly
  - Each element of the array are copied, re-constructed into an array

##### Copy initialization

- Direct initialization: The compiler use function matching to select a constructor to construct the object
- Copy initialization: Compiler copy the object on the right to the object being created on the left
  - Usually copy using the copy constructor, sometime the compiler could decide to use the move constructor
- Some class type use copy initialization to create new object
  - "insert / push" member of container

##### Parameters + return values

- Parameter + return type of non-reference type are copy initialized
  - Copy initialize the result to the call site

##### Constraints on copy initialization

- Cannot implicitly use an "explicit" constructor to pass argument / return value

##### The compiler can bypass the copy constructor

- Compiler can decide to not use the copy + move constructor
  - It must be available for the compiler at the point of the program

#### 13.1.2 The copy-assignment operator

- Compiler will synthesizes a copy-assignment operator if not explicitly defined

##### Introducing overloaded assignment

- Functions with the name "operator" + symbol to be defined
  - Requires return type + parameter list
- Some operands needs to be member function (to define operands between the same object)
- Assignment operators should usually return the left-hand operands

##### The synthesized copy-assignment operator

- Assign each non-static member of the right-hand object to left-hand object, using the copy-assignment operator for each type
- Each element of the array are assigned individually
- Returns a reference to the left-hand object

#### 13.1.3 The destructor

- Free resources used by object + destroy non-static data member

- Function has a prefix of "~"

  - ```c++
    class class_Name
    {
        class_Name(); // default constructor
        ~class_Nam(); // destructor, cannot be overloaded
    }
    ```

##### What a destructor does

- First execute the destructor function body + destroy member in the reverse order they are being initialized
  - Object destruction is implicit
  - Class type will run their own destructor
  - Smart pointer has destructors, smart are automatically destroyed during destruction

##### When a destructor is called

- Called whenever the object of the type is destroyed
  - When variable go out of scope
  - When an object is destroyed, all its member object is also destroyed
  - When an container is destroyed, all its element are also destroyed
  - When an pointer is deleted using "delete"
  - Temporary object at the end of the full expression
- Destructor are run automatically, usually don't have to worry about calling it
  - Most of the time, only have to worry about manually manage built-in pointers + reference, to avoid memory leak

##### The synthesized destructor

- Compiler defines a synthesized destructor if not explicitly defined
  - An empty function body

- Note: The destructor does not destroy the member directly, it's done implicitly at the destruction phase

#### 13.1.4 The rule of 3/5

- No needs to define all of the copy / move assignments
  - However, they should be defined in group/pairs

##### Destructors + copy + assignment

- More obvious to determine the need of destructor over copy + assignment
  - If a class needs destructor, most likely will also need copy + assignment
- If we allocate dynamic memory in constructor, need destructor to free the allocated memory
  - If use default synthesized version of copy control + copy-assignment could lead to serious error
    - The pointer is copied, multiple pointers now points to the same address, The same address will be deleted twice since there are 2 pointers, error

##### Copy + assignment

- Class that needs copy also needs assignment
- Defines how object are copied + how values are assigned

#### 13.1.5 Using "= default"

- To explicit ask the compiler to use the synthesized version of the copy-control member

  - ```c++
    class class_Name
    {
        class_name() = default;
    }
    ```

- Can also use to specify definition + operand overload

#### 13.1.6 Preventing copies

- There are cases where class needs to prevent copies + assignment from object
  - i.e. The IO stream library

##### Defining a function as deleted

- Deleted function: Function declared to cannot be used

- Define a function to be deleted by assigning the keyword "delete" to function

  - ```c++
    class_Name(const class_Name&) = delete;
    ```

- The "deleted" needs to be used to declare the function

  - Compiler needs to know the deletion of the function to generate code

##### The destructor should not be a deleted member

- If destructor is "deleted", won't be anyway to destroy object of the type
- Possible to create the object through dynamically allocation, however, cannot free the object

##### The copy-control members may be synthesized as deleted

- In certain situation, compiler will declare certain synthesized member as deleted
  - If the destructors/copy constructor / copy-assignment operator is inaccessible / class contain deleted destructor member/deleted copy constructor/ copy-assignment operator
  - default constructor is defined deleted if
    - class has deleted / inaccessible destructor
    - Reference member not initialized
    - const member without default constructor + no in-class initializer

##### "private" copy control

- Copy constructor + copy-assignment operator can defined as private
- If left blank, even friend class cannot access the copy control No code to implement
  - Will likely result in compile time error
- Similar effect as to set the function to be "delete"

### 13.2 Copy control + resource management

- Resource management: Classes that manage resources of objects in another class
  - Must define copy-control members
    - Need destructor to clean up resources allocated, therefore, will most likely also need
- Copy the object either by value / pointer
  - Copy by value copy creates separate state, values are independent from each other
  - Copy by pointer shares the same address, Changes to the copy affect changes at original

#### 13.2.1 Classes that act like values

- Each object copies the resource the class manage
  - Object requires the copy constructor to copies value instead of pointer
  - Destructor to free the object
  - Copy-assignment operator to free existing object + copy from right-hand

##### Value-like copy-assignment operator

- Assignment operator: usually function like destructor + copy constructor
	- Ensure the sequence is correct for object assigned to self
		- Usually create the new object + delete the current value / pointer address + re-assign pointer to a new location + return the object
		- If delete the object first + using the assignment operator on self, pointer will point to a invalid / deleted address

#### 13.2.2 Defining classes that act like pointers

- Use “shared_ptr” to make class act like pointer when managing resource
	- Don’t have to worry about freeing up the memory after using a “shared_ptr”
		- Otherwise, require implementing a “reference count” to free memory after the last pointers goes away

##### Reference counts

- The class constructor creates a reference counter, keep track how many object share states with the object being created
	- Initialize the counter to 1
- Copy constructor copies data member of the object (include the counter)
	- Every time the object is copied, increase the reference counter by 1
- Destructor decrement the reference counter
	- If the counter is 0, delete the state
- Copy assignment operator increment counter on the right + decrement counter on the left
	- If the left-hand object has counter of 0, destroy the state of the left-hand operand
- Store the reference counter independent from the object, in dynamic memory
	- When creating an object, allocate a new reference counter
	- All object points to the same reference counter

##### Pointerlike copy member “fiddle” the reference count

- Destructor require conditional to delete reference data member
	- Only decrement the reference counter if it’s not 0
- Copy-assignment operator needs to handle self-assignment
	- Increment the counter on the right hand side before decrement on the left hand side
		- Handle the counter change before deciding if the reference should be deleted

### 13.3 Swap

- “swap”: A very important operation if classes needs to implement algorithms to re-order elements
	- Use class-specific “swap” if defined, if not, use the “swap” function from library
	- In many cases, “swap” operation could be performed on pointer level, swap the pointer between 2 objects
		- Does not require memory allocation

##### Writing our own “swap” function

