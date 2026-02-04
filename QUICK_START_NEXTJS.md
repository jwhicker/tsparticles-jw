# Quick Start: Particle Grid for Next.js 16

This guide shows you how to quickly set up a persistent particle grid background in your Next.js 16 application using the existing tsParticles features.

## Installation

```bash
npm install @tsparticles/engine @tsparticles/react \
  @tsparticles/plugin-interactivity \
  @tsparticles/path-grid \
  @tsparticles/path-perlin-noise \
  @tsparticles/plugin-themes \
  @tsparticles/plugin-responsive
```

## Basic Setup

### 1. Create Particles Context (`contexts/ParticlesContext.tsx`)

```typescript
'use client';

import { createContext, useContext, useRef, useEffect, ReactNode } from 'react';
import { Container, Engine, ISourceOptions } from '@tsparticles/engine';

interface ParticlesContextValue {
  container: Container | null;
  initialize: (engine: Engine, options: ISourceOptions) => Promise<void>;
}

const ParticlesContext = createContext<ParticlesContextValue | null>(null);

export function ParticlesProvider({ children }: { children: ReactNode }) {
  const containerRef = useRef<Container | null>(null);
  const canvasRef = useRef<HTMLCanvasElement | null>(null);

  const initialize = async (engine: Engine, options: ISourceOptions) => {
    if (!containerRef.current && canvasRef.current) {
      containerRef.current = await engine.load({
        element: canvasRef.current,
        options
      });
    }
  };

  useEffect(() => {
    return () => {
      containerRef.current?.destroy();
    };
  }, []);

  return (
    <ParticlesContext.Provider value={{ container: containerRef.current, initialize }}>
      <canvas
        ref={canvasRef}
        style={{
          position: 'fixed',
          top: 0,
          left: 0,
          width: '100vw',
          height: '100vh',
          zIndex: -1,
          pointerEvents: 'auto'
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
```

### 2. Wrap App with Provider (`app/layout.tsx`)

```typescript
import { ParticlesProvider } from '@/contexts/ParticlesContext';

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

### 3. Initialize Particles (`components/ParticlesInit.tsx`)

```typescript
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';
import { tsParticles } from '@tsparticles/engine';
import { loadInteractivityPlugin } from '@tsparticles/plugin-interactivity';
import { loadGridPath } from '@tsparticles/path-grid';
import { loadPerlinNoisePath } from '@tsparticles/path-perlin-noise';

export function ParticlesInit() {
  const { initialize } = useParticles();

  useEffect(() => {
    const init = async () => {
      // Load plugins
      await loadInteractivityPlugin(tsParticles);
      await loadGridPath(tsParticles);
      await loadPerlinNoisePath(tsParticles);

      // Initialize with config
      await initialize(tsParticles, {
        background: {
          color: {
            value: 'transparent'
          }
        },
        fpsLimit: 120,
        particles: {
          number: {
            value: 80,
            density: {
              enable: true
            }
          },
          color: {
            value: '#888888'
          },
          shape: {
            type: 'circle'
          },
          opacity: {
            value: 0.5
          },
          size: {
            value: { min: 1, max: 3 }
          },
          move: {
            enable: true,
            speed: 1,
            path: {
              enable: true,
              generator: 'perlinNoise',
              options: {
                columns: 10,
                rows: 10
              }
            }
          },
          links: {
            enable: true,
            distance: 150,
            color: '#888888',
            opacity: 0.4,
            width: 1
          }
        },
        interactivity: {
          events: {
            onHover: {
              enable: true,
              mode: 'grab'
            },
            onClick: {
              enable: true,
              mode: 'push'
            }
          },
          modes: {
            grab: {
              distance: 200,
              links: {
                opacity: 0.8
              }
            },
            push: {
              quantity: 4
            }
          }
        },
        detectRetina: true
      });
    };

    init();
  }, [initialize]);

  return null;
}
```

### 4. Add to Page (`app/page.tsx`)

```typescript
import { ParticlesInit } from '@/components/ParticlesInit';

export default function Home() {
  return (
    <>
      <ParticlesInit />
      <main>
        <h1>Your Content Here</h1>
      </main>
    </>
  );
}
```

## Grid Configuration

For a more structured grid appearance:

```typescript
const gridOptions = {
  particles: {
    number: {
      value: 100
    },
    move: {
      enable: true,
      speed: 0.5,
      path: {
        enable: true,
        generator: 'grid',
        options: {
          cellSize: 50 // Grid cell size in pixels
        }
      }
    }
  }
};
```

## Dark/Light Mode Support

### Add Theme Plugin

```bash
npm install @tsparticles/plugin-themes
```

### Configure Themes

```typescript
import { loadThemesPlugin } from '@tsparticles/plugin-themes';

await loadThemesPlugin(tsParticles);

const optionsWithThemes = {
  // ... other options
  themes: [
    {
      name: 'light',
      default: {
        value: true,
        mode: 'light'
      },
      options: {
        particles: {
          color: { value: '#000000' },
          links: { color: '#000000' }
        }
      }
    },
    {
      name: 'dark',
      default: {
        value: true,
        mode: 'dark'
      },
      options: {
        particles: {
          color: { value: '#ffffff' },
          links: { color: '#ffffff' }
        }
      }
    }
  ]
};
```

## Responsive Configuration

```bash
npm install @tsparticles/plugin-responsive
```

```typescript
import { loadResponsivePlugin } from '@tsparticles/plugin-responsive';

await loadResponsivePlugin(tsParticles);

const responsiveOptions = {
  // ... other options
  responsive: [
    {
      maxWidth: 768,
      options: {
        particles: {
          number: { value: 40 }, // Fewer particles on mobile
          move: { speed: 0.5 }
        }
      }
    },
    {
      maxWidth: 1024,
      options: {
        particles: {
          number: { value: 60 }
        }
      }
    }
  ]
};
```

## Loading State Hook

```typescript
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

export function useLoadingEffect(isLoading: boolean) {
  const { container } = useParticles();

  useEffect(() => {
    if (!container) return;

    if (isLoading) {
      // Apply loading effect
      container.pause();
      // Add custom loading animation here
    } else {
      // Resume normal behavior
      container.play();
    }
  }, [isLoading, container]);
}
```

## Next Steps

For advanced features like text formations and custom transitions, see the full [PARTICLE_GRID_IDEATION.md](./PARTICLE_GRID_IDEATION.md) document.

## Available Plugins

- **Paths**: grid, perlinNoise, simplexNoise, curlNoise, spiral, polygon, curves
- **Interactions**: grab, attract, repulse, bubble, push, trail, bounce, parallax
- **Shapes**: circle, square, triangle, star, polygon, text, image, emoji
- **Effects**: trail, shadow, bubble
- **Updaters**: color, size, opacity, rotate, tilt, roll, wobble, twinkle

## Troubleshooting

### SSR Issues
If you get SSR errors, ensure components using tsParticles are client components with `'use client'` directive.

### Performance Issues
- Reduce particle count
- Disable links on mobile
- Use simpler shapes
- Reduce interactivity distance

### Canvas Not Showing
- Check z-index in styles
- Ensure canvas has proper dimensions
- Verify plugins are loaded before initialization

---

For complete documentation, visit: https://particles.js.org
