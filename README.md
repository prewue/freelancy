# Freelancy

Bill by the hour and watch the money tick in the notch.

Freelancy is a Droplet for [Droppy](https://getdroppy.app), the Dynamic Island
and shelf for Mac. It is a small time and money meter for hourly work: projects
with a rate, a currency and a colour, pausable sessions inside them, and the
clock and the amount earned so far sitting beside the notch while a session
runs.

## What it does

- **Timer.** The current session of the current project: the clock, the money
  under it, pause and resume at one hand, a fresh session at the other. The
  project's badge above the clock switches projects; asking first while
  another project's session runs.
- **Projects and sessions.** Cards in a row that scrolls sideways. Project
  cards are glass in the project's colour. Hover for analytics, edit, rename
  and delete; a delete asks first, right on the shelf.
- **New project.** Three short steps on the shelf: a name, an icon and a
  colour from two wheels, then an hourly rate and its currency.
- **Analytics.** One project at a time: the totals, two weeks of time and
  money as bars (hover a bar for that day), and a CSV of that project's
  sessions saved to Downloads.
- **Notch.** While a session runs, the clock in the project's colour and the
  amount earned sit beside the notch and tick every second, only the changed
  digit moving. A brief strip can confirm a start or a pause.
- **Settings.** Defaults for new projects, project names, colours, icons,
  rates and currencies, session names and lengths, and the shelf's questions.

Typing a name on the shelf needs Droppy to give the widget keyboard focus,
which current Droppy builds do only some of the time; names can always be
changed in Settings.

## Requirements

Droppy 15.3 or later, or the free
[Droppy Playground](https://getdroppy.app/download/playground).

## Installing

From the Store, once published: the Community shelf, under 8lx.

From this repository:

```bash
droppykit build
droppykit install
```

`droppykit build` writes `.build/FreelanceTracker.droplet`; `droppykit install`
puts it into Droppy Playground and relaunches it. In Droppy itself, add the
bundle under Settings, Store, Local droplets and approve the build.

## Building

The package is built with [DroppyKit](https://getdroppy.app/docs/droppykit)
and needs Swift 6.2 (Xcode 26 or later).

```bash
droppykit build      # the bundle
droppykit validate   # the checks a submission runs
droppykit run        # the harness window, one page per surface
swift test           # the model, formatting and CSV tests
```

`AGENTS.md` is the brief for coding agents; `.mcp.json` and `.cursor/mcp.json`
connect the DroppyKit MCP server for Claude Code and Cursor.

## Licence

MIT. See `LICENSE`.
