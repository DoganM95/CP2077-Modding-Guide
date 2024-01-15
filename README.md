# Intro

Personal guide for cyberpunk 2077, to get started with modding by writing own scripts and db modifications. The guide contains methods (redmod) that are not popular but necessary to do at least once, to understand how modding for this game works. Afterwards, the most state-of-the-art method can be used only and other methods be left behind, to ensure gradual learning.  
- Every time `./` is used, it references the game's root directory, e.g. `C:/Games/Steam/Cyberpunk 2077/`

# Getting started

## Setting up REDmod

First, get REDmod. It's a dlc for cyberpunk, which needs to be installed by going to the game"s settings in steam, then to dlc"s and enabling the REDmod dlc. This will create new folders in the game directory, including
- `./tools`
- `./tools/redmod`

## Mod game-logic (scripts) using REDmod

The reason, there is almost no info on how to create a redmod mod, is its huge drawback. If two mods try to modify the same file, it will conflict and the user needs to decide, which one to keep. However, here is how to do it.

### Activate

Run the redmod.exe, click the gear (settings) and enable mods. 

### Browse the code

The sub-folder `./tools/redmod/scripts` contains all the game-logic that cyberpunk runs on. These are mainly C# classes and functions, containing stuff like what should happen, when a car is being stolen by the player (should the owner get angry?) etc.  
Using the global search function of your ide is a good approach, trying to find what is of interest and needs to be modded.

### Create a mod (logic)

Create a new folder with the new mod's name under `/mods`, e.g. `/mods/CarStealNoAngryOwner`.  
Inside that, create an `info.json`, which is then in `/mods/CarStealNoAngryOwner/info.json` and contains this example:

```json
{
    "name": "CarStealNoAngryOwner",
    "description": "When i steal someone's car, they should just be chill about it.",
    "version": "1.0.0",
    "customSounds":    [ ]
}
```
After the function to be modified has been found, the whole file containing it needs to be copied, including their relative path. So 
```cs
```
