# Super Mario Bros. 3 (NES) Game Genie codes

These codes were made from PRG1, the US release that the disassembly
rebuilds byte for byte. They have not been checked against PRG0 or any other
revision. Every code is the 8-letter form with a compare byte. All effects
come from the disassembly. They have not been tested in play unless an entry
says so.

## Time moves only while a button is held

**Codes:** `YPSEVTNT` + `AASENTGA` + `ZGVANTPP` (use all three)
**Raw form:** `8E5E:17:E7` + `8E5F:00:04` + `8E67:42:11`
(address:new-byte:compare-byte, bank 30)
**Recorded:** 2026-10-16 21:45:33

Tags: `demo-ready` `play-start` `curated`

**Mechanism:** Each frame, `Level_MainLoop` (`prg030.asm`) checks for a
Start press, then checks `Level_PauseFlag`. If the game is paused, it draws
the "PAUSE" sprites and jumps back to `Level_MainLoop`, which skips the rest
of the frame:

```
	LDA SndCur_Pause
	BNE PRG030_8E79	 ; Can't unpause game while pause sound is playing

	LDA <Pad_Input	
	AND #PAD_START
	BEQ PRG030_8E79	 ; If Player is NOT pressing START, jump to PRG030_8E79

	; (toggles Level_PauseFlag and queues the pause sound)

PRG030_8E79:
	LDA Level_PauseFlag
	BEQ PRG030_8EAD	 	; If not paused, jump to PRG030_8EAD

	; (draws the PAUSE sprites)

	; Nothing else to do while paused; loop!
	JMP Level_MainLoop
```

The first two codes change `LDA SndCur_Pause` (`$04E7`) to `LDA $0017`,
which is `Pad_Holding`. The third code changes the target of the
`BEQ PRG030_8E79` after the Start test to the `JMP Level_MainLoop` at
`$8EAA`.

- If any button is held, the `BNE` goes to `PRG030_8E79`. `Level_PauseFlag`
  is 0, so the frame runs as normal.
- If no button is held, the code reads `Pad_Input`. `Read_Joypads`
  (`prg031.asm`) makes `Pad_Input` from the new presses `AND` the buttons
  held, so it is also 0. The `BEQ` goes straight to `JMP Level_MainLoop`.
  This skips the rest of the frame, and also the PAUSE sprites and
  `Sprite_RAM_Clear`, so the screen stays as it was.

The Start toggle can now run only if Start is newly pressed while no button
is held. That can't happen, so `Level_PauseFlag` stays 0 and the pause sound
is never queued.

The game doesn't stay frozen for good, because `Read_Joypads` runs in the
NMI (`IntNMI`, `prg031.asm`), not in the skipped part of the loop.

**Evidence:** `Level_MainLoop`, `Read_Joypads` and `IntNMI`. The source,
`smb3_original.nes` and the built ROM have
`AD E7 04 D0 17 A5 18 29 10 F0 11` at `$8E5D`, and `4C D8 8D`
(`JMP Level_MainLoop`) at `$8EAA`. `Pad_Holding` is `$0017`. Bank 30 is
fixed at `$8000`, so no other bank can match.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Use all three codes. Without the third code, an idle frame goes on to
  `PRG030_8E79` and runs as normal, so time never stops. The third code
  alone freezes the game. Start still toggles the pause, and a press that
  unpauses runs one frame.
- Every button counts, Start too, so the game can't be paused.
- Don't turn the codes on while the game is paused. `Level_PauseFlag`
  stays 1 and nothing can clear it.
- The NMI still increments `Counter_1` and calls `Randomize`. Tile
  animations (`?` blocks, coins and so on) run before the pause check and
  use `Counter_1`, so they keep moving while the game is frozen. The random
  numbers keep changing too.
- The level timer is updated in the skipped part of the loop, so it stops
  while you're idle.
- The sound engine runs in the NMI, so music and sound effects keep playing.
- Only levels use `Level_MainLoop`. The world map, the title screen and the
  bonus games run at normal speed.
- In a 2-player game, `Pad_Holding` belongs to the player whose turn it is.

## Jump again in mid-air

**Code:** `AAKZEKII`
**Raw form:** `AC40:00:5D`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:25:48

Tags: `demo-ready` `play-start` `grab-bag-collection` `kids-collection` `curated`

**Mechanism:** `Player_JumpFlyFlutter` (`prg008.asm`) starts a jump when
A is newly pressed, but only if the Player is on the ground or the short
"walked off a ledge" grace time (`Player_AllowAirJump`) is still running:

```
	LDA Player_AllowAirJump
	BNE PRG008_AC41	 ; If Player_AllowAirJump <> 0, jump to PRG008_AC41

	LDA <Player_InAir
	BNE PRG008_AC9E	 ; If Player is mid air, jump to PRG008_AC9E

PRG008_AC41:

	; Play jump sound
```

The code changes the `BNE PRG008_AC9E` offset from `$5D` to `$00`, so
the branch lands on the next instruction (`PRG008_AC41`) either way. Every
new A press now plays the jump sound and sets `Player_YVel` to a full jump
velocity, in the air or not.

**Evidence:** `Player_JumpFlyFlutter`. The source, `smb3_original.nes`
and the built ROM have `A5 D8 D0 5D` (`LDA <Player_InAir` /
`BNE PRG008_AC9E`) at `$AC3D`. No other bank that loads at `$A000` has
`$5D` at `$AC40`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Tested in 1-1 (small Mario): a press while falling with `Player_YVel`
  `$05` set it to `$C9`, a full jump. Not tested with the suits.
- Works in every land movement mode: small, big, Fire/Hammer, Raccoon,
  Frog, Tanooki and Kuribo's Shoe. Swimming has its own code and isn't
  changed.
- With the Raccoon or Tanooki suit, an A press in the air jumps instead of
  only wagging the tail. The tail wag still starts on the same press, so
  slow falls and flight still work but get the jump on top.
- The air jump can start a backflip with a Starman, as a ground jump does.

## Speed up and slow down as fast as Frog Mario

**Codes:** `OXOXELSX` + `GEOXOUST` (use both)
**Raw form:** `AB98:A9:A5` + `AB99:04:ED`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:27:40

**Mechanism:** `Player_GroundHControl` (`prg008.asm`) picks a row of
`Player_XAccelMain` and `Player_XAccelPseudoFrac` by the current suit:

```
PRG008_AB98:
	LDA <Player_Suit
	ASL A
	ASL A
	ASL A
	TAY		 ; Y = Player_Suit << 3
```

Each row has a stopping rate, a normal rate and a skid rate. For every suit
but the Frog, the normal rate is `0` plus a fraction of `$E0`. The
fraction adds 1 only when it carries with `Counter_Wiggly`, so the X speed
goes up by about 0.9 per frame. The Frog row has a normal rate of exactly 2
and a stopping rate of exactly -1.

The codes change `LDA <Player_Suit` (`A5 ED`) to `LDA #$04` (`A9 04`),
which is `PLAYERSUIT_FROG`. Every suit now uses the Frog row: about 2.3x
the normal acceleration. Stopping is a little faster too (-1 per frame, in
place of about -0.6 for small Mario and about -0.9 for the others). The skid
rate is the same for every suit.

**Evidence:** `Player_GroundHControl`, `Player_XAccelMain` and
`Player_XAccelPseudoFrac`. The source, `smb3_original.nes` and the built
ROM have `A5 ED 0A 0A 0A A8` at `$AB98`. `Player_Suit` is `$ED`. No
other bank that loads at `$A000` has the same bytes at `$AB98` or
`$AB99`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Use both codes. The first alone makes it `LDA #$ED`: a row past the
  end of the tables.
- Top speed doesn't change. Pair with "Run much faster" for more speed.
- Only the table row changes. The suit's other abilities, and how the Frog
  suit hops, are as before.
- The same rates apply in the air.
- Ice and underwater pick their rows elsewhere and don't change.

## All ground is slippery like ice

**Code:** `YXNLVSZU`
**Raw form:** `BDF6:2F:3A`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:28:38

Tags: `demo-ready` `play-start`

**Mechanism:** Each frame, `prg008.asm` sets `Player_Slippery` to 0, then
checks the ground under the Player, but only in ice levels. Snow ground
makes it 1, and ice blocks, frozen coins and frozen Munchers make it 2.
`Player_GroundHControl` uses it to pick the slippery rows of the
acceleration tables. Outside ice levels, the `BNE PRG008_BE31` skips all
of it. On ice blocks, the code sets it to 2:

```
	LDA #$00
	STA Player_Slippery	 ; Player_Slippery = 0 (default ground not slippery)

	LDA Level_TilesetIdx
	CMP #11
	BNE PRG008_BE31	 ; If not in an ice level, jump to PRG008_BE31
	...
PRG008_BE26:
	LDA #$02	 
	STA Player_Slippery	 ; Player_Slippery = 2 (ground is REALLY slippery!)

	JMP PRG008_BE31	 ; Jump to PRG008_BE31
```

The code changes the `BNE` offset from `$3A` to `$2F`, so it jumps to
`PRG008_BE26`. Every level that is not an ice level now acts like the
Player is on an ice block. Ice levels work as before.

**Evidence:** The slippery logic and `Player_GroundHControl`
(`prg008.asm`). The source, `smb3_original.nes` and the built ROM have
`C9 0B D0 3A` at `$BDF3` and `A9 02 8D 89 05` at `$BE26`
(`$BDF7 + $2F = $BE26`). No other bank that loads at `$A000` has
`$3A` at `$BDF6`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The jump skips the "Player is in air" check, so the slippery rows are
  also used in the air. They have a weaker skid rate, so turning around in
  a jump is slower than usual.
- In ice levels, only the usual snow and ice tiles are slippery.

## Low gravity: higher jumps that are slow to come down

**Code:** `PEUZKGIA`
**Raw form:** `ACB4:01:05`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:29:21

Tags: `demo-ready` `play-start` `grab-bag-collection`

**Mechanism:** While the Player is in the air, `Player_JumpFlyFlutter`
(`prg008.asm`) adds gravity to `Player_YVel` each frame:

```
PRG008_ACB3:

	; Player is mid air...

	LDY #$05	 ; Y = 5

	LDA <Player_YVel
	CMP #-$20
	BGS PRG008_ACC8	 ; If Player's Y velocity >= -$20, jump to PRG008_ACC8
	...
	LDA <Pad_Holding
	BPL PRG008_ACC8	 ; If Player is NOT pressing 'A', jump to PRG008_ACC8

	LDY #$01	 ; Y = 1
	...
PRG008_ACCD:
	TYA	
	ADD <Player_YVel
	STA <Player_YVel ; Player_YVel += Y
```

Gravity is 1 only while A is held and the Player is still rising fast.
Otherwise it is 5. The code changes the 5 to 1, so gravity is always 1.

A jump starts at about `-$38` (`PLAYER_JUMP`). Holding A, the jump now
rises about 98 pixels in place of about 72. A short tap of A jumps as high
as a held jump. Falling speeds up 5x more slowly. It takes about a second
to reach the top fall speed (`FALLRATE_MAX`, `$40`), which doesn't
change.

**Evidence:** `Player_JumpFlyFlutter`. The source, `smb3_original.nes`
and the built ROM have `A0 05 A5 CF C9 E0` at `$ACB3`. No other bank
that loads at `$A000` has `$05` at `$ACB4`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Every fall uses it: walking off a ledge, bouncing off an enemy,
  Kuribo's Shoe and so on. Bounces off enemies and springboards go much
  higher.
- A held A no longer gives a higher jump than a tap.
- Swimming has its own code and doesn't change.
- The Raccoon and Tanooki tail wag still limits the fall speed to
  `PLAYER_TAILWAG_YVEL` (`$10`).

## Climb in open air by pressing Up

**Codes:** `AANXKALP` + `AEKZUATA` (use both)
**Raw form:** `A87C:00:13` + `A8C3:00:06`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:29:58

Tags: `demo-ready` `play-start` `curated`

**Mechanism:** The vine climbing logic in `prg008.asm` starts climbing
only when the tile at the Player is a vine:

```
	LDA <Temp_Var1
	CMP #TILE1_VINE
	BNE PRG008_A890	 ; If tile is not the vine, jump to PRG008_A890

	LDA Player_IsClimbing
	BNE PRG008_A898	 ; If climbing flag is set, jump to PRG008_A898

	LDA <Pad_Holding
	AND #(PAD_UP | PAD_DOWN)
	BEQ PRG008_A890	 ; If Player is not pressing up or down, jump to PRG008_A890
```

While climbing, Up moves the Player up only if the tile above is also a
vine:

```
	JSR Player_GetTileAndSlope	; Get tile

	CMP #TILE1_VINE
	BNE PRG008_A8CA	 ; If tile is NOT another vine, jump to PRG008_A8CA

	LDY #-$10
	STY <Player_InAir ; Flag Player as "in air"
```

Both codes change a `BNE` offset to `$00`, so each branch lands on the
next instruction either way. The first code lets the climb start on any
tile. The second lets the Player keep climbing up into open air.

**Evidence:** The vine climbing logic (`PRG008_A86C` on). The source,
`smb3_original.nes` and the built ROM have `C9 85 D0 13` at `$A879`
and `C9 85 D0 06` at `$A8C0`. `TILE1_VINE` is `$85`. No other bank
that loads at `$A000` has `$13` at `$A87C` or `$06` at `$A8C3`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Up starts a climb on the ground. Up or Down starts one in the air,
  including in a jump.
- Once climbing, the Player stays in the air without falling. Let go of
  Up and Down and the Player hangs in place. Hold Down to climb down to
  the ground, where letting go ends the climb.
- Left and Right move the Player while climbing, as on a vine.
- Not when in water, holding something or in Kuribo's Shoe, as before.
- Up on a door or a pipe may climb in place of entering. That code runs
  before this, so it probably still wins.
- Not known whether the climb can pass through solid blocks.

## Climb up and down twice as fast

**Codes:** `EVKZSAEY` (up) + `AXXZKAAP` (down)
**Raw form:** `A8C5:E0:F0` + `A8A4:20:10`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:30:36

**Mechanism:** While the Player is climbing, the vine climbing logic in
`prg008.asm` sets `Player_YVel` from the pad. Down is `$10` and Up is
`-$10`:

```
	LDY #$10 	; Y = $10 (will be Y velocity down if Player is pressing down)
	...
	CMP #TILE1_VINE
	BNE PRG008_A8CA	 ; If tile is NOT another vine, jump to PRG008_A8CA

	LDY #-$10
	STY <Player_InAir ; Flag Player as "in air"

PRG008_A8C8:
	STY <Player_YVel  ; Set Player's Y Velocity
```

The codes change these to `-$20` (`$E0`) and `$20`, which is 2 pixels
per frame in place of 1.

**Evidence:** The vine climbing logic (`PRG008_A898` on). The source,
`smb3_original.nes` and the built ROM have `A0 10` at `$A8A3` and
`A0 F0 84 D8` at `$A8C4`. No other bank that loads at `$A000` has the
same bytes at `$A8A4` or `$A8C5`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Either code works alone, for one direction.
- The up value is also stored in `Player_InAir`. Any nonzero value means
  "in air", so this should not matter.
- Sideways climbing is not changed. The right speed (`LDY #$10` at
  `$A8CA`) has the same byte at `$A8CB` in bank 24, where it is part
  of the title screen's RAM-clear loop (`STA Title_MLAccelCnt,X`). A Game
  Genie would patch both, so there is no code for it.
- Works with "Climb in open air by pressing Up".

## Flight time never runs out

**Code:** `AEKXTGPA`
**Raw form:** `A4CE:00:01`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:32:24

**Mechanism:** A jump at full power sets `Player_FlyTime` to `$80`.
`Player_PowerUpdate` (`prg008.asm`) counts it down every other frame.
When it reaches 0, the P-meter is cleared and the flight ends:

```
	LDA <Counter_1
	AND #$01
	BEQ PRG008_A4D5	 ; Every other tick, jump to PRG008_A4D5

	DEY			 
	STY Player_FlyTime ; Player_FlyTime--

PRG008_A4D5:
	TYA		 ; Y (Player_FlyTime) -> A 
	BNE Sound_FullPowerRing	 ; If Player_FlyTime <> 0, jump to Sound_FullPowerRing

	STY Player_Power ; Otherwise, clear Player_Power
```

The code changes `AND #$01` to `AND #$00`. The result is always 0, so
the `BEQ` always skips the decrement and `Player_FlyTime` never goes
down.

**Evidence:** `Player_PowerUpdate`. The source, `smb3_original.nes` and
the built ROM have `A5 15 29 01 F0 04 88 8C 6E 05` at `$A4CB`.
`Player_FlyTime` is `$056E`. No other bank that loads at `$A000` has
`$01` at `$A4CE`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- You still need a full P-meter for the first flight. After that, the
  flight time stays at `$80` and the P-meter stays full with its ringing
  sound, even after landing, so every later jump can fly. It works like
  the P-Wing, except that `Player_FlyTime` is not `$FF`.
- Landing without a flying suit still clears `Player_FlyTime` (`STA
  Player_FlyTime` at `PRG008_AC9E`). A full-power jump without the
  Raccoon or Tanooki suit gets the fast-jump pose until you land.

## Slow fall and fly without the Raccoon or Tanooki suit

**Code:** `AEVZVKAA`
**Raw form:** `ACE6:00:08`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:32:56

Tags: `demo-ready` `play-start` `curated`

**Mechanism:** In the air, `Player_JumpFlyFlutter` (`prg008.asm`)
starts a tail wag when A is pressed, but only if the suit has the flight
ability:

```
PRG008_ACD9:
	LDA Player_Kuribo
	BNE PRG008_ACEF	 ; If Player is wearing Kuribo's shoe, jump to PRG008_ACEF

	LDX <Player_Suit

	LDA PowerUp_Ability,X	; Get "ability" flags for this power up
	AND #$01	 
	BEQ PRG008_ACEF	 	; If this power up does not have flight, jump to PRG008_ACEF

	LDY <Temp_Var1		; Y = $80 if Player was pressing 'A' when this all began
	BEQ PRG008_ACEF	 	; And if he wasn't, jump to PRG008_ACEF

	LDA #$10	 
	STA <Player_WagCount 	; Otherwise, Player_WagCount = $10
```

While `Player_WagCount` is set, the code after `PRG008_ACEF` limits the
fall speed to `PLAYER_TAILWAG_YVEL` (`$10`). If `Player_FlyTime` is
set, it makes the Player rise at `PLAYER_FLY_YVEL` instead.

The code changes the `BEQ` offset from `$08` to `$00`, so the ability
check is skipped. Every suit now slows its fall when you tap A in the air.

**Evidence:** `Player_JumpFlyFlutter`. The source, `smb3_original.nes`
and the built ROM have `BD E0 C3 29 01 F0 08` at `$ACE0`. No other
bank that loads at `$A000` has `$08` at `$ACE6`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Every suit can also fly. A jump at full power sets `Player_FlyTime` for
  any suit, and the wag code then lifts the Player. Without a flying suit,
  `Player_FlyTime` is cleared on landing as usual.
- Only the Raccoon and Tanooki suits show the tail wag frames and play the
  wag sound. Other suits keep their jump pose, with no sound.
- Not in Kuribo's Shoe, which is checked first.

## Hop around as if in Kuribo's Shoe

**Code:** `AAKZNPGA`
**Raw form:** `A947:00:04`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:34:31

Tags: `demo-ready` `play-start`

**Mechanism:** The Player's movement code is picked from a table of
pointers by suit (`prg008.asm`, "MOVEMENT LOGIC PER POWER-UP / SUIT").
Kuribo's Shoe has its own entry, 14, which is used when `Player_Kuribo`
is set:

```
	LDA Player_Kuribo
	BEQ PRG008_A94C	 ; If Player is not wearing Kuribo's shoe, jump to PRG008_A94C

	; If in Kuribo's shoe...

	LDA #14		 ; A = 14 (Kuribo's shoe code pointer)
	BNE PRG008_A956	 ; Jump (technically always) to PRG008_A956
```

The code changes the `BEQ` offset from `$04` to `$00`, so the shoe
entry is always used. `Move_Kuribo` makes the Player hop along when Left
or Right is held, and sets `Player_Frame` from `Player_KuriboFrame`, so
the Player is drawn in the shoe.

`Player_Kuribo` itself stays 0. A Game Genie can't set RAM, and the only
code that sets the flag is an `INC` (in `Player_Update`). Making that run
every frame would count the flag past `$FF` back to 0.

**Evidence:** The movement dispatch and `Move_Kuribo`. The source,
`smb3_original.nes` and the built ROM have `AD 77 05 F0 04 A9 0E` at
`$A943`. `Player_Kuribo` is `$0577`. No other bank that loads at
`$A000` has `$04` at `$A947`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only the movement and the look change. Everything else that checks
  `Player_Kuribo` acts as if there is no shoe. You can't walk on Munchers
  or stomp spiny enemies, a hit costs a suit as usual, and the shoe never
  falls off.
- The shoe is drawn with the Player's own palette, so it may not be green.
- Used in water too, in place of swimming, so you can only hop
  underwater.
- The suit's own abilities (fireballs, tail, flight) are not used, since
  their movement code doesn't run.

## Run much faster

**Codes:** `AGNXOUAX` + `ASEZXUAU` + `AVXUNYAG` (use all three)
**Raw form:** `AB79:48:28` + `AB82:58:38` + `BFAF:60:40`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:35:55

**Mechanism:** The first two codes change the top speeds in
`Player_GroundHControl` (`prg008.asm`): B running goes from `$28` to
`$48`, and full P-meter speed from `$38` to `$58`.
That is 4.5 and 5.5 pixels per frame, about 1.6x the normal speeds.

Those speeds are past the hard cap in `Player_ApplyXVelocity`:

```
Player_ApplyXVelocity:
	LDX #$00	; X = 0
	LDY #PLAYER_MAXSPEED	; Y = PLAYER_MAXSPEED
	...
PRG008_BFAC:
	STA <Temp_Var16		; Store absolute value Player_XVel -> Temp_Var166
	CMP #PLAYER_MAXSPEED
	BLS Player_ApplyVelocity ; If we haven't hit the PLAYER_MAXSPEED yet, apply it!
	STY <Player_XVel	 ; Otherwise, cap at max speed!
```

The third code changes the `CMP #PLAYER_MAXSPEED` to `CMP #$60`, so the
cap only applies at `$60` and up. The cap still sets the speed to
`$40` (`LDY #PLAYER_MAXSPEED`), but normal running never reaches `$60`,
because acceleration stops at the top speed.

**Evidence:** `Player_GroundHControl` and `Player_ApplyXVelocity`. The
source, `smb3_original.nes` and the built ROM have `A0 28` at `$AB78`,
`A0 38` at `$AB81` and `C9 40 30 02 84 BD` at `$BFAE`. No other bank
that loads at `$A000` has the same bytes at `$AB79`, `$AB82` or
`$BFAF`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Use all three. Without the third code, the cap cuts both speeds back to
  `$40`.
- The `LDY #PLAYER_MAXSPEED` values aren't changed. The right one has the
  same byte (`$40` at `$BFA0`) in bank 21. If something pushes the
  Player to `$60` (a long slide down a slope, for example), the speed
  drops back to `$40` for that frame.
- Speed-indexed tables run past their ends: `Player_WalkAnimTickMax`
  (by `XVel >> 3`) reads `Player_RootJumpVel` and `Player_SpeedJumpInc`
  for speeds `$48` and up, and `Player_SpeedJumpInc` (by `XVel >> 4`)
  reads the unclaimed table after it for speeds `$40` and up. The walk
  animation should just run fast, and running jumps get a slightly smaller
  boost.
- At 5.5 pixels per frame, the Player may pass through thin things such as
  enemies or the edges of blocks more easily.

## All solid blocks are jump-through for the player

**Codes:** `XVEXZNTY` + `AEKLTSPA` + `XVKUYSTY` (use all three)
**Raw form:** `A78A:E2:7E` + `B5C6:00:09` + `B5CF:E2:7E`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:37:51

**Mechanism:** Each tileset has two sets of solid-tile limits in
`Tile_AttrTable`, one per tile quadrant (`$00`, `$40`, `$80`, `$C0`).
A tile at or above the limit at `Tile_AttrTable+0..3` is solid from
above (floor). A tile at or above the limit at `Tile_AttrTable+4..7` is
also solid from the side and from below (wall and ceiling). The Player's
wall and ceiling checks are in `prg008.asm`:

```
	; Checks for solid tile at Player's head
	LDA <Temp_Var1	 
	CMP Tile_AttrTable+4,Y	; Wall/ceiling-solid tile quadrant limits begin at Tile_AttrTable+4
	BLT PRG008_A7AD	 ; If tile index is less than value in Tile_AttrTable (not solid for wall/ceiling), jump to PRG008_A7AD
	...
Level_CheckGndLR_TileGTAttr:

	LDX Level_Tile_Quad+1,Y	; Get this particular "quad" (0-3) index
	LDA Level_Tile_GndR,Y		; Check the tile here
	CMP Tile_AttrTable+4,X
	BGE PRG008_B5D0			; If the tile is >= the attr value, jump to PRG008_B5D0 (NOTE: Carry set when true)

	LDX Level_Tile_Quad,Y		; Get this particular "quad" (0-3) index
	LDA Level_Tile_GndL,Y		; Check the tile here
	CMP Tile_AttrTable+4,X		; Set carry if tile is >= the attr value
```

`Level_CheckGndLR_TileGTAttr` is used for walls (twice) and for the
ceiling when jumping. The first code is the "stuck in a block" check.

The first and third codes change the operand `Tile_AttrTable+4`
(`$7E98`) to `$E298`, in fixed bank 31, which holds `FF FF FF FF`. No
tile below `$FF` counts as wall or ceiling solid. The right-tile compare
can't be changed the same way (see Caveats), so the second code sets the
`BGE` offset to `$00`. The routine always falls through to the left-tile
compare, which decides the result alone. Floors use `Tile_AttrTable+0..3`
and are not changed, so the Player still lands on top of blocks.

**Evidence:** `Player_Control` and `Level_CheckGndLR_TileGTAttr`.
The source, `smb3_original.nes` and the built ROM have `D9 98 7E` at
`$A788` and `BE 09 06 B9 04 06 DD 98 7E B0 09 BE 08 06 B9 03 06 DD 98 7E 60`
at `$B5BC`. Bank 31 has `FF FF FF FF` at `$E298`. No other bank that
loads at `$A000` has the same bytes at `$A78A`, `$B5C6` or `$B5CF`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Use all three. The Player walks and jumps through walls and ceilings,
  and lands on top of anything solid.
- Changing the right-tile compare at `$B5C4` the same way was rejected:
  bank 28 (music data) has `$7E` there.
- Without a ceiling hit, the Player can't bump ? blocks or bricks from
  below.
- Tile `$FF` still counts as solid, because `$FF` is not below `$FF`.
- Only the Player is changed. Enemies and objects still treat blocks as
  walls.
- The Player can get inside blocks, and may land on a floor inside them or
  fall out of the bottom of the level.

## All ground is a conveyor belt

**Codes:** `AASUSIIP` + `AAVLSSZA` + `XGVLVSXX` (use all three)
**Raw form:** `BD5D:00:15` + `BD65:00:0A` + `BD66:CA:AA`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:39:04

Tags: `demo-ready` `play-start` `grab-bag-collection`

**Mechanism:** The conveyor check in `Player_DoSpecialTiles`
(`prg008.asm`) runs when the Player is on the ground and no P-Switch is
active:

```
	LDX #$01	 ; X = 1 (check one tile by foot, then check the other!)

PRG008_BD59:
	LDA ConveyorEnable,Y
	BEQ PRG008_BD73	 ; If value is zero, jump to PRG008_BD73

	; Otherwise...
	SUB Level_Tile_GndL,X	 ; Subtract value we just retrieved 
	CMP #$02	 
	BGE PRG008_BD70	 ; If tile is >= 2, this is not a conveyor tile, jump to PRG008_BD70

	; We're on a conveyor tile!!

	TAX		 	; Transfer conveyor tile index -> 'X'
	LDA ConveyorSlide,X	; Get appropriate conveyor slide amount
	STA Player_SlideRate	; Set slide factor for conveyor
```

`ConveyorSlide` is `.byte 16, -16`. `Player_SlideRate` is added to
`Player_XVel` for one frame of movement, so the Player is pushed 1 pixel
per frame.

The first code sets the `BEQ` offset to `$00`, so tilesets without
conveyors don't skip the check. The second code does the same to the
`BGE`, so every tile counts as a conveyor. The third code replaces the
`TAX` (`X` would be a meaningless value here) with `DEX`. That makes `X`
0, so the push is `16`, to the right.

**Evidence:** `Player_DoSpecialTiles`. The source, `smb3_original.nes`
and the built ROM have `A2 01 B9 C4 C3 F0 15 38 FD 03 06 C9 02 B0 0A AA BD
4A BC` at `$BD57`. No other bank that loads at `$A000` has the same
bytes at `$BD5D`, `$BD65` or `$BD66`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Use all three codes.
- The push applies on all ground. It does not apply in the air, during a
  P-Switch, or while the Player is a statue, tail attacking or doing a
  flip.
- On slopes, a non-zero slide rate changes how `Player_SlideRate` is
  checked at `PRG008_A72B` (sliding by ducking). This was not looked at
  closely.
- Only the Player is pushed. Enemies and items are not.

## Every power-up block gives a Super Mushroom

**Codes:** `AONLIYEY` + `KVSLINXN` (use both)
**Optional, for Star blocks too:** `AAEUOAEA` + `IAEUVAGA`
**Raw form:** `B7F5:10:F0` + `B7D5:EC:FA` + optional `B809:00:80` + `B80E:05:04`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:41:46

**Mechanism:** When the Player bumps a block, its kind from `LATP_PowerUps`
(`prg008.asm`) picks a handler from `LATP_JumpTable`. Each handler returns an index into
`Bouncer_PUp` (`prg001.asm`) in `Y`:

```
LATP_JumpTable:
	.word LATP_None		; 0 = None
	.word LATP_Flower	; 1 = Mushroom/Flower
	.word LATP_Leaf		; 2 = Mushroom/Leaf
	.word LATP_Star		; 3 = Star
	...
LATP_Flower:
	LDA #$00
	STA PUp_StarManFlash	 ; PUp_StarManFlash = 0 (don't activate star man flash)

	LDY #$05	 ; Y = 5 (spawn a mushroom) (index into PRG001 Bouncer_PUp)

	LDA <Player_Suit
	BEQ PRG008_B7F9	 ; If Player is small, jump to PRG008_B7F9

	LDY #$02	 ; Y = 2 (spawn a fire flower) (index into PRG001 Bouncer_PUp)

PRG008_B7F9:
	RTS		 ; Return
```

`LATP_Leaf` is the same with `LDY #$03` (leaf), and `LATP_Star` sets
`PUp_StarManFlash` to `$80` and returns `Y = 4`.

The first code changes the `BEQ` in `LATP_Flower` to `BPL`. `Player_Suit`
is never negative, so the branch is always taken and `LDY #$02` is
skipped: the block always gives a mushroom (`Y = 5`). The second code
points table entry 2 (Mushroom/Leaf) at `LATP_Flower` (`$B7FA` to
`$B7EC`), so leaf blocks give the same thing.

The optional codes change `LATP_Star` to set `PUp_StarManFlash` to `0`
and return `Y = 5`, so Star blocks give the same item.

**Evidence:** `LATP_JumpTable` and the handlers after it. The source,
`smb3_original.nes` and the built ROM have `E9 B7 EC B7 FA B7 08 B8` at
`$B7D1`, `A9 00 8D 86 05 A0 05 A5 ED F0 02 A0 02 60` at `$B7EC`
(`LATP_Flower`)
and `A9 80 8D 86 05 A0 04 60` at `$B808` (`LATP_Star`). No other bank that
loads at `$A000` has the same bytes at the patched addresses.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- A big Player gets no upgrade from the mushroom.
- Only blocks the Player bumps go through this code. Blocks hit by a tail
  swipe or a kicked shell were not checked and may still give their
  normal item.
- Star blocks are not changed unless the optional codes are used.
- 1-up, vine, coin and P-Switch blocks are not changed.

## Every power-up block gives a Fire Flower

**Codes:** `AUNLIYEY` + `KVSLINXN` (use both)
**Optional, for Star blocks too:** `AAEUOAEA` + `ZAEUVAGA`
**Raw form:** `B7F5:30:F0` + `B7D5:EC:FA` + optional `B809:00:80` + `B80E:02:04`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:41:47

**Mechanism:** When the Player bumps a block, its kind from `LATP_PowerUps`
(`prg008.asm`) picks a handler from `LATP_JumpTable`. Each handler returns an index into
`Bouncer_PUp` (`prg001.asm`) in `Y`:

```
LATP_JumpTable:
	.word LATP_None		; 0 = None
	.word LATP_Flower	; 1 = Mushroom/Flower
	.word LATP_Leaf		; 2 = Mushroom/Leaf
	.word LATP_Star		; 3 = Star
	...
LATP_Flower:
	LDA #$00
	STA PUp_StarManFlash	 ; PUp_StarManFlash = 0 (don't activate star man flash)

	LDY #$05	 ; Y = 5 (spawn a mushroom) (index into PRG001 Bouncer_PUp)

	LDA <Player_Suit
	BEQ PRG008_B7F9	 ; If Player is small, jump to PRG008_B7F9

	LDY #$02	 ; Y = 2 (spawn a fire flower) (index into PRG001 Bouncer_PUp)

PRG008_B7F9:
	RTS		 ; Return
```

`LATP_Leaf` is the same with `LDY #$03` (leaf), and `LATP_Star` sets
`PUp_StarManFlash` to `$80` and returns `Y = 4`.

The first code changes the `BEQ` in `LATP_Flower` to `BMI`. `Player_Suit`
is never negative, so the branch is never taken and `LDY #$02` always
runs: the block gives a Fire Flower even to small Mario. The second
code points table entry 2 (Mushroom/Leaf) at `LATP_Flower` (`$B7FA` to
`$B7EC`), so leaf blocks give the same thing.

The optional codes change `LATP_Star` to set `PUp_StarManFlash` to `0`
and return `Y = 2`, so Star blocks give the same item.

**Evidence:** `LATP_JumpTable` and the handlers after it. The source,
`smb3_original.nes` and the built ROM have `E9 B7 EC B7 FA B7 08 B8` at
`$B7D1`, `A9 00 8D 86 05 A0 05 A5 ED F0 02 A0 02 60` at `$B7EC`
(`LATP_Flower`)
and `A9 80 8D 86 05 A0 04 60` at `$B808` (`LATP_Star`). No other bank that
loads at `$A000` has the same bytes at the patched addresses.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only blocks the Player bumps go through this code. Blocks hit by a tail
  swipe or a kicked shell were not checked and may still give their
  normal item.
- Star blocks are not changed unless the optional codes are used.
- 1-up, vine, coin and P-Switch blocks are not changed.

## Every power-up block gives a Super Leaf

**Codes:** `AONLIYEY` + `LENLZYIA` + `KVSLINXN` (use all three)
**Optional, for Star blocks too:** `AAEUOAEA` + `LAEUVAGA`
**Raw form:** `B7F5:10:F0` + `B7F2:03:05` + `B7D5:EC:FA` + optional `B809:00:80` + `B80E:03:04`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:41:47

Tags: `kids-collection`

**Mechanism:** When the Player bumps a block, its kind from `LATP_PowerUps`
(`prg008.asm`) picks a handler from `LATP_JumpTable`. Each handler returns an index into
`Bouncer_PUp` (`prg001.asm`) in `Y`:

```
LATP_JumpTable:
	.word LATP_None		; 0 = None
	.word LATP_Flower	; 1 = Mushroom/Flower
	.word LATP_Leaf		; 2 = Mushroom/Leaf
	.word LATP_Star		; 3 = Star
	...
LATP_Flower:
	LDA #$00
	STA PUp_StarManFlash	 ; PUp_StarManFlash = 0 (don't activate star man flash)

	LDY #$05	 ; Y = 5 (spawn a mushroom) (index into PRG001 Bouncer_PUp)

	LDA <Player_Suit
	BEQ PRG008_B7F9	 ; If Player is small, jump to PRG008_B7F9

	LDY #$02	 ; Y = 2 (spawn a fire flower) (index into PRG001 Bouncer_PUp)

PRG008_B7F9:
	RTS		 ; Return
```

`LATP_Leaf` is the same with `LDY #$03` (leaf), and `LATP_Star` sets
`PUp_StarManFlash` to `$80` and returns `Y = 4`.

This uses `LATP_Flower` rather than `LATP_Leaf`, because the matching
bytes in `LATP_Leaf` are also in other banks. The first code changes
the `BEQ` to `BPL`, which is always taken, so the `LDY #$02` is
skipped. The second code changes the first `LDY #$05` to `LDY #$03`, so
the block always gives a leaf. The third code points table entry 2
(Mushroom/Leaf) at `LATP_Flower` (`$B7FA` to `$B7EC`), so both kinds of
block run the changed routine.

The optional codes change `LATP_Star` to set `PUp_StarManFlash` to `0`
and return `Y = 3`, so Star blocks give the same item.

**Evidence:** `LATP_JumpTable` and the handlers after it. The source,
`smb3_original.nes` and the built ROM have `E9 B7 EC B7 FA B7 08 B8` at
`$B7D1`, `A9 00 8D 86 05 A0 05 A5 ED F0 02 A0 02 60` at `$B7EC`
(`LATP_Flower`)
and `A9 80 8D 86 05 A0 04 60` at `$B808` (`LATP_Star`). No other bank that
loads at `$A000` has the same bytes at the patched addresses.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only blocks the Player bumps go through this code. Blocks hit by a tail
  swipe or a kicked shell were not checked and may still give their
  normal item.
- Star blocks are not changed unless the optional codes are used.
- 1-up, vine, coin and P-Switch blocks are not changed.

## Every power-up block gives a Starman

**Codes:** `AESLLNKV` + `EUSLGYNU` + `AESLINXN` + `EUSLTYNU` (use all four)
**Raw form:** `B7D3:08:EC` + `B7D4:B8:B7` + `B7D5:08:FA` + `B7D6:B8:B7`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:41:47

Tags: `demo-ready` `play-start` `grab-bag-collection` `long` `curated`

**Mechanism:** When the Player bumps a block, its kind from `LATP_PowerUps`
(`prg008.asm`) picks a handler from `LATP_JumpTable`. Each handler returns an index into
`Bouncer_PUp` (`prg001.asm`) in `Y`:

```
LATP_JumpTable:
	.word LATP_None		; 0 = None
	.word LATP_Flower	; 1 = Mushroom/Flower
	.word LATP_Leaf		; 2 = Mushroom/Leaf
	.word LATP_Star		; 3 = Star
	...
LATP_Flower:
	LDA #$00
	STA PUp_StarManFlash	 ; PUp_StarManFlash = 0 (don't activate star man flash)

	LDY #$05	 ; Y = 5 (spawn a mushroom) (index into PRG001 Bouncer_PUp)

	LDA <Player_Suit
	BEQ PRG008_B7F9	 ; If Player is small, jump to PRG008_B7F9

	LDY #$02	 ; Y = 2 (spawn a fire flower) (index into PRG001 Bouncer_PUp)

PRG008_B7F9:
	RTS		 ; Return
```

`LATP_Leaf` is the same with `LDY #$03` (leaf), and `LATP_Star` sets
`PUp_StarManFlash` to `$80` and returns `Y = 4`.

The codes point table entries 1 (Mushroom/Flower) and 2 (Mushroom/Leaf)
at `LATP_Star` (`$B808`). Each entry needs two codes because the high
byte changes from `$B7` to `$B8`.

**Evidence:** `LATP_JumpTable` and the handlers after it. The source,
`smb3_original.nes` and the built ROM have `E9 B7 EC B7 FA B7 08 B8` at
`$B7D1`, `A9 00 8D 86 05 A0 05 A5 ED F0 02 A0 02 60` at `$B7EC`
(`LATP_Flower`)
and `A9 80 8D 86 05 A0 04 60` at `$B808` (`LATP_Star`). No other bank that
loads at `$A000` has the same bytes at the patched addresses.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Use all four. With only one pair, one kind of block points into the
  middle of other code.
- Only blocks the Player bumps go through this code. Blocks hit by a tail
  swipe or a kicked shell were not checked and may still give their
  normal item.
- 1-up, vine, coin and P-Switch blocks are not changed.

## All ground is even more slippery than ice

**Codes:** `YXNLVSZU` + `EYUZGTEI` + `EYUXATEI` + `AZUZTTAT` + `AZUXZTEG`
(use all five)
**Raw form:** `BDF6:2F:3A` + `A634:F0:D0` + `A638:F0:D0` + `A636:20:60` +
`A63A:20:C0`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:43:01

Tags: `demo-ready` `play-start` `long`

**Mechanism:** The first code is "All ground is slippery like ice", which
sets `Player_Slippery` to 2 everywhere outside ice levels. With
`Player_Slippery` at 2, `Player_GroundHControl` (`prg008.asm`) uses row
`$48` of the acceleration tables. The other four codes change that row
of `Player_XAccelPseudoFrac`:

```
Player_XAccelPseudoFrac_UW:
	; If on the ground		If swimming above the ground
	.byte $30, $00, $00, $00, 	$E0, $30, $80, $00

	.byte $A0, $E0, $C0, $00, 	$A0, $E0, $20, $00
	.byte $D0, $E0, $60, $00, 	$D0, $E0, $C0, $00
```

The last line is the very slippery row (F, N, S, X without B, then with
B). Its `Player_XAccelMain` values are `-1, 0, 0, 0`. Each frame, the
fraction is added to `Counter_Wiggly` and the carry goes into
`Player_XVel`:

```
PRG008_ABF5: 
	LDA <Temp_Var1
	ADD Counter_Wiggly	; actual value not used, looking for a semi-random carry

	LDA <Player_XVel
	ADC <Temp_Var2
	STA <Player_XVel	; Player_XVel += Temp_Var2 (and sometimes carry)
```

For friction (F), the speed goes down by 1 unless there is a carry, so a
higher fraction means less friction. The codes change F from `$D0` to
`$F0`: the Player slows on about 1 frame in 16, instead of 3 in 16 on
ice. For skidding (S, pressing the other way), the fraction is negated
first, so a lower fraction means a weaker skid. The codes change S from
`$60` and `$C0` to `$20`: turning around slows the Player on about 1
frame in 8, instead of 3 in 8 (or 6 in 8 with B) on ice. Speeding up (N)
is not changed.

**Evidence:** `Player_GroundHControl` and the tables above it. The
source, `smb3_original.nes` and the built ROM have `D0 E0 60 00 D0 E0 C0
00` at `$A634` (`Player_XAccelPseudoFrac` is at `$A5EC`, so row `$48`
starts at `$A634`). No other bank that loads at `$A000` has the same
bytes at `$A634`, `$A636`, `$A638` or `$A63A`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Without the first code, only ice blocks in ice levels get the new
  values.
- In ice levels, the ordinary snowy ground still uses the milder row
  (`Player_Slippery` 1), and only ice blocks get the new values.
- The caveats of "All ground is slippery like ice" also apply. The skid
  row is also used in the air, so turning around in a jump is even
  slower.
- Stopping takes a long time, so the Player may slide off ledges and into
  enemies.

## No music

**Code:** `ATKVYUSZ`
**Raw form:** `E34F:60:AD`
(address:new-byte:compare-byte, bank 31, fixed)
**Recorded:** 2026-10-16 22:43:50

Tags: `demo-ready` `play-start`

**Mechanism:** Each frame, `Sound_Process` (`prg028.asm`) runs every
sound player, music last, and then clears the music queues:

```
Sound_Process:
	; Queue + Play all sounds in...
	JSR Sound_PlayLevel2	 ; Level 2
	JSR Sound_PlayPlayer	 ; Player
	JSR Sound_PlayLevel1	 ; Level 1
	JSR Sound_PlayMapSounds	 ; Map sounds
	JSR Sound_PlayMusic	 ; Music

	; Clear any music queues
	LDA #$00	 
	STA Sound_QMusic2
	STA Sound_QMusic1
```

The code changes the first instruction of `Sound_PlayMusic`
(`prg031.asm`), `LDA Sound_QMusic1`, to `RTS`. No song is ever started
or played, and the queued song is thrown away. Sound effects are played
by the other routines and still work.

**Evidence:** `Sound_Process` and `Sound_PlayMusic`. The source,
`smb3_original.nes` and the built ROM have `AD F4 04` at `$E34F`.
`$E000-$FFFF` always holds bank 31, so no other bank is affected.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- `SndCur_Music1` and `SndCur_Music2` stay 0. Code that waits for a
  song to end goes on at once: the time bonus at the end of a level starts
  right away, and the Game Over sequence doesn't wait for its music. Both
  should still work.
- Jingles that are songs are silent too: the warp whistle, the music
  box, Course Clear, Player death and boss victory.

## Small Mario can break bricks

**Code:** `EIKLSAEY`
**Raw form:** `B845:D0:F0`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:44:46

**Mechanism:** `LATP_Brick` (`prg008.asm`) runs when the Player hits a
plain brick:

```
LATP_Brick:
	JSR LATP_GetCoinAboveBlock	; Get coin above block, if any

	CPX #$04
	BEQ PRG008_B84E	 ; If on tile check index 4 (tail attack's tile), jump to PRG008_B84E (bust brick!)

	LDA <Player_Suit
	BNE PRG008_B84E	 ; If Player is not small, jump to PRG008_B84E (bust brick!)

	LDY #$01	 ; Y = 1 (spawn a coin) (index into PRG001 Bouncer_PUp, i.e. nothing)

	RTS		 ; Return
```

The code changes the `BEQ` to `BNE`. Any hit from the head or feet (tile
check index 0-3) now busts the brick. A tail hit (index 4) falls through
to the suit check, and the Player always has a suit when using the tail,
so it busts the brick as before.

**Evidence:** `LATP_Brick`. The source, `smb3_original.nes` and the
built ROM have `20 2A B9 E0 04 F0 07 A5 ED D0 03 A0 01 60` at `$B840`.
`LATP_GetCoinAboveBlock` saves and restores `X`. No other bank that
loads at `$A000` has `$F0` at `$B845`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Changing `BNE` at `$B849` to `BPL`, or `LDA <Player_Suit` at `$B847` to
  `LDA #`, was rejected because the compare byte is in another bank (3 and
  9).
- Only plain bricks change. Bricks that hold a coin, a power-up, a vine
  or a P-Switch work as before.
- `LATP_BrickAltClear` (the unused pipe junction tile) calls
  `LATP_Brick` and is affected in the same way.

## Any pipe or door ends the level

**Codes:** pipes: `PAXUXYGA` + `GAXUKYLA` + `EPXUSYEY`; doors:
`PASZOALA` + `GPVXUAEP` (each set works alone)
**Raw form:** pipes: `BF2A:01:04` + `BF2C:04:03` + `BF2D:90:F0`; doors:
`A851:01:03` + `A86B:14:90`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:48:23

**Mechanism (pipes):** `PipeEntryPrepare` (`prg008.asm`) turns the pipe
type in `Y` into a pipe behavior in `A`, which goes into bits 2-4 of
`Level_PipeMove`:

```
	; If Y = 3, A = 4
	LDA #$04
	CPY #$03
	BEQ PRG008_BF49
	...
	; Use A = 1 unless Level_PipeNotExit is set, in which case use A = 3
	LDA #$01	; A = 1 (pipe will exit level)
```

When the pipe trip ends, `prg029.asm` copies the behavior to
`Level_JctCtl`, and behavior 1 means "return to map":

```
	STA Level_JctCtl ; Push it right into Level_JctCtl!
	CMP #$01	 ; Although Level_JctCtl = 1 is not valid, this is used as "return to map"...
	BNE PRG029_D395	 ; If Level_JctCtl <> 1, jump to PRG029_D395

	INC <Level_ExitToMap	 ; Level_ExitToMap = 1 (return to map)

	LDA #$00
	STA Map_ReturnStatus	 ; Map_ReturnStatus = 0 (completed level, i.e. did not die)
```

The pipe codes change the first test to `LDA #$01` / `CPY #$04` /
`BCC PRG008_BF49`. The pipe type is always 0-3, so the branch is always
taken with `A = 1`, and every pipe (alternate area, Big ? Block area,
in-level transit and exit area pipes) returns to the map as a cleared
level.

**Mechanism (doors):** The door logic sets `Level_JctCtl` to 1 or 3 and
then snaps `Player_X` to the doorway:

```
	LDY #$01	; Y = 1

	LDA Level_PipeNotExit
	BEQ PRG008_A852	 ; If pipe should exit to map, jump to PRG008_A852

	LDY #$03	 ; Otherwise, Y = 3

PRG008_A852:
	STY Level_JctCtl ; Set appropriate value to Level_JctCtl

	LDY #0
	STY Map_ReturnStatus	 ; Map_ReturnStatus = 0
	...
PRG008_A864:
	TYA	
	ADD <Player_X	 ; Add offset to Player_X if needed
	AND #$F0	 ; Lock to nearest column (place directly in doorway)
	STA <Player_X	 ; Update Player_X
```

The first door code changes `LDY #$03` to `LDY #$01`, so
`Level_JctCtl` is always 1, as it is after an exit pipe. The second
changes `STA <Player_X` to `STA <Level_ExitToMap` (`$14`). The doorway
column is nonzero for most doors, so `Level_ExitToMap` is set, with
`Map_ReturnStatus` already 0.

**Evidence:** `PipeEntryPrepare`, the door logic in `Player_Control`,
and the pipe-end code in `prg029.asm`. The source, `smb3_original.nes`
and the built ROM have `A9 04 C0 03 F0 1A` at `$BF29`, `A0 01 AD 75 03 F0
02 A0 03 8C DE 03` at `$A849` and `29 F0 85 90` at `$A868`.
`Level_ExitToMap` is `$0014`. Other code only checks it for zero or
nonzero. No other bank that loads at `$A000` has the same bytes at
`$BF2A`, `$BF2C`, `$BF2D`, `$A851` or `$A86B`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Each level counts as cleared, and the Player keeps their suit, as with
  a normal exit pipe. No card is given.
- Doors: `Player_X` is no longer snapped to the doorway. A door at the
  left edge of a 256-pixel page (doorway column `$00`) stores 0, so it
  works as a normal door, but goes to the area in the level header
  rather than the general junction. Changing `AND #$F0` to `ORA` would
  fix that, but `$A868` matches bank 1.
- Doors: storing to `Level_JctCtl` with a RAM mirror of `$14` was rejected:
  `$A854` matches bank 19 (level loading code).
- Fortress doors that lead to the boss room end the level too, so the
  boss and its orb are skipped.

## Swim through the air

**Codes:** `AAKZNPGA` + `YAKXOOTA` (use both)
**Raw form:** `A947:00:04` + `A949:07:0E`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:50:43

Tags: `demo-ready` `play-start`

**Mechanism:** The Player's movement code is picked from a table of
pointers by suit (`prg008.asm`, "MOVEMENT LOGIC PER POWER-UP / SUIT"):

```
	LDA Player_Kuribo
	BEQ PRG008_A94C	 ; If Player is not wearing Kuribo's shoe, jump to PRG008_A94C

	; If in Kuribo's shoe...

	LDA #14		 ; A = 14 (Kuribo's shoe code pointer)
	BNE PRG008_A956	 ; Jump (technically always) to PRG008_A956

PRG008_A94C:
	LDA <Player_Suit

	LDY Player_InWater
	BEQ PRG008_A956	 ; If Player is not under water, jump to PRG008_A956

	ADD #$07	 ; Otherwise, add 7 (underwater code pointers)

PRG008_A956:
	ASL A		 ; 2-byte pointer
	TAY		 ; -> Y
```

`PowerUpMovement_JumpTable` has ground movement for suits 0-6 (entries
0-6), underwater movement for suits 0-6 (entries 7-13) and `Move_Kuribo`
(entry 14).

The first code is the one from "Hop around as if in Kuribo's Shoe": it
sets the `BEQ` offset to `$00`, so the `LDA #14` path is always taken.
The second code changes the `14` to `7`, so entry 7 (`Swim_SmallBigLeaf`) is always
used, on land and under water.

`Swim_SmallBigLeaf` runs `Player_UnderwaterHControl`, `Player_SwimV` and
`Player_SwimAnim`. Each press of A is a swim stroke, the Player sinks
slowly, and the swim frames are shown.

**Evidence:** The movement dispatch and `Swim_SmallBigLeaf`. The source,
`smb3_original.nes` and the built ROM have `AD 77 05 F0 04 A9 0E` at
`$A943`. No other bank that loads at `$A000` has `$04` at `$A947` or `$0E`
at `$A949`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only the movement changes. The game still thinks the Player is out of
  water, so there are no bubbles.
- Fire and Hammer Mario don't get the throwing animation, because
  `Player_ShootAnim` is only in `Swim_FireHammer`.
- Kuribo's Shoe uses this path too, so the shoe's own movement is
  replaced as well.

## Swim through the air like Frog Mario

**Codes:** `AAKZNPGA` + `LAKXOOTE` (use both)
**Raw form:** `A947:00:04` + `A949:0B:0E`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:50:44

Tags: `demo-ready` `play-start`

**Mechanism:** The Player's movement code is picked from a table of
pointers by suit (`prg008.asm`, "MOVEMENT LOGIC PER POWER-UP / SUIT"):

```
	LDA Player_Kuribo
	BEQ PRG008_A94C	 ; If Player is not wearing Kuribo's shoe, jump to PRG008_A94C

	; If in Kuribo's shoe...

	LDA #14		 ; A = 14 (Kuribo's shoe code pointer)
	BNE PRG008_A956	 ; Jump (technically always) to PRG008_A956

PRG008_A94C:
	LDA <Player_Suit

	LDY Player_InWater
	BEQ PRG008_A956	 ; If Player is not under water, jump to PRG008_A956

	ADD #$07	 ; Otherwise, add 7 (underwater code pointers)

PRG008_A956:
	ASL A		 ; 2-byte pointer
	TAY		 ; -> Y
```

`PowerUpMovement_JumpTable` has ground movement for suits 0-6 (entries
0-6), underwater movement for suits 0-6 (entries 7-13) and `Move_Kuribo`
(entry 14).

The first code is the one from "Hop around as if in Kuribo's Shoe": it
sets the `BEQ` offset to `$00`, so the `LDA #14` path is always taken.
The second code changes the `14` to `11`, so entry 11 (`Swim_Frog`) is always
used, on land and under water.

`Swim_Frog` sets `Player_YVel` straight from Up and Down (doubled when
A is held) and slows the Player when they are let go. It is the Frog
suit's swimming, so the Player can move freely in all directions in
the air, in any suit.

**Evidence:** The movement dispatch and `Swim_Frog`. The source,
`smb3_original.nes` and the built ROM have `AD 77 05 F0 04 A9 0E` at
`$A943`. No other bank that loads at `$A000` has `$04` at `$A947` or `$0E`
at `$A949`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The frog swim frames are used with the current suit's graphics, so the
  Player may look wrong.
- Only the movement changes. The game still thinks the Player is out of
  water, so there are no bubbles.
- Kuribo's Shoe uses this path too, so the shoe's own movement is
  replaced as well.

## Hop around like Frog Mario in any suit

**Codes:** `AAKZNPGA` + `GAKXOOTA` (use both)
**Raw form:** `A947:00:04` + `A949:04:0E`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:50:44

Tags: `demo-ready` `play-start`

**Mechanism:** The Player's movement code is picked from a table of
pointers by suit (`prg008.asm`, "MOVEMENT LOGIC PER POWER-UP / SUIT"):

```
	LDA Player_Kuribo
	BEQ PRG008_A94C	 ; If Player is not wearing Kuribo's shoe, jump to PRG008_A94C

	; If in Kuribo's shoe...

	LDA #14		 ; A = 14 (Kuribo's shoe code pointer)
	BNE PRG008_A956	 ; Jump (technically always) to PRG008_A956

PRG008_A94C:
	LDA <Player_Suit

	LDY Player_InWater
	BEQ PRG008_A956	 ; If Player is not under water, jump to PRG008_A956

	ADD #$07	 ; Otherwise, add 7 (underwater code pointers)

PRG008_A956:
	ASL A		 ; 2-byte pointer
	TAY		 ; -> Y
```

`PowerUpMovement_JumpTable` has ground movement for suits 0-6 (entries
0-6), underwater movement for suits 0-6 (entries 7-13) and `Move_Kuribo`
(entry 14).

The first code is the one from "Hop around as if in Kuribo's Shoe": it
sets the `BEQ` offset to `$00`, so the `LDA #14` path is always taken.
The second code changes the `14` to `4`, so entry 4 (`GndMov_Frog`) is always
used, on land and under water.

`GndMov_Frog` stops the Player between hops (`Player_FrogHopCnt`) and
plays the frog hop sound, so the Player moves along in short hops when
Left or Right is held, as Frog Mario does on land. Under water, it
uses the same ground movement, so the Player can't swim.

**Evidence:** The movement dispatch and `GndMov_Frog`. The source,
`smb3_original.nes` and the built ROM have `AD 77 05 F0 04 A9 0E` at
`$A943`. No other bank that loads at `$A000` has `$04` at `$A947` or `$0E`
at `$A949`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The frog hop frames are used with the current suit's graphics, so the
  Player may look wrong.
- The speeds still come from the current suit. Only the Frog suit has
  the faster frog acceleration.
- With this code on, the Player can't swim under water.
- Kuribo's Shoe uses this path too, so the shoe's own movement is
  replaced as well.

## Swim in the air and walk under water

**Code:** `EISZOPEY`
**Raw form:** `A951:D0:F0`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:50:44

**Mechanism:** The Player's movement code is picked from a table of
pointers by suit (`prg008.asm`, "MOVEMENT LOGIC PER POWER-UP / SUIT"):

```
	LDA Player_Kuribo
	BEQ PRG008_A94C	 ; If Player is not wearing Kuribo's shoe, jump to PRG008_A94C

	; If in Kuribo's shoe...

	LDA #14		 ; A = 14 (Kuribo's shoe code pointer)
	BNE PRG008_A956	 ; Jump (technically always) to PRG008_A956

PRG008_A94C:
	LDA <Player_Suit

	LDY Player_InWater
	BEQ PRG008_A956	 ; If Player is not under water, jump to PRG008_A956

	ADD #$07	 ; Otherwise, add 7 (underwater code pointers)

PRG008_A956:
	ASL A		 ; 2-byte pointer
	TAY		 ; -> Y
```

`PowerUpMovement_JumpTable` has ground movement for suits 0-6 (entries
0-6), underwater movement for suits 0-6 (entries 7-13) and `Move_Kuribo`
(entry 14).

The code changes `BEQ PRG008_A956` to `BNE`. Out of water, 7 is added, so
the Player uses their suit's underwater movement (swimming in the air).
Under water, nothing is added, so the Player uses ground movement:
walking, running and jumping along the bottom.

**Evidence:** The movement dispatch. The source, `smb3_original.nes` and
the built ROM have `A5 ED AC 75 05 F0 03 18 69 07 0A A8` at `$A94C`. No
other bank that loads at `$A000` has `$F0` at `$A951`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only the movement code changes. `Player_InWater` is unchanged, so
  splashes and bubbles still happen only in water. How jumps feel under
  water was not checked.
- The Frog suit swims freely in the air (`Swim_Frog`).
- Walking under water with only the movement code changed was not made:
  changing `ADD #$07` to `AND` matches bank 15, and changing the `$07` to
  `$00` matches bank 18.
- Kuribo's Shoe is not affected.

## Turn into a statue without the Tanooki suit

**Codes:** small: `SIVZSPUA` + `XZVZVOOX`; big: `SIVZNOVZ` + `XZVXEOOX`;
Raccoon: `SIVXUPUI` + `XZVXKOOX` (use the pair for each suit you want)
**Raw form:** small: `A965:D5:83` + `A966:AA:A9`; big: `A967:D5:AE` +
`A968:AA:A9`; Raccoon: `A96B:D5:D3` + `A96C:AA:A9`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 22:52:44

Tags: `demo-ready` `play-start` `curated`

**Mechanism:** The Player's movement code is picked from
`PowerUpMovement_JumpTable` by suit (`prg008.asm`, "MOVEMENT LOGIC PER
POWER-UP / SUIT"). Only the Tanooki entry calls the statue code:

```
GndMov_Tanooki:
	JSR Player_TanookiStatue  ; Change into/maintain Tanooki statue (NOTE: Will not return here if statue!)
	JSR Player_GroundHControl ; Do Player left/right input control
	JSR Player_JumpFlyFlutter ; Do Player jump, fly, flutter wag
	JSR Player_AnimTailWag ; Do Player's tail animations
	JSR Player_TailAttackAnim ; Do Player's tail attack animations
	RTS		 ; Return
```

`Player_TanookiStatue` turns the Player into a statue when Down and B
are pressed together, and doesn't check the suit. Each pair of codes
changes one ground entry of the table (small `GndMov_Small`, big
`GndMov_Big` or Raccoon `GndMov_Leaf`) to `GndMov_Tanooki` (`$AAD5`).
That suit then moves like the Tanooki suit on land, including the statue.

**Evidence:** `PowerUpMovement_JumpTable` and `GndMov_Tanooki`. The
source, `smb3_original.nes` and the built ROM have `83 A9 AE A9 B9 A9 D3
A9 E1 A9 D5 AA B9 A9` at `$A965`. No other bank that loads at `$A000` has
the same bytes at `$A965`-`$A968`, `$A96B` or `$A96C`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The statue frame (`PF_STATUE`) is drawn from the current suit's
  graphics pages (`Player_PUpRootPage`), not the Tanooki's, so the
  statue will probably look wrong. It uses the statue palette, and the
  statue still works: enemies can't hurt the Player.
- `Player_TailAttackAnim` sets `Player_TailAttack` (a tail swipe) on every
  B press. Whether the swipe hits enemies and blocks for these suits was
  not checked.
- `Player_AnimTailWag` plays the tail wag sound and uses tail frames when
  A is pressed in the air. These frames will look wrong for small and big
  Mario.
- Small Mario loses the small jump frame, and big Mario loses
  `Player_SoarJumpFallFrame`.
- Fire, Frog and Hammer were not made: the high byte of each of their
  entries (`$A96A`, `$A96E`, `$A972`) matches other banks (20 and 26, 7,
  and 4 and 20).
- Under water, the usual swim code runs, so there's no statue there.

## Spawn multiple copies of each enemy

**Codes:** `AAKUUPEA` (horizontal levels) + `AEEUNLEA` (vertical levels)
**Raw form:** `B94B:00:80` + `BB8F:00:80`
(address:new-byte:compare-byte, bank 5)
**Recorded:** 2026-10-16 22:53:49

Tags: `demo-ready` `play-start` `grab-bag-collection` `curated`

**Mechanism:** When `Level_ObjectsSpawnByScroll` (`prg005.asm`) spawns
an object, it sets bit 7 of the object's `Level_ObjectsSpawned` byte:

```
	LDY <Temp_Var2			; Object index -> 'Y'

	; Mark this object as spawned
	LDA Level_ObjectsSpawned,Y
	ORA #$80
	STA Level_ObjectsSpawned,Y
```

The spawn loop skips objects with bit 7 set (`BMI PRG005_B863`), so each
object exists only once until it is removed. `Level_ObjectsSpawnByScrollV`
has the same code for vertical levels. The codes change both `ORA #$80`
to `ORA #$00`, so the object is never marked. It spawns again on each
frame while its column is the look-ahead column, while there is a free
object slot.

**Evidence:** `Level_ObjectsSpawnByScroll` and
`Level_ObjectsSpawnByScrollV`. The source, `smb3_original.nes` and the
built ROM have `B9 10 7B 09 80 99 10 7B` at `$B947` and `$BB8B`. No
other bank that loads at `$A000` has `$80` at `$B94B` or `$BB8F`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- How many copies appear depends on scroll speed: the look-ahead column
  changes every 16 pixels, so walking slowly makes more copies. Standing
  still with the look-ahead column on an enemy fills all five object
  slots with it.
- This applies to every object placed in the level, including bosses,
  items, platforms and the end-of-level card. The card always goes in
  slot 6, so it is started again each frame while its column is the
  look-ahead column.
- Objects that mark themselves (Toad and the King, autoscroll, the
  treasure box, the bonus controller) use a separate `ORA #$80` and are
  not changed.

## Kicked shells hit every enemy on screen

**Codes:** `AAXSKGGZ` + `AAKISKLA` (use both)
**Raw form:** `DC2C:00:24` + `DC45:00:0B`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-16 22:54:54

Tags: `kids-collection`

**Mechanism:** A kicked shell calls `ObjectToObject_HitTest`
(`prg000.asm`) every other frame. It loops over the other object slots,
skips objects that are off screen or in a state that doesn't collide, and
then tests the two bounding boxes:

```
	JSR Object_CalcBoundBox	 ; Calculate OTHER object's bounding box
	JSR ObjectObject_Intersect	 ; Returns carry SET if object and OTHER object intersected
	BCC PRG000_DC51	 	; If object did NOT intersect the OTHER object, jump to PRG000_DC51 (do nothing)
	...
	ADC #$80	
	LDA <Temp_Var1	
	ADC #$00	
	BNE PRG000_DC51	 ; If the objects did not actually collide, jump to PRG000_DC51 (do nothing)

	SEC		 ; Object's collided!  Set carry!
```

The codes set both branch offsets to `$00`, so any other object that is
on screen counts as hit. Objects with `OAT_HITNOTKILL` (such as
power-ups) are still skipped. Each test returns the first object found,
and the shell kills it, so the shell kills the enemies on screen one at a
time, about every other frame, with the usual rising points.

**Evidence:** `ObjectToObject_HitTest`. The source, `smb3_original.nes`
and the built ROM have `20 2F D9 20 7B D9 90 24` at `$DC25` and `69 80 A5
00 69 00 D0 0B 38` at `$DC3E`. No other bank that loads at `$C000` has the
same bytes at `$DC2C` or `$DC45`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- A held shell or other held object uses the same test. It will "hit"
  any enemy on screen at once, and the held object is killed (as when a
  held shell touches an enemy). A held object can only be carried while
  no other enemy is on screen.
- Two kicked shells always hit each other and both die.
- Enemies that ignore shell hits in their own code still ignore them.

## Pick up and kick any enemy

**Codes:** `AAUIYUIP` + `AAXKOTOT` (use both)
**Raw form:** `D337:00:1D` + `CE29:00:E1`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-16 22:57:49

Tags: `demo-ready` `play-start` `grab-bag-collection`

**Mechanism:** When the Player touches an enemy anywhere except from
above, `Player_HitEnemy` (`prg000.asm`) jumps to
`Object_HoldKickOrHurtPlayer`:

```
Object_HoldKickOrHurtPlayer:
	LDA Objects_State,X
	CMP #OBJSTATE_SHELLED
	BNE PRG000_D355	 ; If object state is not Shelled, jump to PRG000_D355 (hurt Player!)
	...
	BIT <Pad_Holding
	BVS PRG000_D34F	 	; If Player is holding B, jump to PRG000_D34F
	JMP Player_KickObject	 ; Kick away the object and don't come back!

PRG000_D34F:
	LDA #OBJSTATE_HELD
	STA Objects_State,X
```

`AAUIYUIP` sets the `BNE` offset to `$00`, so every enemy is
treated as a shell: holding B picks it up, and otherwise the Player
kicks it.

That alone isn't enough. A held object calls `Object_ShellDoWakeUp`
every frame from `ObjState_Held`. Once `Objects_Timer3` reaches 0,
the object "wakes up" and goes back to its Normal state. A shell starts
that timer at `$FF` when it is stomped, but an enemy that was never
stomped has 0 there, so it would leave the Player's hands at once.
`AAXKOTOT` changes `JSR Object_ShellDoWakeUp` (`JSR $D0E1`) to
`JSR $D000`, which is an `RTS` inside the same routine. A held
object then never wakes up.

**Evidence:** `Object_HoldKickOrHurtPlayer` and `ObjState_Held`.
The source, `smb3_original.nes` and the built ROM have `C9 03 D0 1D`
at `$D334`, `20 E1 D0` at `$CE28` and `60` at `$D100`. No
other bank that loads at `$C000` has the same bytes at `$D337` or
`$CE29`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Touching an enemy from the side never hurts the Player. Without B
  held, the enemy is kicked like a shell. This includes enemies that are
  normally only dangerous, such as Spinies.
- Landing on an enemy that can't be stomped (such as a Spiny), or on
  any enemy under water, still hurts the Player. That path skips
  `Object_HoldKickOrHurtPlayer`.
- Only enemies that use the shared `Player_HitEnemy` routine are
  changed. Bosses and some others have their own contact code.
- A kicked enemy uses the kicked-shell code for movement. It slides
  along the ground and kills the enemies it hits, but it may be drawn
  with its normal walking frames.
- Held shells no longer wake up in the Player's hands, and a held object
  pushed into a wall is no longer killed by the wake-up check. (The
  separate check in `Player_KickObject` still kills an object kicked
  into a wall.)
- In Kuribo's Shoe or as a statue, touching any enemy kicks it.

## Basic walking enemies move 4x as fast

**Codes:** `XKELIUAI` + `OXELTLUU` (use both)
**Raw form:** `B385:C2:58` + `B386:A9:B3`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-16 22:59:58

Tags: `demo-ready` `play-start` `grab-bag-collection` `curated`

**Optional 2x speed instead:** `TSELIUAS` alone (`B385:5E:58`).

**Mechanism:** Eight enemies share `ObjNorm_GroundTroop`
(`prg004.asm`): Green Koopa Troopa, the hopping green Paratroopa, Buzzy
Beetle, Spiny, Goomba, and the big Green Troopa, big Goomba and big
hopper. Every frame it sets the enemy's X velocity from a table, by the
way it faces:

```
GroundTroop_XVel:
	.byte -$08, $08
	.byte -$06, $06
	.byte -$05, $05

	.byte $F0, $10, $F4, $0C, $F7, $09
	...
	; Set proper ground troop X velocity
	LDA GroundTroop_XVel,Y
	STA <Objects_XVel,X
```

Rows 2 and 3 are only used just after a left/right bouncing block has
hit the enemy.

The table has no Game Genie-safe way to change the normal row: bank 8
also has `$08` at `$B359`, in Player code. So the codes point
`LDA GroundTroop_XVel,Y` at `$A9C2` instead. That address is the
second byte of `FireBro_FireballXVel` (`-$20`), followed by a
`JSR` opcode (`$20`). The enemies read `-$20` and `$20`,
4x the usual `$08`.

The 2x code points the `LDA` at the unused `$F0, $10` row in
the table itself.

**Evidence:** `ObjNorm_GroundTroop` and `FireBro_FireballXVel`.
The source, `smb3_original.nes` and the built ROM have `F8 08 FA 06
FB 05 F0 10 F4 0C F7 09` at `$B358`, `B9 58 B3` at `$B384`
and `20 E0 20` at `$A9C1`. No other bank that loads at `$A000`
has the same bytes at `$B385` or `$B386`.

Not used: `ETSUAUEY` + `AZSUPUAA` (`B358:E0:F8` +
`B359:20:08`) would change the table, but the second code would also
change bank 8.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The hoppers (the green Paratroopa and the big hopper) jump just as high
  and stay in the air just as long, so each hop goes about 4x as far.
- Only the speed changes. The walking animation runs at the usual rate.
- With the 4x codes, an enemy hit by a left/right bouncing block reads
  whatever follows (`$47`, `$C4`, `$AD`, `$F1`) for a
  moment, so it shoots off very fast. The 2x code reads sensible values
  there.
- Shells, kicked enemies and other enemy types are not changed.

## Bullet Bills and Missile Bills fly at 1/3 speed

**Codes:** `VUULXNPZ` + `SUULUNNU` + `AASLIOAO` + `EYSLTOEV` (use all
four)
**Raw form:** `BFB2:B6:29` + `BFB3:BD:BF` (bank 7) + `B155:08:18` +
`B156:F8:E8` (bank 4)
(address:new-byte:compare-byte)
**Recorded:** 2026-10-16 23:04:53

Tags: `long` `kids-collection`

**Mechanism:** `CFire_BulletBill` (`prg007.asm`) fires a Bill toward
the Player at a fixed speed:

```
Bill_XVelTowardsPlayer:	.byte $18, -$18
	...
	; Bill fires towards Player
	LDA Bill_XVelTowardsPlayer,Y
	STA <Objects_XVel,X
```

Bank 9 also has `$18` at `$BF29`, so the table can't be changed
safely. Instead, the first two codes point the `LDA` at `$BDB6`,
the middle of `FourWay_CPXOff`, which has `$08, $F8`. Bills now
fire at `$08` (half a pixel per frame) instead of `$18`.

A Missile Bill (`OBJ_BULLETBILLHOMING`) later turns around in
`ObjNorm_BulletBill` (`prg004.asm`). It speeds up one step at a time
until its speed is exactly `BulletBill_XLimit` (`$18, -$18`). The
last two codes change that limit to `$08, -$08`, so it stops at the
new speed.

**Evidence:** `CFire_BulletBill`, `ObjNorm_BulletBill` and
`FourWay_CPXOff`. The source, `smb3_original.nes` and the built ROM
have `B9 29 BF` at `$BFB1`, `18 E8` at `$BF29` and `08 F8` at
`$BDB6` (bank 7), and `18 E8` at `$B155` (bank 4). No other bank
that loads at `$A000` has the same bytes at `$BFB2`, `$BFB3`,
`$B155` or `$B156`.

Not used: `TAXUONAP` (`BF29:06:18`), which would change the table
itself, but bank 9 has the same byte there.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- 1/3 speed, not 1/4. No `$06, $FA` pair was found in bank 7.
- Without the last two codes, a Missile Bill turns around and then
  speeds up to the usual `$18`.
- Other projectiles (hammers, fireballs, cannonballs and so on) are not
  changed. They all move through `SObj_AddXVelFrac` and
  `SObj_AddYVelFrac` (`prg007.asm`), and the bytes that would need to
  change in `SObj_AddXVelFrac` are also in banks 1 and 5.

## Enemies face away from Mario

**Code:** `ALUSOIAP`
**Raw form:** `DD39:30:10`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-16 23:08:36

Tags: `demo-ready` `play-start` `grab-bag-collection` `kids-collection` `curated`

**Mechanism:** Almost every enemy that turns toward the Player, walks
toward him or aims at him asks `Level_ObjCalcXDiffs` (`prg000.asm`)
which side he is on. It is called from about 80 places:

```
Level_ObjCalcXDiffs:
	LDA <Player_X
	SUB <Objects_X,X
	STA <Temp_Var16	 ; Temp_Var16 = difference between Player's X and object's X

	LDY #$00	 ; Y = 0
	LDA <Player_XHi
	SBC <Objects_XHi,X
	BPL PRG000_DD3C	 ; If Player's X Hi >= Object's X Hi, jump to PRG000_DD3C (RTS)

	INY		 ; Otherwise Y = 1
```

The code changes `BPL` to `BMI`, so `Y` is always the wrong side.
Goombas, Koopa Troopas, Buzzy Beetles and Spinies start out facing and
walking away from the Player. Paragoombas keep turning away from him.
Bros, Bullet Bill cannons and other enemies that aim at the Player aim
the other way.

**Evidence:** `Level_ObjCalcXDiffs` and its callers, for example
`ObjInit_GroundTroop` and the Paragoomba code (`prg004.asm`). The
source, `smb3_original.nes` and the built ROM have `A5 90 38 F5 91 85
0F A0 00 A5 75 F5 76 10 01 C8 60` at `$DD2C`. No other bank that
loads at `$C000` has `$10` at `$DD39`.

A code for only the walking enemies was not possible. Their init reads
`LDA GroundTroop_FlipTowardsPlayer,Y` at `$B22D`. Swapping the
table (`$B21E`), the high byte of the address (`$B22F`) or the
`LDA` opcode (`$B22D`) would also change banks 18, 21 or 12.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- This changes everything that uses `Level_ObjCalcXDiffs`, not just
  enemies. Some things that will be backward:
  - A kicked shell goes back toward the Player's side instead of
    away from him, and Mario turns to face away from it.
  - A shell that wakes up in Mario's hands faces away from him.
  - Enemies knocked out by a shell, a tail or a block fly toward the
    Player instead of away.
  - The "kicked shell moving the same way as the Player" check in
    `Object_HoldKickOrHurtPlayer` gets the wrong side, so a shell
    the Player is running alongside may or may not hurt him.
- Enemies that bounce off walls and each other still turn around as
  usual, so a Goomba that hits a wall comes back.

## P-meter fills as soon as you move with B held

**Codes:** `AENXGGTZ` + `PANZXUAZ` (use both)
**Raw form:** `A4FC:00:26` + `AB72:01:28`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-16 23:09:36

Tags: `demo-ready` `play-start` `grab-bag-collection` `curated`

**Mechanism:** The P-meter (`Player_Power`) fills one bit at a time,
and 7 bits (`$7F`) is full. `Player_PowerUpdate` (`prg008.asm`)
only adds a bit when `Player_RunFlag` is set, and only when the
`Player_PMeterCnt` timer has run down (`$08` frames per bit while
running):

```
PRG008_A4F8:
	LDA Player_PMeterCnt
	BNE PRG008_A523	 ; If Player_PMeterCnt <> 0, jump to PRG008_A523

	SEC		 ; Set carry
	ROL Player_Power ; Player_Power is shifted left 1, its old bit 7 in the carry, and '1' introduced at bit 0
```

`Player_GroundHControl` sets `Player_RunFlag` only on the ground,
with B held, at full running speed:

```
	LDA <Temp_Var3
	CMP #PLAYER_TOPRUNSPEED
	BMI PRG008_AB78	 ; If Player's X Velocity magnitude is less than PLAYER_TOPRUNSPEED, jump to PRG008_AB78

	; Player is going fast enough while holding B on the ground; flag running!
	INC Player_RunFlag ; Player_RunFlag = 1
```

The first code sets the `BNE` offset to `$00`, so the timer is never
waited on and the meter changes one bit every frame. The second code
changes `CMP #PLAYER_TOPRUNSPEED` (`$28`) to `CMP #$01`, so any
movement on the ground with B held counts as running. The meter is full
7 frames after you start moving.

An earlier version used only the first code, and it was reported not
to work. It still needed full running speed first, which takes about as
long as filling the meter normally. That is probably why it made little
difference, but this was not confirmed.

**Evidence:** `Player_PowerUpdate` and `Player_GroundHControl`. The
source, `smb3_original.nes` and the built ROM have `AD 15 05 D0 26`
at `$A4F8` and `A5 02 C9 28 30 03 EE 7B 05` at `$AB6F`. No other
bank that loads at `$A000` has the same bytes at `$A4FC` or
`$AB72`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- With a full meter the top speed is the P-meter speed, so the Player
  quickly reaches it. The Raccoon and Tanooki suits can take off after a
  few steps.
- The meter also empties one bit per frame when you stop running or
  jump before it is full.

## Falling into a pit bounces the Player back up

**Codes:** `EPOZTGAA` + `NGOXAKSV` + `GZOXPGAX` + `SZXZLGSA` (use all
four)
**Raw form:** `A416:90:00` + `A418:CF:ED` + `A419:2C:20` + `A423:A5:85`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-17 11:32:45

Tags: `demo-ready` `long` `play-start`

**Mechanism:** This is the SMB3 version of the SMW code "Falling into a
pit launches Mario back up instead of killing him". Every frame,
`Player_Update` (`prg008.asm`) checks if the Player has fallen to the
bottom of the screen:

```
	; "Fall in a pit and die"!
	LDA <Player_SpriteY
	AND #$F0
	CMP #$c0
	BNE PRG008_A427	 ; If Player_SpriteY < $C0 && Player_SpriteY > $CF, jump to PRG008_A427

	; Fell in a pit and died
	LDA #PLAYERSUIT_SMALL
	STA <Player_Suit ; Player_Suit = PLAYERSUIT_SMALL

	JSR Player_Die	 ; Begin death sequence

	; This jumps the initial part of the death sequence
	LDA #$c0
	STA Event_Countdown ; Event_Countdown = $C0
	LDA #$02
	STA <Player_IsDying	; Player_IsDying = 2 (already dropped off screen)

	BNE PRG008_A472	 	; Jump (technically always) to PRG008_A472
```

The four codes change this to:

```
	LDA #-$70
	STA <Player_YVel	; Player_YVel = -$70
	BIT Player_Die		; (was JSR; now does nothing)
	LDA #$c0
	STA Event_Countdown
	LDA #$02
	LDA <Player_IsDying	; (was STA) A = 0, since the Player isn't dying here
	BNE PRG008_A472		; Not taken, so normal play goes on at PRG008_A427
```

So the Player keeps his suit, doesn't die, and is sent up at `-$70`. The
check runs again on every frame he is still in the `$C0`-`$CF` band, so
he always leaves it going up.

Without A held, gravity is 5 per frame, so `-$70` rises about 78
pixels, about 5 tiles. A normal jump starts at `-$38` (`PLAYER_JUMP`).
With A held, gravity is only 1 until the speed slows to `-$20`, so the
bounce is very high (more than a screen) and he drops back from above.

**Variants:** the bounce speed is the first code. It is signed, and a
negative value is up.

| Code | Value | Effect |
|------|-------|--------|
| `EPOZTGAA` | `$90` | -$70, about 5 tiles without A |
| `EZOZTGAA` | `$A0` | -$60, about 3.5 tiles without A; may not clear a high ledge |
| `EAOZTGAA` | `$80` | -$80, about 6.5 tiles without A |

**Evidence:** `smb3_original.nes` has `29 F0 C9 C0 D0 12 A9 00 85 ED 20
7C DA A9 C0 8D 10 05 A9 02 85 F1 D0 4B` at `$A40F` in bank 8.
`Player_YVel` is `$CF` (`Player_Die` has `A9 C0 85 CF` at `$DAA1`),
`Player_Suit` is `$ED` and `Player_IsDying` is `$F1`. No other bank
that loads at `$A000` has the same compare bytes. `LDA #$02` was not
changed to `#$00` because bank 5 has `$02` at `$A422`; changing the
`STA` to `LDA` avoids that. `Player_IsDying` must be 0 here, because
the code before it skips this check while the Player is dying.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- If there is no ground the Player can reach, he falls back in and
  bounces again. It is an endless trampoline, not a death.
- This conflicts with "Falling into a pit drops the Player back in from
  the top of the screen". Both change `$A418`.
- Holding A makes the bounce go far off the top of the screen. He comes
  back down where he went up, unless he steers.
- `Event_Countdown` is still set to `$C0` on each bounce. The pipe code
  and `Player_Die` set it themselves before they use it, so this should
  not matter.
- Only the pit check changes. Enemies and the timer still kill the Player
  as usual.
- In levels where the screen scrolls up and down, this is the same
  "bottom of the screen" test the game uses for pit deaths, so the
  bounce happens wherever a pit death would.

## Falling into a pit drops the Player back in from the top of the screen

**Codes:** `NAOXAKST` + `GZOXPGAX` + `SZXZLGSA` (use all three)
**Raw form:** `A418:87:ED` + `A419:2C:20` + `A423:A5:85`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-17 12:55:18

Tags: `demo-ready` `play-start` `kids-collection` `curated`

**Mechanism:** This is the SMB3 version of the SMW code "Falling into a
pit drops Mario back in from the top of the level". It changes the same
pit check in `Player_Update` (`prg008.asm`) as "Falling into a pit
bounces the Player back up":

```
	; Fell in a pit and died
	LDA #PLAYERSUIT_SMALL
	STA <Player_Suit ; Player_Suit = PLAYERSUIT_SMALL

	JSR Player_Die	 ; Begin death sequence

	; This jumps the initial part of the death sequence
	LDA #$c0
	STA Event_Countdown ; Event_Countdown = $C0
	LDA #$02
	STA <Player_IsDying	; Player_IsDying = 2 (already dropped off screen)

	BNE PRG008_A472	 	; Jump (technically always) to PRG008_A472
```

The three codes change this to:

```
	LDA #$00
	STA <Player_YHi		; (was Player_Suit) Player_YHi = 0
	BIT Player_Die		; (was JSR; now does nothing)
	LDA #$c0
	STA Event_Countdown
	LDA #$02
	LDA <Player_IsDying	; (was STA) A = 0, since the Player isn't dying here
	BNE PRG008_A472		; Not taken, so normal play goes on at PRG008_A427
```

At the bottom of a normal level the Player's Y is `$1xx`, so
`Player_YHi` is 1. Setting it to 0 moves him exactly 256 pixels up. The
check fires when `Player_SpriteY` is `$C0`-`$CF`, so he ends up 49-64
pixels above the top of the screen. He keeps his suit, his X position and
his falling speed, so he drops back in from the top of the screen, right
above the pit.

`Player_Draw` (`prg029.asm`) later in the same frame sets
`Player_AboveTop` from `Player_YHi` minus `Level_VertScrollH`. It is
now nonzero, so on the next frames the pit check (`LDA Player_AboveTop /
BNE PRG008_A427`) is skipped until he is back on the screen.

It takes about a second to fall back to the bottom, so the player has
time to steer left or right onto ground.

Two codes were not possible. The `JSR Player_Die`, the
`Player_IsDying` store and the move each need their own byte. A
`PLA / PLA / RTS` would skip the `Player_IsDying` store, but none is at
an address a one-byte change to the `JSR` can reach. No routine in page
`$DA` of bank 0 moves the Player either.

**Evidence:** `smb3_original.nes` has `A9 00 85 ED 20 7C DA A9 C0 8D 10
05 A9 02 85 F1 D0 4B` at `$A415` in bank 8. `Player_YHi` is `$87`,
`Player_Suit` is `$ED` and `Player_IsDying` is `$F1`. No other bank
that loads at `$A000` has the same compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- This conflicts with "Falling into a pit bounces the Player back up".
  Both change `$A418`. The other two codes are the same in both.
- If he doesn't steer onto ground, he falls back into the pit and drops
  in from the top again. It is an endless loop, not a death.
- In vertical levels, `Player_YHi` is the screen number, so he moves to
  the top of the *level*, not the top of the screen, and falls down
  through the whole level.
- If a pit is ever reached with `Player_YHi` already 0, nothing moves
  him. He falls on below the screen without dying.
- `Event_Countdown` is still set to `$C0`, as in the bounce code. The pipe
  code and `Player_Die` set it themselves before they use it.
- Only the pit check changes. Enemies and the timer still kill the Player
  as usual.

## Enemies and spikes never hurt the Player (mercy invincibility all the time)

**Code:** `AVSIUOSZ`
**Raw form:** `D9D3:60:AD`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 13:51:22

Tags: `demo-ready` `play-start` `kids-collection` `curated`

**Mechanism:** After a hit, the game sets `Player_FlashInv` to `$71` and
the Player flashes until it counts down to zero. This is the game's
"mercy invincibility". Every hit goes through `Player_GetHurt`
(`prg000.asm`), which starts by returning early while the Player is
flashing:

```
Player_GetHurt:
	LDA Player_FlashInv		; ... flashing invincible ...
	ORA Player_Statue		; ... a statue ...
	ORA Player_StarInv		; ... invincible by star ...
	ORA Player_SuitLost		; ... just lost a power-up suit ...
	ORA <Player_HaltGame		; ... gameplay halted ...
	ORA Player_HaltTick		; ... Player halted ...
	ORA Player_DebugNoHitFlag	; ... debug invincibility flag is set (unused) ...
	BNE PRG000_D9B7	 ; ... then jump to PRG000_D9B7 (RTS)
```

The code changes the first byte, the `LDA` opcode (`$AD`), to `RTS`
(`$60`). `Player_GetHurt` now always returns at once, as it does while
the Player is flashing. Enemies, enemy shots, spike and muncher tiles,
and everything else that calls `Player_GetHurt` can no longer shrink or
kill the Player. It is the same as the unused `Player_DebugNoHitFlag`
being always set.

The Player does not flash. Only the "can't be hurt" part of mercy
invincibility is kept.

**Evidence:** `smb3_original.nes` has `AD 52 05 0D 7A 05 0D 53 05 ...` at
`$D9D3` in bank 0 (file offset `0x19E3`). `Player_GetHurt` is at
`$D9D3`. About 30 places call it or jump to it (objects in banks 1-5,
special objects in bank 7, Player tile checks in banks 8 and 9). No
other bank that loads at `$C000` has `$AD` at `$D9D3`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only one code, so it fits on a real Game Genie.
- These still kill the Player, as they do during normal mercy
  invincibility: pits, death tiles such as lava (`prg007.asm`
  `PRG007_A183` calls `Player_Die` directly), being crushed, and the
  timer running out.
- Stomping, kicking shells, tail attacks and Starman still work. Those
  paths don't go through `Player_GetHurt`.
- Real mercy invincibility also skips some object code before the hurt
  call (`PRG000_D355` checks `Player_FlashInv` first). With this code that
  code still runs. For example, an enemy with `OAT_BOUNCEOFFOTHERS`
  still turns to face away from the Player when it touches him, and the
  Player still can't be hurt by it.
- Special objects (hammers, fireballs and so on) check
  `Player_FlashInv` themselves and skip the Player while he flashes.
  That check was not changed, so the recovered wand and the Micro-Goomba
  still grab onto the Player as usual. Changing that check would also
  stop the Player from picking up the wand at the end of a castle.
- A Kuribo's Shoe, Tanooki suit or other power-up is never lost to a hit.

## Time running out doesn't kill the Player

**Code:** `AAXXLGEA`
**Raw form:** `A42B:00:80`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-17 13:52:06

Tags: `demo-ready` `play-start` `kids-collection` `curated`

**Mechanism:** Each frame, `Player_Update` (`prg008.asm`) checks the
timer. The game's unused debug mode (`Debug_Flag` = `$80`) skips the
check:

```
PRG008_A427:
	LDA Debug_Flag
	CMP #$80
	BEQ PRG008_A45A	 ; If debug mode is enabled, jump to PRG008_A45A

	LDA Level_TimerEn
	AND #$7f
	BNE PRG008_A472	 ; If the clock is disabled, jump to PRG008_A472

	LDA Level_TimerMSD
	ORA Level_TimerMid
	ORA Level_TimerLSD
	BNE PRG008_A45A	 ; If there's some sort of time left, jump to PRG008_A45A

	; TIME UP!
	JSR Player_Die	 ; Begin death sequence
```

The code changes `CMP #$80` to `CMP #$00`. In normal play
`Debug_Flag` is 0, so the `BEQ` is now always taken and the time-up
death never runs. `PRG008_A45A` is the same place the game goes when
there is time left, so play goes on as normal.

The timer itself still counts down. When it reaches 000,
`StatusBar_Fill_Time` (`prg026.asm`) sets all three digits to 0 and
it stays at 000.

**Evidence:** `smb3_original.nes` has `AD 60 01 C9 80 F0 2C` at `$A427`
in bank 8 (file offset `0x1043B` for the `$80`). No other bank that
loads at `$A000` has `$80` at `$A42B`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only one code, so it fits on a real Game Genie.
- The "hurry up" music still plays at 100. The timer stays at 000 after
  that.
- No time bonus is added at the end of a level that is finished at 000.
- If the debug menu is used to set `Debug_Flag` to `$80`, the time-up
  death comes back. The debug menu is not reachable in normal play.
- Only the time-up check changes. Enemies, pits and death tiles still
  kill the Player.

## Press Select to turn invincibility on and off

**Code:** `LPOGVPYX`
**Raw form:** `C916:1B:27`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 14:12:59

Tags: `interesting` `play-start`

**Mechanism:** This turns on an old debug routine that is still in the
ROM but never runs. `Objects_HandleScrollAndUpdate` (`prg000.asm`)
starts with a jump over it:

```
Objects_HandleScrollAndUpdate:
	JMP PRG000_C927	 ; Jump to PRG000_C927

PRG000_C918:
	.byte $35, $F0, $0C

; $C91B
	LDA <Pad_Input
	AND #PAD_SELECT
	BEQ PRG000_C927	 ; If Player is NOT pressing SELECT, jump to PRG000_C927

	; Toggle invincibility flag
	EOR Player_DebugNoHitFlag
	STA Player_DebugNoHitFlag

PRG000_C927:
```

The code changes the `JMP` target from `$C927` to `$C91B`, so the
routine runs every frame before the object update. Each press of Select
flips `Player_DebugNoHitFlag` between `$00` and `$20`.
`Player_GetHurt` still checks this flag and returns at once while it is
set, so nothing that calls `Player_GetHurt` can hurt the Player.

**Evidence:** `smb3_original.nes` has `4C 27 C9 35 F0 0C A5 18 29 20 F0
06 4D FF 7C 8D FF 7C` at `$C915` in bank 0 (file offset `0x926` for
the `$27`). `Pad_Input` is `$18` and `Player_DebugNoHitFlag` is
`$7CFF`. No other bank that loads at `$C000` has `$27` at `$C916`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only one code, so it fits on a real Game Genie.
- Nothing on screen shows whether invincibility is on. It starts off.
- `Player_DebugNoHitFlag` is cleared when a level loads (`prg005.asm`),
  so press Select again in each level.
- When it is on, it works like "Enemies and spikes never hurt the
  Player": pits, death tiles, being crushed and the timer still kill the
  Player. The Player does not flash.
- If used with "Press Select to cycle through all the power-ups", one
  Select press does both.

## Press Select to cycle through all the power-ups

**Code:** `AAXXKVLZ`
**Raw form:** `AE2C:00:2B`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-17 14:13:00

Tags: `demo-ready` `play-start` `kids-collection` `curated`

**Mechanism:** This turns on the debug mode power-up switch in
`Player_CommonGroundAnims` (`prg008.asm`) without debug mode:

```
PRG008_AE26:
	LDA Debug_Flag
	CMP #$80
	BNE PRG008_AE58	 ; If we're not in debug mode, jump to PRG008_AE58

	; DEBUG SUIT/POWER-UP SWITCH AND KURIBO TOGGLE

	LDA <Pad_Input
	AND #PAD_SELECT
	BEQ PRG008_AE58	 ; If Player is NOT pressing select, jump to PRG008_AE58

	LDA <Pad_Holding
	AND #(PAD_A | PAD_B)
	BNE PRG008_AE50	 ; If Player is holding A or B, jump to PRG008_AE50
```

The code sets the `BNE` offset to `$00`, so the branch goes to the
next instruction either way and the switch always runs:

- Select alone sets `Player_QueueSuit` to the next power-up: Small,
  Super, Fire, Raccoon, Frog, Tanooki, Hammer, then back to Small.
- Select while holding A or B turns Kuribo's Shoe on or off.

**Evidence:** `smb3_original.nes` has `AD 60 01 C9 80 D0 2B` at
`$AE26` in bank 8 (file offset `0x10E3C` for the `$2B`). No other
bank that loads at `$A000` has `$2B` at `$AE2C`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only one code, so it fits on a real Game Genie.
- `Player_CommonGroundAnims` runs in the normal (not climbing, not
  swimming) movement code, so this may not work while climbing or under
  water.
- The power-up only lasts for the level, as with a normal power-up. The
  map power-up is not changed.
- Holding A or B and pressing Select gives Kuribo's Shoe. It was only
  meant for testing, so the shoe may behave oddly in some places (pipes,
  water, level ends).
- If used with "Press Select to turn invincibility on and off", one
  Select press does both.

## Fly anywhere with the D-pad

**Codes:** `GGOXXSSX` + `XTOXUIVU` + `UGOXKITA` (use all three)
**Raw form:** `AD1A:4C:AD` + `AD1B:EA:B6` + `AD1C:C3:06`
(address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-17 14:13:01

Tags: `interesting` `play-start`

**Mechanism:** Bank 0 still has an old routine at `$C3EA` that nothing
calls. It looks like a debug "float around" mode (`prg000.asm`):

```
PRG000_C3E7:
	.byte $00, $30, -$30

; $C3EA 
	LDA <Pad_Holding
	AND #(PAD_LEFT | PAD_RIGHT)
	TAY		 ; Y = 1 or 2

	; Set Player X velocity directly??
	LDA PRG000_C3E7,Y
	STA <Player_XVel

	LDA <Pad_Holding
	LSR A
	LSR A
	AND #((PAD_UP | PAD_DOWN) >> 2)
	TAY		 ; Y = 1 or 2

	; Set Player Y velocity directly??
	LDA PRG000_C3E7,Y
	STA <Player_YVel

	RTS		 ; Return
```

It sets `Player_XVel` and `Player_YVel` straight from the D-pad: `$30`
for Right or Down, `-$30` for Left or Up, and 0 when no direction is
held.

The codes replace the last part of `Player_JumpFlyFlutter`
(`prg008.asm`), the uphill speed check at `PRG008_AD1A`, with
`JMP $C3EA`. Every ground movement mode (all suits and Kuribo's Shoe)
calls `Player_JumpFlyFlutter` once per frame, after left/right control,
jumping and gravity. The routine then sets the speeds last, and its
`RTS` returns from `Player_JumpFlyFlutter`. So gravity is canceled and
the Player moves in the direction the D-pad is held, and stops in the air
when it is let go.

**Evidence:** `smb3_original.nes` has `AD B6 06 F0 0F` (`LDA
Player_UphillSpeedIdx / BEQ`) at `$AD1A` in bank 8 (file offsets
`0x10D2A`-`0x10D2C`), and `00 30 D0 A5 17 29 03 A8 B9 E7 C3 85 BD ...
60` at `$C3E7` in bank 0. Bank 0 is at `$C000` during levels.
`Pad_Holding` is `$17`, `Player_XVel` is `$BD` and `Player_YVel`
is `$CF`. No other bank that loads at `$A000` has the same compare
bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Three codes, so it fits on a real Game Genie.
- Y velocity is only applied while `Player_InAir` is set. On the ground,
  Up does nothing. To take off, hold Up and press A (the jump sets
  `Player_InAir`), or walk off a ledge. Normal jumps no longer go up
  without Up held.
- Walls, floors and ceilings still stop the Player.
- `$30` is a little slower than full P-speed running (`$38`). There is no
  speed-up; the Player starts and stops at once.
- Swimming and climbing use their own code, so this does nothing under
  water or on vines.
- The uphill speed check that was replaced only limits jumps while
  walking up slopes. Its loss doesn't matter here, since the routine
  sets the Y speed anyway.
- Raccoon flight and tail flutter still run first, but the routine
  overrides their speed.
- The Player can fly above the top of the screen and off the level. Pits
  still kill if the Player flies down into one.

## Goombas are replaced by "lost" orange Cheep Cheeps

**Codes:** `SYEXGEZZ` + `XIEXIAXL` + `EEOXGAAE` (use all three)
**Raw form:** `A00C:F5:2A` + `A00D:D2:B2` + `A09C:88:00`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 15:01:13

**Mechanism:** When an object is first run, `ObjState_Initializing`
(`prg000.asm`) sets its state to Normal, sets
`Y = ObjGroupRel_Idx * 2`, and jumps through
`ObjectGroup_InitJumpTable`. The Goomba (`OBJ_GOOMBA`, `$72`) is entry 6
of group 3 (bank 4), so `Y` is `$0C`.

Bank 0 has code that changes an object into the ID in the low byte of its
`ObjectGroup_CollideJumpTable` entry, used when a stomped enemy turns
into another one (`PRG000_D2F5`):

```
	; Change to alternate ObjectID in low byte
	LDA ObjectGroup_CollideJumpTable,Y
	STA Level_ObjectID,X

	INC Objects_HitCount,X	 ; Give hit back (restore to zero)
	JMP PRG000_D31E	 ; Jump to PRG000_D31E

PRG000_D31E:
	; Halt vertical movement
	LDA #$00
	STA <Objects_YVel,X

	RTS		 ; Return
```

The first two codes change the Goomba's init pointer from
`ObjInit_GroundTroop` (`$B22A`) to `$D2F5`. The third code changes the
low byte of the Goomba's collide entry from `$00` to `$88`
(`OBJ_ORANGECHEEP`). So when a Goomba spawns, its ID becomes `$88` and it
runs as an orange Cheep Cheep from the next frame on.
`ObjInit_OrangeCheep` is just an `RTS`, so nothing is lost by not running
it.

**Evidence:** `smb3_original.nes` has `2A B2` at `$A00C` and `00 00` at
`$A09C` in bank 4, and `B9 90 A0 9D 71 06 FE F6 7C 4C 1E D3` at `$D2F5`
in bank 0. `Level_ObjectID` is `$0671`. Both objects ask for the same
sprite bank (`OPTS_SETPT6 | $4F`) and palette (`OA1_PAL3`), so the Cheep
Cheep's graphics are loaded wherever Goombas are.

The collide entry stays `$0088`, not `$08xx`, so it is not read as a
"change to this ID when stomped" value. Goombas don't have the
`Object_AttrFlags` bit 7 that makes the game jump to the collide entry, so
it is never used as an address either.

`$A09C:88:00` also matches bank 11. There, `$A09C` is slot 6 of
`W7_Obj_Ys` (World 7 map object Y positions). Slot 6 is `MAPOBJ_EMPTY` in
`W7_Obj_IDs`, and the map code skips empty slots before it reads a Y.
The N-Spade, White Toad House and coin ship set their own Y when they use a
free slot. So the change in bank 11 should not do anything. The other two
codes match no other bank.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Three codes, so it fits on a real Game Genie.
- The Cheep Cheep starts where the Goomba was and swims to the left, bobbing
  up and down, through walls and floors. It can't be stomped. It leaves when
  it goes off-screen.
- Only Goombas that start in the level (and pipe Goombas, if their pipe
  runs the init) are changed. A Paragoomba that loses its wings becomes a
  plain Goomba without running the init, so it stays a Goomba. So does the
  Goomba knocked out of a Kuribo's Shoe.
- Big Goombas, Paragoombas and Goombas in Kuribo's Shoe are not changed.
- `Objects_HitCount` is 1 instead of 0. The orange Cheep Cheep can't be
  stomped, so this shouldn't matter.

## Red Koopa Troopas are replaced by Charging Cheep Cheeps

**Codes:** `GYEZZEZX` + `KZEZLAXL` + `LLVXZGAE` (use all three)
**Raw form:** `A002:7C:2A` + `A003:A4:B2` + `A46A:3B:00`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 15:01:14

**Mechanism:** When an object is first run, `ObjState_Initializing`
(`prg000.asm`) sets its state to Normal (2), sets
`Y = ObjGroupRel_Idx * 2`, and jumps through
`ObjectGroup_InitJumpTable`. The Red Koopa Troopa (`OBJ_REDTROOPA`,
`$6D`) is entry 1 of group 3 (bank 4), so `Y` is 2.

`ObjInit_HammerBro` (`prg004.asm`) has code that turns a Hammer Bro into
the enemy of a map battle and runs the init again:

```
	LDY Map_EnterViaID	; Y = Map_EnterViaID

	LDA BattleEnemy_ByEnterID,Y
	CMP #OBJ_HAMMERBRO
	BEQ PRG004_A490	 	; If battling a hammer brother, jump to PRG004_A490

	; Not a hammer brother...

	STA Level_ObjectID,X	; Store the enemy ID

	DEC Objects_State,X	 ; Take state back to 0 

	RTS		 ; Return
```

The first two codes change the Red Koopa's init pointer from
`ObjInit_GroundTroop` (`$B22A`) to `$A47C`, the `LDA
BattleEnemy_ByEnterID,Y`, skipping the `LDY`. With `Y` = 2, it reads
`BattleEnemy_ByEnterID+2`. That is the "Airship" row, which is `$00` and
unused. The third code sets it to `$3B` (`OBJ_CHARGINGCHEEPCHEEP`). The
Koopa's ID becomes `$3B`, and the `DEC` puts its state back to 1, so on
the next frame the game runs the Charging Cheep Cheep's own init
(`ObjInit_ChasingFish`) as if it had just spawned.

**Evidence:** `smb3_original.nes` has `2A B2` at `$A002` and `00 00 00 81
82 86 87 00 00 00 AD 66 05 C9 07 D0 16 AC 1E 00 B9 68 A4 C9 81 F0 0D 9D 71
06 DE 61 06 60` at `$A468` in bank 4. No other bank that loads at
`$A000` has the same compare bytes.

The real Hammer Bro only reads this row when `Level_Event` is 7 (a
treasure-box battle) and the Player came in through the airship
(`Map_EnterViaID` = 2), which doesn't happen.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Tested by the user: works.
- Three codes, so it fits on a real Game Genie.
- `ObjInit_ChasingFish` moves the fish to the bottom of the screen
  (`Level_VertScroll` + 160) and makes it hop up, so the Cheep Cheep
  appears at the bottom of the screen below where the Koopa was, not at the
  Koopa's spot. It then swims back and forth there and jumps at the Player
  when he comes close, as in 1-? water levels. It moves through floors and
  walls.
- When it goes too far from the Player it uses the Boss Bass style respawn
  (`Fish_Repawn`).
- The two objects use the same sprite bank (`OPTS_SETPT6 | $4F`), so
  the graphics are loaded. The palette changes to the Cheep Cheep's own
  (`OA1_PAL1`) when its init runs.
- Only Red Koopas that start in the level are changed. A red Paratroopa
  that loses its wings becomes a Red Koopa without running the init, so it
  stays a Red Koopa. Big Red Koopas use a different init and are not
  changed.

## Green Koopa Paratroopas are replaced by Pile Driver Micro-Goombas

**Codes:** `SYEZGEZZ` + `XIEZIAXL` + `LVOZGEGV` (use all three)
**Raw form:** `A004:F5:2A` + `A005:D2:B2` + `A094:6B:6C`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 15:08:11

**Mechanism:** This works like "Goombas are replaced by "lost" orange
Cheep Cheeps". The hopping Green Paratroopa (`OBJ_PARATROOPAGREENHOP`,
`$6E`) is entry 2 of group 3 (bank 4), so `Y` is 4 when its init runs.

Its collide entry is already a "change to this object when stomped" value:

```
	.word OCSPECIAL_KILLCHANGETO | OBJ_GREENTROOPA	; Object $6E - OBJ_PARATROOPAGREENHOP
```

When the Player stomps it, `PRG000_D2F5` (`prg000.asm`) stores the low
byte (`OBJ_GREENTROOPA`) in `Level_ObjectID`, so it loses its wings:

```
	; Change to alternate ObjectID in low byte
	LDA ObjectGroup_CollideJumpTable,Y
	STA Level_ObjectID,X

	INC Objects_HitCount,X	 ; Give hit back (restore to zero)
	JMP PRG000_D31E	 ; Jump to PRG000_D31E
```

The first two codes change the Paratroopa's init pointer from
`ObjInit_GroundTroop` (`$B22A`) to `$D2F5`, so this runs as soon as it
spawns. The third code changes the low byte from `$6C` (`OBJ_GREENTROOPA`)
to `$6B` (`OBJ_PILEDRIVER`). So the Paratroopa becomes a Pile Driver
before it is ever drawn. `ObjInit_PileDriver` is just an `RTS`, so nothing
is lost by not running it.

The game picks the object's bank from `Level_ObjectID` every frame
(`ObjGroupRel_Idx` and `PAGE_A000`), so the Pile Driver's code in bank 3
runs from the next frame on.

**Evidence:** `smb3_original.nes` has `2A B2` at `$A004` and `6C 08` at
`$A094` in bank 4, and `B9 90 A0 9D 71 06 FE F6 7C 4C 1E D3` at `$D2F5`
in bank 0. No other bank that loads at `$A000` has the same compare bytes.
Both objects ask for the same sprite bank (`OPTS_SETPT6 | $4F`).

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Three codes, so it fits on a real Game Genie.
- Only the hopping Green Paratroopa is changed. Red Paratroopas that fly up
  and down (`OBJ_FLYINGREDPARATROOPA`) and plain Green Koopas are not.
- The Pile Driver keeps the Paratroopa's palette (`OA1_PAL2`) instead of
  its own (`OA1_PAL3`), because the palette is set before the init runs.
  It may look a different color than a normal Pile Driver.
- `Objects_HitCount` is 1 instead of 0, so if the Pile Driver can be
  killed by a stomp, it may take two stomps.

## Micro-Goomba-dropping Paragoombas are replaced by Bullet Bills

**Codes:** `SYOZAEZZ` + `XIOZPAXL` + `ANXZAAZN` (use all three)
**Raw form:** `A010:F5:2A` + `A011:D2:B2` + `A0A0:78:72`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 15:18:58

**Mechanism:** This works like "Green Koopa Paratroopas are replaced by
Pile Driver Micro-Goombas". The Paragoomba that drops Micro-Goombas
(`OBJ_PARAGOOMBAWITHMICROS`, `$74`) is entry 8 of group 3 (bank 4), so
`Y` is `$10` when its init runs. Its collide entry is a "change to this
object when stomped" value:

```
	.word OCSPECIAL_KILLCHANGETO | OBJ_GOOMBA	; Object $74 - OBJ_PARAGOOMBAWITHMICROS
```

`PRG000_D2F5` (`prg000.asm`) stores the low byte of that entry in
`Level_ObjectID`:

```
	; Change to alternate ObjectID in low byte
	LDA ObjectGroup_CollideJumpTable,Y
	STA Level_ObjectID,X

	INC Objects_HitCount,X	 ; Give hit back (restore to zero)
	JMP PRG000_D31E	 ; Jump to PRG000_D31E
```

The first two codes change the Paragoomba's init pointer from
`ObjInit_GroundTroop` (`$B22A`) to `$D2F5`, so this runs as soon as it
spawns. The third code changes the low byte from `$72` (`OBJ_GOOMBA`) to
`$78` (`OBJ_BULLETBILL`). The Bullet Bill's own init is
`ObjInit_Set3DoNothing`, which is just an `RTS`.

**Evidence:** `smb3_original.nes` has `2A B2` at `$A010` and `72 08` at
`$A0A0` in bank 4, and `B9 90 A0 9D 71 06 FE F6 7C 4C 1E D3` at `$D2F5`
in bank 0. No other bank that loads at `$A000` has the same compare bytes.
Both objects use `OPTS_SETPT6 | $4F` and `OA1_PAL3`.

A code for the plain hopping Paragoomba (`OBJ_PARAGOOMBA`, `$73`) was not
possible. Its third code would be `A09E:78:72`, and bank 19 has `$72` at
`$A09E` too (a pattern of tile `$9E` in `Tile_Layout_TS5_TS11_TS13`).

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Three codes, so it fits on a real Game Genie.
- The Bullet Bill does not move. `ObjNorm_BulletBill` only applies
  `Objects_XVel`, which a cannon normally sets. `Level_PrepareNewObject`
  sets it to 0, so the Bullet Bill hangs in the air where the Paragoomba
  appeared, until it goes off-screen. It still hurts the Player.
- `Objects_HitCount` is 1 instead of 0, so it may take two stomps to
  knock it down.
- Plain hopping Paragoombas are not changed.
- The third code alone is "Stomping a Micro-Goomba-dropping Paragoomba turns it into a Bullet Bill".

## Stomping a Micro-Goomba-dropping Paragoomba turns it into a Bullet Bill

**Code:** `ANXZAAZN`
**Raw form:** `A0A0:78:72`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 15:18:59

**Mechanism:** When the Player stomps the Paragoomba that drops
Micro-Goombas (`OBJ_PARAGOOMBAWITHMICROS`, `$74`), its collide entry
(`OCSPECIAL_KILLCHANGETO | OBJ_GOOMBA`) makes `PRG000_D2F5`
(`prg000.asm`) change it into a plain Goomba:

```
	; Change to alternate ObjectID in low byte
	LDA ObjectGroup_CollideJumpTable,Y
	STA Level_ObjectID,X

	INC Objects_HitCount,X	 ; Give hit back (restore to zero)
	JMP PRG000_D31E	 ; Jump to PRG000_D31E

PRG000_D31E:
	; Halt vertical movement
	LDA #$00
	STA <Objects_YVel,X

	RTS		 ; Return
```

The code changes the low byte from `$72` (`OBJ_GOOMBA`) to `$78`
(`OBJ_BULLETBILL`). So the stomped Paragoomba becomes a Bullet Bill. Its
vertical speed is cleared, but its `Objects_XVel` is kept, and
`ObjNorm_BulletBill` keeps moving it at that speed.

**Evidence:** `smb3_original.nes` has `72 08` at `$A0A0` in bank 4. No
other bank that loads at `$A000` has the same compare byte. The same code
for the plain hopping Paragoomba (`$A09E`) would also change bank 19 (see
"Micro-Goomba-dropping Paragoombas are replaced by Bullet Bills").

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The Bullet Bill flies at the Paragoomba's speed, not a cannon Bullet
  Bill's. While it flutters down that is `$08` (`ObjNorm_ParaGoombaBomber`),
  about a third of a fired Bullet Bill's speed. When it is stomped while
  hovering, it may be slower or not move at all. It faces the way the
  Paragoomba faced.
- Bumping it from below (`Object_HandleBumpUnderneath`) uses the same
  entry, so that may also make a Bullet Bill.
- Plain hopping Paragoombas still lose their wings as usual.
- Also part of "Micro-Goomba-dropping Paragoombas are replaced by Bullet Bills". Using both is the same as using that one.

## Green Koopa Troopas are replaced by Bullet Bills

**Codes:** `GYEZAEZX` + `KZEZPAXL` + `AYVXAGAE` (use all three)
**Raw form:** `A000:7C:2A` + `A001:A4:B2` + `A468:78:00`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 15:22:22

**Mechanism:** This works like "Red Koopa Troopas are replaced by Charging
Cheep Cheeps". The Green Koopa Troopa (`OBJ_GREENTROOPA`, `$6C`) is entry
0 of group 3, so `Y` is 0 when its init runs.

The first two codes change its init pointer from `ObjInit_GroundTroop`
(`$B22A`) to `$A47C` in `ObjInit_HammerBro`:

```
	LDA BattleEnemy_ByEnterID,Y
	CMP #OBJ_HAMMERBRO
	BEQ PRG004_A490	 	; If battling a hammer brother, jump to PRG004_A490

	STA Level_ObjectID,X	; Store the enemy ID

	DEC Objects_State,X	 ; Take state back to 0 

	RTS		 ; Return
```

With `Y` = 0 it reads `BattleEnemy_ByEnterID+0`, the "(Unused, standard
level entry)" row. The third code changes it from `$00` to `$78`
(`OBJ_BULLETBILL`). The Koopa's ID becomes `$78` and its state goes back
to 1, so the Bullet Bill's own init (`ObjInit_Set3DoNothing`, just an
`RTS`) runs on the next frame.

**Evidence:** `smb3_original.nes` has `2A B2` at `$A000` and `00` at
`$A468` in bank 4. No other bank that loads at `$A000` has the same
compare bytes.

The collide-table route (as in "Goombas are replaced by "lost" orange Cheep
Cheeps") was not possible here. The Green Koopa's collide entry is at
`$A090`, and bank 28 has `LDA #$00` at `$A08F` (it clears the sound
queues), so that code would also have broken sound.

The real Hammer Bro only reads row 0 when `Level_Event` is 7 (a
treasure-box battle) and `Map_EnterViaID` is 0. Treasure-box battles are
entered from a map battle object, so this shouldn't happen.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Three codes, so it fits on a real Game Genie.
- The Bullet Bill does not move. `ObjNorm_BulletBill` only applies
  `Objects_XVel`, which a cannon normally sets. `Level_PrepareNewObject`
  sets it to 0, so the Bullet Bill hangs in the air where the Koopa
  appeared, until it goes off-screen. It still hurts the Player and can be
  stomped.
- Only Green Koopas that start in the level are changed. Green
  Paratroopas that lose their wings become Green Koopas without running
  the init, so they stay Koopas. Big Green Koopas are not changed.
- Can be used with "Red Koopa Troopas are replaced by Charging Cheep
  Cheeps". Both use the same code in `ObjInit_HammerBro` but change
  different bytes.

## World 1 levels load the enemies of another map spot

**Code:** `TIELAGZI`
**Raw form:** `B400:56:52`
(address:new-byte:compare-byte, bank 12)
**Recorded:** 2026-10-17 15:31:52

**Mechanism:** This is the SMB3 version of the SMW code "Every level is
populated with the next level's enemies".

When the Player enters a map spot, `Map_PrepareLevel` (`prg012.asm`) finds
the spot's index in the world's map lists, then gets the spot's object
(enemy) list pointer from that world's `Wn_ObjSets` table:

```
	; Temp_Var6/5 form an address to Map_ObjSets
	LDA Map_ObjSets,Y	 
	STA <Temp_Var5		 
	LDA Map_ObjSets+1,Y	 
	STA <Temp_Var6		 
	...
	; Store address of object set into Level_ObjPtr_AddrL/H and Level_ObjPtrOrig_AddrL/H
	LDA [Temp_Var5],Y	 
	STA <Level_ObjPtr_AddrL
	STA Level_ObjPtrOrig_AddrL	 
	INY		 
	LDA [Temp_Var5],Y	 
	STA <Level_ObjPtr_AddrH		 
	STA Level_ObjPtrOrig_AddrH	 
```

`Map_ObjSets` (`$B400`) starts with the address of `W1_ObjSets`,
`$B452`. The code changes the low byte to `$56`, so the World 1 table
starts two entries (4 bytes) later. Each World 1 spot gets the object list
of the spot two places after it in `W1_ObjSets`. Its terrain, tileset,
music and everything else stay its own. Other worlds are not changed.

The SMW code could just move the read one level along. That doesn't work
in SMB3, because each `Wn_ObjSets` table also holds values that are not
object lists: `$0700` for a Toad House (the high byte is the treasure type)
and `$0001` for a Spade game. A level that gets one of those copies RAM as
its object list. Of the shifts checked (1 to 12 entries either way), two
entries forward is the one where the most World 1 levels get a real list.

With this code (from `PRG/maps/World1S.asm` and `PRG/prg006.asm`):

- 1-1 gets 1-2's enemies.
- 1-2 gets the World 1 Hammer Bro battle list (`W1HBO`): a Hammer Bro, and
  a treasure box that appears when he is beaten.
- 1-3 gets the "Unused Level 6" list (`W1UNO`).
- 1-4 gets the Hammer Bro battle list.
- The fortress (`W10FO`) gets the Hammer Bro battle list.
- 1-5 gets 1-6's enemies.
- 1-6 reads past the end of `W1_ObjSets`, into the first word of
  `W1_LevelLayout` (`$BB82`). `LevelLoad_CopyObjectList` runs with
  bank 26 at `$A000`, and bank 26 has `$FF` at `$BB83`, so the list is
  empty: 1-6 has no enemies.

**Evidence:** `smb3_original.nes` has `52 B4` at `$B400` in bank 12, and
`W1_ObjSets` at `$B452` holds `$C527` (1-1), `$C72B` (`W1HBO`),
`$C6AA` (1-2) and so on, then `$BB82` just past the end. No other bank
that loads at `$A000` has `$52` at `$B400`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- The graphics are probably wrong for many of the moved enemies, because
  the level's sprite graphics bank is not changed.
- Enemies are placed by column and row for a different level, so they can
  be inside walls, in the air or below the floor.
- Both Toad Houses get a real object list pointer as their treasure type
  (`$D3` and `$C7`, from `W10FO` and `W1HBO`). Toad and the boxes still
  appear (the game always uses `TOADO` for a Toad House), but opening a
  box reads `ToadHouse_ItemOff` far past its end, so the item you get is
  garbage and may break the inventory. Don't open the boxes.
- The Spade game spot gets `$0700` instead of `$0001`. It was not checked
  whether the Spade game uses this value.
- The World 1 map Hammer Bro's battle uses the list of the spot two places
  after the spot he is on. That may be a level's enemies with no Hammer
  Bro, which would leave the Player stuck in the battle with no treasure
  box. Some of his spots also get `$0700` or `$0001` (RAM read as an
  object list). Avoid fighting him.
- The castle's own spot (`KINGO`) gets the Hammer Bro battle list, but
  the King's room after the airship is set up by a different path
  (`Player_FallToKing`), so it is not changed.

## Paragoombas are replaced by Spinies

**Codes:** `GYEXTEZX` + `KZEXYAXL` + `PYNZTGYA` (use all three)
**Raw form:** `A00E:7C:2A` + `A00F:A4:B2` + `A476:71:07`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 15:44:30

**Mechanism:** This works like "Red Koopa Troopas are replaced by Charging
Cheep Cheeps". The hopping Paragoomba (`OBJ_PARAGOOMBA`, `$73`) is entry
7 of group 3, so `Y` is `$0E` when its init runs.

The first two codes change its init pointer from `ObjInit_GroundTroop`
(`$B22A`) to `$A47C` in `ObjInit_HammerBro`:

```
ObjInit_HammerBro:
	LDA Level_Event
	CMP #$07
	BNE PRG004_A48F	 ; If we're not getting a treasure box (i.e. a Hammer Bro battle), jump to PRG004_A48F (RTS)

	LDY Map_EnterViaID	; Y = Map_EnterViaID

	LDA BattleEnemy_ByEnterID,Y	; <- $A47C
	CMP #OBJ_HAMMERBRO
	BEQ PRG004_A490	 	; If battling a hammer brother, jump to PRG004_A490

	STA Level_ObjectID,X	; Store the enemy ID

	DEC Objects_State,X	 ; Take state back to 0 

	RTS		 ; Return
```

`BattleEnemy_ByEnterID` is at `$A468` and has 10 entries, so
`BattleEnemy_ByEnterID+$0E` is `$A476`. That is the operand of
`CMP #$07` above. The third code changes it from `$07` to `$71`
(`OBJ_SPINY`). The Paragoomba's ID becomes `$71` and its state goes back
to 1, so the Spiny's own init (`ObjInit_GroundTroop`) runs on the next
frame.

**Evidence:** `smb3_original.nes` has `2A B2` at `$A00E` and `AD 66 05 C9
07` at `$A472` in bank 4. No other bank that loads at `$A000` has the
same compare bytes.

The collide-table route (as in "Goombas are replaced by "lost" orange Cheep
Cheeps") was not possible. The Paragoomba's collide entry is at `$A09E`,
and bank 19 has `$72` there too (a pattern of tile `$9E` in
`Tile_Layout_TS5_TS11_TS13`).

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Three codes, so it fits on a real Game Genie.
- Side effect: `Level_Event` is never `$71`, so `ObjInit_HammerBro`
  always returns at the `BNE`. On the map, Boomerang Bro, Fire Bro and
  Heavy Bro battles give plain Hammer Bros instead. Normal Hammer Bro
  battles and Hammer Bros in levels are not changed.
- The Spiny appears where the Paragoomba was, falls to the ground and
  walks. It can't be stomped.
- The Spiny asks for a different sprite bank (`OPTS_SETPT5 | $0B`) than
  the Paragoomba (`OPTS_SETPT6 | $4F`). Other enemies on screen that use
  the fifth sprite bank may look wrong while it is there.
- Only plain hopping Paragoombas are changed. Paragoombas that drop
  Micro-Goombas are not, and Paragoombas that lose their wings already
  became Spinies, so the "becomes a Goomba when stomped" step never
  happens.
- Can be used with the other `ObjInit_HammerBro` codes ("Red Koopa
  Troopas are replaced by Charging Cheep Cheeps", "Green Koopa Troopas are
  replaced by Bullet Bills"). They change different bytes.

## Cheep Cheeps jump out of the bottom of the screen in every level

**Codes:** `AEOUNUYP` + `SNXLULEX` + `KUXLKLUS` (use all three)
**Raw form:** `BB9F:00:1F` + `BBA3:FD:A0` + `BBA4:BC:D3`
(address:new-byte:compare-byte, bank 5)
**Recorded:** 2026-10-17 15:55:00

Tags: `demo-ready` `play-start` `curated`

**Mechanism:** SMB3's version of SMB1's Cheep Cheep frenzy is a "level
event". A level starts one by placing `OBJ_CHEEPCHEEPBEGIN` (`$B4`), which
sets `Level_Event` to 1. Every frame, gameplay calls `LevelEvent_Do`:

```
LevelEvent_Do:
	LDA Level_Event	
	BEQ PRG005_BBBF	 ; If no level event, jump to PRG005_BBBF (RTS)
	JSR DynJump	 ; Dynamic jump based on Level_Event... 
	
	; THESE MUST FOLLOW DynJump FOR THE DYNAMIC JUMP TO WORK!! 

	.word LevelEvent_DoNothing	; 0 - Do nothing (not used!)
	.word LevelEvent_CheepCheep	; 1 - Cheep Cheep attack
```

The first code changes the `BEQ` offset from `$1F` to `$00`, so when
`Level_Event` is 0 the branch goes to the next instruction and the
`DynJump` runs with index 0. The other two codes change jump table entry 0
from `LevelEvent_DoNothing` (`$D3A0`) to `LevelEvent_CheepCheep`
(`$BCFD`). So in any level with no event, the Cheep Cheep frenzy runs.

`LevelEvent_CheepCheep` tries to spawn every 32 frames (`Level_NoStopCnt
& $1F`) while fewer than 3 `OBJ_JUMPINGCHEEPCHEEP` are alive. Each one
starts at the bottom of the screen (`Level_VertScroll` + `$C0`) at a random
X offset, with a random X velocity and a Y velocity of `-$48`.

A one-code version is not possible. `LevelEvent_CheepCheep` is `$15D`
bytes past the `BEQ`, out of branch range, and there's no `JMP` or `JSR`
to it within range to branch to.

**Evidence:** `smb3_original.nes` has `AD 66 05 F0 1F 20 99 FE A0 D3 FD
BC` at `$BB9B` in bank 5. No other bank that loads at `$A000` has the
same compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Three codes, so it fits on a real Game Genie.
- Levels that set their own event (Spike Cheeps, Para-Beetles, Lakitu
  fleeing, clouds, wooden platforms, treasure box) run that event instead,
  with no Cheep Cheeps. Levels that already have the Cheep Cheep frenzy
  are not changed.
- An `OBJ_CANCELEVENT` sets `Level_Event` to 8, not 0, so the Cheep
  Cheeps stop for the rest of that level.
- The Jumping Cheep Cheep's graphics may not be loaded in levels that
  don't have Cheep Cheeps, so it may look wrong there.
- The Cheep Cheeps take object slots, so with 3 of them out other enemies
  may not spawn.

## Cheep Cheep frenzies throw Podoboos instead

**Codes:** `VPOLOITN` + `VPELSITN` (use both)
**Raw form:** `BD11:9E:76` + `BD05:9E:76`
(address:new-byte:compare-byte, bank 5)
**Recorded:** 2026-10-17 16:21:42

**Mechanism:** `LevelEvent_CheepCheep` (`Level_Event` = 1) runs the
Cheep Cheep frenzy, where Cheep Cheeps jump out of the bottom of the
screen. It uses `OBJ_JUMPINGCHEEPCHEEP` (`$76`) twice:

```
LevelEvent_CheepCheep:	
	LDA Level_NoStopCnt
	AND #$1f	 ; Cap 0 - 31
	BNE PRG005_BD53	 ; If not zero, jump to PRG005_BD53 (RTS)

	LDA #OBJ_JUMPINGCHEEPCHEEP	; <- $BD05
	JSR Level_CountNotDeadObjs
	CPY #$03
	BGE PRG005_BD53	 ; If there are already at least 3 Jumping Cheep Cheeps, jump to PRG005_BD53 (RT)S

	JSR Level_SpawnObj	 ; Spawn new object (Note: If no slots free, does not return)

	; Set the Cheep Cheep's object ID
	LDA #OBJ_JUMPINGCHEEPCHEEP	; <- $BD11
	STA Level_ObjectID,X
```

The first code changes the spawned object to `OBJ_PODOBOO` (`$9E`). The
second changes the count check to match, so there are still at most 3 at
a time. The spawn position and velocities are unchanged.

Unlike ground enemies such as the Koopa Troopa, `ObjNorm_Podoboo` never
checks the level while it is going up. It applies its own gravity
(`YVel` += 2 per frame, up to `$70`) and on the way down only checks for
`TILE2_LAVATOP`. So each
Podoboo starts at the bottom of the screen with the frenzy's Y velocity
of `-$48`, rises through the floor, and falls back down through it.

**Evidence:** `smb3_original.nes` has `A9 76` at `$BD04` and at `$BD10`
in bank 5. No other bank that loads at `$A000` has `$76` at `$BD05` or
`$BD11`. In 1-1 with the every-level codes, a Podoboo (`Level_ObjectID` = `$9E`) was seen
rising with `Objects_YVel` `$C4`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Only checked in RAM. Not yet played.
- Two codes, so it fits on a real Game Genie. In the original game the
  frenzy only runs in 4-2 and the third World 8 Hand Trap. Add the codes
  from "Cheep Cheeps jump out of the bottom of the screen in every level"
  to get Podoboos in every level (five codes, emulator only).
- `ObjNorm_Podoboo` never applies X velocity, so the Podoboos go straight
  up and down instead of arcing sideways.
- They can spawn right on top of the Player at the start of a level.
  In one 1-1 test the Player was hit almost as soon as the level began.
- `Level_SpawnObj` sets `OBJSTATE_NORMAL` directly, so
  `ObjInit_Podoboo` never runs. Its "home" height (`Objects_Var5` and
  `Objects_Var4`) stays 0. Landing on lava sets its next jump from
  `Podoboo_YVelByHeight` by its height above that home, which reads past
  the end of the table. Expect odd jumps in lava levels.
- Its palette (`Objects_SprAttr`) is left over from whatever last used
  that object slot, so it may be the wrong color.
- The Podoboo asks for a different sprite bank (`OPTS_SETPT5 | $12`).
  Other enemies on screen that use the fifth sprite bank may look wrong
  while one is there.

## Stomping a green Koopa Paratroopa turns it into a 1-Up

**Codes:** `LEUXAEGV` + `LEOZGEGV`
**Raw form:** `A0B8:0B:6C` + `A094:0B:6C`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 16:30:24

**Mechanism:** This works like "Stomping a Micro-Goomba-dropping
Paragoomba turns it into a Bullet Bill". Both green Paratroopas have
`OCSPECIAL_KILLCHANGETO | OBJ_GREENTROOPA` as their collide entry, so a
stomp changes them into a Green Koopa Troopa:

```
	.word OCSPECIAL_KILLCHANGETO | OBJ_GREENTROOPA	; Object $6E - OBJ_PARATROOPAGREENHOP
	...
	.word OCSPECIAL_KILLCHANGETO | OBJ_GREENTROOPA	; Object $80 - OBJ_FLYINGGREENPARATROOPA
```

The codes change the low byte of each from `$6C` (`OBJ_GREENTROOPA`) to
`$0B` (`OBJ_POWERUP_1UP`). The first code is for the flying green
Paratroopa (`$80`, entry at `$A0B8`), the second for the hopping one
(`$6E`, entry at `$A094`). `PRG000_D2F5` (`prg000.asm`) stores the
new ID in `Level_ObjectID` and clears `Objects_YVel`. The object keeps
its position, `Objects_XVel` and palette, and `ObjNorm_PUp1UpMush` runs
from the next frame on.

**Evidence:** `smb3_original.nes` has `6C 08` at `$A094` and at
`$A0B8` in bank 4. No other bank has `$6C` at `$A0B8`. Bank 6 also has
`$6C` at `$A094`, but bank 6 holds object layouts (`PRG/prg006.asm`),
which are mapped at `$C000`, so the second code shouldn't reach it.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Either code works alone, for just that kind of Paratroopa.
- `ObjInit_PUpMush` never runs. The 1-Up keeps the Paratroopa's palette,
  which is palette 2 for both, the same as the 1-Up's own (`OA1_PAL2`).
- The 1-Up keeps the Paratroopa's X velocity. `ObjNorm_PUp1UpMush` only
  bounces it off walls, so it may move at the Paratroopa's speed instead
  of a normal mushroom's.
- `PowerUp_DoRaise` uses `Objects_Timer`. The flying Paratroopa sets that
  timer to `$30` each time it reaches its top speed, so a 1-Up made from it soon
  after that may play the "rising out of a block" step first.
- Bumping the Paratroopa from below uses the same entry, so that may also
  make a 1-Up.
- Red Paratroopas are not changed.

## Stomping a Goomba turns it into a 1-Up

**Codes:** `LEOXGAAE` + `AEOXIAAE`
**Raw form:** `A09C:0B:00` + `A09D:08:00`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 16:35:48

**Mechanism:** The Goomba's (`OBJ_GOOMBA`, `$72`) collide entry is
`$0000`:

```
	.word $0000					; Object $72 - OBJ_GOOMBA
```

The codes make it `OCSPECIAL_KILLCHANGETO | OBJ_POWERUP_1UP` (`$080B`).
A normal stomp goes to `PRG000_D2B4` (`prg000.asm`) for every enemy, and
that checks the high byte for `$08xx` after giving the points, whatever
the enemy's attributes are:

```
	LDA ObjectGroup_CollideJumpTable+1,Y	 ; Get the CollideJumpTable upper byte
	AND #%11111000
	CMP #%00001000
	BNE PRG000_D301	 ; If the upper byte is not ($08xx - $0Fxx), jump to PRG000_D301

	; SPECIAL JUMP TABLE VALUE ($08xx - $0Fxx): Change to alternate object

	; Change to alternate ObjectID in low byte
	LDA ObjectGroup_CollideJumpTable,Y
	STA Level_ObjectID,X

	INC Objects_HitCount,X	 ; Give hit back (restore to zero)
	JMP PRG000_D31E	 ; Jump to PRG000_D31E
```

It returns before the squash code, so the Goomba becomes a 1-Up instead of
being squashed. `Object_DoCollision` jumps to the entry as an address, but
only for objects that call `Object_HitTestRespond`. The Goomba's original
`$0000` would crash there too, so the Goomba presumably never goes that
way.

**Evidence:** `smb3_original.nes` has `00 00` at `$A09C` in bank 4. Bank
11 also has `$00` at both addresses and is mapped at `$A000` on the
world map. Those bytes are the Y positions of World 7's map objects 6 and
7 (`PRG/maps/World7OY.asm`), and both of those are `MAPOBJ_EMPTY`
(`World7O.asm`). `nes_gg.py` reports `ALSO MATCHES banks [11]` for both
codes. No clean code was found: both original bytes are `$00`, and the
code that reads them is shared by every enemy. Bank 30 also has `$00` at
`$A09D`, but it's fixed at `$8000`, so it isn't affected.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Both codes are needed.
- The codes also change the Y position of two empty World 7 map object
  slots from `$00` to `$0B` and `$08`. They're empty, so this probably
  does nothing.
- The 1-Up keeps the Goomba's palette (`OA1_PAL3`), so it will probably
  be the wrong color.
- The 1-Up keeps the Goomba's X velocity and isn't initialized, as in
  "Stomping a green Koopa Paratroopa turns it into a 1-Up".
- Paragoombas turn into Goombas when stomped, so stomping one twice
  should give a 1-Up.
- Stomping a Goomba in Kuribo's Shoe or as a statue changes it to a 1-Up
  and then sets `OBJSTATE_SHELLED`, which may look odd or break.
- Hitting a Goomba from below or with a shell doesn't use this entry,
  since the Goomba doesn't have `OA3_DIESHELLED`.

## Stomping a red Koopa Troopa turns it into a 1-Up

**Codes:** `LEOZZAAE` + `AEOZLAAE`
**Raw form:** `A092:0B:00` + `A093:08:00`
(address:new-byte:compare-byte, bank 4)
**Recorded:** 2026-10-17 16:40:36

Tags: `demo-ready` `play-start` `kids-collection` `curated`

**Mechanism:** This works like "Stomping a Goomba turns it into a 1-Up".
The red Koopa Troopa's (`OBJ_REDTROOPA`, `$6D`) collide entry is
`$0000`:

```
	.word $0000					; Object $6D - OBJ_REDTROOPA
```

The codes make it `OCSPECIAL_KILLCHANGETO | OBJ_POWERUP_1UP` (`$080B`).
A stomp goes to `PRG000_D2B4` (`prg000.asm`), which changes the ID to
`OBJ_POWERUP_1UP` and returns before `Object_SetShellState`, so the Koopa
becomes a 1-Up instead of a shell.

**Evidence:** `smb3_original.nes` has `00 00` at `$A092` in bank 4.
`nes_gg.py` reports `ALSO MATCHES banks [11]` for both codes. In bank 11
(the world map) those bytes are the Y positions of World 6's map objects 5
and 6 (`PRG/maps/World6OY.asm`), and both are `MAPOBJ_EMPTY`
(`World6O.asm`). No clean code was found: both original bytes are `$00`,
and the code that reads them is shared by every enemy.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Both codes are needed.
- The codes also change the Y position of two empty World 6 map object
  slots from `$00` to `$0B` and `$08`. They're empty, so this probably
  does nothing.
- The 1-Up keeps the red Koopa's palette (`OA1_PAL1`). That's the Super
  Mushroom's palette (`OBJ_POWERUP_MUSHROOM`), so the 1-Up will probably
  look like a Super Mushroom. It still gives a 1-Up.
- The 1-Up keeps the Koopa's X velocity and isn't initialized, as in
  "Stomping a green Koopa Paratroopa turns it into a 1-Up".
- Red Paratroopas turn into red Koopa Troopas when stomped, so stomping
  one twice should give a 1-Up.
- The red Koopa has `OA3_DIESHELLED`, so hitting it from below (and
  other kills that go through `PRG000_D19E`) also changes it to a 1-Up,
  but then puts it in `OBJSTATE_SHELLED`. That may look odd or break.
- Green Koopa Troopas are not changed.

## Stomped enemies go into a shell that can be kicked

**Codes:** `AAESALPA` + `AAEGELAP`
**Raw form:** `D308:00:01` + `CB00:00:10`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 16:45:39

Tags: `demo-ready` `play-start` `curated`

**Mechanism:** After a stomp, `PRG000_D301` (`prg000.asm`) kills enemies
that have `OA2_NOSHELLORSQUASH`, and puts the rest into
`OBJSTATE_SHELLED`:

```
PRG000_D301:
	LDY ObjGroupRel_Idx	 ; Y = object group relative index
	LDA ObjectGroup_Attributes2,Y	 ; Get object attributes set 2
	AND #OA2_NOSHELLORSQUASH	 
	BEQ PRG000_D323	 	; If OA2_NOSHELLORSQUASH is NOT set (shelled enemy), jump to PRG000_D323
```

Then `ObjState_Shelled` sends enemies with `OA3_SQUASH` (like Goombas) to
`OBJSTATE_SQUASHED`:

```
ObjState_Shelled:
	LDY ObjGroupRel_Idx	; Y = object's group-relative index

	LDA ObjectGroup_Attributes3,Y 
	AND #OA3_SQUASH
	BEQ PRG000_CB10	 ; If OA3_SQUASH is NOT set (stomp does not "shell" this object), jump to PRG000_CB10  
```

Both codes change the `AND` value to `#$00`, so both `BEQ`s always
jump. Every enemy that reaches the normal stomp code goes into the shell
state and stays there. Touching an enemy in the shell state kicks it
(`PRG000_D2A2`), like a Koopa shell.

**Evidence:** `smb3_original.nes` has `29 01` at `$D307` and `29 10` at
`$CAFF` in bank 0. `nes_gg.py` finds no other bank with the same bytes.
The other `AND #OA3_SQUASH` at `$D28E` (statue and Kuribo's Shoe stomps)
is not changed.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Each code works alone. The first is for enemies that normally die when
  stomped. The second is for enemies that normally get squashed, like
  Goombas.
- The graphics will probably be wrong. `Object_DrawShelled` draws frame 2
  of the enemy's own sprites, mirrored. For a Koopa that's the shell; for
  other enemies it's whatever their frame 2 is, which may be garbage.
- A shelled enemy wakes up after a while (`Object_ShellDoWakeUp`), like a
  Koopa does.
- Enemies that hurt the Player when stomped (`OA3_NOTSTOMPABLE`), that
  ignore stomps (`OA2_STOMPDONTCARE`), or that handle stomps in their own
  code are not changed.
- Stomping as a statue or in Kuribo's Shoe still kills enemies that
  would normally go into a shell. Enemies that would normally be squashed
  are put in the shell state there too, so with the second code they also
  stay in a shell.
- The 1-Up stomp entries change the enemy's ID before this code runs, so
  with those codes on, the 1-Up still appears instead of a shell.

## Stomping any enemy turns it into a 1-Up

**Codes:** `AENIGXGA` + `SXNIIXOU` + `XONIYZEZ` (use all three)
**Raw form:** `D2F4:00:0C` + `D2F5:AD:B9` + `D2F7:92:A0`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 17:55:19

Tags: `demo-ready` `play-start` `kids-collection`

**Mechanism:** After a stomp gives its points, `PRG000_D2B4`
(`prg000.asm`) checks the enemy's `ObjectGroup_CollideJumpTable` entry
for `OCSPECIAL_KILLCHANGETO`. Only Paratroopas and Paragoombas have it:

```
	LDA ObjectGroup_CollideJumpTable+1,Y	 ; Get the CollideJumpTable upper byte
	AND #%11111000
	CMP #%00001000
	BNE PRG000_D301	 ; If the upper byte is not ($08xx - $0Fxx), jump to PRG000_D301

	; SPECIAL JUMP TABLE VALUE ($08xx - $0Fxx): Change to alternate object

	; Change to alternate ObjectID in low byte
	LDA ObjectGroup_CollideJumpTable,Y
	STA Level_ObjectID,X

	INC Objects_HitCount,X	 ; Give hit back (restore to zero)
	JMP PRG000_D31E	 ; Jump to PRG000_D31E
```

`AENIGXGA` sets the `BNE` offset to `$00`, so every stomped enemy takes
the "change to alternate object" path. The other two codes change
`LDA ObjectGroup_CollideJumpTable,Y` (`B9 90 A0`) to `LDA $9290`
(`AD 90 92`). `$9290` is in bank 30, which is always mapped at `$8000`,
and holds `$0B` (`OBJ_POWERUP_1UP`). It's the offset byte of
`BEQ PRG030_929C` after `CMP #$06` in `prg030.asm`. So every stomped
enemy's ID becomes `OBJ_POWERUP_1UP`, and `PRG000_D31E` clears its Y
velocity.

**Evidence:** `smb3_original.nes` has `D0 0C B9 90 A0` at `$D2F3` in bank
0, and `$0B` at `$9290` in bank 30. `nes_gg.py` finds no other bank
with the same compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Only enemies that go through the normal stomp code are changed.
  Enemies that hurt the Player when stomped (`OA3_NOTSTOMPABLE`), that
  ignore stomps (`OA2_STOMPDONTCARE`), or that handle stomps in their own
  code (bosses, for example) are not changed. An enemy that takes more
  than one stomp only changes on the last one.
- Stomping an enemy that is already in a shell kicks the shell as before.
- Each 1-Up keeps the enemy's palette, so many will be the wrong color.
  It also keeps the enemy's position and X velocity, and isn't
  initialized, as in "Stomping a green Koopa Paratroopa turns it into a
  1-Up".
- Stomping as a statue or in Kuribo's Shoe also makes a 1-Up, but then
  sets `OBJSTATE_KILLED` (so the 1-Up falls off the screen) or, for
  enemies that would be squashed, `OBJSTATE_SHELLED`.
- The points for the stomp are still given.
- This makes the other 1-Up stomp entries do nothing extra. It also
  stops "Stomped enemies go into a shell that can be kicked" from
  working, because the stomp code returns before reaching its shell
  code.

## Stomping any enemy turns it into a Goomba

**Codes:** `AENIGXGA` + `SXNIIXOU` + `NENITZEP` + `ESNIYZEZ` (use all four)
**Raw form:** `D2F4:00:0C` + `D2F5:AD:B9` + `D2F6:87:90` + `D2F7:D0:A0`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 18:08:44

**Mechanism:** Same as "Stomping any enemy turns it into a 1-Up", but the
last two codes point the load at `$D087` instead of `$9290`. `$D087` is
in bank 0, the bank this code runs from, so it's always mapped here. It
holds `$72` (`OBJ_GOOMBA`): the operand of `CMP #OBJ_GOOMBA` after `PRG000_D07E`. So
`LDA ObjectGroup_CollideJumpTable,Y` becomes `LDA $D087`, and every
stomped enemy's ID becomes `OBJ_GOOMBA`.

No byte at `$xx90` in the always-mapped banks holds `$72`, so this needs
four codes instead of three.

**Evidence:** `smb3_original.nes` has `D0 0C B9 90 A0` at `$D2F3` in bank
0, and `$72` at `$D087`. `nes_gg.py` finds no other bank with the same
compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Four codes: more than a real Game Genie holds.
- Goombas use the common enemy graphics (`OPTS_SETPT6 | $4F`), so they
  should look right in most levels. The palette comes from the old enemy,
  so the color may be wrong.
- A stomped Goomba turns into a Goomba again, so Goombas can't be
  squashed with this on.
- Only enemies that go through the normal stomp code are changed, as in
  "Stomping any enemy turns it into a 1-Up". Statue and Kuribo's Shoe
  stomps change the enemy and then kill it or put it in a shell.
- The new enemy keeps the old one's position, flip and palette.
  `ObjInit_GroundTroop` never runs, but it only sets the flip to face
  the Player. `ObjNorm_GroundTroop` sets the walking speed from the flip
  every frame, so it should walk normally.
- Can't be used with the other "Stomping any enemy" entries (they change
  the same bytes), and stops "Stomped enemies go into a shell that can be
  kicked" from working.

## Stomping any enemy turns it into a Green Koopa Troopa

**Codes:** `AENIGXGA` + `SXNIIXOU` + `TENITZEP` + `EKNIYZEZ` (use all four)
**Raw form:** `D2F4:00:0C` + `D2F5:AD:B9` + `D2F6:06:90` + `D2F7:C0:A0`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 18:08:44

**Mechanism:** Same as "Stomping any enemy turns it into a 1-Up", but the
last two codes point the load at `$C006` instead of `$9290`. `$C006` is
in bank 0, the bank this code runs from, so it's always mapped here. It
holds `$6C` (`OBJ_GREENTROOPA`): the low byte of `.word Level_SlopeQuadC0` at the start of `prg000.asm`. So
`LDA ObjectGroup_CollideJumpTable,Y` becomes `LDA $C006`, and every
stomped enemy's ID becomes `OBJ_GREENTROOPA`.

No byte at `$xx90` in the always-mapped banks holds `$6C`, so this needs
four codes instead of three.

**Evidence:** `smb3_original.nes` has `D0 0C B9 90 A0` at `$D2F3` in bank
0, and `$6C` at `$C006`. `nes_gg.py` finds no other bank with the same
compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Four codes: more than a real Game Genie holds.
- Koopas use the common enemy graphics (`OPTS_SETPT6 | $4F`), so they
  should look right in most levels. The palette comes from the old enemy,
  so the color may be wrong.
- A stomped Koopa turns into a Koopa again, so Koopas can't be stomped
  into shells with this on.
- Only enemies that go through the normal stomp code are changed, as in
  "Stomping any enemy turns it into a 1-Up". Statue and Kuribo's Shoe
  stomps change the enemy and then kill it or put it in a shell.
- The new enemy keeps the old one's position, flip and palette.
  `ObjInit_GroundTroop` never runs, but it only sets the flip to face
  the Player. `ObjNorm_GroundTroop` sets the walking speed from the flip
  every frame, so it should walk normally.
- Can't be used with the other "Stomping any enemy" entries (they change
  the same bytes), and stops "Stomped enemies go into a shell that can be
  kicked" from working.

## Stomping any enemy turns it into a Bullet Bill

**Codes:** `AENIGXGA` + `SXNIIXOU` + `EVNIYZEX` (use all three)
**Raw form:** `D2F4:00:0C` + `D2F5:AD:B9` + `D2F7:E8:A0`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 18:08:44

**Mechanism:** Same as "Stomping any enemy turns it into a 1-Up", but the
third code points the load at `$E890` instead of `$9290`. `$E890` is in
bank 31, which is always mapped there, and holds `$78` (`OBJ_BULLETBILL`). So
`LDA ObjectGroup_CollideJumpTable,Y` becomes `LDA $E890`, and every
stomped enemy's ID becomes `OBJ_BULLETBILL`.

**Evidence:** `smb3_original.nes` has `D0 0C B9 90 A0` at `$D2F3` in bank
0, and `$78` at `$E890` in bank 31. `nes_gg.py` finds no other bank with
the same compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Bullet Bills use the common enemy graphics (`OPTS_SETPT6 | $4F`), so
  they should look right in most levels. The palette comes from the old
  enemy, so the color may be wrong.
- `ObjNorm_BulletBill` only applies the X velocity it already has. A
  Bullet Bill made from a walking enemy flies at that enemy's walking
  speed, and one made from an enemy that wasn't moving sideways stays in
  place.
- Stomping the Bullet Bill gives another Bullet Bill.
- Only enemies that go through the normal stomp code are changed, as in
  "Stomping any enemy turns it into a 1-Up". Statue and Kuribo's Shoe
  stomps change the enemy and then kill it or put it in a shell.
- The new enemy keeps the old one's position, X velocity, flip and
  `Objects_Var` values, and its init routine never runs.
- Can't be used with the other "Stomping any enemy" entries (they change
  the same bytes), and stops "Stomped enemies go into a shell that can be
  kicked" from working.

## Stomping any enemy turns it into a homing Bullet Bill

**Codes:** `AENIGXGA` + `SXNIIXOU` + `NENIYZEZ` (use all three)
**Raw form:** `D2F4:00:0C` + `D2F5:AD:B9` + `D2F7:87:A0`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 18:08:44

**Mechanism:** Same as "Stomping any enemy turns it into a 1-Up", but the
third code points the load at `$8790` instead of `$9290`. `$8790` is in
bank 30, which is always mapped there, and holds `$79` (`OBJ_BULLETBILLHOMING`). So
`LDA ObjectGroup_CollideJumpTable,Y` becomes `LDA $8790`, and every
stomped enemy's ID becomes `OBJ_BULLETBILLHOMING`.

**Evidence:** `smb3_original.nes` has `D0 0C B9 90 A0` at `$D2F3` in bank
0, and `$79` at `$8790` in bank 30. `nes_gg.py` finds no other bank with
the same compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Homing Bullet Bills use the common enemy graphics (`OPTS_SETPT6 |
  $4F`), so they should look right in most levels. `ObjNorm_BulletBill`
  sets their palette every frame (color cycling), so the color should be
  right too.
- The homing code uses `Objects_Var3` and `Objects_Var4`, which keep the
  old enemy's values, so it may wait, or never turn, depending on the
  enemy.
- Stomping the Bullet Bill gives another homing Bullet Bill.
- Only enemies that go through the normal stomp code are changed, as in
  "Stomping any enemy turns it into a 1-Up". Statue and Kuribo's Shoe
  stomps change the enemy and then kill it or put it in a shell.
- The new enemy keeps the old one's position, X velocity, flip and
  `Objects_Var` values, and its init routine never runs.
- Can't be used with the other "Stomping any enemy" entries (they change
  the same bytes), and stops "Stomped enemies go into a shell that can be
  kicked" from working.

## Stomping any enemy turns it into a flying green Koopa Paratroopa

**Codes:** `AENIGXGA` + `SXNIIXOU` + `ONNIYZEX` (use all three)
**Raw form:** `D2F4:00:0C` + `D2F5:AD:B9` + `D2F7:F9:A0`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 18:08:44

**Mechanism:** Same as "Stomping any enemy turns it into a 1-Up", but the
third code points the load at `$F990` instead of `$9290`. `$F990` is in
bank 31, which is always mapped there, and holds `$80` (`OBJ_FLYINGGREENPARATROOPA`). So
`LDA ObjectGroup_CollideJumpTable,Y` becomes `LDA $F990`, and every
stomped enemy's ID becomes `OBJ_FLYINGGREENPARATROOPA`.

**Evidence:** `smb3_original.nes` has `D0 0C B9 90 A0` at `$D2F3` in bank
0, and `$80` at `$F990` in bank 31. `nes_gg.py` finds no other bank with
the same compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Flying green Paratroopas use the common enemy graphics
  (`OPTS_SETPT6 | $4F`), so they should look right in most levels. The
  palette comes from the old enemy, so the color may be wrong.
- Stomping the Paratroopa gives another flying Paratroopa, so it never
  loses its wings.
- Only enemies that go through the normal stomp code are changed, as in
  "Stomping any enemy turns it into a 1-Up". Statue and Kuribo's Shoe
  stomps change the enemy and then kill it or put it in a shell.
- The new enemy keeps the old one's position, X velocity, flip and
  `Objects_Var` values, and its init routine never runs.
- Can't be used with the other "Stomping any enemy" entries (they change
  the same bytes), and stops "Stomped enemies go into a shell that can be
  kicked" from working.

## Stomping any enemy turns it into a Starman (or a super suit)

**Codes:** `AENIGXGA` + `SXNIIXOU` + `OONITZEP` + `KKNIYZEX` (use all four)
**Raw form:** `D2F4:00:0C` + `D2F5:AD:B9` + `D2F6:91:90` + `D2F7:CC:A0`
(address:new-byte:compare-byte, bank 0)
**Recorded:** 2026-10-17 18:09:55

**Mechanism:** Same as "Stomping any enemy turns it into a 1-Up", but the
last two codes point the load at `$CC91` instead of `$9290`. `$CC91` is
in bank 0, the bank this code runs from, so it's always mapped here. It
holds `$0C` (`OBJ_POWERUP_STARMAN`): the operand of `LDA #$0c` after
`PRG000_CC8D`. So every stomped enemy's ID becomes `OBJ_POWERUP_STARMAN`.

`OBJ_POWERUP_STARMAN` is also the super suits. `ObjHit_StarOrSuit`
(`prg001.asm`) picks which one by `Objects_Frame`:

```
	; Different "frames" of the "Starman" power up include the super
	; suits; Tanooki, Frog, and Hammer, in order
PUp_StarOrSuitFrames:	.byte $00, $05, $04, $06	; Star, Tanooki, Frog, Hammer
```

`ObjInit_StarOrSuit` normally sets the frame, but it never runs here, so
the object keeps the enemy's animation frame.

No byte at `$xx90` in the always-mapped banks holds `$0C`, so this needs
four codes instead of three.

**Evidence:** `smb3_original.nes` has `D0 0C B9 90 A0` at `$D2F3` in bank
0, and `A9 0C` at `$CC90`. `nes_gg.py` finds no other bank with the same
compare bytes.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Four codes: more than a real Game Genie holds.
- What you get depends on the enemy's animation frame when stomped: frame
  0 gives a Starman, 1 a Tanooki Suit, 2 a Frog Suit, 3 a Hammer Suit.
  Walking enemies switch between frames 0 and 1, so it will often be a
  Tanooki Suit instead of a Starman. The object is drawn with the same
  frame, so it should look like what it gives.
- An enemy on frame 4 or higher reads past `PUp_StarOrSuitFrames` into
  code, which gives a bad `Player_Suit` value. That will probably glitch
  or crash.
- Power-ups don't change the sprite bank (`OPTS_NOCHANGE`), so the Starman
  should look right. The suit tiles may not be loaded in every level.
- The palette comes from the enemy, not `Star_Palettes`, so the color may
  be wrong until the Starman starts flashing.
- Only enemies that go through the normal stomp code are changed, as in
  "Stomping any enemy turns it into a 1-Up". It keeps the enemy's
  position and X velocity.
- Can't be used with the other "Stomping any enemy" entries (they change
  the same bytes), and stops "Stomped enemies go into a shell that can be
  kicked" from working.

## Hold B to tail-attack again and again with no delay

**Code:** `YOKUIEAP`
**Raw form:** `B0CD:17:18` (address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-25 18:58:10

**Mechanism:** `Player_TailAttackAnim` (`PRG/prg008.asm`) starts a tail
attack only on the frame B is first pressed:

```
	BIT <Pad_Input
	BVC PRG008_B0E2	 ; If Player is NOT pressing 'B', jump to PRG008_B0E2

	LDA Player_TailAttack
	BNE PRG008_B0E2	 ; If Player is performing tail attack, jump to PRG008_B0E2

	LDA #$12
	STA Player_TailAttack	 ; Player_TailAttack = $12
```

So Raccoon or Tanooki Mario has to let go of B and press it again for
each swing. The code changes the operand from `Pad_Input` (`$18`, new
presses) to `Pad_Holding` (`$17`, buttons held). Now a new swing starts
on the first frame after the last one ends, for as long as B is held.
`GndMov_Leaf` and `GndMov_Tanooki` both call this routine.

**Evidence:** `smb3_original.nes` has `A5 17 29 04 D0 3D 24 18 50 12` at
`$B0C6` in bank 8 (`Player_TailAttackAnim`). `nes_gg.py` finds no other
bank that loads at `$A000` with `$18` at `$B0CD`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- B is also the run and carry button, so Mario swings his tail the whole
  time he runs or carries a shell.
- While a tail attack is on, `Player_DoSpecialTiles` is skipped. So while
  B is held, Mario can't enter pipes, conveyors don't move him, and
  Muncher tiles don't hurt him. Let go of B to go down a pipe.
- Fire Mario has his own code, "Hold B to throw fireballs with no
  delay" (`YPVZZXAP`).

## Hold B to throw fireballs with no delay

**Code:** `YPVZZXAP`
**Raw form:** `A262:17:18` (address:new-byte:compare-byte, bank 7)
**Recorded:** 2026-10-25 20:35:39

**Mechanism:** `Gameplay_UpdateAndDrawMisc` (`PRG/prg007.asm`) throws a
fireball only on the frame B is first pressed:

```
	BIT <Pad_Input
	BVC PRG007_A268	 ; If Player is NOT pressing B, jump to PRG007_A268

	JSR PlayerProj_ThrowWeapon	 ; Player throws weapon, whatever's appropriate
```

So Fire Mario has to let go of B and press it again for each fireball.
There is no timer. The code changes the operand from `Pad_Input` (`$18`,
new presses) to `Pad_Holding` (`$17`, buttons held). Now
`PlayerProj_ThrowWeapon` runs on every frame B is held. It throws a new
fireball as soon as one of its two slots (`PlayerProj_ID`) is free, so a
fireball comes out each time an old one hits something or leaves the
screen.

**Evidence:** `smb3_original.nes` has `24 18 50 03 20 AF A2` at `$A261`
in bank 7 (file offset `$E271`). `nes_gg.py` finds one other bank with
`$18` at `$A262`: bank 24 (`TAndK_WaitPlayerButtonA`, `LDA <Pad_Input`,
`A5 18`). See the caveats.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Hammer Mario uses the same check, so he throws hammers with no delay
  too. No single byte can change only the fireballs.
- B is also the run and carry button, so Fire Mario throws fireballs the
  whole time he runs or carries a shell.
- Still two fireballs at a time.
- The code also changes bank 24. In the King's room, `King_DoDialog`
  waits for A in `TAndK_WaitPlayerButtonA`, which then reads the held
  buttons. So holding A closes the message as soon as it has finished,
  with no new press.

## Raccoon Mario throws a fireball with each tail attack

**Code:** `LAKXZZTA`
**Raw form:** `A24A:03:06` (address:new-byte:compare-byte, bank 7)
**Optional, Tanooki Mario instead:** `IAKXZZTA` (`A24A:05:06`)
**Recorded:** 2026-10-25 20:42:24

**Mechanism:** `Gameplay_UpdateAndDrawMisc` (`PRG/prg007.asm`) lets
only Hammer and Fire Mario throw:

```
	LDA <Player_Suit
	CMP #PLAYERSUIT_HAMMER
	BEQ PRG007_A251	 ; If Player is wearing a Hammer Suit, jump to PRG007_A251

	CMP #PLAYERSUIT_FIRE
	BNE PRG007_A268	 ; If Player is not Fire, jump to PRG007_A268
```

The code changes the first `CMP` from `PLAYERSUIT_HAMMER` (6) to
`PLAYERSUIT_RACCOON` (3). Now a press of B by Raccoon Mario goes on to
`PlayerProj_ThrowWeapon`. That routine throws a hammer only for the
Hammer Suit, so Raccoon Mario throws a fireball. In the same frame,
`Player_TailAttackAnim` starts the tail attack, which also starts on a
new press of B. The ground and air movement code for Raccoon Mario
doesn't call `Player_ShootAnim`, so the fireball pose doesn't replace
the tail swing. The optional code uses `PLAYERSUIT_TANOOKI` (5) instead.

**Evidence:** `smb3_original.nes` has `A5 ED C9 06 F0 04 C9 02 D0 17` at
`$A247` in bank 7 (file offset `$E257`), and `C9 06 F0 04 C9 02 D0`
appears only once in the ROM. `nes_gg.py` finds no other bank with `$06`
at `$A24A`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Hammer Mario can't throw hammers. The test for his suit is gone.
- The code and the optional code change the same byte. Use one or the
  other.
- Fire Mario is not changed.
- A fireball comes out on every new press of B with a free slot, even
  while a tail attack is still going, so a press can throw a fireball
  with no new tail swing. Still two fireballs at a time.
- Swimming Raccoon Mario throws fireballs too.
- Fire Mario doesn't load any extra sprite tiles for the fireball (only
  the Hammer Suit does), so the fireball should look right. This was not
  checked.
- With "Hold B to throw fireballs with no delay" (`YPVZZXAP`), Raccoon
  Mario throws fireballs the whole time B is held.

## Hold A to jump again every time Mario lands

**Code:** `YPUZOKAP`
**Raw form:** `AC31:17:18` (address:new-byte:compare-byte, bank 8)
**Recorded:** 2026-10-27 20:20:01

**Mechanism:** This is the SMB3 version of the SMW code "Hold B to jump
again every time Mario lands". `Player_JumpFlyFlutter`
(`PRG/prg008.asm`) starts a jump only on the frame A is first pressed:

```
PRG008_AC30:

	LDA <Pad_Input
	AND #PAD_A
	STA <Temp_Var1	 ; Temp_Var1 = $80 if Player is pressing 'A', otherwise 0
	BEQ PRG008_AC9E	 ; If Player is NOT pressing 'A', jump to PRG008_AC9E
```

So the player has to let go of A and press it again for each jump. The
code changes the operand from `Pad_Input` (`$18`, new presses) to
`Pad_Holding` (`$17`, buttons held). Now, while A is held, Mario jumps
again on the first frame he is back on the ground. A is still held
during the jump, so every jump is a full-height jump.

The same `Temp_Var1` also starts the Raccoon and Tanooki tail wag:
`LDY <Temp_Var1` sets `Player_WagCount` to `$10` when it is not 0. Now
`Player_WagCount` is set again on every frame A is held in the air. So
Raccoon and Tanooki Mario float down slowly while A is held. With flight
time left (`Player_FlyTime`), they keep rising until it runs out.

**Evidence:** `smb3_original.nes` has `AD 1F 04 F0 03 CE 1F 04 A5 18 29 80
85 00 F0 66` at `$AC28` in bank 8 (`Player_JumpFlyFlutter`), so the
`Pad_Input` operand `$18` is at `$AC31`. `nes_gg.py` finds no other bank
that loads at `$A000` with `$18` at `$AC31`.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Not tested.
- Swimming has its own code (`Player_SwimV`) and still needs a new press
  for each stroke.
- With "Jump again in mid-air" (`AAKZEKII`), a new jump starts on every
  frame A is held, so Mario shoots straight up.

## All music plays with the pluck instrument

**Code:** `OXETYNPX`
**Raw form:** `E787:A9:29`
(address:new-byte:compare-byte, bank 31, fixed)
**Recorded:** 2026-11-01 21:07:59

Tags: `demo-ready` `play-start` `curated`

**Optional, a swelling instrument instead:** `OXETYNPX` + `AKEVAYAY`
(`E787:A9:29` + `E788:40:70`)

**Optional, a trembling instrument instead:** `OXETYNPX` + `ASEVAYAY`
(`E787:A9:29` + `E788:50:70`)

**Mechanism:** This is the SMB3 version of the SMB2 code "All music
plays with the strings instrument". Every frame, `Music_Sq1Track` and
`Music_Sq2Track` (`PRG/prg031.asm`) call `Music_PatchGetCTL` with the
note's patch in X. It turns the patch into an index into
`Music_PatchTableLong` and `Music_PatchTableShort`:

```
Music_PatchGetCTL:
	PHA		 ; Save A
	TXA
	AND #$70
	LSR A
	LSR A
	LSR A
	TAX
```

The code changes the `AND` opcode (`$29`) to `LDA #` (`$A9`), so the
same operand loads `$70` as a fixed patch, whatever the note's patch is.
`$70` gives index `$0E`, `PatL8` and `PatS8` (`PRG/prg028.asm`).
Their bytes are the same as SMB2's "pluck" instrument: 50%, 25% and
12.5% duty at full volume, then a fast decay to a quiet 25% tone. The
optional codes also change the operand: `$40` gives `PatL5`/`PatS5`
(25% duty that swells up and fades), and `$50` gives `PatL6`/`PatS6`
(25% duty whose volume rises and drops four times per long note).

The overworld song uses mostly patches `$00` and `$10` (`PatL1`, a
steady 50% tone, and `PatL2`, a 25% tone that decays), so the change is
heard from the start of World 1-1.

**Evidence:** `smb3_original.nes` has `48 8A 29 70 4A 4A 4A AA` at
`$E785` (`Music_PatchGetCTL`), and `Music_PatchTableLong` is at `$E765`
with 8 entries. Bank 31 is fixed at `$E000`, so no other bank can
match. The only callers are the two square track routines.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Tested by the user: works, as a patched ROM. The optional codes were
  not tested.
- Mesen2 applies any 8-letter Game Genie code for `$E000`-`$FFFF` as
  `$00`, so in Mesen2 this code writes `BRK` at `$E787` instead. That
  changes the music differently and glitches the map tiles. The raw cheat
  `E787:A9:29` and the patched ROM work correctly in Mesen2.
- The triangle bass, the drums, the DPCM samples and the sound effects
  don't use patches and don't change.

## Every level song plays the next song in the list

**Codes:** `AUEVPUYZ` + `GUEVYULU` + `AKOVALYK`
**Raw form:** `E389:30:2F` + `E38F:3C:3B` + `E398:48:47`
(address:new-byte:compare-byte, bank 31, fixed)
**Recorded:** 2026-11-01 21:40:12

Tags: `demo-ready` `play-start` `curated` `video-ready`

**Mechanism:** This is the SMB3 version of the SMB2 code "Every song
plays the next song in the list". `SndMus_QueueSet2B` (`PRG/prg031.asm`)
starts the level songs (`MUS2B_OVERWORLD` `$10` to
`MUS2B_WORLD8LETTER` `$C0`). It shifts the song ID down to an index
from 1 to 12 in Y and reads three tables with it:

```
	TAY		 		; Y = A
	LDA Music_Set2B_Starts-1,Y
	STA Music_Start
	LDA Music_Set2B_Ends-1,Y
	ADD #$02
	STA Music_End
	LDA Music_Set2B_Loops-1,Y
	STA Music_Loop
```

Each code adds 1 to the low byte of one table address, so all three
tables are read one entry later. Each song plays the segments of the
next song in the `MUS2B_` list:

| Song | Plays instead |
|---|---|
| Overworld | Underground |
| Underground | Water |
| Water | Fortress |
| Fortress | Boss |
| Boss | Airship |
| Airship | Hammer Bros. battle |
| Hammer Bros. battle | Toad House |
| Toad House | Athletic (overworld 2) |
| Athletic | P-switch (the Toad House song) |
| P-switch | Bowser |
| Bowser | World 8 letter |

The world map songs, invincibility and the other Set 2A songs, and the
Set 1 songs (death, game over and others) use other tables and don't
change.

**Evidence:** `smb3_original.nes` has `B9 2F B5` at `$E388`,
`B9 3B B5` at `$E38E` and `B9 47 B5` at `$E397`. The tables in bank 28
are `Music_Set2B_Starts` at `$B530`, `Music_Set2B_Ends` at `$B53C` and
`Music_Set2B_Loops` at `$B548`, each 12 bytes. Bank 31 is fixed at
`$E000`, so no other bank can match.

**Revisions:** PRG1 only. Other revisions were not checked.

**Caveats:**

- Tested by the user: works, as a patched ROM.
- The World 8 letter song (`$C0`) reads past the end of each table: the
  start is `$06` (the first byte of the ends table), the end is `$01`
  and the loop is `$A2`. The end comes before the start, so that song
  probably plays the wrong segments or garbage.
- Mesen2 applies any 8-letter Game Genie code for `$E000`-`$FFFF` as
  `$00`, so in Mesen2 these codes write the wrong bytes. Use the raw
  cheats or a patched ROM there.
