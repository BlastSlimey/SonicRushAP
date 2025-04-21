search for function that controls where to go after clearing an act/boss
possible functions
02008500 lots of calculations, many callings => probably no
0201b060 lots of calculations, many callings => probably no
0202a1a0 lots of calculations, many callings => probably no
---
0204e744 (later part, looks promising) check for called functions
020451c8 defeating boss - not called by 0204e744, boss to overworld, boss to level select 
0204d684 act to next act, act to level select, boss to overworld, boss to level select
0204e88c RETURN WITHIN FUN_0204e744
02054378 going into act/boss, defeating boss, boss to overworld, boss to level select
02054ef4
---
0204ec40 repeating function setting char related things => no, called too often
02053b04 FUN_SHOW_SIDEKICK => no, already switching to overworld
02058474 limiting some save data => probably no

search for early function to manipulate save data
FUN_022cf3b0
FUN_022cf324
FUN_022c8b14
FUN_022d8e58 YES
FUN_022c8b38
FUN_022d49a0 YES

TODO (DEPRECATED)
find hook to insert custom data upon entering character select
find hook to insert custom data upon selecting character
find hook to insert custom data upon entering zone
find how to after extra zone (0% corruption)
find function that sets score for each act => also send check
find if the scores of f-zone, point w, and extra zone are saved anywhere

022C5C80 overlay 0000 and 0001 insert
overlay 0000 is inside gameplay and character select, starts with value 10402de9, ends at 0231f03f
overlay 0001 is menu and overworld, starts with 011c0020, ends at 02369abf
maybe 022c5c00 as AP storage? would provide 128 bytes

0207D374 frame counter?
020927f4 ZONE X font color (only blaze?)
020927F8 visual extra life count, has to be written ziffernweise in 2 bytes
02092868 visual corruption count, has to be written ziffernweise in 2 bytes
020928CA visual emeralds
022b4574 SEEMS TO BE IMPORTANT
022c4560 selected character, 1 is blaze, 0 is sonic
022C4680 SAVE DATA BEGIN?
022C4688 byte 1 bit 1 difficulty, bit 2 time limit, byte 2 bit 5 extra zone cleared
022C468C sonic story progression
022C468E sonic extra life count, doesn't visually update automatically
022C468F sonic chaos emeralds
022c4690 sonic level scores, 4 bytes each for act 1 2 and boss, all zones in order
022C46E4 blaze story progression
022C46E6 blaze extra life count, doesn't visually update automatically
022c46e8 blaze level scores, 4 bytes each for act 1 2 and boss, all zones in order

search for setting visual emeralds:
chaos emerald readings at 0204EC78, 022E5A96, 0201B074, 020588CC, 0205852C
possible scenario:
0204EC78: chaos emerald bits to 022c4588
FUN_02063f50 reads 022c4588 and stores it somewhere unknown
calling FUN_020369ac directly after
visual emeralds set somewhere around FUN_020369ac

possible functions upon entering character select:
address 022c600e NO
FUN_0205247c NO
FUN_022d6eac YES
FUN_022e8a24 NO
FUN_02300378 NO

FUN_0204ec40 
	called after character select
	reads current storyprog 
	writes to-be-shown emrald bits into 022c4588
FUN_022e2640 
	called after pressing A on zone
	contains a lot of nested ifs
	contains a lot of comparing against storyprog 
FUN_022e252c 
	called after pressing A on zone
	contains a lot of comparing against storyprog 
FUN_020533e0
	called after pressing A on zone
	contains 2 switch blocks
	checks if storyprog < 3
FUN_02059bdc
	function that stores param1 in sonic's storyprog
FUN_02059bcc
	function that stores param1 in blaze's storyprog
FUN_022e2640
	translates sonics storyprog into something else
	0 => -1
	1, 2, 3 => 0
	4, 5, 6, 7 => 1
	8, 9, a, b => 2
	c, d, e, f => 3
	10, 11, 12, 13 => 4
	14, 15, 16 => 5
	17, 18, 19, 1a... => 6
	=> checks whether a chosen zone is fully cleared according to storyprog





sonic story progression
	00 immediate leaf storm
	01 leaf storm levels
	02 first ow cutscene
	03 first char select, 1 full, 2 start
	04 1 2 full
	05 cutscene after zone 2
	06 ???
	07 1 2 full, 3 start
	08 1 2 3 full
	09 cutscene after zone 3
	0a ???
	0b 1 2 3 full, 4 start
	0c 1 2 3 4 full
	0d roadblock unlock after zone 4
	0e 1 2 3 4 full, roadblock
	0f 1 2 3 4 full, 5 start
	10 1 2 3 4 5 full
	11 island with blaze unlock after zone 5
	12 1 2 3 4 5 full, island with blaze
	13 1 2 3 4 5 full, 6 start
	14 1 2 3 4 5 6 full
	15 cutscene after zone 6
	16 1 2 3 4 5 6 full, 7 start
	17 1 2 3 4 5 6 7 full
	18 cutscene after zone 7
	19 all unlock
	1a all unlock, after f-zone
blaze story progression
	00 starting cutscene
	01 zone 1 start
	02 zone 1 full
	03 cutscene after zone 1
	04 ???
	05 zone 2 start
	06 zone 2 full
	07 cutscene after 2, zone 3 start
	08 zone 3 start
	09 zone 3 full
	0a cutscene after 3, knuckles cutscene available
	0b knuckles cutscene available
	0c zone 4 start
	0d zone 4 full
	0e amy cutscene available, amy appears on map
	0f amy cutscene available
	10 zone 5 start
	11 zone 5 full
	12 cutscene after 5
	13 sonic cutscene available
	14 zone 6 start
	15 zone 6 full
	16 second amy cutscene available (amy appears on map)
	17 second amy cutscene available
	18 zone 7 start
	19 zone 7 full
	1a cutscene after 7
	1b all unlocked
	1c after point w















cheat codes:
	infinite boost energy
620907b4 00000000
5207b8a4 ffffffff
12090b10 00000640
d2000000 00000000
	low time?
620907b4 00000000
5207b8a4 ffffffff
022c4580 00000020
d2000000 00000000

action replay/codebreaker/game genie:
02xxxxxx xxxxxxxx write 4 bytes
22xxxxxx xxxxxxxx write 1 byte
12xxxxxx xxxxxxxx write 2 bytes
04xxxxxx xxxxxxxx continue if less (signed)
D4xxxxxx 0141xxxx continue if less (signed)
62xxxxxx xxxxxxxx continue if not equal
52xxxxxx xxxxxxxx continue if equal
D2000000 00000000 end any repeats and conditions, set offset and storage registers to zero










plan:

unlock_zone_x_char unlocks entering a zone from the world map, player has to start from act 1
progressive_level_select_char makes entering an unlocked zone instead go into level select, showing all progessively unlocked level selects
level is reachable if unlock_zone_x_char OR (count(proglvlselchar) >= x AND any(unlock_zone_1...count_char))

on entering character select:
- set sonic storyprog to blaze unlocked
- check chaos and sol emeralds for access to extra zone

on selecting character
- set storyprog to after extra zone     
	=> this is to always have the character select screen AND the full map AND normal ow music
- if char is blaze:
	- set visual sol emerald bits to collected 

on selecting zone
- if unlock_zone_x:
	- set storyprog to 1...y full with y = count progressive_level_select
		=> if enough progressive_level_select and selected zone unlocked, then level select will open
		=> else only start from act 1
- else:
	- do nothing, stay in overworld

on returning to overworld from in-level
=> the same as on selecting char

on beating extra zone
=> possible functions: FUN_02059b2c
- set ap extra zone flag to true, signaling the client to goal




