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

##### Writing custom "swap" function

```c++
class class_Name
{
    friend void swap (class_Name&, class_Name&);
}

inline void swap(class_Name& lhs, class_Name& rhs)
{
}
```

- The function should be a "friend" function to access private data member
- The function should also be "inline", for optimization issue
- Swap is not necessary to define, but an important optimization process

##### Use "swap" over "std::swap"

- For data member that are not built-in type + have custom "swap" function, should always call "swap" over "std::swap"
  - Better match than the swap from the standard library

##### Using "swap" in assignment operators

- "swap" could be used to define assignment operator
  - Copy + swap: Swap the left operand with a copy of right operand
    - The right hand operand pass by values + constructed as a new object
    - Perform a swap between the left hand operand's pointer with the new object's pointer
    - Destroy the new copied object using its destructor
- Automatically handle self assignment + exception safe

### 13.5 Classes that manage dynamic memory

- Class should usually built-in library to manage dynamic memory using a container
	- However, in some case, classes need to perform custom allocation
	- Needs to define a custom copy-control member to manage memory allocation

Dynamic memory custom class design

- The class needs to pre-allocate enough storage to hold some elements
	- Every time a new elements are added, if enough space, construct a new object at the next available spot
	- Else, find a new, bigger space in memory, move all existing elements to the new space + add the new elements
- Use “allocator” to obtain raw memory
	- Use “construct” to create object in allocated space
	- Use “destroy” to remove member from the allocated memory
- Requires 3 pointers to manage the memory
	- “first”: points to the first element of the allocated memory
	- “first_Free”: points to the first available allocated memory to place a element
	- “caps”: Points to the end of the allocated memory (after the last element)
- Class should have a static data member of “allocator<T>” to allocate
	- A memory space factory available to all objects in class
	- Will return a pointer to the first location of the allocated memory
##### Using “construct”

- The function which adds an element to the custom dynamic memory must first check if there are room for new element.
  - Memory start as unconstructed
- Takes at least2 arguments, first argument is a pointer to the first allocated, unconstructed space
  - Remaining arguments determine what object will be constructed

- Increment the first free space pointer to indicate a new element has been constructed

##### Copy-control members

- Perform custom copy-control requires 2 functions
  - An "Allocate n copy" function, which takes a range of object, allocate dynamic memory for them + use "uninitialized_copy" to copy the element to the newly allocated memory + return the first + one past last pointer in a pair
  - An "free" function, which first destroy all the stored objects from the memory + deallocate the entire memory block
    - Ensure the memory is allocated with the "allocate" function + both pointers points to the same allocated memory
- "Allocate n copy" should be called before "free" to protect self-assignment

##### Moving, not copying element during reallocation

- "string" have value like behavior
  - When copy a "string", we generate a new "string", changes to the old "string" won't affect the new "string"
- When "reallocate" copies of "strings", only 1 copy of the string will remain
  - Should consider using "move" instead of "copy" to improve performance by avoiding allocating + deallocating overhead

##### Move constructors + "std::move"

- Many library class defines a "move constructor"
  - Library guarantees that "moved from" string remain valid + destructible state
- The "utility" header defines a "move" function
  - Do not usually provide a using declaration for move, call with "std::move"

### 13.6 Moving objects

- Moving can provide significant performance boost over copy
- IO / "unique_ptr" cannot be copied, but can be moved
- Container type does not have to be copyable as long it is movable

#### 13.6.1 R-value references ("&&")

- A references that must be bound to a r-value
  - Bound to an object about to be destroyed
- L-value vs. R-value
  - L-value: Object that occupies some identifiable location in memory
  - R-value: Expression that cannot have a value assigned to it  
    - Temporary object / literals
- Cannot bind a R-value to a L-value reference (regular references)
  - Cannot bind a L-value to a R-value reference

##### L-value persist; R-value are ephemeral

- R-value reference's object is about to be destroyed
- R-value reference's object is unique
- Code can "steal" state from the r-value reference, source object will be left in a empty / default state

##### Variables are L-value

- Variable expression are L-values
  - Therefore, cannot bind a R-value reference to another R-value reference
  - The right hand operand will become a L-value

##### The library "move" function

- Possible to cast an L-value to be a R-value reference type
- Use the "move" function, returns a R-value reference to a given object
  - Calling "move" on a object indicate that the object will not be used again (unless to be assigned / destroyed)
  - Values on the moved-from object will be valid but unspecified
  - After re-assign values to the object, the object can be used as normal

#### 13.6.2 Move constructor + move assignment

- Define a custom move constructor + move assignment operator to enable move operation to a custom classes
  - Similar to the copy constructor, but steals data from the given object instead of copying
- Move constructor's initial parameter is a R-value reference to the class type
  - Additional parameter needs to have default arguments
- Move constructor needs to handle leaving the moved-from object in a harmless state
  - Remove all pointer to the moved resource
- "noexcept": signal the constructor to not throw any exceptions
- The move constructor does not allocate new memory, but transfer ownership over to another object
  - The "moved-from" object continue to exist

##### Move operations, library containers + exceptions

- Move operation does not normally allocate resource -> usually will not throw a exceptions
  - Add the "noexcept" keyword to save extra work from the library
  - Specify the "noexpect" keyword right after the parameter list
    - Between parameter list + construct initializer list in constructor
- Needs to specify "noexpert" on declaration + definition if the function is used outside the class
- Move operation usually don't throw exception, but is permitted to do so
  - Library container guarantees actions when exception happens
- Since the move operation changes the state of the moved-from object, throw exception will not usually meet the exception requirements
  - Copy operation is free to throw exception since the original object is unchanged

##### Move-assignment operator

- Move-assignment operator  = destructor + move constructor
  - Should also be set to "noexcept"
- The move-assignment operator should implement logic for self-assignment
  - Directly check if the left hand and right hand operand are the same
    - If 2 objects are the same, move operation are not necessary
- Otherwise, Free (deallocate) the memory of the left-hand side object + take over the memory address from the right hand object

##### A moved-from object must be destructible

- Needs to ensure after the move operation, the moved-from object is able to run the destructor
  - Usually by setting pointer member to nullptr
- The moved-from object must remain valid, safely to assign new value / used in other ways
  - However, does not need to guarantee the validity of the remaining value on the moved-from object

##### The synthesized move operations

- The complier is able to synthesized move constructor + move-assignment operator for a class
  - For copy constructor + copy-assignment operator, operations either define memberwise copy / assign object / deleted function
- If a class defines copy constructor / copy-assignment operator / destructor, class will not synthesize move constructor + move-assignment operator
  - When a class is missing move operations, use copy operation instead
- Class will only generate synthesize move constructor / move-assignment operator if copy control members are not defines + every non-static data member can be moved
  - Every data member either needs to be built-in type or the class type has move operations
- Cannot set move operation explicitly to be deleted
  - Operations are never implicitly set to "delete"
  - If define the move operation explicitly using the "default" keyword, and move operation is not possible (If some of the members are not movable), the complier will treat the function as deleted
  - Move operations is defined as deleted if class has member with copy constructor but not move constructor
  - Move operations is deleted if a object's copy construction is deleted
  - Move operation is deleted if destructor is not available
  - Move operation is deleted if class has cost / reference member
- If the class defines a move constructor  + move-assignment operator, synthesized copy operations will be deleted

##### R-values are moved, L-value are copied (with move constructor)

- Compiler use function matching to determine which constructor to use
  - Depends on the argument type, if arguments are R-Value (non-const)

- R-value are copied if there are no move constructor

##### Copy + swap assignment operators + move

- Move operations takes non-const parameter
  - Needs to able to modify the moved-from object
  - Move-assignment operator uses non-const, non-reference parameter
    - Parameter is copy initialized
    - Copied / moved depends on L/R-value
    - Single operator acts for copy-assignment + move-assignment
- Both copy + swap and move + swap utilize the "swap" operation between 2 operands
  - The left hand-operand will holds a pointer to a object, destroyed when going out of scope
- The rule of 3 really should be rule of 5, all 5 of them should be considered

##### Move Iterators (make_move_iterator)

- Adapt a given iterator to be able to iterate through deference operator
  - Each pointed element is a R-value reference
- Takes an iterator, return a R-value iterators
  - This iterator can be used in generic algorithms, especially to provide a range of R-value
- Compiler does not check if the algorithms can use move iterators
  - Move operation also modify the source, no guarantee the algorithm will run correctly
  - Developer needs to make sure algorithm does not access the same element twice
- Use "std::move" only when move is guaranteed safe + move is necessary

#### 13.6.3 R-value references + member functions

- All member function can benefit from providing copy + move versions of the function
  - Use the same parameters as the copy/move constructor + assignment operator
    - Const L-value reference vs. non-const R-value
  - The "move" member function is free to steal resources from the parameters

##### R-value + L-value reference member functions

- It's possible to a function on a object regardless if it's L-value / R-value
  - Use a reference qualifier to indicate return of L-value
  - Use 2 reference qualifier to indicate return of R-value reference
  - Add the reference qualifier at the end of the function declaration + definition
    - If the function has the reference qualifier, return a reference + L-value

##### Overloading + reference functions

- Possible to overload a function base on reference qualifier
  - Depends on the object type, perform differently optimized function
- If 2 function have the same name + same parameter list -> must have the same reference qualifier
