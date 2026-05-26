# Authoring a campaign for Afterware

A campaign is a single self-contained data object. The engine never references a specific story, so writing a new one is a data task, not an engine task. You can register a campaign in source, or paste it into the deck at runtime with no code edits at all.

This guide covers the full schema, the built-in puzzles and their specs, the validator, and the paste-to-load flow.

## The fastest path: paste and play

1. Power on the deck and enter any handle. You land in the campaign library.
2. Press A (ADD A CAMPAIGN), or type `add-campaign` at the deck.
3. Paste your campaign object (or JSON) into the box and press LOAD PACK.

The deck validates the pack first. If anything is wrong it refuses to add it and lists the reasons, leaving your library unchanged. If it passes, the pack is added to your library, where it appears in the list ready to play. The importer accepts either strict JSON or a plain JavaScript object literal, so trailing commas and unquoted keys are fine. When Afterware runs as a local file, imported packs are saved to your browser's local storage and are still in the library next time you open it (type `forget` at the deck to clear saved data).

To ship a campaign permanently instead, drop it into the source and register it:

```js
Campaigns.register(yourCampaign);
```

If more than one campaign is registered, they all appear in the library, the selection hub shown after boot.

## Top-level schema

```js
{
  id,                  // required, unique string, used internally
  title,               // required, shown on screen
  blurb,               // optional one-line description shown in the campaign picker
  themeOverrides,      // optional map of CSS variable -> value (a default tint)
  continueCodes,       // optional array of one 4-digit string per mission

  handler: {
    name,              // required, the handler's display name (spoken as "NAME:: ...")
    intro: [ ... ],    // required, lines spoken before the first contract
    quips: {           // optional reactive lines
      clean,           // spoken when a breach is clean (no fails, no near-miss)
      nearmiss         // spoken on completing a contract where the trace ran hot
    }
  },

  missions: [ ... ],   // required, one or more mission objects (see below)
  targets: { ... },    // required, host -> target object (see below)

  // Endings. Provide one of these, or both:
  endings: { key: [ ... ] },   // branching endings keyed by finale option key
  ending:  [ ... ],            // single-path fallback used when there is no branch
  finale: {                    // optional branching finale on the LAST mission
    prompt: [ ... ],           // lines spoken when the choice is offered
    options: [ { key, label } ]// each key MUST exist in endings
  }
}
```

Notes:

- A line that begins with the handler name (for example `STATIC:: ...`) is rendered in the handler color. Everything else uses the default phosphor color.
- Any line spoken by the handler (the intro, quips, mission beats, file reactions, the finale prompt, and endings) may contain the token `{handle}`, which the engine replaces with the handle the player typed at boot. This is how a campaign greets the player by name and makes the handler feel like they know who they recruited. Use it sparingly so it lands.
- `blurb` is shown under the title in the campaign picker when more than one campaign is loaded. Keep it to one line: genre, hook, and contract count works well.
- Do not use em dashes in any on-screen text. Use commas, periods, or parentheses.
- The handler is addressed by name, so keep `handler.name` short.

## Mission object

```js
{
  codename,            // required, the contract name
  briefing: [ ... ],   // required, lines shown when the contract opens
  targetId,            // required, must be a key in targets
  traceRate,           // required, how fast the trace climbs while idle (try 2 to 6)
  unlockCmd,           // required, the intrusion verb the player must run
  puzzleId,            // required, a registered puzzle (see below)
  puzzleSpec,          // optional, configuration passed to the puzzle
  objective,           // required, the filename to download to complete the contract
  decoys: [            // optional extra hosts shown in scan
    { host, note, honeypot }
  ],
  beats: {             // optional handler lines at each step, each an array
    onConnect,
    onAccess,
    onDownload
  }
}
```

- `unlockCmd` should match the puzzle. The built-in verbs are `crack`, `decrypt`, `seq`, `route`, and `intercept`.
- `objective` must be a file that exists on the mission's target.
- Decoys are flavor hosts. A decoy with `honeypot: true` is bait. Connecting to it does not grant access, it flags the operator, and the next real connection starts with the trace already partway up. Give honeypots inviting notes and harmless ones mundane notes.

## Target object

```js
{
  banner,              // required, the connect banner, shown bright
  motd: [ ... ],       // optional login-of-the-day flavor lines, shown faint
  services: [ ... ],   // required, lines listed by probe
  locked,              // required, a description of the locked subsystem
  files: {
    name: {
      access,          // "open", "locked", or "deleted"
      text,            // a string or an array of lines
      recover,         // optional array of extra lines, revealed on read only if the player owns the Trace Scrubber
      react            // optional handler line spoken the first time the file is read
    }
  }
}
```

- A line in a file that begins with `//` is rendered faint, which reads well as a margin note or a system comment.
- `react` is the cheapest way to make the handler feel present. Attach one to the documents that matter, and the handler comments the first time the player opens each.
- Use `access` to control visibility. `open` files are always readable. `locked` files need a granted breach (the mission's puzzle solved). `deleted` files are hidden from `ls`, `cat`, and `download` entirely until the player owns the Trace Scrubber, at which point they surface in `ls` tagged `[recvrd]` and become readable. This is a clean way to reward the Scrubber with bonus story rather than mechanics.
- `recover` is the lighter version of the same idea. Put extra lines on any normal file, and they are appended as a recovered fragment when the file is read, but only if the player owns the Scrubber. Good for slipping an unredacted line into an otherwise visible document.

## Built-in puzzles and their specs

All puzzle specs are optional unless noted. The Signal Analyzer tool adds one hint to each, handled by the engine, so you do not author hints.

| puzzleId | unlockCmd | spec fields | meaning |
| --- | --- | --- | --- |
| `passcrack` | `crack` | `length`, `title` | recover a key of `length` glyphs from the deck charset |
| `decrypt` | `decrypt` | `plain` (required), `title` | the plaintext to recover from a Caesar-shifted channel; use uppercase letters and spaces |
| `seqlock` | `seq` | `rounds` (array), `title` | a memory sequence per round, for example `[4, 5, 6]` |
| `route` | `route` | `w`, `h`, `walls`, `src`, `sink`, `title` | a maze; omit to use the default solvable maze |
| `intercept` | `intercept` | `rounds` (number), `title` | a timing lock with this many rounds |
| `pretext` | `social` | `mark`, `rounds`, `title` | a social-engineering dialogue; pick the right cover story |

For `route`, if you supply explicit `walls`, the validator runs a breadth-first search and rejects the campaign if the maze is unsolvable. `walls`, `src`, and `sink` are `[row, col]` pairs. The default grid is 7 wide by 6 tall.

For `decrypt`, the `plain` text is the answer the player rotates the channel to reveal. Keep it uppercase letters and spaces.

For `pretext`, the social-engineering puzzle, you describe a person (the `mark`) and a series of conversational beats. Each round shows the mark's mood and a set of lines the player can try, and the player presses the number of the line they want. The right line advances; a wrong line is a nudge, not a failure, so the puzzle never hard-stops a reader. If you supply no `rounds`, a built-in two-round script is used. The shape is:

```js
puzzleId: "pretext",
unlockCmd: "social",
puzzleSpec: {
  title: "PRETEXT :: FRONT DESK",
  mark: "a bored receptionist, late Friday",
  rounds: [
    {
      ask: "the line is flat, going through the motions.",
      options: [
        { text: "Sound like IT doing a routine reset.", ok: true,  reply: "'oh, finally. extension 4417.'" },
        { text: "Demand a supervisor.",                  ok: false, reply: "that just wakes them up." }
      ]
    }
    // ...more rounds
  ]
}
```

Each option has `text` (what the player sees), `ok` (whether it advances), and `reply` (the mark's response, shown either way). Solving the last round opens the node.

To add your own puzzle, register a factory and reference its id from a mission. See the README section on adding a puzzle.

## The validator

The deck validates every campaign on load and on import. It reports problems on screen and in the browser console rather than crashing. It checks that:

- `id`, `title`, and `handler.name` are present.
- there is at least one mission, and an `ending` or `endings`.
- each mission points at a real `targetId` and a registered `puzzleId`.
- each mission's `objective` is a real file on its target.
- any explicit `route` maze is solvable.
- a `decrypt` spec's `plain` text contains letters to encode.
- `continueCodes`, if present, has one code per mission.
- every `finale` option key has a matching entry in `endings`.

A clean campaign produces no messages.

## Minimal working skeleton

```js
Campaigns.register({
  id: "demo",
  title: "DEMO RUN",
  handler: {
    name: "ECHO",
    intro: ["one contract. one door. go."]
  },
  ending: ["ECHO:: clean. we are done here."],
  missions: [
    {
      codename: "ONLY DOOR",
      briefing: ["crack the gate and pull the file."],
      targetId: "gate.demo.net",
      traceRate: 3,
      unlockCmd: "crack",
      puzzleId: "passcrack",
      puzzleSpec: { length: 4, title: "AUTH :: GATE" },
      objective: "payload.dat",
      decoys: [ { host: "trap.demo.net", note: "(open? suspicious.)", honeypot: true } ],
      beats: { onAccess: ["ECHO:: in. grab it and go."] }
    }
  ],
  targets: {
    "gate.demo.net": {
      banner: "DEMO GATE // public",
      motd: ["welcome. you should not be here."],
      services: ["22/ssh admin (auth required)"],
      locked: "admin shell behind a 4-glyph key",
      files: {
        "readme.txt": { access: "open", text: ["nothing to see. yet."] },
        "payload.dat": {
          access: "locked",
          text: ["the thing you came for.", "// it was here the whole time."],
          react: "ECHO:: that's it. that's the whole story."
        }
      }
    }
  }
});
```

Paste that into the importer to see it run end to end, then build outward from it.

## A branching ending

To give the last contract a choice, add a `finale` and an `endings` map whose keys match the finale option keys:

```js
finale: {
  prompt: ["NAME:: this is the part i cannot do alone. choose."],
  options: [
    { key: "free",  label: "open the door" },
    { key: "keep",  label: "keep it shut" }
  ]
},
endings: {
  free: ["NAME:: it's open. thank you."],
  keep: ["NAME:: ...you walked away. i understand."]
}
```

The player presses a number to decide, and the matching ending plays, followed by the credits crawl.

## License

GPL-3.0

This document accompanies Afterware, which is distributed under the GNU General Public License version 3 or, at your option, any later version. It is provided without any warranty.
