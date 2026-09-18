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
  project's name above the clock is a badge; tap it to switch projects.
- **Projects and sessions.** Cards in a row that scrolls sideways. Tap one to
  pick it, hover for analytics, edit and delete. A new project takes a rate
  from a slider, a currency from a row of tabs and a colour from a palette.
- **Analytics.** One project at a time: the totals, two weeks of time and
  money as bars, and a CSV of that project's sessions saved to Downloads.
- **Notch.** While a session runs, the clock in the project's colour and the
  amount earned sit beside the notch and tick every second, only the changed
  digit moving. A brief strip confirms a start or a pause.
- **Settings.** Defaults for new projects, project and session names, rates
  and lengths, whether a new session starts running, and whether the amount or
  the clock gets the big digits.

Names are typed in Settings; the shelf never asks for a text field.

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
```

`AGENTS.md` is the brief for coding agents; `.mcp.json` and `.cursor/mcp.json`
connect the DroppyKit MCP server for Claude Code and Cursor.

## Licence

MIT. See `LICENSE`.
