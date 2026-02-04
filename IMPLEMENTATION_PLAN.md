# Multi-Phase Implementation Plan

## Overview

This document provides a complete, actionable implementation plan for adding the particle grid configuration framework to the tsParticles repository. The plan follows Test-Driven Development (TDD) principles, references all design documents, and provides clear deliverables for each phase.

---

## Plan Structure

- **Phase 1:** Foundation & Core Configuration System (Week 1)
- **Phase 2:** Next.js Integration & Effects Library (Week 2)
- **Phase 3:** Text/Image Formation Plugin (Weeks 3-5)
- **Phase 4:** Polish, Documentation & Examples (Week 6)

Each phase follows TDD:
1. Write tests first
2. Implement to pass tests
3. Refactor for quality
4. Document and validate

---

## Phase 1: Foundation & Core Configuration System

**Duration:** 1 week  
**Goal:** Create the type-safe configuration system and route-based particle management  
**Reference:** [ARCHITECTURE_REVIEW.md](./ARCHITECTURE_REVIEW.md), [NEXTJS_CONFIGURATION_FRAMEWORK.md](./NEXTJS_CONFIGURATION_FRAMEWORK.md)

### 1.1: Type System Definition (Day 1)

#### TDD Approach

**Step 1: Write Type Tests**
```typescript
// tests/types/particle-config.test.ts
import { describe, it, expect } from 'vitest';
import type { 
  IParticleRouteConfig, 
  IRouteParticleMap,
  InteractionMode,
  GridStyle 
} from '@/types/particle-config';

describe('Type System', () => {
  it('should accept valid route configuration', () => {
    const config: IParticleRouteConfig = {
      gridStyle: 'perlin-noise',
      particleCount: 80,
      hoverMode: 'grab',
      clickMode: 'push',
    };
    expect(config).toBeDefined();
  });

  it('should accept array of interaction modes', () => {
    const config: IParticleRouteConfig = {
      hoverMode: ['grab', 'bubble', 'connect'],
    };
    expect(config).toBeDefined();
  });

  it('should enforce readonly on formation arrays', () => {
    const config: IParticleRouteConfig = {
      textFormations: [
        { text: 'TEST', duration: 1000 },
      ],
    };
    // TypeScript should enforce readonly - this would fail at compile time:
    // config.textFormations[0] = { text: 'CHANGED' }; // Error!
    expect(config.textFormations).toBeDefined();
  });
});
```

**Step 2: Implement Types**
```typescript
// types/particle-config.ts
import type { ISourceOptions, RecursivePartial } from '@tsparticles/engine';

export type InteractionMode = 
  | 'grab' 
  | 'attract' 
  | 'repulse' 
  | 'bubble' 
  | 'push' 
  | 'connect' 
  | 'trail' 
  | 'bounce'
  | 'parallax'
  | 'none';

export type GridStyle = 
  | 'perlin-noise' 
  | 'simplex-noise' 
  | 'curl-noise' 
  | 'grid' 
  | 'organic';

export interface ITextFormation {
  readonly text: string;
  readonly duration?: number;
  readonly delay?: number;
  readonly font?: string;
  readonly fontSize?: number;
}

export interface IParticleRouteConfig {
  readonly gridStyle?: GridStyle;
  readonly particleCount?: number;
  readonly hoverMode?: InteractionMode | ReadonlyArray<InteractionMode>;
  readonly clickMode?: InteractionMode | ReadonlyArray<InteractionMode>;
  readonly transitionIn?: string;
  readonly transitionOut?: string;
  readonly textFormations?: ReadonlyArray<ITextFormation>;
  readonly useSystemTheme?: boolean;
  readonly forceTheme?: 'light' | 'dark';
  readonly customOptions?: RecursivePartial<ISourceOptions>;
}

export interface IRouteParticleMap {
  readonly [route: string]: IParticleRouteConfig;
}
```

**Step 3: Run Tests**
```bash
npm test -- types/particle-config.test.ts
```

**Deliverables:**
- ✅ Type definitions in `types/particle-config.ts`
- ✅ Type tests passing
- ✅ JSDoc comments on all interfaces

---

### 1.2: Design Token System (Day 1-2)

#### TDD Approach

**Step 1: Write Token Tests**
```typescript
// tests/config/design-tokens.test.ts
import { describe, it, expect } from 'vitest';
import { designTokens } from '@/config/design-tokens';

describe('Design Tokens', () => {
  it('should have light and dark colors for all token types', () => {
    expect(designTokens.colors.primary.light).toBeDefined();
    expect(designTokens.colors.primary.dark).toBeDefined();
    expect(designTokens.colors.particle.light).toBeDefined();
    expect(designTokens.colors.particle.dark).toBeDefined();
  });

  it('should have consistent spacing values', () => {
    expect(designTokens.spacing.xs).toBe('0.25rem');
    expect(designTokens.spacing['2xl']).toBe('3rem');
  });

  it('should have particle density values for all breakpoints', () => {
    expect(designTokens.particles.density.mobile).toBeLessThan(
      designTokens.particles.density.tablet
    );
    expect(designTokens.particles.density.tablet).toBeLessThan(
      designTokens.particles.density.desktop
    );
  });
});
```

**Step 2: Write CSS Token Utility Tests**
```typescript
// tests/lib/css-tokens.test.ts
import { describe, it, expect } from 'vitest';
import { generateCSSVariables } from '@/lib/css-tokens';

describe('CSS Token Utilities', () => {
  it('should generate valid CSS variables for light theme', () => {
    const css = generateCSSVariables('light');
    expect(css).toContain('--color-primary:');
    expect(css).toContain('--color-particle:');
    expect(css).toContain('--space-md:');
  });

  it('should generate different values for dark theme', () => {
    const lightCss = generateCSSVariables('light');
    const darkCss = generateCSSVariables('dark');
    expect(lightCss).not.toBe(darkCss);
  });
});
```

**Step 3: Implement Design Tokens**
```typescript
// config/design-tokens.ts
export const designTokens = {
  colors: {
    primary: { light: '#3b82f6', dark: '#60a5fa' },
    secondary: { light: '#8b5cf6', dark: '#a78bfa' },
    // ... rest of implementation from ARCHITECTURE_REVIEW.md
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    // ... rest
  },
  particles: {
    density: { mobile: 40, tablet: 60, desktop: 80 },
    size: { min: 1, max: 3 },
    opacity: { min: 0.3, max: 0.7 },
    speed: { slow: 0.5, medium: 1, fast: 2 },
  },
} as const;
```

**Step 4: Run Tests**
```bash
npm test -- config/design-tokens.test.ts
npm test -- lib/css-tokens.test.ts
```

**Deliverables:**
- ✅ Design tokens in `config/design-tokens.ts`
- ✅ CSS generation utilities in `lib/css-tokens.ts`
- ✅ All token tests passing

---

### 1.3: Configuration Converter (Day 2-3)

#### TDD Approach

**Step 1: Write Converter Tests**
```typescript
// tests/lib/config-converter.test.ts
import { describe, it, expect } from 'vitest';
import { convertConfigToOptions } from '@/lib/config-converter';
import { designTokens } from '@/config/design-tokens';

describe('Configuration Converter', () => {
  it('should convert basic config to tsParticles options', () => {
    const config = {
      gridStyle: 'perlin-noise' as const,
      particleCount: 80,
      hoverMode: 'grab' as const,
    };

    const options = convertConfigToOptions(config, designTokens, 'light');

    expect(options.particles?.number?.value).toBe(80);
    expect(options.particles?.move?.path?.generator).toBe('perlinNoise');
    expect(options.interactivity?.events?.onHover?.mode).toContain('grab');
  });

  it('should handle array of interaction modes', () => {
    const config = {
      hoverMode: ['grab', 'bubble', 'connect'] as const,
    };

    const options = convertConfigToOptions(config, designTokens, 'light');
    const modes = options.interactivity?.events?.onHover?.mode;

    expect(Array.isArray(modes)).toBe(true);
    expect(modes).toContain('grab');
    expect(modes).toContain('bubble');
    expect(modes).toContain('connect');
  });

  it('should apply theme colors correctly', () => {
    const config = { particleCount: 50 };

    const lightOptions = convertConfigToOptions(config, designTokens, 'light');
    const darkOptions = convertConfigToOptions(config, designTokens, 'dark');

    expect(lightOptions.particles?.color?.value).toBe(
      designTokens.colors.particle.light
    );
    expect(darkOptions.particles?.color?.value).toBe(
      designTokens.colors.particle.dark
    );
  });

  it('should merge custom options correctly', () => {
    const config = {
      particleCount: 50,
      customOptions: {
        particles: {
          opacity: {
            value: 0.9,
          },
        },
      },
    };

    const options = convertConfigToOptions(config, designTokens, 'light');
    expect(options.particles?.opacity?.value).toBe(0.9);
    expect(options.particles?.number?.value).toBe(50); // Should not override
  });
});
```

**Step 2: Write Type Guard Tests**
```typescript
// tests/lib/type-guards.test.ts
import { describe, it, expect } from 'vitest';
import { isObject, isValidGridStyle } from '@/lib/type-guards';

describe('Type Guards', () => {
  it('should identify objects correctly', () => {
    expect(isObject({})).toBe(true);
    expect(isObject({ key: 'value' })).toBe(true);
    expect(isObject(null)).toBe(false);
    expect(isObject([])).toBe(false);
    expect(isObject('string')).toBe(false);
  });

  it('should validate grid styles', () => {
    expect(isValidGridStyle('perlin-noise')).toBe(true);
    expect(isValidGridStyle('invalid')).toBe(false);
  });
});
```

**Step 3: Implement Converter** (from ARCHITECTURE_REVIEW.md)
```typescript
// lib/config-converter.ts
import type { ISourceOptions } from '@tsparticles/engine';
import type { IParticleRouteConfig, IDesignTokens } from '@/types/particle-config';

function isObject(value: unknown): value is Record<string, unknown> {
  return value !== null && typeof value === 'object' && !Array.isArray(value);
}

function deepMerge<T extends Record<string, unknown>>(
  target: T, 
  source: RecursivePartial<T>
): T {
  // Implementation from ARCHITECTURE_REVIEW.md
}

export function convertConfigToOptions(
  config: IParticleRouteConfig,
  tokens: IDesignTokens,
  theme: 'light' | 'dark'
): ISourceOptions {
  // Full implementation from ARCHITECTURE_REVIEW.md
}
```

**Step 4: Run Tests**
```bash
npm test -- lib/config-converter.test.ts
npm test -- lib/type-guards.test.ts
```

**Deliverables:**
- ✅ Configuration converter in `lib/config-converter.ts`
- ✅ Type guards in `lib/type-guards.ts`
- ✅ All converter tests passing

---

### 1.4: Route Matcher Utility (Day 3-4)

#### TDD Approach

**Step 1: Write Route Matcher Tests**
```typescript
// tests/lib/route-matcher.test.ts
import { describe, it, expect } from 'vitest';
import { getRouteConfig, validateRouteConfig } from '@/lib/route-matcher';

describe('Route Matcher', () => {
  it('should match exact routes', () => {
    const config = getRouteConfig('/about');
    expect(config).toBeDefined();
    expect(config.gridStyle).toBe('organic'); // from route-particles config
  });

  it('should match dynamic routes with patterns', () => {
    const config = getRouteConfig('/products/123');
    expect(config).toBeDefined();
    // Should match /products/[id] pattern
  });

  it('should match partial routes', () => {
    const config = getRouteConfig('/blog/my-post');
    expect(config).toBeDefined();
    // Should match /blog config
  });

  it('should return default for unmatched routes', () => {
    const config = getRouteConfig('/unknown-route');
    expect(config).toBeDefined();
    expect(config.gridStyle).toBe('perlin-noise'); // default
  });

  it('should prioritize exact matches over patterns', () => {
    const exactConfig = getRouteConfig('/contact');
    const patternConfig = getRouteConfig('/contact/form');
    expect(exactConfig).not.toBe(patternConfig);
  });
});

describe('Route Config Validation', () => {
  it('should validate valid configurations', () => {
    const config = { particleCount: 80 };
    expect(validateRouteConfig(config)).toBe(true);
  });

  it('should reject negative particle counts', () => {
    const config = { particleCount: -10 };
    expect(validateRouteConfig(config)).toBe(false);
  });

  it('should reject invalid grid styles', () => {
    const config = { gridStyle: 'invalid-style' as any };
    expect(validateRouteConfig(config)).toBe(false);
  });
});
```

**Step 2: Implement Route Matcher** (from ARCHITECTURE_REVIEW.md)
```typescript
// lib/route-matcher.ts
import type { IParticleRouteConfig } from '@/types/particle-config';

function patternToRegex(pattern: string): RegExp {
  const escaped = pattern.replace(/\[.*?\]/g, '[^/]+');
  return new RegExp(`^${escaped}$`);
}

export function getRouteConfig(pathname: string): IParticleRouteConfig {
  // Implementation from ARCHITECTURE_REVIEW.md
}

export function validateRouteConfig(config: IParticleRouteConfig): boolean {
  // Implementation from ARCHITECTURE_REVIEW.md
}
```

**Step 3: Run Tests**
```bash
npm test -- lib/route-matcher.test.ts
```

**Deliverables:**
- ✅ Route matcher in `lib/route-matcher.ts`
- ✅ All route matching tests passing
- ✅ Edge cases covered (exact, pattern, partial, default)

---

### 1.5: Route Configuration Definitions (Day 4)

#### TDD Approach

**Step 1: Write Config Tests**
```typescript
// tests/config/route-particles.test.ts
import { describe, it, expect } from 'vitest';
import { routeParticleConfig, defaultParticleConfig } from '@/config/route-particles';

describe('Route Particle Configuration', () => {
  it('should have configurations for all defined routes', () => {
    expect(routeParticleConfig['/']).toBeDefined();
    expect(routeParticleConfig['/about']).toBeDefined();
    expect(routeParticleConfig['/projects']).toBeDefined();
  });

  it('should have valid interaction modes', () => {
    const homeConfig = routeParticleConfig['/'];
    expect(['grab', 'attract', 'repulse', 'bubble', 'push', 'none']).toContain(
      Array.isArray(homeConfig.hoverMode) 
        ? homeConfig.hoverMode[0] 
        : homeConfig.hoverMode
    );
  });

  it('should have default configuration', () => {
    expect(defaultParticleConfig).toBeDefined();
    expect(defaultParticleConfig.gridStyle).toBeDefined();
  });
});
```

**Step 2: Implement Route Configs**
```typescript
// config/route-particles.ts
import type { IRouteParticleMap, IParticleRouteConfig } from '@/types/particle-config';
import { particlePresets } from './particle-presets';

export const routeParticleConfig: IRouteParticleMap = {
  '/': {
    ...particlePresets.default,
    particleCount: 100,
    hoverMode: ['grab', 'bubble'],
    clickMode: 'push',
  },
  '/about': {
    ...particlePresets.minimal,
    gridStyle: 'organic',
    hoverMode: 'parallax',
  },
  // ... rest from NEXTJS_CONFIGURATION_FRAMEWORK.md
};

export const defaultParticleConfig: IParticleRouteConfig = particlePresets.default;
```

**Step 3: Run Tests**
```bash
npm test -- config/route-particles.test.ts
```

**Deliverables:**
- ✅ Route configurations in `config/route-particles.ts`
- ✅ Particle presets in `config/particle-presets.ts`
- ✅ All configuration tests passing

---

### 1.6: Phase 1 Integration Testing (Day 5)

**Integration Test Suite**
```typescript
// tests/integration/phase1.test.ts
import { describe, it, expect } from 'vitest';
import { getRouteConfig } from '@/lib/route-matcher';
import { convertConfigToOptions } from '@/lib/config-converter';
import { designTokens } from '@/config/design-tokens';

describe('Phase 1 Integration', () => {
  it('should convert route config to valid tsParticles options', () => {
    const routeConfig = getRouteConfig('/projects');
    const options = convertConfigToOptions(routeConfig, designTokens, 'light');

    expect(options).toBeDefined();
    expect(options.particles).toBeDefined();
    expect(options.interactivity).toBeDefined();
  });

  it('should handle all defined routes', () => {
    const routes = ['/', '/about', '/projects', '/contact', '/blog'];
    
    routes.forEach(route => {
      const config = getRouteConfig(route);
      const options = convertConfigToOptions(config, designTokens, 'light');
      expect(options).toBeDefined();
    });
  });

  it('should apply theme changes correctly', () => {
    const config = getRouteConfig('/');
    const lightOptions = convertConfigToOptions(config, designTokens, 'light');
    const darkOptions = convertConfigToOptions(config, designTokens, 'dark');

    expect(lightOptions.particles?.color?.value).not.toBe(
      darkOptions.particles?.color?.value
    );
  });
});
```

**Run Full Phase 1 Tests**
```bash
npm test
```

**Phase 1 Deliverables Summary:**
- ✅ Complete type system with strict TypeScript
- ✅ Design token architecture
- ✅ Configuration converter (type-safe)
- ✅ Route matching utility
- ✅ Route configuration definitions
- ✅ 100% test coverage for core functionality
- ✅ All tests passing

---

## Phase 2: Next.js Integration & Effects Library

**Duration:** 1 week  
**Goal:** Create React Context, Next.js integration, and transition effects  
**Reference:** [NEXTJS_CONFIGURATION_FRAMEWORK.md](./NEXTJS_CONFIGURATION_FRAMEWORK.md), [ARCHITECTURE_REVIEW.md](./ARCHITECTURE_REVIEW.md)

### 2.1: Particles Context Provider (Day 6-7)

#### TDD Approach

**Step 1: Write Context Tests**
```typescript
// tests/contexts/ParticlesContext.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, waitFor } from '@testing-library/react';
import { ParticlesProvider, useParticles } from '@/contexts/ParticlesContext';

describe('ParticlesContext', () => {
  it('should provide particles context', () => {
    const TestComponent = () => {
      const { container, isReady } = useParticles();
      return <div data-testid="status">{isReady ? 'ready' : 'loading'}</div>;
    };

    const { getByTestId } = render(
      <ParticlesProvider>
        <TestComponent />
      </ParticlesProvider>
    );

    expect(getByTestId('status')).toBeDefined();
  });

  it('should throw error when used outside provider', () => {
    const TestComponent = () => {
      useParticles();
      return <div />;
    };

    expect(() => render(<TestComponent />)).toThrow();
  });

  it('should handle theme changes', async () => {
    const TestComponent = () => {
      const { theme, setTheme } = useParticles();
      return (
        <button onClick={() => setTheme('dark')}>
          {theme}
        </button>
      );
    };

    const { getByRole } = render(
      <ParticlesProvider>
        <TestComponent />
      </ParticlesProvider>
    );

    const button = getByRole('button');
    expect(button.textContent).toBe('light');

    button.click();
    await waitFor(() => {
      expect(button.textContent).toBe('dark');
    });
  });

  it('should handle errors gracefully', async () => {
    const TestComponent = () => {
      const { error } = useParticles();
      return <div data-testid="error">{error?.message || 'no error'}</div>;
    };

    const { getByTestId } = render(
      <ParticlesProvider>
        <TestComponent />
      </ParticlesProvider>
    );

    expect(getByTestId('error').textContent).toBe('no error');
  });
});
```

**Step 2: Implement Context** (from ARCHITECTURE_REVIEW.md)
```typescript
// contexts/ParticlesContext.tsx
'use client';

import {
  createContext,
  useContext,
  useRef,
  useEffect,
  useState,
  useCallback,
  useMemo,
  type ReactNode,
} from 'react';
// Full implementation from ARCHITECTURE_REVIEW.md Part 4
```

**Step 3: Run Tests**
```bash
npm test -- contexts/ParticlesContext.test.tsx
```

**Deliverables:**
- ✅ Particles context with error handling
- ✅ Memoized callbacks and values
- ✅ Proper cleanup
- ✅ All context tests passing

---

### 2.2: Plugin Architecture (Day 7-8)

#### TDD Approach

**Step 1: Write Plugin Tests**
```typescript
// tests/lib/particle-route-plugin.test.ts
import { describe, it, expect } from 'vitest';
import { RouteParticlePlugin } from '@/lib/particle-route-plugin';

describe('RouteParticlePlugin', () => {
  it('should have correct plugin id', () => {
    const plugin = new RouteParticlePlugin();
    expect(plugin.id).toBe('routeParticles');
  });

  it('should create plugin instance', async () => {
    const plugin = new RouteParticlePlugin();
    const mockContainer = {} as any;
    const instance = await plugin.getPlugin(mockContainer);
    expect(instance).toBeDefined();
  });

  it('should always need plugin', () => {
    const plugin = new RouteParticlePlugin();
    expect(plugin.needsPlugin()).toBe(true);
  });
});
```

**Step 2: Implement Plugin** (from ARCHITECTURE_REVIEW.md)
```typescript
// lib/particle-route-plugin.ts
import type { Container, IPlugin, IContainerPlugin } from '@tsparticles/engine';
// Full implementation from ARCHITECTURE_REVIEW.md Part 3
```

**Step 3: Run Tests**
```bash
npm test -- lib/particle-route-plugin.test.ts
```

**Deliverables:**
- ✅ RouteParticlePlugin implementing IPlugin
- ✅ Plugin instance with lifecycle hooks
- ✅ All plugin tests passing

---

### 2.3: Transition Effects (Day 8-9)

#### TDD Approach

**Step 1: Write Transition Tests**
```typescript
// tests/lib/transitions.test.ts
import { describe, it, expect, vi } from 'vitest';
import { applyTransitionEffect } from '@/lib/transitions';

describe('Transition Effects', () => {
  it('should apply fade-in effect', async () => {
    const mockContainer = {
      particles: {
        array: [
          { opacity: { value: 1 } },
          { opacity: { value: 1 } },
        ],
      },
    } as any;

    await applyTransitionEffect(mockContainer, 'fade-in', 'in');
    
    // Should animate opacity
    expect(mockContainer.particles.array[0].opacity.value).toBeDefined();
  });

  it('should apply swirl effect', async () => {
    const mockContainer = {
      loadOptions: vi.fn(),
    } as any;

    await applyTransitionEffect(mockContainer, 'swirl', 'in');
    expect(mockContainer.loadOptions).toHaveBeenCalled();
  });

  it('should handle invalid effects gracefully', async () => {
    const mockContainer = {} as any;
    await expect(
      applyTransitionEffect(mockContainer, 'invalid' as any, 'in')
    ).resolves.not.toThrow();
  });
});
```

**Step 2: Implement Transitions**
```typescript
// lib/transitions.ts
import type { Container } from '@tsparticles/engine';
import type { AnimationEffect } from '@/types/particle-config';

export async function applyTransitionEffect(
  container: Container,
  effect: AnimationEffect,
  direction: 'in' | 'out'
): Promise<void> {
  // Implementation from NEXTJS_CONFIGURATION_FRAMEWORK.md Part 5
}
```

**Step 3: Run Tests**
```bash
npm test -- lib/transitions.test.ts
```

**Deliverables:**
- ✅ Transition effects library
- ✅ All transition tests passing
- ✅ Error handling for invalid effects

---

### 2.4: Loading State Effects (Day 9-10)

#### TDD Approach

**Step 1: Write Loading Effect Tests**
```typescript
// tests/presets/loading-effects.test.ts
import { describe, it, expect } from 'vitest';
import { loadingEffects } from '@/presets/loading-effects';

describe('Loading Effects', () => {
  it('should have vortex effect configuration', () => {
    expect(loadingEffects.vortex).toBeDefined();
    expect(loadingEffects.vortex.emitters).toBeDefined();
    expect(loadingEffects.vortex.absorbers).toBeDefined();
  });

  it('should have pulsating effect configuration', () => {
    expect(loadingEffects.pulsatingCenter).toBeDefined();
  });

  it('should have orbital spinner effect', () => {
    expect(loadingEffects.orbitalSpinner).toBeDefined();
  });
});
```

**Step 2: Implement Loading Effects**
```typescript
// presets/loading-effects.ts
import type { ISourceOptions } from '@tsparticles/engine';

export const loadingEffects = {
  vortex: {
    // Implementation from PARTICLE_GRID_IDEATION.md
  },
  pulsatingCenter: {
    // Implementation
  },
  orbitalSpinner: {
    // Implementation
  },
} as const;
```

**Step 3: Run Tests**
```bash
npm test -- presets/loading-effects.test.ts
```

**Deliverables:**
- ✅ Loading effect presets
- ✅ Loading state hook
- ✅ All loading effect tests passing

---

### 2.5: Phase 2 Integration Testing (Day 10)

**Integration Test Suite**
```typescript
// tests/integration/phase2.test.tsx
import { describe, it, expect } from 'vitest';
import { render } from '@testing-library/react';
import { ParticlesProvider } from '@/contexts/ParticlesContext';

describe('Phase 2 Integration', () => {
  it('should render particles provider with children', () => {
    const { container } = render(
      <ParticlesProvider>
        <div data-testid="child">Child</div>
      </ParticlesProvider>
    );

    expect(container.querySelector('[data-testid="child"]')).toBeDefined();
    expect(container.querySelector('canvas')).toBeDefined();
  });

  it('should handle route changes', async () => {
    // Test route-based configuration changes
  });

  it('should apply loading effects', async () => {
    // Test loading state effects
  });
});
```

**Run Full Phase 2 Tests**
```bash
npm test
```

**Phase 2 Deliverables Summary:**
- ✅ Particles Context Provider with error boundaries
- ✅ Plugin architecture (IPlugin/IContainerPlugin)
- ✅ Transition effects library
- ✅ Loading state effects
- ✅ All React hooks properly memoized
- ✅ 100% test coverage
- ✅ All tests passing

---

## Phase 3: Text/Image Formation Plugin

**Duration:** 3 weeks  
**Goal:** Implement dot-matrix text/image formations with morphing  
**Reference:** [PARTICLE_GRID_IDEATION.md](./PARTICLE_GRID_IDEATION.md) Section "Approach 1"

### 3.1: Formation Parser (Week 3)

#### TDD Approach

**Step 1: Write Parser Tests**
```typescript
// tests/plugins/formations/parsers.test.ts
import { describe, it, expect } from 'vitest';
import { TextFormationParser, ImageFormationParser } from '@/plugins/formations/parsers';

describe('TextFormationParser', () => {
  it('should parse text to coordinate array', () => {
    const parser = new TextFormationParser();
    const coordinates = parser.parse({
      text: 'TEST',
      font: 'Arial',
      fontSize: 48,
      resolution: 5,
    });

    expect(Array.isArray(coordinates)).toBe(true);
    expect(coordinates.length).toBeGreaterThan(0);
    expect(coordinates[0]).toHaveProperty('x');
    expect(coordinates[0]).toHaveProperty('y');
  });

  it('should respect resolution parameter', () => {
    const parser = new TextFormationParser();
    const highRes = parser.parse({
      text: 'A',
      fontSize: 48,
      resolution: 10,
    });
    const lowRes = parser.parse({
      text: 'A',
      fontSize: 48,
      resolution: 2,
    });

    expect(highRes.length).toBeGreaterThan(lowRes.length);
  });
});

describe('ImageFormationParser', () => {
  it('should parse image to coordinates', async () => {
    const parser = new ImageFormationParser();
    const coordinates = await parser.parse({
      src: 'data:image/png;base64,...', // Test image
      resolution: 5,
      threshold: 128,
    });

    expect(Array.isArray(coordinates)).toBe(true);
    expect(coordinates.length).toBeGreaterThan(0);
  });

  it('should sample colors when enabled', async () => {
    const parser = new ImageFormationParser();
    const coordinates = await parser.parse({
      src: 'data:image/png;base64,...',
      resolution: 5,
      colorSampling: true,
    });

    expect(coordinates[0]).toHaveProperty('color');
  });
});
```

**Step 2: Implement Parsers**
```typescript
// plugins/formations/src/parsers/TextFormationParser.ts
export class TextFormationParser {
  parse(options: ITextFormationOptions): IVector[] {
    // 1. Create offscreen canvas
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d')!;
    
    // 2. Render text
    ctx.font = `${options.fontSize}px ${options.font}`;
    ctx.fillText(options.text, 0, options.fontSize);
    
    // 3. Sample pixels at resolution intervals
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const coordinates: IVector[] = [];
    
    for (let y = 0; y < canvas.height; y += options.resolution) {
      for (let x = 0; x < canvas.width; x += options.resolution) {
        const i = (y * canvas.width + x) * 4;
        const alpha = imageData.data[i + 3];
        
        if (alpha > 128) { // Threshold
          coordinates.push({ x, y });
        }
      }
    }
    
    return coordinates;
  }
}
```

**Step 3: Run Tests**
```bash
npm test -- plugins/formations/parsers.test.ts
```

**Deliverables:**
- ✅ TextFormationParser
- ✅ ImageFormationParser
- ✅ All parser tests passing

---

### 3.2: Formation Manager (Week 3-4)

#### TDD Approach

**Step 1: Write Manager Tests**
```typescript
// tests/plugins/formations/FormationManager.test.ts
import { describe, it, expect } from 'vitest';
import { FormationManager } from '@/plugins/formations/FormationManager';

describe('FormationManager', () => {
  it('should load formation', async () => {
    const manager = new FormationManager();
    await manager.loadFormation('welcome', {
      type: 'text',
      options: {
        text: 'WELCOME',
        fontSize: 48,
      },
    });

    expect(manager.getCurrentFormation()).toBe('welcome');
  });

  it('should transition between formations', async () => {
    const manager = new FormationManager();
    await manager.loadFormation('first', { type: 'text', options: { text: 'FIRST' } });
    await manager.transitionFormation('second', { type: 'text', options: { text: 'SECOND' } });

    expect(manager.getCurrentFormation()).toBe('second');
  });

  it('should handle particle assignment', () => {
    const manager = new FormationManager();
    const particles = [/* mock particles */];
    const targets = [/* mock targets */];

    const assignments = manager.assignParticlesToTargets(particles, targets);
    expect(assignments.size).toBe(Math.min(particles.length, targets.length));
  });
});
```

**Step 2: Implement Manager**
```typescript
// plugins/formations/src/FormationManager.ts
export class FormationManager {
  private _formations: Map<string, IFormation>;
  private _currentFormation?: string;
  private _parser: TextFormationParser | ImageFormationParser;

  async loadFormation(id: string, config: IFormationConfig): Promise<void> {
    // Parse configuration to coordinates
    const coordinates = await this._parseFormation(config);
    
    this._formations.set(id, {
      id,
      coordinates,
      config,
    });
    
    this._currentFormation = id;
  }

  async transitionFormation(
    toId: string,
    config: IFormationConfig,
    options?: ITransitionOptions
  ): Promise<void> {
    // Implementation with easing and stagger
  }

  assignParticlesToTargets(
    particles: Particle[],
    targets: IVector[]
  ): Map<Particle, IVector> {
    // Use nearest-neighbor or Hungarian algorithm
  }
}
```

**Step 3: Run Tests**
```bash
npm test -- plugins/formations/FormationManager.test.ts
```

**Deliverables:**
- ✅ FormationManager
- ✅ Particle assignment algorithm
- ✅ All manager tests passing

---

### 3.3: Formation Updater (Week 4)

#### TDD Approach

**Step 1: Write Updater Tests**
```typescript
// tests/plugins/formations/FormationUpdater.test.ts
import { describe, it, expect } from 'vitest';
import { FormationUpdater } from '@/plugins/formations/FormationUpdater';

describe('FormationUpdater', () => {
  it('should move particles toward targets', () => {
    const updater = new FormationUpdater();
    const mockParticle = {
      position: { x: 0, y: 0 },
      velocity: { x: 0, y: 0 },
      formationTarget: { x: 100, y: 100 },
    } as any;

    updater.update(mockParticle, { value: 1 } as any);

    // Velocity should be toward target
    expect(mockParticle.velocity.x).toBeGreaterThan(0);
    expect(mockParticle.velocity.y).toBeGreaterThan(0);
  });

  it('should slow down near target', () => {
    const updater = new FormationUpdater();
    const mockParticle = {
      position: { x: 95, y: 95 },
      velocity: { x: 10, y: 10 },
      formationTarget: { x: 100, y: 100 },
    } as any;

    updater.update(mockParticle, { value: 1 } as any);

    // Velocity should decrease near target
    expect(Math.abs(mockParticle.velocity.x)).toBeLessThan(10);
  });
});
```

**Step 2: Implement Updater**
```typescript
// plugins/formations/src/FormationUpdater.ts
import type { IParticleUpdater, Particle, IDelta } from '@tsparticles/engine';

export class FormationUpdater implements IParticleUpdater {
  update(particle: Particle, delta: IDelta): void {
    if (!particle.formationTarget) return;

    const dx = particle.formationTarget.x - particle.position.x;
    const dy = particle.formationTarget.y - particle.position.y;
    const distance = Math.sqrt(dx * dx + dy * dy);

    if (distance < 1) return; // At target

    // Apply attraction with damping
    const attraction = 0.1;
    const damping = Math.max(0.1, Math.min(1, distance / 100));

    particle.velocity.x += (dx / distance) * attraction * damping;
    particle.velocity.y += (dy / distance) * attraction * damping;

    // Apply drag
    particle.velocity.x *= 0.95;
    particle.velocity.y *= 0.95;
  }
}
```

**Step 3: Run Tests**
```bash
npm test -- plugins/formations/FormationUpdater.test.ts
```

**Deliverables:**
- ✅ FormationUpdater implementing IParticleUpdater
- ✅ Target-seeking behavior
- ✅ All updater tests passing

---

### 3.4: Formation Plugin (Week 4-5)

#### TDD Approach

**Step 1: Write Plugin Tests**
```typescript
// tests/plugins/formations/FormationsPlugin.test.ts
import { describe, it, expect } from 'vitest';
import { FormationsPlugin } from '@/plugins/formations';

describe('FormationsPlugin', () => {
  it('should have correct plugin id', () => {
    const plugin = new FormationsPlugin();
    expect(plugin.id).toBe('formations');
  });

  it('should need plugin when formations are defined', () => {
    const plugin = new FormationsPlugin();
    const options = {
      formations: [{ id: 'test', type: 'text', options: { text: 'TEST' } }],
    };
    expect(plugin.needsPlugin(options)).toBe(true);
  });

  it('should create plugin instance', async () => {
    const plugin = new FormationsPlugin();
    const mockContainer = {} as any;
    const instance = await plugin.getPlugin(mockContainer);
    expect(instance).toBeDefined();
  });
});
```

**Step 2: Implement Plugin**
```typescript
// plugins/formations/src/FormationsPlugin.ts
import type { IPlugin, Container, IContainerPlugin } from '@tsparticles/engine';

export class FormationsPlugin implements IPlugin {
  readonly id = 'formations';

  async getPlugin(container: Container): Promise<IContainerPlugin> {
    const { FormationsPluginInstance } = await import('./FormationsPluginInstance.js');
    return new FormationsPluginInstance(container);
  }

  loadOptions(container: Container, options: any, source?: any): void {
    // Load formation options
  }

  needsPlugin(options?: any): boolean {
    return !!options?.formations?.length;
  }
}

export async function loadFormationsPlugin(engine: Engine): Promise<void> {
  await engine.addPlugin(new FormationsPlugin());
  await engine.addParticleUpdater('formation', () => new FormationUpdater());
}
```

**Step 3: Run Tests**
```bash
npm test -- plugins/formations/FormationsPlugin.test.ts
```

**Deliverables:**
- ✅ FormationsPlugin
- ✅ Plugin registration
- ✅ All plugin tests passing

---

### 3.5: Phase 3 Integration Testing (Week 5)

**Integration Test Suite**
```typescript
// tests/integration/phase3.test.ts
import { describe, it, expect } from 'vitest';
import { tsParticles } from '@tsparticles/engine';
import { loadFormationsPlugin } from '@/plugins/formations';

describe('Phase 3 Integration', () => {
  it('should load formations plugin', async () => {
    await loadFormationsPlugin(tsParticles);
    // Plugin should be registered
  });

  it('should create text formation', async () => {
    await loadFormationsPlugin(tsParticles);
    
    const container = await tsParticles.load({
      options: {
        formations: [{
          id: 'test',
          type: 'text',
          options: {
            text: 'TEST',
            fontSize: 48,
          },
        }],
      },
    });

    expect(container).toBeDefined();
  });

  it('should transition between formations', async () => {
    // Test formation transitions
  });
});
```

**Run Full Phase 3 Tests**
```bash
npm test
```

**Phase 3 Deliverables Summary:**
- ✅ Text formation parser
- ✅ Image formation parser
- ✅ Formation manager
- ✅ Particle assignment algorithm
- ✅ Formation updater
- ✅ Formations plugin
- ✅ Plugin registration
- ✅ 100% test coverage
- ✅ All tests passing

---

## Phase 4: Polish, Documentation & Examples

**Duration:** 1 week  
**Goal:** Create examples, optimize performance, complete documentation

### 4.1: Example Application (Day 21-22)

**Create Next.js Example App**
```typescript
// examples/nextjs-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── about/page.tsx
│   ├── projects/page.tsx
│   └── under-construction/page.tsx
├── components/
│   ├── ParticlesInitializer.tsx
│   ├── ThemeSwitcher.tsx
│   └── ModeSelector.tsx
├── contexts/
│   └── ParticlesContext.tsx
└── config/
    ├── design-tokens.ts
    ├── route-particles.ts
    └── particle-presets.ts
```

**Example Tests**
```typescript
// examples/nextjs-app/__tests__/integration.test.tsx
import { describe, it, expect } from 'vitest';
import { render } from '@testing-library/react';
import HomePage from '../app/page';

describe('Example App', () => {
  it('should render home page with particles', () => {
    const { container } = render(<HomePage />);
    expect(container.querySelector('canvas')).toBeDefined();
  });
});
```

**Deliverables:**
- ✅ Complete Next.js example application
- ✅ All 5 page examples working
- ✅ Theme switcher functional
- ✅ Mode selector functional
- ✅ Example tests passing

---

### 4.2: Performance Optimization (Day 23)

**Performance Tests**
```typescript
// tests/performance/optimization.test.ts
import { describe, it, expect } from 'vitest';
import { convertConfigToOptions } from '@/lib/config-converter';
import { designTokens } from '@/config/design-tokens';

describe('Performance', () => {
  it('should convert config in under 10ms', () => {
    const config = { particleCount: 100 };
    const start = performance.now();
    
    for (let i = 0; i < 100; i++) {
      convertConfigToOptions(config, designTokens, 'light');
    }
    
    const end = performance.now();
    const avgTime = (end - start) / 100;
    expect(avgTime).toBeLessThan(10);
  });

  it('should handle large particle counts efficiently', () => {
    const config = { particleCount: 1000 };
    const start = performance.now();
    convertConfigToOptions(config, designTokens, 'light');
    const end = performance.now();
    
    expect(end - start).toBeLessThan(50);
  });
});
```

**Optimization Tasks:**
- ✅ Add memoization where needed
- ✅ Optimize particle assignment algorithm
- ✅ Add particle pooling
- ✅ Implement spatial partitioning
- ✅ Performance tests passing

---

### 4.3: Documentation Completion (Day 24-25)

**Documentation Checklist:**
- ✅ API documentation (JSDoc)
- ✅ Usage guides (updated)
- ✅ Migration guide
- ✅ Troubleshooting section
- ✅ Performance tips
- ✅ Accessibility guidelines
- ✅ Contributing guide

**Documentation Tests**
```bash
# Generate documentation
npm run docs

# Verify all exports are documented
npm run docs:verify
```

---

### 4.4: Final Testing & Validation (Day 25-26)

**Full Test Suite**
```bash
# Run all tests
npm test

# Run with coverage
npm test -- --coverage

# Check coverage thresholds
npm run test:coverage
```

**Coverage Requirements:**
- ✅ Line coverage: > 90%
- ✅ Branch coverage: > 85%
- ✅ Function coverage: > 90%
- ✅ Statement coverage: > 90%

**Manual Testing Checklist:**
- ✅ Test on Chrome, Firefox, Safari
- ✅ Test on mobile devices
- ✅ Test with screen readers
- ✅ Test reduced motion preference
- ✅ Test all route transitions
- ✅ Test theme switching
- ✅ Test mode switching
- ✅ Test text formations (Phase 3)
- ✅ Test error scenarios

---

## Continuous Integration

### CI Pipeline Configuration

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage
      - run: npm run build
      
  integration:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:integration
      
  e2e:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:e2e
```

---

## Testing Infrastructure

### Test Setup

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
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
});
```

```typescript
// tests/setup.ts
import { expect, afterEach } from 'vitest';
import { cleanup } from '@testing-library/react';
import '@testing-library/jest-dom';

afterEach(() => {
  cleanup();
});
```

---

## Timeline Summary

| Phase | Duration | Key Deliverables | TDD Approach |
|-------|----------|------------------|--------------|
| **Phase 1** | Week 1 (Days 1-5) | Type system, converters, route matching | Write tests → Implement → Refactor |
| **Phase 2** | Week 2 (Days 6-10) | Context, plugin, transitions, effects | Write tests → Implement → Refactor |
| **Phase 3** | Weeks 3-5 (Days 11-20) | Formation parsers, manager, plugin | Write tests → Implement → Refactor |
| **Phase 4** | Week 6 (Days 21-26) | Examples, optimization, docs | Integration tests → Polish → Validate |

---

## Success Criteria

### Phase 1 Success Criteria
- ✅ All type definitions complete
- ✅ Configuration converter working
- ✅ Route matcher functional
- ✅ 100% test coverage
- ✅ Zero TypeScript errors
- ✅ Zero ESLint warnings

### Phase 2 Success Criteria
- ✅ React Context working
- ✅ Plugin architecture implemented
- ✅ Transitions functional
- ✅ Effects library complete
- ✅ 100% test coverage
- ✅ No memory leaks

### Phase 3 Success Criteria
- ✅ Text formation working
- ✅ Image formation working
- ✅ Morphing smooth
- ✅ Performance acceptable (< 16ms/frame)
- ✅ 100% test coverage

### Phase 4 Success Criteria
- ✅ Example app working
- ✅ Documentation complete
- ✅ Performance optimized
- ✅ All tests passing
- ✅ CI pipeline green

---

## Risk Mitigation

### Identified Risks

1. **Performance Issues with Large Formations**
   - Mitigation: Implement particle pooling early
   - Fallback: Reduce particle count dynamically

2. **Browser Compatibility**
   - Mitigation: Test on all major browsers
   - Fallback: Provide graceful degradation

3. **Memory Leaks in Long-Running Apps**
   - Mitigation: Comprehensive cleanup in useEffect
   - Fallback: Add memory monitoring

4. **Complex Particle Assignment Algorithm**
   - Mitigation: Start with nearest-neighbor, optimize later
   - Fallback: Use simpler random assignment

---

## Next Steps After Completion

1. **Community Feedback**
   - Release beta version
   - Gather user feedback
   - Iterate on improvements

2. **Performance Tuning**
   - Profile real-world usage
   - Optimize hot paths
   - Add WebWorker support

3. **Feature Enhancements**
   - SVG path formations
   - 3D formations
   - Custom easing functions
   - Formation templates

4. **Integration Examples**
   - Gatsby example
   - Remix example
   - SvelteKit example

---

## References

- [ARCHITECTURE_REVIEW.md](./ARCHITECTURE_REVIEW.md) - Production architecture
- [NEXTJS_CONFIGURATION_FRAMEWORK.md](./NEXTJS_CONFIGURATION_FRAMEWORK.md) - Configuration system
- [PARTICLE_GRID_IDEATION.md](./PARTICLE_GRID_IDEATION.md) - Formation plugin design
- [IMPLEMENTATION_EXAMPLE.md](./IMPLEMENTATION_EXAMPLE.md) - Usage examples
- [FEATURE_SUPPORT_MATRIX.md](./FEATURE_SUPPORT_MATRIX.md) - Feature breakdown

---

## Conclusion

This multi-phase implementation plan provides a complete roadmap for adding the particle configuration framework to tsParticles. By following TDD principles, maintaining high test coverage, and implementing in logical phases, we ensure a high-quality, maintainable solution that meets all requirements.

Each phase builds on the previous, allowing for early validation and course correction if needed. The plan prioritizes foundation over features, ensuring a solid base before adding complex functionality.
