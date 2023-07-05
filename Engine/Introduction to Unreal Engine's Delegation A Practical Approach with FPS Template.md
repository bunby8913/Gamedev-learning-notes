# Introduction to Unreal Engine's Delegation: A Practical Approach with FPS Template

## Introduction

​	Delegation in Unreal Engine serves as an object that holds a reference to a method, where the type of which is unknown until runtime. Methods can be triggered by broadcasting the delegate, which enables event-driven design and development. At its core, Unreal Engine's delegation utilizes C++ function pointers. However, it significantly enhances security, contextual clarity, flexibility, and data sharing, thus making function pointers more straightforward and convenient.

## Function Pointers Recap

​	A function pointer can be defined as an ordinary pointer that references a function. The type of this pointer is determined by the function's return type and argument list. Unlike regular functions, function pointers can be used as parameters or return types for other functions. The compiler can determine the type of function pointer by using the "auto" / "decltype" keyword along with the function it intends to point to. However, the return type must still be converted to a pointer type.

## Steps to Implement a Delegate

1. First, declare the delegate, specifying its return type (any dynamic delegate must return 'void') and parameters. Delegate is usually declared in the header file.

2. Create a delegate object within the class (usually in the same file it's declared) where it will be utilized.

3. Connect/subscribe/bind the function to the delegate object. This process can be carried out either in C++ or Blueprint. Where a Single-cast delegate can only have a single function as a subscriber, a multi-cast delegate allows multiple functions to be subscribed to the same event, specifically the delegate object.

4. Once functions have subscribed to the delegate, the delegate can be invoked using the 'broadcast' function on the delegate object to activate subscribed functions, just don't forget to provide all the necessary arguments.

   ```c++
   OnUseItem.Broadcast();
   ```

## Case Study: FPS Startup Projects

​	The FPS template project contains 2 user-defined delegate objects to allow the player to walk to a rifle actor placed within the level, automatically pick up the rifle, and fire projectiles when the primary action key bind is used. This is accomplished using 3 C++ classes and a blueprint.

##### Delegation in 'Pickup_Component'

​	The 'Pickup_Component' is the scene component of the rifle weapon. It defines the "physical" form and interaction of the actor. The "pickup_component" utilizes two delegates: one pre-defined "On_Actor_Overlap_Begin" and one user-defined. The user-defined delegate is declared, and the corresponding object is created in the header file; the delegate object is public and allows access from another object.

```c++
// Declaration
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnPickUp, AMyProject2Character*, PickUpCharacter);
// Initialization
FOnPickUp OnPickUp; 
```

- Here we are declaring a dynamic, multi-cast delegate with 1 parameter, a pointer to a player character object

​	'Pickup_Component' is a class derived from 'U_Sphere_Component,' containing a special delegate triggered when an actor overlaps the sphere component. 

```c++
OnComponentBeginOverlap.AddDynamic(this, &UTP_PickUpComponent::OnSphereBeginOverlap); // Register the onSphereBeginOverlap function to OnComponentBeginOverlap (inherited)
```

​	In the "Pickup_Component" class, we define a member function subscribing to this special delegate. This member function is responsible for determining if the overlapped is a Player character, and if so, broadcast the user-defined delegate and un-register the current member function from the On-Actor-Overlap delegate.

```c++
// Take all 6 parameters from the inherited function
void UTP_PickUpComponent::OnSphereBeginOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
  // Checking if it is a First Person's Character overlapping
  AMyProject2Character* Character = Cast<AMyProject2Character>(OtherActor);
  if(Character != nullptr)
  {
     // Notify that the actor is being picked up
     OnPickUp.Broadcast(Character);
     // Unregister from the Overlap Event so it is no longer triggered
     OnComponentBeginOverlap.RemoveAll(this);
  }
}
```

​	The subscribing process for the user-defined delegate occurs in the Blueprint actor class. The delegate object passes its single parameter (A pointer to the player character object) is passed to the function subscribing to the delegate, which will be explained in more detail later.

##### The 'Weapon_Component' Class

​	Derived from the 'Actor_Component' class, the 'Weapon_Component' class represent the logic/gameplay portion of the weapon. This includes functions to attach the weapon to the character, storing and activating supporting animation and sound data, determine and spawning the projectile class at the appropriate location, and ultimately, firing the projectile.

​	The 'Attach Weapon' function is the function subscribed to the 'Pickup_Component' delegate, which is activated when the 'Pickup_Component' object overlaps with another actor. The function first validates the parameter, then attach the root component to the pointed player character, and finally, Subscribes its Firing function to the player character's delegate object.

```c++
// Bind the fire function to the character "onUseItem" delegate
     Character->OnUseItem.AddDynamic(this, &UTP_WeaponComponent::Fire);
```

​	The "Weapon_Component" is also responsible for cleaning up after itself. At the end of the game, The "Weapon_Component" object will remove the subscription from the character's delegate object

```c++
Character->OnUseItem.RemoveDynamic(this, &UTP_WeaponComponent::Fire);
```

##### The "BP_Rifle" blueprint actor

![Delegation_IMG](D:\Code\Gamedev-learning-notes\Engine\Delegation_IMG.png)

​	The "BP_Rifle" creates a loose coupling relationship between the "Weapon_Component" and the "Pickup_Component" through composition. The blueprint function subscribes the "Attach weapon" function from the "Weapon_Component" to the delegate object on "Pickup_Component." An equivalent C++ code would be the following

```c++
OnPickUp.AddDynamic(this, Weapon_Component::AttachWeapon(Player_Character));
```

##### Delegation in Player Character class

​	The player character also contains a user-defined delegate object declared and initialized within its header file. 

```c++
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnUseItem); // Delegate declaration
UPROPERTY(BlueprintAssignable, Category = "Interaction")
  FOnUseItem OnUseItem;
```

- The player character has a dynamic multi-cast delegate that does not take any arguments

​	This delegate object is wrapped within a function and bound to a player input action. When the input action is triggered, the function is called, which will broadcast the delegate object

```c++
void AMyProject2Character::OnPrimaryAction()
{
 	// Trigger the OnItemUsed Event
 	OnUseItem.Broadcast();
}
```

​	Initially, the function it binds to is a null pointer, and no action is performed when the event is triggered. The 'Attach Weapon' function uses the character reference to subscribe the firing function to the player character's delegate object.

### Conclusion

By breaking down the steps, the concept of Unreal Engine's delegation becomes less daunting and more approachable. The practical example from the FPS startup project provides a useful illustration of the application of this powerful feature in game development. By mastering these concepts, you can unlock greater flexibility and modularity in your Unreal Engine projects. In the next article, I will go into more detail about the different types of delegation available in the engine, their available return and some more advanced feature of Delegation in Unreal

### Reference

1. https://benui.ca/unreal/delegates-intro/