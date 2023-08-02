# Effective C++ - Designs and declarations

- Designing + declaring good C++ interface

### Make interfaces easy to use correctly + hard to use incorrectly

- If the client uses the interface incorrectly, the interface is partially to blame
  - Consider the mistakes that will occur
- Solution: Use custom new types as arguments, provide more information for the client + prevent interface usage errors
  - in addition, restrict the type of the value
    - `enum` is an option, but not type-safe, using `static` functions might be the better choices (Ensure the function is loaded + initialized before usage)
  - Restrict available operation on the type
  	- Use `const` as much as possible + the type should behave consistent with built-in types
- keep function name consistent across multiple types (Use `size` member function to get how many objects are in the container)
- Design the interface to make the client to the least amount of things, reduce # of client errors
- `shared_ptr` is an easy way to eliminate client errors
	- Will use more resources than raw pointer, but reduction in client error is more beneficial

### Treat class design as type design

-  Designing class should be utilize all the available class tools (Function / operation overloading, memory allocation / deallocation, define object initialization / destruction), just like built-in types
-  Design an effective class require careful consideration of many issues
  -  Object construction / destruction: Include memory allocation + deallocation
  -  Object initialization vs. object assignment: Determines the different between constructor + assignment operator
  -  How to pass by value with the new object
  -  Legal value restrictions: Determines error checking the member function has to perform + affect the exceptions to throw
  -  Type within inheritance graph: Determines if the function are virtual / non-virtual + especially declare destructors as virtual
  -  Kind of type conversion: Conversion should either be implicit / explicit
  	-  Explicit conversion are natively safer, use explicitly stated function
  	-  Implicit conversion uses type conversion function / non-explicit constructor with a single parameter
  -  What operator + functions to include: Determine what kind of functions the type will include
  	-  Any unwanted standard function should be declared as `private`
  -  New type vs. non-member function / template: Some functionality could be better achieved with non-member function / template over derived class
  -  How general is the new type: If trying to create a family of types, should create a type template instead
  -  "Undeclared interface": Guarantee the type offers i.e. exception safety, resource usage, etc.

### Prefer pass-by-reference-to-const to pass-by-value

- C++ pass by value by default
	- Functions will copy all the parameters + return a copied value (could be expensive operation)
		- When copying an object for function, every data member needs to be constructed + destroyed for every function call

- Pass by reference-to-const is a more efficient alternative
	- Keeping the reference `const` to ensure the function will not modify the object being passed in
		- Not have to worry about is the object is copied

- Avoid the "slicing problem" with derived class object
	- If passing object as a copy as a base object, only the base object constructor will run, the derived class portion of the object will be sliced off
		- Cannot recover the sliced off version from a copied initialized base object

	- Pass by reference-to-const essentially pass the pointer to object, can be treated as either base / derived class base on polymorphism

- Any build-in types + STL Iterator + function are designed to be passed by value, efficient to copy + won't cause the slicing problem
- Small object != good to copy-by-value
	- Small object (usually with pointer data member) will have to copy the pointed object when copy-by-value (Potentially very expensive operation)
	- Small object could become big during development cycle

- User-defined copy-by-value could come with performance penalty
	- Compiler might treat built-in type different from user-defined type
		- Refuse to store user-defined type in register but will store pointers address in there

- Apart from built-in type + STL iterator + function, every other type should be pass-by-reference-to-const > pass-by-value

### Don't try to return a reference when you must return an object

- Never pass reference to object that doesn't exist
- Reference is simply a different name for an already existing object
  - For a function to return a reference to an object, the object must exist / the function must create the object + return the reference to it
- Functions can only create new object on heap / stack (have to pay for construction cost)
	- On stack is a bad idea, local object create by function will be deleted when function ends, return a reference broken object, undefined behavior
	- On heap is also a bad idea, dynamically allocated object needs to be re-allocated at some point, cannot promise the collection of the object
- Creating a `static` object in the function call + return the reference to static member create even more problem
	- Thread-safety concern
	- When multiple functions call are made + result are compare, only 1 object is returned between multiple function calls, result will always be true
	- No, a static array will not work

- In this case, return a new object is the way to go (pass-by-value)
	- Small price to pay for the correct behavior
	- In many case, compiler will be able to optimize code without affect the behavior


### Declare data member `private`

- Keeping all data member `private` + use interface keeps public interface clean + consistent
- Using function interface to access data member provide more fine control of read-write access
- Encapsulation: Data member implementation can change without affecting the interface
	- i.e. from getting data member to computation
	- Tradeoff between memory usage vs. access speed
- Implementation flexibility: Delegation to call / notify other function, verify data member + apply pre + post conditions, apply synchronization
- Public = unencapsulated = unchangeable
- `protect`ed data member are not encapsulated
	- Any derived class that uses `protect`ed data member will be affected with any changes (unknowably amount of code might be broken)
	- Hard to change any `public` / `protect`ed data member once its in use

### Prefer non-member non-friend functions to member functions

- OOP will suggest to bundle up data member + function to operate on them, indicate member function is the better choice (Incorrect)
	- Encapsulate as much as possible, but member function have the opposite effect
- Non-member function has better packaging flexibility
- Encapsulation: Codes that are hidden away from the view
	- The more the code is encapsulated -> the fewer interface is exposed -> the greater flexibility in changing the code -> the less people the code change will affect
	- Member function can access many more parts of the class than non-member, non-friend function, therefore, are less encapsulated
		- Note `friend` function still have access to `private` class member
		- Note the non-member, non-friend function can always be a member function of another class (a utility class)
	- The non-member, non-friend function simply provide encapsulation of functionality that already exist, client should always be able to call the encapsulated functions individually
- Its generally better to separate different categories of convenience functions interfaces into different header files + under the same namespace
	- Also how C++ STL organize code
	- Not possible with class member function
- Makes it easy for client to extends the code further by adding more non-member, non-friend functions
	- Can be included in separate header files + the same namespace
	- Cannot be achieved with inheritance

### Declare non-member function when type conversions should apply to all parameters

- Classes should generally not support implicit type conversions (with some exception)
	- Integer implicit conversion should generally be allowed
- Intuitively, functions that could perform implicit type conversion should be a member function, but have unintended behavior
	- Implicit conversion will not work on certain object, cannot call a member function using implicit conversion
- Only parameters in the parameter list can be implicitly type converted
	- Hence non-member functions should be used for any argument type conversion
- Set the function as `friend` to the argument object should be avoided unless necessary
	- Unless the function needs to access `protected` data member of the object

### Consider support for a non-throwing `swap`

- `swap`: An important way to achieve exception-safe programming

	- Good to deal with assignment to self

- The default `swap` function involves copying assignment operator + copy constructor

	- Copy 1 of the object to a temp object, swap both objects, and store the temp object to the other object
	- Copying object are usually unnecessary + slow in performance

- The "Pimpl idiom" (Pointer to implementation) approach

	- The object is managed by another class, the swap of pointer is happening between the management object
	- Require a specialized `std::swap` to call object member function to actually perform the swap
		- However, the `std::swap` solution does not work for class template, require a function overload that could result in undefined behavior (adding additional functions to the std namespace)

- The solution: Creating a non-member `swap` function that calls the member `swap` function in a specific namespace

	- To add the option to use `std::swap` too, add the function to the namespace right before usage

		- ```c++
			template <typename T>
			void doSomething(T& obj1, T& obj2)
			{
			    using std::swap;
			    swap(obj1, obj2);
			}
			```

	- Compiler will determine which version of `swap` is available + should be called

		- `swap` in the namespace > `std::swap` specialized > `std::swap`

- If default `swap` implementation (Copy operations) is not efficient enough, use pointer swap

	- Create `swap` member function for class template, all value swap between 2 objects
		- Function should never throw exception
		- `swap` usually applied to built-in types + built-in types don't throw exception
	- Create a non-member `swap` function in the same namespace + template, call the member `swap` function
	- A class should have its own specialized `std::swap` function, call the member `swap` function



