# Role Calibration Tool — Design

## Purpose

A facilitation artifact for group role calibration. When a hiring group has been talking past each other about what a role needs to optimize for, this tool has each participant privately allocate a fixed budget of points across a set of weighted dimensions, then overlays every allocation on a single radial chart so the group can see convergence and divergence at a glance.

It is not a candidate scoring tool, HR platform, or team product. It's a single artifact you bring into a short facilitation meeting once per hiring decision.

## Non-goals

The following are explicitly out of scope. Any of them can be added later without a rewrite.

- Candidate scoring (may be bolted on as a second mode later if useful)
- Multi-session management, saved history, backend storage
- Real-time sync between participants
- Authentication, user accounts, analytics
- Mobile layout
- Server-side anything

## Users and flow

Three roles during a meeting.

**Organizer.** Opens the tool, switches to Configure mode, fills in the role title, a short framing paragraph for participants, the point budget, and the dimensions (id, label, hint for each). Clicks "Save & generate share URL." The tool encodes the configuration into the URL hash and surfaces a shareable URL. The organizer copies that URL and shares it with the group.

**Participants.** Open the shared URL. The tool reads the config from the URL hash and drops them into Allocate mode. They privately distribute their points, copy a short human-readable code, and paste it into the group chat (Google Meet, Slack, etc.).

**Facilitator.** Switches the same page to Compare mode, pastes every line from the chat into a textarea, and renders an overlaid radial chart. One person can be organizer and facilitator, or two different people, or everyone.

## The three modes

### Configure mode

A form with:

- Role title (text)
- Framing paragraph for participants (textarea)
- Point budget (number, default 20)
- Dimensions list, each row with:
  - Short ID (1–2 letters, used in the copy code)
  - Label
  - Hint textarea (1–2 sentences describing the dimension)
- "Add dimension" button (max 8 dimensions)
- "Save & generate share URL" button — validates and, on success, encodes the config into the URL hash and reveals a share banner
- "Reset" button

Share banner shows the full URL and has Copy / "Try it in Allocate mode" buttons.

### Allocate mode

What the participant sees when they open a shared URL:

- Role title and framing (from the decoded config)
- The dimensions as a list, each with label, hint, and a number input (0 to budget)
- Live "Remaining: X / budget" counter
- Live radial chart showing their current allocation as a single polygon
- Optional name text field
- "Copy my code" button, enabled only when the total equals the budget exactly

On copy, the clipboard receives a line like `Name: A4 B6 C4 D3 E3`. Button text briefly changes to "Copied ✓" and reverts after 2 seconds.

### Compare mode

What the facilitator uses during the meeting:

- A textarea for pasting all lines from the group chat, one per line
- "Render" button
- A radial chart with one colored polygon per submission, overlaid on the same axes, plus a dashed "group average" polygon
- A legend with name and color swatch
- A stats table showing per-dimension average, min, max, standard deviation
- A "Clear" button

If the URL has no config loaded, both Allocate and Compare modes show an empty state pointing the user at Configure mode.

## Data model

### Config structure

The config lives in memory as a plain object and is encoded into the URL hash for sharing:

```js
{
  role: {
    title: "...",
    framing: "..."
  },
  budget: 20,
  dimensions: [
    { id: "A", label: "...", hint: "..." },
    { id: "B", label: "...", hint: "..." },
    ...
  ]
}
```

### URL hash encoding

`#c=<url-encoded-JSON>`. Simple `encodeURIComponent(JSON.stringify(config))`. No compression, no base64. Human-readable in the URL bar, trivially shareable, and small enough for typical configs to fit well within browser URL length limits.

### Code format

`Name: A4 B6 C4 D3 E3`

- Name is optional. If the `:` is missing, the parser uses `Anon 1`, `Anon 2`, etc.
- Dimension letters match the `id` fields in the current config exactly. Order-independent.
- Numbers are 1–2 digit non-negative integers.
- Whitespace-tolerant.
- Validation on parse: allocation must sum to the configured budget, must include every dimension id, must not include unknown ids. Invalid lines surface as inline errors in Compare mode.

Why this format over JSON or URL hash: it's readable at a glance in a chat window, people can visually react to each other's allocations while they're being pasted, and it survives copy-paste robustly.

## Visual design

Editorial aesthetic, not a dashboard.

- Warm off-white background
- Serif headline (system serif stack), sans-serif body (system UI stack), no custom fonts or CDNs
- One accent color (warm terracotta)
- Single centered column, ~640px max width
- Radial chart as hand-drawn inline SVG, no charting library
- Mode switcher in the top-right corner
- No animations beyond the chart smoothly updating as values change

## Architecture

Single static `index.html` file with:

- Inline `<style>` block
- Inline `<script>` block with:
  - Pure helpers for config encode/decode, code parse/serialize, stats compute
  - SVG chart renderer
  - Mode-switching logic
  - Form wiring for Configure, Allocate, Compare
- No build step, no package.json, no dependencies
- Works from `file://` for local testing or any static file server

## Deployment

Published as a GitHub Pages static site. The same file serves every role — configuration happens in the browser at meeting time.

## Error handling

- **Configure:** "Save" button validates. Errors surface in a list above the share banner (missing title, duplicate dimension IDs, invalid ID format, too few/too many dimensions).
- **Allocate:** "Copy" button disabled unless the sum equals the budget exactly. Live counter turns red when over budget and terracotta when at budget.
- **Compare:** invalid lines surface as inline errors ("Line 3 couldn't be parsed: ...") above the chart but don't block valid lines from rendering. Unknown tokens and missing dimensions both produce a parse error for the line.
- **Clipboard API failure:** if `navigator.clipboard.writeText` is unavailable, fall back to a visible text box with the code selected so the participant can copy manually.

## Testing

Manual verification through a real browser. No unit tests — the tool is small enough that a single manual pass through all three modes catches any real issue. Pure parse and serialize functions could be tested later if the tool grows.

## Reusability

Intentionally simple: the tool ships as a generic calibration runner, and the organizer configures it per-meeting through the Configure mode. The URL hash carries the configuration, so there is no commit-and-push loop when running a new exercise. One tool, many meetings.

## Open questions resolved during design

- **Pre-meeting vs. in-meeting fill:** in-meeting.
- **Backend or not:** no backend. URL hash carries the config; codes are pasted via the group chat.
- **Candidate scoring in the same tool:** not in v1. Bolt-on-able later as a second mode if it proves useful.
- **Dimension count:** configurable, 2 to 8.
- **Code format:** human-readable shorthand, not JSON or base64.
- **Reusability mechanism:** URL-encoded config in the hash, one generic tool, no per-role files in the repo.
