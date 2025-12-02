# 🚀 Node.js Monorepo & Private Modules — Full Guide

**Author:** Maryam Skaik  
**Topic:** Monorepos & Private Modules in Node.js

![Course](https://img.shields.io/badge/Course-Node.js%20Advanced-%23ff6f61)
![Language](https://img.shields.io/badge/Language-JavaScript-%23e57373)
![Category](https://img.shields.io/badge/Category-Monorepo%20%26%20Packages-%23ba68c8)
![Level](https://img.shields.io/badge/Level-Intermediate%20%2F%20Advanced-%2381c784)
![Purpose](https://img.shields.io/badge/Purpose-Teaching-%234fc3f7)

---

## 📝 Introduction

This guide demonstrates how to organize multiple Node.js packages in a **single monorepo**, including:

- Shared libraries  
- Multiple packages consuming shared code  
- Private package publishing via GitHub Packages  

You will learn:

- ✅ What a monorepo is  
- ✅ How Node.js resolves dependencies in a monorepo  
- ✅ How to use workspace linking for local development  
- ✅ How to publish packages privately  
- ✅ How to manage CJS and ESM module boundaries in a monorepo 

---

## 📚 What is a Monorepo?

A **monorepo** (mono repository) is a single repository that hosts **multiple related packages/projects**.

Example structure:

```bash
node-monorepo/
├─ packages/api # backend
├─ packages/web # frontend
└─ packages/shared # reusable library
```

**Benefits:**

🟢 Single source of truth  
🟢 Easier cross-package refactoring  
🟢 Shared tooling, linting, tests  
🟢 Simplified dependency updates  

**Trade-offs:**

🔴 Larger repository size  
🔴 Tooling may be more complex  
🔴 CI pipelines may run slower if not optimized  

---

## ⚡ Why Use a Monorepo in Node.js?

Node.js projects often have **multiple interdependent packages**. Monorepos allow:

- **Code sharing:** Import shared libraries locally without publishing.  
- **Atomic changes:** Update shared code and consumers in a single commit.  
- **Consistent tooling:** ESLint, TypeScript, testing configs can be shared.  
- **Simplified versioning:** Keep package versions consistent.  

---

## 🧩 Monorepo & CJS vs ESM

Node.js supports two module systems:

- **CommonJS (`.cjs` or `.js` with `"type": "commonjs"`)** – uses `require()` / `module.exports`.  
- **ES Modules (`.mjs` or `.js` with `"type": "module"`)** – uses `import` / `export`.  

Mixing them in the same project can cause errors because Node.js cannot automatically determine module type for every file.  

**How monorepos help:**

- **Separate CJS and ESM into different packages**:

```bash
node-monorepo/
├─ packages/api # CJS
├─ packages/web # ESM
└─ packages/shared # ESM
```

- **Package-level `"type"`** ensures Node knows how to interpret each module.  
- **Isolated migration:** Convert one package at a time from CJS → ESM without breaking other packages.  
- **Clear dependency boundaries:** Workspace linking handles inter-package imports cleanly.  

**Important:** Monorepo **does not automatically fix CJS/ESM conflicts**. You still need to manage `"type"`, file extensions, and import syntax per package.

---

## 📂 Shared Folder Recommendations

The **`shared` folder** contains code used by both `api` and `web`. Choosing `.cjs` vs `.mjs` depends on the consumers:

1. **If all consumers are ESM (`web` and modern Node.js packages):**  
 - Use **ESM** (`.mjs` or `"type": "module"`).  
 - Cleanest, no interop issues.  

2. **If some consumers are CJS (`api`) and cannot migrate:**  
 - Either use **CJS** (`.cjs`) with ESM interop (`await import()` in ESM).  
 - Or provide **dual builds** (`index.cjs` + `index.mjs`) using a bundler like **Babel**, **tsup**, or **Rollup**.  

✅ **Recommendation:** Default to **ESM** unless you have legacy CJS consumers. Monorepo makes it easy to isolate module types per package.

---

## 🛠 Workspace Tools for Node.js Monorepos

| Tool | Description | Pros | Cons |
|------|------------|------|------|
| npm Workspaces | Built-in npm support | Simple, minimal tooling | Limited features |
| Yarn Workspaces | Yarn v4+ | Plug’n’play, zero-installs | Requires Yarn |
| pnpm Workspaces | Fast, disk-efficient, strict | Strict, fast, shared store | Requires pnpm installation |

> **Tip:** Use **pnpm** for speed & strictness, **npm** for simplicity, Yarn for advanced workspace features.

---

## 💻 Practical Example: Building a Monorepo

### Step 1: Folder Structure

```bash
node-monorepo-private-modules/
├─ packages/
│ ├─ shared/ # reusable library (ESM)
│ ├─ api/ # backend (CJS or ESM)
│ └─ web/ # frontend (ESM)
├─ examples/ # test scripts
├─ package.json
└─ pnpm-workspace.yaml
```

### Step 2: Initialize Packages

**Shared (`packages/shared/package.json`)**
```json
{
  "name": "@maryam-skaik/shared",
  "version": "0.1.0",
  "type": "module"
}
```

**API (`packages/api/package.json`)**

```json
{
  "name": "@maryam-skaik/api",
  "version": "0.1.0",
  "type": "module",
  "dependencies": {
    "@maryam-skaik/shared": "workspace:*"
  }
}
```

**Web (`packages/web/package.json`)**

```json
{
  "name": "@maryam-skaik/web",
  "version": "0.1.0",
  "type": "module",
  "dependencies": {
    "@maryam-skaik/shared": "workspace:*"
  }
}
```

**Root workspace config (`pnpm-workspace.yaml`)**

```yaml
packages:
  - 'packages/*'
```

### Step 3: Create Shared Module

`packages/shared/index.js`

```js
export function greet(name) {
  return `Hello, ${name}! Welcome to our monorepo.`;
}
```

### Step 4: Consume Shared Module

`packages/api/app.js`

```js
import { greet } from '@maryam-skaik/shared';
console.log(greet("Maryam"));
```

`examples/consume-shared.mjs`

```js
import { greet } from '../packages/shared/index.js';
console.log(greet("Student"));
```

### Step 5: Install Dependencies

```bash
pnpm install
```

> pnpm automatically links workspace packages locally.

---

## ## 📦 Publishing Private Packages to GitHub

1. Generate a Personal Access Token (classic) with scopes:  
   - ✅ `write:packages`  
   - ✅ `read:packages`

2. Create `~/.npmrc`:

```bash
@maryam-skaik:registry=https://npm.pkg.github.com/
//npm.pkg.github.com/:_authToken=YOUR_TOKEN_HERE
```

3. Publish shared package:

```bash
cd packages/shared
npm publish --access=restricted
```

**Note:** This package is now published.
   - **If it is public, anyone can install it in their project using:**
```bash
npm install @maryam-skaik/shared
```
   - **If it is private (restricted), users must configure their npm registry and provide a valid token to access it.**

## ▶ Running Examples

```bash
# Run API package
node packages/api/app.js

# Run Web package
node packages/web/index.js

# Run example script
node examples/consume-shared.mjs
```

**Expected output:**

```css
Hello, Maryam! Welcome to our monorepo.
Hello, Maryam! Welcome to our monorepo.
Hello, Student! Welcome to our monorepo.
```

---

## 🏗 Professional Repo Structure & Tips

- Keep `packages/` small and focused per module.
- Use **consistent naming** (`@scope/package`) for clarity.
- Share ESLint, Prettier, and test configs at root.
- Automate CI/CD for selective builds of changed packages.
- Version packages independently or together based on your workflow.
- Use monorepos to **isolate CJS vs ESM** packages and simplify migration.

