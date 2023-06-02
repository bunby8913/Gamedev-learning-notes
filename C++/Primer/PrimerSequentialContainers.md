# Primer Sequential containers

- Container: Holds a collection of objects of a specific type
	- Sequential container: The programmer is able to control the order of the elements in the container
		- Order does not depend on the value of the element

### 9.1 Overview of the sequential containers

- C++ offer many different types of sequential containers
	- Trade offs in cost to add/delete / access element non-sequentially
- “vector”: flexible-size array
	- Stores element in contiguous memory
	- Fast random access
	- Slow inserting/ deleting in the middle as all the element after have to be relocated to different memory locations
- “deque”: double-ended queue
	- More complicated data structure
	- Fast adding/removing element at either end (front + end)
	- Fast random access
- “list”: doubly-linked list
	- Higher memory overhead
	- Bi-directional sequential access
	- Do not support random access
	- Fast insert/delete at any point in the container
- “forward_list”: singly-linked list
	- For hand-written, singly linked list
	- Higher memory overhead
	- Sequential access in 1 directions
	- Do not support fast access 
	- Fast insert/delete at any point in the container
- “array”: fixed-sized array
	- Safer, easier to use + alternative to built in array
	- Fast random access
	- Cannot add/ remove elements
- “string”: special types of container, similar to “vector”
	- Hold elements in continuous memory
	- Only contains character
	- Fast random access
	- Fast insert/delete at the end

##### Deciding which sequential container to use

- Usually best to use vector unless for specific reasons
- If contains many small elements + space overhead matters, avoid “list” + “forward_list”
- If requires random access element, use “vector” / “deque”
- If requires to insert element in the middle, use “list” / “forward_list”
- If only needs to insert element at front + back, use “deque”
- If requires insert element in the middle + access middle element in the container
	- Consider if inserting in the middle is necessary, if must do so, consider using a list in input phase, than once input over, copy all element to a vector
- Depends on the trade off between # of random access operation vs. # of insert + delete element in the middle, use testing to determine which container to use 

### 9.2 Container library overview

##### Constraints on types that a container can hold

- Container can hold almost all types
	- Include another container (2D container)
		- Older container may require spacing at the end angle brackets
- Container operation can only be applied to element types that meets operation’s requirements
	- If the contained types does not have a default constructor, we can construct container with the type, but cannot construct object using only element count
		- Element initializer must be supplied

#### 9.2.1 Iterators

- Iterator have common interface
	- Return reference of the element
	- Deference operator
	- Increment / decrement operator to access the next / previous element
		- 1 exception, “forward_list” cannot use decrement operator
	- Equality / inequality operator

##### Iterator ranges

- Pair of iterators, mark range of element from the container
	- begin, end
		- End is a misleading term as in many cases, end refer to 1 pass the last element
- Both iterators needs to refer to the same container + possible to reach the end by repeatedly increment begin
- Left-inclusive interval: Standard mathematical notation to indicate ranges
	- Include the first element, but exclude the last element, stops 1 before the last element

##### Programming implications of using left-inclusive ranges

- left-inclusive ranged have 3 important properties
	- if begin == end, range is empty
	- is begin != end, at least 1 element in range
	- begin can be incremented until = end

#### 9.2.2 Container type members

- size_type, iterator, const_iterator
- reverse iterators: iterator that goes backwards through the container
- Use type aliases to define iterator to get element stored in container without knowing the type

#### 9.2.3 “begin” + “end” members

- “begin” + “end” operations produces iterator of the first + last element of the container
	- To create reverse iterators, use the “rbegin” + “rend”
	- To create const iterator, use the “cbegin” + “cend” operations
		- Preferred when writing access is not needed
- All operations (apart from the const iterator operations) are overloaded
	- Return iterator if the container stored const object
		- Able to convert “iterator” object to a “const_iterator” object
	- Return “const_Iterator” if the container stored const object

#### 9.2.4 Defining + initializing a container

- Container has default constructor (apart from array)

##### Initializing a container as copy of another container

- direct copy of the container / copy range of element, indicated by pair of iterator
- To copy container, container + element type needs to match
	- If copying range of element using iterators, just need element type + container type can be convert to one another
		- New container will have same size as the # of element in range
		- Can be used to copy a sub-set of sequential elements fro an vector

##### List initialization

- List explicitly specify value of each element in container
- List implicitly specify the size of container (exclude “array”)

##### Sequential container size-related constructors

- Can initialize sequential containers (exclude “array”) using size + default element initializers
	- If default element initializers not supplied, use default value-initialized
- ``` C++
	vector<T> variable1 (size, initial_Value); // size amount of elements, all initialized to initial_Value
	forward<T> variable2 (size)
	// size amount of element, all initialized to 0
  ```

##### Library arrays have fixed size

- When defining library array, needs specify size in initialization
- ``` C++
	array<T, size>; // array is T type with size of “size”

- Size if part of array’s type
	- Default array will not be empty, all elements are default initialized;
	- To list initialize an array, the list needs <= size of the array
	- array’s type must support default constructor, permit value initialization
- As long array type match, can copy + assign array to another array

#### 9.2.5 Assignment + swap

- Assignment-related operator (“=“), replace range of element on left with copy of element on right
	- Same size after assignment, same element after assignment
- for library “array” type, support assignment, but does not support assign using braced list of values

##### Use “assign” (sequential container only) (“assign()”)

- Left + right hand side must have the same type for assignment
	- Sequential container allows for implicit conversion + assign from subset of right hand container using pairs of iterators
- Iterator must not be from the container being “assigned”
- “assign()” also can take int value + element value
	- Essentially to re-initialize the value with # of elements, all initial ideas to the elements value

##### Using swap

- Exchange content of 2 containers of the same type (exclude “array”)
	- Do not copy, delete / insert element, guarantee to run in constant time
- Swapping array will exchange element
	- Will run in linear time
- Good practice to use the non-member version of swap

#### 9.2.6 Container size operations

- Container have 3 size operations
	- “size”: the # of elements in the container
	- “empty” returns a bool to indicate if size if 0 or not
	- max_size: the max amount of element the container can contain

#### 9.2.7 Relational operators

- All containers support equality + relational operators
	- must be the same kind of container + same type of element on both end
- Perform pair-wise comparison
	- If container are same size + same elements, they are equal
	- If containers have different sizes, but corresponding element are the same, the container with less elements are smaller
	- If container have same initial element values, the first unequal elements determine which is bigger

##### Relational operators use their element’s relational operator

- If element type does not support operational operator, container contain those elements cannot be compared
  

### 9.3 Sequential container operations

#### 9.3.1 Adding elements to a sequential container

- All library container provide flexible memory management (exclude array)
	- Elements can be added / removed + size of container can be changed at run time
- Adding elements to “victor” / “string” may require re-allocation for new memory, moving elements from old space -> new space

##### Using “push_back”

- Every sequential container support push_back (apart form “array” + “forward_list”)
- Creates new element at the end of the container + container size + 1
	- Container uses copy of object, not object itself, no relationship between container’s element + original elements

##### Using “push_front”

- “list”, “forward_list” + “duque” supports “push_front”
- Creates new element at the front of the container + container size + 1

##### Adding elements at a specified point in the container (“insert()”)

- insert 0 or more elements any where in the container
- Supported by all container (exclude “array” + “forward_list” have a special implementation)
- “insert” function takes iterator as first argument
	- Defines the container the elements going to be inserted
		- Any position in the container + 1 after the last element
	- “insert” operation insert the element before the iterator
- “insert” can be used when “push_front” is not available (“vector” + “string”)

##### Inserting a range of elements

- An overloaded version of “insert” operation
	- Can include an int value, indicate # of identical elements added before the given iterator position
- Inset can also take initializer list to insert element right before the iterator position
	- Cannot use current container’s iterator for insert
- Returns the iterator of the first element inserted, if range is empty, return the first argument iterator

##### Using the return from insert

- Can use the value returned by the “insert” operation to repeatedly insert at the same location

##### Using the emplace operations (“emplace()”)

- construct instead of copy element
	- Pass the argument to element types’s constructors, construct the object directly in the space
	- Argument must match the argument of the constructor of the element type
- 3 emplace members
	- “emplace”: equivalent to “insert” operation 
	- “emplace_front”: equivalent to “push_front”
	- “emplace_back”: equivalent to “push_back”

#### 9.3.2 Accessing elements (front() + back())

- Access operation are un-defined when the container is empty
- All container has a “front” member + all container has a “back” member (exclude “forward_list”)
	- reference to the first + last element
- Remember that “end” if the 1 past the last element, to get the last element, need to move the iterator back by 1
	- Needs to check if container is empty, before looking for “front” + “back” elements

##### The access members return references

- When access element through operation, subscript + at, receive the reference of the element
	- Change the value of reference will change the value of the element
- If use auto to store the element, only store the value of the element, changes the stored unit does NOT change the element

##### Subscriptions + safe random access (“[]”) + (“at()”)

- string, vector, duque + array offers subscript operators (“[]”)
	- Takes index, return the reference of the element at that position in the container
		- Does not check validity of the index, developers needs to check index is in range
- The (“at()”) operation to check the validly of the index, if the index is invalid, operation throw a “out_of_range” exception

#### 9.3.3 Erasing elements

##### The “pop_front” + “pop_back” members

- Developer needs to make sure the element exist before removing
- “vector” + “string” has no “pop_front” operation
- “forward_list” has no “pop_back” operation
- the “pop” operation returns void, store the popped element beforehand

##### Removing an element from within the container (“erase()”)

- Erase delete a single element (using 1 integrator) / range of elements (using pair of iterators)
	- Return an iterator of the one after the last element deleted

##### Removing multiple elements (“clear()”)

- 2 ways to remove all elements in container
	- “clear()”: Delete all elements within the container
	- “erase()”: Use a pair of begin + end iterators of the container

#### 9.3.4 Specialized “forward_list” operations

- To add / remove an element in “forward_list” will require access of the pre-recess or elements
	- Hard to access in a singly linked data structure
- Therefore, add / remove elements in “forward_list” is performed on the element after the current 1
	- Have access to the element affect by the changes
- “forward_list” defines “insert_after”, “emplace_after”, “erase_after”
	- To add/ remove to the beginning, “forward_list” defines a “before_begin” iterator, 1 off the beginning element

#### 9.3.5 Resizing a container （”resize()”)

- Use “resize()” operation to change the size of container (exclude arrays)
	- If current container is smaller than the new size, add new elements to the back
	- If current consider is bigger than the new size, remove elements from the back
- Takes additional arguments to assign a initialization value to the new elements
	- For class type, must supply an initializer

#### 9.3.6 Container operations may invalidate iterators

- Add/remove elements can invalidate pointers / reference + iterators
  - When they no longer denotes to an element, like an uninitialized pointers

- Adding elements to a container could result in invalidate references the following way
  - Iterator/ pointers / reference to "vector" / "string" become invalidate if container has been relocated
    - If not relocation, indirect reference to element before the inserting are valid, not after

  - Iterator/ pointers / reference become invalid for "deque" if insertion happens, otherwise, only the iterator becomes invalid
  - Iterator/ pointers/ reference remain of "list" + "forward_list" valid after insertion

- Removing element from a container could result in invalidate references the following way
  - Iterator/ pointers/ references of "list" + "forward_list" remain valid
  - For "deque", Iterator/ pointers/ reference becomes invalid if removed middle elements, otherwise, if remove the back element, the back + 1 element become invalid, if remove from the front, all remain valid

- For "string" + "vector", all elements remain valid, except the last + 1 iterator
- Iterators should be re-generated after insert + remove elements from the container
  - Especially vector + string + deque


##### Writing loops that change a container

- Loops that changes the container must refresh the container every iteration
  - Container use "insert" + "erase" will return new iterator (easy)
    - For "insert" operation, move the iterator up by 2, "insert" operation operate the element 1 before

##### Avoiding storing the iterator returned from end

- Any add/ remove element operation will invalidate the end iterator
  - Loop should always call the "end" iterator instead of storing it

### 9.4 How a “vector” grows

- "vector" elements are stored contiguously
  - Adjacent to each other in memory
  - If there are no room for new element in memory block, the container must re-allocate to a different area
    - Allocate new memory to hold existing elements + new elements
    - Move the element from old memory location + add the new elements
    - De-allocate old memory space
- Library will usually assign more memory to "vector" + "string" beyond what currently needed

##### Members to manage capacity ("capacity()" + "reserve()")

- "capacity()": Return # of elements the container can hold before re-allocation
- "reserve()": tells # of elements the container expect to hold
  - Does not change # of elements currently in container
  - Only changes if the requested space > current capacity
  - Library may allocate more than requested amount to the container
  - If requested # of element < current capacity, do nothing
- "shrink_to_fit()" to reduce the size of container, release unneeded memory
  - Not guaranteed to release memory

##### "capacity" + "size"

- "capacity": The # of element the container can hold
- "size" : the # of elements the container currently holds
- Memory allocation won't happen until its forced to do so

### 9.6 Container adaptors

- Sequential container adaptors: "stack", "queue", "priority_queue"
- Adaptor: Making an existing container type, make it act like a different type

##### Defining an adaptor

- Adaptor has 2 constructor
  - Empty constructor, creates an empty object
  - Take a container + initialize the adaptor by copying another container
  - Can implement an adaptor container on top of the existing element in another container
- "array" + "forward_array" cannot be used with adaptors (needs to add/ remove elements)

##### Stack adaptor

- Part of the "stack" header

- First in, last out

- No access to the original container operations

- ```c++
  stack<T> stack_Name; // empty stack declaration
  ```

- Stack can perform "pop() (Remove but do not return)", "push()", "emplace()" + "top()(Return but do not remove)" operation

##### The queue adaptors

- "queue" + "priority_queue" both part of the "queue" header
- Support "pop()", "front()", "back()", "top()", "empalce()" + "push()"
- First in, first out storage

