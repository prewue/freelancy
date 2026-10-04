# Freelancy, a Droplet for Droppy

<!-- Written by `droppykit agent`. Add your own notes below; the file is only rewritten with --force. -->

This package is a **Droplet**: an extension that runs inside Droppy, the
Dynamic Island and shelf for Mac, written in SwiftUI against **DroppyKit**.
Droppy loads the built `.droplet` bundle into its own process and draws it on
the notch, the shelf, the lock screen and the menu bar.

- Droplet id: `freelance-tracker`. It is also `FreelanceTrackerDroplet.id` in Swift and `id` in `droplet.json`; the three must agree or the loader refuses the bundle.
- Swift product: `FreelanceTracker`, a dynamic library. The harness target is `FreelanceTrackerHarness`.
- SDK checkout: `/Users/maksim/desktop/droppykit` (DroppyKit 1.20.1). Docs online: https://getdroppy.app/docs/droppykit
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
  padding, `.padding(context.contentInsets)`, and nothing on top of it. It clears the host's
  rounded clip: 8 on every edge under the 18pt card corner (a notch, and any row on either
  shape), 0 alone on the island, where the island shell clips the card and keeps one even gap.
  The shelf's chrome already inset the rectangle, so never add your own padding or corner
  inset on top: it doubles the gap.
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
  case everywhere, and never paint your own background on a widget. Never repeat the page
  header in a settings pane: Droppy already shows the droplet's icon, name, summary and maker
  above it, so the pane starts with its first setting (the Store refuses a pane that shows the
  droplet's own icon or its name as a title). Settings panes are rooted
  in `DropletSettingsPane` (`minAPI` 1.9.0) and built from `DropletSettingsCard`,
  `DropletSettingsSection`, `DropletControlRow`, `DropletToggleRow`, `DropletStackedRow` and
  `DropletSliderRow`; the host mounts the pane in its native grouped form, so the pane's
  content is a list of sections, never a `VStack`, and it places no dividers.
- **`droplet.json` is the truth for the build.** `Info.plist` is generated from it.
  `version` is numeric `major.minor.patch`; `summary` is at most 60 characters;
  `minAppVersion` stays `15.3.0` unless the droplet needs something newer; `kit.minAPI` is
  the oldest DroppyKit API the droplet actually calls. `droppykit build` fails when it is
  older than that and names the calls and the version to set; set that version, never
  lower it to get a droplet loaded.
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
`creator` and `source` in `droplet.json`, write `CHANGELOG.md`, and run `droppykit submit`.
The Store is a repository (gitlab.com/droppyformac1/droplets, one folder per droplet):
`submit` forks it, copies this package into `droplets/<id>/`, pushes a branch and opens
the merge request; when a maintainer merges it, the pipeline builds, signs and publishes
it. `droppykit submit --print` shows the plan without doing it (needs `glab`, signed in).
Opening the merge request means accepting the Droplet developer terms at
https://getdroppy.app/droplet-developer-terms.

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
  174pt: nav row, an 80pt body, a button row with "or press ⏎". Steps: `.name` → `.look` → `.rate`
  → `.currency` (opened from the rate step's top-right pill; `pendingCurrency` holds the choice,
  Confirm applies, Back discards). `.look` is the icon wheel alone, centred; its nav row ends in
  a "Color" pill (`DroppyGlassButtonStyle(fill: .tinted(colour))`; the title stays centred because
  `FlowNav` lays it over the row in a `ZStack`) that opens `.color`, "Choose color", the same way:
  `pendingColor` holds the choice, Confirm applies, Back discards, back to `.look`. `ColorStep`
  is the colour wheel (`WheelStyle.colors`), each colour a plain disc (`ColorOrb`). Back walks the steps and leaves from the first. Steps swap on
  `DroppyTransition.element`. `.rate` (`RateStep`) is the amount in the project's colour at 26pt
  (click it to type an exact one) over a `RulerWheel` (whole amounts 1 to 500; arrows step 1,
  Shift 10; a typed amount glides the ruler to it). `.currency` (`CurrencyStep`) has a search in
  the nav row (`NavSearch`: a 28pt glass disc that opens across the row in beats: the title fades
  out, the disc springs open on `DroppyAnimation.tabSelect` (the review of
  !41 asked for a preset; it is the nearest to the 0.46 / 0.8 spring it replaced) with the magnifier riding from the middle to the leading edge (leading-aligned,
  centred in the disc by its padding), then 260 ms in a SwiftUI placeholder and the close
  button come in out of a blur over a left-aligned `FlowTextField`; closing runs backwards; it
  matches code, name and symbol; Return confirms, Escape closes) and shows its matches as a
  bounded row (`wraps: false`); the codes are text only and pass under one glass lens held in
  the middle, so the selection moves with the scroll. Create / Save is plain glass
  (`DroppyGlassButtonStyle()`); the colour step's Confirm is tinted glass in the chosen colour. The
  review of !41 asked for Droppy's own tinted glass on every coloured button (a solid fill under
  the glass, `VividGlassButtonStyle`, was removed) and for "color" in every visible string.
- **Start screen** (`StartScreen.swift`). With no projects the menu is `StartScreen` instead:
  `StartEmblem` (the logo with its long hand turning once every six seconds and glass pills of
  earnings, `+$124` and so on, and one "By 8LX" tinted in the logo's blue, all `droppyGlassControlChrome` with no fill of our own, orbiting it on a tilted ring,
  behind and in front of it, on a `TimelineView`), "Welcome to Freelancy" (20pt), two lines
  (13pt, 5pt line spacing, the secondary text colour; the review of !41 asked for no gradient), and "Create
  project" as a plain `DroppyGlassButtonStyle` button, which starts the project flow. The review
  of !41 asked for no effect on the button (an animated beam around it was removed) and for the
  logo to carry the welcome. `widgetHeight` gives the menu `startScreenHeight` (195) then.
- **Colours on the shelf.** The shelf can be light (the island follows the appearance), so
  nothing on it is hard-coded white: text uses `AdaptiveColors.notchSurface*Text`, fills
  `notchSurfaceCardFill` (the tokens resolve in Droppy's own DroppyKit at run time). The only
  whites left are the logo's own artwork on its opaque blue squircle, like the icon.
- **Logo and icon.** The artwork was drawn in Figma (`app-logo.svg`, kept out of the package so
  it does not ship to the Store). The droplet icon (`FreelanceTracker.icon`) follows Droppy's
  icon language (review of !41): the `system-dark` tile, like Teleprompter and Thaw, with three
  clean SVG layers made from the artwork (actool cannot draw the Figma export's `foreignObject`
  conic gradients or its blur filter): `face` (the dial in the accent #5B3FFF, a light-to-deep
  gradient, glass on), `dots` (sixteen white dots with the conic's opacity baked in per dot:
  0.98 at twelve, 0.15 just past it, rising to 0.92 round the dial) and `hands` (white, glass
  on). The welcome screen's `FreelancyLogo` still draws the earlier blue tile artwork in SwiftUI
  from the SVG's own measures on its 1224 canvas, so the long hand can turn; `FreelancyLogo.light`
  is that conic, taken relative to the hand, so the lit dots trail it.
- **Keyboard in the flow.** Typing needs Droppy's notch panel (`NotchWindow`) to be the key
  window, and only the host can make it so (DroppyKit 1.10.0, hence `minAPI` 1.10.0; hosts on
  1.8.1, such as Playground 1.0.20, refuse the bundle). `FlowFocus.ensureKey` calls
  `host.shelf.requestKeyboardFocus()` through `FlowFocus.requestKey` (set in `activate`, cleared in
  `deactivate`) and never `makeKey()`; the request is refused unless the descriptor currently says
  `.keyboardFocusable`, so the views retry on `FlowFocus.retryDelays` while the host re-reads the
  descriptor. Once granted, the panel stays eligible while the widget is on the shelf. Info-level droplet logs never reach the
  unified log; use `host.log.error` for a temporary diagnostic and remove it after. SwiftUI's
  `@FocusState` did not make the field first responder inside the notch panel (the Playground
  logged `noResponder(keyDown:)` beeps while the panel was key), so the flow uses AppKit directly. `FlowTextField` wraps an `NSTextField` (centred text and
  placeholder, Return submits, Escape goes back through the delegate); `FlowKeyReceiver` is a
  zero-hit-test `NSView` on the other steps that turns Return, Escape and the arrows into
  callbacks through a local `.keyDown` monitor, not as first responder: a click on a step (a
  colour, an icon, the slider) hands first responder to the SwiftUI hosting view, and when the
  receiver had to win it back Return died once its retries ran out. Only the newest receiver
  answers (two steps overlap while they swap), and a field editor keeps its own keys. Both still
  claim first responder when their window is or becomes key, which is what asks for focus. While
  the flow is up the shelf is held open (`setHoldsOpen`) and the descriptor's `focusPolicy` is
  `.keyboardFocusable`, `.none` otherwise.
- **Wheels** (`Wheel.swift`). One system for every picker on the shelf. `WheelEngine.wraps` is an
  endless ring when true and a row that stops at its ends when false (clamped, rubber-banding
  under a drag); `InfiniteWheel(wraps:)` passes it and re-points the engine when its items change
  (a search narrowing them). `RulerWheel` is a bounded engine drawn as tick marks: the middle tick
  tall and tinted, two either side tinted fainter at the grey ticks' height, heights and tint on smooth curves of the
  distance, with the same fade and edge blur. An `InfiniteWheel` driven
  by a `WheelEngine`, drawn to a `WheelStyle` preset (`.icons`, `.colors`, `.currencies`), edges
  softened by `ProgressiveEdgeBlur`. A new wheel adds a preset (pitch, height, edge blur and its
  band, fade) and brings its own cell; the curves are shared and must stay so, the wheels are
  meant to feel identical. An endless ring (virtual indices wrap, six items a side), the centred
  item chosen. Every curve is smooth through the ring: the row gathers on a `tanh` (full pitch at
  the centre), size and opacity fall on a Gaussian (flat at the top, so the centre item eases
  rather than pops); an earlier piecewise squeeze changed an item's speed three steps out, which
  read as items coasting in and then starting to scale. `ProgressiveEdgeBlur` is a blur by screen
  position from public SwiftUI: the wheel drawn four times (sharp and three rising radii), each
  copy masked to a triangular weight across the edge band and added with `plusLighter`, so the
  weights sum to one and the blur changes continuously with x; only the sharp copy is
  hit-testable or visible to VoiceOver. There is no public variable blur (SDK 27 included), and a
  Metal `layerEffect` would need a metallib, which `droppykit build` does not compile and the Store
  does not accept prebuilt. It is expensive, so it draws as little as it can: each blurred copy
  draws only the items that reach into its part of the band (`EdgeBlurCopy.reach`,
  `InfiniteWheel.virtuals`), and a cell reads `isBlurredCopy` to draw a cheap face there (a
  currency pill drops its glass; glass in every copy cost most of the frame rate). The dark edge
  fade is 56pt; the blur does the softening. The harness draws neither blur nor glass; look in
  the Playground. A click on an item more than one away glides there on an ease-out as long as
  the gap (up to 0.75 s) instead of the settle spring, which leapt. Input: trackpad and mouse
  wheel through a local `.scrollWheel` monitor that only consumes events while the pointer is
  over the wheel (snaps 90 ms after the last event, a haptic tick per item), drag with momentum
  from the predicted end, click on any item, arrows. The monitor is removed on disappear. Colours
  are plain 44pt discs (`ColorOrb`), currencies 64 by 32 glass pills (`CurrencyPill`).
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
  Delete is red-tinted glass, Start / Create is `DroppyGlassButtonStyle(fill: .tinted(...))` in the project colour. The
  new-session question says whether a running session will be paused (or keeps running when
  "Start new sessions running" is off). `confirmPending()` acts. After a new session,
  `freshSessionID` is set for 2.4 s and `SessionNameLine` shows "✓ Session N started" in the
  project colour on the timer and the paired timer. Picking another project from a card or the
  paired list goes through `requestOpenProject`, which asks first while a session of a different
  project runs; the running session keeps running. Both questions can be switched off in
  Settings, Shelf ("Ask before starting a new session", "Ask before switching projects"; the
  shelf buttons call `requestNewSession` and `requestOpenProject`, which read them).
- **Settings pane.** The root is `DropletSettingsPane`, every group a `DropletSettingsSection`
  over a `DropletSettingsCard`, no dividers: on macOS 15+ Droppy mounts it in its grouped Form
  and every row is a Form row (macOS 14 gets the legacy cards, so the custom project and
  session rows pad themselves there through `settingsRowPadding()`). Currency is a native
  `Picker` (`CurrencyPicker`), rate and a session's length are native rounded-border fields
  (`ValueField`: Return or focus loss commits, Escape puts the stored value back), and text
  fields take `.labelsHidden()`, or the Form turns their title into a row label. Sessions of
  the expanded project are rows of their own, indented under its name. A project row starts with `ProjectLookMenu`: a 28pt superellipse square in the project colour
  with its icon, opening a menu of inline pickers for colour and icon. Deleting a project or a
  session in Settings asks inline (`InlineDeleteConfirmation`: the question, Cancel, Delete) in
  place of the row's controls; a session row also hides its day and length while it asks. The HUD toggle is "Show a strip on start and pause".
- **Session rename and naming.** The pencil on a session card calls `startRenamingSession`;
  `.renameSession` shows `SessionRenameScreen` (same field, nav and "or press ⏎" row as the
  project flow, Save in the project colour), backed by `sessionRename`. "Name new sessions" (the
  `namesNewSessions` preference, default on) sends the plus to the same page first
  (`startNewSession` → `startNamingNewSession`, `sessionRename.sessionID` `nil`): the
  placeholder is the default name (`defaultSessionName`), the button says Start, an empty field
  keeps the default, and Back returns to the timer or Sessions (`returnScreen`). With "Ask
  before starting a new session" on (default now off) the page follows the question. A grouped
  row has no room for it, so there the session starts at once; Start on a project with no
  session also starts at once. The page counts as a typing flow for the shelf hold.
- **Rates.** Each `TrackerCurrency` has a `rateStep` (1 for dollars and euros up to 1000 for won)
  so the ruler's 500 steps and the Settings slider cover a normal hourly rate in any currency; an
  exact rate can always be typed.
- **Card sizes.** Projects and Sessions come to the timer's height (`cardRowHeight` 100 for the
  row, `timerRowHeight` 68 for the timer's clock row: all three 152), so moving between them
  never resizes the shelf. `CardRow` and the cards take the height the screen has left and
  spread their three rows with flexible gaps, so a solo island card, which the host does not
  inset, just gets roomier cards. Project cards pad 10 by 12 with at least 6pt between name,
  figure and bottom row; session cards pad 8 by 12; 12pt between cards. Hover controls sit over
  the card and never move its content.
- **Start new sessions running.** The `startsSessionsRunning` preference (default on). Off, a new
  session is created paused and no HUD shows. A new session always stops the one running before
  it, either way. Start on a project with no
  session still starts one running (`newSession(on:running: true)`): Start means start.
- **Show me the money.** The `moneyFirst` preference (Settings, Shelf) swaps the two figures on
  the timer, the paired timer, the running menu item and every card: `droplet.ordered(time:money:)`
  hands them back in the chosen order, and it swaps the notch row's two figures too.
- **Paired.** `PairedView` switches on `screen`: `.projects` is a short list of projects to pick
  from (dot, name, total time) and `.sessions` the same list of the current project's sessions
  (`PairedSessionsView`), both in a `VerticalFadingScroll`; anything else the timer, whose clock shrinks with its length (`PairedTimerView.clockSize`: 22pt
  for `00:00` down to 16 for `120:45:21`, so it fits between the discs) and whose bottom
  row is two 22pt glass capsules: the session (bars and name, a button that opens the sessions
  list and grows under the pointer) and the rate. A pending confirmation replaces any of them. The descriptor has ONE height for
  both compositions (a grouped row honours the solo height, per the SDK), so the widget
  remembers how Droppy last mounted it (`isInGroupedRow`, set in `makeWidgetView`, never in the
  harness or a preview) and, grouped, declares `groupedHeight` (110: the paired timer's 94 and
  the host's insets), which the paired timer, lists, empty state and a compact question all fit.
  Declaring the solo screen's height there made a grouped row as tall as the start screen. A
  grouped mount with projects sends a solo-only screen back to the timer; with no projects it
  leaves the start screen alone (it used to send it to `.timer`, 142pt, which drew the start
  screen cut off until the flow's Back restored `.menu`).
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
- **Notch row (live activity).** Published only while a session runs; a paused session gives the
  notch back (review of !41), and it is resumed from the shelf widget. Not interactive: Droppy
  opens the shelf on hover or click and the row gets no pointer events, so no button, hold or
  hover action can work there; Droppy 16.0.1 never reads `preferredWidth`, so the wings keep the
  host's fixed widths. Both wings are `CompactFigureView`: the amount subtle on the left (primary
  text at 45%) and the time in the project's colour on the right; "Show me the money"
  (`moneyFirst`) swaps them. Same type (12pt medium rounded), each centred in its wing.
- **Colour.** `ProjectColor` (nine cases) is stored on the project; a project saved before it
  existed, or with an unknown value, decodes as cyan. It paints the badge, the name on a project card, the Sessions title,
  the analytics tab and earnings bars, and the time in the notch row's trailing wing.
- **Analytics figures.** A session's time is booked to the day it was created on, so a session
  that runs past midnight stays on its first day. The CSV (`TrackerCSV.render`, one project, named
  after it) is one line per session, oldest first: project, session, date, start, duration, decimal hours, rate, currency,
  earned, status.
- **Corners and insets.** The root pads `context.contentInsets` once and nothing else: the
  host's clearance for its clip, 8 on every edge under the 18pt card corner (a notch, any
  grouped row) and 0 on a solo island card (DroppyKit 1.20.1, Droppy 16.0.1, Playground
  1.0.27; older hosts hand 0 and 12). There is no screen padding of our own any more (the
  review of !41 asked for exactly that). `widgetHeight` budgets `hostInsets`, the 16 the
  corner takes.
- **Height.** `FreelanceTrackerDroplet.widgetHeight` derives it from the screen (or is
  `groupedHeight` in a grouped row; a timer with no project is the start screen's height);
  every change goes to the host through `publishLayoutChange()`: the
  `ShelfWidgetDynamicLayoutProviding` publisher (hosts on DroppyKit 1.20.0+ also call
  `currentLayoutTraits` when they size the shelf, so a reopened shelf never uses a stale height;
  the protocol dates from 1.0.0, no `minAPI` change) and `invalidateLayout` for older hosts; the menu is
  never shorter than the paired timer needs;
  every mutation captures the height before and calls `refreshLayout(from:)`.
- **Digits.** `RollingDigits` renders one character per cell; a changed character slides up
  and blurs out while the new one rises in, on `DroppyAnimation.state`. Used for the notch
  wings and the timer. The clock is the project's colour in the notch; on the timer the clock is
  primary and the money tertiary, as in the sketch.
- **Files.** `TrackerModel.swift` (currency table, icons, colours, project, session, duration parsing),
  `ProjectFlow.swift` (the create and edit flow and its steps), `Wheel.swift` (the wheel system),
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
- **Tests.** `Tests/` is a package of its own (`Tests/Package.swift`, the droplet by path `..`),
  because the Store refuses a `.testTarget` in a droplet's Package.swift and a `Tests/` folder in
  its droplet folder. `Tests/Tests/FreelanceTrackerTests` covers the clock strings and duration
  parsing, money formatting, decoding projects saved by older versions, session durations and the
  CSV. Run `swift test` from `Tests/` with `DEVELOPER_DIR` set as below. `.gitignore` lists
  `Tests/`, `Package.resolved` and `.claude/`: `droppykit submit` copies the whole package into the
  Store checkout and stages only what `.gitignore` lets through, so they stay out of the Store. The
  test files are tracked here anyway; add a new one with `git add -f`.
- **Toolchain.** The SDK needs Swift 6.2 (Xcode 26+). `xcode-select` here points at Command
  Line Tools (Swift 6.0.3), so `.mcp.json` and `.cursor/mcp.json` pass
  `DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer`; export the same in a shell before
  `droppykit …`. `sudo xcodebuild -runFirstLaunch` has been run, so actool compiles the icon
  into `Assets.car` and an `.icns`; if a build log ever says it could not, run it again.
- **Demo data.** In the harness only (`host.environment.isHarness`), three projects with a
  few sessions are seeded so every shot shows something. Droppy and the Playground start empty.
