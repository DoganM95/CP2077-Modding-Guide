# Cyberpunk 2077 Modding Guide

## Intro
A comprehensive guide to modding Cyberpunk 2077, focusing on script and database modifications. It covers REDmod, advances to Redscript, and concludes with TweakXL.

## Preliminaries
Root Directory Reference: `./` denotes the game's root directory, e.g., `C:/Games/Steam/Cyberpunk 2077/`.

# Getting started

## Setting up REDmod

First, get REDmod. It's a dlc for cyberpunk, which needs to be installed by going to the game"s settings in steam, then to dlc"s and enabling the REDmod dlc. This will create new folders in the game directory, including
- `./tools/redmod/bin`
- `./tools/redmod/scripts`
- `./tools/redmod/tweaks`

## Mod using REDmod

The reason, there is almost no info on how to create a redmod mod, is its huge drawback. If two mods try to modify the same file, it will conflict and the user needs to decide, which one to keep. However, here is how to do it.

### Activate

Run the redmod.exe, click the gear (settings) and enable mods. 

### Create a mod (logic-manipulation)

The sub-folder `./tools/redmod/scripts` contains all the game-logic that cyberpunk runs on. These are mainly C# classes and functions, containing stuff like how much should it cost, if you want to respec your character (reset all perks).
Using the global search function of your ide is a good approach, trying to find what is of interest and needs to be modded. In this case, searching for "respeccost" contained the following function in the results.  
  
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

Now the mmodified copy of `playerDevelopmentSystem.script` needs to be placed in `./mods/NoRespecCost/scripts/playerDevelopmentSystem.script`, containing the changed function, as well as all the other code. 

### Create a mod (database-manipulation)

The sub-folder `./tools/redmod/tweaks` contains data structures which make up a database, that cyberpunk takes its values from. These are mainly C# structs, containing stuff like the ammo capacity of a car with weapons. In this case, the said ammo capacity will be modded.

Create a new folder with the new mod's name under `./mods`, e.g. `./mods/IncreasedCarAmmo`.  
Then create `./mods/IncreasedCarAmmo/info.json`, which  contains this example:

```json
{
    "name": "IncreasedCarAmmo",
    "description": "More cap for car integrated guns.",
    "version": "1.0.0",
    "customSounds":    [ ]
}
```
After finding a value, which could make the game more fun by modification, the whole file containing it needs to be copied just as in logic part, including their relative path. So in this case  
- Create a copy of the file `./tools/redmod/tweaks/base/gameplay/static_data/database/items/weapons/ranged/custom/custom_vehicle_weapons/generic_vehicle_weapons.tweak`
- Modify the `Base_Vehicle_Power_Weapon_Technical_Stats` function from the following

```cs
Base_Vehicle_Power_Weapon_Technical_Stats : StatModifierGroup
{
	statModifiers = 
	[
		{
			value = 1.f;
		} : NumShotsToFireModifier, 
		{
			value = 0.1f;
		} : CycleTimeModifier, 
		{
			value = 50.f;
		} : MagazineCapacityModifier, 
		{
			value = 1.f;
		} : ProjectilesPerShotModifier, 
		"BaseStats.ProjectilesPerShotCombinedModifier"
	];
}
```

to this, to increase capacity to 555 per "mag":

```cs
Base_Vehicle_Power_Weapon_Technical_Stats : StatModifierGroup
{
	statModifiers = 
	[
		{
			value = 1.f;
		} : NumShotsToFireModifier, 
		{
			value = 0.1f;
		} : CycleTimeModifier, 
		{
			value = 555.f;
		} : MagazineCapacityModifier, 
		{
			value = 1.f;
		} : ProjectilesPerShotModifier, 
		"BaseStats.ProjectilesPerShotCombinedModifier"
	];
}
```

Now the mmodified copy of `generic_vehicle_weapons.tweak` needs to be placed in `./mods/NoRespecCost/tweaks/base/gameplay/static_data/database/items/weapons/ranged/custom/custom_vehicle_weapons/generic_vehicle_weapons.tweak`, containing the changed value, as well as all the other code. 

### Conclusion on Redmod

- The base source code file will essentially be overwritten with the modified one, so the all the remaining unchanged code also needs to be kept in the modded file.  
- Assuming there are 2 different mods, manipulating the same file, one will get overwritten. This is where The next, superior method comes into play.  

## Mod using Redscript (logic-manipulation)

Redscript is like an advanced version of redmod. It is able to replace only specific functions instead of whole files, making it alot more fine-granular and reducing conflict potential to a minimum.  
Here are some key aspects:
- Redscript usse a [swift-like syntax](https://wiki.redmodding.org/redscript/language/intro/redscript-in-2-minutes)
- Files are saved with `.reds` extension
- Documentation can be found [here](https://wiki.redmodding.org/redscript/)

### Create a mod 

Fist, [redscript needs to be downloaded](https://github.com/jac3km4/redscript) and installed by unpacking the latest release into the game directory. 

Now to modifying the `GetTotalRespecCost()` function done above in redmod part, but the redscript way.  
- First create a parent directory in `./r6/scripts`, in this case `./r6/scripts/NoRespecCost`
- Create the file containing the mod, e.g. `./r6/scripts/NoRespecCost/freeRespec.reds`
- Create a function to override the stock one, with only the following content:

```swift
// you can replace existing in-game methods by specifying the class they belong to
@replaceMethod(PlayerDevelopmentData)
public final const func GetTotalRespecCost -> Int32 {
	return 0;
}
```

Starting the game will now have the mod applied.  
There are more annotations, that can be used to e.g. add a method instead od replacing it, or wrap it. More on that in the [redscript docs](https://github.com/jac3km4/redscript)

## Mod using TweakXL (database-manipulation)


