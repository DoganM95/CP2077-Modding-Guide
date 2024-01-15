# Intro

Personal guide for cyberpunk 2077, to get started with modding by writing own scripts and db modifications. The guide contains methods (redmod) that are not popular but necessary to do at least once, to understand how modding for this game works. Afterwards, the most state-of-the-art method can be used only and other methods be left behind, to ensure gradual learning.  
- Every time `./` is used, it references the game's root directory, e.g. `C:/Games/Steam/Cyberpunk 2077/`

# Getting started

## Setting up REDmod

First, get REDmod. It's a dlc for cyberpunk, which needs to be installed by going to the game"s settings in steam, then to dlc"s and enabling the REDmod dlc. This will create new folders in the game directory, including
- `./tools/redmod/bin`
- `./tools/redmod/scripts`
- `./tools/redmod/tweaks`

## Mod game-logic (scripts): REDmod

The reason, there is almost no info on how to create a redmod mod, is its huge drawback. If two mods try to modify the same file, it will conflict and the user needs to decide, which one to keep. However, here is how to do it.

### Activate

Run the redmod.exe, click the gear (settings) and enable mods. 

### Browse the code

The sub-folder `./tools/redmod/scripts` contains all the game-logic that cyberpunk runs on. These are mainly C# classes and functions, containing stuff like how much should it cost, if you want to respec your character (reset all perks).
Using the global search function of your ide is a good approach, trying to find what is of interest and needs to be modded. In this case, searching for "respeccost" contained the following function in the results.

### Create a mod (logic)

Create a new folder with the new mod's name under `./mods`, e.g. `./mods/NoRespecCost`.  
Then create `./mods/NoRespecCost/info.json`, which  contains this example:

```json
{
    "name": "NoRespecCost",
    "description": "The total respec cost will always be 0, eliminiting the cost.",
    "version": "1.0.0",
    "customSounds":    [ ]
}
```
After finding a function, which could make the game more fun by modification, the whole file containing it needs to be copied, including their relative path. So in this case  
- Create a copy of the file `./tools/redmod/scripts/cyberpunk/systems/playerDevelopmentSystem.script`
- Modify the `GetTotalRespecCost()` function from the following

```cs
// ... Code before
public const function GetTotalRespecCost() : Int32
{
    var cost : Int32;
    var basePrice : Int32;
    var singlePerkPrice : Int32;
    basePrice = ( ( Int32 )( TweakDBInterface.GetConstantStatModifierRecord( T"Price.RespecBase" ).Value() ) );
    singlePerkPrice = ( ( Int32 )( TweakDBInterface.GetConstantStatModifierRecord( T"Price.RespecSinglePerk" ).Value() ) );
    cost = basePrice + ( singlePerkPrice * ( GetSpentPerkPoints() + GetSpentTraitPoints() ) );
    return cost;
}
// ... Code after
```

to the following, to return a respec cost of 0 always, making respec free:

```cs
// ... Code before
public const function GetTotalRespecCost() : Int32
{
    return 0;
}
// ... Code after
```

Now the mmodified copy of `playerDevelopmentSystem.script` needs to be placed in `./mods/NoRespecCost/scripts/playerDevelopmentSystem.script`, containing your change, as well as all the other code. The base source code file will essentially be overwritten with the modified one, so the other functions need to be kept in the modded file.



