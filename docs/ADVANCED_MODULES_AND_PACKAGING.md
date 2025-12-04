# 🚀 Advanced Modules & Packaging

## 📦 Part A — ESM vs CommonJS

### 🔍 1. What They Are and Why Node Supports Both?

- **CommonJS (CJS)**: the original Node module format (uses `require()`/`module.exports`). Modules are loaded synchronously and executed with a per-file wrapper (module scope). It is stable and ubiquitous in existing Node code. 
- **ECMAScript Modules (ESM)**: the standardized JavaScript module system (`import/export`) used by browsers and modern tooling. ESM supports static analysis, tree-shaking, and top-level await. Node implements ESM to align with the JS standard and browser modules. 

**Why both?** compatibility and ecosystem history: a huge amount of existing Node code uses CJS; Node added ESM to interoperate with the standard and
browsers while preserving backward compatibility.

---

### ✍️ 2. Syntax Differences

- **CommonJS**

```js
// math.cjs
function add(a,b){ return a+b; }
module.exports = { add };
```

```js
// app.cjs
const { add } = require('./math.cjs');
console.log(add(2,3)); // 5
```

- **ESM**

```js
// math.mjs
export function add(a,b){ return a+b; }
```

```js
// app.mjs
import { add } from './math.mjs';
console.log(add(2,3)); // 5
```

- **Top-level await (ESM only):**

```js
// fetchData.mjs
const res = await fetch('https://jsonplaceholder.typicode.com/todos/1');
console.log(await res.json());
```

> (Top-level await requires ESM.)

---

### 🧭 3. Module Resolution Rules

- **CJS resolution**: `require()` resolves synchronously using Node's CJS algorithm: look for core modules, then file paths with extensions `.js`, `.json`, `.node`, or a directory with `package.json → main` or `index.js`. Relative/absolute and node_modules search strategy apply.
- **ESM resolution in Node**: uses URLs (file URLs internally). import specifiers must be full (relative paths with extension or package exports). Node's ESM resolution is stricter (requires explicit file extensions when importing local files unless loader/customization is used). `package.json` `"type": "module"` controls defaults.

**Analogy:** CJS is like dynamic “call a store” lookup (you can ask for “utils” and the resolver figures it out); ESM is like a postal address—you must be more explicit for the mail to be delivered reliably.

---

### 🔄 4. Interop Rules (CJS ↔ ESM)

Node.js supports both CommonJS (CJS) and ES Modules (ESM), but interoperability between them has some rules:

1. **ESM importing CJS**
    - **Behavior:** When an ES module imports a CommonJS module, Node wraps the CJS module.exports as a **default export**. Named exports aren’t automatic; you access properties via the default object.

    **Example:**

    ```js
    // utils.cjs
    module.exports = {
        greet: () => console.log("Hello!"),
    };
    ```

    ```js
    //app.mjs
    import utils from './utils.cjs';
    utils.greet(); // prints "Hello!"
    ```

    > ✅ ESM → CJS works smoothly using this interop wrapper.

2. **CJS requiring ESM**

    - **Behavior:** CommonJS cannot directly require() an ES module. Doing so throws an error because **CJS doesn’t understand ESM syntax (export).**
    - **Workarounds:**
        1. **Dynamic `import` in async context:**

        ```js
        // utils.mjs
        export function greet() {
            console.log("Hello from ESM!");
        }
        ```

        ```js
        // app.cjs
        async function main() {
            const utils = await import('./utils.mjs');
            utils.greet();
        }
        main();
        ```

        2. **Dual package approach:** Publish both CJS (main field) and ESM (module field) builds in package.json. CJS consumers require the CJS build; ESM consumers import the ESM build. 
        3. **Wrapper files:** Create tiny CJS or ESM files that bridge the modules.

3. **Using createRequire() in ESM**

    - Sometimes you need to load a CJS module from an ESM file. Node provides createRequire():

    ```js
    // app.mjs
    import { createRequire } from 'module';
    const require = createRequire(import.meta.url);
    const cjsModule = require('./utils.cjs');
    cjsModule.greet();
    ```
    - **Practical pattern:**
        - Publish packages supporting both: main for CJS, module for ESM. 
        - Use createRequire() or dynamic import() to mix module types when needed.

---

### 🎯 5. When to Use Which Today

- **Prefer ESM** for new codebases, libraries aimed at modern toolchains, browser-compatible modules, or when you need static analysis and top-level await.
- **Keep CJS** for legacy projects, when depending on many CJS-only packages, or when migrating incrementally. Many production apps are hybrid.

---

### 🌍 6. Real-World Examples

- **Frameworks:** Next.js, Vite-ecosystem packages, and many modern libs publish ESM builds. Many backend services still run CJS code in older repos.
- **Libraries:** Popular libraries ship both ESM and CJS builds (dual-package) to maximize compatibility.

---

### 🔧 7. Migration: CJS → ESM (step-by-step)

- **High-level plan**
    1. Add "type": "module" in package.json **only when you’re ready to treat `.js` as ESM**. Otherwise use `.mjs`/`.cjs` extensions. 
    2. Convert files: `module.exports` = → `export default` / `export const.` `require()` → `import`. 
    3. Replace dynamic CJS patterns (e.g., conditional require() in runtime) with dynamic `import()` or keep those files as `.cjs`. 
    4. For dependencies that are CJS-only: either keep parts of your app as CJS (`.cjs`), use `createRequire()` (`import { createRequire } from 'module'`), or publish a dual build in libraries. 
    5. Update build/test tooling (transpilers, linters) to support ESM. 
    6. Add compatibility shims: e.g., for modules that expect `__dirname`/`__filename`, use `import.meta.url` and `fileURLToPath()` from url module.

- **Quick runnable example of mixed approach:**
    - Keep `server.cjs` (CJS entry) that conditionally imports `esm-main.mjs` when needed: 

    ```js
    // server.cjs
    if (process.env.USE_ESM) {
        (async()=>{ await import('./esm-main.mjs'); })();
    } else {
        const app = require('./legacy-app.cjs');
        app.start();
    }
    ```

- **Tools & tips:**  
    - Use tsc or Babel to emit ESM/CJS dual outputs if you maintain TypeScript source.
    - Prefer .mjs for gradual migration to avoid breaking existing .js behavior. 
    - Test thoroughly; resolution differences often cause missing-file errors.

---

### ⚠️ 8. Pitfalls, Performance Notes, Ecosystem Impact

- **Startup cost:** ESM loader does more upfront work (parsing static imports), but once loaded runtime is comparable. Dynamic `import()` and top-level await change startup semantics. 
- **Tooling:** older bundlers/test runners might assume CJS; update them. 
- **Interoperability surprises:** default vs named import mapping can cause undefined when consuming CJS incorrectly. 
- **Package publishing:** authoring correct exports and types in `package.json` is essential to support both consumers. 
- **DevDependencies vs production:** ensure build-time-only tools (dev deps) are not assumed available at runtime (practical deployment rule)

---

## 🔌 Part B — Custom Loaders (Node.js ESM Loaders)

### 🧐 1. What Custom Loaders Are & Why They Exist

- **Custom loaders (module customization hooks)** let you intercept Node’s module resolution/compilation for ESM; you can transform source, implement alternate specifier resolution, serve virtual modules, or support new file types (e.g., `.ts`, `.yaml`) **at runtime**. They solve integration problems like running TypeScript without precompilation or mapping bare imports to remote URLs. Node documents these hooks and their design.

---

### 🔎 2. How Node resolves and loads modules

1. **Specifier encountered** (import x from 'foo') in ESM. 
2. **Resolve hook:** convert specifier to an absolute URL (file:///... or other) — this uses Node’s resolver or a **customization hook**. 
3. **Load hook:** fetch the resource content for that URL (could be from disk, network, virtual source). 
4. **Transform/compile:** if the loader provides transforms (e.g., transpile TS → JS), code is compiled here. 
5. **Instantiate/execute:** Node evaluates the module and links exports to importers.

Loader hooks that map to these steps are typically resolve and load or getFormat/transformSource in older designs; Node's current design centralizes this under module customization hooks.

---

### 🧪 3. Example: minimal custom loader

- **Note:** Node’s loader API has evolved. The example below captures the conceptual idea; for exact API use the Node docs and current loader examples.
- **Goal:** support .yaml imports as modules exporting parsed JSON.
- **File:** yaml-loader.mjs loader entry, when Node is launched with **-- experimental-loader ./yaml-loader.mjs** historically.

```js
import { readFile } from 'fs/promises';
import yaml from 'js-yaml';

export async function resolve(specifier, context, defaultResolve) {
  if (specifier.endsWith('.yaml')) {
    return {
      ...(await defaultResolve(specifier, context, defaultResolve)),
      shortCircuit: true
    };
  }

  return defaultResolve(specifier, context, defaultResolve);
}

export async function load(url, context, defaultLoad) {
  if (url.endsWith('.yaml')) {
    const src = await readFile(new URL(url));
    const parsed = JSON.stringify(yaml.load(src.toString()));
    const jsSource = `export default ${parsed};`;

    return {
      format: 'module',
      source: jsSource,
      shortCircuit: true
    };
  }

  return defaultLoad(url, context, defaultLoad);
}
```

- **Usage:** `node --experimental-loader ./yaml-loader.mjs app.mjs` (API and flags depend on Node version).

---

### 🧰 4. Examples of building useful loaders

- **TypeScript loader:** run TypeScript files directly by transpiling in the loader (fast for dev). Many projects prefer building artifacts ahead of time (stability/security) instead of runtime transpile.
- **Virtual modules:** provide modules that don’t exist on disk (e.g., compiled templates, in-memory generated code). **Helpful in frameworks and plugin systems**.
- **Custom resolution:** implement browser-like bare specifier resolution or aliasing; useful for monorepos and bundler-less experiments.

---

### 🌐 5. Real-world use cases

- **Frameworks and build tools:** some bundlers, dev servers, or framework runtime plugins rely on loaders to support non-JS asset imports in dev mode.
- **Tooling:** quick prototyping or plugin systems where you want to **inject modules without changing disk layout**.

---

### 🛡️ 6. Security, limitations, and debugging

- **Security:** loaders run with the same privileges as your app; they can execute arbitrary code. Avoid loading remote code without validation. Beware of supply-chain attacks via third-party loaders.
- **Limitations:** Node’s loader API is versioned; loader behavior changes between Node releases — test and pin Node versions. Loaders historically are ESM-only (they don't affect CJS require()), and some loader features are experimental.
- **Debugging:** use verbose Node flags, console/logging inside the loader, and unit-tests for the loader logic. Keep transforms deterministic. Refer to Node loader design docs.

---

## 🏗️ Part C — Monorepos and Private Modules

### 🧱 1. What is a monorepo, why use it, and trade-offs

- **Monorepo:** single repository hosting multiple packages/projects (e.g., backend, frontend, shared libs). Companies use monorepos for easier code sharing, atomic refactors, consistent tooling, and simplified dependency updates.
- **Benefits:** single source of truth, easier cross-package changes, consistent versioning/CI.
- **Trade-offs:** tooling complexity, larger repository size, potential for slower CI if not optimized. Use when many related packages are maintained together; smaller independent projects may be better in separate repos.

---

### 🧰 2. Comparing workspace tools

- **npm Workspaces:** built-in to npm, supports basic workspace flows. **Good for straightforward setups and when you want minimal extra tooling.**
- **Yarn Workspaces (esp. Yarn v4):** mature workspace support, plug’n’play features, zero-installs options. **Historically popular for monorepos.**
- **pnpm Workspaces:** uses a content-addressable store and strict node_modules layout, fast installs, disk-efficient (shared store), enforces strict dependency boundaries (helps catch missing transitive deps). **Increasingly recommended for large monorepos.**

**Rule of thumb:** choose pnpm for speed and strictness; npm if you want a simple solution with minimal new toolchains; Yarn if you use Yarn-specific features.

--- 

### 🗂️ 3. Structuring a Node monorepo (example)

- **Repo layout**

```js
/my-monorepo
    /packages
        /api
            packages.json
        /ui
            packages.json
        /shared
            packages.json
    package.json
    pnpm-workspace.yaml   // (or package.json workspace field)
    turbo.json(optional)  // for task runner like Turborepo
```

- **Example `package.json` at root (pnpm workspaces):**

```json
{
    "name":"my-monorepo",
    "private": true,
    "workspace": ["packages/*"],
    "devDependencies": {
        "typescript": "^5",
        "pnpm": "^8"
    }
}
```

- **Simple shared package packages/shared/package.json**

```json
{
    "name":"myorg/shared",
    "version":"0.1.0",
    "main":"dist/index.cjs.js",
    "module":"dist/index.mjs.js",
    "types":"dist/index.d.ts"
}
```

- **Linking with pnpm:** `pnpm install` at root will symlink local packages into workspace projects.


---

### 🔐 4. Publishing private packages

- **Options**

    - **npm private packages** (scoped packages with private visibility). `npm publish --access=restricted.`
    - **GitHub Packages:** store and consume packages through GitHub’s package registry; requires token-based auth and configuration. Useful when your organization already uses GitHub.
    - **Private npm registries** (Verdaccio, GitHub, registries offered by cloud vendors) — pick based on enterprise needs.

- **Basic steps (GitHub Packages example):**
    1. Create package repository (can be same monorepo or separate). 
    2. Configure ~/.npmrc or CI to authenticate using a GitHub token with write:packages/read:packages. 
    3. npm publish (with scope and registry configured).

---

### 🏛️ 5. Design patterns for scalable monorepos

- **Package boundaries:** keep small, focused packages (single responsibility). 
- **Versioning strategy:** use independent versions per package or a single version across repo (Monorepo-wide bump). Tools: changesets, pnpm/lerna workflows.
- **CI optimization:** run tests/builds only for changed packages (use tooling like Turborepo, Changesets, or custom scripts). 
- **Enforce dependency contracts:** use strict package manager modes (pnpm’s strictness) to detect undeclared transitive 

---

### ⚠️ 6. Pitfalls & dependency management issues
- **Hoisting/duplicate versions:** some package managers hoist dependencies; pnpm’s strict node_modules layout avoids hidden transitive usage but needs adjustments. 
- CI scale: running full builds/test for monorepos without change detection is slow — use incremental builds. 
- Publishing workflows: managing internal versions and consumers requires automated release tooling (e.g., Changesets + GitHub Actions).

