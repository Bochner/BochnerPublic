# Mud Program (MPROG/OPROG/RPROG) Scripting Guide

This guide details how to use Mud Programs (MPROGs, OPROGs, and RPROGs) to add dynamic behavior to Mobs, Objects, and Rooms respectively.

## Table of Contents
- [Mud Program (MPROG/OPROG/RPROG) Scripting Guide](#mud-program-mprogoprogrprog-scripting-guide)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction](#1-introduction)
  - [2. General Syntax and Structure](#2-general-syntax-and-structure)
    - [Script Format](#script-format)
    - [Control Flow](#control-flow)
    - [Variables](#variables)
    - [Commands](#commands)
  - [3. Available Script Variables](#3-available-script-variables)
  - [4. Ifcheck Conditions](#4-ifcheck-conditions)
    - [Ifcheck Syntax](#ifcheck-syntax)
    - [String Operators (`mprog_seval`)](#string-operators-mprog_seval)
    - [Numeric Operators (`mprog_veval`)](#numeric-operators-mprog_veval)
    - [List of Ifchecks](#list-of-ifchecks)
  - [5. MPROGs (Mob Programs)](#5-mprogs-mob-programs)
    - [MPROG Triggers](#mprog-triggers)
  - [6. OPROGs (Object Programs)](#6-oprogs-object-programs)
    - [OPROG Triggers](#oprog-triggers)
  - [7. RPROGs (Room Programs)](#7-rprogs-room-programs)
    - [RPROG Triggers](#rprog-triggers)
  - [8. Special Commands in Progs](#8-special-commands-in-progs)

## 1. Introduction

Mud Programs allow builders to write scripts that are triggered by various game events. These scripts can make entities react to players, the environment, or time.
*   **MPROGs** are attached to mobile (NPC) definitions.
*   **OPROGs** are attached to object definitions.
*   **RPROGs** are attached to room definitions.

Each prog consists of:
*   **Type:** The event that triggers the prog (e.g., `GREET_PROG`, `SPEECH_PROG`).
*   **Argument List (`arglist`):** A string that modifies the trigger's behavior (e.g., a percentage, keywords, a VNUM).
*   **Command List (`comlist`):** The actual script commands to be executed, one command per line.

## 2. General Syntax and Structure

### Script Format
Scripts are a series of commands, each on a new line. Lines are processed sequentially.

'''
if rand(50)
  say Hello there, $n!
else
  emote $i grumbles.
endif
mpforce $n smile
'''

### Control Flow
Progs support `if`, `or`, `else`, and `endif` statements for conditional execution.
*   `if <ifcheck_condition>`
*   `or <ifcheck_condition>` (must be within an if block, before an else or endif)
*   `else`
*   `endif`

Ifchecks can be nested up to `MAX_IFS` (default 20) levels.

### Variables
Scripts can use special variables (e.g., `$n`, `$i`, `$o`) that are replaced with context-specific names or descriptions. See [Available Script Variables](#available-script-variables) for a full list.

### Commands
Most lines in a prog are standard MUD commands that the mob/object/room (via `supermob`) will execute (e.g., `say`, `emote`, `mpforce`, `mpoload`).

## 3. Available Script Variables

These variables are substituted into command lines before execution. They provide context about the trigger event.

| Variable | Represents                                       | Example Replacement (Name) | Example Replacement (Short Descr) | Notes                                                                 |
| :------- | :----------------------------------------------- | :------------------------- | :-------------------------------- | :-------------------------------------------------------------------- |
| `$i`     | The mob/object/room the prog is on               | `guard`                    | `a city guard`                    | The entity the prog is attached to.                                   |
| `$I`     | Short description of `$i`                        |                            | `a city guard`                    |                                                                       |
| `$n`     | The primary actor/instigator of the trigger      | `PlayerX` or `mobname`     | `PlayerX the Brave` or `a goblin` | Name is uppercased if PC.                                               |
| `$N`     | Full name/short desc of `$n` (visible)         |                            | `PlayerX the Brave` or `a goblin` | For PCs, includes title. For NPCs, short description.                 |
| `$t`     | The primary target of the action (char/obj)      | `VictimY` or `sword`       | `VictimY the Scared` or `a rusty sword` | From `void *vo`. Name uppercased if PC.                               |
| `$T`     | Full name/short desc of `$t` (visible)         |                            | `VictimY the Scared` or `a rusty sword` | For PCs, includes title. For NPCs/Objs, short description.             |
| `$r`     | A random PC in the same room as `$i`             | `RandomPC`                 | `RandomPC the Lucky`              | Name uppercased. Can be NULL if no other PCs.                       |
| `$R`     | Full name/short desc of `$r` (visible)         |                            | `RandomPC the Lucky`              | For PCs, includes title.                                              |
| `$o`     | The primary object involved in the trigger       | `potion`                   | `a bubbling potion`               | E.g., item given, worn, used.                                         |
| `$O`     | Short description of `$o`                        |                            | `a bubbling potion`               |                                                                       |
| `$p`     | The secondary object involved (from `void *vo`)  | `scroll`                   | `an ancient scroll`               | E.g., target object of a spell/action if `$t` is a character.         |
| `$P`     | Short description of `$p`                        |                            | `an ancient scroll`               |                                                                       |
|          |                                                  |                            |                                   |                                                                       |
| **Pronouns (for `$n` - actor):**                         |                            |                                   |                                                                       |
| `$e`     | Subjective: he, she, it (based on `$n`'s sex)    | `he`                       |                                   | Fallback: "someone" (if visible) or "it".                             |
| `$m`     | Objective: him, her, it (based on `$n`'s sex)    | `him`                      |                                   | Fallback: "someone" (if visible) or "it".                             |
| `$s`     | Possessive: his, her, its (based on `$n`'s sex)  | `his`                      |                                   | Fallback: "someone's" (if visible) or "its'".                          |
|          |                                                  |                            |                                   |                                                                       |
| **Pronouns (for `$t` - target, if character):**            |                            |                                   |                                                                       |
| `$E`     | Subjective: he, she, it (based on `$t`'s sex)    | `she`                      |                                   | Fallback: "someone" (if visible) or "it".                             |
| `$M`     | Objective: him, her, it (based on `$t`'s sex)    | `her`                      |                                   | Fallback: "someone" (if visible) or "it".                             |
| `$S`     | Possessive: his, her, its (based on `$t`'s sex)  | `hers`                     |                                   | Fallback: "someone's" (if visible) or "its'".                          |
|          |                                                  |                            |                                   |                                                                       |
| **Pronouns (for `$i` - self, if character):**              |                            |                                   |                                                                       |
| `$j`     | Subjective: he, she, it (based on `$i`'s sex)    | `it`                       |                                   | Fallback: "it".                                                       |
| `$k`     | Objective: him, her, it (based on `$i`'s sex)    | `it`                       |                                   | Fallback: "it".                                                       |
| `$l`     | Possessive: his, her, its (based on `$i`'s sex)  | `its`                      |                                   | Fallback: "it" (intended: "its'").                                    |
|          |                                                  |                            |                                   |                                                                       |
| **Pronouns (for `$r` - random PC):**                       |                            |                                   |                                                                       |
| `$J`     | Subjective: he, she, it (based on `$r`'s sex)    | `he`                       |                                   | Fallback: "someone" (if visible) or "it".                             |
| `$K`     | Objective: him, her, it (based on `$r`'s sex)    | `him`                      |                                   | Fallback: "someone's" (Note: code uses objective `him_her` but possessive fallback). |
| `$L`     | Possessive: his, her, its (based on `$r`'s sex)  | `his`                      |                                   | Fallback: "someone" (Note: code uses possessive `his_her` but non-possessive fallback). |
|          |                                                  |                            |                                   |                                                                       |
| **Utility:**                                               |                            |                                   |                                                                       |
| `$a`     | "a" or "an" determined by `$o`'s name          | `a`                        |                                   | Uses first keyword of `$o`'s name.                                      |
| `$A`     | "a" or "an" determined by `$p`'s name          | `an`                       |                                   | Uses first keyword of `$p`'s name.                                      |
| `$$`     | A literal dollar sign                            | `$`                        |                                   |                                                                       |

**Notes on Variables:**
* If a character/object variable points to something that doesn't exist (e.g., `$r` when no random PC is available, or `$t` if not applicable), it usually defaults to a generic term like "someone", "something", or "it", or may be an empty string.
* The exact character or object assigned to `$n, $t, $o, $p` depends on the specific trigger. See trigger descriptions.
* For OPROGs and RPROGs, `$i` refers to a special "supermob" that acts on behalf of the object or room. Its name/short_descr will reflect the object/room.

## 4. Ifcheck Conditions

Ifchecks are used in `if` and `or` statements to control script flow.

### Ifcheck Syntax
`if <check_name>(<argument_variable_or_value>) <operator> <value>`
Or for checks without operator/value (boolean result):
`if <check_name>(<argument_variable_or_value>)`

*   `<check_name>`: The name of the condition to check (e.g., `rand`, `ispc`, `ovnumhere`).
*   `<argument_variable_or_value>`: Often one of `$i, $n, $t, $r, $o, $p` to specify which entity the check applies to. Can also be a direct value (VNUM, percent, string, etc.) for some checks. If not a variable, its direct value is used.
*   `<operator>`: (Optional) A comparison operator. See below.
*   `<value>`: (Optional) The value to compare against (referred to as `rval` in code).

Example: `if ispc($n)`
Example: `if hitprcnt($i) < 50`
Example: `if name($o) == sword`
Example: `if clan($n) == "Jedi"`

### String Operators (`mprog_seval`)
Used when comparing strings (like names). `lhs` is the game value (e.g., from `name($o)`), `rhs` is the `<value>` from the script. Comparisons are case-insensitive.
*   `==`: Exact match.
*   `!=`: Not an exact match.
*   `/`: Substring (true if `rhs` is found within `lhs`).
*   `!/`: Not a substring (true if `rhs` is NOT found within `lhs`).

### Numeric Operators (`mprog_veval`)
Used when comparing numbers.
*   `==`: Equal to.
*   `!=`: Not equal to.
*   `>`: Greater than.
*   `<`: Less than.
*   `>=`: Greater than or equal to.
*   `<=`: Less than or equal to.
*   `&`: Bitwise AND (useful for flags).
*   `|`: Bitwise OR.

### List of Ifchecks
(Variable arguments like `char_var` usually take `$i, $n, $t, $r`. Object arguments `obj_var` usually take `$o, $p`. The entity derived from the variable is `chkchar` or `chkobj` internally.)

**General Checks:**
*   `rand (percent)`: True if a random number (1-100) is less than or equal to `percent`. E.g., `rand(50)`.
*   `economy (room_vnum_or_none) opr value`: Compares area economy for the specified room (or current room of `$i` if no vnum).
    *   Example: `if economy() > 100000` (checks economy of `$i`'s room)
    *   Example: `if economy(3001) < 50000` (checks economy of room 3001)
*   `mobinroom (mob_vnum) opr count`: Compares count of specified mob VNUM in `$i`'s current room. `count` defaults to 1 if omitted.
    *   Example: `if mobinroom(1234) >= 3`
*   `timeskilled (mob_vnum_or_char_var) opr value`: Compares how many times `mob_vnum` (or template of `char_var`) has been killed.
    *   Example: `if timeskilled($i) > 10` or `if timeskilled(1234) == 0`
*   `ovnumhere (obj_vnum) opr count`: Counts objects with `obj_vnum` in `$i`'s inventory OR in `$i`'s room. `count` defaults to 1.
*   `otypehere (obj_type_name_or_num) opr count`: Counts objects of `obj_type` (name or number) in `$i`'s inventory OR in `$i`'s room. `count` defaults to 1.
*   `ovnumroom (obj_vnum) opr count`: Counts objects with `obj_vnum` in `$i`'s room. `count` defaults to 1.
*   `otyperoom (obj_type_name_or_num) opr count`: Counts objects of `obj_type` in `$i`'s room. `count` defaults to 1.
*   `allvnumcarry (obj_vnum) opr count`: Checks if `$n` (actor) is carrying `count` (default 1) of `obj_vnum`.
    *   Example: `if allvnumcarry(500) == 1`
*   `ovnumcarry (obj_vnum) opr count`: Counts objects (VNUM) `$i` is carrying (not worn). `count` defaults to 1.
*   `otypecarry (obj_type_name_or_num) opr count`: Counts objects (type) `$i` is carrying (not worn). `count` defaults to 1.
*   `ovnumwear (obj_vnum) opr count`: Counts objects (VNUM) `$i` is wearing. `count` defaults to 1.
*   `otypewear (obj_type_name_or_num) opr count`: Counts objects (type) `$i` is wearing. `count` defaults to 1.
*   `ovnuminv (obj_vnum) opr count`: Counts objects (VNUM) in `$i`'s inventory (must not be worn). `count` defaults to 1.
*   `otypeinv (obj_type_name_or_num) opr count`: Counts objects (type) in `$i`'s inventory (must not be worn). `count` defaults to 1.

**Character-Specific Checks (`char_var` can be `$i, $n, $t, $r`):**
*   `ovnumchinv (char_var) == obj_vnum`: True if `char_var` carries any object with `obj_vnum`. Operator must be `==`. The `obj_vnum` comes from the `<value>` part of the ifcheck.
    *   Example: `if ovnumchinv($n) == 101` (Is `$n` carrying any object with VNUM 101?)
*   `ispc (char_var)`: True if `char_var` is a Player Character.
*   `isnpc (char_var)`: True if `char_var` is an NPC.
*   `isinvis (char_var)`: True if `char_var` is a PC and affected by `AFF_INVISIBLE`.
*   `ismobinvis (char_var)`: True if `char_var` is an NPC and has `ACT_MOBINVIS` flag.
*   `mobinvislevel (char_var) opr value`: Compares `char_var`'s mobinvis level (if NPC).
*   `class (char_var) opr value`: Compares `char_var->main_ability`. (Value is class number/ID).
*   `ismounted (char_var)`: True if `char_var` is mounted (`position == POS_MOUNTED`).
*   `iswanted (char_var)`: True if PC `char_var` is wanted by `$i` (mob with the prog, based on `vip_flags` and `wanted_flags`).
*   `isgood (char_var)`: True if `char_var` alignment is good.
*   `isneutral (char_var)`: True if `char_var` alignment is neutral.
*   `isevil (char_var)`: True if `char_var` alignment is evil.
*   `iskiller (char_var)`: True if PC `char_var` has `PLR_KILLER` flag.
*   `isfight (char_var)`: True if `char_var` is currently fighting (`who_fighting(char_var) != NULL`).
*   `isimmort (char_var)`: True if `char_var` is an Immortal (`get_trust(char_var) >= LEVEL_IMMORTAL`).
*   `ischarmed (char_var)`: True if `char_var` is affected by `AFF_CHARM`.
*   `isfollow (char_var)`: True if `char_var` is following someone and is in the same room as their master.
*   `isaffected (char_var) opr affect_name`: True if `char_var` has the specified affect flag (e.g., `blind`, `poison`). `<value>` is the affect name string.
    *   Example: `if isaffected($n) == blind`
*   `hitprcnt (char_var) opr value`: Compares `char_var`'s current hit points percentage (0-100).
    *   Example: `if hitprcnt($n) < 25`
*   `inroom (char_var) opr room_vnum`: Compares VNUM of the room `char_var` is in.
*   `wasinroom (char_var) opr room_vnum`: Compares VNUM of the room `char_var` was previously in (`was_in_room`).
*   `norecall (char_var)`: True if `char_var` is in a `ROOM_NO_RECALL` room.
*   `sex (char_var) opr sex_value`: Compares `char_var`'s sex (0=neutral, 1=male, 2=female).
*   `position (char_var) opr pos_value`: Compares `char_var`'s position (e.g., 0=dead, ..., 8=standing). See `constants.c` or similar for position numbers.
*   `doingquest (char_var) opr quest_vnum`: If PC, compares `char_var->pcdata->quest_number`.
*   `ishelled (char_var) opr release_time`: If PC, compares `char_var->pcdata->release_date` (Unix timestamp). `opr 0` typically means not helled.
*   `level (char_var) opr value`: Compares `get_trust(char_var)` (effective level for players, level for NPCs).
*   `combat_level (char_var) opr value`: Compares skill level in `COMBAT_ABILITY`. (And similar for `piloting_level`, `engineering_level`, `hunting_level`, `smuggling_level`, `diplomacy_level`, `leadership_level`, `force_level`, `slicer_level`).
*   `max_combat_level (char_var) opr value`: Compares max skill level in `COMBAT_ABILITY`. (And similar for `max_piloting_level`, etc.).
*   `goldamt (char_var) opr value`: Compares amount of gold `char_var` has.
*   `race (char_var) opr "race_name"`: Compares `char_var`'s race name (string).
*   `clan (char_var) opr "clan_name"`: Checks if `char_var` is a member of the specified clan/faction (string).
*   `council (char_var) opr "council_name"`: (Likely obsolete/custom or tied to specific faction logic) Always false in base code analysis.
*   `senator (char_var) opr "senate_name"`: (Likely obsolete/custom) Always false in base code analysis.
*   `clantype (char_var) opr value`: (Likely obsolete/custom) Always false in base code analysis.
*   `str (char_var) opr value`: Compares `char_var`'s current strength (`get_curr_str`).
*   `(and similar for wis, int, dex, con, cha, lck, frc)`: Compares current wisdom, intelligence, etc.
*   `iscarrying (char_var) == obj_vnum`: True if `char_var` is carrying an object with `obj_vnum`. Operator must be `==`. `<value>` is `obj_vnum`.
*   `iswearing (char_var) == obj_vnum`: True if `char_var` is wearing an object with `obj_vnum`. Operator must be `==`. `<value>` is `obj_vnum`.

**Object-Specific Checks (`obj_var` can be `$o, $p`):**
*   `objtype (obj_var) opr item_type_value`: Compares `obj_var->item_type` (numeric value). Refer to game constants for item type numbers (e.g., LIGHT, SCROLL).
*   `objval0 (obj_var) opr value`: Compares `obj_var->value[0]`.
*   `(and similar for objval1, objval2, objval3, objval4, objval5)`

**Generic Checks (context-dependent `var` can be `char_var` or `obj_var`):**
*   `number (var) opr vnum`:
    *   If `var` resolves to an NPC: compares `var->pIndexData->vnum`. (Special case: if `var` is `$i` (self NPC), it compares `$i->gold` instead of VNUM. This seems unusual and might be a specific codebase feature or misinterpretation; typically mob VNUM is desired). *Correction based on user notes: If cvar is $i and it's a mob, compares mob->gold.*
    *   If `var` resolves to an Object: compares `var->pIndexData->vnum`.
*   `name (var) opr "name_string"`: Compares `var->name` (keywords/namelist).

## 5. MPROGs (Mob Programs)

MPROGs are attached to mobs and define their reactions.
The mob itself is available as `$i` in the script.

### MPROG Triggers

| Trigger Name     | `arglist` Type & Meaning                                  | When it Fires                                                              | `$i` (Mob w/Prog) | `$n` (Actor)       | `$t` (Target Char) | `$o` (Object)      | `$p` (Target Obj)  |
| :--------------- | :-------------------------------------------------------- | :------------------------------------------------------------------------- | :---------------- | :----------------- | :----------------- | :----------------- | :----------------- |
| `ACT_PROG`       | Keyword list (space separated, or `p "<phrase>"`)         | Mob `$i` sees `ch` perform an action/speech `buf` matching keywords.       | `mob`             | `ch` (action doer) | `vo` (if char)     | `obj` (if any)     | `vo` (if obj)      |
| `BRIBE_PROG`     | Minimum gold amount                                       | `ch` gives `$i` at least `arglist` gold.                                   | `mob`             | `ch` (briber)      |                    | Created money obj  |                    |
| `DEATH_PROG`     | Percent chance (1-100), default 100                       | `$i` is killed by `killer`. Fires if `rand(arglist)` is true.               | `mob` (dying)     | `killer`           |                    |                    |                    |
| `ENTRY_PROG`     | Percent chance (1-100), default 100                       | `$i` "enters" (e.g., loaded/reset). Fires if `rand(arglist)` is true.      | `mob`             | (Usually NULL)     |                    |                    |                    |
| `FIGHT_PROG`     | Percent chance (1-100), default 100                       | `$i` starts fighting `ch`. Fires if `rand(arglist)` is true.               | `mob`             | `ch` (opponent)    |                    |                    |                    |
| `GIVE_PROG`      | Object name keywords or "all"                             | `ch` gives `obj` to `$i`, and `obj` name matches `arglist` or `arglist`="all". | `mob` (receiver)  | `ch` (giver)       |                    | `obj` given        |                    |
| `GREET_PROG`     | Percent chance (1-100), default 100                       | `ch` enters the same room as `$i`. Stops after one mob (per `ch`) greets. Fires if `rand(arglist)` is true. | `mob` (greeter)   | `ch` (entering)    |                    |                    |                    |
| `ALL_GREET_PROG` | Percent chance (1-100), default 100                       | `ch` enters the same room as `$i`. All mobs with prog may greet. Fires if `rand(arglist)` is true. | `mob` (greeter)   | `ch` (entering)    |                    |                    |                    |
| `HITPRCNT_PROG`  | HP Percentage (1-100)                                     | `$i`'s HP drops below `arglist` percent during combat *initiated by or involving* `ch`. | `mob` (damaged)   | `ch` (attacker/opponent) |                    |                    |                    |
| `RAND_PROG`      | Percent chance (1-100), default 100                       | Randomly, per pulse/tick, for `$i`. Fires if `rand(arglist)` is true.      | `mob`             | (Usually NULL)     |                    |                    |                    |
| `SPEECH_PROG`    | Keyword list (space separated, or `p "<phrase>"`)         | `actor` speaks `txt` in `$i`'s room, and `txt` matches keywords.           | `mob` (listener)  | `actor` (speaker)  |                    |                    |                    |
| `SCRIPT_PROG`    | Hour number (0-23) or empty                             | For pausable scripts. Triggers if `arglist` is empty, or mob's `mpscriptpos > 0`, or game hour matches `arglist`. | `mob`             | (Usually NULL)     |                    |                    |                    |
| `TIME_PROG`      | Hour number (0-23)                                        | When game hour matches `arglist`. Fires once when hour first matches.      | `mob`             | (Usually NULL)     |                    |                    |                    |
| `HOUR_PROG`      | Hour number (0-23)                                        | When game hour matches `arglist`. Fires repeatedly (every tick) while hour matches. | `mob`             | (Usually NULL)     |                    |                    |                    |

## 6. OPROGs (Object Programs)

OPROGs are attached to objects. A "supermob" acts on behalf of the object.
In scripts: `$i` is this supermob (representing the object), `$o` is the object itself that has the prog.

### OPROG Triggers

| Trigger Name   | `arglist` Type & Meaning                                  | When it Fires                                                                    | `$i` (Supermob) | `$n` (Actor)   | `$t` (Target Char) | `$o` (Obj w/Prog) | `$p` (Target Obj) |
| :------------- | :-------------------------------------------------------- | :------------------------------------------------------------------------------- | :-------------- | :------------- | :----------------- | :---------------- | :---------------- |
| `ACT_PROG`     | Keyword list                                              | Object `$o` (mobj) sees `ch` perform an action/speech `buf` matching keywords.   | `supermob(mobj)`| `ch` (action doer)| `vo` (if char)   | `mobj`            | `vo` (if obj)     |
| `DAMAGE_PROG`  | Percent chance, default 100                               | Object `$o` is damaged by `ch`. Fires if `rand(arglist)` is true.                | `supermob(obj)` | `ch` (damager, can be NULL) |          | `obj` (damaged)   |                   |
| `DROP_PROG`    | Percent chance, default 100                               | `ch` drops object `$o`. Fires if `rand(arglist)` is true.                        | `supermob(obj)` | `ch`           |                    | `obj` (dropped)   |                   |
| `EXA_PROG`     | Percent chance, default 100                               | `ch` examines object `$o`. Fires if `rand(arglist)` is true.                     | `supermob(obj)` | `ch`           |                    | `obj` (examined)  |                   |
| `GET_PROG`     | Percent chance, default 100                               | `ch` gets object `$o`. Fires if `rand(arglist)` is true.                         | `supermob(obj)` | `ch`           |                    | `obj` (gotten)    |                   |
| `GREET_PROG`   | Percent chance, default 100                               | `ch` enters room containing object `$o`. Fires if `rand(arglist)` is true.       | `supermob(obj)` | `ch` (entering)|                    | `obj` (greeting)  |                   |
| `PULL_PROG`    | Percent chance, default 100                               | `ch` pulls object `$o` (e.g., a lever). Fires if `rand(arglist)` is true.        | `supermob(obj)` | `ch`           |                    | `obj` (pulled)    |                   |
| `PUSH_PROG`    | Percent chance, default 100                               | `ch` pushes object `$o` (e.g., a button). Fires if `rand(arglist)` is true.      | `supermob(obj)` | `ch`           |                    | `obj` (pushed)    |                   |
| `RAND_PROG`    | Percent chance, default 100                               | Randomly, per pulse/tick, for object `$o`. Fires if `rand(arglist)` is true.     | `supermob(obj)` | (Usually NULL) |                    | `obj` (random)    |                   |
| `REMOVE_PROG`  | Percent chance, default 100                               | `ch` removes (stops wearing) object `$o`. Fires if `rand(arglist)` is true.      | `supermob(obj)` | `ch`           |                    | `obj` (removed)   |                   |
| `REPAIR_PROG`  | Percent chance, default 100                               | `ch` repairs object `$o`. Fires if `rand(arglist)` is true.                      | `supermob(obj)` | `ch`           |                    | `obj` (repaired)  |                   |
| `SAC_PROG`     | Percent chance, default 100                               | `ch` sacrifices object `$o`. Fires if `rand(arglist)` is true.                   | `supermob(obj)` | `ch`           |                    | `obj` (sacrificed)|                   |
| `SPEECH_PROG`  | Keyword list                                              | `ch` speaks `txt` in room with object `$o`, and `txt` matches.                   | `supermob(obj)` | `ch` (speaker) |                    | `obj` (hearing)   |                   |
| `USE_PROG`     | Percent chance, default 100                               | `ch` uses object `$o` (e.g., zap wand, quaff potion). Fires if `rand(arglist)` true. | `supermob(obj)` | `ch`           | `vict` (if char target from `vo`) | `obj` (used)    | `targ` (if obj target from `vo`) |
| `WEAR_PROG`    | Percent chance, default 100                               | `ch` wears object `$o`. Fires if `rand(arglist)` is true.                        | `supermob(obj)` | `ch`           |                    | `obj` (worn)      |                   |
| `ZAP_PROG`     | Percent chance, default 100                               | `ch` zaps object `$o` (wand/staff). Fires if `rand(arglist)` true. `vo` is target. | `supermob(obj)` | `ch`           | Target from `vo` (if char)  | `obj` (zapped)    | Target from `vo` (if obj) |
| `SCRIPT_PROG`  | Hour number or empty                                      | Same as MPROG SCRIPT_PROG, but for object `$o`. Uses `obj->mpscriptpos`.        | `supermob(obj)` | (Usually NULL) |                    | `obj` (scripting) |                   |

## 7. RPROGs (Room Programs)

RPROGs are attached to rooms. A "supermob" acts on behalf of the room.
In scripts: `$i` is this supermob (representing the room).

### RPROG Triggers

| Trigger Name   | `arglist` Type & Meaning                                  | When it Fires                                                                    | `$i` (Supermob)  | `$n` (Actor)    | `$t` (Target Char) | `$o` (Object)   | `$p` (Target Obj) |
| :------------- | :-------------------------------------------------------- | :------------------------------------------------------------------------------- | :--------------- | :-------------- | :----------------- | :-------------- | :---------------- |
| `ACT_PROG`     | Keyword list                                              | `ch` performs an action/speech `buf` matching keywords in room `$i`.             | `supermob(room)` | `ch` (action doer)| `vo` (if char)   | `obj` (if any)  | `vo` (if obj)     |
| `ENTER_PROG`   | Percent chance, default 100                               | `ch` enters room `$i`. Fires if `rand(arglist)` is true.                         | `supermob(room)` | `ch` (entering) |                    |                 |                   |
| `LEAVE_PROG`   | Percent chance, default 100                               | `ch` leaves room `$i`. Fires if `rand(arglist)` is true.                         | `supermob(room)` | `ch` (leaving)  |                    |                 |                   |
| `SLEEP_PROG`   | Percent chance, default 100                               | `ch` starts sleeping in room `$i`. Fires if `rand(arglist)` is true.             | `supermob(room)` | `ch` (sleeping) |                    |                 |                   |
| `REST_PROG`    | Percent chance, default 100                               | `ch` starts resting in room `$i`. Fires if `rand(arglist)` is true.              | `supermob(room)` | `ch` (resting)  |                    |                 |                   |
| `RFIGHT_PROG`  | Percent chance, default 100                               | A fight involving `ch` starts/occurs in room `$i`. Fires if `rand(arglist)` true. | `supermob(room)` | `ch` (fighter)  |                    |                 |                   |
| `RDEATH_PROG`  | Percent chance, default 100                               | `ch` (anyone) dies in room `$i`. Fires if `rand(arglist)` is true.               | `supermob(room)` | `ch` (victim)   | (NULL)             |                 |                   |
| `SPEECH_PROG`  | Keyword list                                              | `ch` speaks `txt` in room `$i`, and `txt` matches keywords.                      | `supermob(room)` | `ch` (speaker)  |                    |                 |                   |
| `RAND_PROG`    | Percent chance, default 100                               | Randomly, per pulse/tick, for room `$i` (often when `ch` is present/active). Fires if `rand(arglist)` is true. | `supermob(room)` | `ch` (nearby char, can be NULL)|                    |                 |                   |
| `SCRIPT_PROG`  | Hour number or empty                                      | Same as MPROG SCRIPT_PROG, but for room `$i`. Uses `room->mpscriptpos`.           | `supermob(room)` | (Usually NULL)  |                    |                 |                   |
| `TIME_PROG`    | Hour number (0-23)                                        | Game hour matches `arglist` for room `$i`. Fires once when hour first matches.   | `supermob(room)` | (Usually NULL)  |                    |                 |                   |
| `HOUR_PROG`    | Hour number (0-23)                                        | Game hour matches `arglist` for room `$i`. Fires repeatedly while hour matches.  | `supermob(room)` | (Usually NULL)  |                    |                 |                   |

## 8. Special Commands in Progs

Besides standard MUD commands, progs have a few special internal commands:
*   `mpsleep <duration_in_ticks>`: Pauses the current prog's execution for the specified number of game ticks. This is primarily used by `SCRIPT_PROG`s to create delays or resume later.
*   `break`: Immediately stops the execution of the current prog. No further lines in this prog will be processed for this trigger instance.

This guide should provide a solid foundation for your builders to create engaging and dynamic content using Mud Programs! Remember to test progs thoroughly. 