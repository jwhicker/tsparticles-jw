# Real-World Implementation Example

## Complete Next.js 16 Particle System Setup

This document provides a complete, copy-paste-ready implementation of the particle system with per-route behavior, CSS tokens, and all the features discussed in the configuration framework.

---

## Project Structure

```
my-nextjs-app/
├── app/
│   ├── layout.tsx                 # Root layout with ParticlesProvider
│   ├── page.tsx                   # Home page
│   ├── about/page.tsx             # About page with custom config
│   ├── projects/page.tsx          # Projects page with high interactivity
│   ├── contact/page.tsx           # Contact page
│   ├── under-construction/        # Construction page with text sequence
│   │   └── page.tsx
│   └── globals.css                # Global styles with CSS tokens
├── components/
│   ├── ParticlesInitializer.tsx   # Initialize particle engine
│   ├── ThemeSwitcher.tsx          # Theme toggle component
│   └── ModeSelector.tsx           # Mode selector (reading/focus/presentation)
├── contexts/
│   └── ParticlesContext.tsx       # Main particle context with route awareness
├── config/
│   ├── design-tokens.ts           # Centralized design tokens
│   ├── particle-presets.ts        # Reusable particle configurations
│   ├── route-particles.ts         # Per-route particle configurations
│   └── mode-particles.ts          # Mode-specific configurations
├── lib/
│   ├── config-to-options.ts       # Convert config to tsParticles options
│   └── css-tokens.ts              # CSS variable generation
├── types/
│   └── particle-config.ts         # TypeScript type definitions
└── hooks/
    ├── useRouteParticles.ts       # Hook for dynamic route configs
    └── useReducedMotion.ts        # Accessibility hook
```

---

## Step 1: Install Dependencies

```bash
npm install @tsparticles/engine @tsparticles/react \
  @tsparticles/plugin-interactivity \
  @tsparticles/plugin-themes \
  @tsparticles/plugin-responsive \
  @tsparticles/path-grid \
  @tsparticles/path-perlin-noise \
  @tsparticles/path-simplex-noise \
  @tsparticles/path-curl-noise \
  @tsparticles/shape-text \
  @tsparticles/shape-image
```

---

## Step 2: Copy All Configuration Files

All configuration files from [NEXTJS_CONFIGURATION_FRAMEWORK.md](./NEXTJS_CONFIGURATION_FRAMEWORK.md) should be created in your project.

Key files to create:
1. `types/particle-config.ts` - Type definitions
2. `config/design-tokens.ts` - Design tokens
3. `config/particle-presets.ts` - Preset configurations
4. `config/route-particles.ts` - Per-route configurations
5. `lib/config-to-options.ts` - Configuration converter
6. `lib/css-tokens.ts` - CSS token utilities
7. `contexts/ParticlesContext.tsx` - Main context
8. `app/globals.css` - Global styles

---

## Step 3: Root Layout Setup

```typescript
// app/layout.tsx
import { ParticlesProvider } from '@/contexts/ParticlesContext';
import { ParticlesInitializer } from '@/components/ParticlesInitializer';
import { ThemeSwitcher } from '@/components/ThemeSwitcher';
import '@/app/globals.css';

export const metadata = {
  title: 'My Particle App',
  description: 'Next.js app with interactive particle background',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <ParticlesProvider>
          <ParticlesInitializer />
          
          {/* Theme switcher in top-right */}
          <div style={{ position: 'fixed', top: 20, right: 20, zIndex: 1000 }}>
            <ThemeSwitcher />
          </div>
          
          {/* Main content */}
          <div style={{ position: 'relative', zIndex: 1 }}>
            {children}
          </div>
        </ParticlesProvider>
      </body>
    </html>
  );
}
```

---

## Step 4: Page Examples

### Home Page (Default Behavior)

```typescript
// app/page.tsx
export default function HomePage() {
  return (
    <main className="container mx-auto px-4 py-16">
      <h1 className="text-5xl font-bold mb-8">Welcome</h1>
      <p className="text-xl">
        This page uses the default particle configuration with interactive grab and push modes.
      </p>
    </main>
  );
}
```

### About Page (Minimal Particles)

```typescript
// app/about/page.tsx
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

export default function AboutPage() {
  const { updateConfig } = useParticles();

  useEffect(() => {
    // Override with minimal particles for reading focus
    updateConfig({
      gridStyle: 'organic',
      particleCount: 20,
      hoverMode: 'parallax',
      clickMode: 'none',
      transitionIn: 'fade-in',
    });
  }, [updateConfig]);

  return (
    <main className="container mx-auto px-4 py-16">
      <h1 className="text-5xl font-bold mb-8">About Us</h1>
      <p className="text-xl mb-4">
        This page has minimal particles to reduce distraction while reading.
      </p>
      <p className="text-lg text-gray-600">
        Notice the subtle parallax effect as you move your mouse. 
        The organic grid style creates a calming atmosphere.
      </p>
    </main>
  );
}
```

### Projects Page (High Interactivity)

```typescript
// app/projects/page.tsx
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

export default function ProjectsPage() {
  const { updateConfig } = useParticles();

  useEffect(() => {
    // High interactivity for showcase
    updateConfig({
      gridStyle: 'curl-noise',
      particleCount: 120,
      hoverMode: ['grab', 'bubble', 'connect'],
      clickMode: 'repulse',
      transitionIn: 'swirl',
      transitionOut: 'explode',
    });
  }, [updateConfig]);

  return (
    <main className="container mx-auto px-4 py-16">
      <h1 className="text-5xl font-bold mb-8">Our Projects</h1>
      <p className="text-xl mb-4">
        This page features high particle interactivity. Try hovering and clicking!
      </p>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mt-8">
        <ProjectCard title="Project 1" description="Interactive web app" />
        <ProjectCard title="Project 2" description="Mobile application" />
        <ProjectCard title="Project 3" description="Data visualization" />
      </div>
    </main>
  );
}

function ProjectCard({ title, description }: { title: string; description: string }) {
  return (
    <div className="p-6 bg-white dark:bg-gray-800 rounded-lg shadow-lg">
      <h3 className="text-2xl font-bold mb-2">{title}</h3>
      <p className="text-gray-600 dark:text-gray-300">{description}</p>
    </div>
  );
}
```

### Under Construction Page (Sequential Text)

```typescript
// app/under-construction/page.tsx
'use client';

import { useEffect } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

export default function ConstructionPage() {
  const { playTextFormations, updateConfig } = useParticles();

  useEffect(() => {
    // Configure for text display
    updateConfig({
      gridStyle: 'grid',
      particleCount: 150,
      hoverMode: 'attract',
      clickMode: 'push',
      transitionIn: 'fade-in',
    });

    // Play sequential text formations
    playTextFormations([
      {
        text: 'UNDER CONSTRUCTION',
        duration: 3000,
        delay: 500,
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
    ]);
  }, [playTextFormations, updateConfig]);

  return (
    <main className="flex items-center justify-center min-h-screen">
      <div className="text-center">
        <div className="text-6xl mb-8">🚧</div>
        <h1 className="text-4xl font-bold mb-4">Page Under Construction</h1>
        <p className="text-xl text-gray-600 dark:text-gray-300">
          Watch the particles form text!
        </p>
      </div>
    </main>
  );
}
```

### Contact Page (Moderate Interaction)

```typescript
// app/contact/page.tsx
export default function ContactPage() {
  // Uses route-based config from route-particles.ts
  return (
    <main className="container mx-auto px-4 py-16">
      <h1 className="text-5xl font-bold mb-8">Contact Us</h1>
      
      <form className="max-w-lg">
        <div className="mb-4">
          <label className="block mb-2 font-semibold">Name</label>
          <input 
            type="text" 
            className="w-full p-2 border rounded"
            placeholder="Your name"
          />
        </div>
        
        <div className="mb-4">
          <label className="block mb-2 font-semibold">Email</label>
          <input 
            type="email" 
            className="w-full p-2 border rounded"
            placeholder="your@email.com"
          />
        </div>
        
        <div className="mb-4">
          <label className="block mb-2 font-semibold">Message</label>
          <textarea 
            className="w-full p-2 border rounded h-32"
            placeholder="Your message"
          />
        </div>
        
        <button 
          type="submit"
          className="px-6 py-3 bg-blue-500 text-white rounded hover:bg-blue-600"
        >
          Send Message
        </button>
      </form>
    </main>
  );
}
```

---

## Step 5: Add Mode Selector Component

```typescript
// components/ModeSelector.tsx
'use client';

import { useState } from 'react';
import { useParticles } from '@/contexts/ParticlesContext';

type Mode = 'default' | 'reading' | 'presentation' | 'focus';

export function ModeSelector() {
  const { updateConfig } = useParticles();
  const [mode, setMode] = useState<Mode>('default');

  const handleModeChange = (newMode: Mode) => {
    setMode(newMode);
    
    const configs = {
      default: {
        particleCount: 80,
        hoverMode: 'grab' as const,
        clickMode: 'push' as const,
        gridStyle: 'perlin-noise' as const,
      },
      reading: {
        particleCount: 20,
        hoverMode: 'none' as const,
        clickMode: 'none' as const,
        gridStyle: 'organic' as const,
      },
      presentation: {
        particleCount: 150,
        hoverMode: ['grab', 'bubble', 'connect'] as const,
        clickMode: 'repulse' as const,
        gridStyle: 'curl-noise' as const,
      },
      focus: {
        particleCount: 10,
        hoverMode: 'parallax' as const,
        clickMode: 'none' as const,
        gridStyle: 'organic' as const,
      },
    };

    updateConfig(configs[newMode]);
  };

  return (
    <div className="fixed bottom-4 left-4 bg-white dark:bg-gray-800 p-4 rounded-lg shadow-lg z-50">
      <label className="block mb-2 text-sm font-semibold">Particle Mode</label>
      <select
        value={mode}
        onChange={(e) => handleModeChange(e.target.value as Mode)}
        className="w-full p-2 border rounded bg-white dark:bg-gray-700"
      >
        <option value="default">Default</option>
        <option value="reading">Reading</option>
        <option value="presentation">Presentation</option>
        <option value="focus">Focus</option>
      </select>
    </div>
  );
}
```

Then add it to your layout:

```typescript
// app/layout.tsx (updated)
import { ModeSelector } from '@/components/ModeSelector';

// ... in the body
<ParticlesProvider>
  <ParticlesInitializer />
  <ThemeSwitcher />
  <ModeSelector /> {/* Add this */}
  <div style={{ position: 'relative', zIndex: 1 }}>
    {children}
  </div>
</ParticlesProvider>
```

---

## Step 6: Add Navigation

```typescript
// components/Navigation.tsx
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';

export function Navigation() {
  const pathname = usePathname();

  const links = [
    { href: '/', label: 'Home' },
    { href: '/about', label: 'About' },
    { href: '/projects', label: 'Projects' },
    { href: '/contact', label: 'Contact' },
    { href: '/under-construction', label: 'Construction' },
  ];

  return (
    <nav className="fixed top-0 left-0 right-0 bg-white/80 dark:bg-gray-900/80 backdrop-blur-sm z-50 border-b border-gray-200 dark:border-gray-700">
      <div className="container mx-auto px-4">
        <ul className="flex space-x-6 py-4">
          {links.map((link) => (
            <li key={link.href}>
              <Link
                href={link.href}
                className={`hover:text-blue-500 transition-colors ${
                  pathname === link.href
                    ? 'text-blue-500 font-semibold'
                    : 'text-gray-700 dark:text-gray-300'
                }`}
              >
                {link.label}
              </Link>
            </li>
          ))}
        </ul>
      </div>
    </nav>
  );
}
```

Add to layout:

```typescript
// app/layout.tsx (final version)
import { ParticlesProvider } from '@/contexts/ParticlesContext';
import { ParticlesInitializer } from '@/components/ParticlesInitializer';
import { ThemeSwitcher } from '@/components/ThemeSwitcher';
import { ModeSelector } from '@/components/ModeSelector';
import { Navigation } from '@/components/Navigation';
import '@/app/globals.css';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <ParticlesProvider>
          <ParticlesInitializer />
          <Navigation />
          
          <div style={{ position: 'fixed', top: 20, right: 20, zIndex: 1000 }}>
            <ThemeSwitcher />
          </div>
          
          <ModeSelector />
          
          {/* Add padding-top to account for fixed nav */}
          <div style={{ position: 'relative', zIndex: 1, paddingTop: '80px' }}>
            {children}
          </div>
        </ParticlesProvider>
      </body>
    </html>
  );
}
```

---

## Step 7: Add Styling

```css
/* app/globals.css (additions) */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* CSS Variables from design tokens */
:root {
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
  :root:not([data-theme='light']) {
    --color-primary: #60a5fa;
    --color-secondary: #a78bfa;
    --color-background: #0f172a;
    --color-surface: #1e293b;
    --color-text: #f8fafc;
    --color-particle: #94a3b8;
    --color-particle-link: #475569;
  }
}

/* Smooth transitions for theme changes */
body {
  background-color: var(--color-background);
  color: var(--color-text);
  transition: background-color 0.3s ease, color 0.3s ease;
}

/* Ensure content is above particles */
main {
  position: relative;
  z-index: 1;
}

/* Custom button styles using tokens */
.btn-primary {
  background-color: var(--color-primary);
  color: white;
  padding: var(--space-md) var(--space-xl);
  border-radius: var(--space-sm);
  transition: all 0.2s ease;
}

.btn-primary:hover {
  opacity: 0.9;
  transform: translateY(-2px);
}
```

---

## Step 8: Testing the Implementation

1. **Start the dev server:**
   ```bash
   npm run dev
   ```

2. **Test each page:**
   - **Home** - Should show default interactive particles
   - **About** - Should transition to minimal particles
   - **Projects** - Should show high interactivity (grab, bubble, connect)
   - **Contact** - Should show moderate interaction
   - **Construction** - Should play text sequence (when Phase 3 is implemented)

3. **Test interactions:**
   - Hover over particles on each page
   - Click to see click interactions
   - Toggle theme switcher (top-right)
   - Try mode selector (bottom-left)

4. **Test responsiveness:**
   - Resize browser window
   - Check mobile view (fewer particles automatically)
   - Verify theme respects system preference

---

## Customization Tips

### Add Your Own Route Config

```typescript
// config/route-particles.ts
export const routeParticleConfig: RouteParticleMap = {
  // ... existing configs
  
  '/my-custom-page': {
    gridStyle: 'simplex-noise',
    particleCount: 100,
    hoverMode: 'bubble',
    clickMode: 'attract',
    transitionIn: 'swirl',
    customOptions: {
      // Override any tsParticles option
      particles: {
        opacity: {
          value: 0.8,
        },
      },
    },
  },
};
```

### Create Custom Presets

```typescript
// config/particle-presets.ts
export const particlePresets = {
  // ... existing presets
  
  myCustomPreset: {
    gridStyle: 'curl-noise',
    particleCount: 75,
    hoverMode: ['grab', 'connect'],
    clickMode: 'push',
    transitionIn: 'fade-in',
    useSystemTheme: true,
  } as ParticleRouteConfig,
};
```

### Override Colors per Route

```typescript
// In your page component
updateConfig({
  gridStyle: 'perlin-noise',
  particleCount: 80,
  hoverMode: 'grab',
  customOptions: {
    particles: {
      color: {
        value: '#ff0000', // Red particles
      },
      links: {
        color: '#ff0000',
      },
    },
  },
});
```

---

## Troubleshooting

### Particles Not Showing
- Check browser console for errors
- Verify all plugins are loaded in `ParticlesInitializer`
- Check canvas z-index styling

### Route Transitions Not Working
- Ensure `usePathname()` is working (Next.js 13+ App Router)
- Check that `updateConfig` is called in `useEffect`
- Verify route patterns in `route-particles.ts`

### Theme Not Switching
- Check `data-theme` attribute on `<html>` element
- Verify CSS variables are defined
- Check browser DevTools for CSS variable values

### Performance Issues
- Reduce particle count for affected pages
- Disable links on mobile: add to responsive config
- Use simpler grid styles (organic vs curl-noise)

---

## Next Steps

1. ✅ Basic setup complete
2. ✅ Per-route configuration working
3. ✅ Theme switching functional
4. ⏳ Implement Phase 3 for actual text formations
5. ⏳ Add custom transition effects
6. ⏳ Optimize performance based on analytics

---

## Summary

You now have a complete, production-ready particle system with:
- ✅ Per-route particle behavior
- ✅ CSS design token integration
- ✅ Theme switching
- ✅ Mode selector
- ✅ Responsive design
- ✅ Accessibility support
- ✅ Type-safe configuration

The sequential text formations ("UNDER CONSTRUCTION" → "COMING SOON") will work once the Phase 3 formations plugin is implemented. Until then, the framework supports all other features out of the box!
