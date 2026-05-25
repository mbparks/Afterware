# Afterware

A single-file cyberpunk hacking simulator that runs entirely in the browser. You sit at a salvaged 1980s CRT terminal (a CRT-80), take contracts from an anonymous handler, and break into a megacorporation one layer at a time, uncovering its secret as you go.

Version 0.2. Vanilla HTML, CSS, and JavaScript in one file. Zero dependencies, no build step, no network calls.

## Running it

Double-click `afterware.html`, or open it in any modern browser. There is nothing to install.

The screen starts powered off. Click anywhere or press any key to power on. That first gesture also unlocks audio, since browsers block sound until the user interacts with the page. There is a mute toggle on the bezel, and a fuller `settings` panel (volume, effects intensity, screen tint) described below.

## How to play

After power-on the deck runs a short boot sequence, paints the AFTERWARE logo, and asks for an operator handle. Type anything and press Enter. Your handler, STATIC, then briefs you on the first contract.

Each contract follows the same rhythm:

1. `scan` to find the target host on the local net. Other hosts show up too. Some are dead ends, and some are bait.
2. `connect <host>` to dial in. A trace meter begins tracking the connection.
3. `probe` to map the node and reveal the locked subsystem, which tells you the intrusion verb to use.
4. Run that verb (`crack`, `decrypt`, `seq`, `route`, or `intercept`) to launch the mission's puzzle. Solving it grants access and unlocks the data store.
5. `ls` and `cat <file>` to read the leaked documents, then `download <objective>` to exfiltrate the file the contract wants.
6. `disconnect` to get out clean before the trace meter fills.

Read the files. The story lives in them. The corporation's secret unspools document by document, and the ending reframes who your handler really is. STATIC reacts as you go, to a clean breach, to a near-miss with the trace, and to specific documents you open.

Across the top of the screen, an objective line always shows your current step, so you never have to remember what comes next.

### The trace meter and covering your tracks

While you are connected, a trace meter sits in the HUD. The key thing to understand is that it only climbs while you are sitting idle at the prompt of a live connection. It freezes whenever the deck is printing output, while a download runs, and while a puzzle is open. You are never penalized for reading the story, watching an animation, or working a lock. The pressure is on hesitation, not on the game's own pacing.

The deeper the corporate layer, the faster idle time costs you. The meter turns amber as a caution and red near the top, with an alarm to match. Breaching a system (solving its puzzle) cools the meter down and buys you a comfortable window to read and exfiltrate.

You are no longer stuck with a passive meter. The `cover` command wipes your access logs and drops the trace at the cost of a moment of activity. It is limited per connection, and the Trace Scrubber tool (see the store below) makes it stronger and reusable.

If the meter ever reaches 100 percent you are traced and bailed out automatically, losing your foothold. If you are carrying a Dead-Man Failsafe charge, it trips instead and keeps you connected once. And if the objective was already downloaded, the data is still considered clear, and the contract completes.

### Honeypots

Not every host in a `scan` is safe. Some of the extra hosts are tarpits left out as bait. They tend to look a little too inviting, an open relay or an unsecured sandbox where there should not be one. Connecting to one does not get you in. It flags your handle, and your next real connection starts hot, with the trace already partway up. Read the notes next to each host and think before you dial.

### Proof, tools, and the store

STATIC pays in proof. You earn it by completing contracts, more for a clean run with no near-misses and no fumbled locks. Between jobs, at the deck (not while connected), type `store` to see the black market and `buy <tool>` to spend proof. Tools carry across the whole run:

- **Trace Scrubber**: two jobs. `cover` wipes more heat and can be run twice per connection, and it recovers deleted and redacted documents on a node. Recovered files surface in `ls` marked `[recvrd]`, and some files reveal extra recovered fragments when you read them. The story rewards owning it.
- **Signal Analyzer**: every lock opens with one extra hint.
- **Dead-Man Failsafe**: survive one trace-out per charge without losing access. Stackable.

Your proof balance and owned tools show in `status`, and proof sits in the HUD.

### The choice at the end

The final contract does not end on rails. Once you reach the core and hold the release interlock, you decide what happens to the forty-one minds Vanta is running. Each choice leads to a distinct ending. There is no undo, only the switch and your hand on it.

### Continue codes

The deck persists nothing between loads, so every contract has its own 4-digit continue code, shown on the yellow sticky note beside the screen. The note always displays the code for the contract you are currently on, and it updates as you advance.

If you close the deck, or get knocked offline and want to step away, you can pick up later. Boot a fresh session, enter any handle, and type `resume <code>` (for example, `resume 1984`) to jump straight back to that contract. Loading a code starts that contract fresh, so you replay its puzzle but skip everything before it. An invalid code is rejected, and you cannot load a code while you are still connected to a node.

A campaign can define its own thematic codes in a `continueCodes` array. If it does not, the deck generates a stable code for each mission automatically.

## Settings

Type `settings` (or `config`, `options`, `prefs`) to open the deck settings panel. Click a control to change it, and press ESC to close.

- **Volume**: a ten-segment level for all synthesized sound.
- **Effects**: cycles full, reduced, and minimal. Reduced drops the flicker and grain. Minimal also removes scanlines and the curvature glow and stops the glitch animation. This is both polish and accessibility, since the full CRT effect set is intense.
- **Phosphor**: cycles the screen tint between green, amber, and ice blue. This is the player-facing theme switch, separate from any per-campaign tint.
- **Import campaign**: paste a campaign object or JSON into the box and click Load to boot straight into a story without touching the engine. The deck validates it first and refuses a broken pack with a list of reasons. See `CAMPAIGN_AUTHORING.md`.

## Commands

Intrusion verbs only work when they match the current contract's locked subsystem. If a verb does not apply, the deck says so and points you back to `probe`. Use `man <command>` for a longer explanation of any command.

| Command | Aliases | Purpose |
| --- | --- | --- |
| `help` | `?`, `commands` | List available operations |
| `scan` | | Scan the local net for reachable hosts |
| `connect <host>` | `dial`, `ssh` | Connect to a host |
| `probe` | `nmap`, `recon` | Probe the connected node |
| `crack` | | Brute-force an auth challenge |
| `decrypt` | | Break an encrypted channel |
| `seq` | | Defeat a sequence lock |
| `route` | | Route past a firewall |
| `intercept` | | Intercept a packet window |
| `ls` | `dir` | List files on the connected node |
| `cat <file>` | `read`, `open`, `more` | Read a file |
| `download <file>` | `get`, `exfil` | Exfiltrate a file |
| `cover` | `scrub`, `wipe` | Wipe logs to cut the trace (while connected) |
| `disconnect` | `exit`, `logout`, `quit` | Close the current connection |
| `status` | `whoami`, `stat` | Show operator status, proof, and tools |
| `store` | `shop`, `market` | Browse the tool market (at the deck) |
| `buy <tool>` | | Purchase a tool with proof |
| `contract` | `contracts`, `mission`, `brief` | Reread the current brief |
| `resume <code>` | `continue`, `code` | Reload a contract from a 4-digit continue code |
| `settings` | `config`, `options`, `prefs` | Open the deck settings panel |
| `man <command>` | | Detailed help for a command |
| `clear` | `cls` | Clear the screen |

The line editor supports the up and down arrows to walk back through your command history, and Tab to complete command names and, after a file command, filenames on the current node.

## Puzzles

Each contract is gated by one minigame. Press ESC at any time to abort a puzzle and return to the terminal. The Signal Analyzer tool adds one hint to each.

- **passcrack** (`crack`): Recover a multi-glyph key from a fixed charset. Type a full-length guess and press Enter for feedback per slot (exact, wrong slot, not present), in the style of a password Mastermind. Analyzer reveals one correct glyph.
- **decrypt** (`decrypt`): A Caesar-shifted channel. Use the up and down arrows to rotate the offset and watch the plaintext resolve live, then press Enter to confirm. Analyzer reveals the first plaintext letter.
- **seqlock** (`seq`): A memory lock. Watch the symbol sequence flash, then replay it with the arrow keys. Each round adds a symbol. Analyzer flashes the sequence more slowly.
- **route** (`route`): A signal-routing maze. Move with the arrow keys from the source (S) to the sink (X) without entering a firewall cell. Analyzer faintly marks the shortest path.
- **intercept** (`intercept`): A timing lock. A marker sweeps across a window. Press Space to lock it inside the green zone. The window shrinks and the marker speeds up each round. Analyzer widens the window.

## Campaign 01: THE VANTA FILES

Five escalating contracts against VANTA BIOSYSTEMS, whose public face is neural-wellness implants and whose private face is something far worse:

1. **FRONT DOOR** (`vanta-www.pub`, crack): breach the public gateway and pull the staff directory.
2. **PAPER TRAIL** (`mail.vanta-int.net`, decrypt): break the executive mail channel and read the redacted thread.
3. **DEAD DROP** (`hr.vanta-int.net`, seq): open the HR vault, pull the volunteer roster, and read what a volunteer left behind.
4. **WET WIRE** (`lab7.vanta-sec.net`, route): route past the research firewall and read the Continuity log.
5. **THE CORE** (`core.vanta-blacksite`, intercept): reach the black-site core, pull the board minutes, and choose what happens next.

## Architecture

The file is organized into clearly separated modules inside one script. Search the source for these section headers:

`[CONFIG]`, `[BUS]`, `[AUDIO]`, `[FX]`, `[TERM]`, `[STATE]`, `[TRACE]`, `[CONTINUE]`, `[CMD]`, `[PUZZLE]`, `[GAME]`, `[CAMPAIGN]`, `[CONTENT]`, `[V0.2 SYSTEMS]`, `[BOOT/INIT]`.

The guiding principle is a hard line between engine and content. The engine (terminal, command dispatch, puzzles, trace meter, economy, audio, CRT effects) never references a specific story. All lore lives in content packs. Systems stay decoupled through a small event bus, so the trace meter, audio, and screen effects react to events like `trace.spike`, `system.breached`, and `mission.complete` rather than calling each other directly.

### Adding a command

Register one object. No parser edits are required, and `help` is generated automatically from the registry. An optional `man` array provides the detailed help text.

```js
Commands.register({
  name: "ping",
  aliases: ["p"],
  help: "test a host's latency",
  man: ["sends an echo request and reports round-trip time."],
  run: async (args) => { await Term.print("pong"); }
});
```

### Adding a puzzle

Every minigame is a factory that returns an input handler and a teardown function. It renders into the overlay body and signals completion through the shared api. If `spec.assist` is set (the Signal Analyzer), offer one hint.

```js
Puzzles.register("mygame", (spec, api) => {
  // api = { body, foot(txt), solve(), fail(msg), bump(n) }
  api.body.appendChild(/* your UI */);
  api.foot("controls go here");
  if (spec.assist) { /* show one hint */ }
  return {
    onInput(e) { /* if solved */ api.solve(); },
    teardown() { /* clear any timers */ }
  };
});
```

## Campaign framework

A story is a single self-contained data object validated against the schema below. The corporation story is just Campaign 01. Adding a new story is a data task, not an engine task. The full schema, with every optional field and the paste-to-load flow, is documented in `CAMPAIGN_AUTHORING.md`.

```js
{
  id, title, themeOverrides,            // themeOverrides is optional
  continueCodes,                        // optional, one 4-digit code per mission
  handler: {
    name,
    intro: [ ...lines ],
    quips: { clean, nearmiss }          // optional reactive handler lines
  },
  endings: { key: [ ...lines ] },       // branching endings, keyed by finale option
  ending: [ ...lines ],                 // optional single-path fallback ending
  finale: {                             // optional branching finale on the last mission
    prompt: [ ...lines ],
    options: [ { key, label } ]         // each key must exist in endings
  },
  missions: [
    {
      codename, briefing: [ ...lines ],
      targetId, traceRate,
      unlockCmd, puzzleId, puzzleSpec,  // puzzleSpec is optional
      objective,                        // filename to download
      decoys: [ { host, note, honeypot } ],  // optional; honeypot true makes it bait
      beats: { onConnect, onAccess, onDownload }  // each optional, arrays of lines
    }
  ],
  targets: {
    host: {
      banner,
      motd: [ ...lines ],               // optional login-of-the-day flavor
      services: [ ...lines ],
      locked,                           // description of the locked subsystem
      files: {
        name: { access, text, react }   // access is "open" or "locked"; text is string or array; react is an optional first-read handler line
      }
    }
  }
}
```

### How to add a new campaign

1. Write a campaign object against the schema above.
2. Optionally register any new puzzles it needs with `Puzzles.register(...)`.
3. Register the campaign with `Campaigns.register(yourCampaign)`, or paste it into the settings importer.

No engine edits are required. The deck runs a validator on load and on import. It checks that every mission points at a real target and a registered puzzle, that the objective is a real file, that any explicit routing maze is solvable, and that branching finale options have matching endings. Problems are reported on screen and in the console rather than crashing the deck. If more than one campaign is registered, a start menu appears at boot.

## Configuration and theming

Tunable constants live in the `CONFIG` block at the top of the script:

- `typeSpeed`, `bootSpeed`: typewriter pacing in milliseconds per character.
- `traceTickMs`: how often the trace meter climbs while connected.
- `traceWarnAt`, `traceCritAt`: the percentages where the caution and critical accents kick in.
- `charset`: the glyph set used by the password puzzle.

The look is driven by CSS variables in the `:root` block. The player can switch the screen tint between green, amber, and ice blue from the settings panel. A campaign can still set a default tint per story with `themeOverrides`, a map of CSS variable names and values that the campaign manager applies on load.

## Technical notes

- Vanilla JavaScript only. CRT effects favor CSS and a light animation loop to hold a steady frame rate.
- Nothing is persisted. Each load is a fresh boot. Refresh the page to run the campaign again.
- All audio is synthesized at runtime with the Web Audio API. There are no asset files.
- Tested in current versions of Chromium-based browsers and Firefox.

## Author

Michael B. Parks

## License

GPL-3.0

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY, without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
