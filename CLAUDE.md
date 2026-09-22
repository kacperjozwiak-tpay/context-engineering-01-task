# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project Overview

"Conduit" — a React + Redux implementation of the [RealWorld](https://github.com/gothinkster/realworld) spec (a Medium.com-style social blogging app). Plain JavaScript (no TypeScript), bootstrapped with Create React App (`react-scripts` 1.1.1). It talks to a hosted REST API (`https://conduit.productionready.io/api`, configured in `src/agent.js`) for auth, articles, comments, and profiles.

## Code Style

1. **Functional, arrow-function style throughout.** Components, action creators, and API helpers are written as arrow functions (`const Foo = props => { ... }`), not `function` declarations or classes, except where React lifecycle/class features are actually needed.
2. **Single quotes, semicolons, 2-space indentation.** Match the existing formatting in `src/` exactly — no reformatting unrelated lines in a diff.
3. **Named export objects for API/action groups.** Group related request helpers into a plain object and export it (see `src/agent.js`: `Auth`, `Articles`, `Comments`, `Profile`, `Tags`), rather than exporting many individual functions.
4. **Action types live in `src/constants/actionTypes.js`.** Never hardcode a raw string for a Redux action type in a reducer or action creator — import the constant.
5. **Reducers are pure and split by domain.** One reducer file per feature under `src/reducers/` (e.g. `article.js`, `auth.js`, `settings.js`), combined in `src/reducer.js`. Do not mutate state — always return a new object/array.
6. **Components stay presentational where possible.** Container-style wiring (`connect`, `mapStateToProps`/`mapDispatchToProps`) belongs at the top-level page/container component; nested components under a page (e.g. `src/components/Article/*`) receive data via props.
7. **No prop destructuring convention change mid-file.** Follow whatever the surrounding component already does (`props.x` vs destructured `{ x }`) rather than mixing styles within the same file.

## Project Structure

- `src/index.js` — app entry point, renders the root `App` component and wires the Redux `Provider`.
- `src/store.js` — Redux store creation and middleware setup.
- `src/reducer.js` — root reducer, combines all domain reducers.
- `src/reducers/` — one reducer per domain/feature (`auth.js`, `article.js`, `articleList.js`, `editor.js`, `home.js`, `profile.js`, `settings.js`, `common.js`).
- `src/constants/actionTypes.js` — single source of truth for Redux action type strings.
- `src/middleware.js` — custom Redux middleware (dispatches side-effecting API calls triggered by actions).
- `src/agent.js` — all HTTP calls to the backend API (via `superagent`); the only place `API_ROOT` and auth token handling live.
- `src/components/` — React components. Top-level pages (`App.js`, `Login.js`, `Register.js`, `Settings.js`, `Editor.js`, `Profile.js`, `Header.js`, `ListPagination.js`, `ListErrors.js`, `ArticlePreview.js`) with feature subfolders:
  - `src/components/Home/` — home page (banner, tag list, main article feed view).
  - `src/components/Article/` — article detail page, comments, article actions/meta, delete button.
- `public/` — static HTML shell and favicon used by Create React App.

## Workflow

- Install dependencies: `npm install`
- Start dev server (port 4100): `npm start`
- Run tests (Jest via react-scripts, jsdom env): `npm test`
- Type/build check (no TypeScript in this project — use a production build to surface compile-time and bundling errors): `npm run build`
- There is no separate lint script configured; if linting is added, wire it through `npm run lint` and document it here.

## Constraints

1. **Do not change the API contract.** All backend interaction must go through `src/agent.js` and match the [RealWorld API spec](https://github.com/GoThinkster/productionready/blob/master/api) — do not invent new endpoints or response shapes.
2. **Do not introduce TypeScript, class components, or new state-management libraries.** This codebase is intentionally plain JS with functional components and Redux; keep new code consistent with that (unless the user explicitly asks for a migration).
3. **Do not commit secrets or environment-specific config.** API keys/local overrides belong in a git-ignored `.env` file, never hardcoded into `src/agent.js` or committed.
4. **Do not bump `react-scripts` or core dependency major versions without explicit request.** This project pins older versions (`react-scripts@1.1.1`, React 16, Redux 3) deliberately; upgrading can break the build tooling.
5. **Preserve JWT auth flow.** The token is stored in `localStorage` and injected via `agent.js`'s `tokenPlugin`; do not change how/where the token is persisted without discussing it first, since it affects session behavior across the whole app.
