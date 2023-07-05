# Primer Specialized Tools + Techniques

### Run-time type identification

- Achieved using 2 operations
  - "typeid": provide the type of the given expression
  - "dynamic_cast": covert a pointer / reference of base type -> pointer/ reference to derived type
- Those operations can use the dynamic type of the object (usually the static type of the object determines the operation available)
  - Useful when we want to apply a derived object's operation on base-class object through pointer/ reference + cannot use virtual function
    - Virtual function is able to select the correct version of the function base on dynamic type
- Warning: More error-prone than using virtual function

#### The "dynamic_cast" operator

- Available syntax

  - ```c++
    dynamic_cast<type*>(e); // e must be an pointer
    dynamic_cast<type&>(e); // e must be a L-value
    dynamic_cast<type&&>(e); // e cannot b a L-value
    ```

- "type" must be a class type

  - "e" must either be a child type of "type",  a base class of "type" / same as "type"
  - Result in "bad_cast" / 0

##### Pointer-type "dynamic_cast"

- If the base class has at least 1 virtual function + is the parent of a derived class

  - Possible to cast a pointer / reference of the base class to use derived class's operation using "dynamic_cast"

  - ```c++
    if (Derived *dp = dynamic_cast<derived*>(bp)){
        // Code that operate on bp as it is an dp
    } else {
        // conversion failed
    }
    ```

- "dynamic_cast" is able to covert a null pointer to the requested type

- "dynamic_cast" allow test + cast to be done with single statement

##### Reference-type dynamic_cast

- Since there are no null reference, cast a reference-type should use try + catch exception to perform the cast

  - ```c++
    try {
        const Derived &d = dynamic_cast<const Derived&>(br);
    } catch (bad_cast)
    {
        //handle the bad cast call
    }
    ```

#### The "typeid" operator

- Ask an expression what is the type of its object

  - ```c++
    typeid(e); // e is an expression / type name
    ```

  - Return an const reference to an object "type_info" (or its derived object)

- Using the "typeinfo" header

- If used on a reference, return the type that the reference is referring to

- If used on an array, return the type of the array

- If used on a class type with virtual function / L-value reference to a class, "typeid" determine the type at run-time, else, return the static type

##### Using the "typeid" operator

- Used to compare types of 2 expressions / compare an expression to a type

- ```c++
  if (typeid(*bp) == typeid(*dp))
  {}
  // This is evaluated at compile time, static type
  if (typeid(*bp) == typeid(Derived))
  {}
  // This is evaluated at run-time if contain at least 1 virtual function, dynamic type
  if (typeid(bp) == typeid(Derived))
  ```

#### Using RTTI

- Particularly useful to solve problem when operations are required between base + derived class
  - Depends on the parameter of the virtual function, some data members is not available
- We can use "typeid" to first check if the 2 objects are the same base type, if so, call virtual function on the derived class + cast the other operand to the same type + perform operation
  - Any derived class operation will have to cast the base object to the derived object to perform virtual operations
  - The base class Dose not, Since the Reference the function use is already base type

#### The "type_info" class

- Exact definition varies from compiler
  - Class guarantee to provide equality operator, name access + if a types comeb "before" another type
- No default constructor, deleted copy + move + assignment constructor
- "type_info" name are in c-style character string, guarantee to represent different type with different string, but does not guarantee the actual implementation

### Enumerations

- Group together a set of integral constants

  - Literal types

- 2 types of enumerations available

  - Scoped: defined using the keyword "enum class" / "enum struct"

    - Follow by enumeration names + curly braces wrapped comma separate list of enumerators

    - ```c++
      enum class enum_Example {input, output, append};
      ```

  - Un-scoped: Similar to scope enumerations, however, omit the "class / struct" keyword

    - Name is optional

    - ```c++
      enum color {red, yellow, blue};
      enum {floatRed = 6, doubleYellow = 8; intBlue = 15};
      ```

##### Enumerators

- Enumeration name can only be accessed within scope
  - Unnamed enumeration's enumerator names are in the same scope
- Enumerator start with the value 0, each enumerator proceed should have value at least 1 greater
  - As showed earlier, can supply initializer value
- Enumerator must be const + initialized with constant expression
- Enumerator itself is also a constant expression
  - Used when constant expression is needed
  - i.e. "switch" case label

##### Like classes, enumerations define new types

- Named enums are a custom type, enum object can only be initialized + assigned by enumerator / another enum of the same type
- Un-scope enums can be coverted to integral type automatically, can be used when integral value is needed (variable assignment)

##### Specifying the size of an "enum"

- "enums" value represented by a built-in type

  - "enum" with name can specify the type of value stored

    - ```c++
      enum intValues : unsigned long long{
          
      };
      ```

  -  If the type of values is specified, error to store values that is too large for the type

- Specify type allow compiler to stay consistent

##### Forward declarations for enumerations

- "enum" can be forward declared
  - Needs to specify the size of the "enum" (implicit / explicit)
  - Un-scoped "enum" must declare size explicitly
- Declare + definition for "enum" must match
  - scoped vs. un-scoped
  - The size of the enumeration must stay consistent

##### Parameter matching + enumerations

- Integer value with the same value as an enumerator cannot be used to call a function expecting an "enum argument"
- The opposite is okay, possible to pass object of enumerator / un-scoped enumeration to a integral type parameter 
  - The "enum" values can convert to "int" / other integral value
  - Cannot be convert to a "const char", even if the number is within range

### Pointer to class member

- A pointer can points to a non-static member of a class
  - Normally pointer points to object, members of an object / static member of a class
- Initialize pointer to point to a specific member of a class without using  object

#### Pointers to data members

- A pointer to member also include the class that contains the member

  - ```c++
    const T Class_Name::*ptr; // pointer declaration
    ```

  - Pointer to a member of class "Class_Name" of type "const T"

- To initialize the pointer, state the member the pointers to

  - ```c++
    ptr = &Class_Name::var1;
    ```

- Can use the "auto" / "decltype" to automatically determine the type + combine declaration + initialization together

##### Using a pointer to data member

- The pointer initially do not point to any data

  - Bind to a specific member but data are stored on specific object

- Pointer to member can dereference data from supplied object

- 2 ways to dereference

  - ```c++
    My_Class myClass, *pClass = &myClass;
    auto s = myClass.*pdata;
    s = pClass->*pdata;
    ```

  - First, dereference the pointe to get the member info

  - Then, use the member info to fetch the member's data from the supplied object

##### A function returning a pointer to data member

- Normal access controls apply to pointers to member
- Usually, access of private member is achieved through a "static" function that returns a pointer to the member

#### Pointers to member functions

- Pointers can also points to a member function of a class
  - Best to use "auto" to deduce the type of the function
- The type of the pointer is determined by function's return type + parameter list
- If function is overloaded, must provide enough information to declare the type explicitly
- No conversion between member function + pointer to the member

##### Using a pointer to member function

- Same as member pointer, Use ".*" / "->*" to call member function through pointer to member
  - Parentheses are required to represent precedence
    - The class scope + member pointer should be grouped together

##### Using type aliases for member pointers

- Make pointer to member easier to read

  - ```c++
    using aliasesName = T (Class_Name::*)(Class_Name::var1, Class_name::var2) const;
    ```

  - "aliasesName" represent a const member function in "Class_Name" that has 2 argument of var1 and var2 in the "Class_Name" class, returns a T type

- Pointer-to-member function can be used as return type / parameter type for a function

- Can have default arguments

##### Pointer-to-member function tables

- Pointers to member function are commonly used in function table

#### Using member functions as callable objects

- Pointer to member is not a callable object
  - Cannot be passed to an generic algorithm

##### Using "function" to generate a callable

- Create a callable function bind to member function by using library "function" template



