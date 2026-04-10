# Implementation Plan

Scoped from `design.md`. Single-file HTML tool, no build step, no dependencies.

## Build order

1. **`index.html` skeleton** — doctype, meta viewport, `<header>`, `<main>` with three mode sections, `<footer>`
2. **`<style>` block** — editorial CSS, warm off-white background, serif headline, sans-serif body, one accent color, centered 640px column, styling for all three modes
3. **Helpers** — `clearNode`, `encodeConfigToHash`, `decodeConfigFromHash`, `buildShareUrl`, `parseCode`, `serializeCode`, `computeStats`
4. **SVG chart renderer** — `renderChart(svg, submissions)` that reads from `currentConfig`, clears the SVG, draws concentric gridlines, axes, labels, and one polygon per submission
5. **Configure mode UI** — form with title, framing, budget, dimension rows (id/label/hint), add/remove dimension, Save, Reset. Save validates and encodes into URL hash, then reveals a share banner.
6. **Allocate mode UI** — empty state when no config loaded. Otherwise renders dimension inputs, live counter, live chart, name field, Copy button with clipboard fallback.
7. **Compare mode UI** — empty state when no config loaded. Otherwise textarea, Render button, error list, overlaid chart, legend, stats table.
8. **Mode switcher** — three-button toggle in the header; clicking switches body class
9. **URL hash wiring** — on load, check for `#c=...`, decode, apply config, default to Allocate mode if present, Configure mode if absent. Listen for `hashchange` in case someone pastes a new URL.

## Test before push

- Serve the repo locally (`python3 -m http.server 8765`), load in a browser or via playwright
- Configure mode: fill in title, framing, 3 dimensions, save, verify share banner appears and URL hash updates
- Allocate mode: verify counter, copy-disable logic, chart updates live, copy button works
- Compare mode: paste sample codes with a mix of valid and invalid lines, verify errors surface inline and valid lines still render
- Hash round-trip: reload the page with the share URL and verify it lands in Allocate mode with the right config
- Check browser console for errors

## Git + deploy

1. `git init -b main`, set local user.name/user.email to the right GitHub identity
2. Commit the generic tool (no hardcoded role data anywhere)
3. `gh repo create <owner>/role-calibration --public --source . --remote origin --push`
4. Enable GitHub Pages on `main` at root
5. Verify the deploy URL loads the empty Configure state

## Out of build scope

- Unit tests (tool is small enough for manual verification)
- Error telemetry, analytics, or logging
- Accessibility beyond native form elements (defer to a future pass)
- Favicon (can add later)
