# new data addresses for AP
022c475e-022c476f is AP storage
022c475e last received item index, as halfword int
022c4760 zone unlocks sonic, as bits
022c4761 zone unlocks blaze, as bits
022c4762 progressive level unlocks sonic, as byte int
022c4763 progressive level unlocks blaze, as byte int
022c4764 completed special stages, as bits
022c4765 sol emeralds, as bits
022c4766 f-zone, point w, extra zone cleared flag, as bits (in that order)
022c4767 Sidekick showing, as bits (first Tails, then Cream)
022c476f Save data initialization flag for client, has to stay 0

# vanilla addresses of data, for reference
022c4560 selected character, 1 is blaze, 0 is sonic
022C468F chaos emeralds, as bits
020928CA visual emeralds
022c4588 visual emeralds buffer
022c4688 overall story progression
022c4689 5th bit = 1 is extra zone cleared ergo corruption free overworld
022C468C sonic story progression (NEVER SET TO 0)
022C46E4 blaze story progression
add sonic extra lives to 022C468E
add blaze extra lives to 022C46E6
022c4690 sonic level scores, 4 bytes each for act 1 2 and boss, all zones in order
022c46e8 blaze level scores, 4 bytes each for act 1 2 and boss, all zones in order

# code changes
### show sidekick
on deciding whether to show the sidekick
=> change to account for ap items/flags
--- function 02053b04 in arm9, in ARM:
--- remain
if (*piVar3 == 0)
--- override inside if-block
=> r2,r3 available
if sidekick showing & (1 << selected char) == 0
  return

### transitioning from result screen
DONE
on transitioning from act/boss result screen
deciding how and to where to transition
=> complete replace to account for completed act/boss, entered zone, and ap items
=> if not enough progressive level select, go to act2/boss/overworld
=> if enough progressive level select, go always to overworld
--- function 0204e744 in arm9 in ARM:
=> use r0,r1,r2 = current zone,r3 = current act,r12 = progressive level select
if current act > 2
  nothing?
else if current act == 2
  set storyprog to full
  back to overworld from boss # needs FUN_0205500c
if current zone < progressive lvl sel (sonic/blaze)
  set storyprog to full
  back to overworld from act # needs FUN_02054ff0
else
  if current act == 0
    to next act # needs LAB_0204e4b8
  else
    to boss # needs LAB_0204e498
--- init
stmdb sp!,{lr}
sub sp,sp,#0x4
--- return
add        sp,sp,#0x4
ldmia      sp!,{lr}
bx         lr

### setting flags for final bosses
DONE
on defeating f zone, point w, extra zone
setting storyprog to last bzw extra zone flag
=> set boss flags in ap storage instead
--- function 022d2db8 in overlay0001:
--- replace
func_0x02059bdc(0x1a); -> FUN_BOSS_CLEARED(0)
func_0x02059bcc(0x1c); -> FUN_BOSS_CLEARED(1)
func_0x02059b2c(); -> FUN_BOSS_CLEARED(2)
--- new function FUN_BOSS_CLEARED(r4) at 0231f164 in THUMB:
ldr r0,[DAT_BOSS_FLAGS]
mov r1,#0x1
lsl r1,r4
ldrb r4,[r0,#0x0]
orr r4,r1
strb r4,[r0,#0x0]
DAT_BOSS_FLAGS 022c4766

### early save data setup
DONE
on entering main menu, checking if time trials have been notified
=> save data setup
--- function 022d8e58 in overlay0001:
--- delete everything
--- replace
push {lr}
bl FUN_SAVE_SETUP
pop {pc}
--- FUN_SAVE_SETUP at 0231f100 in THUMB
=> only r0 and r1 available
b LAB_INSERT
ldr r0,[DAT_SAV] # LAB_INSERT_BACK
mov r1,#0x1a
strb r1,[r0,#0xc]
ldr r1,[r0,#0x8] # DO NOT ldrb
mov r2,#0x10
lsl r2,r2,#0x8
orr r1,r2
str r1,[r0,#0x8]
mov r0,r2
pop {r2}
lsr r0,r0,#0x4
and r1,r0
cmp r1,#0x0
beq LAB_FALSE
mov r0,#0x1
bx lr
mov r0,#0x0 # LAB_FALSE
bx lr
DAT_SAV 022C4680
push {r2} # LAB_INSERT
ldr r0,[DAT_SAV_BLAZE]
mov r1,#0x1c
strb r1,[r0,#0x0]
ldr r0,[DAT_SAV_SONIC]
ldrb r1,[r0,#0x0]
cmp r1,#0x0
bne LAB_INSERT_BACK
ldr r0,[DAT_AP_STORAGE]
mov r2,r0
add r2,#0x12
mov r1,#0x0
strb r1,[r0,#0x0] # LAB_FOR_LOOP
add r0,#0x1
cmp r0,r2
bne LAB_FOR_LOOP
b LAB_INSERT_BACK
DAT_SAV_BLAZE 022C46E4
DAT_SAV_SONIC 022C4680
DAT_AP_STORAGE 022c475e

### Prevent arm9 from clearing custom code
DONE
on arm9 initialization, causing the great whiteout
=> make an exception by splitting for loop in 2
--- arm9 entering code, address 0200088c
--- delete
for...
--- replace
FUN_WHITEOUT()
r0 = 0
--- new function FUN_WHITEOUT at 02085060, ARM instructions
r1 = 0x0207cfe0
r2 = 0x02085000
r0 = 0
compare r1,r2
strcc r0,[r1],#0x4
bcc 2 instructions back
r1 = 0x02085500
r2 = 0x022c5c80
compare r1,r2
strcc r0,[r1],#0x4
bcc 2 instructions back
return

### Change extra zone conditions
DONE
on entering character select, checking for extra zone availability
=> check for both emeralds instead of story progression
=> complete override
--- function 02356408 in overlay0000:
if 022C468F == 0x7f and 022c4765 == 0x7f
	return 1
return 0

### Always enable character select
DEPRECATED, CAN BE REMOVED FROM PATCH
on entering char select, before reading sonic's storyprog
=> set sonic storyprog to 0x1a, always giving char select
=> needs to be it's own function
--- function 022d6eac in overlay0001:
--- delete
*DAT_022d6f34 = 0;
--- replace
FUN_ALW_CHAR_SEL()
--- new function FUN_ALW_CHAR_SEL at 02315f04
022C468C = 0x1a
022C4560 = 0;

### Setup overworld
DONE
on selecting character, setting some visual data
also on returning from in-level to overworld
ergo on loading overworld
=> set both storyprog to last and extra zone cleared
=> set correct display of sol emeralds if blaze selected
=> add to the beginning
--- function 02063f50 in arm9:
  022C468C = 0x1a
  022C46E4 = 0x1c
  022c4688 = 022c4688 | 0x1000 <- DEPRECATED, CAN BE REMOVED FROM PATCH
  if (022c4560 == 1)
    022c4588 = 022c4765

### Change zone entering
DONE
on selecting zone, deciding what to do based on selected character, selected zone, and what zones have level select unlocked
iVar3 holds the selected zone as an index (0...7)
=> change level select availability checking to checking for zone unlocked and enough progressive level selects
=> add do-nothing if zone not unlocked
--- function 022e612c in overlay0001:
--- remain
  ...
  if (-1 < iVar3) {
--- override
=> r0 free, r1 iVar3, r2 piVar2 but can be exchanged for value and then freed, r3 free but return pointer
    ap_pointer = 022c4760
    if *piVar2 == 1
      ap_pointer++
    if *ap_pointer & (1 << (byte)iVar3) != 0
      if *(ap_pointer+2) > (byte)iVar3 && (byte)iVar3 < 7
        goto level_select
      goto act_1
    goto ret
level_select:
    set storyprogs to 1...*(ap_pointer+2) full
    FUN_022e5b6c();
    func_0x02052104(0xd);
    goto ret
act_1:
    FUN_022e5ba0();
    func_0x02052104(0xb);
ret:
    add sp,#0x4
    pop r3
    return
--- remain
  }
  ...
--- new function set storyprogs to 1...x full in overlay0001 at 0231f0b0:
switch x
  case 1: 022C468C = 0x1, 022C46E4 = 0x2, return
  case 2: 022C468C = 0x4, 022C46E4 = 0x6, return
  case 3: 022C468C = 0x8, 022C46E4 = 0x9, return
  case 4: 022C468C = 0xc, 022C46E4 = 0xd, return
  case 5: 022C468C = 0x10, 022C46E4 = 0x11, return
  case 6: 022C468C = 0x14, 022C46E4 = 0x15, return
  default: 022C468C = 0x17, 022C46E4 = 0x19, return

### Redirect extra zone flag setting
DEPRECATED, CAN BE REMOVED FROM PATCH
on clearing extra zone, setting flag at 022c4689 to cleared
=> change to instead set flag in AP storage
=> complete override
--- function 02059b2c in arm9:
022c4766 = 0x1

### Redirect special stage flag setting
DONE
on clearing special stage, adding the collected chaos emerald to the emerald buffer
=> add checked special stage to flags in AP storage instead of emerald buffer
--- function 0230c104 in overlay0001:
--- remain
  ...
  if (*(int *)(iVar7 + 0x24) != 0) {
--- override
    022c4764 = 022c4764 | (ushort)(1 << (*(uint *)(iVar7 + 4) & 0xff));
--- remain
  }
  ...




