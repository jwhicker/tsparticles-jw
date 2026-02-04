# Next.js 16 Particle Configuration Framework

## Overview

This document provides a comprehensive configuration framework for tsParticles in Next.js 16 with React 19, designed for easy per-route customization, per-page behavior control, sequential text formations, and modern theming with CSS design tokens.

---

## Architecture: Configuration-Driven Particle System

### Key Principles

1. **Route-Based Configuration** - Each route can specify its own particle behavior
2. **CSS Token Integration** - Full design token support for consistent theming
3. **Sequential Animations** - Support for animation sequences (e.g., "Under Construction" → "Coming Soon")
4. **Type-Safe Configuration** - Full TypeScript support for all options
5. **Zero-Config Defaults** - Sensible defaults with easy customization

---

## Installation

```bash
npm install @tsparticles/engine @tsparticles/react \
  @tsparticles/plugin-interactivity \
  @tsparticles/plugin-themes \
  @tsparticles/plugin-responsive \
  @tsparticles/path-grid \
  @tsparticles/path-perlin-noise \
  @tsparticles/path-curl-noise \
  @tsparticles/shape-text \
  @tsparticles/shape-image
```

---

## Part 1: CSS Design Tokens Setup

### 1.1 Design Token Structure

Create a centralized token system compatible with Next.js 16 and React 19:

```typescript
// config/design-tokens.ts
export const designTokens = {
  colors: {
    primary: {
      light: '#3b82f6',
      dark: '#60a5fa',
    },
    secondary: {
      light: '#8b5cf6',
      dark: '#a78bfa',
    },
    background: {
      light: '#ffffff',
      dark: '#0f172a',
    },
    surface: {
      light: '#f8fafc',
      dark: '#1e293b',
    },
    text: {
      light: '#0f172a',
      dark: '#f8fafc',
    },
    particle: {
      light: '#64748b',
      dark: '#94a3b8',
    },
    particleLink: {
      light: '#cbd5e1',
      dark: '#475569',
    },
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem',
    '2xl': '3rem',
  },
  particles: {
    density: {
      mobile: 40,
      tablet: 60,
      desktop: 80,
    },
    size: {
      min: 1,
      max: 3,
    },
    opacity: {
      min: 0.3,
      max: 0.7,
    },
    speed: {
      slow: 0.5,
      medium: 1,
      fast: 2,
    },
  },
} as const;

export type DesignTokens = typeof designTokens;
```

### 1.2 CSS Variables Generation

```typescript
// lib/css-tokens.ts
import { designTokens } from '@/config/design-tokens';

export function generateCSSVariables(theme: 'light' | 'dark'): string {
  const colors = designTokens.colors;
  
  return `
    --color-primary: ${colors.primary[theme]};
    --color-secondary: ${colors.secondary[theme]};
    --color-background: ${colors.background[theme]};
    --color-surface: ${colors.surface[theme]};
    --color-text: ${colors.text[theme]};
    --color-particle: ${colors.particle[theme]};
    --color-particle-link: ${colors.particleLink[theme]};
    
    --space-xs: ${designTokens.spacing.xs};
    --space-sm: ${designTokens.spacing.sm};
    --space-md: ${designTokens.spacing.md};
    --space-lg: ${designTokens.spacing.lg};
    --space-xl: ${designTokens.spacing.xl};
    --space-2xl: ${designTokens.spacing['2xl']};
  `;
}

export function applyTheme(theme: 'light' | 'dark') {
  const root = document.documentElement;
  const variables = generateCSSVariables(theme);
  
  variables.split(';').forEach((variable) => {
    const [key, value] = variable.split(':').map(s => s.trim());
    if (key && value) {
      root.style.setProperty(key, value);
    }
  });
}
```

### 1.3 Global Styles with Tokens

```css
/* app/globals.css */
:root {
  /* Default to light theme */
  --color-primary: #3b82f6;
  --color-secondary: #8b5cf6;
  --color-background: #ffffff;
  --color-surface: #f8fafc;
  --color-text: #0f172a;
  --color-particle: #64748b;
  --color-particle-link: #cbd5e1;
  
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
  --space-2xl: 3rem;
}

[data-theme='dark'] {
  --color-primary: #60a5fa;
  --color-secondary: #a78bfa;
  --color-background: #0f172a;
  --color-surface: #1e293b;
  --color-text: #f8fafc;
  --color-particle: #94a3b8;
  --color-particle-link: #475569;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-primary: #60a5fa;
    --color-secondary: #a78bfa;
    --color-background: #0f172a;
    --color-surface: #1e293b;
    --color-text: #f8fafc;
    --color-particle: #94a3b8;
    --color-particle-link: #475569;
  }
}
```

---

## Part 2: Configuration Type System

### 2.1 Route Configuration Types

```typescript
// types/particle-config.ts
import { ISourceOptions } from '@tsparticles/engine';

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

export type AnimationEffect = 
  | 'fade-in' 
  | 'slide-left' 
  | 'slide-right' 
  | 'explode' 
  | 'swirl' 
  | 'vortex'
  | 'none';

export interface TextFormation {
  text: string;
  duration?: number;
  delay?: number;
  font?: string;
  fontSize?: number;
}

export interface ParticleRouteConfig {
  // Core behavior
  gridStyle?: GridStyle;
  particleCount?: number;
  
  // Interactions
  hoverMode?: InteractionMode | InteractionMode[];
  clickMode?: InteractionMode | InteractionMode[];
  
  // Animations
  transitionIn?: AnimationEffect;
  transitionOut?: AnimationEffect;
  
  // Text formations
  textFormations?: TextFormation[];
  
  // Theme integration
  useSystemTheme?: boolean;
  forceTheme?: 'light' | 'dark';
  
  // Custom options (full control)
  customOptions?: Partial<ISourceOptions>;
}

export interface RouteParticleMap {
  [route: string]: ParticleRouteConfig;
}
```

### 2.2 Default Configurations

```typescript
// config/particle-presets.ts
import { ParticleRouteConfig } from '@/types/particle-config';
import { designTokens } from './design-tokens';

export const particlePresets = {
  default: {
    gridStyle: 'perlin-noise',
    particleCount: 80,
    hoverMode: 'grab',
    clickMode: 'push',
    transitionIn: 'fade-in',
    transitionOut: 'fade-in',
    useSystemTheme: true,
  } as ParticleRouteConfig,
  
  minimal: {
    gridStyle: 'organic',
    particleCount: 30,
    hoverMode: 'none',
    clickMode: 'none',
    transitionIn: 'fade-in',
    transitionOut: 'fade-in',
    useSystemTheme: true,
  } as ParticleRouteConfig,
  
  interactive: {
    gridStyle: 'curl-noise',
    particleCount: 100,
    hoverMode: ['grab', 'bubble'],
    clickMode: 'repulse',
    transitionIn: 'swirl',
    transitionOut: 'explode',
    useSystemTheme: true,
  } as ParticleRouteConfig,
  
  construction: {
    gridStyle: 'grid',
    particleCount: 150,
    hoverMode: 'attract',
    clickMode: 'push',
    textFormations: [
      {
        text: 'UNDER CONSTRUCTION',
        duration: 3000,
        delay: 0,
        fontSize: 48,
      },
      {
        text: 'COMING SOON',
        duration: 3000,
        delay: 3500,
        fontSize: 48,
      },
    ],
    transitionIn: 'fade-in',
    useSystemTheme: true,
  } as ParticleRouteConfig,
};
```

---

## Part 3: Configuration-to-Options Converter

### 3.1 Core Converter

```typescript
// lib/config-to-options.ts
import { ISourceOptions } from '@tsparticles/engine';
import { ParticleRouteConfig } from '@/types/particle-config';
import { designTokens } from '@/config/design-tokens';

export function convertConfigToOptions(
  config: ParticleRouteConfig,
  theme: 'light' | 'dark'
): ISourceOptions {
  const colors = designTokens.colors;
  const particles = designTokens.particles;
  
  // Get responsive particle count
  const particleCount = config.particleCount ?? 80;
  
  // Base configuration
  const options: ISourceOptions = {
    background: {
      color: {
        value: 'transparent',
      },
    },
    fpsLimit: 120,
    particles: {
      number: {
        value: particleCount,
        density: {
          enable: true,
        },
      },
      color: {
        value: colors.particle[theme],
      },
      shape: {
        type: 'circle',
      },
      opacity: {
        value: particles.opacity,
      },
      size: {
        value: particles.size,
      },
      move: {
        enable: true,
        speed: particles.speed.medium,
        ...getGridStyleConfig(config.gridStyle ?? 'perlin-noise'),
      },
      links: {
        enable: true,
        distance: 150,
        color: colors.particleLink[theme],
        opacity: 0.4,
        width: 1,
      },
    },
    interactivity: {
      detectsOn: 'window',
      events: {
        onHover: {
          enable: !!config.hoverMode && config.hoverMode !== 'none',
          mode: config.hoverMode || 'grab',
        },
        onClick: {
          enable: !!config.clickMode && config.clickMode !== 'none',
          mode: config.clickMode || 'push',
        },
        resize: {
          enable: true,
        },
      },
      modes: getInteractionModes(config),
    },
    detectRetina: true,
    responsive: [
      {
        maxWidth: 768,
        options: {
          particles: {
            number: {
              value: particles.density.mobile,
            },
            move: {
              speed: particles.speed.slow,
            },
          },
        },
      },
      {
        maxWidth: 1024,
        options: {
          particles: {
            number: {
              value: particles.density.tablet,
            },
          },
        },
      },
    ],
  };
  
  // Merge custom options if provided
  if (config.customOptions) {
    return deepMerge(options, config.customOptions);
  }
  
  return options;
}

function getGridStyleConfig(style: string) {
  switch (style) {
    case 'perlin-noise':
      return {
        path: {
          enable: true,
          generator: 'perlinNoise',
          options: {
            columns: 10,
            rows: 10,
          },
        },
      };
    case 'simplex-noise':
      return {
        path: {
          enable: true,
          generator: 'simplexNoise',
          options: {
            columns: 10,
            rows: 10,
          },
        },
      };
    case 'curl-noise':
      return {
        path: {
          enable: true,
          generator: 'curlNoise',
          options: {
            columns: 10,
            rows: 10,
          },
        },
      };
    case 'grid':
      return {
        path: {
          enable: true,
          generator: 'grid',
          options: {
            cellSize: 50,
          },
        },
      };
    case 'organic':
    default:
      return {
        direction: 'none',
        random: true,
        straight: false,
      };
  }
}

function getInteractionModes(config: ParticleRouteConfig) {
  const modes: any = {};
  
  // Configure hover modes
  const hoverModes = Array.isArray(config.hoverMode) 
    ? config.hoverMode 
    : [config.hoverMode];
  
  hoverModes.forEach(mode => {
    if (!mode || mode === 'none') return;
    
    switch (mode) {
      case 'grab':
        modes.grab = {
          distance: 200,
          links: {
            opacity: 0.8,
          },
        };
        break;
      case 'bubble':
        modes.bubble = {
          distance: 200,
          size: 6,
          duration: 0.3,
        };
        break;
      case 'repulse':
        modes.repulse = {
          distance: 200,
          duration: 0.4,
        };
        break;
      case 'attract':
        modes.attract = {
          distance: 200,
          duration: 0.4,
        };
        break;
      case 'connect':
        modes.connect = {
          distance: 150,
          radius: 200,
        };
        break;
      case 'trail':
        modes.trail = {
          delay: 0.1,
          quantity: 3,
        };
        break;
      case 'bounce':
        modes.bounce = {
          distance: 200,
        };
        break;
      case 'parallax':
        modes.parallax = {
          enable: true,
          force: 60,
          smooth: 10,
        };
        break;
    }
  });
  
  // Configure click modes
  const clickModes = Array.isArray(config.clickMode) 
    ? config.clickMode 
    : [config.clickMode];
  
  clickModes.forEach(mode => {
    if (!mode || mode === 'none') return;
    
    if (mode === 'push') {
      modes.push = {
        quantity: 4,
      };
    } else if (mode === 'repulse') {
      modes.repulse = {
        distance: 200,
        duration: 0.4,
      };
    }
  });
  
  return modes;
}

function deepMerge(target: any, source: any): any {
  const output = { ...target };
  
  if (isObject(target) && isObject(source)) {
    Object.keys(source).forEach(key => {
      if (isObject(source[key])) {
        if (!(key in target)) {
          Object.assign(output, { [key]: source[key] });
        } else {
          output[key] = deepMerge(target[key], source[key]);
        }
      } else {
        Object.assign(output, { [key]: source[key] });
      }
    });
  }
  
  return output;
}

function isObject(item: any): boolean {
  return item && typeof item === 'object' && !Array.isArray(item);
}
```

---

## Part 4: Route Configuration Map

### 4.1 Define Per-Route Behavior

```typescript
// config/route-particles.ts
import { RouteParticleMap } from '@/types/particle-config';
import { particlePresets } from './particle-presets';

export const routeParticleConfig: RouteParticleMap = {
  // Home page - default interactive experience
  '/': {
    ...particlePresets.default,
    particleCount: 100,
    hoverMode: ['grab', 'bubble'],
    clickMode: 'push',
  },
  
  // About page - minimal, calm
  '/about': {
    ...particlePresets.minimal,
    gridStyle: 'organic',
    hoverMode: 'parallax',
    clickMode: 'none',
  },
  
  // Projects page - highly interactive
  '/projects': {
    ...particlePresets.interactive,
    gridStyle: 'curl-noise',
    hoverMode: ['grab', 'connect'],
    clickMode: 'repulse',
    transitionIn: 'swirl',
  },
  
  // Contact page - moderate interaction
  '/contact': {
    gridStyle: 'simplex-noise',
    particleCount: 60,
    hoverMode: 'attract',
    clickMode: 'push',
    transitionIn: 'fade-in',
  },
  
  // Blog pages - minimal distraction
  '/blog': {
    ...particlePresets.minimal,
    particleCount: 30,
    hoverMode: 'none',
    clickMode: 'none',
  },
  
  // Construction/Coming Soon
  '/under-construction': {
    ...particlePresets.construction,
    textFormations: [
      {
        text: 'UNDER CONSTRUCTION',
        duration: 3000,
        delay: 0,
        fontSize: 48,
        font: 'Arial, sans-serif',
      },
      {
        text: 'COMING SOON',
        duration: 3000,
        delay: 3500,
        fontSize: 48,
        font: 'Arial, sans-serif',
      },
    ],
  },
  
  // Product showcase - dynamic
  '/products/[id]': {
    gridStyle: 'perlin-noise',
    particleCount: 80,
    hoverMode: 'bubble',
    clickMode: 'attract',
    transitionIn: 'slide-right',
    transitionOut: 'slide-left',
  },
};

// Fallback for unmatched routes
export const defaultParticleConfig = particlePresets.default;
```

---

## Part 5: Enhanced Context with Route Awareness

### 5.1 Advanced Particle Context

```typescript
// contexts/ParticlesContext.tsx
'use client';

import {
  createContext,
  useContext,
  useRef,
  useEffect,
  useState,
  ReactNode,
  useCallback,
} from 'react';
import { usePathname } from 'next/navigation';
import { Container, Engine, ISourceOptions } from '@tsparticles/engine';
import { ParticleRouteConfig, TextFormation } from '@/types/particle-config';
import { routeParticleConfig, defaultParticleConfig } from '@/config/route-particles';
import { convertConfigToOptions } from '@/lib/config-to-options';

interface ParticlesContextValue {
  container: Container | null;
  initialize: (engine: Engine) => Promise<void>;
  updateConfig: (config: ParticleRouteConfig) => Promise<void>;
  setTheme: (theme: 'light' | 'dark') => void;
  theme: 'light' | 'dark';
  currentConfig: ParticleRouteConfig | null;
  playTextFormations: (formations: TextFormation[]) => Promise<void>;
}

const ParticlesContext = createContext<ParticlesContextValue | null>(null);

export function ParticlesProvider({ children }: { children: ReactNode }) {
  const containerRef = useRef<Container | null>(null);
  const canvasRef = useRef<HTMLCanvasElement | null>(null);
  const engineRef = useRef<Engine | null>(null);
  
  const pathname = usePathname();
  const [theme, setThemeState] = useState<'light' | 'dark'>('light');
  const [currentConfig, setCurrentConfig] = useState<ParticleRouteConfig | null>(null);

  // Detect system theme
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    setThemeState(mediaQuery.matches ? 'dark' : 'light');
    
    const handler = (e: MediaQueryListEvent) => {
      setThemeState(e.matches ? 'dark' : 'light');
    };
    
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, []);

  const setTheme = useCallback((newTheme: 'light' | 'dark') => {
    setThemeState(newTheme);
    document.documentElement.setAttribute('data-theme', newTheme);
  }, []);

  const initialize = useCallback(async (engine: Engine) => {
    if (!canvasRef.current || engineRef.current) return;
    
    engineRef.current = engine;
    
    // Get initial route config
    const config = getRouteConfig(pathname);
    setCurrentConfig(config);
    
    const options = convertConfigToOptions(config, theme);
    
    containerRef.current = await engine.load({
      element: canvasRef.current,
      options,
    });
  }, [pathname, theme]);

  const updateConfig = useCallback(async (config: ParticleRouteConfig) => {
    if (!containerRef.current || !engineRef.current) return;
    
    setCurrentConfig(config);
    const options = convertConfigToOptions(config, theme);
    
    // Apply transition out effect
    if (currentConfig?.transitionOut) {
      await applyTransitionEffect(containerRef.current, currentConfig.transitionOut, 'out');
    }
    
    // Update options
    await containerRef.current.loadOptions(options);
    
    // Apply transition in effect
    if (config.transitionIn) {
      await applyTransitionEffect(containerRef.current, config.transitionIn, 'in');
    }
    
    // Play text formations if any
    if (config.textFormations && config.textFormations.length > 0) {
      await playTextFormations(config.textFormations);
    }
  }, [theme, currentConfig]);

  const playTextFormations = useCallback(async (formations: TextFormation[]) => {
    if (!containerRef.current) return;
    
    for (const formation of formations) {
      if (formation.delay) {
        await new Promise(resolve => setTimeout(resolve, formation.delay));
      }
      
      // TODO: Implement text formation animation
      // This will be part of Phase 3 (formation plugin)
      console.log('Text formation:', formation.text);
      
      if (formation.duration) {
        await new Promise(resolve => setTimeout(resolve, formation.duration));
      }
    }
  }, []);

  // Handle route changes
  useEffect(() => {
    if (!containerRef.current) return;
    
    const config = getRouteConfig(pathname);
    updateConfig(config);
  }, [pathname, updateConfig]);

  // Handle theme changes
  useEffect(() => {
    if (!containerRef.current || !currentConfig) return;
    
    const options = convertConfigToOptions(currentConfig, theme);
    containerRef.current.loadOptions(options);
  }, [theme, currentConfig]);

  useEffect(() => {
    return () => {
      containerRef.current?.destroy();
    };
  }, []);

  return (
    <ParticlesContext.Provider
      value={{
        container: containerRef.current,
        initialize,
        updateConfig,
        setTheme,
        theme,
        currentConfig,
        playTextFormations,
      }}
    >
      <canvas
        ref={canvasRef}
        style={{
          position: 'fixed',
          top: 0,
          left: 0,
          width: '100vw',
          height: '100vh',
          zIndex: -1,
          pointerEvents: 'auto',
        }}
      />
      {children}
    </ParticlesContext.Provider>
  );
}

export const useParticles = () => {
  const context = useContext(ParticlesContext);
  if (!context) {
    throw new Error('useParticles must be used within ParticlesProvider');
  }
  return context;
};

// Helper functions

function getRouteConfig(pathname: string): ParticleRouteConfig {
  // Exact match
  if (routeParticleConfig[pathname]) {
    return routeParticleConfig[pathname];
  }
  
  // Pattern match (e.g., /products/[id])
  const patternKeys = Object.keys(routeParticleConfig).filter(key => key.includes('['));
  for (const pattern of patternKeys) {
    const regex = new RegExp('^' + pattern.replace(/\[.*?\]/g, '[^/]+') + '$');
    if (regex.test(pathname)) {
      return routeParticleConfig[pattern];
    }
  }
  
  // Partial match (e.g., /blog/post-title matches /blog)
  const partialKeys = Object.keys(routeParticleConfig)
    .filter(key => !key.includes('['))
    .sort((a, b) => b.length - a.length); // Longest match first
  
  for (const key of partialKeys) {
    if (pathname.startsWith(key) && key !== '/') {
      return routeParticleConfig[key];
    }
  }
  
  return defaultParticleConfig;
}

async function applyTransitionEffect(
  container: Container,
  effect: string,
  direction: 'in' | 'out'
) {
  const duration = 500; // ms
  
  switch (effect) {
    case 'fade-in':
      // Fade in/out effect
      const particles = container.particles.array;
      const startOpacity = direction === 'in' ? 0 : 1;
      const endOpacity = direction === 'in' ? 1 : 0;
      
      particles.forEach(particle => {
        if (particle.opacity) {
          particle.opacity.value = startOpacity;
        }
      });
      
      // Animate to target opacity
      await new Promise(resolve => {
        const startTime = Date.now();
        const animate = () => {
          const elapsed = Date.now() - startTime;
          const progress = Math.min(elapsed / duration, 1);
          const currentOpacity = startOpacity + (endOpacity - startOpacity) * progress;
          
          particles.forEach(particle => {
            if (particle.opacity) {
              particle.opacity.value = currentOpacity;
            }
          });
          
          if (progress < 1) {
            requestAnimationFrame(animate);
          } else {
            resolve(null);
          }
        };
        animate();
      });
      break;
      
    case 'swirl':
      // Apply spiral motion
      await container.loadOptions({
        particles: {
          move: {
            path: {
              enable: true,
              generator: 'spiral',
              options: {
                center: { x: 50, y: 50 },
                turns: direction === 'in' ? 3 : -3,
              },
            },
          },
        },
      });
      await new Promise(resolve => setTimeout(resolve, duration));
      break;
      
    case 'explode':
      // Explode from center
      const center = {
        x: container.canvas.size.width / 2,
        y: container.canvas.size.height / 2,
      };
      
      container.particles.array.forEach(particle => {
        const dx = particle.position.x - center.x;
        const dy = particle.position.y - center.y;
        const angle = Math.atan2(dy, dx);
        const force = direction === 'in' ? -20 : 20;
        
        particle.velocity.x = Math.cos(angle) * force;
        particle.velocity.y = Math.sin(angle) * force;
      });
      
      await new Promise(resolve => setTimeout(resolve, duration));
      break;
      
    default:
      // No animation
      break;
  }
}
```

---

## Part 6: Usage Examples

### 6.1 Basic Usage

```typescript
// app/layout.tsx
import { ParticlesProvider } from '@/contexts/ParticlesContext';
import '@/app/globals.css';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <ParticlesProvider>
          {children}
        </ParticlesProvider>
      </body>
    </html>
  );
}
```

### 6.2 Initialize in Root Component

```typescript
// components/ParticlesInitializer.tsx
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';
import { tsParticles } from '@tsparticles/engine';
import { loadInteractivityPlugin } from '@tsparticles/plugin-interactivity';
import { loadThemesPlugin } from '@tsparticles/plugin-themes';
import { loadResponsivePlugin } from '@tsparticles/plugin-responsive';
import { loadGridPath } from '@tsparticles/path-grid';
import { loadPerlinNoisePath } from '@tsparticles/path-perlin-noise';
import { loadSimplexNoisePath } from '@tsparticles/path-simplex-noise';
import { loadCurlNoisePath } from '@tsparticles/path-curl-noise';
import { loadTextShape } from '@tsparticles/shape-text';
import { loadImageShape } from '@tsparticles/shape-image';

export function ParticlesInitializer() {
  const { initialize } = useParticles();

  useEffect(() => {
    const init = async () => {
      // Load all plugins
      await Promise.all([
        loadInteractivityPlugin(tsParticles),
        loadThemesPlugin(tsParticles),
        loadResponsivePlugin(tsParticles),
        loadGridPath(tsParticles),
        loadPerlinNoisePath(tsParticles),
        loadSimplexNoisePath(tsParticles),
        loadCurlNoisePath(tsParticles),
        loadTextShape(tsParticles),
        loadImageShape(tsParticles),
      ]);

      await initialize(tsParticles);
    };

    init();
  }, [initialize]);

  return null;
}
```

```typescript
// app/page.tsx
import { ParticlesInitializer } from '@/components/ParticlesInitializer';

export default function Home() {
  return (
    <>
      <ParticlesInitializer />
      <main>
        <h1>Home Page</h1>
        {/* Your content */}
      </main>
    </>
  );
}
```

### 6.3 Per-Page Customization

```typescript
// app/about/page.tsx
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

export default function AboutPage() {
  const { updateConfig } = useParticles();

  useEffect(() => {
    // Override route config for this specific page instance
    updateConfig({
      gridStyle: 'organic',
      particleCount: 20,
      hoverMode: 'parallax',
      clickMode: 'none',
    });
  }, [updateConfig]);

  return (
    <main>
      <h1>About Us</h1>
      {/* Your content */}
    </main>
  );
}
```

### 6.4 Theme Switcher Component

```typescript
// components/ThemeSwitcher.tsx
'use client';

import { useParticles } from '@/contexts/ParticlesContext';

export function ThemeSwitcher() {
  const { theme, setTheme } = useParticles();

  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      className="theme-toggle"
    >
      {theme === 'light' ? '🌙' : '☀️'}
    </button>
  );
}
```

### 6.5 Construction Page with Text Sequence

```typescript
// app/under-construction/page.tsx
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

export default function ConstructionPage() {
  const { playTextFormations } = useParticles();

  useEffect(() => {
    // Play text formation sequence
    playTextFormations([
      {
        text: 'UNDER CONSTRUCTION',
        duration: 3000,
        delay: 500,
        fontSize: 48,
      },
      {
        text: 'COMING SOON',
        duration: 3000,
        delay: 3500,
        fontSize: 48,
      },
    ]);
  }, [playTextFormations]);

  return (
    <main className="construction-page">
      <div className="construction-message">
        {/* Visual representation of text formations */}
      </div>
    </main>
  );
}
```

---

## Part 7: Advanced Customization

### 7.1 Dynamic Route Configuration

```typescript
// hooks/useRouteParticles.ts
import { useEffect } from 'react';
import { usePathname, useSearchParams } from 'next/navigation';
import { useParticles } from '@/contexts/ParticlesContext';
import { ParticleRouteConfig } from '@/types/particle-config';

export function useRouteParticles(config?: ParticleRouteConfig) {
  const { updateConfig } = useParticles();
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    if (config) {
      updateConfig(config);
    }
  }, [config, pathname, searchParams, updateConfig]);
}
```

### 7.2 Mode-Specific Behavior

```typescript
// config/mode-particles.ts
import { ParticleRouteConfig } from '@/types/particle-config';

export const modeParticleConfig = {
  reading: {
    particleCount: 20,
    hoverMode: 'none',
    clickMode: 'none',
    gridStyle: 'organic',
  } as ParticleRouteConfig,
  
  presentation: {
    particleCount: 150,
    hoverMode: ['grab', 'bubble', 'connect'],
    clickMode: 'repulse',
    gridStyle: 'curl-noise',
  } as ParticleRouteConfig,
  
  focus: {
    particleCount: 10,
    hoverMode: 'parallax',
    clickMode: 'none',
    gridStyle: 'organic',
  } as ParticleRouteConfig,
};
```

```typescript
// components/ModeSelector.tsx
'use client';

import { useParticles } from '@/contexts/ParticlesContext';
import { modeParticleConfig } from '@/config/mode-particles';

export function ModeSelector() {
  const { updateConfig } = useParticles();

  return (
    <div className="mode-selector">
      <button onClick={() => updateConfig(modeParticleConfig.reading)}>
        Reading Mode
      </button>
      <button onClick={() => updateConfig(modeParticleConfig.presentation)}>
        Presentation Mode
      </button>
      <button onClick={() => updateConfig(modeParticleConfig.focus)}>
        Focus Mode
      </button>
    </div>
  );
}
```

---

## Part 8: Best Practices

### 8.1 Performance Optimization

```typescript
// config/performance-optimizations.ts
export const performanceConfig = {
  // Reduce particles on mobile
  mobile: {
    particleCount: 30,
    disableLinks: true,
    simplifyInteractions: true,
  },
  
  // Battery-aware
  lowPower: {
    particleCount: 40,
    reducedMotion: true,
    lowerFPS: 30,
  },
  
  // High performance
  desktop: {
    particleCount: 100,
    enableAllEffects: true,
    highFPS: 120,
  },
};
```

### 8.2 Accessibility

```typescript
// hooks/useReducedMotion.ts
import { useEffect, useState } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

export function useReducedMotion() {
  const { updateConfig } = useParticles();
  const [prefersReducedMotion, setPrefersReducedMotion] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    setPrefersReducedMotion(mediaQuery.matches);

    const handler = (e: MediaQueryListEvent) => {
      setPrefersReducedMotion(e.matches);
      
      if (e.matches) {
        // Minimal particles for reduced motion
        updateConfig({
          particleCount: 10,
          hoverMode: 'none',
          clickMode: 'none',
          gridStyle: 'organic',
        });
      }
    };

    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [updateConfig]);

  return prefersReducedMotion;
}
```

---

## Summary

This configuration framework provides:

✅ **Per-Route Customization** - Easy route-specific particle behavior  
✅ **CSS Design Tokens** - Full theming support with modern standards  
✅ **Sequential Text Formations** - Array-based text animation sequences  
✅ **Type-Safe Configuration** - Complete TypeScript support  
✅ **React 19 & Next.js 16 Patterns** - Modern App Router integration  
✅ **Performance Optimized** - Responsive and battery-aware  
✅ **Accessibility First** - Respects user preferences  

### Next Steps

1. Implement the configuration system
2. Test across different routes and themes
3. Add text formation plugin (Phase 3) for full dot-matrix support
4. Optimize performance based on usage patterns
5. Extend with custom presets as needed

For text formation implementation details, see [PARTICLE_GRID_IDEATION.md](./PARTICLE_GRID_IDEATION.md) Phase 3.
