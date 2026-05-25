# AfterWare

A single-file cyberpunk hacking simulator that runs entirely in the browser. You sit at a salvaged 1980s CRT terminal, take contracts from an anonymous handler, and break into a megacorporation one layer at a time, uncovering its secret as you go.

Version 0.1. Vanilla HTML, CSS, and JavaScript in one file. Zero dependencies, no build step, no network calls.

## Running it

Double-click `hacksim.html`, or open it in any modern browser. There is nothing to install.

The screen starts powered off. Click anywhere or press any key to power on. That first gesture also unlocks audio, since browsers block sound until the user interacts with the page. There is a mute toggle on the bezel if you would rather play in silence.

## How to play

After power-on the deck runs a short boot sequence and asks for an operator handle. Type anything and press Enter. Your handler, STATIC, then briefs you on the first contract.

Each contract follows the same rhythm:

1. `scan` to find the target host on the local net.
2. `connect <host>` to dial in. A trace meter begins tracking the connection.
3. `probe` to map the node and reveal the locked subsystem, which tells you the intrusion verb to use.
4. Run that verb (`crack`, `decrypt`, `seq`, `route`, or `intercept`) to launch the mission's puzzle. Solving it grants access and unlocks the data store.
5. `ls` and `cat <file>` to read the leaked documents, then `download <objective>` to exfiltrate the file the contract wants.
6. `disconnect` to get out clean before the trace meter fills.

Read the files. The story lives in them. The corporation's secret unspools document by document, and the ending reframes who your handler really is.

### The trace meter

While you are connected, a trace meter sits in the HUD. The key thing to understand is that it only climbs while you are sitting idle at the prompt of a live connection. It freezes whenever the deck is printing output, while a download runs, and while a puzzle is open. In other words, you are never penalized for reading the story, watching an animation, or working a lock. The pressure is on hesitation, not on the game's own pacing.

The deeper the corporate layer, the faster idle time costs you. The meter turns amber as a caution and red near the top, with an alarm to match. Breaching a system (solving its puzzle) cools the meter down and buys you a comfortable window to read and exfiltrate. If the meter ever reaches 100 percent you are traced and bailed out automatically, losing your foothold (though if the objective was already downloaded, the data is still considered clear, and the contract completes).

### Continue codes

The deck persists nothing between loads, so every contract has its own 4-digit continue code, shown on the yellow sticky note beside the screen. The note always displays the code for the contract you are currently on, and it updates as you advance.

If you close the deck, or get knocked offline and want to step away, you can pick up later. Boot a fresh session, enter any handle, and type `resume <code>` (for example, `resume 1984`) to jump straight back to that contract. Loading a code starts that contract fresh, so you replay its puzzle but skip everything before it. An invalid code is rejected, and you cannot load a code while you are still connected to a node.

A campaign can define its own thematic codes in a `continueCodes` array. If it does not, the deck generates a stable code for each mission automatically.

## Commands

Intrusion verbs only work when they match the current contract's locked subsystem. If a verb does not apply, the deck says so and points you back to `probe`.

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
| `disconnect` | `exit`, `logout`, `quit` | Close the current connection |
| `status` | `whoami`, `stat` | Show operator status |
| `contract` | `contracts`, `mission`, `brief` | Reread the current brief |
| `resume <code>` | `continue`, `code` | Reload a contract from a 4-digit continue code |
| `clear` | `cls` | Clear the screen |

## Puzzles

Each contract is gated by one minigame. Press ESC at any time to abort a puzzle and return to the terminal.

- **passcrack** (`crack`): Recover a multi-glyph key from a fixed charset. Type a full-length guess and press Enter for feedback per slot (exact, wrong slot, not present), in the style of a password Mastermind.
- **decrypt** (`decrypt`): A Caesar-shifted channel. Use the up and down arrows to rotate the offset and watch the plaintext resolve live, then press Enter to confirm.
- **seqlock** (`seq`): A memory lock. Watch the symbol sequence flash, then replay it with the arrow keys. Each round adds a symbol.
- **route** (`route`): A signal-routing maze. Move with the arrow keys from the source (S) to the sink (X) without entering a firewall cell.
- **intercept** (`intercept`): A timing lock. A marker sweeps across a window. Press Space to lock it inside the green zone. The window shrinks and the marker speeds up each round.

## Campaign 01: THE VANTA FILES

Five escalating contracts against VANTA BIOSYSTEMS, whose public face is neural-wellness implants and whose private face is something far worse:

1. **FRONT DOOR** (`vanta-www.pub`, crack): breach the public gateway and pull the staff directory.
2. **PAPER TRAIL** (`mail.vanta-int.net`, decrypt): break the executive mail channel and read the redacted thread.
3. **DEAD DROP** (`hr.vanta-int.net`, seq): open the HR vault and pull the volunteer roster.
4. **WET WIRE** (`lab7.vanta-sec.net`, route): route past the research firewall and read the Continuity log.
5. **THE CORE** (`core.vanta-blacksite`, intercept): reach the black-site core, pull the board minutes, and decide what happens next.

## Architecture

The file is organized into clearly separated modules inside one script. Search the source for these section headers:

`[CONFIG]`, `[BUS]`, `[AUDIO]`, `[FX]`, `[TERM]`, `[STATE]`, `[TRACE]`, `[CMD]`, `[PUZZLE]`, `[CAMPAIGN]`, `[CONTENT]`, `[BOOT/INIT]`.

The guiding principle is a hard line between engine and content. The engine (terminal, command dispatch, puzzles, trace meter, audio, CRT effects) never references a specific story. All lore lives in content packs. Systems stay decoupled through a small event bus, so the trace meter, audio, and screen effects react to events like `trace.spike`, `system.breached`, and `mission.complete` rather than calling each other directly.

### Adding a command

Register one object. No parser edits are required, and `help` is generated automatically from the registry.

```js
Commands.register({
  name: "ping",
  aliases: ["p"],
  help: "test a host's latency",
  run: async (args) => { await Term.print("pong"); }
});
```

### Adding a puzzle

Every minigame is a factory that returns an input handler and a teardown function. It renders into the overlay body and signals completion through the shared api.

```js
Puzzles.register("mygame", (spec, api) => {
  // api = { body, foot(txt), solve(), fail(msg), bump(n) }
  api.body.appendChild(/* your UI */);
  api.foot("controls go here");
  return {
    onInput(e) { /* if solved */ api.solve(); },
    teardown() { /* clear any timers */ }
  };
});
```

## Campaign framework

A story is a single self-contained data object validated against the schema below. The corporation story is just Campaign 01. Adding a new story is a data task, not an engine task.

```js
{
  id, title, themeOverrides,            // themeOverrides is optional
  continueCodes,                        // optional array of one 4-digit code per mission
  handler: { name, intro: [ ...lines ] },
  ending: [ ...lines ],
  missions: [
    {
      codename, briefing: [ ...lines ],
      targetId, traceRate,
      unlockCmd, puzzleId, puzzleSpec,  // puzzleSpec is optional
      objective,                        // filename to download
      decoys: [ ...hosts ],             // optional flavor hosts in scan
      beats: { onConnect, onAccess, onDownload }  // each optional, arrays of lines
    }
  ],
  targets: {
    host: {
      banner,
      services: [ ...lines ],
      locked,                           // description of the locked subsystem
      files: {
        name: { access, text }          // access is "open" or "locked"; text is a string or array of lines
      }
    }
  }
}
```

### How to add a new campaign

1. Write a campaign object against the schema above.
2. Optionally register any new puzzles it needs with `Puzzles.register(...)`.
3. Register the campaign with `Campaigns.register(yourCampaign)`.

No engine edits are required. If more than one campaign is registered, a start menu appears at boot and the player picks one. With a single campaign, the deck boots straight into it.

## Configuration and theming

Tunable constants live in the `CONFIG` block at the top of the script:

- `typeSpeed`, `bootSpeed`: typewriter pacing in milliseconds per character.
- `traceTickMs`: how often the trace meter climbs while connected.
- `traceWarnAt`, `traceCritAt`: the percentages where the caution and critical accents kick in.
- `charset`: the glyph set used by the password puzzle.

The look is driven by CSS variables in the `:root` block. To change the base color, edit `--phosphor` (green by default, with amber and ice-blue values noted in the comments). A campaign can override these per story by setting `themeOverrides` to a map of CSS variable names and values, which the campaign manager applies on load.

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
