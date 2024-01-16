# Cyberpunk 2077 Modding Guide

## Intro

A comprehensive guide to modding Cyberpunk 2077, focusing on script and database modifications. It covers REDmod, advances to Redscript, and concludes with TweakXL.

## Preliminaries

- **Root Directory Reference**: `./` denotes the game's root directory, e.g., `C:/Games/Steam/Cyberpunk 2077/`.

## Setting Up and Modding with REDmod

The following mods for REDmod are pseudo-mods and just illustrate how modding would work. The mods are probably not practically usable in the game.

- **Installation**: Activate the REDmod DLC via the game's Steam settings.
- **New Folders**: added by REDmod
  - `./tools/redmod/bin`
  - `./tools/redmod/scripts`
  - `./tools/redmod/tweaks`
- **Activation**: two options  
  a) **Automatic**: Set game launch options in steam to `-modded --launcher-skip`   
  b) **Manual**: Launch `./REDprelauncher.exe`, access settings (cogwheel), and enable mods  


### Logic-Manipulation Mod

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



### Database-Manipulation Mod

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

### Notes on REDmod

- The base source code file will essentially be overwritten with the modified one, so the all the remaining unchanged code also needs to be kept in the modded file.  
- Assuming there are 2 different mods, manipulating the same file, one will get overwritten. This is where The next, superior method comes into play.  

## Logic Manipulation with Redscript

Redscript is like an advanced version of redmod. It is able to replace only specific functions instead of whole files, making it alot more fine-granular and reducing conflict potential to a minimum.  
The example this time will be a working one, with visible effect ingame.
- **Key aspects**:
  - Redscript uses a [swift-like syntax](https://wiki.redmodding.org/redscript/language/intro/redscript-in-2-minutes)
  - Files are saved with `.reds` extension
  - Is state-of-the-art for script mods and replaces REDmod
- **Installation**: [Download Redscript](https://github.com/jac3km4/redscript) and extract it directly into the Cyberpunk 2077 directory
- **Setup**: Start the game for Redscript setup. Verify installation by checking for `./r6/logs/redscript_rCURRENT.log`
- **Usage**: Place Redscript mods in `./r6/scripts` for automatic loading
- **Example**: Modding `GetTotalRespecCost()` the redscript way
  - Create a new file `./r6/scripts/NoRespecCost/freeRespec.reds`
  - Create a function in it, to override the stock one using
  ```swift
  // you can replace existing in-game methods by specifying the class they belong to
  @replaceMethod(ReloadEvents)
  protected const func GetReloadAnimSpeed ( const statType : gamedataStatType, weaponRecord : WeaponItem_Record ) -> Float {
      return 100.0;
  }
  ```
  - Start the game and check if it kicks in, this time the reload animation (only!) should be really quick
  - Check the [redscript docs](https://github.com/jac3km4/redscript) for more, e.g. `@addMethod` & `@wrapMethod`
- **Extras**: Tools for convenience
  - [redscript-ide-vscode](https://github.com/jac3km4/redscript-ide-vscode): vscode extension for syntax highlighting, definition peeking, autocompletion, etc.

## Database Manipulation with TweakXL


# Resources

- https://cdn-l-cyberpunk.cdprojektred.com/REDmod-docs.pdf
- https://github.com/jac3km4/redscript?tab=readme-ov-file
- https://wiki.redmodding.org/cyberpunk-2077-modding/for-mod-creators/modding-tools/redmod
- https://forums.cdprojektred.com/index.php?threads/redmod-tutorials.11107406/
- https://wiki.redmodding.org/cyberpunk-2077-modding/for-mod-users/users-modding-cyberpunk-2077
