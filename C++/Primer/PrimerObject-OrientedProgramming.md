

# Primer Object-oriented programming

- Encapsulation (data abstraction), inheritance, polymorphism (Dynamic binding)

### OOP: an overview

- Data abstraction: separate interface + implementation

##### Inheritance

- Inheritance: Define relationship with other class of similar type
- Base class (root of the hierarchy) + derived class
  - Base class define common member to all classes, derived class define specific members
- Base class contain 2 types of functions, type dependent + function expected to not be changed when inherit
  - use the "virtual" keyword to define function that the derived classes is expected to change
- Derived class use a "class derivation list" (":", each class is separated with ",") to indicate any inheritance
- Derived class needs to explicitly note to "override" a member function

##### Dynamic binding (Polymorphism)

- Dynamic binding: Use object of several types + ignore how they differ
- Use the same code to process different classes inherited from the same base class
- The version of the virtual function to run depends on the type of the object when called through reference + pointer to a base class
- Decision which function to use is determined at run-time

### Defining base + derived classes

#### Defining a base class

- By default, the destructor should be define as virtual

##### Member functions + inheritance

- Derived class needs to override inherited definition + provide new, own definition
- When calling "virtual" function through pointer + reference, type of the object will be used to determine which function to execute
- Any non-static (other than constructor) function can be "virtual"
  - Only add in the function declaration, not definition

##### Access control + inheritance

- Derived class's function cannot access private members of the base class
  - Can only be access within a class
- For derived class to access members in base class + prohibit other class from access the member, use the "private" keyword

#### Defining a derived class

- Derived class can provide access specifier to each class it inherit from
  - If base class is "public", public members of the base class is also available to derived class
  - Can bind a public derived type to pointer / reference of the base type
- Single inheritance vs. multi inheritance

##### Virtual functions in the derived class

- Derived classes not mandatory to override virtual function from base class
  - If not override, use the inherited version of the function
- Derived classes not mandatory to include the "virtual" on function being override
  - Must explicit add the keyword "override" after argument list (after const for reference / const functions)

##### Derived-class objects + the derived-to-base conversion

- Derived class include data member from the base + derived class self
  - Key to how inheritance works
- Derived class's object can be used as base type's object
  - Bind base class reference / pointer to derived class's object
  - Derived-to-base conversion, applied implicitly

##### Derived-class constructors

- Each classes control how data are initialized
  - Use the base-class constructor to initialize base-class members
- Use constructor initializer to pass arguments to base-class constructor
  - Base class should be initialized first

##### Using members of the base class from the derived class

- Derived class can access public + protected members of base class
  - No distinction between access specifier of derived class member + base class member
- Should always use the interface provided by the base class to access derived class member
  - Should use constructor instead

##### Inheritance + "static" members

- "static" member is not inherited, there will only be 1 instance
- "static" member follow same access rule
  - If "private", derived class cannot access

##### Declaration of derived classes

- Declared just like any other class
  - Do not need to include derivation list

##### Classes used as a base class

- Class must be defined before able to become a base class
  - Derived classes require the definition of base class for their construction
- Class cannot derive from itself

##### Preventing inheritance ("final")

- Use the "final" keyword to prevent a class from being inherited from
  - keyword applied after the class name

#### Conversions + inheritance

- Normally, reference / pointer can only be bind to the object of same type / const difference
  - Inherited object can be bind to the base-class type reference / pointer / smart pointers
  - Reference / pointer of base object can be base object / derived object

##### Static type + dynamic type

- Static type: Known at compile time, the type pointers / references declared as
- Dynamic type: Known at run time, the actual type of pointers/ references, change through program execution, may not be the same as static type
- Variable of an class object will always have the same static + dynamic type

##### There is no implicit conversion from base to derived

- No guarantee based object can be converted to derived object
  - Derived object may contain functions + data member the base object does not have
- Sometime, cannot convert based -> derived even base object pointers points to derived object
  - Compiler cannot know the type of object at the address
  - However, possible to use "static_cast" + "dynamic_cast" to cast base class pointer to derived class pointer

##### No conversion between objects

- Base class type does not behave properly when directly convert to derived class type
  - Constructor + assignment operator are both from the base class, The new data member will not be initialized + assigned (sliced down)

### Virtual functions

- Virtual function must always be defined (base / derived implementation)
	- Compiler cannot determine if the virtual function is used or not

##### Calls to virtual functions may be resolved at run time

- Compiler generate code to determine which virtual function to use
	- Determine by the dynamic type of the pointer / reference
- Dynamic binding only happens when object is in pointer / reference form
	- Otherwise, call bind at compiler time, use static type and type cannot be changed
	- Delay the decision of which function to use until run-time

##### Virtual functions in a derived class

- derived can (not required) to repeat the “virtual” keyword
	- Function will remain “virtual” for all derived classes
- Overridden “virtual” function in derived class must have exact arguments as the base class
	- Return type must be the same / or a reference related by inheritance
		- Requires derived-to-based conversion to be accessible

##### The “final” + “override” specifiers

- Possible to define a function with same name as “virtual” function in derived class with different parameter list
	- Does not override “virtual” function, treated as a new function
	- Hard to spot mistake
- Better to use the “override” keyword to specify virtual function override in derived class
	- Enlist compiler to spot the argument difference
	- Override cannot be applied to non-virtual + virtual function with the wrong argument list
- “final” will prevent a function from override

##### Virtual functions + default arguments

- Virtual function can use default argument
	- Use the static type default value (base-class value)
- Derived class cannot change virtual function’s default value

##### Circumventing the virtual mechanism

- Possible to call a particular version of the virtual function
- Use the scope operator to specify the function version
	- Resolved in compile time
	- Should only be used inside member function
	- Usually used for derived class to call base class virtual function
		- Base-class function are usually general purpose
- If the scope operator is missing, function will call for derived virtual function, resolved at run-time, cause a infinite recursion

### Abstract base classes

##### Pure virtual functions

- Prevent user from Creating the object of the class, class should only be inherited and defined there
- Pure virtual function does not have to be defined
	- ```C++
		T function_Name(parameter_List) = 0;
	```
- Only in declaration of virtual function
	- If pure virtual function must have a definition, must be defined outside the function the scope of the class

##### Classes with pure virtual s are abstract base classes

- Define interface for derived class to override
- Cannot directly create object from abstract base class

##### Re-factoring

- Re-design class hierarchy, move operation / data from 1 class to another
	- Usually for better readability + easier to understand
	- The operational code remain the same, but require a recompile

### Access control + inheritance

- Each base class controls the accessibility of data members in the derived class

##### “protected” members

- Data members shared with derived class, but not un-related classes
	- Blend of private + public
	- Can also be accessed by “friend” classes
- The derived class can have access to inherited protected data member within the derived type object
	- No special access to protected data member in the base type

##### “public”, “private” + “protected” inheritance

- Access control: access specifier for each data member in base class + access specified in derivation list
	- Derivation access specifier does not provide access control for the current derived class
		- Controls access of the derived class from the current derived class
	- Derivation access control will be passed down the inheritance hierarchy
	- “protected” derivation specifier will turn public member to protected, private member will remain private

##### Accessibility to Derived-to-base conversions

- Depends on access specifier of the derived class’s derivation
	- Only public derivation inherit can perform derived-to-base conversion
	- Member function + friend class of the derived class can use derived-to-base conversions (always accessible)
	- Member function + friend class inherited from the derived class can use derived-to-base conversion if derived class used “public” / “protected” derivation

##### Friendship + inheritance

- Friendship is not transitive + inherited
- A friend class to base class can access the base class’s object, include the sub-object in the derived class’s object

##### Exempting individual members (“using”)

- Use the “using” declaration to change the access level of name
	- ```C++
		public: // the same applies to protected + private access level
			using class_Name::variable_Name;
	```
- Only able to provide “using” declaration on names that the class are accessible

##### Default inheritance protection levels

- By defaults, all class’s data member are “private”, all struct’s data member are “private”
	- Should always explicitly specify “private” in class

### Class scope under inheritance

- Each class it’s a separate scope
	- Derived class is nested inside the scope of the base class
- If a name is not found in the scope of the derived class, the compiler will search for the name in the base class

##### Name lookup happens at compile time

- Static type of the object (reference / pointer) is set at compile time
	- Static type determine what members can be used
	- Name scope is determined by the static type

##### Name collisions + inheritance

- Names defined in the derived class hide use of the name in the base class
	- Derived class data member with the same name as the base class’s data member hides access to the base-class member

##### Using the scope operator to use hidden members

- Possible to use the scope operator “::” to access hidden name outside the scope
	- Bypass the normal lookup
- Steps of name lookup for inherited object
	1. Determine the static type of the object
	2. Look up the name in the static type, if not found, keep looking in its base class until the found / last classes being searched
		- If not found, the program won’t compile
	3. Once found, perform type checking, make sure the call is valid
	4. If the call is valid, generate the code
		- If a virtual function call, generate code to determine which version of function to use at run time
		- If a normal function, generate normal function call

##### Name lookup happens before type checking

- Inner scope function does not overload function declared on the outer scope
	- If derived-class function has the same name as the base-class function, the base-class function are hidden
		- Event if functions have different parameter list + return type

##### Virtual functions and scope

- If based + derived virtual function have different arguments
	- No way for the compiler to call the derived virtual function through pointer / reference of the base class
	- Function with the same name as virtual function but different argument will be treated as a separate, local, non-virtual member function

##### Overriding overloaded functions

- Virtual function can be overloaded
	- Derived class must override each overloaded version of the virtual function to use all available version of the virtual function
- Use the “using” keyword to add all overloaded instance of the virtual function to the scope of the derived class
	- Derived class only needs to define a selected overloaded virtual function + take inherited definition for the rest
	- Overloaded functions must be accessible by the derived class

### Constructors + copy control

#### Virtual destructors

- Each class control how object are created, copied, moved, assigned + destroyed
  - If not specified, compiler will synthesized (could be deleted function) for the class
- Destructor runs when a dynamically allocated pointer is deleted
  - All pointer will be base type, compiler needs to run different code for different derived class
    - Therefore, destructor should be a “virtual” function
  - Undefined behavior if the destructor is not virtual
- Base almost always need destructor for derived class, even if the base class might not need a destructor (especially abstract base class)
  - Existence of destructor is not a indication of existence of assignment operator + copy constructor

##### Virtual destructors turn off synthesized move

- If a class defines a destructor (virtual / explicitly defined as synthesized), compiler will not synthesized a move operator

#### Synthesized copy control + inheritance

- Synthesized copy control, constructor, assignment operator + destructor for base + derived class behave in similar fashion
  - In derived class, synthesized operations will directly call operations from the base class
    - Base class operations needs to be accessible + not deleted
- Derived will call all the available base destructor until the root of hierarchy
- If the base class don't have move operation, derived class also will not

##### Base classes + deleted copy control in the derived

- In certain condition, define copy-control + constructor certain way in the base class could define them as "deleted" in derived class
  - If constructor + copy-controls is not accessible / deleted in base class, they are deleted + inaccessible in derived class
  - If destructor is deleted / inaccessible in base class, copy-control is "deleted" in derived class

##### Move operations + inheritance

- Base class should define move operation if it's needed by derived class
  - Base class needs to explicitly define move constructor even to default (synthesized)
    - Must also explicitly define the copy control

#### Derived-class copy-control members

- Copy / move operation must operate on the base part + new member from the derived class

##### Defining a derived copy / move constructor

- Copy the base-class constructor
- To use the base copy / move constructor, pass the reference of the derived object to the base class copy / move constructor
  - The Base class copy / constructor will be matched, covert the derived object to a base class object parameter

##### Derived-class assignment operator

- For derived-class to have assignment operator, base class must assign it explicitly
  - Derived class assignment operator will call the base class operator to assign the base part
- Derived constructor / assignment operator can use the synthesized / user-defined base constructor / assignment operations

##### Derived-class destructor

- Derived destructor is only responsible of deleting the resources allocated by derived class
  - The data members are implicitly destroyed + destructor are internally called
- Derived destructor runs, first, then the base class, all the way until the root of the hierarchy

##### Calls to virtual in constructors + destructors

- Objects are incomplete when construing + destructing
  - Base part of the object will be constructed first + destructed last
- Compiler treats constructor + destructor as they change the type of the object
  - Object will always be the same class as the constructor

#### Inherited constructors ("using")

- Derived class can use constructors from the base class

  - Class can only inherit direct base's constructor

  - Cannot inherit default, copy + move constructors

  - ```c++
    using base_Class::base_Class; // inherit base_Class's constructor
    ```

- The "using" declaration ask the compiler to generate code

  - Generate a constructor with the same parameter list

  - ```c++
    derived_Class(params) : base_Class(args) {}
    ```

- Derived class's data are default initialized

##### Characteristics of an inherited constructor

- Constructor inherited using the "using" keyword does not change access level
- Cannot specify "explicit" + "constexpr" to "using", passed construction will maintain the "explicit" + "constexpr" status from the base class
- If base class constructor have default argument, Compiler will generate multiple constructors in derived class, each with 1 of the default argument omitted
- If base class have multiple constructors, all constructors will be copied over
  - Constructor defined in the derived class with the argument list will not be inherited
  - copy, default + move constructor will not be inherited

### Containers + inheritance

- To use container to store object of inheritance hierarchy, should always store the object indirectly
  - Container cannot change type at runtime

##### Put (smart) pointers in containers

- Container typically holds pointer of base / derived type of the object
  - The version of the function called depends on the dynamic type of the object
- Smart pointer of a derived type can be convert to a smart pointer of the base type (implicit conversion)