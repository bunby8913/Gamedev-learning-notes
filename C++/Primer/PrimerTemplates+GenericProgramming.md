# Primer templates + generic programming

- Templates are the foundation of generic programming
  - Blueprint to create classes / functions

### Defining a template

#### Function templates

- A formula to generate type-specific version of the function

  - ```c++
    template <typename T>
    T1 function_Name(T variable1, T variable2)
    {
        //Code...
    }
    ```

- The template parameter list cannot be empty

  - Comma separated list, surround by "<>"
  - Define local variable to be a specified type, but does not initialize the variable
    - Variable initialized at run-time

- The type "T" is determined at compiler time

  - With in the template function, "T" is use to refer a type

##### Instantiating a function template

- Compiler uses the argument of the function call to determine the type of "T" (template arguments)
  - The compiler will instantiate a specific version of function from the template
    - The instance function will replace template parameters with template arguments

##### Template type parameters

- In most case, type parameter behave same to built-in / class specifier
  - Can be return type / function parameter type
  - Can set pointer to the type

- Each type parameter must proceed by "class" / "typename"
  - "typename" is the preferred prefix, however, some legacy program maybe using "class" keyword


##### Non-type template parameters

- Template can take non-type parameter to customize the behavior of the template 
  - Specified with the type name

- Value supplied by user / deduced by the compiler
- Can be integer, pointer / references to object / function type (Must be constant expression)
  - Integral type must be constant expression
  - Pointers / reference must be static life time (Value known at compile time)


##### "inline" + "constexpr" function templates

- Function template can be "inline" + "constexpr", keyword needs to be specified after the template parameter list

##### Writing type-independent code

- Function parameter needs to be a reference to const
  - Ensure function can be used on type cannot be copied
  - More efficient to operate on large objects
- Limit the amount of operator requirement to support more types

##### Template compilation

- Compiler only generate code when instantiate a specific template
- Template header include definition + declarations

##### Compilation errors are mostly reported during instantiation

- 3 stages where compiler will flag errors for a template
  - Compiler will look for syntax error when compiling the template itself
  - Compiler will perform arguments check when it sees the use of template
  - Compiler will check type-related error during instantiation
- The caller needs to ensure the type passed to the template support operation used by the template
  - Un-supported type can only be detected at the last stage (instantiation)

#### Class templates

- A blueprint to generate classes
- Class template cannot deduce the type of template parameter
	- Must supply a list of template argument (type) inside the angle brackets	

##### Defining a class template

- ```c++
  template <typename T> class class_name {
  }
  ```

- Use template parameter as place holder and write the class as usual
	- “T” will be replaced by template argument type in during instantiate

##### Instantiating a class template

- Class template require a list of explicit template arguments
  - The template arguments is used to instantiate specific class using template
  - Each instantiation should be considered as independent class
    - No relationship between them

##### References to a template type in the scope of the template

- Class template is not the name of the type
  - Name of the type have to include template arguments
- Class template often use other template, which also have to supply template arguments to
  - Pass the template argument of the class to each templates within the class

##### Member functions of class templates

- Member function can be defined inside + outside the class
  - Internal member functions are implicitly “inline”
- Template member function technically doesn’t belong to a class (Ordinary function)
	- External template member function require the same template parameter
		- add "template" keyword + provide class template parameter list in angle brackets
  - ```c++
    template <typename T>
    ret-type class_Name<T>:: member_Function(param_list)
    {
    }
    ```
- Needs to specify the function is part of a template

##### Instantiating of class-template member functions

- Class template member functions are only instantiated when used
	- Allow some type that doesn’t support all the operations to be created + used

##### Simplifying use of a template class name inside class code

- If the member function is within the scope of the template class, template argument is not mandatory
	- Within template class scope, compiler treats reference to template already include template argument’s parameter
	- ```C++
		class_Name& == class_Name<T>& // within the scope of the class template
		```
##### Using a class template name outside the class template body

- The return type appear outside the scope of the template class
	- Need to specify the type (class template + template argument type)

##### Class templates + friends

- class template can be friend with normal class
	- Class template’s “friend” will have access to all instantiating of the template class
	- As a “friend”,  template class can control which instantiating gets access to the other class

##### One-to-one friendship

- Friendship between a instantiations and its friend is the most common friendship of class template
	- Template and the friend template must be declared before assigning friendship
- By specifying the template parameter, friend template object can only access data within the friend of the same type
	- Both object have to be the same instantiation type

##### General and specific template friendship

- Non-template class can make a template as or specific instantiations of the template as friend
	- ```C++
		template <typename T> friend class template_Class_Name; // for normal class, all instance of the template_Class_Name is friend with the class
		template <typename X> friend class template_Class_Name; // All instance of the template class is friend with the template class
		```

##### Befriending the template’s own type parameter

- can make the template parameter type as friend
	- ```C++
		template <typename T> class template_Class_Name
		{
		friend T;
		}
		```
- Template could be friend with a built-in type

##### Template type aliases (“typedef”)

- Use “typedef” to bind a name to a instantiated class
	- ```C++
		typedef template_Class<string> strClass;
		```
- Cannot bind a name using “typedef” to a template
	- Must specify the type
-  However, can use the “using” keyword to define type alias for template
	- ```C++
		template <typename T> using alias_Name = vector<T>;
		```
- Use the left hand operant name to represent right hand template operant
	- Needs to provide template argument for the type alias
- Possible to fix one or more template parameter
	- Once fixed, template argument cannot be changed
##### “static” members of class templates

- Template class can declare “static” members
- Each class (instantiation) will have their own, unique “static” member (1 per class)
- To define a “static” member of class template, first needs to specify the scope of the member + indicate its part of a class template
	- Similar to member function definition
- To access “static” member of a class template, must specific instantiation (template arguments)
	- Or with a instantiated template object, can access the data member directly through scope operator + access operator

#### Template parameters

- Template parameter can be any name
	- Just a place holder for the incoming type

##### Template parameters + scope

- Parameter becomes valid after being declared
	- The scope of the name ends after the end of the template declaration
	- Hides any declaration of the of the name from the outer scope
- Error to re-define template argument as another variable
- Each template argument must have different name

##### Template declarations

- Must include template parameter
	- The name of the parameter does not affect the declaration + definition of the template
- Template declaration should be at the top of the file, before any usage

##### Using class members that are types

- Compiler won’t know the property of class member until instantiation (type / static)
	- By default, compiler assume name access using scope operator is “static”
	- To access type data member using scope operator, needs to explicitly state the name if a type (using the “typename” keyword)
	- ```C++
		template <typename T>
		typename T::some_Type();
		```

##### Default template arguments

- Default arguments can be applied to class + function template
  - i.e. Used to set the default functions (callable object) to process the input
    - Pass a template functions so the type of the function can be determined at compile time

- When calling the function, user does not have to provide the optional arguments
  - Optional arguments needs to match return type, argument type of the default argument

- A argument can be default only if all the arguments on its right is also a default argument

##### Template default arguments + class templates

- Normally, class template's name follows by a brackets

  - The brackets include the template type used for instantiation

- If class template has a default type + to use the default type, leave the bracket pair empty (Do not provide any template type information)

  - ```c++
    templase <class T  = int> template_Class{
    }
    template_Class<> some_Class; // Some class are an template_Class with int type
    ```

#### Member templates

- Member function of a class (template or ordinary) could be a template
  - Member templates
  - Must not be virtual

##### Member templates of ordinary classes

- Member template must start with the template parameter list

  - ```c++
    template <typename T> ret_Type
    ```

- The member template function can be called with a ordinary + temporary object

- Template member function can be passed as parameter for pointer operations

  - Like other template function, function won't instantiate until used

##### Member templates of class templates

- Class + member will have independent template parameters

  - Member template are function template

- Functions needs its own template type parameter, cannot override the class template parameter
- To define a member template outside the class template's body, must provide class + function template parameter list

  - ```c++
    template <typename T>
    template <typnmaet FT>
    	class_Name<T>::Func(FT a, FT b);
    ```


##### Instantiation + member templates

- Supply both class + member template parameters to instantiate a member function
  - Compiler is able to deduce the type of member template base on input argument

#### Controlling instantiations

- Possible for multiple object file to have the same instantiation
  - Situation can be avoided with "explicit instantiation"
- Ensure that there could be several "extern" declarations but only 1 definition for the instantiation
  - Compiler will not generate instantiation of the template if it sees the "extern" keyword in the file
  - The "extern" declaration needs to be before any appearances (instantiation)
  - Compiler will only generate code at instantiation definition

##### Instantiation definitions instantiate all members

- Instantiation version of the class template will instantiate all member function
  - Normal class template will only generate the code for the function about to be used
  - Explicit instantiation should only be applied to types that can be used by all members of the template

#### Efficiency + flexibility

- A case study of how the library smart pointer types are designed

##### Building the "deleter" at run time ("shared_ptr")

- Type of the deleter is not known at run time
  - Must be stored as a pointer
- Pointer on the "shared_ptr" should point to a helper object, which contain a delete function

##### Binding the deleter at compile time ("unique_ptr")

- The deleter is a part of the type
  - The type supplied by a template parameter
  - Template code are generated at compiler time, binding the deleter type at compile time
- Avoid cost of indirect call

### Template argument deduction

- Determining the template arguments from function arguments
  - find a template argument that best matches the given call

#### Conversions + template type parameters

- The passed argument determines the function's parameter
  - Only a limited amount of automatic conversion is allowed for function parameter to use
    - Compiler generates new instantiation to fit the parameter
  - "const" conversion: "const" parameter can be replaced with the equivalent "non-const" object
    - Objects are copied
  - Array / function conversions: Will apply automatic pointer conversion
    - Array will return the first element
    - Function argument will be converted to pointer to function's type

##### Function parameters that use the same template parameter type

- For a template type parameter can only be used as multiple function parameter, if they are the exact same type
- To use normal conversion, Each parameter must be represented by a different function parameters
  - Needs to make sure the operator support implicit conversion between the type

##### Normal conversions apply for ordinary arguments

- Functions parameter can have ordinary parameter types (Not a template parameter)
  - Receive normal arguments, can apply normal argument conversion

#### Function-template explicit arguments

- Help to solve many issues when function's return type differ from the parameter list

##### Specifying an explicit template argument

- Apply explicit template argument when some of the parameter list cannot be deduced (i.e. does not take a parameter)
  - Specified in "<>" + right after the function name
    - Does not have to supply template argument type for all, the compiler is able to deduce the type for arguments
    - Reading parameter from left to right, only able to omit the right parameters

##### Normal conversions apply for explicitly specified arguments

- Once a template parameter argument are explicitly specified, normal  conversation can be applied to that parameter
  - All implicit conversion becomes available

#### Trailing return types + type transformation

- Use trailing return type to determine the return type base on the type of arguments of the function template

  - ```c++
    auto func(T a, U b) -> decltype(a){
    }
    ```

  - Avoid explicit determine the template arguments, reduce the burden from the user

##### The type transformation library template classes

- Obtain the element type using the type transformation template

  - Used for template metaprogramming (not covered)

- Use the "remove_reference" template, Uses 1 template type parameter + 1 type member, stores the referred type of the template type parameter

  - Will also strip the references, return the element itself

  - ```c++
    auto func(T a, U b) -> typename remove_reference<decltype(a)>::type{
      } // To get the type of element without reference
    ```

  - Must use the keyword "typename" to let the compiler we are passing a type

- There are many other transformation template, working similar to "remove_reference"

#### Functions pointer + argument deduction

- Function pointer can be used to point to an instantiation of a template function
  - Must have matching return type + arguments type
- Template arguments must be able to determine the function pointer type
  - Template type must be unique with each function pointer
    - Can be solved if calling function pointer with explicit template arguments
      - Specify the arguments type of the template function

#### Template argument deduction + references

- Normal reference binding rules apply
- "const" are low-level, not top-level

##### Type deduction from L-value reference function parameters

- If passing a ordinary reference, the passing type must be a L-value reference
  - The passed in type determine if the template arguments will be "const" or not
- If passing a const reference, can be bind to any type of argument
  - Even R-values
  - In this case, template parameter will not be "const", "const" if part of the function parameter, the "const"-ness will not be passed down


##### Type deduction from R-value reference function parameters

- The function takes R-value arguments, and can take L-values as result of reference collapsing
  - The deduction behave the same as ordinary L-value reference

##### Reference collapsing + R-value reference parameters

- Normally, cannot bind R-value reference to L-value
  - But when passing L-value to function parameter -> R-value reference parameter, Compiler deduce template type to be a L-value reference
- If indirectly create a reference to a reference, the references will "collpase"
  - the L-values reference (could be L-value / R-value) can be collapsed down to a L-value reference
- Therefore, R-value reference function parameter can take both L-value + R-value reference as input

##### Writing template functions with R-value reference parameters

- Depends on if the argument parameters is a R-value / L-value references, the type of template argument changes
  - Could be a value / references
- Depends on the type of template arguments, values are copied / bind to
- Could lead to unexpected result
  - Where type transformation is encouraged to avoid error like this
- R-value references are not generally used in template functions
  - For overloading purposes / forwarding arguments

#### Understanding "std::move"

- A good example of template using R-value references
- The "move" operation can bind R-value reference to an L-value

##### "std::move" definition

- ```c++
  template <typename T> typename remove_reference<T>::type&& move(T&& t) {
      	return static_cast<typename remove_reference<T>::type&&>(t);
  }
  ```

- The function takes a R-value reference, with reference collapsing, the function can take both L-value + R-value references

##### The theory behind "std::move"

- If passing a R-value references
  - Deduced type will be non-reference (ordinary object)
  - "remove_reference" will return the plain object
  - Will return a R-type reference
  - the function's parameter will take a R-value reference
  - The "static_cast" will not change the type, remain as a R-value references

- If passing a L-value reference
  - Deduced the type to be a references to the object
  - remove_references will determine the type to be a plain object
  - The return type will be a R-type reference
  - function parameter will collapse to a L-value reference
  - The "static_cast" will cast the L-value -> R-value reference


##### "static_cast" from L-value -> R-value references is permitted

- Language forces the conversion to happen explicitly using "static_cast"
  - Prevent accidental conversions
  - Usually just easier to use the "move" function instead

#### Forwarding

- Forward 1 or more arguments from 1 function to another forwarded-to function
  - Also pass the argument type
    - "const"-ness, L-value vs. R-value
- Usually the references parameter will cost a lot of headaches
  - Reference value are usually copied, not bounded, changes will not be reflected on the original reference

##### Defining function parameters that retain type information

- The parameter needs to preserve the L-value + "const" of the argument
- Template type parameter should be defined using R-value reference
  - Preserve "const" as "const" is low-level on reference
  - Preserve "L-value" property through reference collapsing
- However, still cannot forward R-value
  - R-value will be provided a name for forwarding, changing it into a L-value

##### Using "std::forward" to preserve type information in a call

- Part of the "utility" header
- A template, must be called with an explicit argument type
  - Return a R-value reference to the explicit argument type "T&&"
- If arguments are R-value, forward will preserve reference as a R-value
- If arguments are L-value, through reference collapsing (on argument + on return type) the argument will be collapsed to a L-value reference

### Overloading + templates

- Function templates can be overloaded by ordinary / other template functions
  - Same overloading rules apply (different # of argument / type of arguments)
- Function templates are always viable, since only viable version of the template function will be instigated
- All viable function are ranked by conversion, the best matched function will be selected
- If there are multiple best-match functions
  - Select the non-template function if there are only 1 of them
  - If there are no non-template functions, select the template functions that are more specialized
- Otherwise, call is ambiguous

##### Multiple viable templates

- When normal function matching cannot determine the best match, the more specialized template are being used
  - Otherwise, the specialized will never be called

##### Missing declarations can cause the program to misbehave

- The declaration of the template function with specific arguments must be in the scope
  - Otherwise, the compiler will instantiate a template version for the argument type, result in unexpected behaviour

### Variadic templates

- Template function / class that take different # of parameters
  - parameter pack: The varying parameters
    - Template parameter pack vs. function template pack
- Use ellipsis "..." to represent a pack
  - The name of the type is placed after the ellipsis
  - The ellipsis represent a list of 0 or more non-type parameter
  - In the function parameter list, represent a pack by template parameter pack name followed by ellipsis + type name
- Compiler deduce the type from function's argument + # of arguments in the pack

##### The "sizeof..." operator

- The "sizeof..." operator returns a constant expression, represent the # of type + function parameters in the pack
  - Does not evaluate the arguments

#### Writing a variadic function template

- Especially useful when we don't know the # of argument / types of argument being passed

- Often recursive, process the first argument than recursively call the other arguments

  - Usually requires a non-variadic version of the function to stop the recursion

- The list argument can be processed by function that takes different # of arguments

  - ```c++
    template <typename T, typename... args>
    ret_type &func(T1 variable1, const T &t, const args&...rest)
    {
        // operations to process t
        return func(variable1, rest...);
    }
    ```

  - In this case, The function separate out the first argument in the argument pack, process it + pass the rest of the arguments down

    - Every iteration the first argument in the pack is removed

  - If the argument pack have 1 arguments left, call the non-variadic version of the function

    - Both functions are viable, but the non-variadic template function is more specialized, better matched
      - Needs to ensure the non-variadic template function is in scope to be called

#### Pack expansion (Iterate through the pack)

- Parameter pack only permit size checking + expansion
- Unroll the pack, generate code to iterate every arguments in the pack at compile time
- Trigger an expansion by adding ellipsis on the right of the pattern (the type name)

##### Understanding pack expansions

- Expanded the pack into constituent parts
- Pattern in an expansion applies separately to each elements in the pack

#### Forwarding parameter packs

- Parameter packs can be passed unchanged to another function
- A 2-step process
  - Define function parameter as R-value reference to a template type parameter
  - Use "forward" to preserve the argument type being passed
- Expand the template parameter + function parameter using pack expansion
  - Ensure that R-value reference will stay as R-value

### Template specializations

- Specialized version of class/ function temp late are needed when we don't / can't use the template version
- A separate definition of a template, 1 or more template parameters are specified to be particular type

##### Defining a function template specialization

- Defined with the "template" keyword + empty pair of angle brackets "<>"
  - Needs to supply arguments for every template parameter
- The supplied parameter type needs to match the template type
  - i.e. reference, pointer, "const", etc.
- Add the specialized parameter in front of the parameter type from the original template

##### Functions overloading vs. template specializations

- Supplying definition to use a specific instantiation of the original template
  - Bypassing the compiler
  - An instantiation of the original template, not a overload operation
- The same function matching rules apply
- Declaration of the original template must be in scope
  - Compiler will generate code using the original template, result in unexpected behavior

##### Class template specializations

- Usually define a specialization of a template to use custom data type
  - To specialize a class template, must be in the same namespace of the original template
- The class member can be defined in / out of the class scope
  - The specialized class needs overloaded call operator, default constructor + copy-assignment operator

##### Class-template partial specializations

- Class template specialization does not require every template parameter
  - Partial specialization: user only have to supply arguments that are not fixed by the specialization
- Uses the same name as the template is specializes
  - Specify the argument for template parameter after the class name

##### Specializing members but not the class

- Possible to only specialize specific member function of class
- Use the same "template" keyword + empty angle bracket
  - Make sure to declare the scope of the function
  - Include the specialized type parameter in the function's template parameter list 
