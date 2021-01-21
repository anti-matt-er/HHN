# Poker Standard Notation (PSN) Specification - Request for Comments

## General

### Purpose
The idea is to create a standard notation for the Texas Hold'em variant of Poker, similar to the widely-used *Algebraic Notation* of Chess, created for *Passion Poker* but hopefully to be widely adopted as a standard.

Currently, there is no widely-used standard and every network uses their own proprietary format.

To facilitate this, this specification and its implementation is hereby released open source under the **MIT License** so that any individual, network or other entity may adopt and improve upon it.

### Criticisms
I've heard it said that the reason such a standard does not yet exist is that it would be more useful to bots and cheaters than legitimate players, however it is my firm belief that this is an example of [Security Through Obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity) and therefore is *not* a valid reason for one to be created.

Putting it another way: it is entirely trivial for a malicious entity to create software that can parse the many proprietary formats used by poker networks and this is not a barrier to them, especially when the potential profit is so high. Proprietary formats therefore do not have a bearing on these entities; they will operate regardless.

On the other hand, there is a clear benefit to the community at large. Hand histories are already shared widely across forums and other websites and are used to by players to improve their game and share interesting stories (such as "Bad Beats"), having a standard notation would help facilitate this so that everyone is using the same language.

### Goals
*PSN* is designed to be:
- **Concise**: It should convey a hand/game using the fewest bytes whilst representing as much information as possible.
- **Readable**: It should be clear and unambiguous for a person to read and quickly understand the actions performed in a game and its outcome.
- **Parsable**: It should be as trivial as possible for any software to parse the hand without sacrificing human readability.
- **Flexible**: It should accomodate any style of play / type of game while not compromising the other goals, and should provide multiple methods of representing the same types of information where the choice comes down to personal preference (for example, using Positions instead of Seat Numbers).
- **Adoptable**: It should be easy for any individual to learn to read and write this notation, to the extent that *Algebraic Notation* is for chess. It should also be able to represent any hand history that any other Poker network uses, so that they may easily be converted to this standard.

## Examples
The best way to learn *PSN* is to look at some examples.

Consider the 2 following examples, these represent the same game:

**Example 1**
```
NLH 200 BTN 3/7/10
4=6B 5=10B 6=12B
#P 6:R3B 4:RA 5:C 6:C
#F[7h2h5s] 5:C 6:RA 5:X
#T[5d]
#R[4d]
#S 4=P 4[QdAh]PA+AQ 6[JdAc]PA+AJ
```

**Example 2**
```
NLH 100|200 7
BTN=3210 SB=6B BB=2000 UTG=12B U+1=8B199 U+2=9B25 CO=4045 
#P UTG:R3B U+1:X U+2:X CO:X BTN:X SB:RA BB:C UTG:CA
#F[7h 2h 5s] BB:C UTG:RA BB:X UTG[Qd Ah] BB[Jd Ac]
#T[5d]
#R[4d]
#S UTG=3850 UTG[Qd Ah]PA+AQ
```

Example 1 (and therefore Example 2) can be broken down as follows:
```
NLH 200 BTN 3/7/10	;No Limit Hold'em, 7 out of 10 possible seats taken, ante is 25, small blind is 100 big blind is 200, Seat #3 is the BTN
4=6B				;SB has 1200 chips
5=10B				;BB	has 2000 chips
6=12B				;UTG has 2400 chips
#P					;Preflop, start of the game
6:R3B				;UTG raises 3 big blinds
4:RA				;SB raises 6 big blinds and is all-in
5:C					;BB calls 6 big blinds
6:C					;UTG calls 6 big blinds
#F[7h2h5s]			;Flop goes 7h 2h 5s
5:C					;BB checks
6:RA				;UTG raises 6 big blinds and is all-in
5:X					;BB folds
#T[5d]				;Turn goes 5d, board is 7h 2h 5s 5d
#R[4d]				;River goes 4d, board is 7h 2h 5s 5d 4d
#S					;Showdown
4=P					;SB is awarded pot worth 3850
4[QdAh]PA+AQ		;SB shows Qd Ah - hand is pair with Ace and Queen kickers
6[JdAc]PA+AJ		;UTG shows Jd Ac - hand is pair with Ace and Jack kickers
```

## Specification
A *PSN* hand has the following anatomy:
```
GAME
INFO
NAMES
STACKS
STREETS
```

### General
- All lines are mandatory with the exception of INFO and NAMES which are optional
- The order of lines *must* follow the order shown above, with the exception of NAMES and STACKS which may be swapped around
- *Spaces* and *New Lines* are interchangable, with the following caveats:
	- There *must* be a new line after GAME and INFO
	- There *must not* be any new lines when listing cards in a hand or a street, spaces are optional
	- There *must not* be any new lines within the text after the `INFO` tag of INFO
	- Parsers *must* allow any variant of new line from any OS and treat them as a single whitespace character
- Comments are made with the `;` character and are permitted anywhere where there is whitespace and are terminated by a new line
- Until this specification is expanded to other Poker variants, every *PSN* hand *must* start with `NLH` as the first 3 bytes

### Reading this specification
- Unless otherwise specified:
	- `<...>` denotes a required field
	- `[...]` denotes an optional field
	- `{...}` denotes a field to be formatted according to its corresponding section in this specification
- All other characters outside of the above delimiters *must be written verbatim*

### GAME
**`NLH {Bets} {Seats}`**
- *Must* start with `NLH` characters followed by a space
- **{Bets}**
	**`[Ante]|[Small Blind]|<Big Blind>`**
	- Bets format can be `A|SB|BB`, `SB|BB` or just `BB`
	- `[Small Blind]` is assumed to be 1/2 of `<Big Blind>` if omitted
	- If using `[Ante]`, `[Small Blind]` *must* be specified
- **{Seats}**
	**`BTN <Dealer>/<Seats>/[Max]`**
	**`<Seats>/[Max]`**
	- Seats format can be `BTN Dealer/Seats/Max`, `BTN Dealer/Seats`, `Seats/Max` or just `Seats`
	- `BTN` followed by a space denotes BTN notation
	- BTN notation means players *must* be identified by their seat number
	- If not using BTN notation, players *must* be identified by their position, see **Values -> Positions**
- *Mandatory* new line after end of line

### INFO
**`DATE <Date>`**
**`CASH <Currency>`**
**`LVL <Level>`**
**`BUY <Amount>[Currency]`**
**`INFO <Info>`**
- This line is entirely optional
- Each tag is optional, as is the order, with the exception of `INFO` which *must* come after all other tags
- `DATE` denotes the date time the game/hand started at Preflop, *must* be followed by a valid [ISO 8601][https://en.wikipedia.org/wiki/ISO_8601] timestamp in the format `YYYY-MM-DDThh:mm:ssZ` or `YYYY-MM-DDThh:mm:ss±hh:mm`
- `CASH` denotes a cash game, *must* be followed by a valid [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217#Active_codes) `<Currency>` code
- `LVL` denotes a tournament gane, *must* be followed by current level # of tournament
- `BUY` denotes the buy-in
	- *Must* be followed by the monetary `<Amount>` of the buy-in including decimal point
	- *May* be followed by a valid [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217#Active_codes) `[Currency]` code
		- `[Currency]` required if game is tournament
		- If `[Currency]` is present and the `CASH` tag is not, this indicates a cash game (to save specifying the currency twice)
- `INFO` denotes any information about the game that does not have a bearing on how the hand is played, followed by any text with the exception of `;` and new lines
- *Mandatory* new line after end of line

### NAMES
**`<Seat/Position>="<Name>"`** *(Multiple)*
- This line is entirely optional
- `=` denotes player's real name or username/handle
- `<Name>` *must* be surrounded by double quotes
- **Hero**
	**`<Seat/Position>=HERO`**
	- *may* be used to signify that the indicated player is the Hero (owner of the hand history)
	- `HERO` *must* be in all-caps, and *must not* be surrounded by double quotes to distinguish it from a name entry
	- This *may* be present with or without the corresponding player's name entry

### STACKS
**`<Seat/Position>=<Chips>`** *(Multiple)*
- Only players who don't fold in Preflop *must* be represented, but all may be if desired
- `=` denotes player's chips
- `nB` means ***n*** multiples of Big Blind
- Any number after `B` is the remainder of chips not divisible by the Big Blind
- A number on its own, without `B`, is simply the amount of chips
- Chip amounts (without `B`) and remainders *must* be decimal in case of cash games

### STREETS
**`#P {Actions...}`** 						*Preflop*
**`#F{Cards} {Actions...}`** 				*Flop*
**`#T{Card} {Actions...}`** 				*Turn*
**`#R{Card} {Actions...}`** 				*River*
**`#S {Payouts...} {Winning Hands...}`** 	*Showdown*
**`#E {Payout}`** 							*End before showdown (All Fold)*
- `#P` is mandatory, there cannot be a hand without Preflop
- Either `#S` or `#E` *must* be present, there must be a logical end to the hand
- **{Card(s)}**
	**`[<Rank><Suit> <Rank><Suit> <Rank><Suit>]`**
	**`[<Rank><Suit> <Rank><Suit>]`**
	**`[<Rank><Suit>]`**
	- *Must* be enclosed in Square Brackets (these do not denote optional parameters like elsewhere in the document)
	- `<Rank>` *must* be a valid rank, see **Values -> Ranks**
	- `<Suit>` *must* be a valid suit, see **Values -> Suits**
	- Can only contain 3 cards (preflop), 2 cards (player hands), or 1 card (river, turn)
	- Optional space between each card
- **{Actions}**
	**`<Seat/Position>:<Action>[Amount]`** *(Multiple)*
	- There *must* be a space between each player's action
	- If preflop, any omissions are assumed to indicate that the omitted players have folded
	- `:` denotes a player's action
	- `<Action>` *must* be one of `C K L R X`:
		- `C` is either **Check** or **Call**, depending on context (if there's a bet to call or not)
			- If checking, amount *must not* be present
			- If calling, amount is optional
		- `K` is **Definitive Check**, amount *must not* be present
		- `L` is **Definitive Call**, amount is optional
		- `R` is **Raise**, amount *must* be present
		- `X` is **Fold**, amount *must not* be present
	- Amount can be `A` for all-in, or a chip amount (see **{Stacks}** for format). These can be combined if it desired to show both
	- **Early Showdown**
		**`<Seat/Position>{Cards}`**
		- An optional *pseudo* action can be used to signify a player showing their hand
		- Even if this is omitted, the parser should use any and all information from the `#S` or `#E` sections to reveal the same information, if present
		- See **{Card(s)}** for cards format
- **{Payout(s)}**
	**`<Seat/Position>=<Pot>`** *(Multiple)*
	- `=` denotes player's chips
	- `<Pot>` can be `P` or `POT` if the specific amount is unimportant or to be calculated by the parser, or a chip amount (see **{Stacks}** for format). These *must not* be combined
- **{Winning Hands}**
	**`<Seat/Position>{Cards}<Hand ID>{Kickers}`** *(Multiple)*
	- *Must* show at minimum all winners of the pot
	- Any additional entries reveal other seat's hole cards
	- If an omitted player was still in play by showdown, this means they mucked their cards
	- See **{Card(s)}** for cards format
	- `<Hand ID>` *must* be a valid winning hand, see **Values -> Hands**
	- **{Kickers}**:
		**`+<Rank><Rank><Rank>`**
		**`+<Rank><Rank>`**
		**`+<Rank>`**
		- `+` denotes the rank of one or more kicker cards that came into decision
		- `<Rank>` *must* be a valid rank, see **Values -> Ranks**

### Values

#### Positions:
- Dealer:			`BTN`, `BU` or `D`
- Small Blind:		`SB` or `S` (if heads-up, *Dealer* positions may be used to indicate the same)
- Big Blind:		`BB` or `B`
- Under the Gun:	`UTG`, `UG` or `U`
- Seats after UTG:	`UTG+n` or `U+n`
- Middle Position:	`MP` or `M`
- Seats after MP:	`MP+n`, `M+n`
- Cutoff:			`CO` or `C`
- Hijack:			`HJ` or `H`

#### Ranks:
- Valid values: `2 3 4 5 6 7 8 9 T J Q K A`
- `10` *must not* be used in place of `T`

#### Suits:
- Valid values: `c d h s` or `♣ ♦ ♥ ♠`
- *Must* be lowercase if using letters

#### Hands:
- Royal Flush:		`RF`
- Straight Flush:	`SF`
- Four of a Kind:	`4K`
- Full House:		`FH`
- Flush:			`FL`
- Straight:			`ST`
- Three of a Kind:	`3K`
- Two Pair:			`2P`
- Pair:				`PA`
- High Card:		`HC`