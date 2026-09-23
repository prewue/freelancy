# Freelancy, a Droplet for Droppy

<!-- Written by `droppykit agent`. Add your own notes below; the file is only rewritten with --force. -->

This package is a **Droplet**: an extension that runs inside Droppy, the
Dynamic Island and shelf for Mac, written in SwiftUI against **DroppyKit**.
Droppy loads the built `.droplet` bundle into its own process and draws it on
the notch, the shelf, the lock screen and the menu bar.

- Droplet id: `freelance-tracker`. It is also `FreelanceTrackerDroplet.id` in Swift and `id` in `droplet.json`; the three must agree or the loader refuses the bundle.
- Swift product: `FreelanceTracker`, a dynamic library. The harness target is `FreelanceTrackerHarness`.
- SDK checkout: `/Users/maksim/desktop/droppykit` (DroppyKit 1.16.0). Docs online: https://getdroppy.app/docs/droppykit
- Host: Droppy 15.3 or later, which runs an unsigned bundle once its user approves that build under Settings, Store, Local droplets and asks again each time it opens, or the free Droppy Playground (https://getdroppy.app/download/playground), which loads unsigned bundles without asking.

## The loop

Every change goes through all of this, in order. A droplet can compile,
validate and then draw nothing, so a green build is not the end.

1. Edit `Sources/FreelanceTracker/`. The manifest is `droplet.json`.
2. `droppykit build` writes `.build/FreelanceTracker.droplet`, universal, linked against the
   framework Droppy ships. Never a bare `swift build` for the bundle: it folds a second
   copy of DroppyKit into the droplet, and that bundle loads in the harness and dies
   inside Droppy at dyld with "Symbol not found".
3. `droppykit validate` runs the exact checks the Store's intake runs.
4. `droppykit run -- --shots ./shots --report ./shots/report.json` renders every surface
   to a PNG without opening a window and writes a JSON verdict. Look at the pictures.
   Read `report.json`: `problems` must be empty and every surface you declared must be
   `provided`.
5. Put the bundle into Droppy Playground and confirm it loaded. Copy
   `.build/FreelanceTracker.droplet` to
   `~/Library/Application Support/Droppy Playground/Droplets/freelance-tracker/FreelanceTracker.droplet`,
   relaunch the Playground, and read its Store row: the subtitle is the loader's verdict.

With the DroppyKit MCP server connected, the same steps are the tools `droppykit_build`,
`droppykit_validate`, `droppykit_shots` and `droppykit_install`, and `droppykit_shots`
returns the images inline. This package carries the server in `.mcp.json` (Claude Code)
and `.cursor/mcp.json` (Cursor). Codex: `codex mcp add droppykit -- /Users/maksim/desktop/droppykit/Scripts/droppykit mcp`.
The other tools are `droppykit_manifest` (a static check, no build), `droppykit_docs`
(the guides and a search over the SDK sources), `droppykit_doctor`, `droppykit_new`,
`droppykit_open_harness` and `droppykit_submit`.

`droppykit run` with no arguments opens the harness window for a person: Droppy's own
Settings panel with a page per surface. You cannot see that window. The shots are your
eyes; take them after every visual change.

`droppykit version` says which SDK checkout the scripts come from and which tag this
package pins; `droppykit update` moves both to the newest release. A build that stops with
"no compiled objects" or "DroppyKit.o not found" is an SDK older than 1.2.1: update it.

## Rules

- **Surfaces and conformances agree.** `surfaces` in `droplet.json` lists what the droplet
  provides; the droplet conforms to the matching protocol for each of them and to nothing it
  does not list. Disagreement is the most common reason a droplet validates and then does
  nothing.
- **Every shelf widget declares both widths.** `preferredSoloWidth` and
  `preferredPairedWidth` are required; Droppy refuses a descriptor that leaves either to a
  host fallback. Solo and paired are different compositions, not one view at two widths:
  branch on `context.isPaired`.
- **Layout traits describe the widget's rectangle.** Every number in
  `ShelfWidgetLayoutTraits` is the area the widget draws in, in points at the Regular shelf
  size, exactly what the harness renders; Droppy adds its own chrome around it. `.fixed(150)`
  is a 150-point rectangle, alone and in a row, clamped to 48 through 480. A widget that needs
  more height declares more; it never pads its way out of a clip. A solo widget is never
  narrower than 352 on a notch or 370 on an island, so lay out to `context.availableSize`.
- **No card, no border around the widget.** Droppy paints nothing behind a widget and almost
  every one of its own widgets lays its content directly on the shelf's black. Put no
  background, fill, outline or rounded box on the widget's root view. `notchSurfaceCardFill`
  is for a tile or a chip inside the widget that has to read as raised, never a frame.
- **Lay the widget out like Droppy's.** The root view fills the rectangle
  (`.frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .topLeading)`) with ONE
  padding, `.padding(context.contentInsets)`, and nothing on top of it. That is the host's own
  inset for the slot and it is ZERO under a notch: the shelf's chrome already inset the
  rectangle, so padding again puts the widget lower and narrower than the built-in beside it.
  Leading text, trailing `.monospacedDigit()` numbers, rows that span the
  full width, a header row of a 12pt symbol and a 12pt semibold title with the widget's control
  at its trailing end, `DroppySpacing` steps between rows. Declare the height the content
  needs; never leave unused space or fill it with padding.
- **The physical notch is cleared for you, except on a HUD strip.** The shelf, a takeover,
  a HUD card and a live activity are inset past the camera housing by the host; never pad for
  it. A HUD strip is handed the whole width across the housing: put its content at the two
  outer edges and nothing in the middle (`host.environment.notchGeometry.closedWidth` is the
  housing), the way the World Clock example's strip does. Never centre a strip.
- **Droppy owns every motion around your surfaces; you animate only what you swap.** A live
  activity row and a HUD are mounted exactly like Droppy's own, so never put an entrance
  transition on the view a factory returns (it plays twice). A live activity is compact only:
  hovering or clicking it opens the shelf, `makeExpanded` is not mounted, and Stop / Start
  belong in the shelf widget. A glyph, label or control you swap while a surface is up takes
  `DroppyTransition.compactContent` in a wing or strip and `DroppyTransition.element` in a
  card, never a transition or spring of your own (`minAPI` 1.8.0).
- **Buttons are Liquid Glass, Droppy's own.** `DroppyCircleButtonStyle` (20pt on an item,
  24pt in a row) for an icon action, `DroppyQuietButtonStyle` and `DroppyAccentButtonStyle`
  (`.small`) for labelled ones, `DroppyGlassButtonStyle` for a label with its own sizing.
  Never a flat wash, a bordered chip or a white button of your own. A list with a control per
  row wraps in `droppyFlatGlassControls()`. Only a live activity row's controls keep
  `DroppyLiveActivityControlStyle`.
- **Everything `activate(host:)` starts, `deactivate()` stops.** Timers, observers, tasks,
  connections. Swift cannot unload code, so anything left running runs until Droppy
  relaunches.
- **A widget the user watches holds the shelf open while it runs.** The shelf closes when the
  pointer leaves it; a teleprompter, countdown or live transcript calls
  `host.shelf.setHoldsOpen(true)` when its work starts and `false` the moment it ends
  (`shelf-write`, `minAPI` 1.5.0). A widget with nothing to show until it is configured sends
  the user to its settings pane with `host.workspace.openSettings()`, and asks for permissions
  there through `host.permissions`, never from the shelf.
- **Host calls are gated by `capabilities`.** A service call without its capability in
  `droplet.json` is refused: it returns `false` or `nil` and logs one line. Declare what you
  use and only that; the user sees the list.
- **The principal class does nothing.** `FreelanceTrackerPrincipal` is `@objc`, is named in the
  bundle's `NSPrincipalClass`, and only creates the droplet. It runs before the host is ready.
- **No `main.swift`.** The harness entry is `@main` in
  `Sources/FreelanceTrackerHarness/FreelanceTrackerHarness.swift`, and a file named `main.swift` cannot
  coexist with `@main`.
- **Look like Droppy, not like a guest.** Surfaces are dark. Foreground colours come from
  `AdaptiveColors`, spacing from `DroppySpacing`, radii from `DroppyRadius` with
  `style: .continuous`. No borders or outlines, no gradients, no ALL-CAPS labels, sentence
  case everywhere, and never paint your own background on a widget. Settings panes are built
  from `DropletSettingsCard`, `DropletControlRow`, `DropletToggleRow`, `DropletStackedRow`
  and `DropletSliderRow`.
- **`droplet.json` is the truth for the build.** `Info.plist` is generated from it.
  `version` is numeric `major.minor.patch`; `summary` is at most 60 characters;
  `minAppVersion` stays `15.3.0` unless the droplet needs something newer; `kit.minAPI` is
  the oldest DroppyKit API the droplet actually calls.
- **Do not edit anything under `/Users/maksim/desktop/droppykit`.** That is the SDK checkout; fixes there go
  upstream. This package is where the work is.

## Where the truth is

Read these before guessing at an API. They are on disk, in the SDK checkout.

- Guides, as Markdown: `/Users/maksim/desktop/droppykit/Sources/DroppyKit/Documentation.docc/`
  `CreateYourFirstDroplet.md`, `DropletSetup.md`, `DesignGuidelines.md`, `ShelfWidgets.md`,
  `LiveActivities.md`, `ExpandedSurfaces.md`, `SettingsPanes.md`, `Icons.md`, `Harness.md`,
  `HostSupport.md`, `Playground.md`, `Submitting.md`, and `BuildWithCodingAgents.md` for this
  workflow in full.
- The surface protocols, one file each: `/Users/maksim/desktop/droppykit/Sources/DroppyKit/Capabilities/`
- The host services a droplet calls: `/Users/maksim/desktop/droppykit/Sources/DroppyKit/Services/DropletServices.swift`
  and `/Users/maksim/desktop/droppykit/Sources/DroppyKit/Core/DropletHost.swift`
- The manifest type, with every field documented: `/Users/maksim/desktop/droppykit/Sources/DroppyKit/Bundle/DropletManifest.swift`
- Design tokens and the settings components: `/Users/maksim/desktop/droppykit/Sources/DroppyKit/DesignSystem/`
- A complete droplet that uses every surface: `/Users/maksim/desktop/droppykit/Examples/WorldClock/`
- The compatibility promise and the version ledger: `/Users/maksim/desktop/droppykit/COMPATIBILITY.md`

## Surfaces

| `surfaces` value in droplet.json | Conform to | Shot |
| --- | --- | --- |
| `shelf-widget` | `ShelfWidgetProviding` | `shelf-widget.png` |
| `live-activity` | `LiveActivityProviding` | `live-activity.png` |
| `expanded-surface` | `ExpandedSurfaceProviding` | `expanded-surface.png` |
| `settings-pane` | `SettingsPaneProviding` | `settings-pane.png` |
| `hud` | `HUDPresenting` | `hud.png` |
| `lock-screen-status` | `LockScreenStatusProviding` | `lock-screen.png` |
| `menu-bar-extra` | `MenuBarExtraProviding` | `menu-bar.png` |

`overview.png` shows the identity card with a verdict pill per surface, `capabilities.png`
the capability switches, `preferences.png` every stored value, and `activity.png` every host
call in order, refused ones marked.

## Done means

- `droppykit build` and `droppykit validate` both pass.
- The report's `problems` is empty and `activation.error` is null.
- You have looked at the shot of every surface you touched.
- The bundle loaded in Droppy Playground: `droppykit_install` says loaded, or the Store row
  shows it switched on.
- `droplet.json` still describes what the code does: surfaces, capabilities, summary.

## Package layout

```
Package.swift                     product FreelanceTracker (dynamic) and FreelanceTrackerHarness
droplet.json                      the manifest; Info.plist is generated from it
Sources/FreelanceTracker/              the droplet
Sources/FreelanceTrackerHarness/       the @main harness entry; never main.swift
FreelanceTracker.icon/                 Icon Composer document, required
Assets/Creator.png                square creator avatar, at least 256px, required
.build/FreelanceTracker.droplet        what droppykit build writes
AGENTS.md, CLAUDE.md, .cursor/    this brief and the agent wiring
.mcp.json, .cursor/mcp.json       the DroppyKit MCP server; absolute paths for this Mac
```

## Submitting

Your own build runs in Droppy only after you approve it under Settings, Store, Local
droplets, and Droppy asks again each time it opens; everyone else gets the droplet from
the Store, signed by Droppy after review.
When the droplet is done: replace the placeholder icon and creator avatar, fill in
`creator` and `source` in `droplet.json`, push the repository, and run `droppykit submit`. It
opens getdroppy.app/submit-droplet with the repository, commit and id filled in.

## Project notes (Freelancy)

- **What it is.** Projects (a client or a contract) with an hourly rate and a currency, and
  pausable sessions inside them. A session banks the seconds of finished stretches in
  `accumulated` and keeps the start date of the running stretch in `activeSince`, so it
  survives a Droppy relaunch. One session runs at a time; starting or resuming one pauses
  every other. `Project.currentSessionID` is the session the timer shows.
- **The flow on the shelf** (`TrackerScreen`): `menu` (Timer, Projects, Analytics, Settings discs;
  Settings calls `host.workspace.openSettings()`) → `timer` (back disc, project badge as tinted glass in
  the project's colour that opens `projects` and grows under the pointer, sessions disc that opens
  `sessions`; pause/resume disc at the clock's left hand, solid new-session disc at its right,
  money under the clock; at the bottom the last five sessions as bars (`SessionBars`, the current
  one bright, growing as it ticks), the session name and the rate) → `projects` / `sessions` (one
  row of 176pt cards that scrolls sideways inside `FadingScroll`, which fades the edge with more
  behind it; a project card (`ProjectCard`) is glass tinted in the project colour: icon and name
  in the colour, the first figure large, then "Earned" or "Time spent" and "Rate" with their
  values; a session card (`ListCard`) is the name, the time large, the money and the day; tap
  picks, hover shows analytics, edit (projects) and delete; the plus is a glass disc) → `newProject` / `editProject` (the project flow, below) → `analytics`
  (`TrackerAnalytics.swift`: project tabs in the nav row, then three pages with dots under them:
  the totals centred, the 14-day time chart, the 14-day money chart; the corner disc writes the
  CSV of the chosen project to `~/Downloads` and reveals it). While a session runs the menu's Timer
  disc is tinted glass in the project's colour with the clock under it. Sessions are named
  `Session N` and renamed in Settings. A menu would drop below the shelf and close it, hence
  carousels and tabs, not pickers. The paired composition is the timer at a smaller scale. Corner
  discs are `navHeight` (28pt).
- **Project flow** (`ProjectFlow.swift`). `startNewProject()` or `edit(id)` fills
  `droplet.draft` (`ProjectDraft`) and shows `.newProject` / `.editProject`; every step is the same
  174pt: nav row, an 80pt body, a button row with "or Press ⏎". Steps: `.name` → `.look` → `.rate`
  → `.currency` (opened from the rate step's top-right pill; `pendingCurrency` holds the choice,
  Confirm applies, Back discards). Back walks the steps and leaves from the first. Steps swap on
  `DroppyTransition.element`. Create / Save is `VividGlassButtonStyle`: the glass chrome tinted in
  the project colour over a stronger fill of the same colour.
- **Keyboard in the flow.** Typing needs Droppy's notch panel (`NotchWindow`) to be the key
  window, and a droplet cannot make it so: `ShelfWidgetFocusPolicy.keyboardFocusable` is recorded
  but has no effect in current hosts (see its doc comment), and while the panel is not key it
  reports `canBecomeKey == false`, so `makeKey()` does nothing. When Droppy has made the panel key
  for its own reasons, the field takes the input at once. Info-level droplet logs never reach the
  unified log; use `host.log.error` for a temporary diagnostic and remove it after. SwiftUI's
  `@FocusState` did not make the field first responder inside the notch panel (the Playground
  logged `noResponder(keyDown:)` beeps while the panel was key), so the flow uses AppKit directly. `FlowTextField` wraps an `NSTextField` (centred text and
  placeholder, Return submits, Escape goes back through the delegate); `FlowKeyReceiver` is a
  zero-hit-test `NSView` that takes first responder on the other steps and turns Return, Escape and
  the arrows into callbacks. Both claim first responder when their window is or becomes key. While
  the flow is up the shelf is held open (`setHoldsOpen`) and the descriptor's `focusPolicy` is
  `.keyboardFocusable`, `.none` otherwise.
- **Wheels.** `InfiniteWheel` + `WheelEngine` draw the icon and currency pickers: an endless ring
  (virtual indices wrap), the centred item chosen, scale and opacity falling with distance, a
  96pt fade at each edge. Input: trackpad and mouse wheel through a local `.scrollWheel` monitor
  that only consumes events while the pointer is over the wheel (snaps 90 ms after the last event,
  a haptic tick per item), drag with momentum from the predicted end, click on any item, arrows.
  The monitor is removed on disappear.
- **Icon.** `ProjectIcon` (16 SF Symbols, all in macOS 14) is stored on the project; a project
  saved before it existed, or one with an unknown value, decodes as the briefcase. It shows on the
  timer badge (solo and paired), on project cards and in the analytics tabs.
- **Name.** The droplet is called Freelancy (`name` in droplet.json, the widget title, the
  activation log line). The id `freelance-tracker`, the Swift product `FreelanceTracker` and the
  file names stay: the id is the preferences namespace and the bundle folder.
- **Confirmations on the shelf.** `droplet.ask(_:)` sets `pendingConfirmation`
  (`.deleteProject`, `.deleteSession`, `.newSession`, `.switchProject`) and the widget shows
  `ConfirmScreen` in
  place of its screen, solo and paired (`compact`), at the same height. Cancel is plain glass;
  Delete is red-tinted glass, Start / Create is `VividGlassButtonStyle` in the project colour. The
  new-session question says whether a running session will be paused (or keeps running when
  "Start new sessions running" is off). `confirmPending()` acts. After a new session,
  `freshSessionID` is set for 2.4 s and `SessionNameLine` shows "✓ Session N started" in the
  project colour on the timer and the paired timer. Picking another project from a card or the
  paired list goes through `requestOpen`, which asks first while a session of a different
  project runs; the running session keeps running. Both questions can be switched off in
  Settings, Shelf ("Ask before starting a new session", "Ask before switching projects"; the
  shelf buttons call `requestNewSession` and `requestOpen`, which read them).
- **Settings pane.** A project row starts with `ProjectLookMenu`: a 28pt superellipse square in the project colour
  with its icon, opening a menu of inline pickers for colour and icon. Deleting a project or a
  session in Settings asks inline (`InlineDeleteConfirmation`: the question, Cancel, Delete) in
  place of the row's controls; a session row also hides its day and length while it asks. The HUD toggle is "Show a strip on start and pause".
- **Session rename.** The pencil on a session card calls `startRenamingSession`; `.renameSession`
  shows `SessionRenameScreen` (same field, nav and "or Press ⏎" row as the project flow, Save in
  the project colour), backed by `sessionRename`. It counts as a typing flow for the shelf hold.
- **Card sizes.** Project cards are 112pt tall with 10pt between name, figure and the bottom row
  and 12pt side padding; session cards stay 90pt with 14pt side padding; 12pt between cards. The
  Projects screen is therefore taller than the timer and the Sessions screen. Hover controls sit
  over the card and never move its content.
- **Start new sessions running.** The `startsSessionsRunning` preference (default on). Off, a new
  session is created paused, nothing else is paused, and no HUD shows.
- **Show me the money.** The `moneyFirst` preference (Settings, Shelf) swaps the two figures on
  the timer, the paired timer, the running menu item and every card: `droplet.ordered(time:money:)`
  hands them back in the chosen order. The notch wings do not swap.
- **Paired.** `PairedView` switches on `screen`: `.projects` is a short list of projects to pick
  from (dot, name, total time) and `.sessions` the same list of the current project's sessions
  (`PairedSessionsView`), both in a `VerticalFadingScroll`; anything else the timer, whose bottom
  row is two 22pt glass capsules: the session (bars and name, a button that opens the sessions
  list and grows under the pointer) and the rate. A pending confirmation replaces any of them. Timer and sessions come to the same `widgetHeight` (140), the
  projects list to 162, and `makeWidgetView` sends a compact mount on any other screen
  back to the timer, so a grouped row never takes the height of a solo-only screen.
- **Analytics.** The nav row is back disc, the project tabs (glass pills as tall as the discs,
  the chosen one tinted in its colour, in a `FadingScroll`), CSV disc. Under it `PagedScroll`:
  full-width pages with a 12pt gap snapping one at a time; the moment a page moves both edges fade
  in (0.12 s) and, once it rests, linger a quarter second and ease away over 0.45 s. Dots under it.
  (16pt gap). Chart pages (`ChartPage`): the period and the total (16pt) on top; 14 capsule bars
  6pt apart, never shorter than their width plus 4pt; an empty day is a dim circle as wide as a
  bar; today's bar at full strength with a dot in the project colour under it, a weekday letter
  under every bar. Hovering a bar dims the rest and puts that day's date and value in the header.
  While the pages scroll, and for 220 ms after, `PagedScroll` sets `analyticsIsPaging` and the bars
  ignore the pointer, so paging does not hover every bar it passes.
  The pages are 112pt, 20pt under the tabs. Figures use
  `.contentTransition(.numericText())` and bars animate on their values, so switching project
  rolls everything to the new numbers. A project card's hover controls are analytics, edit,
  delete; analytics opens on that project through `analyticsProjectID`.
- **Colour.** `ProjectColor` (nine cases) is stored on the project; a project saved before it
  existed, or with an unknown value, decodes as cyan. It paints the badge, the name on a project card, the Sessions title,
  the analytics tab and earnings bars, and the clock in the leading notch wing.
- **Analytics figures.** A session's time is booked to the day it was created on, so a session
  that runs past midnight stays on its first day. The CSV (`TrackerCSV.render`, one project, named
  after it) is one line per session, oldest first: project, session, date, start, duration, decimal hours, rate, currency,
  earned, status.
- **Corners.** The shelf clips the widget's corners, so every screen pads `screenSide` 10,
  `screenTop` 6 and `screenBottom` 8; a 24pt disc then sits outside the curve.
- **Height.** `FreelanceTrackerDroplet.widgetHeight` derives it from the screen; the menu is
  never shorter than the paired timer needs;
  every mutation captures the height before and calls `refreshLayout(from:)`.
- **Digits.** `RollingDigits` renders one character per cell; a changed character slides up
  and blurs out while the new one rises in, on `DroppyAnimation.state`. Used for the notch
  wings and the timer. The clock is the project's colour in the notch; on the timer the clock is
  primary and the money tertiary, as in the sketch.
- **Files.** `TrackerModel.swift` (currency table, icons, colours, project, session, duration parsing),
  `ProjectFlow.swift` (the create and edit flow, `CenterCarousel`),
  `FreelanceTrackerDroplet.swift` (lifecycle, store, screens, CSV export, live activity, HUD,
  descriptors), `TrackerWidget.swift` (every screen but analytics, the shared pieces, the paired
  view), `TrackerAnalytics.swift` (figures, CSV, the analytics screen), `RollingDigits.swift`,
  `TrackerSettings.swift` (defaults, notch toggles, project and session names, rate pills,
  currency menu, session lengths).
- **Capabilities.** `hud` for the strip, `shelf-write` for `invalidateLayout`. Nothing else.
- **Menus in Settings.** A `Menu` with `.menuStyle(.button)` and `.buttonStyle(.plain)` keeps
  the label's glass chrome; `.borderlessButton` paints the label accent blue with no chrome.
- **Shots of other screens.** The harness renders the start screen only; to see another,
  temporarily change the `screen = .menu` line in `activate` (the one followed by `now = Date()`)
  to that case, and for `editProject` also set `editingProjectID`. Never sed every
  `screen = .x` line: `open`, `createProject`, `newSession` and `select` set `.timer` at the same
  indent, and rewriting those sends every action back to the menu.
- **Tests.** `Tests/FreelanceTrackerTests` covers the clock strings and duration parsing, money
  formatting, decoding projects saved by older versions, session durations and the CSV. Run
  `swift test` with `DEVELOPER_DIR` set as below; the droplet build ignores the test target.
- **Toolchain.** The SDK needs Swift 6.2 (Xcode 26+). `xcode-select` here points at Command
  Line Tools (Swift 6.0.3), so `.mcp.json` and `.cursor/mcp.json` pass
  `DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer`; export the same in a shell before
  `droppykit …`. `sudo xcodebuild -runFirstLaunch` has been run, so actool compiles the icon
  into `Assets.car` and an `.icns`; if a build log ever says it could not, run it again.
- **Demo data.** In the harness only (`host.environment.isHarness`), three projects with a
  few sessions are seeded so every shot shows something. Droppy and the Playground start empty.
