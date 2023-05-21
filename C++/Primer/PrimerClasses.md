# Primer classes

- Data abstraction: Separation of interface + implementation
  - Expose the interface of a class for user to execute
  - Implementation is hidden away, include class data members, function bodies + any other functions not designed for general uses
- Encapsulation: Enforce data abstraction
- Class use data abstraction + encapsulation defines an abstract data type

### 7.1 Defining abstract data types

#### 7.1.2 Defining a class

- Defines + declare member function similar to ordinary function
  - Declared within the class
  - Can be defined outside the class
  - Non-member function are declared + defined outside the class

##### Introducing "this"

- Apart from 1 exception, we call member function on the object
- Member function can access object being called using extra, implicit parameter ("this")
- When calling a member function, we passe the object's address to the implicit this parameter
- Does not have to use the member access operator "this" pointer
  - Any use of member of the class is assumed to be implicit reference through this
- Illegal to define a parameter to be called "this"
- This is a constant pointer, address cannot be changed

##### Introducing const member functions

- Use the "const" keyword to modify the type of the implicit "this" pointer
- By default, "this" is a const pointer points to a non-const class type
  - Cannot bind "this" to a const object -> cannot call non-const member function on a const object
- If "this" is declared as const pointer to a const object give more flexibility to the member function
  - Const following parameter list indicate the "this" is a pointer to const object
- By stating "this" points to a constant pointer in a function also indicate the member function cannot change the object they are called by

##### Class scope + member functions

- The member function must be declared within the class scope
  - However, the definition of the function can be defined outside the scope (even after the class definition)
- Compiler will compile member declarations first, then compile the member function bodies

##### Defining a member function outside the class

- Member definition needs to match member declaration
- The const-ness of the return type + parameter list + name must match exactly
- Must include the name of the class in definition

##### Defining a function to return "this" object

- When returning the object using "this" keyword, return the object as a object pointer
  - Need to use the "this" keyword to access the object
  - The return type of the function also needs to be a references type

#### 7.1.3 Defining Non-member class-related functions

- Possible to define functions that are part of the class interface, but not part of the class itself
- Declares non-member functions like any other function
  - The functions should be declared in the same header as the class itself
    - Not necessarily defined together

#### 7.1.4 Constructor

- Special member functions that defines how objects can be initialized
  - Functions run whenever an object is created
  - Use the same name as the class
  - Have no return type
  - Can have 0 or more parameter list + function body (could be empty)
  - Each class can have multiple constructors
    - Different constructors must differ in parameter list (# of types)
  - Constructor cannot be declared as const
    - Object assume its constness after the construction
      - Allow constructor to write to const object

##### The synthesized default constructor

- Default constructor: an implicit constructor, used when the class does not explicitly define any constructors (synthesized default constructor)
  - If there are in-class initializer, use that to initialize class member
  - Else, use the default initialize value for the member

##### Some classes cannot rely on the synthesized default constructor

- If any constructors has been defined, the default constructor will not be used unless it's been defined by developer
- The synthesized default constructor sometime does the wrong thing
  - Some value when default initialized has un-defined value
    - class should only use synthesized default constructor if all members have in-class initializers

- Sometime, compiler is unable to synthesize constructor
  - If a class include a member of a class that does not have default constructor, compiler cannot initiable that member


##### What does "= default" means

- Assign the default behavior same as the synthesized version of the constructor to a variable
  - Can appear with declaration (inside class body) or with class definition (outside class body)

##### Constructor initializer list

- If compiler does not support in-class initializers, default constructor should use constructor initializer list

- Constructor initializer list specifies initial values for 1 or more data when created
  - ```c++
    T (T1 variable_Initialize_Value): variable1(variable_Initialize_Value) {} 
    ```

- If a value is not explicitly initialize in the list, it's initialized using synthesize default constructor method

- Should always use in-class initializer if the compiler supports it

##### Defining a constructor outside the class body

- Constructor have no return type
- Need to specify the class of the constructor is part of

#### 7.1.5 Copy, assignment + destruction

- If copy, assignment + destruction operation are not explicitly defined, compiler will synthesize

##### Some classes cannot rely on the synthesized versions

- If the class allocate resources outside the class object, synthesized operation might behave unexpectedly
  - i.e. dynamic memory
- However, most of those situations can (+ should) be avoided by using vector / string 

### 7.2 Access control + encapsulation

- C++ uses access specifiers to enforce encapsulation
- Public: members under the public specifier are accessible by all parts of the program
  - Defines the interfaces
- Private: members are only accessible by code in the class
  - Encapsulate the implementation
- Class can have 0 or more access specifiers
  - They can appear multiple times in the same class
  - The access specifier level remain same until the next access specifier

##### Using the class / struct keyword

- Struct + class has different
  - "struct" member before the first access specifier are public
  - "class" member before the first access specified are private
- If we want all members to be public, use struct
  - If we want private members, use private

#### 7.2.1 Friends

- Class can have other class / functions to access private members by using "friend"
- Use the keyword "friend" in front of the function / class declaration
  - "friend" declaration can only be inside the class definition
    - Good idea to group all friend declaration at beginning / ends of the code

##### Declarations for friends

- Declaration for friend only grant access, but does not declare the function
  - The "friend" function must be declared  + defined elsewhere
- Declare each friend in the same header as the class
- Some compiler might support call to friend function without ordinary declaration, but not a good practice (not support by all)

### 7.3 Additional class features

#### 7.3.1 Class members revisited

##### Defining a type member

- Class can define own local names for types

  - Hides the data type of the member from the user

- Type member can be defined using type member / type alias

  - ```c++
    typedef member_type T;
    using T = member_type; // equivalent to define a type member
    ```

  - Members to define a type must be declared before usage

    - Should be placed at the beginning of the class


##### Making members inline

- Member function defined inside the class is automatically inline
  - Inline: Ask the compiler to expand in place, rather than calling the function and run from a different memory address
- Inline should always be specified at function declaration (within the class scope) (Better practice)
  - However, can also specify inline on the function definition (outside the class body)

##### Overloading member functions

- Member functions may be overloaded, same function overloading rules apply

##### "mutable" data members

- Allow data member to be modified by const member function
  - Indicate the member with the "mutable" keyword
- "mutable" data member is never const, even if the object is const

##### Initializers for data members of class type

- To initialize member of class type, supply arguments the constructor requires
  - In-class initializers needs to use either "=" form of initialization / direct form of initialization (inside braces)  

#### 7.3.2 Functions that return "*this"

- Will return a reference to the object called the function

##### Returning "*this" from a const member function

- If the function is a const member, then the return pointer + return object will also be const
  - However, non-const operations cannot use const object

##### Overloading based on const

- Function can be overloaded base on the const-ness of the pointer parameter
- All overloaded function can call the same support function to do all the work
  - Overloaded function's return references depends on the const-ness of the function

#### 7.3.3 Class types

- Every class defines a unique type
  - Even if they have the same members, still different types
- Possible to use class name as type name
  - Also can include the keyword "class" / "struct" before the class name on declaration
    - Inherited from C, but also valid in C++

##### Class declarations

- Possible to declare a class without defining it
  - Forward declaration : Introduce the type name, indicate the type name refer to a class
    - Becomes a incomplete type
    - Can only be used in limit amount of ways
      - Defines pointer / reference of the type
      - Declare function uses the incomplete type as parameter / return type
- Class must be declared + defined before creation
  - Does not know how much the object requires / does not know the member of the object
- Apart from "static" class member, data member can only be class type if it's defined
- Since class is not defined until the end of body block, class cannot contain themself as a member
  - But class can have pointer/ reference to themself

#### 7.3.4 Friendship revisited

##### Friendship between classes

- Each classes control which classes / functions they are friends with
  - Friendship is not transitive
- To access private data from another classes, the current class must add that class as a friend

##### Making a member function a friend

- Instead making the entire class a friend, it's possible to make only a member function to be friend
- The program needs to be specified in a special ways
  - The class the friend member function resides needs to be declared first, but does not define the member function yet
  - Define the current class first, declare the friendship with the member function
  - Finally, define the member function, which can now be used in the current class

##### Overloaded functions + friendship

- Overloaded functions share the same name, but identify as different functions
- Each overloaded function needs to declare friendship independently

##### Friend declarations + scope

- Classes + non-member functions does not needs to de declared before using a friend declarations
  - The functions still requires a explicit definition before usage
- Friend declaration is not a normal declarations

### 7.4 Class scope

- Every class has its own new scope
  - Data + function within the class scope can only be accessed through object, reference / pointers
  - Access member through the scope operator ("::")

##### Scope + members defined outside the class

- Once the compiler sees the class name, everything after is considered to be within the class scope
  - parameter list + function body
    - Function body can access in class members without stating the class the member are from
- Return type is outside the class scope
  - If we want to return type must specify the class its member of

#### 7.4.1 Name lookup + class scope

- Name lookup: Compiler process for looking up declarations with the matching name
  1. Look for the declaration with in the block (named have to be declared before the current line)
  2. If not name found, look within the scope
  3. If not found, return error
- Class definition processed in 2 phases
  1. The compiler scan for all class member declaration, create symbol table for class
     - Also determine the size of the class
  2. The compiler process definition of member function + member variable
     - After the entire class declaration has been processed
- Member function can use any name defined in the class

##### Name lookup for class member declarations

- Name in declarations (return type + parameter) needs to declared before used
  - Still follows the name lookup rules

##### Type names are special

- Usually inner scope can re-define a name from outer scope
  - But for class, the name has the same definition as in the outer scope
- Definition of type name should be at the beginning of the class

##### Normal block-scope name lookup inside member definition

- Name used in member function are resolved in 3 steps
  1. Look for declaration inside the member function (Declaration needs to be before the current line)
  2. If declaration not found in the member function, lookup all the class declarations
  3. If declaration not found in class, lookup declaration in scope, before the member function definition 
- Bad idea to use name of another member as name for parameter in member function

##### After class scope, look in the surrounding scope

- If the code wants to use object from outer scope (usually hidden away), explicitly state the location with scope operator
  - If the class name is empty, the scope operator assume it's the global scope

##### Names are resolved where they appear within a file

- As long the declaration of global function is in front of member function definition, it can be used in the member function

### 7.5 constructor revisited

#### 7.5.1 Constructor initializer list

- If a variable is defined but not explicitly initialized in the constructor, the member will be default initialized before the class body execution

- ```c++
  T::T(T1 variable1, T2 variable2) : local_Variable1(variable1), local_Variable2(variable2){} // Initialize data members
  
  class_Name::class_Name(T1 variable1, T2 variable2) // assign value to data mebmer
  {
      local_Variable1 = variable1;
      local_Variable2 = variable2;
  }
  // Both constructor have the same effect, but second method is sloppier
  ```

##### Constructor initializers are sometimes required

- Some variables must be initialized with value
  - Const variable + references
- Member of class type that does not have default constructor also must be initialized
  - There's no way to initialize class without arguments
- Should always consider to use constructor initializers
  - Better for low-level efficiency
  - Able to initialize some data members

##### Order of member initialization

- Constructor initialized list only specifies the value used to initialize the member, not the order initialization is performed
- Avoid using members to initialize other member in constructor
- Always good to use constructor's permeameter rather than other data member in class

##### Default arguments + constructors

- We can use default argument to define a single constructor to be default constructor + constructor that takes arguments
  - Constructor supplies default argument to all parameters is also considered as default constructor
    - Does not work well with constructor that required multiple values, and values are related to each other

#### 7.5.2 Delegating constructors

- Delegating constructor: uses another constructor within the class to perform initialization

  - Delegate some work to other constructor

- Delegating constructor has a single entry, follow by parenthesized list of argument

  - Argument list needs to match with another constructor in class

- ```c++
  T(T1 varialbe1, T2 variable2): local_Variable1(variable1), local_Variabler(variable2);
  class_Name():class_Name(null, null){}
  class_Name(some_Variable):class_Name(some_Variable, null){}
  ```

- The body of the constructor runs after the delegated constructor has finished

  - The body code of the delegated constructor will run before the function body of delegating constructor

#### 7.5.3 The role of the default constructor

- Default initialization is used in the following situation
  - When define non-static variable / array in block scope without initialization
  - When class has member using synthesized default constructor
  - When class member not explicitly initialized in the constructor initializer list
- Value initialization occurs when 
  - Array initialization, when there are fewer initialization value than the size of the array
  - Define local static object without initializer
  - Explicitly request value initialization using T() (T is the name of the type)
    - (# of elements, element initialized value)
- Best practice to always provide default constructor if other constructor are available

##### Using the default constructor

- When using the default constructor to create an object, remove empty parentheses
  - Otherwise, we are defining a function with 0 parameter that returns the object type

#### 7.5.4 Implicit class-type conversions

- Classes can define implicit conversion just like automatics conversions between built-in types
- Converting constructors: Constructor with a single parameter arguments implicit convert the argument to a class type
  - If the variable have the same type as the single parameter arguments, we can use that variable where the object of constructor class type is expected

##### Only 1 class-type conversion is allowed

- Compiler can only apply 1 layer of class-type conversion
  - To make multiple layers of conversion, programmer needs to explicitly converting the variable

##### Class-type conversions are not always useful

- Not useful when convert from temporary objects that cannot be accessed after the operation
  - The object is constructed + discarded right after (waste resources)

##### Suppressing implicit conversions defined by constructors

- Prevent use of constructor to perform implicit conversions by declaring the constructor as explicit

  - ```c++
    explicit T(T1 variable1, T2 variable2):local_Variable1(variable1), local_Variable(variable2){}
    ```

  - Keyword is only meaningful in constructor

  - Only needs to be applied to constructor that takes a single argument

  - Only allowed on a constructor declaration in a class header

- The compiler will prevent the user to compile using suppressed implicit conversion

##### "explicit" constructor can be used only for direct initialization

- Cannot use explicit constructor with copy form of initialization ("=")
  - Only works with direct initialization (T object1(other_object))

##### Explicitly using constructors for conversions

- Can force the conversion to happen with 2 ways
  - Construct a temporary object using the variable
  - Use "static_cast" to perform explicit conversion ("explicit" keyword controls implicit conversions)

#### 7.5.5 Aggregate classes

- Given user direct access to the member + has special initialization syntax
  - All data members are public
  - Does not define constructors
  - No in-class initializers
  - No base class / virtual functions
- Initialize class + data member by providing braced list of member initializers
  - Initializer needs to be same order as the data member declaration
  - If initializers has less elements than the class member, the rest class member will be value initialized
    - Initializers cannot contain more element than the class members
- 3 drawbacks of using aggregate classes
  - Requires all data member to be public
  - Puts the burden of initialization on user rather than developer
    - Error-prone
  - If any member added / removed, all the initializers in the code needs to be changed

#### 7.5.6 Literal classes

- Possible for class to be literal types
  - Literal types of classes may have function that are "constexpr" (values are evaluated during compile time)
    - Implicit const
- Aggregated class with all literal type member are literal classes, or
  - Data member are all literal type
  - Class must have at least 1 "constexpr" constructors
  - If the class has in-class initializer, initializer for built-in type must be "constexpr", or if initializer for class type it must use member's "constexpr" constructor
  - Class can only use default definition destructor

##### "constexpr" constructors

- Construction in literal class can be "constexpr" (needs to have at least 1)

- "constexpr" constructor must follow the same rule as regular constructor

  - No return statement
  - But constexpr can only consist of return statement
  - Therefore, most of the time, constructor body is empty

- ```c++
  constexpr T = default{}
  constexpr T(T1 variable): local_Variable1(variable), local_Variabler2(variable){}
  
  ```

- "constexpr" must initialize every data member

- Used to generate "constexpr" objects, parameters for "constexpr" return function

### 7.6 "static" class members

- "static" class member are members of classes associated with the class, rather than individual objects

##### Declaring "static" members

- "static" member can be public /private, "const", reference, array / class types
- Static member is outside the objects
  - Object does not store static member locally
- Static function is not bound to objects
  - Do not have the "this" pointer
  - Cannot be declared as "const"

##### Using a class "static" member

- Access static member through the scope operator (::)
  - Indicate the class the "static" member is from
- "static" member can be accessed through object, reference / pointer of the class
- Member function can access "static" member without the scope operator

##### Defining "static" members

- "static" member can be defined inside/outside the class body
  - Must be declared within the class body
  - Outside the class body, does not need to use the "static" keyword anymore  (Only needs to define the scope of the "static" member)
- Not initialized by the class constructor + cannot initialized a "static" member inside the class
  - "static" member must be defined outside the class body + only once
- Once defined, continue to exist until program ends
- Static member definition can access private member function

##### In-class initialization of "static" data members

- For literal / "constexpr" "static" member, use in-class initializers for the
  - Initializers must also be "constexpr"
  - The "static" member itself is also a "constexpr"
- Even so, to use the "const static" member outside the class, the member should still be initialized outside the class definition
- If value of "static" member is defined in class, the value should be re-defined outside the class

#####  "static" members can be used in ways ordinary members can't

- "static" member can have incomplete type, can be the same type as the class type
- "static" member can be used as a default argument
  - Provide default value shared among all objects of the class

##### 

  