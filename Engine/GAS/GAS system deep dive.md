# GAS system deep dive

### ASC

- Ability system component is the interface between all the component and the actor that uses the GAS
- ASC should be used as it is
- If actor needs to have attribute / gameplay effects stored between spawns, ASC should be placed on the player state
	- Needs to also increase net update frequency for the player state for multiplayer games (Use adaptive network update frequency)
		- Dynamically adapt the update frequencies of actor to save CPU cycles
- If any actor uses ability system component else where, they should use the `IAbilitySystemInterface` to override the `GetAbilitySysteCompoent()` function to points to the correct ASC for the actor
- ASC holds all the active gameplay effects in a active gameplay effect container, and holds all the granted gameplay abilities
	- Like any iterations process, the length of the list should not be changed during iteration to avoid index error
- ASC are typically constructed in owner actor's constructor
	- If needs to be replicated, needs to explicitly set replication to be true
- Gameplay tags are stored on the character

### Gameplay tag delegate

- ASC supports gameplay tag delegate

	- Gameplay tag delegate can be trigger when a tag is added/removed / the number of tag map count changes

- ```c++
	AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("Tag.Name")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
	```

### Ability tasks

- Ability task has a game-wide 1000 maximum concurrent ability task limit

### Gameplay cue

- Gameplay cue should be proper way to activate visual effects in game
- Gameplay cue are triggered by gameplay tag, the tags can be added to gameplay effects, and the gameplay cue with the matching tags will be triggered when the effect are executed
- Static vs. actor gameplay cue
	- Static: A class default U object, no instance, one-off effects (hit impact)
	- Actor: Spawn an U Actor, effects are repeated until removed
		- Looping effect that are associated with duration + infinite gameplay effect
			- Multiple visual effect can be added at once

		- Needs to use the option auto destroy on remove, so gameplay cu e tag can be added

- Gameplay cue parameters
	- Gameplay cues uses a `FGameplayCueParameters` to pass information to the cue
	- If the gameplay cue is triggered by gameplay effect, then the following data are automatically
		- Source + target tags
		- Gameplay effect level
		- Ability level
		- Effect context
		- Magnitude

	- If triggering gameplay cue manually, use `Sourceobjcet` to pass any data to the gameplay cue

- Gameplay cue manager: Component that scans game directory, load them into memory, prepare them for usage
	- In `DefaultGame.ini`, we change the directory to scan for faster scan time
	- Is possible to async load the gameplay cues to game as needed
		- Trade offs, however, if using SSD, the as needed loading delay is non-existence

	- There is a function called `ShouldAsyncLoadRuntimeObjectLibraries()`, should return a Boolean value that determine if async loading is possible or not, override the function in `UgameplayCueManager`

- Gameplay cue events:
	- `OnActive`: Functions called when the gameplay cue is added
	- `WhileActive`: Called once if the gameplay cue is active
		- gameplay cue notifier actor can use the tick for repeated call

	- `Removed`: Called when the gameplay cue is removed
	- `Executed`: Called when the gameplay cue is executed, once for static + tick for actor

- The initial effect should be on on active, use while active for any effect that is ongoing and later joiner can also see
- Gameplay cues should be considered unreliable

