# Unreal gameplay showcase - A simple reactive AI behavior

As a software developer, keeping up with the latest tech stack and learning on a daily basis is only half the battle; putting the knowledge into action is just as, if not more important, and we often learn the most when things don't go as planned. Following up on the short introduction of the ability task from last week, I came up with a small showcase based on the ability task I introduced. In this article, I will demonstrate the process of deploying a practical use case of the ability task and, ultimately, designing and implementing NPC character actions. To achieve it, we will use Unreal Engine's gameplay ability system and the Animation blueprint, AI behaviour tree, and more to make the action more feature-rich and enhance the overall player experience.

### Ability task re-cap

In case you missed last week's article, here is a quick recap on Ability Task. Ability task is an integral part of the Gameplay Ability System, the component that extends the single-frame executions of the Gameplay Ability. We used the Spawn Actor Ability Task as a starting point and took it as an opportunity to explore optimization tricks, implementation details and its myriad features.

I made some changes to the Ability Task from our previous discussion for this project. Instead of using a Target Actor or the ability system component to determine the location to spawn the actor, the Ability Task will now accept a location offset vector as input and use the offset to spawn the actor in front of the character some distance away. The ability task uses the avatar actor information gathered from the owning ability to calculate the direction and rotation of the spawned actor to perform the final spawn transform. This crucial step sets the stage for the NPC character's action.

### Gameplay abilities

The NPC character will use 3 distinct gameplay abilities for this NPC action sequence. Choosing gameplay abilities over basic character class functions isn't just for show. The choice makes the actions easier to edit, boost clarity, and apply gameplay effects or restrictions on both the ability's user and target. Overall, it creates a more flexible system for the gameplay designer to use. The 3 gameplay abilities are as follows. Let's dive into the abilities.

1. Spawn short wall ability: When activated, This gameplay ability will spawn a short wall ability actor using the previously mentioned ability task. In addition, the gameplay ability also stores the spawned actor as a reference on the character, and finally, walks to the dedicated location marked by the ability actor and kicks off the AI fighting sequence.
2. The Hide Behind Wall ability: This is pretty simple. The gameplay ability plays an animation montage of the character hiding behind the wall. The play rate of the animation montage depends on a float variable stored on the NPC character. The Gameplay Ability will end when the animation montage has been completed.
3. The Firing projectile ability: The firing projectile ability controls the spawning, direction, initial velocity, and the firing of the projectile. While object pooling for NPC characters isn't implemented at the moment (Since it's just a proof of concept), it's definitely an optimization trick worth considering in the future.

### Ability Actor - What is it?

If you are wondering what exactly is a "short wall ability actor," it's basically an Actor class derived from `AActor`, with an override virtual function and 2 additional components. Here's the gist

Begin play: Begin play is an inherited virtual function, which will be called when the actor first appears on the level. It is tasked to change the actor's Z-axis value smoothly over time using LERPs. 

Child actor component: The child actor is an "invisible actor" (Ah actor without static mesh); the actor represents the location relative to the wall where the NPC character should move to take cover

Blocking volume: This blocking volume springs into action when the NPC character is hiding, an invisible shield blocking any projectile coming directly to the wall. However, there is not much the blocking volume could do if the character got shot from the side or the back.

- ### NPC character setup

	Our NPC doesn't just pop up out of nowhere - he's got a pedigree! Inherited from a base character, the character has the ability system component set up right from the get-go. On top of the rest of the inherited data members, this NPC character still requires some additional variables and functions listed below

	- `IsFiring` (Boolean): This Boolean variable indicates the animation blueprint for when to enter and leave the firing state (Animation blueprint). 
	- `IsInCombat` (Boolean): A Boolean variable, we use this to represent if the character is in combat mode. This switch tells the AI Behavior system when to activate the combat gameplay abilities.
	- Actor reference: Remember the ability actor from earlier? This is where we store the actor reference, which is handy for accessing the actor and its internal function later on
	- Got Hit function: Activated when the character takes gameplay attribute damages, the got hit function is responsible for calculating the damage impact direction and ordering the NPC character to respond appropriately
	- A Tick function: The tick function will run at every game cycle. When the character is in combat mode, it will use this function to keep track of the player's location (enemy), complete with a creepy red laser trace following them.
	- An On Health changed function: it's no fun playing a video game if the enemy is invincible. This function is called every time the player's health attribute has changed. This function keeps track of the character's current health; if the character is out of juice (health = 0), the character will play a death animation depending on the state of the character, disable any collision detection and eventually fade out from the level
	- A Set of functions to turn on / off the firing state: This is a set of helper functions used by the behaviour tree task to transition from the Ducking animation state to the Firing animation state and back. The turn-on firing state function is called right before the task tries to activate the firing projectile abilities, and the firing state is turned off immediately.

### AI behaviors

Now that we've learned how the gameplay abilities are being made and how the NPC character has been set up, It's time to focus on the character's brain, the AI Behaviors tree component.

The AI behaviour tree system can be seen as a conditional to-do list. If the conditions are met, complete the task; if not, skip to the next item. The NPC character's behaviour tree is super compact, consisting of only 3 nodes. 2 of them are tasks the character should complete, and the third is a simple wait node, where the character takes a tiny break before going through all the nodes again. The execution conditions are stored on a blackboard with just single variables, a Boolean value which indicates if the character is currently in combat. Here's what each task does

- Hide behind the wall: This task will first call the generate random play rate function on the character to generate a new play rate for the hiding animation, just to randomize the time the character hides behind the wall a little bit, then activate The Hide behind wall ability.
- Fire projectile: Nothing special here. The task first switches on the NPC character's firing state and then activates the firing projectile ability before turning off the firing state again.

One thing to note is that the Behavior tree task executes within a single frame and does not have a task node similar to the Ability Task to extend the execution time. Therefore, we had to manually calculate the duration of the animation montage and delay for the duration before finish executing the current behaviour tree task before activating the next.

### Animation blueprint

Now, on to the animations. The character's animation is controlled through a simple yet efficient animation blueprint. Tailored to the NPC character, the blueprint contains 4 different states, let's break it down: 

- Idle: Idle is the default state of the character. The character will remain idle until the player aims at them; then, the game is on! 
- Moving: The moving state does not play a significant role in the action. It simply provides an animation for the NPC to move from location A to B.
- Ducking: Here is the interesting part: The Ducking animation is not the same as the hiding animation. This states it's the bridge between any animation sequence from behind the wall. It's the transition space that ensures that all animations flow smoothly.
- Shooting: Finally, the shooting animation simply loops and provides a visual indication that the character is shooting

As a side note, the hiding animation is achieved using animation montage and activated by a gameplay ability. The gameplay ability will also read the current hiding animation play rate variable from the NPC character and use that to determine the play rate of the animation montage, ultimately determining how long the character should be hiding behind the wall.

### Player character setup

The player character is responsible for initiating the interaction. Within the tick function of the player character, a continuous `line trace by channel` function is constantly performed. The function checks where the player character's weapon is pointed at every game cycle. When the line trace intersects with the game level, its hit information is captured, and the hit actor will be used to determine if the targeted actor is an NPC character. If it's the match, we would notify the NPC character through a public function, which activates the spawn short wall gameplay ability and enables its combat mode. The interaction is fluid and intuitive, offering the player a deeply engaging player experience.

### Gameplay tags setup

As we have mentioned earlier, one of the reasons that gameplay ability is preferred over simple member function is that it makes applying restrictions on both the ability's user and target extremely easy. This gameplay tag setup is the perfect example of just that. Imagine a scenario where the characters are engaged in a shootout. Naturally, the character being struck by a bullet should not continue to shoot seamlessly; being hit should cause some disorientation and hinder the character's shooting ability. To simulate that, I introduced a gameplay tag, `CharacterState.GotHit` and a gameplay effect to apply it. This tag is applied temporarily to the character hit by a projectile and will cancel and block any shooting abilities for a short period. This mechanism rewards the character with higher precision. 

### Summary

This showcase, although only as a proof of concept project, offered a good opportunity to get my hands dirty and apply my understanding of ability tasks, animation systems and AI behaviour into actions. Like any other prototype, there are still many areas for improvement. They serve as fuel, motivating me to dive deeper and refine further. I look forward to showcasing an even more refined NPC action.

### The Video showcase



