# Effective C++ - Templates + generic programming

### Understand implicit interface and compile-time polymorphism

- Ordinary OOP requires explicit interfaces + run-time polymorphism
	- Explicit interface: Interface determined by header source file
	- Run-time polymorphism: The type of function going to be called is base on the dynamic type of the object, determined at run-time
- Template + generic programming are the opposite, uses implicit interfaces + compile-time polymorphism
	- Implicit interface: The template will use a set of functions that must be present in the type for the template to compile
	- Compile-time polymorphism: Template will be instantiated during compile time, instantiating function template determine the function to be called
		- Similar to how compiler determine which overloaded function to call (base on return type + function parameters) + determine which version of the virtual function to call base on the dynamic type of the object
- Template class does not care for the implementation of the function within each potential class, as long the function is defined and provide appropriate resulting types, the template can compile
	- Or if the resulting type is compatible / can be implicitly converted to the proper type
	- Implicit interface are also just a set of valid expression

### Understand the 2 meanings of `typename`

- In template declarations, `class` + `typename` means the same thing

	- Usage should depends on preference + code standard specification

- 2 kinds of name in a template

	- Dependent names: Type names that are dependent on the template parameter object
		- Nested dependent name: Names that are nested within a class
		- Generally more difficult to deal + parse with, depends on the implementation of the class
	- Non-dependent names: Types of name that are not depend on any template parameter

- When the compiler encounter a nested dependent name in template, it will assume its not a type name unless its specified by add the `typename` keyword in front of the dependent name variable

	- In short, any nested dependent type name requires the keyword `typename` in front of it

	- Exception: the `typnname` keyword is not required in base class identification + member initialization list

		- ```c++
			template <typename T>
			class DerivedClass : public Base<T>::Nested {
			    public;
			    explicit DerivedClass(x):Base<T>::nested(x)
			    {
			        typename Base<T>::Nested nestedTempVar;
			    }
			}
			```

- Tip: For any long nested dependent type name, consider using a a `typedef` to shorten the name + include `typename` in the `typedef` too

### Know how to access names in templatized base classes

- Usually a good practice to inherit from base class to add functionality to the base class

	- Reasonable for the inherit a template class to use any non-virtual base function inherited in the derived template class
		- However, compiler could not find the definition for non-virtual base function in derived class in a template

- Problem: The compiler does not know class inheritance history

	- Template class is not instigated until compile time + its interface detail is unknown
	- When a template class derive from another template class, no way to know what functions the base template class contain

- Total template specialization: Provide specialized implementation of a template for a specific type

	- The compiler will first check through specialization, if no specialization found, the general template will be used

- Compiler recognize that class template may be specialized, not all version of the class template may have the same interface, compiler generally refuse to look in templatized base class for definition

- 3 solutions to the problem do disable C++ from stop looking in templatized base classes 

	- Use `this`: This will instigate the object, which will determine + make template's implementation concrete, make the function usable

		- ```c++
			this->sendClear(info);
			```

	- Use `using`: Similar to how we solved the issue with hidden inherited names, use the `using` keyword can bring hidden name to derived class's scope

		- ```c++
			using MsgSender<Company>::sendClear;
			...
			    sendClear(info);
			```

	- Explicitly declare the scope of the function is part of the base class

		- ```
			MsgSender<Company>::sendClear(info);
			```

		- The least desirable solution, if the function are `virtual`, this will remove any virtual binding behavior for the function 

- Make promise to the compiler, that all generalized bass class template will support the operation

	- Compiler will believe the promise until proven otherwise (If the instigated class does contain the needed function)


### Factor parameter-independent code out of templates

- Template can lead to code bloat: binary code full of similar code + data
	- Source code will look clean, but the object will be chunky
- Perform name commonality + variability analysis on the code
	- If multiple function implementation contain the same block of codes, move the  common code to a different function, let all function that requires the same implementation to call the same function
	- If multiple classes contain the same set of function + data members, move them to a new class + use inheritance / composition to retrieve common feature 
- Same analysis can be applied to template too
	- However, replication in non-template are explicit, duplication between classes are obvious
		- Replication in template class implicit, all classes are generated by the same set of functions
- Use inheritance to have single code block applied to multiple derived template classes
	- Declare + define the function in the base class, use parameter to pass any data from derived class to base class function
		- Make sure the function is not declared as `inline`, in that case, the function code will be added to each derived class, back to bloated code
	- The derived class should `private`ly inherit from the base class, only facilitate implementation, does not represent "is-a" relationship
- Data can either be passed through argument list / as private base data member
- Tradeoffs between have bloated objects, better compile time (constant) vs. better code organization / optimization, decrease the size of the executable, could also improve run-time speed of the program
	- Tradeoffs between unnecessary data stored in the derived object (Most derived class can access the private data member used for the base class function in a different way) vs. lose encapsulation in the base class (resources management complications)
- The type parameter could also lead to bloated object code. Different types might have the same binary representation underneath, implementation member function might be similar in nature

### Use member function templates to accept "all compatible types"

- Is hard for user-defined smart pointer to act like real pointers

	- Implicit conversions: Real pointers support many implicit conversions
		- Implicit convert from derived class pointer to base class pointe / convert pointer to non-const object to pointer to const object
	- No inherit relationship between different classes in template
		- Needs to program conversion explicitly

- Would be impossible to write out every single constructor needed in the smart pointer classes

	- Means we will have to change the smart pointer implementation every time a new class is inherited from the chain of hierarchy
		- Will need unlimited amount of constructor

- Constructor template > multiple constructor functions

	- ```c++
		template<typename T>
		class SmartPtr{
		    public:
		    template<typename U>
		    SmartPtr(const SmartPtr<U>& other) // Creating SmartPtr<T> with SmartPtr<U>
		}
		```

	- Generalized copy constructor: a constructor that creates a new object as a copy of multiple type of existing object

		- Should not be declared as `explicit`, just like built-in pointer conversion requires no cast, the smart pointer should do the same

- Generalized copy constructor needs to be restrained

	- Shouldn't be possible to create derived class pointer with a base class pointer
		- Breaking the definition of public inheritance
	- Solution: utilize the `get` member function, return a copy of the built-in pointer held by the smart pointer
		- Use the raw pointer stored in the smart pointer to perform implicit conversion (Reduce the problem to implicit conversion between U and T)
		- If raw pointer cannot be implicitly converted, then the code won't compile

- Using member function for assignments

	- Use `explicit` keyword on constructor with types that are implicit conversion should not be allowed
		- In the case of `shared_ptr`, normal pointer, `weak_ptr` + `auto_ptr` should not be implicitly converted
	- The generalize copy constructor is not explicit, allow any other type implicitly convert to `shared_ptr`

- Generalized copy constructor != "normal" copy constructor

	- When using a generalized copy constructor and both type are the same, the compiler could instantiated a "normal" copy constructor for the type
	- To establish fine control, "normal" copy constructor should be declared explicitly in code to avoid synthesized version being generated

### Define non-member functions inside template when type conversions are desired

- For implicit type conversions on all arguments, should use non-member function instead, allow for more flexibility
- However, the same does not apply to non-member template functions
  - The compiler needs to figure out which function to call when instigate
  	- The compiler know the name of the function, but does not know the type of the arguments it should call for
  	- Each parameter' type are considered separately
  - Implicit type conversion functions are not considered during template argument deduction
  	- Implicit conversion are considered during function call, but not before a function call + when the function are been instigated
- Solution: `friend` declaration in template class can refer to a specific function
	- Declare the appropriate non-member function as friend
	- Declared function can consider implicit conversion during function call
	- When declaring the function, the `<T>` of the template name can be omitted
- However, The `friend` function will only have declarations, but not definition
	- Easiest solution: Add the function definition to function declaration
- Set the non-member function `friend` not to access template private member, but the only way to declare non-member function inside a class
	- The function would be `inline` by default, if the workload is too heavy, consider use the `friend` function to call an external function to complete the task
		- The "Have the friend call a helper" approach
	- The helper function could also be a template function
		- In this case, the `friend` non-member function does the implicit conversion + pass the converted variables to the helper (template) function after

### Use traits classes for information about types

- `advance`: Part of STL utility template, move a specific iterator a specific distance
  - Most Iterators does not support moving > 1 distance at once (`+=`), instead, (`++` / `--`) must be used
- STL iterator categories review
  - Input iterator: Only move forward 1 step at a time, read only + read only once
  - Output Iterators: Only move forward 1 step at a time, write only + write only once
  	- i.e. write pointers to write to the output file
  - Forward iterator: Only move forward, but can read / write to the same location multiple times
  	- Used to implement singly linked list data structure
  - Bi-directional iterators: Adds the ability to move backwards to forward iterator
  	- Used to list, set, multiset, map + multimap
  - Random access iterator: The most powerful iterators, included iterator arithmetic (jump forward + backward at any distance in constant time)
  	- Built-in pointer, used by iterators for Vector, deque + string
- C++ use a "tag_struct" to determine the type of the iterator
  - a inheritance relationship between iterators
- Default solution: Use the lowest common denominator of all, increment once per steps 
  - Issue: not taking advantage of the constant time access with random access iterator
  - Ideally, we want different implementation depends on the type of the iterator
- Use trait to get information about a type during compilation
  - A technique + convention followed by C++ programmer
  - Works on built-in type just as well as user-defined type
  	- Can't nest information within built-in types / pointers
- Trait classes: A template class (Trait are usually `struct` by convention)
	- Each trait classes stored a struct that represents the trait to recognize the object
- For each object that needs to be recognized by trait, use `typedef` to store the trait with a chosen identify name
- Provide template + specialization contains the type to support
- Looking for a conditional construct depends on the type during compilation (using overloading)
	- Different parameter provide different overload
	- Different overloaded constructor will take different object to construct the template function differently
- Create a master function / function template calls the workers, passing information provided by trait class

### Be aware of template metaprogramming

- Writing template based C++ programs, execute during compilation, generate a set of C++ source code for compilation
	- TMP was an unexpected consequence of the introduction of templates in C++
- 2 great strength of TMP
	- Make things that are almost impossible easy to do
	- Shift work from run-time to compile time
	- Smaller executable files, shorter runtimes, less memory requirement
- However, error that can be detected at run-time will not be detected during compile-time
- In many way, the trait approach (Determine which overloaded function to call to construct the template function) is the TMP approach
- TMP also allows invalid code to stay in the code base, however by split into separate function, compiler does not have to check that portion of the code if it's not used, avoid compilation errors stop program from compiling
- TMP are Turing-complete and are able to achieve anything a programming language is able to
	- TMP does not have traditional loop function, uses recursion instead (a core part of functional programming)
		- Uses recursive template instantiations
- The 3 examples of things TMP can do
	- Ensuring dimensional unit correctness: TMP can ensure that dimensional unit combinations in the code are correct
		- Used for early error detection
	- Optimizing matrix operations: TMP can create a expression templates, eliminate temporaries + merge loops, without changing syntax of the client's code
	- Generating custom design pattern implementation: TMP can implement policy-based design, create templates for independent design pattern with custom behavior
		- Generative programming
