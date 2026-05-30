# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Home Layout Planner — a single-page web app for placing real-world-scaled furniture on top of an uploaded floor plan image. The user uploads an image, calibrates pixels-per-inch by clicking two endpoints of a known distance, then drags furniture from the catalog onto an SVG canvas overlaid on the image. Layouts can be exported/imported as JSON (see `floor-plan-*.json` for the schema).

## How to run

There is no build system, no `package.json`, and no test suite. Open `index.html` in a browser (or serve the directory with any static server, e.g. `python3 -m http.server`). React 18 UMD and `@babel/standalone` are loaded from unpkg and compile the inline JSX at runtime.

## Architecture

All application code and CSS live in `index.html`. The JSX is in a single `<script type="text/babel">` block (starting around line 611) compiled at runtime by `@babel/standalone`.

The three logical modules, in the order they appear inside the inline script:
1. **Catalog** (`CATALOG`, `CAT_ORDER`) — every furniture item with real-world dimensions in inches and a `kind` that selects its SVG renderer.
2. **Furniture shapes** (`Bed`, `Sofa`, `Sectional`, …, `FurnitureShape`) — pure SVG components drawn in the item's own inch-space coordinate system; `FurnitureShape` dispatches on `item.kind`.
3. **App** (`App`, `Sidebar`, `UploadZone`, `Placed`, `ScaleModal`) — the planner UI and all interaction state.

## Coordinate systems

Three coordinate spaces are in play and conversions between them are central to almost every interaction:

- **Image-pixel space** — the SVG viewBox matches the uploaded image's natural pixel dimensions (`planSize.w/h`). All placed-item positions (`cx`, `cy`) and the calibration endpoints live here.
- **Screen-pixel space** — actual rendered pixels. `screenToSvg` / `svgToScreen` (in `App`) convert via the SVG element's `getBoundingClientRect()`.
- **Real-world inches** — furniture `w` and `h` from the catalog. `pxPerInch` (image-pixels per real-world-inch) is the bridge; it starts as a width-based guess on upload and is replaced by the user's calibration (click two points → enter the real distance in the `ScaleModal`).

Resizing, rotation snapping (Shift = 15°), and the on-canvas dimension tag all rely on these conversions — when changing drag/resize logic, be careful which space each value is in.

## Persistence

`exportPlan` / `importPlan` (in `App`) serialize `{ planUrl (data URL), planSize, pxPerInch, placed }` to JSON. The included `floor-plan-2026-05-26-05-42.json` is a real example and is the source of truth for the schema; preserve field names when changing it.
