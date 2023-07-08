# Deep Dive into Unreal Engine's Delegation: Part 2 - Reference and Advanced Features

### Introduction

​	In this sequel to our earlier exploration of delegation in Unreal Engine, we dive deeper into the more complex aspects of this powerful concept. Building on our understanding from the FPS template project, we'll unmask the different types of delegates, learn the various way to bind functions, discover deprecated event delegates, and finally, reveal some advanced features. It's worth noting that the delegate object memory is dynamically assigned during runtime, and stored on the heap. As such, even though delegates can be copied, it's considered bad practice, and reference should be preferred. We'll start by discussing the different types of delegate.

### The Four Types of Delegate

​	Unreal Engine offers four different types of delegates, each delegate single-cast/multi-cast and static/dynamic delegate:

1. **Single-cast Delegate**: This type of delegate can only bind one function to the delegate object. Only the last function is "executed" if multiple functions are subscribed. To call another function with the delegate, use `Execute()`, or, to check if a function is bound to the delegate before executing, use `ExecuteIfBound()`. `IsBound()` is a supporting function for only checking if a delegate is bound by a function or not.
2. **Multi-cast Delegate**: As the name suggests, multi-cast delegates can trigger multiple subscribed functions simultaneously, akin to broadcasting to their subscribers. To call other functions with the delegate, use `Broadcast()`.
3. **Static Delegate**: These delegates bind their calling function at compile-time, and the function(s) cannot be changed at run time. While useful in certain circumstances, this means you can't alter the binding function in blueprints.
4. **Dynamic Delegate**: In contrast to static delegates, dynamic delegates can bind and change their functions. This enables us to bind functions in blueprints, as we did in the FPS template project. However, this flexibility results in a heavier overhead since the function's location must be located through name lookup before calling.

## Understanding Delegate Signatures

​	Delegate macros follow specific formats, differentiated by whether they require "payload" variables, support return values, or are single or multi-cast. Here's a quick guide:

- For a basic, single-cast, static delegate with no payload variables:

```
DECLARE_DELEGATE(DelegateName)
```

- For a delegate with function parameters (up to 8):

```
DECLARE_DELEGATE_ONEPARAM (DelegateName, Param1Type, Param1)
DECLARE_DELEGATE_<NUM>PARAMS(DelegateName, Param1Type, Param1, ParamType2, Param2, ...)
```

- For a delegate supporting return values (For static and dynamic single-cast delegates):

```
DECLARE_DELEGATE_RetVal (RetValType, DelegateName)
```

- To declare a multi-cast static delegate:

```
DECLARE_MULTICAST_DELEGATE...()
```

- To declare a single-cast dynamic delegate:

```
DECLARE_DYNAMIC_DELEGATE...()
```

- And finally, to declare a multi-cast dynamic delegate (very commonly used):

```
DECLARE_DYNAMIC_MULTICAST_DELEGATE ...()
```

Note that dynamic delegates don't support payload variables.

## Binding Functions to Delegates

​	Both single-cast and multi-cast delegates share similar functions, differentiated by the prefix: "Bind" for single-cast and "Add" for multi-cast. All multi-cast delegate functions return a handle of the added function, allowing its removal later. The available functions are:

- `BindStatic(&GlobalFunctionName)`: Calls a static function, either globally scoped or a class static.
- `BindUObject(UObject, &UClass::Function)`: Calls a UObject class member function via a TWeakObjectPtr; it won't be called if the object is invalid.
- `BindSP(SharedPtr, &FClass::Function)`: Calls a native class member function via a TWeakPtr; it won't be called if the shared pointer is invalid.
- `BindThreadSafeSP(SharedPtr, &FClass::Function)`: Same as BindSP.
- `BindRaw(RawPtr, &FClass::Function)`: Calls a native class member function with no safety checks.
- `BindLambda(Lambda)`: Calls a lambda function with no safety checks.
- `BindWeakLambda(UObject, Lambda)`: Calls a lambda function only if UObject is still valid; it does not check if the lambda function is safe.
- `BindUFunction(UObject, FName("FunctionName"))`: Usable for both native and dynamic delegates, it calls a UFUNCTION with the specified name.

## Deprecated Event Delegates

​	Since Unreal Engine 5.0, events have been deprecated and replaced by multi-cast delegates. The latter now supports most event functions and should be used instead. The event declaration is extremely similar to multi-cast delegates - simply replace "multicast" with "event" when declaring and calling the delegates.

## Sparse Delegates

​	Sparse delegates are a special, memory-optimized form of delegate. They've been used by many built-in components, including the sphere component, to provide native delegates for handling actor overlaps with the sphere component or collisions. They use only 1 byte of memory if not bound but are more expensive to operate. Hence, they should be used only for infrequently used delegates.

## Serialization

​	Serialization is the process of converting an object into a binary format, allowing the object to be saved on disk and accessed by blueprints in the future. Significantly, only dynamic delegates support serialization, making them incredibly handy when wanting to save the current state of a player character in a game. 
​	Consider the FPS template as an example. If we have multiple types of weapons, each with its own unique version of the firing function, storing the dynamic delegate object on the player character will save the current version of the firing function subscribed to the delegate with it. This feature is practical and an efficient way to optimize the game's save-load cycles. It ensures that the correct firing function remains bound, and gameplay can seamlessly continue upon reloading the game.

## Final Tips

​	As macros may struggle to recognize data structures using commas, it's best to use typedef to assign a new type name to structures that contain commas in their type structure.
​	In conclusion, effectively utilizing Unreal Engine's delegation system can significantly enhance your development workflow, and a concrete understanding of the basic of the unreal engine could be extremely beneficial in the long run.

### Reference

- https://benui.ca/unreal/delegates-advanced/
- https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/
- https://forums.unrealengine.com/t/what-is-sparse-delegate/480282