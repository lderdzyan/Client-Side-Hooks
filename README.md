# 🪝 Client Side Hooks

A TypeScript project demonstrating Git client-side hooks (`pre-commit` and `pre-push`) using a **custom `hooks/` directory** at the project root — fully tracked by version control, instead of the default untracked `.git/hooks/` folder.

> **Repo:** [https://github.com/lderdzyan/Client-Side-Hooks.git](https://github.com/lderdzyan/Client-Side-Hooks.git)

---

## Why a Custom Hooks Directory?

By default, Git stores hooks in `.git/hooks/`, which is **not tracked by version control**. This means every developer must set up hooks manually with no way to share or enforce them through the repo.

This project solves that by committing hooks to a `hooks/` folder at the root and pointing Git to it via `core.hooksPath` — so every developer gets the same quality gates automatically.

---

## Project Structure

```
clientSideHooksTest/
├── coverage/                  # Generated test coverage reports
│   ├── lcov-report/
│   ├── clover.xml
│   ├── coverage-final.json
│   └── lcov.info
├── hooks/
│   ├── pre-commit             # Prettier + ESLint + Jest (runs on every commit)
│   └── pre-push               # TypeScript type check + npm security audit (runs on every push)
├── src/
│   ├── project.ts
│   └── string.ts
├── test/                      # Jest test suites
├── .gitignore
├── .prettierignore
├── .prettierrc
├── eslint.config.cjs
├── jest.config.js
├── package.json
├── tsconfig.json
└── README.md
```

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/lderdzyan/Client-Side-Hooks.git
cd clientSideHooksTest
```

### 2. Install dependencies

```bash
npm install
```

### 3. Configure Git to use the custom hooks directory

```bash
git config core.hooksPath hooks
```

### 4. Make hooks executable

```bash
chmod +x hooks/pre-commit
chmod +x hooks/pre-push
```

> **Note:** Required on macOS and Linux. On Windows, run this inside Git Bash.

---

## Hooks Overview

### `pre-commit`

Runs automatically **before every commit**. Performs three checks in sequence:

#### 1. Prettier — formatting check
Runs `npm run format:check`. If formatting issues are found, it **automatically attempts to fix them** with `npm run format`. If auto-fix fails, the commit is aborted.

#### 2. ESLint — linting
Runs `npx eslint . --ext .ts --fix`, auto-fixing what it can. If unfixable errors remain, the commit is aborted.

#### 3. Jest — test suite with coverage
Runs `npm run test:coverage`. If any tests fail, the commit is aborted.

Only when all three pass does the commit go through.

**To bypass (use sparingly):**
```bash
git commit --no-verify
```

---

### `pre-push`

Runs automatically **before every push**. Performs two checks:

#### 1. TypeScript — type checking
Runs `npx tsc --project tsconfig.json --noEmit` to catch type errors without emitting files. Push is aborted on any type error.

#### 2. npm audit — security vulnerability scan
Scans dependencies for vulnerabilities and reports `low`, `moderate`, `high`, and `critical` counts.

- If **critical** vulnerabilities are found, it automatically runs `npm audit fix --force` to attempt a fix.
- If criticals **still remain** after the fix, the push is aborted with instructions to resolve them manually.
- `low`, `moderate`, and `high` vulnerabilities are reported but **do not block** the push.

**To bypass (use sparingly):**
```bash
git push --no-verify
```

---

## Available Scripts

```bash
# Run tests
npm test

# Run tests with coverage report
npm run test:coverage

# Check formatting
npm run format:check

# Auto-fix formatting
npm run format

# Lint TypeScript files
npx eslint . --ext .ts

# Type check without emitting
npx tsc --noEmit

# Security audit
npm audit
```

---

## How It Works

Git's `core.hooksPath` option (available since Git 2.9) redirects hook resolution from `.git/hooks/` to any committed directory:

```bash
git config core.hooksPath hooks
```

Verify it's active:

```bash
git config --get core.hooksPath
# Expected output: hooks
```

Because `hooks/` is committed to the repo, every developer runs the exact same checks — no drift, no "works on my machine" hook discrepancies.

---

## Team Onboarding

Run once after cloning:

```bash
npm install
git config core.hooksPath hooks
chmod +x hooks/pre-commit hooks/pre-push
```

> 💡 **Tip:** Automate this with a `postinstall` script in `package.json` so no one forgets:
> ```json
> "scripts": {
>   "postinstall": "git config core.hooksPath hooks && chmod +x hooks/pre-commit hooks/pre-push"
> }
> ```

---

## Requirements

- **Node.js** 18+
- **Git** 2.9+ (for `core.hooksPath` support)
- **jq** — used by the `pre-push` hook to parse `npm audit` JSON output
  ```bash
  # macOS
  brew install jq

  # Ubuntu/Debian
  sudo apt install jq
  ```

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| TypeScript | Language |
| Jest | Testing & coverage |
| ESLint | Linting (auto-fix on commit) |
| Prettier | Formatting (auto-fix on commit) |
| npm audit | Security vulnerability scanning (on push) |
| Git hooks | Automated quality gates |

---

## License

MIT
