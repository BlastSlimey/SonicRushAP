# new data addresses for AP
- 022c475e-022c476f is AP storage
- 022c475e last received item index, as halfword int
- 022c4760 zone unlocks sonic, as bits
- 022c4761 zone unlocks blaze, as bits
- 022c4762 progressive level unlocks sonic, as byte int
- 022c4763 progressive level unlocks blaze, as byte int
- 022c4764 completed special stages, as bits
- 022c4765 sol emeralds, as bits
- 022c4766 f-zone, point w, extra zone cleared flag, as bits (in that order)
- 022c4767 Sidekick showing, as bits (first Tails, then Cream)
- 022c4768 Deathlink, as bits (first receiving, then sending)
- ~~022c476f Save data initialization flag for client, has to stay 0x6b~~ DEPRECATED, REPLACED BY SONIC STORYPROG

# vanilla addresses of data, for reference
- 022c4560 selected character, 1 is blaze, 0 is sonic
- 022C468F chaos emeralds, as bits
- 020928CA visual emeralds
- 022c4588 visual emeralds buffer
- 022c4688 overall story progression
- 022c4689 5th bit = 1 is extra zone cleared ergo corruption free overworld
- 022C468C sonic story progression (NEVER SET TO 0, ANYTHING ABOVE 0 SIGNALS THE SAVE FILE BEING READY TO RECEIVE ITEMS)
- 022C46E4 blaze story progression
- add sonic extra lives to 022C468E
- add blaze extra lives to 022C46E6
- 022c4690 sonic level scores, 4 bytes each for act 1 2 and boss, all zones in order
- 022c46e8 blaze level scores, 4 bytes each for act 1 2 and boss, all zones in order

# code changes

### receiving and sending deathlink
- function at 0234419c in overlay0000 in ARM
  - repeats continuously in-level
  - checks and executes death if damaged and no rings
  - => also execute death if deathlink received
  - => new function FUN_RECEIVE_DEATHLINK at 020857a0 in arm9 in ARM
  - => has to use r4 as param
```
--- override at 02344878, pretty much just adds another function call
mov r0,r4
blle FUN_02349038
bl FUN_RECEIVE_DEATHLINK
--- FUN_RECEIVE_DEATHLINK(param1, param2, param3, param4, param5)
if 022c4768 & 1 != 0 # if receive flag set
  FUN_02349038(param5) # function that kills player
  022c4768 &= 0xfe # unsetting receive flag
--- and now in assembly
stmdb sp!,{lr}
ldr r1,[PTR_DEATHLINK]
ldrb r1,[r1,#0x0]
and r1,r1,#0x1
cmp r1,#0x0
beq LAB_RET_FUN_DEATHLINK
mov r0,r4
bl FUN_02349038
ldr r1,[PTR_DEATHLINK]
ldrb lr,[r1,#0x0]
and lr,lr,#0xfe
strb lr,[r1,#0x0]
	LAB_RET_FUN_DEATHLINK
ldmia sp!,{lr}
bx lr
```
- function at 02349038 in overlay0000 in ARM
  - called when hitting damaging object without rings
  - calls a function which calls another function that conditionally subtracts one from the extra lives buffer
  - => set sending deathlink flag if receiving flag not set (because this death might be caused by an incoming deathlink)
  - => new function FUN_SEND_DEATHLINK at 020857dc in arm9 in ARM
```
--- override at 023490dc, original function call must happen instantly
bl FUN_SEND_DEATHLINK
--- FUN_SEND_DEATHLINK(param1, param2)
FUN_02346aa8(param1, param2)
if 022c4768 & 1 == 0
  022c4768 |= 2
--- and now in assembly
stmdb sp!,{lr}
bl FUN_02346aa8
ldr r0,[PTR_DEATHLINK]
ldrb r1,[r0,#0x0]
and r1,r1,#0x1
cmp r1,#0x0
ldrbeq r1,[r0,#0x0]
orreq r1,r1,#0x2
strbeq r1,[r0,#0x0]
ldmia sp!,{lr}
bx lr
	PTR_DEATHLINK 0x022c4768
```

### ap storage protection
- function at 02058474 in arm9 in ARM
- validates and corrects save data (storyprog, extra lives, chaos emeralds, scores, 0xFF part and after?)
- caps 0xFF part at 0x8CA0 (36000) if not 0xFFFF
- => skip loop that would cap ap storage
```
r0 set to 0x0000FFFF, then used, afterwards set => no problem
r1 set to r12, then used, afterwards set => no problem
r2 set to 0x8CA0, then used, afterwards set => no problem
r3 repeatedly set to *[r6,r5], then used, afterwards set => no problem
r4 conditionally and repeatedly set to r1 = r12 = 0x0, afterwards the same => no problem from context (return value, only gets set to 0 conditionally)
r5 repeatedly set to r12 << 1, then used, afterwards set => no problem
r6 set to r7 + 0x16, then used, afterwards set => no problem
r7 set to 0x022c473c, then used, afterwards used => need to keep set instruction
r12 set to 0, then used and incremented, afterwards set => no problem
--- override at 02058570
b 0x020585ac
```

### sol emeralds display
- on updating visual data depending on char, including shown life count and
- function address: 0204ec40, ARM instructions, located in arm9
- => replace visual sol emeralds calculation with loading AP items
- => replace at address 0204ecb8
- => r0-r4, r12, lr available
```
ldr r4,[DAT_AP_PTR]
ldrb lr,[r4,#0x5]
strh lr,[r0,#0x28]
b LAB_0204ecec
=> DAT_AP_PTR 022c4760
```

### show sidekick
- on deciding whether to show the sidekick
- => change to account for ap items/flags
```
--- function 02053b04 in arm9, in ARM:
--- remain
if (*piVar3 == 0)
--- override inside if-block
=> r2,r3 available
if sidekick showing & (1 << selected char) == 0
  return
```

### transitioning from result screen
- DONE
- on transitioning from act/boss result screen
- deciding how and to where to transition
- => complete replace to account for completed act/boss, entered zone, and ap items
- => if not enough progressive level select, go to act2/boss/overworld
- => if enough progressive level select, go always to overworld
```
--- function 0204e744 in arm9 in ARM:
=> use r0,r1,r2 = current zone,r3 = current act,r12 = progressive level select
=> all transitioning returns afterwards
calculate current zone from current act
if current act > 2
  nothing?
else if current act == 2
  set storyprog to full
  if current zone == 7
    f zone cutscene # FUN_0204d684(0x02054f10,3);
  else if zone == 8
    extra zone cutscene # FUN_02054ef4();
  else
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
```

### setting flags for final bosses
- DONE
- on defeating f zone, point w, extra zone
- setting storyprog to last bzw extra zone flag
- => set boss flags in ap storage instead
```
--- function 022d2db8 in overlay0001:
--- replace
func_0x02059bdc(0x1a); -> FUN_BOSS_CLEARED(0)
func_0x02059bcc(0x1c); -> FUN_BOSS_CLEARED(1)
func_0x02059b2c(); -> FUN_BOSS_CLEARED(2)
--- new function FUN_BOSS_CLEARED(r4) at 02085610 in arm9 in THUMB:
ldr r0,[DAT_BOSS_FLAGS]
mov r1,#0x1
lsl r1,r4
ldrb r4,[r0,#0x0]
orr r4,r1
strb r4,[r0,#0x0]
bx lr
DAT_BOSS_FLAGS 022c4766
```

### early save data setup
- on entering main menu, checking if time attack has been notified to the player
- function 022d8e58 in overlay0001 in THUMB
- => save data setup
- => replace original function with call to new function, returning what the original function intended
- => new FUN_SAVE_SETUP at 020856c0 in arm9 in THUMB
```
--- old function
push {lr}
bl FUN_SAVE_SETUP
pop {pc}
--- new function
blaze_storyprog = 0x1c            # full overworld for blaze
if sonic_storyprog == 0           # save data not set up yet, this value never gets set to 0 again
  for ptr = 0x022c475e; ptr < 0x022c4770; ptr++
    *ptr = 0                      # set ap storage to 0
sonic_storyprog = 0x1a            # full overworld for sonic
overall_storyprog |= 0x1000       # check extra zone flag for corruption-less overworld
*022c4770 = 0x6b                  # notify the client that save datat was set up
return overall_storyprog & 0x100  # returns whether time attack has been notified to the player
--- assembly
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
b LAB_INSERT_END
mov r0,#0x0 # LAB_FALSE
b LAB_INSERT_END
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
DAT_CHAR_SEL 022C468c
DAT_AP_STORAGE 022c475e
push {r0} # LAB_INSERT_END
ldr r1,[DAT_AP_PTR]
mov r0,#0x6b
strb r0,[r1,#0xf]
pop {r0}
bx lr
=> DAT_AP_PTR 022c4760
```

### Prevent arm9 from clearing custom code
- DONE
- on arm9 initialization, causing the great whiteout
- => make an exception by splitting for loop in 2
```
--- arm9 entering code, address 0200088c
--- delete
for...
--- replace
FUN_WHITEOUT()
r0 = 0
--- new function FUN_WHITEOUT at 02085670, ARM instructions
r1 = 0x0207cfe0
r2 = 0x02085600
r0 = 0
compare r1,r2
strcc r0,[r1],#0x4
bcc 2 instructions back
r1 = 0x02085d00
r2 = 0x022c5c80
compare r1,r2
strcc r0,[r1],#0x4
bcc 2 instructions back
return
```

### Change extra zone conditions
- DONE
- on entering character select, checking for extra zone availability
- => check for both emeralds instead of story progression
- => complete override
```
--- function 02356408 in overlay0000:
if 022C468F == 0x7f and 022c4765 == 0x7f
	return 1
return 0
```

### Always enable character select
- REMOVED FROM PATCH, WAS DEPRECATED
- on entering char select, before reading sonic's storyprog
- => set sonic storyprog to 0x1a, always giving char select
- => needs to be it's own function
```
--- function 022d6eac in overlay0001:
--- delete
*DAT_022d6f34 = 0;
--- replace
FUN_ALW_CHAR_SEL()
--- new function FUN_ALW_CHAR_SEL at 02085018 in arm9 in THUMB
022C468C = 0x1a
022C4560 = 0;
```

### Entering zone
- DONE
- on selecting zone, deciding what to do based on selected character, selected zone, and what zones have level select unlocked
- iVar3 holds the selected zone as an index (0...7)
- => change level select availability checking to checking for zone unlocked and enough progressive level selects
- => add do-nothing if zone not unlocked
```
--- function 022e612c in overlay0001:
--- remain
  ...
  if (-1 < iVar3) {
--- override
=> r0 free, r1 iVar3 sel zone (NOT LOCATION), r2 piVar2 (SEL CHAR PTR???) but can be exchanged for value and then freed, r3 free but return pointer
    ap_pointer = 022c4760
    if *piVar2 == 1
      ap_pointer++
    if *ap_pointer & (1 << (byte)iVar3) != 0
      if *(ap_pointer+2) > (byte)iVar3 && (byte)iVar3 < 7
        goto level_select
      goto act_1
    goto ret
level_select:
    FUN_SET_STORYPROG_X_FULL(ap_pointer+2)
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
--- new function FUN_SET_STORYPROG_X_FULL(x) in arm9 at 02085750 in THUMB:
=> r0 free, r1 selected zone, r2 = prog lvl sel, r3 ptr to unlocked zones bits
switch x
  case 1: 022C468C = 0x1, 022C46E4 = 0x2, return
  case 2: 022C468C = 0x4, 022C46E4 = 0x6, return
  case 3: 022C468C = 0x8, 022C46E4 = 0x9, return
  case 4: 022C468C = 0xc, 022C46E4 = 0xd, return
  case 5: 022C468C = 0x10, 022C46E4 = 0x11, return
  case 6: 022C468C = 0x14, 022C46E4 = 0x15, return
  default: 022C468C = 0x17, 022C46E4 = 0x19, return
--- and now in assembly
mov	r0,#0x1
mov	r1,#0x2
cmp	r2,#0x1
beq	LAB_STORE_PROG
mov	r0,#0x4
mov	r1,#0x6
cmp	r2,#0x2
beq	LAB_STORE_PROG
mov	r0,#0x8
mov	r1,#0x9
cmp	r2,#0x3
beq	LAB_STORE_PROG
mov	r0,#0xc
mov	r1,#0xd
cmp	r2,#0x4
beq	LAB_STORE_PROG
mov	r0,#0x10
mov	r1,#0x11
cmp	r2,#0x5
beq	LAB_STORE_PROG
mov	r0,#0x14
mov	r1,#0x15
cmp	r2,#0x6
beq	LAB_STORE_PROG
mov	r0,#0x17
mov	r1,#0x19
ldr	r3,[DAT_SONIC_PROG] # LAB_STORE_PROG
strb	r0,[r3,#0x0]
ldr	r3,[DAT_BLAZE_PROG]
strb	r1,[r3,#0x0]
bx	lr
DAT_BLAZE_PROG	022C46E4
DAT_SONIC_PROG	022C468C
```

### Redirect extra zone flag setting
- DEPRECATED, CAN BE REMOVED FROM PATCH
- on clearing extra zone, setting flag at 022c4689 to cleared
- => change to instead set flag in AP storage
- => complete override
```
--- function 02059b2c in arm9:
022c4766 = 0x1
```

### Redirect special stage flag setting
- DONE
- on clearing special stage, adding the collected chaos emerald to the emerald buffer
- => add checked special stage to flags in AP storage instead of emerald buffer
```
--- function 0230c104 in overlay0001:
--- remain
  ...
  if (*(int *)(iVar7 + 0x24) != 0) {
--- override
    022c4764 = 022c4764 | (ushort)(1 << (*(uint *)(iVar7 + 4) & 0xff));
--- remain
  }
  ...
```




