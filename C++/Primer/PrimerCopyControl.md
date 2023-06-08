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

- 

