# Problem memo -- <team name> (Akash Kakumanu, Carlos Sandoval-Gonzalez)

## The user
Fantasy football commissioners who host live, in-person drafts with a physical draft board
and player stickers -- starting with Akash, commissioner of a 12-team fantasy football league. The device sits on the draft table next to the
board, used by a commissioner who is simultaneously a drafter, comissioner, and the party host.

## The problem
An in-person draft runs 2-4 hours with 8-12 managers and has no neutral referee:
- **Disputes over when picks are registered.** "I had my sticker before time ran out" has no
  answer, so the room argues. Our last draft lost roughly nearly an hour to these arguments.
- **Split attention.** The commissioner must run the clock and enforce pick order (comissioner/referee),
  make their own picks (player), and keep the party going (host). Doing all three means doing
  each badly -- usually the commissioner's own draft suffers. This has happened to me where I was too focused as a commissioner so my draft as a player has suffered
- **Bookkeeping errors.** Out-of-order picks, already-drafted players, and a physical board
  that must be hand-typed into Sleeper afterward, where transcription errors creep in.
The cost is time, fairness, and the fun of the event. It recurs every draft: each league's
annual draft, dynasty rookie drafts, and mock drafts.

## Why a device
Sleeper and similar fantasy managing apps are viable and convenient when everyone drafts remotely. In person, they defeat the point: everyone ends up buried in a screen instead of being present, and
custom house rules and clock times don't map cleanly onto the app. An app also still needs a
human operating it -- pressing the timer, typing every pick -- which is the commissioner's
job we are trying to remove. This device observes the physical draft directly (the gavel
and the scanned sticker), so nobody operates it. The absent observer here is the
commissioner's attention: the device must stay awake and correct through hours of noise,
cheering, and table slaps, survive a power blip without losing the draft, and run fully
offline.

## The sensors
- **Accelerometer (MPU-6050) on the gavel's sound block + microphone (INMP441):** fused into
  one strike decision. A real strike shows both an impact and an acoustic crack within a
  few milliseconds; a table slap or cheering shows only one.
- **RC522 NFC reader + NTAG213 tags on each player sticker:** identifies the player.
They cooperate in one decision: a pick commits only when a valid scan and a confirmed strike
fall inside the active turn window, ordered by capture timestamp. The gavel starts the draft,
starts each turn, and seals picks. A timeout raises a warning and pauses the clock; the next
strike puts the pick to the group, and the late team takes a house consequence (logged).

## The mechanisms
- **B, interrupt vs. polling (RC522):** the scan timestamp decides on-time vs. late, so scan
  latency must be measured both ways, idle and loaded. (F, no-drop sampling for the strike
  streams, is a likely fourth.)
- **D, custom storage:** append-only pick log, fsync'd per sealed pick, replayed on boot --
  a power blip at pick 90 of 180 must not lose the draft.
- **E, multi-process + supervisor:** strike capture, scanner, draft engine, and display are
  isolated, so a dead reader or UI can't take the draft state down.

## The risk
Strike classification: whether cheap sensors can reliably tell a gavel strike from table
slaps and crowd noise. Because the gavel is the only command channel, a false strike can seal
a pick or start a clock early. We will record labeled strikes and look-alike events in Week 5
before committing. Secondary: the draft is an occasional, supervised event, so our 48-hour
soak runs in a deliberately noisy room with live mock drafts to measure false strikes per hour.
