# Package Structure Guide

## Overview

This document provides a detailed guide for organizing code in the tsParticles monorepo following proper contribution practices. All new features should be implemented as independent packages within the existing monorepo structure.

---

## tsParticles Monorepo Architecture

### Repository Structure

```
tsparticles-jw/
├── engine/                    # Core tsParticles engine
├── plugins/                   # Plugin packages
│   ├── absorbers/
│   ├── emitters/
│   ├── themes/
│   ├── responsive/
│   └── [new plugins here]
├── paths/                     # Path generator plugins
│   ├── grid/
│   ├── perlin-noise/
│   └── [new paths here]
├── interactions/              # Interaction plugins
│   ├── external/
│   └── particles/
├── updaters/                  # Particle updater plugins
│   ├── wobble/
│   ├── tilt/
│   └── [new updaters here]
├── shapes/                    # Shape drawer plugins
├── utils/                     # Utility packages
└── bundles/                   # Pre-configured bundles
```

### Build System

- **Package Manager:** pnpm (with workspaces)
- **Monorepo Tool:** Lerna
- **Build Tool:** TypeScript compiler (tsc)
- **Test Framework:** Vitest

---

## Proposed New Packages

### Package 1: Formation Plugin

**Location:** `plugins/formations/`

**Purpose:** Text and image formation system with morphing

**Package Name:** `@tsparticles/plugin-formations`

**Structure:**
```
plugins/formations/
├── src/
│   ├── index.ts                      # Main exports
│   ├── FormationsPlugin.ts          # IPlugin implementation
│   ├── FormationsPluginInstance.ts  # IContainerPlugin implementation
│   ├── FormationManager.ts          # Formation orchestration
│   ├── parsers/
│   │   ├── TextFormationParser.ts   # Text to coordinates
│   │   ├── ImageFormationParser.ts  # Image to coordinates
│   │   └── index.ts
│   ├── Options/
│   │   ├── Classes/
│   │   │   ├── Formation.ts
│   │   │   ├── FormationSequence.ts
│   │   │   └── index.ts
│   │   └── Interfaces/
│   │       ├── IFormation.ts
│   │       ├── IFormationSequence.ts
│   │       └── index.ts
│   └── types.ts
├── tests/
│   ├── FormationsPlugin.test.ts
│   ├── FormationManager.test.ts
│   └── parsers.test.ts
├── package.json
├── tsconfig.json
├── vitest.config.ts
└── README.md
```

**package.json:**
```json
{
  "name": "@tsparticles/plugin-formations",
  "version": "0.1.0",
  "description": "Text and image formation plugin for tsParticles",
  "author": "Jonathan Whicker",
  "license": "MIT",
  "main": "dist/index.js",
  "module": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsc",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  },
  "dependencies": {
    "@tsparticles/engine": "workspace:*"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  },
  "peerDependencies": {
    "@tsparticles/engine": "^3.0.0"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/jwhicker/tsparticles-jw.git",
    "directory": "plugins/formations"
  },
  "bugs": {
    "url": "https://github.com/jwhicker/tsparticles-jw/issues"
  },
  "publishConfig": {
    "access": "public"
  }
}
```

**tsconfig.json:**
```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

**index.ts (Main Export):**
```typescript
export * from "./FormationsPlugin.js";
export * from "./FormationManager.js";
export * from "./Options/index.js";
export * from "./types.js";

export { loadFormationsPlugin } from "./FormationsPlugin.js";
```

---

### Package 2: Formation Updater

**Location:** `updaters/formation/`

**Purpose:** Particle updater for target-seeking behavior

**Package Name:** `@tsparticles/updater-formation`

**Structure:**
```
updaters/formation/
├── src/
│   ├── index.ts
│   ├── FormationUpdater.ts
│   ├── Options/
│   │   ├── Classes/
│   │   │   └── FormationOptions.ts
│   │   └── Interfaces/
│   │       └── IFormationOptions.ts
│   └── types.ts
├── tests/
│   └── FormationUpdater.test.ts
├── package.json
├── tsconfig.json
├── vitest.config.ts
└── README.md
```

**package.json:**
```json
{
  "name": "@tsparticles/updater-formation",
  "version": "0.1.0",
  "description": "Formation target updater for tsParticles",
  "author": "Jonathan Whicker",
  "license": "MIT",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    }
  },
  "scripts": {
    "build": "tsc",
    "test": "vitest run"
  },
  "dependencies": {
    "@tsparticles/engine": "workspace:*"
  },
  "peerDependencies": {
    "@tsparticles/engine": "^3.0.0"
  }
}
```

**index.ts:**
```typescript
export * from "./FormationUpdater.js";
export * from "./Options/index.js";

import type { Engine } from "@tsparticles/engine";
import { FormationUpdater } from "./FormationUpdater.js";

export async function loadFormationUpdater(engine: Engine): Promise<void> {
  await engine.addParticleUpdater("formation", () => new FormationUpdater());
}
```

---

### Package 3: Next.js Integration Utilities

**Location:** `utils/nextjs-integration/`

**Purpose:** React/Next.js integration utilities and configuration framework

**Package Name:** `@tsparticles/nextjs-integration`

**Structure:**
```
utils/nextjs-integration/
├── src/
│   ├── index.ts
│   ├── contexts/
│   │   └── ParticlesContext.tsx
│   ├── hooks/
│   │   ├── useParticles.ts
│   │   ├── useRouteParticles.ts
│   │   └── useReducedMotion.ts
│   ├── config/
│   │   ├── design-tokens.ts
│   │   ├── particle-presets.ts
│   │   └── route-particles.ts
│   ├── lib/
│   │   ├── config-converter.ts
│   │   ├── route-matcher.ts
│   │   ├── type-guards.ts
│   │   └── transitions.ts
│   ├── types/
│   │   └── particle-config.ts
│   └── presets/
│       ├── loading-effects.ts
│       └── transition-effects.ts
├── tests/
│   ├── contexts/
│   ├── lib/
│   └── config/
├── package.json
├── tsconfig.json
├── vitest.config.ts
└── README.md
```

**package.json:**
```json
{
  "name": "@tsparticles/nextjs-integration",
  "version": "0.1.0",
  "description": "Next.js and React integration utilities for tsParticles",
  "author": "Jonathan Whicker",
  "license": "MIT",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./config": {
      "types": "./dist/config/index.d.ts",
      "import": "./dist/config/index.js"
    },
    "./contexts": {
      "types": "./dist/contexts/index.d.ts",
      "import": "./dist/contexts/index.js"
    },
    "./hooks": {
      "types": "./dist/hooks/index.d.ts",
      "import": "./dist/hooks/index.js"
    }
  },
  "scripts": {
    "build": "tsc",
    "test": "vitest run"
  },
  "dependencies": {
    "@tsparticles/engine": "workspace:*",
    "@tsparticles/react": "workspace:*"
  },
  "peerDependencies": {
    "@tsparticles/engine": "^3.0.0",
    "react": "^18.0.0 || ^19.0.0",
    "next": "^15.0.0 || ^16.0.0"
  }
}
```

**index.ts:**
```typescript
// Main exports
export * from "./contexts/ParticlesContext.js";
export * from "./hooks/index.js";
export * from "./types/particle-config.js";

// Config exports (also available via "@tsparticles/nextjs-integration/config")
export * from "./config/design-tokens.js";
export * from "./config/particle-presets.js";

// Lib exports
export * from "./lib/config-converter.js";
export * from "./lib/route-matcher.js";
```

---

## Workspace Configuration

### Root-Level Configuration Files

#### lerna.json

```json
{
  "version": "independent",
  "npmClient": "pnpm",
  "packages": [
    "engine",
    "plugins/*",
    "paths/*",
    "interactions/*",
    "updaters/*",
    "shapes/*",
    "utils/*",
    "bundles/*"
  ],
  "command": {
    "publish": {
      "conventionalCommits": true,
      "message": "chore(release): publish"
    },
    "version": {
      "allowBranch": ["main", "v2", "v3"],
      "message": "chore(release): version"
    }
  }
}
```

#### pnpm-workspace.yaml

```yaml
packages:
  - 'engine'
  - 'plugins/*'
  - 'paths/*'
  - 'interactions/*'
  - 'updaters/*'
  - 'shapes/*'
  - 'utils/*'
  - 'bundles/*'
```

#### Root tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ES2020",
    "lib": ["ES2020", "DOM"],
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}
```

---

## Build & Test Commands

### Building Packages

```bash
# Install dependencies
pnpm install

# Build all packages
lerna run build

# Build specific package
cd plugins/formations && pnpm build

# Build with watch mode
cd plugins/formations && pnpm build --watch
```

### Running Tests

```bash
# Test all packages
lerna run test

# Test specific package
cd plugins/formations && pnpm test

# Test with coverage
cd plugins/formations && pnpm test:coverage

# Watch mode
cd plugins/formations && pnpm test:watch
```

### Linting

```bash
# Lint all packages
lerna run lint

# Lint specific package
cd plugins/formations && pnpm lint
```

---

## Package Dependencies

### Dependency Rules

1. **Engine Dependency:** All plugins depend on `@tsparticles/engine`
   ```json
   {
     "dependencies": {
       "@tsparticles/engine": "workspace:*"
     },
     "peerDependencies": {
       "@tsparticles/engine": "^3.0.0"
     }
   }
   ```

2. **Cross-Package Dependencies:** Use workspace protocol
   ```json
   {
     "dependencies": {
       "@tsparticles/plugin-formations": "workspace:*"
     }
   }
   ```

3. **External Dependencies:** Specific versions
   ```json
   {
     "dependencies": {
       "react": "^19.0.0"
     }
   }
   ```

### Dependency Graph

```
@tsparticles/engine (core)
    ↓
@tsparticles/plugin-formations (depends on engine)
    ↓
@tsparticles/updater-formation (depends on engine)
    ↓
@tsparticles/nextjs-integration (depends on engine, formations, updater)
```

---

## Plugin Registration Patterns

### Standard Plugin Registration

```typescript
// plugins/formations/src/FormationsPlugin.ts
import type { IPlugin, Container, IContainerPlugin } from "@tsparticles/engine";

export class FormationsPlugin implements IPlugin {
  readonly id = "formations";

  async getPlugin(container: Container): Promise<IContainerPlugin> {
    const { FormationsPluginInstance } = await import("./FormationsPluginInstance.js");
    return new FormationsPluginInstance(container);
  }

  loadOptions(container: Container, options: any, source?: any): void {
    // Load plugin options
  }

  needsPlugin(options?: any): boolean {
    return !!options?.formations?.length;
  }
}

// Export loader function
export async function loadFormationsPlugin(engine: Engine): Promise<void> {
  await engine.addPlugin(new FormationsPlugin());
}
```

### Usage in Applications

```typescript
import { tsParticles } from "@tsparticles/engine";
import { loadFormationsPlugin } from "@tsparticles/plugin-formations";
import { loadFormationUpdater } from "@tsparticles/updater-formation";

async function initParticles() {
  // Load plugins
  await loadFormationsPlugin(tsParticles);
  await loadFormationUpdater(tsParticles);

  // Use in configuration
  await tsParticles.load({
    id: "particles",
    options: {
      formations: [
        {
          id: "welcome",
          type: "text",
          options: {
            text: "WELCOME",
            fontSize: 48,
          },
        },
      ],
    },
  });
}
```

---

## Testing Structure

### Test Organization

Each package has its own test suite:

```
plugins/formations/
├── src/
└── tests/
    ├── unit/
    │   ├── FormationManager.test.ts
    │   └── parsers.test.ts
    ├── integration/
    │   └── FormationsPlugin.integration.test.ts
    └── setup.ts
```

### Vitest Configuration

**vitest.config.ts (per package):**
```typescript
import { defineConfig } from 'vitest/config';
import path from 'path';

export default defineConfig({
  test: {
    environment: 'jsdom',
    globals: true,
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'dist/',
        '**/*.test.ts',
        '**/*.test.tsx',
      ],
      thresholds: {
        lines: 90,
        branches: 85,
        functions: 90,
        statements: 90,
      },
    },
    setupFiles: ['./tests/setup.ts'],
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

---

## Example: Complete Package Setup

### Step-by-Step: Creating @tsparticles/plugin-formations

**Step 1: Create Directory Structure**
```bash
mkdir -p plugins/formations/src/{Options/{Classes,Interfaces},parsers}
mkdir -p plugins/formations/tests
```

**Step 2: Create package.json**
```bash
cd plugins/formations
cat > package.json << 'EOF'
{
  "name": "@tsparticles/plugin-formations",
  "version": "0.1.0",
  "description": "Text and image formation plugin for tsParticles",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "vitest run"
  },
  "dependencies": {
    "@tsparticles/engine": "workspace:*"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
EOF
```

**Step 3: Create tsconfig.json**
```bash
cat > tsconfig.json << 'EOF'
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
EOF
```

**Step 4: Install Dependencies**
```bash
cd ../..  # Back to root
pnpm install
```

**Step 5: Build**
```bash
cd plugins/formations
pnpm build
```

**Step 6: Verify**
```bash
ls -la dist/  # Should see compiled .js and .d.ts files
```

---

## README Template for Packages

Each package needs a README.md:

```markdown
# @tsparticles/plugin-formations

Text and image formation plugin for tsParticles.

## Installation

```bash
npm install @tsparticles/plugin-formations
# or
pnpm add @tsparticles/plugin-formations
```

## Usage

```typescript
import { tsParticles } from "@tsparticles/engine";
import { loadFormationsPlugin } from "@tsparticles/plugin-formations";

// Load the plugin
await loadFormationsPlugin(tsParticles);

// Use in configuration
await tsParticles.load({
  id: "particles",
  options: {
    formations: [
      {
        id: "welcome",
        type: "text",
        options: {
          text: "WELCOME",
          fontSize: 48,
        },
      },
    ],
  },
});
```

## API

### `loadFormationsPlugin(engine: Engine): Promise<void>`

Loads the formations plugin into the tsParticles engine.

## Options

See [Options Documentation](./docs/options.md)

## License

MIT
```

---

## Publishing Packages

### Preparation

1. Ensure all tests pass
2. Build all packages
3. Update CHANGELOG.md
4. Update version numbers

### Commands

```bash
# Publish all packages
lerna publish

# Publish specific package
cd plugins/formations
pnpm publish

# Publish with specific tag
lerna publish --dist-tag next
```

---

## Summary

Following this package structure ensures:

1. ✅ **Proper separation of concerns** - Each feature is independent
2. ✅ **Follows tsParticles conventions** - Matches existing repo structure
3. ✅ **Independent testing** - Each package has its own tests
4. ✅ **Modular installation** - Users install only what they need
5. ✅ **Easy contribution** - Clear structure for adding features
6. ✅ **Monorepo benefits** - Shared tooling and dependencies

This structure allows the fork to maintain compatibility with upstream tsParticles while adding new features in a clean, maintainable way.
