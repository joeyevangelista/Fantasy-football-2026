# Joey Fantasy Football Draft

A draft-day assistant for a snake draft. Tracks who is gone, tracks your roster,
and tells you who to take next and why.

Built for a 14 team league with a 1 QB / 2 RB / 2 WR / 1 TE / 1 K / 1 DST
lineup, but every one of those numbers is editable in Setup.

---

## Part 1: Get the APK

You do not need Android Studio. GitHub builds it for you.

1. Go to github.com and sign in. Create a new repository. Name it anything.
   Make it **Private** if you like, it works either way.
2. On the new repo page choose **uploading an existing file**.
3. Unzip the folder I gave you and drag **everything inside it** into the
   upload box. Make sure you drag the contents, not the outer folder.
   The `.github` folder must come along, it is the part that runs the build.
4. Click **Commit changes**.
5. Open the **Actions** tab. A job called **Build APK** starts on its own.
   It takes about 4 minutes the first time.
6. When the green check appears, click the job, scroll to **Artifacts**, and
   download `JoeyFantasyDraft-APK`. That zip holds `app-debug.apk`.
7. Move the APK to your phone and tap it. Android will ask you to allow
   installs from your browser or file manager. Allow it, then install.

If the build fails, open the failed step and read the last 20 lines. That text
tells you exactly what went wrong.

---

## Part 2: Load projections before draft day

The app ships with no players. You feed it a CSV, which keeps the numbers
current and lets you blend several sources.

**Where to get one:** FantasyPros lets you export their consensus projections
to CSV for free. Their consensus is already an average of many analysts, so one
file gets you most of the way. Any site that exports a CSV works.

**What the file needs:** a header row with a player column and a projection
column. These all work:

| Column | Accepted names |
|---|---|
| Player | Player, Player Name, Name |
| Position | POS, Position (skip it and set the position at import instead) |
| Team | Team, TM |
| Projection | FPTS, Points, Proj, Projected Points |
| Receptions | REC, Receptions |
| Bye | Bye, Bye Week |

Include the **REC** column if you can. That is what lets the app convert a
file between standard, half PPR and full PPR when you flip the scoring switch.

**To import:** Setup tab, set *File was scored as* to match the file, then tap
**Import projections**.

**To blend several sources:** leave *Blend* on and import a second and third
file. Players that appear in more than one file get averaged. Switch to
*Replace* to start the pool over.

**Per position files:** some sites export one file per position with no POS
column. Set *Position in this file* to RB, import the RB file, then switch to
WR and import that one, and so on.

Do this the night before, not during the draft.

---

## Part 3: Draft day

The **Board** tab has three parts.

**The clock** shows the round, the pick, and how many players go before your
turn comes around.

**Value lost if you wait** is the part worth watching. For each position it
simulates the picks between now and your next turn, then shows how far the best
available player at that position will fall by the time you choose again. A big
red RB number means take a running back now. Small numbers everywhere mean you
can take the best player and not worry.

**The five suggestions** rank players on four things at once: how far above a
replacement level starter they are, how much value at their position evaporates
before your next turn, which of your starting spots are still empty, and how
many picks you have left to fill them. Each one tells you which of those
reasons put it there.

Two buttons on every player:

- **Taken** means someone else drafted him. He leaves the pool.
- **I took him / Mine** adds him to your roster and drives the suggestions.

Get those two straight, the advice depends on it. **Undo last pick** at the top
fixes a mis-tap.

Kickers and defenses are held out of the suggestions until the last two rounds,
then pushed to the top so you do not finish the draft without one.

Everything saves as you tap. Close the app, lose signal, get a phone call, your
draft is still there. The app never needs a connection once projections are in.

---

## Two things this does not do

**It does not connect to CBS.** CBS has no public fantasy API and their draft
room is a live socket that outside apps cannot read. Anything built to scrape it
would break the moment they change a page, most likely mid draft. Tapping
**Taken** takes under a second and never fails.

**It does not scrape projection sites.** Those sites block it, and the numbers
would go stale. Importing a CSV you control is more accurate and takes half a
minute.

---

## Changing the math

`Engine.kt` holds all of it.

- `replacementLevels` sets the baseline: the last starter at each position
  across the league.
- `outlook` models the picks before your next turn and works out what will be
  left when you get there.
- `suggest` combines value, wait cost, roster needs, and urgency into a score.
  The constants near the bottom of that function are the knobs. Raise the `25.0`
  to weight your empty starting spots more heavily. Raise the `0.75` on
  `waitCost` to chase scarcity harder.
