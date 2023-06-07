# Primer dynamic memory

### 12.1 Dynamic memory + smart pointers

- Manage dynamic memory through a pair of operator, "new" + "delete"
  - "new": allocate + possibly initialize an object in dynamic library + return the pointer to the library
  - "delete": takes a pointer to dynamic object, destroy the object + release the associated memory
- Surprisingly hard + to know when to free memory at the right time
  - Forget to free memory -> memory leak
  - Free memory when the pointer is still referring to the memory -> invalidate the data + pointer
- "memory" Library provide 2 smart pointer type to manage dynamic objects, differ by how they managing the pointers
  - "shared_ptr": allow multiple pointer to refer to the same object
    - "weak_ptr": A weak reference to the object managed by a "shared_ptr"
  - "unique_ptr": Single pointer, owns the object it points to

#### 12.1.1 The "shared_ptr" class

- "shared_ptr" are templates, requires additional type information to initialize
- Default initialized smart pointer holds null pointer
- Smart can be dereferenced the same way as regular pointer
- Use smart pointer in conditions test if the smart pointer is a null pointer

##### The "make_shared<T>()" function

- Safest way to allocate dynamic memory
  - Part of the "memory" header
- Specify the type of the pointer on usage
- The value passed need to match the argument to constructor an object
  - If no value is passed then value initialize
- Can use "auto" keyword to define object to hold the result of the smart pointer

##### Copying + assigning "shared_ptr"

- each "shared_ptr" pointer track how many other smart pointers points to the same object
  - An "associated counter" (reference count)
- When the "shared_ptr" reference count becomes 0, automatically free the object
- Still possible to implement pointer counter to keep track how many pointers share state
  - The associated counter is only used to automatically free object when appropriate

##### "shared_ptr" automatically destroy their objects + automatically free the associated memory

- Free memory through a special member function, the destructor
  - Each class has a destructor, controls the event when objects are destroyed
- Destructor usually destroy elements + free memory used for the elements
- A common memory leak comes from putting "shared_ptr" in a container + reorder the container so not all elements are needed
  - Should always remember erase elements no longer needed

##### Classes with resources that have dynamic lifetime

- 3 purposes to use dynamic memory
  1. Don't know how many object is needed
  2. Don't know the precise type of the object needed
  3. Share data between objects

##### Share data between object using dynamic pointer

- If 2 objects share the same set of objects, we want the data to remain even if the object that "owns" the data gets destroyed
  - Needs store the data in dynamic memory

##### Copying, assigning + destroying object utilize "shared_ptr"

- When the object holds the "shared_ptr" gets copied , assigned / destroyed, the "shared_ptr" will be copied, assigned / destroyed
  - Copying will increment the reference counter of the unique pointer
  - Assigning will increment the reference counter for 1 , decrement the reference counter for the other

#### 12.1.2 Managing memory directly

- 2 operators to allocate (new) + do free (delete) memories allocated
  - More error prone than using a smart pointer
- Classes that manage memory directly cannot use default definition to copy, assign + destroy the objects
  - Smart pointer will be easy to write + debug

##### Using "new" to dynamically allocate + initialize objects

- Dynamically allocation using "new" does not name the space being allocated

  - "new" return a pointer to the object being allocated
  - Values are default initialized, build-in type will have un-defined value

- Dynamically allocated object can initialize using direct initialization

  - ```c++
    T *pointer_Name = new T(initialization_Value);
    ```

  - To value initialize the variable, leave the initialization value as empty

- Good practice to initialize dynamically allocated objects

- the "auto" keyword could deduce the type of object to allocate memory for

  - "auto" only work with single initializer inside parentheses

##### Dynamically allocated "const" objects

- The same rule mentioned above also applies to const objects
  - Class type with default constructor may be initialized implicitly
    - All other needs to be explicitly initialized
  - The allocated object will be const + cannot be changed

##### Memory exhaustion

- Once the program used up all the available memory, the "new" operation will fail

  - If failed to allocate new memory, "new" will throw a "bad_alloc" exception

  - ```c++
    T *pointer_Name = new (nothrow) T; // If allocation failed, return a null pointer
    T *pointer_Name = new T; // Will throw the bad_alloc exception
    ```

- Placement new: Pass additional argument to new, i.e. "nothrow"

##### Freeing dynamic memory

- Delete dynamic allocated memory to avoid memory exhaustion

  - ```c++
    delete pointer_Name; // pointer_Name if a dynamically allocated pointer / null
    ```

##### Pointer values + "delete"

- Deleting pointer that are not dynamically allocated / deleting the same pointer more than once is undefined behavior
  - Once the pointer is freed, any pointer that points to the freed pointer will also be free, freeing it again will be undefined behavior
- Possible to delete const dynamic object

##### Dynamically allocated object exist until they are freed

- Dynamic object managed manually (using built-in pointer) exist until its explicitly deleted
  - Caller must delete the object + free the memory
- Even when the local variable associated with the dynamic object goes out of scope, the dynamically allocated object still exist
  - Once out of scope, the program have no way to free that memory
- Either remember to delete the pointer / return the pointer being allocated
- Deleting the same memory twice is also a frequent error

##### Resetting the value of a pointer after a delete provide only limited protection

- "delete" a pointer will set the pointer to be invalid
  - Although it may still points to the freed address
- Dangling pointer: A pointer that points to an memory that had a object but the object no longer exist
  - Like a uninitialized pointer
  - To avoid Dangling pointer, should always delete the dynamic object right before pointer goes out of scope
  - Assign "nullptr" to the pointer after "delete"
- Fundamental problem: Multiple pointers can points to the same address
  - Deleting a dynamic allocated object could create several dangling pointer
  - Surprisingly difficult to detect all the dangling pointers

#### 12.1.3 Using "shared_ptr" with "new"

- Can initialize a "shared_ptr" from pointer returned by "new"

  - Cannot convert a implicitly built-in pointer to a smart pointer

  - ```c++
    shared_ptr<T> variable_Name = new T(argumetns); // error, cannot assign built-in pointer to smart poitner
    shared_ptr<T> variable_Name(new T(argument)); // good, must use direct initialization
    ```

- Same in return statement, cannot replace smart pointer with plain pointer

  - Needs to explicitly bind a "shared_ptr" with a plain pointer


##### Don't mix ordinary pointers + smart pointers

- "shared_ptr" can only manage destruction with other "shared_ptr" that are copies
  - Should always use "make_shared" to make new "shared_ptr" over "new" keyword

- The "new" keyword could create a new "shared_ptr" on the address that already has a "shared_ptr", but does not increment the pointer count
  - Once the new counter destroyed, the original counter will also becomes invalidate

- Dangerous to use built-in pointer to access smart pointer object

##### Don't use "get" to initialize / assign another smart pointer

- The "get" function is a part of the smart pointer, return a built-in pointer points to the object the smart pointer is managing
  - Intended only to use in the case the argument does not accept smart pointer
- Cannot use the built-in pointer from "get" function to initialize / assign another smart pointer

##### Other "shared_ptr" operations

- "reset": Assign a new pointer to a "shared_ptr"
  - Combined with the "unique" function, to first check if the pointer is unique, then change the underlying object

#### 12.1.4 Smart pointer + exceptions

- Resources needs to be properly freed after a exceptions
- Smart pointer ensure that memory is freed even when the block exited prematurely
  - Smart pointer will check the reference counter, and remove the pointer accordingly
- If a built-in pointer is created ("new"), but reached an exception before "delete", the memory won't be freed

##### Smart pointers + dumb classes

- Most C++ classes has destructor to cleaning up resources allocated
  - However, some class for both C + C++ require user to specify methods to clean up
    - Classes that allocate resources + does not free them when destructed
- Use Smart pointer to point to those object to avoid memory leak

##### Using our own deletion code

- By default, when the "shared_ptr" is destroyed, execute delete on the object the pointer holds

- However, can define a function in place of "delete"

  - ```c++
    void end_function (connection *p) {function_To_End(*p);}
    ```

  - The function needs to take the "connection *" type

  - The "delete" function needs to be passed as optional argument

  - ```c++
    shared_ptr<T> p(&c, end_function);
    ```

#### 12.1.5 "unique_ptr"

- "unique_ptr" owns the object it points to

  - Each object can only have 1 "unique_ptr"

- Do not use "make_shared", instead bind to the pointer returned by "new"

  - Must use direct initialization

- "unique_ptr" does not support copy / assignment (ordinarily)

  - Can transfer ownership from 1 "unique_ptr" to another

    - ```c++
      unique_ptr<T> p2(p1.release()); // Release the pointer, p1 is now null
      p2.reset(p1.release()); // Transfer over the control from p1 to p2
      // Reset remove the address p2 previously points to
      ```

    - "release" function break the connection from "unique_ptr" to the object, return a pointer to the object

      - Can be used to initial / assign another smart pointer
      - The pointer can transformed to a built-in pointer, require manual management

##### Passing + returning "unique_ptr"

- Possible to copy / assign "unique_ptr" that are about to be destroyed
  - i.e. When the "unique_ptr" being returned from an function
  - The compiler will perform a special copy operation

##### Passing a "delete" to "unique_ptr"

- Can also override the default "delete" in the "unique_ptr"

  - However, change the default "delete" change the type of "unique_ptr" + how the "unique_ptr" being constructed

  - Needs to supply the delete type (callable object) in the "<>" with the "unique_ptr" pointed type

  - ```c++
    unique_ptr<objT, delT> p (newObj T, fcn);
    ```

#### 12.1.6 "weak_ptr"

- A smart pointer that does not manage the lifetime of the object
  - Point to a object managed by a "shared_ptr"
- Does not change the reference count of the "shared_ptr"
  - Object will be removed when the last "shared_ptr" disappears, even if there are "weak_ptr" points to it
- "weak_ptr" cannot access object directly, have to use the "lock" function
  - "lock": Checks if the object the pointer points to still exist

##### Checked pointer class

- Use "weak_ptr" if the access won't affect the lifetime of the object + ablet to prevent user from access the object if it no longer exist
- If the object is no longer there, "weak_ptr" will return a null pointer + able to throw exception

### 12.2 Dynamic arrays

- Allocate storage to multiple objects at once, i.e. "string" + containers
- 2 method to allocate array of object at once
  - "allocator": separate allocation from initialization
    - Better performance + more flexible memory management
  - Or simply use a in-class container for the application
- Most application should use library container > dynamically allocated array

#### 12.2.1 "new" + arrays

- Specifying the # of object in "[]" using the "new" keyword
  - Allocate # of objects, return the pointer to first
- Also possible to allocate array using type alias "typedef"

##### Allocating an array yields a pointer to the element type

- Does not allocate an array type, but pointer to a series of elements in elements type

##### Initializing an array of dynamically allocated objects

- All elements are default initialized
  - To value initialized the element, add an "()" behind the size
  - Or provide a list of input elements for each elements in array
    - If more element are supplied to the array, the initialization fails

##### Legal to dynamically allocate an empty array

- Dynamically allocated empty array has 0 elements
  - Return an valid, nonzero pointer
  - Cannot be dereferenced
  - An off-the-end pointer
  - Allocate 0 objects

##### Freeing dynamic arrays

- Use a special form of "delete", include "[]" after the "delete" function 
  - Indicate the pointer is the first element of an array
    - Otherwise, undefined behavior
- Mistakes not usually warned by the compiler

##### Smart pointer + dynamic arrays

- Use "unique_ptr" to manage a dynamic array
  - Can use the release function automatically call "delete[]" on the array
  - Cannot use "." + "->" operation
  - Can use subscript operator "[]" to access element in array
- Not supported by "shared_ptr" natively, needs to support custom delete operation
  - Use "get" to covert to built-in pointer, access through the that pointer

#### 12.2.2 The "allocator" class

- "New" combines allocating + constructing into 1 function
  - In real life setting, prefer to decouple allocation form construction
- Part of the "memory" header, a template class
  - Needs to specify the type can allocate, do define the size of element

##### "allocators" allocate unconstructed memory

- Use the memory by constructing object into the memory
- Use the "construct" member to use pointer + 0 / more arguments
  - Arguments provide initializer for the object
- Cannot use the raw memory that has not been constructed
- Each elements needs to be separately deconstructed after usage
  - Using the "destroy" function, which takes the pointer to that element as argument
  - Only elements that has been constructed can be destructed
- Memory can be freed using "deallocate" function, takes the same argument as the "allocate" function
  - Cannot point to null pointer, must be part of the allocated array

##### Algorithms to copy + fill uninitialized memory

- Algorithms to construct objects in uninitialized memory

  - ```c++
    auto p = uninitialized_copy(iterator1, iterator2, allocatedPointer);
    uninitalized_fill_n(p, iteroator.size(), T);
    ```

  - The copy function takes a range using iterators, must be applied to unassigned memory

    - Construct objects at the destination

  - The fill function takes pointer to destination, count + value

    - Construct "count" # of objects at "destination" with the value of "value"





