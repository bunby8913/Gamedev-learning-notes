# Specialized library facilities

### The tuple type

- Very similar to the "pair" data structure
  - Instead of 2 members, "Tuple" can contain any numbers of member
  - Can store elements of different type
- The type of tuple is determined by the # of element + the type of each element
- A "quick + dirty" data structure
  - Combine some data without defining a data structure
- Part of the "tuple" header

#### Defining + initializing "tuples"

- Needs to define the type of each "tuples" on definition

- To initialize, use the "tuple" constructor, value initialize each member

  - Alternatively, use direct initialization syntax

    - ```c++
      tuple<T1, T2, T3> tuple_Var{var1, var2, var3};
      ```

  - Alternatively, use the "make_tuple" function to create a tuple object

    - The compiler use the argument type to set the type of the "tuple"

##### Assigning the members of a "tuple"

- "Tuple" elements are unnamed
  - Access by the "get" function
    - Needs the user to provide explicit template argument (the index of the element we are trying to access)
      - Must be in constant expression integer
  - Use "typedef" + "decltype" to determine the type of the tuple
  - Use the public member from "tuple" "value" determine the position of the members
  - The "tuple_element" template takes a index + a tuple type, use a public member "type" to determine the specific type of the element

##### Relational + equality operators

- Similar rules to operation on containers applies
  - Both "tuples" must have the same # of arguments
  - Must be legal to compare each element using the "==" / "<"

#### Using a "tuple" to return multiple values

- A common use of "tuple" is to return multiple values

##### A function that returns a "tuple"

- Needs to define the "tuples" before using it as a return argument for a function

##### Using a "tuple" returned by a a function

- Once the "tuple" is returned by the function, it can be processed further
  - Running through a for loop to iterate each "tuple"
    - Use constant expression index to access each element in the tuple
- Each data member within the "tuple" could be accessed with a const reference if not trying to change the content

### The "bitset" type

- A library to make using bit operations easier
  - Dealing with bit length longer than the longest integral type

#### Defining + initializing "bitsets"

- A class template, takes a integral constant expression, used to define how many bits the type contain
- Bits in "bitset" are not named
  - Referred positionally, starting at index = 0
  - Ranged form low-order bit (0) to high-order bit(31)

##### Initializing a "bitset" from an "unsigned" value

- Any Integer value will be converted to a "unsigned long long"
  - Remaining high-order bit will be set to "0"
  - If the input value is larger than the "bitset" size, only store the lower-order of the input, discard the high-orders

##### Initializing a "bitset" from a "string"

- Can use "string" / pointers to character array
  - Represent bit pattern directly
  - If string is shorter than "bitset" high-order will be set to "0"
  - Sub-string can also be used to m
- The index of "string" + "bitset" are inversely related

#### Operations on "bitset"

- Ways to set / test 1 or more bits
- Many operations are overloaded
  - No arguments will apply operation on the whole set
  - Position arguments will apply operation only on the given bit
- The "size" function returns a constant expression, can be used whenever a constant expression is needed
- The subscript "[]" is overloaded by "const"
  - The non-const version return a special type of "bitset", allow bit manipulation at the index position
  - The const version simply return the state of the given index

##### Retrieving the value of a "bitset"

- 2 Available operations to return a "bitset" value to a integral values
  - "to_ulong": Covert the "bitset" to a unsigned long value
  - "to_uulong" Covert the "bitset" to a unsigned long long value
  - The size of the "bitset" must be smaller than the size of unsigned long (long) type
    - Over the limit will result in a "overflow_error" exception
- Otherwise, the "bitset" can be covert to a "string"

##### "bitset" IO operators

- The input operator can be used to read input to the put in a "bitset" object
  - Until a character other than "0" / "1" is read
  - Or encounter a "end-of-file" / input error
  - Read as many character as the size of the "bitset" to be written to
    - If there are fewer character, high-order bits will be 0
- For output operation, can covert the "bitset" object to a string, and then apply the output operator

### Regular expressions

- Describing a sequence of character
- Part of the "regex" library
- Have 4 available operations
  - "regex_match": Matches a sequence of character with a regular expression
  - "regex_search": Find the first sequence of the regular expression
    - Return true / false
  - "regex_replace": Replace a regular expression with the new format
  - "regex_iterator": Use "regex_search" to iterate through a string, find all matches
- Regex functions could be overloaded by passing an additional arguments to store additional information on the found match

#### Using the regular expression library



### Random numbers

- "rand" used to be standard function to generate random number
  - Produce pseudorandom integer uniformly distributed between 0 and a maximum value
  - Only produce random number in 1 range + 1 type
- Now use the random-number library
  - Part of the "random" library
  - Uses a random-number engine + random-number distribution classes
    - Engine generate a sequence of number
    - The distribution use the engine-generated number to covert to a specific type given a range
      - Using probability distribution

#### Random-number engine + distribution

- The engine is a function-object class

  - Takes no argument, return a random unsinged number

    - ```c++
      default_random_engine var; // Create a random engine object
      var(); // to produce the next random number
      ```

- Library provide several different random-number engines, defer in performance + quality of randomness

  - The output of the engine should not be used directly (raw random numbers)
    - Require transformation to be within range

##### Distribution types + engines

- ```c++
  uniform_int_distribution<unsigned> distVar(minVal, maxVal);
  ```

  - Uses inclusive range

- Also a function-object classes

  - Override the call operator to take a random-number engine as an argument

- Pass the engine object itself, not the value generated by the engine

  - Would result in a compile time error otherwise

##### Comparing random engines + the "rand" functions

- The output of "default_random_engine" is similar to the old-school "rand" functions
  - A number between 0 + a system-defined range

##### Engines generate a sequence of numbers

- Random number generator will return the same number every time it runs
- To truly randomize the output, both engine + distribution needs to be a "static" object
  - Use the same engine + distribution to continue to generate more random # instead of restarting

##### Seeding a generator

- To provide different output every run, use a "seed"
  - A value that the engine can use to generate # at different location in the sequence
  - Seed can either be created by the user / call the engine's seed member
    - Provided as an argument to the engine
    - Also can create a engine first + assign the seed value after
  - The same seed value will generate the same result
- Hard to pick the correct seed value, the most common approaches is to use the system's "time" function
  - If the "time" arguments is 0, return the current time in second
    - Does not work if the program is repeated multiple times per second

#### Other kinds of distributions

##### Generating random real #

- Produce a random # between 0 -> 1
- Incorrect to just divide the "rand" result by "RAND_MAX", not as precise, some floating # will not be produced
- Should be using the "uniform_real_distribution<double>" distribution object
  - Still needs to specify the the min + max values

##### Using the distribution's default result type

- Single template type parameter, represent the type of # the distribution generate
  - Either floating point / integer
    - "double" + "int" by default template argument
  - To use the default template argument (double / int) leave the "<>" empty

##### Generating numbers that are not uniformly distributed

- The type of distribution can be changed by the type of distribution
  - i.e. "normal_distribution<>"

##### The "bernoulli_distribution" class

- Instead of returning a value, return a "bool" value
  - "true" if the value if above 0.5, else "false"
    - Needs to make sure that both engine + distribution are out of the loop, to avoid repeated value

### The IO library revisited

#### Formatted input + output

- IO stream object maintain a format state to determine how its formatted
  - Precision of float-pointing number, notational base for integer + width of the output element

##### Many manipulators change the format state

- The manipulator will also apply the changes to all subsequent IOs
  - Best to undo the changes as soon as possible

##### Controlling the format of Boolean values ("boolalpha")

- Changes how Boolean values are displayed

  - 1 + 0 by default

  - Apply "boolalpha" will output true + false instead

    - ```c++
      cout << "alpha bool:" << boolalpha << true << false <<noboolalpha<< endl;
      ```

  - To reset the "boolalpha" changes, apply "noboolalpha" to the object

##### Specifying the base for integral values ("hex", "oct", "dec")

- Change the notational system for integer with keywords

  - ```c++
    cout << "octal:" << oct << 20 << "hex:" << hex << 16 << "decimal:" << dec << 10 << endl;
    ```

- The keyword will only affect integral operand, does not affect floating point number

##### Indicating base on the output ("showbase")

- Use the "showbase" keyword to indicate which notational base system the number is in

  - ```c++
    cout << showbase;
    cout << noshowbase; // Remove the base indication infront of integral values
    ```

##### Controlling the format of floating-point values

- By default floating point have 6 digit of precision
- Very large + small value are printed with scientific notation

##### Specifying how much precision to print ("precision")

- Precision can be set with "precision()" / "setprecision()" functions
  - "precision" is overloaded, if not argument, return the current precision, if overloaded with a integral argument, change the precision + return the previous precision

##### Specifying the notation of floating-point numbers ("scientific", "fixed")

- Usually best to let the library to choose notation
- "scientific" vs. "fixed"
- "hexfloat" vs. "defaultfloat"

##### Printing the decimal point ("noshowpoint")

- By default, is floating value is 0, the decimal is not showing
  - Can manipulate that with "showpoint" vs. "noshowpoint"

##### Padding the output

- "setw": the the minimum space between the current + next value
  - Only applied to the next output
- "left / right": left / right-justify the output
- "Internal": Placement of sign on negative value
  - Left-justify the sign + right-justify the value
- "setfill": Specify an another character to pad the output, space by default

##### Controlling input formatting ("noskipws")

- By default, input operator ignores white space
  - If using the "noskipws", white space will be maintained

#### Unformatted input/output operations

- A set of low-level operations that supports unformatted IO

##### Single-byte operations

- Possible to use the "get" + "set" operation to read + write character 1 at a time

  - ```c++
    char ch;
    while(cin.get(ch))
    	cout.put(ch); // Will preserv the white space in input
    ```

##### Putting back onto the input stream

- When a character is read but cannot be used immediately, we can put them back into the stream
- "peek": Return a copy of the next character, does not change the stream
- "unget": Backs up the input stream, last pointed to character will be back at the stream
- "putback": More specialized version of "unget": Same as "unget", but must take the same argument of the character last read

##### "int" return values from input operations

- Input operations return integer to allow return of end-of-file marker
  - Normally char does not have a numeric value to store end-of-file character
  - When reading char, set them unsigned first and covert to signed, and use negative value to represent end-of-file character

##### Multi-byte operations

- Process a chunk of data at a time
  - Error-prone
- "get" + "getline" takes 3 arguments, the pointer to the start reading location, the size of the chunk being read, an character which will stop the reading if read

##### Determining how many character were read ("gcount")

- Determine how many character the last unformatted input operation read

#### Random access to a stream

- The Input have stream have decent support on random access its member
  - system-dependent
- Usually, "cin, cout, cerr, clog" do not support random access
  - Random access will fail + leave the stream in invalid state

##### Seek + tell functions

- IO object maintain a marker of where to access next
- Provide 2 functions
  - Seeking a given position + return the current position





