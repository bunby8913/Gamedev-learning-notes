# Primer tools for large programs

### Exception handling

- Handle error with in sub-systems at run-time
  - Detection + handling the problem are separate process

#### Throwing (raised) an exception

- Type of expression + current call chain determine which handler to use to deal with the exception
- When a "throw" execute, the program will jump to the matching "catch" block (like a "return" statement)
  - Function might exit prematurely
  - Object will be destroyed

##### Stack unwinding

- The process to continue up a chain of function calling to find a matching "catch" block

- If "throw" within a "try" block, examine the corresponding "catch" block
  - If not found, keep searching for nested "try" + "catch" blocks
  - If still not found, exit the current function + search through the calling function
- If a "catch" is found, execute the code in the catch block + continue right after the "throw"
  - If not found, terminate / exit the program

##### Objects are automatically destroyed during stack unwinding

- Local object are destroyed when exiting them
- Compiler will call the destructor for class type object
  - Compiler guarantee that partially constructed class object will be handled
  - Any array + container type will be handled properly

##### Destructors + exceptions

- Use classes to control resource allocation can ensure resource are properly freed
- It's unlikely for destructor to throw error, but destructor should not throw exception without handling it
  - Could cause program termination + memory leak

##### The exception object

- The "throw" expression will copy-initialized a exception object
  - The expression must pass complete type with accessible copy, move + destructor
  - Pointer + array must be in pointer type
- Exception object is managed by compiler, accessible by the "catch" block + destroyed after exception is handled
- Error to throw pointer to a local object
  - The local object will be destroyed before reaching the "catch" block
- The static, compile-time type determine the type of the exception object
  - Object might be sliced down to only its base-class part

#### Catching an exception (exception declaration)

- A function parameter list with 1 parameter
  - Not required if the "catch" block does not access the "throw" expression
- Type of declaration determine what exception handler can catch
  - Cannot be R-value reference
  - The exception object used as declaration can be non-reference + reference type
- The "static" type of exception determine what action the "catch" perform
  - If the parameter takes a base class type, object will be sliced down to its base class form

##### Finding a matching handler

- The first matching "catch" block is selected
  - Handler for derived type needs to appear before the base type
- Most conversion between matching argument are not allowed, except
  - "const" conversion
  - Derived type -> base type
  - Array / function covert to its corresponding pointer type

##### Re-throw

- A "catch" block can passes exception to another "catch" by re-throwing exception

  - ```c++
    throw; //Can only appear in a "catch" block
    ```

  - If re-throw is not contained in a "catch" block, the program will terminate

  - Does not specify additional expression, the current exception object is passed along

  - The compiler will use the status of the exception object to find the first matching "catch" block

##### The "catch-all" handler

- To any exception using a single "catch" block, Use ellipsis as parameter

  - ```c++
    try
    {
        //Some codes
    }
    catch (...)
    {
        throw;
    }
    ```

  - Usually combined with re-throw expression

#### Function "try" block + constructors

- To handle exception during constructor initializer, use a function try block

  - Associate a group of catch with initialization phase + function body

    - ```c++
      func(T1 var1) try: data(var1)
      {
          //Block of codes
      } catch(const T2 var2) {}
      ```

  - The "try" keyword should appear before the constructor initializer list

    - The "try + catch" block is only responsible once the constructor has started
      - Parameter initialization exception should be handled with caller's exception handler

#### The "noexcept" exception specification

- Function can specify that it does not throw exceptions using the "noexcept" keyword

  - ```c++
    void func(T) noexcept;
    ```

  - Allow user to not worry about handling exceptions

  - Allow compiler to optimize the code further for code with no exceptions

- The "noexcept" keyword must appear in both declaration + definition

  - In front of a trailing return

- Cannot appear in a typedef / type alias

- Should appear after "const" / references qualifiers

- "final", "override" / "=0" (default) should be placed after the "noexcept" keyword

##### Violating the exception specification

- The compiler will compile "noexcept" function event if it violate the specification
  - Some complier will throw a warning
- If a "noexcept" function throws an exception, it will terminate the program
- Should be used in 2 situations
  - Confidence that the function won't generate any exception
  - Don't know what to do with the error anyway
- "noexcept" indicate the caller that they don't have to deal with exceptions

##### Arguments to "noexcept" specification

- The "noexcept" keyword takes an optional "bool" value
  - If true, function won't throw exception, vice versa

##### The "noexcept" operator

- "noexcept" is also an operator, return a bool r-value constant expression
  - Used to determine if the given expression is able to throw exception or not
- True if all function called by the expression are "noexcept" + the expression itself does not "throw"
- Possible to use 1 function's "noexcept" state to set another function's "noexcept" state

##### Exception specifications + pointers, virtual + copy control

- The function + pointers to function must have the similar exception specifications
  - A pointer that might "throw" can pints to any functions
- If virtual function promise not to "throw", then all override function must also not throw
- The synthesized copy control will follow the throw specifications of the rest of the operations in class
  - If all corresponding members promise to not "throw", then synthesized member will not throw

#### Exception class hierarchies

- The base exception class defines copy constructor, copy-assignment operator, a virtual destructor + virtual function called "what"
  - The virtual function returns an array of "const char*", guarantee will not throw

##### Exception classes example

- Application often derived a application-specific classes to represent application-specific exceptions
- Exception can be split into 2 categories, run-time + logic errors

### Namespaces

- Due to namespace pollution, it's very likely for some names clash with each other in large applications
  - Traditionally, avoid namespace pollution by using long + specific names
  - Contains prefix to indicate sources / library
- Namespace partition global namespace

#### Namespace definitions

- Begin with the "namespace" keyword, + namespace name + a sequence of declarations + definition in the curly braces
  - May contain classes, variables, functions, templates + other namespaces
  - Name of the namespace must be unique within scope, cannot be inside class + function
- Does not have semicolon ";" at the end

##### Each namespace is a scope

- Each name within the namespace must be unique
  - Different namespace can contain the same name
- Member in 1 namespace can be accessed by another namespace
  - Code must specific which namespace the name belongs to

##### Namespace can be discontinuous

- A namespace can be defined in several part
  - If the namespace already exist, the namespace definition will be added to the existing namespace
    - If not, a new namespace will be created
- Namespace definition can be in separate interface + files
  - Is possible to declare namespace in 1 file + define it in another
  - Separate definition + declaration can allow members to be defined once but available wherever

##### Defining namespace members

- The "#included" header is usually not part of the namespaces
  - Otherwise, the namespace would defines all the name in the header, include standard library
- Functions can use short form of name (omit the namespace name)
- To define namespace member outside the namespace scope, include the namespace scope on the definition
  - Inside the definition is the same as inside the namespace, can omit the namespaces once again
- Cannot define another namespaces inside an unrelated namespaces (even with namespace scope operator)

##### Template specializations

- Template specialization must be in the same namespaces as the original template
  - Can be defined outside the scope using namespace scope operator

##### The global namespace

- Names defined outside any class, function / namespace

  - Implicitly declared, exist in every program

- Global namespace names can exist in any files

- To refer to global namespaces members, omit the name namespaces name

  - ```c++
    ::member_Name
    ```

##### Nested namespace

- A namespace defined inside another namespace

  - Nested scope
  - Name in the inner namespace hide the name from outside
    - Names are local to inner namespace only

- The inner namespaces requires nested scope operator to access

  - ```c++
    Outer_NS::Inner_NS:NS_Member
    ```

##### Inline namespaces

- Direct members of the enclosing namespace

  - Usually used in nested namespace

  - ```c++
    inline namespace some_NS {
        // Some codes
    }
    ```

  - The keyword does not have to appear no later declarations

  - The inline namespaces will be used by default, any other non-inline namespaces have to explicitly called using scope operator

##### Unnamed namespaces

- Explicitly defined name that does not have an explicit name

  - ```c++
    namespace {}
    ```

  - Static lifetime, destroyed when the program ends

- Unnamed namespace does not span across files

  - Each files can have their own unnamed namespace

- Used directly, not possible to apply scope operator on it

- Unnamed namespace can be nested

#### Using namespace Members

- Several ways to make namespace member + names easy to use

##### Namespace aliases

- Associate a shorted name synonym to represent a long namespace name

  - ```c++
    namespace long_Namespace_Name {};
    namespace short_NS = long_Namepspace_Name;
    ```

  - Can also be applied to nested namespace

##### "using" declarations: a recap

- Introduce 1 namespace member at a time
  - Specify the namespace used for the program
  - Obey normal scope rules
    - Members with the same name but from different namespace are ignored
- Can be used on global, local, namespace / class scope (only baes class)

##### "using" directives

- Introduce the entire namespace

  - Allow unqualified form of namespace name, all members are visible for access

  - ```c++
    using namespace std;
    ```

  - Cannot appear in class scope

##### "using" directives + scope

- Using declaration act like a local alias for the namespace member
- "using" directives make the entire namespace available, some that cannot appear in a local scope
  - Appeared in the nearest enclosing namespace scope
- Still possible for "using" directives names to conflict with other names in scope
  - ambiguous error, needs to specify the namespace scope

##### Headers and "using" declaration / directives

- The Directives + declaration in header will be applied to every files that include the header file
  - Header file should not contain any "using" declaration / directives unless in function / namespaces

#### Classes, namespace + scope

- Only names declared before usage + within scope will be considered during name lookup
  - Member -> class -> enclosing scope ( 1 or more namespace)

##### Argument-dependent lookup + parameters of class type

- Compiler will search namespace independently from scope lookup when using object of class type as argument in a function
  - The object determine which namespace the function is from

##### Lookup and "std::move" + "std::forward"

- Normally, If a name is defined for application + part of an library, either overloading determine which function to use / the library version is intended to be used
- However, function template r-value reference will match with any type (L/R-reference)
  - "move" + "forward" function will always be a match
- Important to always use fully qualified version to avoid confusion + unexpected behavior

##### Friend declarations + argument-dependent lookup

- Usually a friend declaration does not make the friend visible
  - However, they are assumed to be under the same enclosing namespace (in the same namespace scope)
  - friend function can be accessed using  argument-dependent lookup

#### Overloading + namespaces

##### Argument-dependent lookup + overloading

- Each namespace that contains the class as an argument is searched during function matching
  - Functions that are outside the scope of the class but within the namespace scope will still be considered

##### Overloading + "using" declarations

- "using" declares a name, not a specific function
  - Brings in all the function with the same name into scope
- "using" declaration effectively expand the set of candidate functions
  - Overload any function declaration with the same name
    - If the "using" declarations has the same function name + function parameters list, compiler will generate an error
  - Hides any declarations from outer scope

##### Overloading + "using" directives

- If a function has the same name as the namespace function being imported, the function is overloaded
  - Not an error for "using" directives to import function with same name + parameter list
    - As long function call specify the original (namespace / current scope) of the function

##### Overloading across multiple "using" directives

- If multiple "using" directives are present, then all names from each namespaces are considered

### Multiple + virtual inheritance

- Multiple inheritance: Derive a class from multiple, direct base class
  - Inherit all properties from all derived class
  - Simple in concept, hard in design + implementation

#### Multiple inheritance

- ```c++
  class Derived_C : public BaseA, public BaseB {};
  ```

- The same access specifier applies

- Derivation list can only include defined class that are not "final"

- Each class can only be included once

##### Multiply derived classes

- The object is combination of all its inherited parts + the new members declared in the current class

##### Derived constructors initialize all base classes

- The derived object's constructor will construct + initialize all direct base sub-objects
  - Can explicitly initialize each sub-base classes / implicitly use default constructor from each inherited class
  - Construction order depends on their order in the derivation list
    - Constructor initializer is irrelevant

##### Inherited constructors + multiple inheritance

- Derived class is able to inherit constructor
  - However, cannot inherit the same constructor from > 1 base class
    - Constructor with the same parameter list
  - The derived class must define their own version of the constructor with the same parameter list

##### Destructors + multiple inheritance

- Destructor is only responsible for cleaning for its own class only
  - Base class members are handled by base class destructor
- Destructors runs in the opposite order of constructor

##### Copy + move operations for multiply derived classes

- Each derived class must define copy + move constructor + assignment operator for the whole object
  - Baes class move + copy operations will only be used if derived class use synthesized version of the members
- The base part copy / move constructor will run first to create part of the object, finally, the derived copy / move constructor will run

#### Conversions + multiple base classes

- Pointer / reference of an object's base classes can be used to point / reference a multi-base object
  - Derived class can be covert to any base class
  - Be aware of overloaded function call causing ambiguous error

##### Lookup based on type of pointer / reference

- The static type of the object / pointer / reference determine what operation is available
  - If a derived object is pointed / referenced by a base-class pointer / reference, it can only use base class functions / properties

#### Class scope under multiple inheritance

- Lookup happen in parallel among all base classes
  - If name found in > 1 classes, then the name is ambiguous
  - To use a name shared between multiple base classes, needs to specify its origins (scope)
- Would still result in ambiguous error if both function have different parameter list + different access specifier
- Advices to avoid potential ambiguities by define a derived-class specific version of the function 

#### Virtual inheritance

- A class in reality can inherit from the same base class more than once
  - Direct inheritance / indirect through  base class
  - The derived class could have multiple the same sub-object
- This could be an issue if an sub-object in the derived class is intended to be shared internally
  - Solved by using virtual inheritance + shared-base class sub-object (virtual base class)
    - The derived will always only contain 1 sub-object

##### Constructing a virtual base class

- The virtual base class require well-design before implementation
  - The virtual class needs to be anticipated to be inherited by multiple derived class + inherited by a single class again
- Virtual derivation does not affect the class directly derived class, but the class derives from the derived class (the grandchild)

##### Using a virtual base class

- Specify the base class is virtual by using the keyword "virtual"

  - ```c++
    class ChildClass : virtual public VParentClass {};
    ```

  - Share single instance of the base class with derived class

- No further action required when inheriting from a class with virtual base class

- Normal conversion between derived + base class are supported as usual

##### Visibility of virtual base-class members

- The virtual base-class sub-object can be access with no scope operator
  - Only 1 exist in the class, will always be unambiguous
- If any class inherit from override the base member, the override version of the  base object will be available to access
  - If multiple class inherit from override the base object, then scope operator is required for access (ambiguous)

#### Constructors + virtual inheritance

- Virtual base is initialized by the most derived constructor
  - The class that define the virtual base class as "virtual" is responsible
    - Otherwise, the virtual base object might initialized more than once, defeat the purpose
- If a class create independent object of the virtual base class, the class construction needs to initialize the virtual base

##### How a virtually inherited object is constructed

- The virtual base sub-object is created first
  - Using the most-derived class's constructor on the virtual base object
- The rest sub-objects are created follows the derivation list order

##### Constructor + destructor order

- A class can have > 1 virtual base class
  - If multiple virtual base class, construct from left -> right order
- Same order for copy + move constructor, destructors runs in the opposite order
