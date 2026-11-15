# Octo

Octo is a single-page web app: an AI text detector and "AI remover" plus everyday tools.

**Developer:** [isdapperdude404](https://github.com/isdapperdude404)

## Tools
AI detector, AI remover, word counter, text cleaner, summarizer, tasks and notes, focus timer, password maker, file converter (PDF, Word, Excel, CSV, text), voice typing, face detection, predictor, AI hints, 2026 watch.

## Run it
Open `index.html` in a browser. No build step.

## Publish on GitHub Pages
1. Create a repository named `octo` on your account.
2. Upload `index.html` and `README.md`.
3. Go to Settings > Pages, set Source to "Deploy from a branch", choose `main` and `/ (root)`, then Save.
4. Your site will be at `https://isdapperdude404.github.io/octo/`.

## Notes
- The AI detector and remover use fixed rules, so scores are estimates.
- Voice typing, AI hints and "Save as Word/text" use features that only exist inside claude.ai. On GitHub Pages those buttons show a message instead of working. Everything else works.
- Face detection needs a browser with the FaceDetector API (Chrome).
