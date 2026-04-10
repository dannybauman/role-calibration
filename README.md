# Role Calibration

A small tool for figuring out what a role actually needs to optimize for, when the hiring group has been talking past each other about it.

Everyone privately allocates a fixed budget of points across a handful of dimensions, then the facilitator overlays all the allocations on a radial chart. You see convergence and divergence at a glance, and the conversation gets a lot more productive.

Built for a specific meeting and designed to be reused for the next one.

**Live tool:** <https://dannybauman.github.io/role-calibration/>

## How it works

You run it in two modes. Everyone opens the same page, and the facilitator flips a small link in the top-right corner to switch between them.

**Allocate mode.** Each participant sees the role, the dimensions, and a live radial chart that updates as they type. They distribute their points (20 by default), optionally add their name, and hit "Copy my code." They paste the result into the Google Meet chat.

**Compare mode.** The facilitator switches modes, pastes every line from the chat into a textarea, hits Render. All submissions show up as overlaid polygons with a group-average outline and a stats table underneath. You spot the tension instantly: where does everyone agree, where does one person weight something differently, what's the implicit disagreement nobody's been naming.

The exercise takes about 5 minutes. The conversation it unlocks takes the rest of the meeting.

## Code format

```
Name: T4 P6 R4 C3 V3
```

Name is optional. If you skip it, the tool labels you `Anon 1`, `Anon 2`, and so on. Dimension letters match the `id` fields in the config, order doesn't matter, and the total has to hit the budget. Whitespace is forgiving.

## Reusing for a new role

The whole point of this was to not build it again every time. To run the exercise for a different role:

1. Copy `index.html` to a new file. Call it whatever, e.g. `security-lead.html`.
2. Edit the `CONFIG` block at the top of the `<script>` tag. Change the title, the framing sentence, and the 5 dimensions. Keep the `id` letters short since they show up in the copy codes.
3. Commit and push. GitHub Pages will serve it at `https://dannybauman.github.io/role-calibration/<your-file>.html`.

That's the reuse model. No config files, no admin UI, no fancy abstraction. Edit, commit, share the link.

## Running it locally

No build step, no dependencies. Open `index.html` in a browser directly, or serve the folder:

```
cd role-calibration
python3 -m http.server 8765
```

Then open <http://localhost:8765/>.

## Design notes

`docs/design.md` has the full rationale: why this is one HTML file instead of a React app, why the code format is a readable shorthand instead of JSON or a URL hash, and what's intentionally left out. Short version: the tool exists to run once for six people in half an hour, and then get picked up again six months later when the next hire comes up. That's the shape.

## What it doesn't do

Candidate scoring, backend storage, real-time sync, authentication, mobile layout. If those ever become useful, they can be added without rewriting the chart or the core parsing. Until then they're out of scope.
