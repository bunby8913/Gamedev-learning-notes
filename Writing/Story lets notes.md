# Story lets notes

- A way to organizing narrative content with more flexibility
- Basic definition
  - Content: Series of dialogue, cut scenes, rewards, quests
  - Pre-requisites: A series of requirements (gameplay tags) determine when to play the content
  - Effects: the effects of the content, changes to the world state after the content
    - Could lead to fulfillment of other pre-requisites

### Quality based + salience-based structures

### How to write long interactive novel that doesn't suck

- Merge plot branches aggressively
  - Issues: Choice in earlier games no longer matters, Reduce the significance of choices dramatically
  - Solution: Delayed branching, earlier choices don't branch right away, stored and used to determine of outcome in later decision
  - Will always go in the next chapter, however, choices made in the previous chaptre will affect the game in later chapter

##### Keep track of delayed branches with numbers

- Store delayed branches as stats
  - We should store them as gameplay tags in Unreal games
- At early game, allow the player to distribution stats freely
- In later games, set stat requirements for options
- Using numeric score allow choices to be meaningful
  - Otherwise would have to keep track of every decision in later games, become complex

##### The entire game is about stat numbers

- Every chapter should be affected by the stat + should affect decision in a meaningful way
- Stat will represent conflicts created in the story, change accordingly

### Standard patterns in choice-based games

##### Time cave: 

- Heavily branched sequences

- All choice are equally important, other story lines does not re-merge
- No need for state tracking

###### Effects: 

- Good narrative for freedom + open opportunities
- Short play through, increase re-playability
- Different branch leads to completely different settings
  - WOW quests in later expansions

##### Gauntlet: 

- linear central thread, branches in the story will eventually join the central thread again

- 1 central story
  - Will have fail conditions + optional content
  - Yakuza, the last of us, uncharted
- Multiple endings will split at the end
- Does not keep states

###### Effects

- realize constraint path
- The structure of side content are very important
- Linear story, player will see important contents

###### Deadly gauntlets

- prune the tree with failure

###### Friendly gauntlets

- short-range rejoining

##### Branch + bottleneck

- The game will branch + rejoin the main story line
- Utilize state tracking
- Plot can branch and never re-join

###### Branch + bottleneck effects

- Events happens in chronological order
- Used to reflect growth of character
- Allow player character development under a story framework
  - Playthrough very similar at the beginning, diverse as game progress
- i.e. Fall out new vegas
- Require a large gameplay area to explore

##### Quest

- Distinct branches, rejoin to a 1 or few winning endings
- Quiet common to re-merge similar quest
- State tracking within each quests

###### Quest effects

- Focused on the setting
- Quest are organized by zone
- Grounded + consistent worlds, player character's situation constantly changin
- Story of the world are told largely through quests
- i.e. FF14

##### Open map

- Quests that divided into regions
- Some quest are time-sensitive
- Travel between major faction + location reversible
  - Easy to travel between areas
- Heavy state-tracing for exploration + narrative progress

###### Open map effects

- Narrative are slower pace, less directed
- i.e. Zelda breath of the wild
- Focus on exploration of the world

##### Sorting hat

- Apply branch + bottleneck at early game
- Help a player to choose a major branch to explore
- Linear branches, quest lines
  - Could appear like gauntlets (with little choices affecting the ending)
- State tracking in early game

###### Sorting hat effects:

- Compromise between story depth + player choices
- System needs to show player that different branches will have different plots + unique experiences
- Writers may have to write a different game for different branches



##### Floating modules

- No central plot, no pre-set story, modular encounter base on states
- Could be randomly generated

###### Floating module effects:

- Hard + challenging to write for
- Need variety in contents to avoid looking boring
- Could make the game more grindy for stats

##### Loop + grow

- Loop around a central thread
- Unlock features + state from replaying the game + state tracking

###### Loop + grow effects

- Maintain regularity of the world + develop narration
- Perform similar activities in familiar space
- I.e. Hades

###### Spoke + hub

- Game has several branches
- All can return to central set of nodes
- Player can replay each Spoke multiple times
- i.e. Diablo

### Threaded conversation