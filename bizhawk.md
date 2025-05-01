# items
- 022c475e last received item index, as halfword int
- 022c4760 zone unlocks sonic, as bits
- 022c4761 zone unlocks blaze, as bits
- 022c4762 progressive level unlocks sonic, as byte int
- 022c4763 progressive level unlocks blaze, as byte int
- 022C468F chaos emeralds, as bits
- 022c4765 sol emeralds, as bits
- 022c4767 Sidekick showing, as bits (first Tails, then Cream)
- bad power-ups TODO
- boost fills and drains TODO
- extra lives
  - 022C468E sonic extra lives, not in-level with Sonic
  - 022C46E6 blaze extra lives, not in_level with Blaze
  - 022c45a4 in-level with according character
- 022c4580 current in-level time

# locations
- 022c4690 sonic level scores, 4 bytes each for act 1 2 and boss, all zones in order
- 022c46e8 blaze level scores, 4 bytes each for act 1 2 and boss, all zones in order
- s rank same as act clear, but only if big number
- special stages 022c4764 bits
- f-zone, point w, extra zone cleared flags at 022c4766 (in that order)

# other
- ~~022c476f Save data initialization flag for client, has to stay 0x6b => 0x6b gets set when save data setup is done~~ DEPRECATED, REPLACED BY SONIC STORYPROG BEING != 0x0
- 022c4768 Deathlink, as bits (first receiving, then sending)
- 022c45b4 might help with determining if in-level (3), transitioning scene (0), or anywhere else (1)
- 022c4560 selected character, Sonic is 0, Blaze is 1
