# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

MathKids: a small single-page app for a child to practice multiplication (tables 1–10) and exact division, with three shuffled multiple-choice answers, a live score counter, and a touch-friendly mobile-first UI. The UI text, code identifiers, and comments are all in Spanish — keep new code consistent with that.

## Commands

Bun is the package manager and runtime (`bun.lockb` is committed; don't add an npm/yarn lockfile).

- `bun install` — install dependencies
- `bun run dev` — Vite dev server at http://localhost:5173
- `bun run build` — production build into `dist/`
- `bun run preview` — serve the built `dist/`

There is no test suite, linter, or formatter configured.

## Architecture

- Stack: Vue 3 (Composition API, `<script setup>`), Vite, Tailwind CSS v4 via the `@tailwindcss/vite` plugin. Tailwind v4 has no `tailwind.config.js`; it's enabled solely by `@import "tailwindcss";` in `src/style.css`. Styling is done entirely with utility classes in templates.
- `vite.config.js` sets `base: './'` so the built `dist/` works from any path (e.g. static hosting in a subdirectory). Keep asset references relative-friendly.
- `src/App.vue` is only a centering layout wrapper; all game logic and UI live in `src/components/MathGame.vue`, using local `ref` state (no store). `BarraPartes.vue` is the tappable column bar used by the fraction/decimal modes (and, non-interactive, in the error modal).
- Modes (`operacionActual`): `'multiplicar' | 'dividir'` are multiple choice (`generarOperacion()`); `'fracciones' | 'decimales'` ask the child to paint N columns of a bar (`generarBarra()`; decimals are always tenths, 10 columns). Division is generated backwards (divisor × quotient = dividend) so results are always whole numbers; distractors are two distinct positives within ±4 of the answer.
- All answers go through `registrarResultado(correcto, textoSolucion)`: correct answers auto-advance after 1200 ms. On a wrong answer, if `textoSolucion` is given (multiplication, fractions, decimals) the modal shows the correct answer and a "Siguiente" button; otherwise (division) it shows "Reintentar" on the same question.
- The score (`aciertos`, `totalIntentos`) persists in `localStorage` under `mathkids-marcador`; on load, a saved score triggers a "continuar / empezar de cero" dialog.
- PWA: static files in `public/` (`manifest.webmanifest`, `sw.js`, PNG icons rendered from `icon.svg`). The service worker is network-first with cache fallback and is only registered in production builds (`src/main.js`). Installation requires HTTPS (or localhost).
- `vue-router` is listed as a dependency but is not currently used.
