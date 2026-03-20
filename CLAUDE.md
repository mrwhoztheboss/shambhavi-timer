# CLAUDE.md

## Current Phase
DEPLOYING

## Project
Shambhavi Timer — a guided Kriya practice timer web app.

## Agent Rules

### When phase is PLANNING
- Output SPEC.md only
- Absolutely no HTML, CSS, or JavaScript code
- No file creation other than SPEC.md

### When phase is CODING
- Read SPEC.md before writing anything
- Output: index.html, README.md, .gitignore only
- No GitHub Actions or deployment files
- All CSS and JS must be inline inside index.html

### When phase is TESTING
- Read index.html thoroughly before writing anything
- Output TEST_REPORT.md first
- Then output a fully corrected index.html

### When phase is DEBUGGING
- Read index.html thoroughly before touching anything
- Do NOT rewrite the whole file
- Do NOT change anything unrelated to the reported bug
- Fix only what is broken, surgically
- Add a comment above every fix: // DEBUG FIX: [what was wrong]
- Output the corrected index.html only

### When phase is DEPLOYING
- Read the tested index.html
- Output deploy.yml inside .github/workflows/
- Print exact git terminal commands
- Do not modify index.html
