# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, zero-dependency PWA: a pixel-art music player holding a curated playlist plus a birthday card, made as a gift. It is deployed as static files on GitHub Pages — pushing to `main` is the deploy.

## Commands

There is no build, bundler, package manager, linter, or test suite. Editing `index.html` is the whole development loop.

Serve locally over HTTP (required — the service worker and `fetch` won't work from `file://`):

```
python -m http.server 8000    # then open http://localhost:8000
```

Most behavior worth testing (iOS lock screen, audio resume after backgrounding, mobile viewport sizing) only reproduces on a real phone against the deployed Pages URL.

## Architecture

Everything is in `index.html` (~1430 lines): markup, CSS, and the player engine in one `<script>`. The only external request is the Google Fonts stylesheet for `Press Start 2P`. Supporting files: `sw.js` (service worker), `manifest.json` (PWA), `songs/*.mp3`, `photos/*`, `icons/*`.

**Content model.** The `songs` array (`index.html:813`) is the single source of content: `{title, artist, src, photo, emoji, why}`. Adding a track means appending an object, dropping the MP3 in `songs/`, and the image in `photos/`. Photo extensions are inconsistent in casing (`.JPEG`, `.JPG`, `.PNG`, `.jpg`) and GitHub Pages is case-sensitive — copy the real filename rather than assuming lowercase. `photo: ""` falls back to rendering `emoji`.

**Two views, one shell.** `switchTab()` toggles `.active` between `#viewPlayer` and `#viewCard`. The card view is static markup (`index.html:766`) — personal letter text, edit in place.

**Audio graph and the iOS constraint.** One `<audio>` element feeds a WebAudio `AnalyserNode` (`ensureAudioCtx`) that drives the canvas visualizer. Because `createMediaElementSource` reroutes the element through the graph, a suspended `AudioContext` means *silence even while the element reports playing*. iOS suspends the context on lock/background without firing any of our own handlers, so resume is attempted from four places: the context's own `statechange`, the `play` event, `visibilitychange`, and `pageshow`. These are redundant on purpose — several past commits exist solely to fix lock-screen playback. Don't consolidate them.

Lock-screen controls come from the Media Session API, re-registered inside `loadSong` on every track change.

**Layout stability is deliberate.** Mobile browser chrome showing/hiding changes `vh`, which visibly reflowed the UI. Three places now pin dimensions explicitly instead of letting flex/viewport units decide: the shell (`width: min(460px, 100vw)`, `height: 100dvh`), the photo panel (`syncPhotoPanelHeight` sets an explicit pixel height, re-run on resize), and the playlist (locked to 2.5 rows of measured row height at startup). Replacing any of these with `flex: 1` or `vh`-based sizing reintroduces the jitter.

**Marquees.** Two different scroll behaviors, both measuring inside `requestAnimationFrame` after text is set: `setupMarquee` bounces title/artist left-and-back; `setupWhyMarquee` duplicates the "why" text with a `✦` separator and translates to `-50%` for a seamless endless loop. Both must be re-invoked on every song load *and* on `resize`, since overflow is measured against the current container width.

**Boot screen and power-on.** `.boot` (`index.html:666`) is a full-screen pixel loader that covers the app from first paint. Progress is milestone-driven — the webfont, the first song's photo, `window.load` — not faked, but it can never strand her on a loading bar: a `MAX_MS` check inside the draw loop and a `setTimeout` bailout both lead to `ready()`, and the bailout matters because `requestAnimationFrame` stalls in a backgrounded tab. `tick()` returns early once `finished` is set, or the next frame repaints the half-finished bar over the 100% `ready()` just wrote.

`ready()` does not dismiss anything — it reveals PRESS START and stops. She taps; `dismiss()` fades the boot and, 180ms in, calls `powerOn()` so the fade and the power-on read as one motion. That tap is also a user gesture, which is the natural place to hang audio unlocking if that's ever wanted.

`powerOn()` runs the CRT open: `.shell` is held collapsed under `.crt-init` (a bright hairline), `.crt` expands it in hard `steps()`, and both classes are stripped 640ms later so `.shell` is left with no transform of its own. Two things make this safe to touch. Only `transform` and `opacity` are animated, so the startup measurements — all `offsetWidth`/`offsetHeight` based (`syncPhotoPanelHeight`, the playlist row lock, the marquees) — still read the true box; had any of them used `getBoundingClientRect`, the collapsed shell would have poisoned them. And the collapsed state is applied from JS, never from the markup, so a script that fails leaves a normal shell rather than one stuck at zero height.

Every `prefers-reduced-motion` override here must repeat its counterpart's selector exactly. `.shell.crt > *` loses to `.shell.crt > *:not(.crt-flash)` on specificity, and `.boot-start` loses to `.boot-start.show` — in both cases the animation survives the override and reduced motion silently does nothing.

**Service worker.** `sw.js` is network-first for the app shell (`./`, `index.html`, `manifest.json`) so pushed edits appear on next load, cache-first for photos and songs since those are large and static. Bump the `CACHE` constant (currently `for-her-v2`) when changing caching strategy — the `activate` handler deletes every cache whose key doesn't match.

## Notes

- Tone matters here: song titles, artist names, and `why` notes are inside jokes, intentionally lowercase and informal. Don't "fix" the copy.
- The empty `{songs,photos,icons}` directory at the repo root is an accidental leftover from a shell brace-expansion; it's untracked and unused.
