# Code Quality Setup Guide

Complete quality tooling setup: OxLint, TypeScript, Prettier, Knip, Taze. Use this for all projects.

## Stack

- **OxLint** - Rust-based linter, 50-100x faster than ESLint
- **tsgo** - Native TypeScript compiler (10x faster)
- **Prettier** - Code formatting
- **Knip** - Dead code detection
- **Taze** - Dependency upgrades (major versions)
- **simple-git-hooks + lint-staged** - Git automation

---

## Installation

```bash
# Core tools
bun add -D oxlint prettier knip taze simple-git-hooks lint-staged prettier-plugin-tailwindcss
# TypeScript - match tsgo version
bun add -D typescript@5.8 oxlint-tsgolint
```

**Finding the right TypeScript version:**

tsgo is based on a specific TypeScript version. Check https://github.com/microsoft/typescript-go README - it explicitly states what TS version it's based on (e.g., "Same types as TS 5.8").

Update TypeScript when tsgo updates its base version.

---

## Configuration

### 1. TypeScript: `tsconfig.json`

**Most projects already have this configured by your framework (Next.js, Vite, etc.).**

**Only verify these critical settings:**

```json
{
  "compilerOptions": {
    "strict": true,        // All type safety checks
    "noEmit": true,        // Type check only, no output (usually already set)
    "incremental": true    // Cache for speed
  }
}
```

**If starting from scratch** (rare), run `bunx tsc --init` and ensure the above settings are enabled.

**Framework-specific configs:**
- Next.js: Already configured, don't touch
- Vite: Already configured, don't touch
- Node.js: Use `bunx tsc --init` and enable the settings above

### 2. OxLint: `.oxlintrc.json`

```json
{
  "$schema": "./node_modules/oxlint/configuration_schema.json",
  "plugins": ["react", "typescript", "unicorn", "import", "jsx-a11y", "promise"],
  "settings": {
    "react": {
      "version": "detect"
    }
  },
  "env": {
    "browser": true,
    "es2025": true
  },
  "categories": {
    "correctness": "error",
    "suspicious": "error"
  },
  "rules": {
    "react/react-in-jsx-scope": "off",
    "import/no-unassigned-import": ["error", { "allow": ["**/*.css"] }]
  },
  "ignorePatterns": ["node_modules", ".next", "out", "dist", "build", ".conductor"]
}
```

**Categories:** Use `correctness` + `suspicious` for bug-catching (minimal maintenance, auto-improves with OxLint updates).

**Plugins:**

**ALWAYS enabled by default:**

- `typescript`, `unicorn`, `oxc`

**ALWAYS add:**

- `import` - Catches import/export bugs
- `promise` - Catches async bugs (critical)

**ALWAYS add for React:**

- `react`, `jsx-a11y`

**ALWAYS add for Next.js:**

- `nextjs`

**Add for testing:**

- `vitest`

**Discover available plugins:**

```bash
bunx oxlint --help | grep -A 20 "Enable Plugins"
```

Check periodically for new plugins.

### 3. Prettier: `.prettierrc`

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "avoid",
  "endOfLine": "lf",
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindFunctions": ["cn", "clsx", "cva"]
}
```

**Do you need `.prettierignore`?**

**No** - Prettier uses `.gitignore` by default (`--ignore-path` defaults to `[.gitignore, .prettierignore]`).

Only create `.prettierignore` if you need to:

- Format files that ARE gitignored
- Skip files that are NOT gitignored

For most projects, skip `.prettierignore` entirely.

### 4. Knip: `knip.ts`

```typescript
import type { KnipConfig } from 'knip'

const config: KnipConfig = {
  ignoreIssues: {
    // UI component libraries: ignore unused exports (components are meant to be reused)
    'components/ui/**': ['exports', 'types', 'nsExports', 'nsTypes', 'enumMembers', 'classMembers'],
  },
  ignoreDependencies: [
    // Add packages Knip can't auto-detect
    // Examples:
    // - CSS imports: 'tw-animate-css'
    // - Peer deps: 'tailwindcss'
    // - Framework implicit deps: 'sharp' (Next.js images)
  ],
  ignore: [
    // Config files for tools that Knip doesn't recognize
    'taze.config.ts',
  ],
}

export default config
```

**Key settings:**
- `ignoreIssues` - Ignore specific issue types in specific paths (e.g., unused exports in UI component library)
- `ignoreDependencies` - Packages Knip can't auto-detect
- `ignore` - Files/patterns to completely skip

### 5. Taze: `taze.config.ts`

```typescript
import { defineConfig } from 'taze'

export default defineConfig({
  // Aggressively upgrade all packages to latest major versions
  mode: 'major',

  // Exclude TypeScript-related packages that need manual version alignment
  packageMode: {
    // TypeScript must match tsgo's base version
    // Check https://github.com/microsoft/typescript-go before updating
    typescript: 'ignore',

    // tsgo itself - update independently when ready
    '@typescript/native-preview': 'ignore',

    // tsgo wrapper - update with care
    'oxlint-tsgolint': 'ignore',
  },

  // Write changes to package.json
  write: true,

  // Install dependencies after updating
  install: true,

  // Show all packages, even up-to-date ones
  all: false,
})
```

**Why exclude TypeScript packages?**

tsgo is based on a specific TypeScript version. Updating TypeScript independently breaks the alignment. Always check https://github.com/microsoft/typescript-go for the correct base version before updating.

**IMPORTANT: Taze requires proper semver in package.json**

Taze only works with proper semver versions:

✅ **Works:** `"next": "^14.0.0"`, `"react": "~18.2.0"`
❌ **Doesn't work:** `"next": "14"`, `"react": "18"`

Bare numbers are treated as "satisfied" ranges. Always use `^`, `~`, or explicit versions.

---

## package.json Scripts

Add to your `package.json`:

```json
{
  "scripts": {
    "lint": "oxlint --type-aware --tsconfig=./tsconfig.json --import-plugin --react-plugin --jsx-a11y-plugin --nextjs-plugin --promise-plugin",
    "typecheck": "tsgo --noEmit",
    "format": "prettier --write .",
    "knip": "knip",
    "check": "bun run lint && bun run typecheck && bun run knip",
    "upgrade": "taze",
    "prepare": "simple-git-hooks"
  },
  "simple-git-hooks": {
    "pre-commit": "bunx lint-staged",
    "pre-push": "bun run check"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx,json,css,md,mdx}": "prettier --write"
  }
}
```

**Customize lint command:**

Adjust plugins based on your project:

```bash
# React + Next.js (standard)
--import-plugin --react-plugin --jsx-a11y-plugin --nextjs-plugin --promise-plugin

# With tests
--vitest-plugin

# Node.js backend (no React)
--import-plugin --node-plugin --promise-plugin
```

**ALWAYS include:**

- `--type-aware` - Type-based rules
- `--tsconfig=./tsconfig.json` - Import resolution
- `--import-plugin` - Import validation
- `--promise-plugin` - Async bug detection

---

## Git Hooks

Initialize after adding scripts:

```bash
bun install
bun run prepare
```

Verify hooks exist:

```bash
git config core.hooksPath  # Check path
ls -la .githooks/          # Or .git/hooks/
```

**Hook behavior:**

- **pre-commit**: Format staged files with Prettier
- **pre-push**: Run full quality check (lint + typecheck + knip)

**Skip when needed:**

```bash
SKIP_SIMPLE_GIT_HOOKS=1 git commit -m "message"
git push --no-verify
```

---

## CI/CD

### GitHub Actions: `.github/workflows/ci.yml`

```yaml
name: CI

on:
  pull_request:
    branches: [main]
    paths-ignore:
      - '**.md'
      - '.prettierrc'
      - '.oxlintrc.json'
      - '.vscode/**'
  push:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest

      - name: Cache dependencies
        uses: actions/cache@v4
        with:
          path: |
            ~/.bun/install/cache
            node_modules
          key: ${{ runner.os }}-bun-${{ hashFiles('bun.lockb') }}
          restore-keys: |
            ${{ runner.os }}-bun-

      - name: Install
        run: bun install --frozen-lockfile --no-save

      - name: Build
        run: bun run build

      - name: Lint
        run: bun run lint

      - name: Type check
        run: bun run typecheck

      - name: Check unused code
        run: bun run knip
```

**Step order matters:**

1. Build - Compilation errors
2. Lint - Code issues
3. Type check - Type errors
4. Knip - Dead code

### GitLab CI: `.gitlab-ci.yml`

```yaml
quality:
  image: oven/bun:latest
  cache:
    paths:
      - node_modules/
  script:
    - bun install
    - bun run build
    - bun run lint
    - bun run typecheck
    - bun run knip
```

---

## Framework Integration

### Next.js

Disable built-in checks in `next.config.mjs`:

```javascript
const nextConfig = {
  typescript: {
    ignoreBuildErrors: true, // Use dedicated typecheck script
  },
  eslint: {
    ignoreDuringBuilds: true, // Use OxLint instead
  },
}

export default nextConfig
```

### Vite

No changes needed - Vite doesn't type check during build by default.

### Other Frameworks

Disable built-in linting/type checking if present.

---

## Setup Checklist

### New Project

- [ ] Install dependencies
- [ ] Verify `tsconfig.json` settings
- [ ] Create `.oxlintrc.json`
- [ ] Create `.prettierrc`
- [ ] Create `knip.ts`
- [ ] Create `taze.config.ts`
- [ ] Create `.vscode/extensions.json`
- [ ] Create `.vscode/settings.json`
- [ ] Add scripts to `package.json`
- [ ] Add git hooks config to `package.json`
- [ ] Run `bun run prepare`
- [ ] Create `.github/workflows/ci.yml`
- [ ] Run `bun run check` to verify
- [ ] Commit and push to test CI

### Existing Project Migration

**Phase 1: Clean Migration**

- [ ] Migrate to Bun: `bun install` (creates `bun.lockb`)
- [ ] Remove old package managers: `rm -rf package-lock.json yarn.lock pnpm-lock.yaml node_modules`
- [ ] Reinstall with Bun: `bun install`
- [ ] Remove ESLint: `bun remove eslint @eslint/* eslint-*`
- [ ] Remove ESLint config: `rm -f .eslintrc* eslint.config.*`
- [ ] Install quality tools (see Installation section)
- [ ] Verify `tsconfig.json` settings
- [ ] Create `.oxlintrc.json`
- [ ] Create `.prettierrc`
- [ ] Create `knip.ts`
- [ ] Create `taze.config.ts`
- [ ] Create `.vscode/extensions.json`
- [ ] Create `.vscode/settings.json`
- [ ] Update `package.json` scripts (remove ESLint scripts, add new scripts)
- [ ] Add git hooks config to `package.json`
- [ ] Run `bun run prepare`
- [ ] Update CI config (remove ESLint, add OxLint)

**Phase 2: Initial Quality Check** (Ignore all errors for now)

- [ ] Fix package.json versions: ensure all use proper semver (`^14.0.0`), not bare numbers (`14`)
- [ ] Run `bun run lint` (note issues, don't fix)
- [ ] Run `bun run typecheck` (note issues, don't fix)
- [ ] Run `bun run knip` (note issues, don't fix)
- [ ] Run `bun run upgrade` (upgrade to latest major versions)
- [ ] Run `bun install`
- [ ] Commit migration changes

**Phase 3: Fix Issues**

- [ ] Fix TypeScript errors: `bun run typecheck`
- [ ] Fix lint errors: `bun run lint`
- [ ] Fix Knip issues (verify each one):
  - Check if unused exports are actually unused (double-check imports)
  - Check if unused dependencies are actually unused (search codebase)
  - Remove confirmed dead code and unused dependencies
  - Add exceptions to `knip.ts` only for false positives (e.g., UI libraries, framework requirements)
- [ ] Run `bun run check` until all pass
- [ ] Test the application works
- [ ] Commit fixes

**When to Stop and Ask:**

Stop the process and ask for guidance if you encounter:

- Breaking API changes in major version upgrades that require architectural decisions
- Type errors that require significant refactoring (>100 lines of changes)
- Knip reporting entire features as unused (possible detection issue)
- Framework-specific issues you're uncertain about
- Test failures that aren't obvious to fix

For minor issues (import typos, missing types, obviously dead code), fix without asking.

---

## VSCode Setup

### Extensions

Create `.vscode/extensions.json`:

```json
{
  "recommendations": [
    "oxc.oxc-vscode",
    "esbenp.prettier-vscode",
    "typescriptteam.native-preview",
    "bradlc.vscode-tailwindcss"
  ]
}
```

**Extensions:**

- `oxc.oxc-vscode` - OxLint integration (linting in editor)
- `esbenp.prettier-vscode` - Prettier formatting
- `typescriptteam.native-preview` - TypeScript native (tsgo) support

### Settings

Create `.vscode/settings.json`:

```json
{
  // Formatting
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,

  // OxLint
  "editor.codeActionsOnSave": {
    "source.fixAll.oxc": "explicit"
  },
  "oxc.enable": true,
  "oxc.lint.run": "onSave",

  // TypeScript - use tsgo
  "typescript.experimental.useTsgo": true,
  "typescript.tsdk": "node_modules/typescript/lib",

  // Tailwind CSS IntelliSense (optional, Tailwind projects only)
  "tailwindCSS.classFunctions": ["cn", "clsx", "cva"],

  // Disable conflicting tools
  "eslint.enable": false,
  "biome.enabled": false,
  "deno.enable": false,
  "javascript.validate.enable": false,
  "typescript.validate.enable": false
}
```

**Key settings:**

- Prettier as default formatter, auto-format on save
- OxLint auto-fix and run on save
- Use tsgo for TypeScript
- Disable conflicting tools (ESLint, Biome, Deno, built-in validation)

### Install Extensions

When opening the project, VSCode prompts to install recommended extensions. Install them all.

---

## Maintenance

### Update dependencies

```bash
bun run upgrade  # Upgrades ALL packages to latest major versions
bun run check    # Verify everything works
```

Taze is configured with `mode: 'major'` which upgrades all packages to latest major versions, except TypeScript-related packages (protected in config).

**If checks fail after upgrade:**

1. Run individual checks to isolate the issue:
   - `bun run typecheck` - Type errors
   - `bun run lint` - Lint errors
   - `bun run knip` - Dead code warnings
2. Fix the breaking changes
3. If the fix requires architectural decisions or significant refactoring, stop and ask how to proceed
4. Commit working state

**Never revert upgrades** - either fix the issues or stop and get guidance.

### Update TypeScript manually

TypeScript must match tsgo's base version. Check https://github.com/microsoft/typescript-go README first.

```bash
# Check current tsgo base version
# Visit https://github.com/microsoft/typescript-go

# Update TypeScript to match
bun add -D typescript@5.8  # or whatever version tsgo uses

# Update tsgo and wrapper
bun update @typescript/native-preview oxlint-tsgolint
```

### Check for new OxLint plugins

```bash
bunx oxlint --help | grep -A 20 "Enable Plugins"
```

Visit https://github.com/oxc-project/oxc/releases for updates.

---

## Summary

Each tool has one job:

- **OxLint** - Lint code
- **tsgo** - Check types
- **Prettier** - Format code
- **Knip** - Find dead code
- **Taze** - Update dependencies
- **Git hooks** - Enforce on commit/push
- **CI** - Enforce on PR/merge

Together they create a complete quality system.
