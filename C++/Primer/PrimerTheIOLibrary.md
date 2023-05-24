# Primer The IO library

- C++ use a family of type defined by standard library to deal with IOs

### 8.1 The IO classes

- IO processing are separated into 3 different headers
  - iostream: Basic type to read + write from stream
  - fstream: Types to read + write named files
  - sstream: read + write in-memory "strings"
- Library define a special types + objects "wchar_t" all have the "w" in front of the name

##### Relationships among the IO types

- IO operations does not change base on character size / kind of devices
- IO library build upon inheritance
  - Let particular class inherits from another class
  - iostream is the parent class, fstream + sstream inherited from it

#### 8.1.1 No copy / assign for IO objects

- Cannot copy / assign object of IO type
- IO are usually passed / returned by reference
- Reading / writing to IO object changes its state, IO objects cannot be "const"

#### 8.1.2 Condition states

- IO classes define function + flags, the condition state of stream can change
- Once IO object encounters an error, IO operation on stream will fail
  - Can only read/write from stream when it's in non-error state
- To check if a stream is in error state, use a "while" conditions
  - If the state remain valid, condition will succeed

##### Interrogating the state of a stream

- Sometimes is important to know why the text stream is invalid
- IO library has a machine-dependent integer “iostate” indicate the the state of stream
- IO library defines 4 different constexpr values to represent bit patterns
- Use bitwise operation to test for multiple flags
	- bad bit: Indicate the stream is corrupted
		- Can’t continue to use stream
	- fail bit: indicate IO operation failed
		- Set after recoverable error, possible to correct + continue to use the stream
	- elf bit: Indicate steam hit end of fuel
	- good bit: Indicate no error state
- Stream has available function “s.fail()” to determine if fail / bad bit is set + “s.good()” to determine if the stream is good

##### Managing the condition state

- “clear()” function clear all the error states
	- Function is overloaded, could also take a single argument of “iostate” type
		- Used to only reset certain bits
- “setstate”: set the conditional bits to indicate problem (reset the clear() function)

#### 8.1.3 Managing the output buffer

- Output stream manages a buffer of data program reads + write
	- Buffer allow the OS to combine multiple output to output together, single system-level write
		- Important performance boost
- Several conditions cause the buffer to be flushed
	- return main flush the buffer
	- When the buffer is full, buffer will flush before writing new value
	- explicit manipulator causing the buffer to flush (endl)
	- use “unitbut” to manually set internal state to flush buffer
		- Default to “cerr”
	- If output stream is tied to another stream, flush the buffer when the other stream is used

##### Flushing the output buffer

- “flush” keyword flush the buffer + does nothing else
- “ends” keyword add a null character at the end + flush the buffer

##### The “unitbuf” manipulator

- use the keyword “unitbuf” to indicate all write will be immediately flushed
- use the keyword “nonunitbuf” to indicate return to normal buffering
- Note: when the program crush, the buffer is not flushed

##### Tying input + output streams together

- Attempt to read input will flush buffer associated with output
	- Good practice to always tie input stream to output stream, allowing output to be written in buffer before reading input
- Use the “tie()” function to connect input + output streams
	- Takes no argument, return a pointer to the output stream
		- Return null pointer is stream is not tied
	- Overloaded function, takes a pointer of an out stream
		- ties an input (output) streams to the output streams
- Pass null pointer to the stream (input/output) to untie the stream
	- Each stream can only be tied to one other streams
	- Multiple streams can tie do the same outstream

### 8.2 File input + output (fstream)

- ifstream: File reader
- ofstream: File writer
- Use the IO operator to read/write files
- Use the “getline()” to read a ifstream, and proceed with iostream

#### 8.2.1 Using file stream objects

- Define a file stream object + associate object with a file to read
- Defines member function “open” to locate + open the file
	- Provide ifstream object with file name automatically opens the file
	- File name can be c-style character / string

##### Using an fstream in place of an iostream&

- Functions that takes reference/pointer of iostream type can also take ofstream or ifstream

##### The “open” + “close” members

- After define an empty file stream object, we can link the object to a file by using the “open()” function
	- Can check if the “open()” operation successes
		- If not, a fail bit will be raised
- File stream will be associated with a file until the file is closed “close()”
	- Cannot open the same file again + cannot open another file

##### Automatic construction + destruction

- Within a for loop / other types of loops, file stream object are created + destroyed after every iteration
	- When file stream object destroyed, the file associated is automatically closed

#### 8.2.2 File Modes

- File modes represent how the file may be used
	- in: Open for input
		- Only set by ifstream / fstream object
	- out: Open for output
		- Only set by ofstream / fstream object
	- app: Open file + jump to the end to write
	- ate: Open file + jump to the end to read
	- trunc: Delete the file after opened
		- Only set when out if specified
	- binary: Do IO operation in binary mode
- By default, file open in out mode, if also in trunc mode, unless app / in is also specified
- ate / binary can be applied to any file stream objects

##### Opening a file in “out” mode discards existing data

##### File mode is determined each time “open” is called

- If no output mode specified, the file will be opened in out + trunc by default
- Need to specifically ask for “ofstream:app” / “ofstream:in” 

### 8.3 "string" streams

- istringstream: read string
- ostringstream: write string
- stringstream: read + write string
- Inherited from the same class with fstream but not directly related to each other

#### 8.3.1 Using an istringstream

- Working on a line of string + working with individual words within lines
	- Use istringstream to read invidious elements in the line

#### 8.3.2 Using istringstream s

- When building output a word at a time, but does not need to print 

