# Repository Guidelines

## Project Structure & Module Organization
- `index.html` is the entire app (HTML, Tailwind via CDN, CSS, and vanilla JS in one file).
- There are no separate assets or test directories; everything is inline.
- Firebase Realtime Database paths are organized by date (e.g., `daily/YYYY-MM-DD`) with a few global collections (e.g., `tutoringSessions`, `retest`).

## Build, Test, and Development Commands
- `python -m http.server 8000` starts a local server for the static file.
- `npx http-server .` is an alternative static server.
- Open `http://localhost:8000/index.html` to run the app.
- There is no build step or bundler.

## Coding Style & Naming Conventions
- Use 2-space indentation to match existing HTML/CSS/JS formatting in `index.html`.
- Keep UI strings in Korean to match the current interface.
- Follow existing naming patterns: `camelCase` for functions and state objects (e.g., `renderDailyTasks`, `tutoringSessions`).
- Keep additions inline in `index.html` and reuse existing state -> Firebase listener -> render flow.

## Testing Guidelines
- No automated tests are present; testing is manual in the browser.
- Use different dates to validate date-scoped data without impacting today’s data.
- For past-incomplete features, create items on prior dates with incomplete flags and confirm they surface correctly.

## Commit & Pull Request Guidelines
- Use the `UI/UX: ...` prefix for commit messages, with a short Korean summary after the colon.
- Keep messages concise and descriptive.
- PRs should include: a clear description, test steps (commands or manual checks), and screenshots or GIFs for UI changes.
- Include a brief checklist using Korean labels when applicable: 업무 체크리스트, 재시험 일정, 영어 단어 시험 목록, 보강대상목록, 학생별 이슈.
- Note any Firebase schema or data behavior changes in the PR description.
- Use the PR template in `.github/PULL_REQUEST_TEMPLATE.md`.

## Security & Configuration Tips
- Firebase credentials are embedded in `index.html`. Avoid sharing keys outside the repo and do not add new secrets.
- There is no auth layer; be careful when testing against the production database.
