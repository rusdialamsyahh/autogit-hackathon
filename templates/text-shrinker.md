---
title: Text Shrinker
app_type: text-shrinker
wallet: 0x20615cb29177Ff5B858da25730a94b4A4A6Ef812
---

▸ what it is ▸

A pasteboard for prose that shows you, in real time, how much smaller the same words become under three different compression formats. Drop a paragraph, a JSON blob, a transcript. The pane tallies bytes in, bytes out, ratio, encode time, decode time. The toolbar lets you switch the algorithm. The chart at the bottom keeps a rolling history of the last twenty inputs so you can compare a wall of repeated logs against a unique poem and see why one shrinks to nothing while the other barely budges.

▸ why this exists ▸

Programmers talk about gzip the way fishermen talk about wind, in vague reverence, rarely measuring it. The browser ships a real compressor now. CompressionStream takes a ReadableStream of bytes and emits a ReadableStream of squeezed bytes. No npm dependency, no shim, no service worker. The same handful of lines also runs DecompressionStream so you can roundtrip and prove the bytes survived.

▸ shape of the screen ▸

Top half: two columns. Left is a textarea labeled raw. Right is a readonly textarea labeled coded, rendered as base64 so the eye can read what would otherwise be a smear of high bytes. Between them sits a vertical strip of three radio buttons, gzip, deflate, deflate raw. The strip glows on the active mode.

Bottom half: a strip of statistics in mono type. Bytes in, bytes out, ratio as a percentage, encode milliseconds, decode milliseconds, integrity check pass or fail. To the right, a sparkline canvas, twenty bars wide, each bar a past sample, height proportional to the saved fraction. Hovering a bar surfaces a tooltip with the raw and coded sizes from that sample.

▸ palette ▸

Background a warm off white, hex `#f5efe4`. Foreground ink a soft black, hex `#1a1a1f`. Accent a single muted teal, hex `#3a7a78`, used for the active radio, the sparkline bars, and the focus ring. Errors take a faded brick, hex `#a44a3f`. No gradients. No shadows. A single hairline rule under the toolbar.

▸ typography ▸

Headings in `Fraunces`, weight 500, optical size set to display. Body and stats in `JetBrains Mono`, weight 400, with `font-feature-settings: "tnum" 1` so the digits in the stats line up cleanly. The textarea uses `IBM Plex Mono` at 13px so the raw and coded panes feel like ledger pages.

▸ behavior ▸

Type or paste into the raw textarea. A 120 ms debounce fires the encode pipeline. The pipeline reads the text as UTF 8 bytes, pipes through `new CompressionStream(format)`, collects chunks, base64 encodes the result for display, and times the whole pass with `performance.now`. A second pipeline pipes the same coded bytes through `DecompressionStream(format)` and compares the recovered string to the original. Mismatch flips the integrity row to a red FAIL. Match leaves it as a quiet OK.

The radio strip swaps the format and reruns the pipeline against the current input. The sparkline pushes a new sample on every successful encode and trims to the last twenty. A tiny clear button under the chart wipes history and resets the bars to zero.

A copy button under the coded pane copies the base64 string. A paste button under the raw pane reads from `navigator.clipboard.readText` if the permission is granted, falling back to a one line note if not.

▸ implementation notes ▸

Build with React 18 and TypeScript. Single default export named `App`. Tailwind handles spacing, color tokens, and the focus ring. No external UI kit. Inline SVG for the sparkline tooltips. The compression call is a small async helper that takes a `Uint8Array` and a format string and returns a `Uint8Array`. A second helper does the inverse. Both wrap the streams with `ReadableStream.from([bytes])` then `pipeThrough` then a chunk reader that concatenates into a single buffer.

State lives in a single `useReducer`. Actions: `setRaw`, `setFormat`, `setResult`, `pushSample`, `clearHistory`. The reducer keeps the last twenty samples and a current snapshot. The encode helper is wrapped in a `useEffect` keyed on raw and format, with cleanup that aborts a stale pass via an `AbortController` if a newer keystroke arrives.

Edge cases the reducer handles: empty input zeroes the stats and skips the sample. A failed decode posts an error sample and does not push to the chart. A format the browser does not support, which mostly means deflate raw on older Safari, surfaces a one line note in the integrity row.

▸ accessibility ▸

Every control has a label. The radio strip is a `<fieldset>` with a visually hidden legend that reads compression format. The sparkline canvas has an `aria-label` summarizing the most recent ratio. Color is never the only signal: PASS and FAIL also carry the words PASS and FAIL.

▸ test cases worth pasting ▸

A 4 KB block of `lorem ipsum` shrinks under gzip to roughly 30 percent of the input. A JSON array of 200 identical objects shrinks to single digit percent. A short tweet barely shrinks at all and may grow due to header bytes, which is itself an interesting demo. The chart makes that pattern visible across a working session.

▸ what is out of scope ▸

No file uploads. No drag and drop. No worker. No share link. No download of the coded blob. The point is a tight loop between paste and number, nothing else. If a future round wants Compression of files, that is a separate template.

▸ acceptance ▸

The first render shows an empty raw pane, an empty coded pane, gzip selected, all stats reading zero, and a flat sparkline. Pasting any non empty string produces fresh stats inside one frame, an integrity row that reads OK, and a new bar in the chart. Switching format reruns and updates without clearing history. Clearing history zeroes the bars and leaves the panes alone.
