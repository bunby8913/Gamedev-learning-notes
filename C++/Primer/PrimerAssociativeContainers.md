# Primer Associative containers

- In associative containers, element stores + retrieve using a key
  - key-value pairs
  - i.e. "map" + "set"
- 8 total associative containers available
  - (map/set), multi (map/set), unordered (map/set), unordered_multi (map/set)

### 11.1 Using an associative container

- "map": Collection of key-value pairs
  - Associative array: A normal array except the subscript is not an integer
- "set": a Collection of keys: Simply to know if a value exist

##### Using a "map"

- Templates, need to specify key + value type
	- If the key is not already in the container, keys will be automatically created
- Container has 2 public data elements (first, second)
	- “first”: the key of the pair
	- “second”: the value of the key

##### Using a “set”

- “set” is also a template, just need to specify the type of the elements in container

### 11.2 Overview of the associative containers

- Associative containers support basic container operation
	- Do not support index based operation
	- Do not support constructors / insert with value + count

#### 11.2.1 Defining an associative container

- Default constructor will define an empty container of the type
	- Constructor also takes copy of another container / range of another container
	- Possible to use list initializer to initialize the container
		- Needs to be convertible to the container type

##### Initializing a “multimap” / “multiset”

- “multimap/set” remove the restriction of 1 element per key
	- several element can associate with the same key

#### 11.2.2 Requirements on key type

- Key type must define a way to compare elements
	- By defaults, associative container use “<“ to compare between keys

##### Key types for ordered container

- Possible to provide custom operation for key comparison
	- Strict weak ordering: a less than order that meets the following properties
		- 2 keys cannot be both less than each other
		- k1 < k2 && k2 < k3 -> k1 < k3
		- k1 == k2 && k2 == k3 -> k1 == k3
	- If neither key are “<“ than the other, than they are equivalent, treated as the same key

##### Using a comparison function for the key type

- The comparison operation should be part of the constructor argument (along the type of the container) to create the container
  - Needs to declare the type of the function (a function pointer type) in the constructor
    - could use "decltype" to get the type of the function

  - Initialize the container with operation after the angle bracket "<>"
    - Now elements in the container will be ordered according to the custom operation provided
    - Can just provide function name instead of a reference, automatically convert into a pointer if needed


#### 11.2.3 The "pair" type

- Part of the "utility" header
- Holds 2 elements, a template so 2 type names must be created
  - Default constructor value initializes the "pair" data members
  - Can also supply initializers for each member, wrapped in "{}"
- data members of "pair" are public
  - "first" + "second"

##### A function to crease "pair" objects

- Possible to list initialize a pair object
  - Useful to return a "pair" data structure
  - If using list initialize, do not need to specify that it's a "pair" do not need to specify the type of data member in "pair"
- Could also use a "make_pair" function to generate a new pair, automatically determine the type of the data members

### 11.3 Operations on associative containers

- Associative containers has additional type aliases
  - "key_type": The type of the key of the container
    - Since key cannot be changed after created, "key_type" is a const
  - "mapped_type": Only available to map types, the values types of the map
  - "value_type": For "sets", it's the same as "key_type", for "maps", is a pair of "pair<key_type, mapped_type>"
- Use the scope operator "::" to fetch type information

#### 11.3.1 Associative container iterators

- Deference an element of associative container returns the "value_type"
  - A "pair" of const "key" + "value"

##### Iterators for "sets" are const

- "set" type will always return a read-only element in the "set"
  - "keys" in "set" are const, cannot be changed

##### Iterating across an associative container

- Associative containers support "begin" + "end" operations
  - Those operations will provide iterators used to traverse the container
- Iterating through "map" will output result in alphabetical order
  - Ascending key order

##### Associative containers + algorithms

- Since "keys" are const, most write generic algorithms cannot be applied to associative containers
- Most read generic algorithm are useless, as keys search is more efficient then iterate through the entire containers
  - The "find()" operation will be more efficient associative containers
- Most common generic algorithms for associative container is to use the container as a whole as a destination
  - i.e. copy(), inserter(), etc.

#### 11.3.2 Adding elements

- "insert()" member add 1 or more elements to the associative containers
  - For "map" + "set", inserting element already in containers has no effect

##### Adding elements to a "map"

- To insert element to a "map", it must be inserted as a "pair"
  - "pair" often are created in the argument list of "insert"

##### Testing the return from "insert"

- If "insert" is applied to container with unique "keys", "insert" + "emplace" returns "pair" with iterator + bool
  - Iterator to the elements with the inserted key
  - The bool value indicate if the insertion is successful or not
    - False if elements is already in the container + do nothing
    - Otherwise, return true

##### Adding elements to "multiset" / "multimap"

- Add additional elements with the same key
- "insert" will always insert the element, will return the iterator

#### 11.3.3 Erasing elements

- Defines 3 version of "erase"
  - Can erase single element / a range of elements with iterator pair
  - Elements are removed, function returns "void"
  - 3rd version takes a "key_type", remove all the elements with the given key, return how many elements has been removed
    - Unique container will return either 0 or 1, if return 0, then the key to be erased is not in the container\
    - Multi-key container could remove more than 1 elements

#### 11.3.4 Subscripting a "map"

- "map" + "unordered_map" has subscript operator + "at" function
  - Cannot subscript "multimap" since there are more than 1 value associated with a key
- Similar to other subscript operation, provide a index(key) and fetches the value associated with the key
  - However, if the key is not presented, a new element with the key added to the container
    - Value initialized the pair
- Can only apply subscript operation "map" that are not const, potentially could write new elements to the "map"

##### Using the value returned from a Subscript operation

- The type returned by the "map" subscript differs from type obtained from dereference a "map" iterator
  - "map" subscript return "mapped_type" (The value of the pair)
  - dereference "map" iterator returns "value_type" (the key-value pair)
- The "map" subscript operator returns "l-value", can be used to read/write to the element

#### 11.3.5 Accessing elements

- Several ways to access particular elements in the container
  - "find(k)": return the first iterator to element with the key "k", return 1 off end iterator if not found
  - "count(k)": return the # of elements with key "k", 0 or 1 for unique container
  - "lower/upper_bound(k)": return the iterator of first element that the key is less/greater than "k"
  - "equal_range(k)": return a pair of iterator that has key "k", if k is not present, both member return "end()";

##### Using "find" instead of subscript for "maps"

- Subscript has important side effect: if the elements does not exist in the "map", elements will eb added
  - If the program intend to not alter the container, should use "find" instead

##### Finding elements in a "multimap" / "multiset"

- In "multimap" / "multiset", elements with the same key will be adjacent to each other in the container
- A simple approach
  - Find the first key using the search operation
  - Find out the number of times the key repeat in the container
  - Use an iterator to iterate the amount of time the key repeat in the container, to get all the entries with the same keys
- Alternatively, solve with "lower/upper_bound"
  - If the key exist in the container, "lower_bound" is the iterator to the first instance of the key, "upper_bound" is the iterator to the last instance of the key
    - Create an iterator range, denotes all the elements with the same key
      - Nodes the upper bound will be 1 off the end iterator
      - If the key is not present, the lower bound will also be 1 off the end iterator
    - If "lower_bound" = "upper_bound", then the given key is not in the container

##### The "equal_range" function

- Function takes a key and return a range of iterator, of the first + 1 past the last elements with the same key

### 11.4 The unordered containers

- Associative containers that uses "hash function" + "==" operator
  - Useful for key type with no obvious ordering relationship between elements
- Requires performance testing + tweaking
  - Usually easier to use ordered container

##### Using an unordered container

- Provided mostly the same operation (apart from hash managing operations)
  - Can pretty easily replace unordered/ordered container with its counterpart
    - However, the order of the element may change in containers

##### Managing the buckets

- Organized as a collection of buckets
  - Use hash function to match element to bucket
- Access element by computes the hash code, then look for the element in the bucket
  - Performance of unordered container depends on the quality of the hash function
- Hash function must always yield the same result
  - However, possible that several elements with different keys are mapped to the same bucket
    - To search for the exact pairing, iterate through every element in the bucket sequentially

##### Requirements on key type for unordered containers

- Use type "hash<key_type>" to generate hash code for each element
  - Possible to define unordered container that the key is one of the built-in type
- Cannot use hash template directly on custom class type, need to supply own version of the "hash" template
- Can alter the "hash" behavior by providing custom compare equation + calculation of hash code
