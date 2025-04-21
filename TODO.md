# TODO


# DONE

- change pointers on act transitions, setting prt_4 afterwards might have been wrong
- act with enough progressive level select still going into full level select
- error when defeating zone 1 boss, arm9 caught in endless loop
  - => modified code for act transitions, should be tested and observed
  - => still happening, looking for storyprog readings
  - => before reaching on loading ow function: 0201B080, 020584E0, 0202A1B4, 0204EC64, 02010024, 022E2A4C, 02053468, 02053808
  - => after reaching function jumping to bad memory
  - => either set full storyprog earlier, or some function has bad stack/link register handling
  - => functions called in chain: 0204e744 act transition, 02053b04 show sidekick, 02063f50 loading overworld
- add sidekicks as ap items
- add time trials and audiotest notification flags on early setup
- check all functions for false jumps to labels of other functions
- check for returning from in-level to overworld
- in-level returning to level select, potentially opening more levels than allowed
  - => make transition go
    - when level select for this zone unlocked, go back to overworld
    - when no level select and from act, go to next act
    - when no level select and from boss, go to overworld
- add flags for defeating f zone/point w and make defeating them setting those flags instead of storyprog
- check for relocated AP storage being initialized as 0xFF and what that does to calculations
  - => set everything to 0 in early setup if sonic storyprog is 0 (this only happens on first start of the save file because it is instantly set to 0x1a and then never set to 0 again)
- set blaze storyprog on early save setup
- relocate AP storage to save data
- still going into first level after first launch => when 022c4689 is read
- overworld still glitching even though extra zone bit is set
- crash on zones 2 and 7
- add counters for halving extra lives sonic/blaze
- look for 0 progressive level select
- watch...
  - 022c5c00-022C5C7f (AP storage) 
    - => is that a good spot for AP storage?
    - => Yes, but there is also space in save data
  - 022C5C80 overlays insert 
    - => are there any other scenarios of overlays switching?
    - => Seems like no
  - function 0204ec40 in arm9, on selecting character, setting some data based on selected character
    - => when is this called? maybe even when returning from ingame to overworld?
    - => yes for setting visual sol emeralds but too late for setting storyprog






