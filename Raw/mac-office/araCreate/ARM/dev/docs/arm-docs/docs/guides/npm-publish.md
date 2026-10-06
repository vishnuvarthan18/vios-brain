# NPM Package Release Workflow

> This is the authoritative, and currently the *only*, path that actually runs `npm publish` — `release.yml`'s CI-driven `semantic-release` handles versioning/changelog only (`npmPublish: false` in `.releaserc.json`). See [RELEASE-PROCESS.md](../RELEASE-PROCESS.md) §3 for how the two relate (and where that relationship isn't fully documented).

## Package

```bash
@aracreate/test-arm-ui
```

---

# 1. Make Component Changes

Update:

* Components
* Styles
* Types
* Exports
* Documentation

Example:

```bash
src/components/Button/Button.tsx
```

---

# 2. Build the Library

```bash
npm run build
```

This runs:

```bash
npm run build:js
npm run build:css
```

---

# 3. Run Tests

```bash
npm test
```

Optional Storybook testing:

```bash
npm run storybook
```

---

# 4. Update Package Version

## Patch Release

For bug fixes:

```bash
npm version patch
```

Example:

```bash
1.0.0 → 1.0.1
```

---

## Minor Release

For new features:

```bash
npm version minor
```

Example:

```bash
1.0.0 → 1.1.0
```

---

## Major Release

For breaking changes:

```bash
npm version major
```

Example:

```bash
1.0.0 → 2.0.0
```

---

## Dev / Prerelease Version

```bash
npm version prerelease --preid=dev
```

Example:

```bash
1.0.0-dev.16 → 1.0.0-dev.17
```

---

# 5. Publish the Package

```bash
npm publish --access public
```

Your `prepublishOnly` script automatically runs:

```bash
npm run clean && npm run build
```

before publishing.

---

# 6. Verify Published Version

```bash
npm view @aracreate/test-arm-ui versions
```

---

# 7. Install the Latest Version

## Latest

```bash
npm install @aracreate/test-arm-ui@latest
```

---

## Specific Version

```bash
npm install @aracreate/test-arm-ui@1.0.1
```

---

# Recommended Release Workflow

```bash
git add .

git commit -m "feat: add new button variants"

npm version patch

git push --follow-tags

npm publish --access public
```

---

# Important Notes

## You Cannot Publish the Same Version Twice

This fails:

```bash
1.0.0-dev.16
```

if already published.

You must increment the version before every publish.

---

# Common Commands

## Check Current User

```bash
npm whoami
```

---

## Check Current Package Version

```bash
npm pkg get version
```

---

## Check Published Package Info

```bash
npm info @aracreate/test-arm-ui
```

---

# Recommended Fix for CommonJS Support

Current output:

```bash
dist/index.js
```

Update `package.json`:

```json
"main": "dist/index.js",
"exports": {
  ".": {
    "types": "./dist/index.d.ts",
    "import": "./dist/index.mjs",
    "require": "./dist/index.js"
  }
}
```

Then publish a new version.

---
