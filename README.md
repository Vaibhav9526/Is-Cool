<p align="center">
  <img src="reference/logo.png" alt="IsCool logo" width="144" height="144">
</p>

<h1 align="center">IsCool</h1>

<p align="center"><strong>A school of AI tutors on your own machine.</strong></p>

IsCool is a desktop study companion with a school-themed floor and a cast of AI characters. Ask questions, work through a topic, practise with quizzes, and keep your study materials together.

Built by **Tizmo AI**, IsCool is the School-only port of Jargon. This version focuses on learning: the Office, Tapri, Mail, and Spectator experiences are not part of the app. Automatic updates are disabled.

## Contents

- [Features](#features)
- [Meet the school](#meet-the-school)
- [Getting started](#getting-started)
- [Study commands](#study-commands)
- [Development](#development)
- [Project structure](#project-structure)
- [Brand assets](#brand-assets)
- [License](#license)

## Features

- **Character-based tutoring** — chat with the Teacher, Topper, Smart Guy, Librarian, or Vice Principal, each with a different approach.
- **Questions from your materials** — paste or attach images and PDFs to discuss their contents.
- **Interactive teaching** — open a hosted classroom with `@teach-me` and continue on the teaching website.
- **Practice and feedback** — use `@quiz` for multiple-choice questions, analysis, and weak-point feedback.
- **Handwritten-style answers** — create PDF explanations with diagrams using `@handwritten`.
- **A shared library** — keep notes, books, and slides with the Librarian and ask characters to answer from them.
- **Local file tools** — convert documents and images or compress images in the Librarian's chat.

## Meet the school

| Character | What they help with |
| --- | --- |
| **Teacher** | Patient, step-by-step explanations, interactive teaching, and quizzes. |
| **Topper** | Answer checks and challenging questions, with a confident personality. |
| **Smart Guy** | Shortcuts, memory tricks, and big-picture insight. |
| **Librarian** | Your notes, books, slides, and local file conversion tools. |
| **Vice Principal** | Keeping the school running and helping you plan your studies. |

## Getting started

### Requirements

- Node.js and npm.
- An installed and authenticated AI CLI compatible with the provider selected in the app.
- A native build toolchain for `better-sqlite3` and `node-pty`. On Windows, this can require Visual Studio Build Tools with C++ support.
- Internet access for installation, AI-provider requests, and the hosted teaching website.

The desktop app runs on your machine, but AI conversations use your configured provider. Running locally does not mean every feature works offline.

### Install and run

From the project directory:

```sh
npm install
npm run dev
```

### Windows installation workaround

On the current Windows development machine, a normal install fails because Node 25's headers cause the native build to request the unavailable ClangCL toolset. Use the verified installation sequence instead:

```sh
npm install --ignore-scripts
node node_modules/electron/install.js
npx --no-install electron-rebuild -f
node tools/ensure-pty-perms.cjs
node tools/patch-node-pty-conpty.cjs
npm run dev
```

If rebuilding reports missing Spectre-mitigated Visual C++ libraries (`MSB8040`), install the matching components in Visual Studio Build Tools. See [AGENTS.md](AGENTS.md) for the machine-specific workaround and native dependency notes.

### Your first study session

1. Configure your AI provider and default command in the app.
2. Register a project or workspace folder through **Add agent → Workspace**.
3. Wake a school character and open their chat.
4. Ask a question, or attach an image or PDF to work from.
5. Add study materials to the Librarian's shelf and use `@memory-notes` to refer to them later.

## Study commands

Type these in a character's chat. Attach a file first where indicated.

| Command | Where | Example / result |
| --- | --- | --- |
| `@teach-me` | With an active Teacher | `@teach-me explain photosynthesis` opens the hosted interactive classroom. |
| `@quiz` | Teacher | `@quiz adding fractions` creates MCQs with analysis and study feedback. |
| `@memory-notes` | Any school character | `@memory-notes explain osmosis from my notes` uses matching library material. |
| `@handwritten` | Any school character | `@handwritten explain how a transistor works` produces a handwritten-style PDF answer. |
| `@convert` | Librarian | Attach a file, then use `@convert to pdf`, `@convert to ppt`, or `@convert to image`. |
| `@compress N%` | Librarian | Attach an image, then use `@compress 50%` to request a size reduction. |

### Hosted classroom

`@teach-me` (also available as `@teacher`) opens the hosted teaching website in an isolated window. Sign in, upload your materials, and start generation there manually. No teaching API access code is required by IsCool; the hosted provider may have its own account or payment requirements. IsCool does not make teaching-generation API calls.

### File-tool limitations

- Conversion and compression run locally; these tools do not upload your files.
- Supported conversions: PDF → PowerPoint, PowerPoint → PDF, image → PDF, and PDF → image.
- PDF → PowerPoint produces one image per slide, not editable slide elements.
- PowerPoint → PDF requires Microsoft PowerPoint on Windows or LibreOffice.
- Image compression accepts a requested reduction between 5% and 95%.

## Development

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start Electron with the development renderer and file watching. |
| `npm run typecheck` | Check main-process and renderer TypeScript. |
| `npm run build` | Build the app and copy main-process runtime assets. |
| `npm run test:focused` | Run the repository's Node test files. |
| `npm run dist:win` | Build Windows installer and portable packages. |
| `npm run dist:mac` | Build macOS packages. |
| `npm run dist:linux` | Build Linux packages. |

Run the focused branding and school checks:

```sh
node --test test/iscool-branding.test.cjs test/staffroom-theme.test.cjs test/school-character-art.test.cjs test/school-leader-switch.test.cjs test/god-identity.test.cjs
```

For teaching, quiz, handwritten-answer, and file-tool checks:

```sh
node --test test/teaching-routing.test.cjs test/teaching-composer.test.cjs test/teaching-window.test.cjs test/teaching-branding.test.cjs test/quiz.test.cjs test/handwritten.test.cjs test/file-tools.test.cjs
```

To rebuild the current school map, install Pillow and run:

```sh
python tools/build-school-world.py
```

Use this generator for the current staffroom theme; older school generators produce superseded layouts. See [AGENTS.md](AGENTS.md) for further platform-specific notes.

## Project structure

```text
reference/          Original IsCool logo (PNG and SVG)
src/main/           Electron lifecycle, AI processes, storage, and local tools
src/preload/        The renderer's controlled bridge to Electron
src/renderer/       React interface, school floor, and character chats
src/shared/         Shared types, command parsing, and validation
resources/          Bundled helpers and agent skills
build/              Desktop icons and packaging resources
test/               Regression tests
tools/              Asset generators and development helpers
```

## Brand assets

The app logo is the supplied pixel-art librarian portrait in [`reference/logo.png`](reference/logo.png), with the original SVG in [`reference/logo.svg`](reference/logo.svg). These are the source assets for the app and this README; keep the originals unchanged.

Regenerate the packaging icons from the supplied logo:

```sh
node tools/build-brand-assets.cjs
```

This writes `build/icon.png`, `build/icon.svg`, `build/icon.ico`, and `build/icon.icns`. The renderer uses `reference/logo.png` directly, and the build copies it beside the main bundle for native window and teaching-window branding.

## License

The application code is licensed under the [MIT License](LICENSE), inherited from Jargon. Artwork has separate terms; see [LICENSE-ASSETS](LICENSE-ASSETS). Do not assume every bundled asset is covered by the code license.
