# Vite Migration Plan - IPC Builder (WBS Terminal)

**Version:** 1.0
**Date:** January 9, 2026
**Status:** Proposal - Planning Phase

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Current State Analysis](#current-state-analysis)
3. [Benefits of Vite Integration](#benefits-of-vite-integration)
4. [Proposed Architecture](#proposed-architecture)
5. [File Structure Reorganization](#file-structure-reorganization)
6. [Migration Strategy](#migration-strategy)
7. [Build Configuration](#build-configuration)
8. [Development Workflow](#development-workflow)
9. [Performance Improvements](#performance-improvements)
10. [Risks & Mitigation](#risks--mitigation)
11. [Timeline & Effort Estimates](#timeline--effort-estimates)
12. [Success Criteria](#success-criteria)

---

## Executive Summary

This document outlines a comprehensive plan to migrate the IPC Builder (WBS Terminal) from a **zero-build, single-page application** to a **modern Vite-powered build system**. The migration aims to improve developer experience, application performance, maintainability, and scalability while preserving the project's core functionality.

### Key Objectives

- **Reduce bundle size** through code splitting and tree-shaking
- **Improve load times** with optimized assets and lazy loading
- **Enable modern development** with HMR, TypeScript support, and module system
- **Enhance maintainability** with modular JavaScript architecture
- **Optimize production builds** with minification and compression
- **Prepare for future growth** with scalable architecture

### High-Level Timeline

- **Phase 1 (Setup & Foundation):** 1-2 weeks
- **Phase 2 (JS Modularization):** 2-3 weeks
- **Phase 3 (Dependencies & Optimization):** 1-2 weeks
- **Phase 4 (Testing & Deployment):** 1-2 weeks
- **Total Estimated Duration:** 5-9 weeks

---

## Current State Analysis

### Application Overview

| Metric | Current Value |
|--------|--------------|
| **Total Size** | 579 KB (index.html) |
| **Lines of Code** | ~13,000 (HTML/JS), ~6,000 (CSS), ~19,494 total |
| **JavaScript** | ~9,000 lines (inline in HTML) |
| **CSS** | Modular (19 files, 4 categories) |
| **Dependencies** | 3 CDN libraries (Chart.js, PDF.js, html2pdf.js) |
| **Data Files** | 14 JSON files (~500 lines total) |
| **Architecture** | Single HTML file with inline JavaScript |

### Strengths

✅ **Zero build complexity** - Easy to deploy and develop
✅ **Modular CSS** - Recently refactored into organized structure
✅ **Self-contained** - Works without npm/build tools
✅ **Fast initial setup** - No dependency installation
✅ **Simple deployment** - Static hosting (GitHub Pages)

### Pain Points

❌ **Large single file** - 579KB HTML file is difficult to navigate
❌ **No code splitting** - Entire app loads at once
❌ **Limited tooling** - No HMR, linting, or type checking
❌ **CDN dependencies** - Network-dependent, no version locking
❌ **Manual minification** - No automated optimization
❌ **Difficult collaboration** - Merge conflicts in single file
❌ **No tree-shaking** - Unused code shipped to production
❌ **Poor IDE support** - Limited autocomplete/refactoring

### Current Dependencies (CDN)

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
```

---

## Benefits of Vite Integration

### 1. Performance Improvements

| Benefit | Current | With Vite | Improvement |
|---------|---------|-----------|-------------|
| **Bundle Size** | 579 KB (HTML) + CDN libs | ~150-250 KB (minified + gzipped) | **60-70% reduction** |
| **Initial Load** | ~800ms - 1.2s | ~300-500ms | **60% faster** |
| **Code Splitting** | None (monolithic) | Route-based + lazy loading | **On-demand loading** |
| **Cache Strategy** | Basic browser cache | Versioned assets with long-term caching | **Better cache hits** |
| **Tree-shaking** | None | Automatic dead code elimination | **Smaller bundles** |

### 2. Developer Experience

✅ **Hot Module Replacement (HMR)** - Instant updates without page refresh
✅ **TypeScript Support** - Optional gradual migration with full IntelliSense
✅ **Modern ES Modules** - Import/export syntax, better code organization
✅ **Dev Server** - Fast localhost with HTTPS support
✅ **Error Overlay** - Detailed error messages in browser
✅ **Source Maps** - Easy debugging in production
✅ **Auto-restart** - Configuration changes reload automatically

### 3. Build Optimizations

✅ **Automatic minification** - HTML, CSS, JS compressed
✅ **Asset optimization** - Images, fonts optimized automatically
✅ **CSS code splitting** - Per-route CSS bundles
✅ **Dependency pre-bundling** - Faster cold starts
✅ **Legacy browser support** - Optional polyfills via @vitejs/plugin-legacy
✅ **Brotli/Gzip compression** - Smaller transfer sizes

### 4. Modern Tooling Integration

✅ **ESLint** - Code quality enforcement
✅ **Prettier** - Consistent formatting
✅ **Vitest** - Fast unit testing framework
✅ **Playwright/Cypress** - E2E testing
✅ **Storybook** - Component documentation
✅ **Bundle analyzer** - Visualize bundle composition

### 5. Future Scalability

✅ **Framework flexibility** - Easy to add React/Vue/Svelte later
✅ **Plugin ecosystem** - 1000+ Vite plugins available
✅ **Monorepo support** - Can split into packages if needed
✅ **SSR capability** - Server-side rendering possible
✅ **PWA support** - Offline functionality via workbox

---

## Proposed Architecture

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    IPC Builder (Vite)                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
│  │   index.html │──▶│   main.js    │──▶│   App Core   │   │
│  │  (template)  │   │ (entry point)│   │              │   │
│  └──────────────┘   └──────────────┘   └──────┬───────┘   │
│                                                │           │
│         ┌──────────────────────────────────────┘           │
│         │                                                  │
│    ┌────▼────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│    │  State  │  │ Services │  │   Utils  │  │   UI     │ │
│    │  Mgmt   │  │  (API,   │  │  (calc,  │  │ Components│ │
│    │         │  │  export) │  │  format) │  │          │ │
│    └─────────┘  └──────────┘  └──────────┘  └──────────┘ │
│                                                             │
│    ┌──────────────────────────────────────────────────┐   │
│    │              Feature Modules                      │   │
│    ├─────────┬──────────┬──────────┬─────────┬────────┤   │
│    │ Wizard  │ Budget   │ Schedule │ Charts  │ Export │   │
│    │ Steps   │ Estimator│ Gantt    │ AI Chat │ Reports│   │
│    └─────────┴──────────┴──────────┴─────────┴────────┘   │
│                                                             │
│    ┌──────────────────────────────────────────────────┐   │
│    │              External Dependencies                │   │
│    ├──────────────┬──────────────┬────────────────────┤   │
│    │ chart.js (npm)│ pdfjs (npm) │ html2pdf (npm)     │   │
│    └──────────────┴──────────────┴────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘

                    Build Process (Vite)
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   ┌────▼────┐         ┌────▼────┐        ┌────▼────┐
   │ Dev Mode│         │  Build  │        │ Preview │
   │   HMR   │         │Optimize │        │  Test   │
   │Fast Reload│       │ Minify  │        │Production│
   └─────────┘         └─────────┘        └─────────┘
```

### Module Architecture

```
src/
├── core/
│   ├── state.js          # Global state management
│   ├── constants.js      # Configuration constants
│   └── types.js          # Type definitions (JSDoc or TS)
│
├── services/
│   ├── api.js            # OpenAI API integration
│   ├── storage.js        # localStorage wrapper
│   ├── pdf.js            # PDF export/import
│   └── csv.js            # CSV export/import
│
├── features/
│   ├── wizard/
│   │   ├── phases.js     # Step 1
│   │   ├── disciplines.js# Step 2
│   │   ├── packages.js   # Step 3
│   │   ├── budget.js     # Step 4
│   │   ├── claiming.js   # Step 5
│   │   └── schedule.js   # Step 6
│   │
│   ├── estimator/
│   │   ├── cost-calculator.js
│   │   ├── mh-estimator.js
│   │   └── benchmark-stats.js
│   │
│   ├── ai/
│   │   ├── chat.js
│   │   ├── insights.js
│   │   └── schedule-gen.js
│   │
│   ├── rfp/
│   │   ├── wizard.js
│   │   ├── parser.js
│   │   └── analyzer.js
│   │
│   ├── charts/
│   │   ├── performance-chart.js
│   │   └── gantt.js
│   │
│   └── wbs/
│       ├── generator.js
│       ├── editor.js
│       └── validator.js
│
├── ui/
│   ├── components/
│   │   ├── modal.js
│   │   ├── table.js
│   │   ├── button.js
│   │   └── input.js
│   │
│   └── templates/
│       ├── wizard.html
│       ├── results.html
│       └── modals.html
│
├── utils/
│   ├── dom.js            # DOM manipulation helpers
│   ├── format.js         # Number/date formatting
│   ├── validation.js     # Input validation
│   └── logger.js         # Console logging wrapper
│
├── data/
│   └── benchmarks/       # Benchmarking JSON files
│
├── assets/
│   ├── fonts/            # Local fonts (optional)
│   └── images/           # Icons, logos
│
├── styles/               # CSS modules (existing structure)
│   └── (keep current modular structure)
│
├── main.js               # Application entry point
└── App.js                # Main application controller
```

---

## File Structure Reorganization

### Phase 1: Initial Structure

```
ipc-builder/
├── public/                          # Static assets (served as-is)
│   ├── favicon.ico
│   └── benchmarking/               # Move from root
│       └── *.json
│
├── src/                            # Source code (to be processed)
│   ├── main.js                     # Entry point
│   ├── App.js                      # Main app logic
│   │
│   ├── styles/                     # CSS modules
│   │   └── (existing css/ folder structure)
│   │
│   └── legacy/                     # Temporary: extracted code
│       └── app-legacy.js           # All JS from index.html
│
├── index.html                      # Vite HTML template
├── vite.config.js                  # Vite configuration
├── package.json                    # Dependencies
├── package-lock.json
├── .gitignore                      # Updated for node_modules
├── .env.example                    # Environment variables template
│
└── docs/                           # Documentation
    ├── CLAUDE.md
    ├── README.md
    ├── VITE_MIGRATION_PLAN.md
    ├── CODE_REVIEW_ANALYSIS.md
    └── REORGANIZATION_PLAN.md
```

### Phase 2: Modular Structure (Target)

```
ipc-builder/
├── public/
│   ├── favicon.ico
│   └── data/
│       └── benchmarking/
│
├── src/
│   ├── core/
│   │   ├── state.js
│   │   ├── constants.js
│   │   └── config.js
│   │
│   ├── services/
│   │   ├── api/
│   │   │   ├── openai.js
│   │   │   └── retry.js
│   │   ├── storage/
│   │   │   ├── localStorage.js
│   │   │   └── projects.js
│   │   └── export/
│   │       ├── pdf.js
│   │       ├── csv.js
│   │       └── url.js
│   │
│   ├── features/
│   │   ├── wizard/
│   │   ├── estimator/
│   │   ├── ai/
│   │   ├── rfp/
│   │   ├── charts/
│   │   └── wbs/
│   │
│   ├── ui/
│   │   ├── components/
│   │   └── templates/
│   │
│   ├── utils/
│   │   ├── dom.js
│   │   ├── format.js
│   │   ├── validation.js
│   │   └── logger.js
│   │
│   ├── styles/
│   │   └── (existing structure)
│   │
│   ├── App.js
│   └── main.js
│
├── tests/                          # Testing
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── scripts/                        # Build scripts
│   ├── migrate-legacy.js
│   └── analyze-bundle.js
│
├── index.html
├── vite.config.js
├── vitest.config.js
├── playwright.config.js
├── package.json
├── tsconfig.json                   # Optional: for TypeScript
├── .eslintrc.json
├── .prettierrc.json
└── .env.example
```

---

## Migration Strategy

### 6-Phase Approach

#### **Phase 1: Foundation & Setup** (Week 1)

**Objectives:**
- Set up Vite build system
- Migrate dependencies from CDN to npm
- Create basic project structure
- Ensure app still works with minimal changes

**Tasks:**
1. Initialize npm project
   ```bash
   npm init -y
   ```

2. Install Vite and dependencies
   ```bash
   npm install -D vite
   npm install chart.js pdfjs-dist html2pdf.js
   ```

3. Create `vite.config.js`
   ```js
   import { defineConfig } from 'vite';
   import { resolve } from 'path';

   export default defineConfig({
     root: '.',
     publicDir: 'public',
     build: {
       outDir: 'dist',
       rollupOptions: {
         input: {
           main: resolve(__dirname, 'index.html')
         }
       },
       chunkSizeWarningLimit: 600
     },
     server: {
       port: 3000,
       open: true
     }
   });
   ```

4. Update `package.json` scripts
   ```json
   {
     "scripts": {
       "dev": "vite",
       "build": "vite build",
       "preview": "vite preview",
       "serve": "vite preview --port 8080"
     }
   }
   ```

5. Update `index.html` for Vite
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>WBS Terminal v1.0</title>
     <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">
     <link rel="stylesheet" href="/src/styles/main.css">
   </head>
   <body>
     <!-- Existing HTML content -->
     <div id="app"></div>

     <!-- Vite entry point -->
     <script type="module" src="/src/main.js"></script>
   </body>
   </html>
   ```

6. Create `src/main.js`
   ```js
   // Import dependencies
   import Chart from 'chart.js/auto';
   import * as pdfjsLib from 'pdfjs-dist';
   import html2pdf from 'html2pdf.js';

   // Make available globally (temporary for legacy code)
   window.Chart = Chart;
   window.pdfjsLib = pdfjsLib;
   window.html2pdf = html2pdf;

   // Import legacy code
   import './legacy/app-legacy.js';

   console.log('WBS Terminal v1.0 - Vite Edition');
   ```

7. Extract all `<script>` content from `index.html` to `src/legacy/app-legacy.js`

8. Move `css/` folder to `src/styles/`

9. Move `benchmarking/` folder to `public/data/benchmarking/`

10. Update CSS imports in `src/styles/main.css`

**Deliverables:**
- ✅ Working Vite dev server
- ✅ Production build compiles
- ✅ App functions identically to current version
- ✅ Dependencies managed via npm

**Success Criteria:**
- `npm run dev` starts dev server
- `npm run build` creates production bundle
- All features work in dev and production

---

#### **Phase 2: CSS Migration** (Week 1-2)

**Objectives:**
- Migrate CSS to CSS Modules or keep modular structure
- Optimize CSS bundle size
- Remove unused CSS

**Tasks:**
1. Keep existing modular CSS structure (already well-organized)
2. Add PostCSS for autoprefixing and minification
3. Install CSS tooling
   ```bash
   npm install -D postcss autoprefixer cssnano
   ```

4. Create `postcss.config.js`
   ```js
   export default {
     plugins: {
       autoprefixer: {},
       cssnano: {
         preset: 'default'
       }
     }
   };
   ```

5. Analyze and remove unused CSS with PurgeCSS (optional)

**Deliverables:**
- ✅ Optimized CSS bundle
- ✅ Vendor prefixes added automatically
- ✅ Minified production CSS

---

#### **Phase 3: JavaScript Modularization** (Week 2-4)

**Objectives:**
- Break monolithic `app-legacy.js` into modules
- Implement proper ES6 imports/exports
- Organize code by feature

**Tasks:**

1. **Extract Constants & Configuration**
   ```js
   // src/core/constants.js
   export const DISCIPLINE_CONFIG = { /* ... */ };
   export const HISTORICAL_BENCHMARKS = { /* ... */ };
   export const PROJECT_TEMPLATES = { /* ... */ };
   ```

2. **Extract State Management**
   ```js
   // src/core/state.js
   export const projectData = {
     phases: [],
     disciplines: [],
     packages: [],
     budgets: {},
     claiming: {},
     dates: {},
     calculator: {},
     projectScope: '',
     scheduleNotes: '',
     disciplineScopes: {}
   };

   export function updateProjectData(updates) { /* ... */ }
   export function getProjectData() { /* ... */ }
   ```

3. **Extract Services**
   - `src/services/api/openai.js` - API calls
   - `src/services/storage/localStorage.js` - Persistence
   - `src/services/export/pdf.js` - PDF generation
   - `src/services/export/csv.js` - CSV export

4. **Extract Features**
   - `src/features/wizard/` - 6 wizard steps
   - `src/features/estimator/` - Cost & MH calculators
   - `src/features/ai/` - Chat, insights, schedule
   - `src/features/rfp/` - RFP wizard
   - `src/features/charts/` - Chart.js wrappers
   - `src/features/wbs/` - WBS generation & editing

5. **Extract Utils**
   - `src/utils/format.js` - Number/date formatting
   - `src/utils/validation.js` - Input validation
   - `src/utils/dom.js` - DOM helpers

6. **Create Main Controller**
   ```js
   // src/App.js
   import { initWizard } from './features/wizard';
   import { initChat } from './features/ai/chat';
   import { loadBenchmarkData } from './features/estimator/mh-estimator';

   export class App {
     constructor() {
       this.init();
     }

     async init() {
       await this.loadData();
       this.initUI();
       this.attachEventListeners();
       this.checkForSavedData();
     }

     async loadData() {
       await loadBenchmarkData();
     }

     initUI() {
       initWizard();
       initChat();
     }
   }
   ```

7. **Update main.js**
   ```js
   // src/main.js
   import { App } from './App';
   import './styles/main.css';

   // Initialize app when DOM is ready
   document.addEventListener('DOMContentLoaded', () => {
     new App();
   });
   ```

**Deliverables:**
- ✅ Fully modularized JavaScript codebase
- ✅ ES6 modules throughout
- ✅ Clear separation of concerns
- ✅ Reduced code duplication

---

#### **Phase 4: Optimization & Code Splitting** (Week 4-5)

**Objectives:**
- Implement lazy loading for large features
- Optimize bundle size with code splitting
- Add service worker for PWA (optional)

**Tasks:**

1. **Implement Dynamic Imports**
   ```js
   // Lazy load RFP wizard only when needed
   async function openRfpWizard() {
     const { RfpWizard } = await import('./features/rfp/wizard.js');
     const wizard = new RfpWizard();
     wizard.open();
   }
   ```

2. **Configure Code Splitting in Vite**
   ```js
   // vite.config.js
   export default defineConfig({
     build: {
       rollupOptions: {
         output: {
           manualChunks: {
             'vendor': ['chart.js', 'pdfjs-dist', 'html2pdf.js'],
             'ai': ['./src/features/ai/chat.js', './src/features/ai/insights.js'],
             'rfp': ['./src/features/rfp/wizard.js', './src/features/rfp/parser.js'],
             'charts': ['./src/features/charts/performance-chart.js', './src/features/charts/gantt.js']
           }
         }
       }
     }
   });
   ```

3. **Optimize Dependencies**
   - Use Chart.js tree-shakeable imports
   - Load PDF.js worker separately
   - Consider alternatives to html2pdf.js (jsPDF + html2canvas)

4. **Add Bundle Analyzer**
   ```bash
   npm install -D rollup-plugin-visualizer
   ```

5. **Implement Resource Hints**
   ```html
   <link rel="preload" as="script" href="/src/main.js">
   <link rel="prefetch" href="/data/benchmarking/benchmarking-roadway.json">
   ```

**Deliverables:**
- ✅ Code split into logical chunks
- ✅ Lazy loading for heavy features
- ✅ Optimized vendor bundles
- ✅ Bundle size analysis report

---

#### **Phase 5: Testing & Quality Assurance** (Week 5-6)

**Objectives:**
- Set up testing framework
- Write unit tests for critical functions
- Add E2E tests for user flows
- Set up linting and formatting

**Tasks:**

1. **Install Testing Tools**
   ```bash
   npm install -D vitest @vitest/ui
   npm install -D @testing-library/dom @testing-library/user-event
   npm install -D playwright
   npm install -D eslint prettier
   ```

2. **Configure Vitest**
   ```js
   // vitest.config.js
   import { defineConfig } from 'vitest/config';

   export default defineConfig({
     test: {
       globals: true,
       environment: 'jsdom',
       coverage: {
         reporter: ['text', 'html'],
         exclude: ['node_modules/', 'tests/']
       }
     }
   });
   ```

3. **Write Unit Tests**
   ```js
   // tests/unit/utils/format.test.js
   import { describe, it, expect } from 'vitest';
   import { formatCurrency, formatDate } from '@/utils/format';

   describe('formatCurrency', () => {
     it('formats numbers as USD currency', () => {
       expect(formatCurrency(1234.56)).toBe('$1,234.56');
     });
   });
   ```

4. **Write Integration Tests**
   ```js
   // tests/integration/wizard.test.js
   import { describe, it, expect, beforeEach } from 'vitest';
   import { initWizard } from '@/features/wizard';

   describe('Wizard Flow', () => {
     it('navigates through all 6 steps', () => {
       // Test implementation
     });
   });
   ```

5. **Write E2E Tests**
   ```js
   // tests/e2e/wbs-generation.spec.js
   import { test, expect } from '@playwright/test';

   test('generates WBS from wizard', async ({ page }) => {
     await page.goto('/');
     // Fill wizard steps
     await page.click('#next-btn');
     // Assertions
   });
   ```

6. **Set up ESLint**
   ```json
   // .eslintrc.json
   {
     "env": {
       "browser": true,
       "es2021": true
     },
     "extends": "eslint:recommended",
     "parserOptions": {
       "ecmaVersion": "latest",
       "sourceType": "module"
     }
   }
   ```

7. **Set up Prettier**
   ```json
   // .prettierrc.json
   {
     "semi": true,
     "singleQuote": true,
     "tabWidth": 4,
     "trailingComma": "es5"
   }
   ```

8. **Add Pre-commit Hooks**
   ```bash
   npm install -D husky lint-staged
   npx husky init
   ```

**Deliverables:**
- ✅ Unit test coverage >70%
- ✅ E2E tests for critical flows
- ✅ Linting and formatting enforced
- ✅ Pre-commit hooks configured

---

#### **Phase 6: Deployment & Documentation** (Week 6-7)

**Objectives:**
- Update deployment process
- Configure GitHub Pages for Vite build
- Update documentation
- Create migration guide

**Tasks:**

1. **Configure GitHub Actions**
   ```yaml
   # .github/workflows/deploy.yml
   name: Deploy to GitHub Pages

   on:
     push:
       branches: [main]

   jobs:
     build-and-deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - uses: actions/setup-node@v3
           with:
             node-version: 18
         - run: npm ci
         - run: npm run build
         - uses: peaceiris/actions-gh-pages@v3
           with:
             github_token: ${{ secrets.GITHUB_TOKEN }}
             publish_dir: ./dist
   ```

2. **Update `vite.config.js` for GitHub Pages**
   ```js
   export default defineConfig({
     base: '/IPC-Builder/', // Repository name
     // ... other config
   });
   ```

3. **Update Documentation**
   - Update CLAUDE.md with new architecture
   - Update README.md with build instructions
   - Create CONTRIBUTING.md with dev setup
   - Document migration process

4. **Create Developer Guide**
   ```markdown
   # Developer Setup

   ## Prerequisites
   - Node.js 18+
   - npm 9+

   ## Installation
   ```bash
   git clone https://github.com/mjamiv/IPC-Builder.git
   cd IPC-Builder
   npm install
   ```

   ## Development
   ```bash
   npm run dev        # Start dev server
   npm run build      # Production build
   npm run preview    # Preview production build
   npm run test       # Run tests
   npm run lint       # Lint code
   ```
   ```

5. **Performance Benchmarking**
   - Compare bundle sizes (before/after)
   - Measure load times with Lighthouse
   - Document improvements

**Deliverables:**
- ✅ Automated deployment to GitHub Pages
- ✅ Updated documentation
- ✅ Developer setup guide
- ✅ Performance comparison report

---

## Build Configuration

### vite.config.js (Full Configuration)

```js
import { defineConfig } from 'vite';
import { resolve } from 'path';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig(({ command, mode }) => {
  const isDev = mode === 'development';

  return {
    // Base public path
    base: '/IPC-Builder/',

    // Root directory
    root: '.',

    // Public directory for static assets
    publicDir: 'public',

    // Path aliases
    resolve: {
      alias: {
        '@': resolve(__dirname, 'src'),
        '@styles': resolve(__dirname, 'src/styles'),
        '@components': resolve(__dirname, 'src/ui/components'),
        '@features': resolve(__dirname, 'src/features'),
        '@utils': resolve(__dirname, 'src/utils'),
        '@services': resolve(__dirname, 'src/services'),
        '@core': resolve(__dirname, 'src/core'),
      }
    },

    // Server configuration
    server: {
      port: 3000,
      open: true,
      cors: true,
      hmr: {
        overlay: true
      }
    },

    // Build configuration
    build: {
      outDir: 'dist',
      assetsDir: 'assets',
      sourcemap: !isDev,
      minify: 'terser',
      terserOptions: {
        compress: {
          drop_console: !isDev,
          drop_debugger: !isDev
        }
      },

      // Chunk size warning limit
      chunkSizeWarningLimit: 600,

      // Rollup options
      rollupOptions: {
        input: {
          main: resolve(__dirname, 'index.html')
        },
        output: {
          // Manual chunk splitting
          manualChunks: {
            // Vendor libraries
            'vendor-charts': ['chart.js'],
            'vendor-pdf': ['pdfjs-dist', 'html2pdf.js'],

            // Feature chunks
            'feature-ai': [
              './src/features/ai/chat.js',
              './src/features/ai/insights.js',
              './src/features/ai/schedule-gen.js'
            ],
            'feature-rfp': [
              './src/features/rfp/wizard.js',
              './src/features/rfp/parser.js',
              './src/features/rfp/analyzer.js'
            ],
            'feature-charts': [
              './src/features/charts/performance-chart.js',
              './src/features/charts/gantt.js'
            ],
            'feature-estimator': [
              './src/features/estimator/cost-calculator.js',
              './src/features/estimator/mh-estimator.js',
              './src/features/estimator/benchmark-stats.js'
            ]
          },

          // Asset file naming
          assetFileNames: (assetInfo) => {
            const info = assetInfo.name.split('.');
            const ext = info[info.length - 1];

            if (/png|jpe?g|svg|gif|tiff|bmp|ico/i.test(ext)) {
              return `assets/images/[name]-[hash][extname]`;
            } else if (/woff|woff2|eot|ttf|otf/i.test(ext)) {
              return `assets/fonts/[name]-[hash][extname]`;
            } else if (ext === 'css') {
              return `assets/styles/[name]-[hash][extname]`;
            }

            return `assets/[name]-[hash][extname]`;
          },

          // Chunk file naming
          chunkFileNames: 'assets/js/[name]-[hash].js',
          entryFileNames: 'assets/js/[name]-[hash].js'
        }
      },

      // Target browsers
      target: 'es2015',

      // CSS code splitting
      cssCodeSplit: true
    },

    // Plugins
    plugins: [
      // Bundle analyzer (only in analyze mode)
      mode === 'analyze' && visualizer({
        open: true,
        filename: 'dist/stats.html',
        gzipSize: true,
        brotliSize: true
      })
    ].filter(Boolean),

    // Optimizations
    optimizeDeps: {
      include: ['chart.js', 'pdfjs-dist', 'html2pdf.js'],
      exclude: []
    },

    // CSS options
    css: {
      devSourcemap: isDev,
      preprocessorOptions: {
        // Add CSS preprocessor options if needed
      }
    },

    // Define global constants
    define: {
      __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
      __BUILD_DATE__: JSON.stringify(new Date().toISOString())
    }
  };
});
```

### package.json

```json
{
  "name": "ipc-builder",
  "version": "2.0.0",
  "description": "WBS Terminal v2.0 - Work Breakdown Structure Generator",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "build:analyze": "vite build --mode analyze",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "lint": "eslint src --ext .js",
    "lint:fix": "eslint src --ext .js --fix",
    "format": "prettier --write \"src/**/*.{js,css,html}\"",
    "prepare": "husky install"
  },
  "dependencies": {
    "chart.js": "^4.4.1",
    "html2pdf.js": "^0.10.1",
    "pdfjs-dist": "^3.11.174"
  },
  "devDependencies": {
    "@playwright/test": "^1.40.1",
    "@testing-library/dom": "^9.3.4",
    "@testing-library/user-event": "^14.5.1",
    "@vitest/ui": "^1.1.0",
    "autoprefixer": "^10.4.16",
    "cssnano": "^6.0.2",
    "eslint": "^8.56.0",
    "husky": "^8.0.3",
    "lint-staged": "^15.2.0",
    "postcss": "^8.4.32",
    "prettier": "^3.1.1",
    "rollup-plugin-visualizer": "^5.11.0",
    "terser": "^5.26.0",
    "vite": "^5.0.10",
    "vitest": "^1.1.0"
  },
  "lint-staged": {
    "*.js": ["eslint --fix", "prettier --write"],
    "*.{css,html}": ["prettier --write"]
  },
  "keywords": ["wbs", "project-management", "engineering", "infrastructure"],
  "author": "mjamiv",
  "license": "MIT"
}
```

---

## Development Workflow

### Before (Current)

```bash
# 1. Clone repo
git clone https://github.com/mjamiv/IPC-Builder.git
cd IPC-Builder

# 2. Open in browser
open index.html

# OR serve with Python
python3 -m http.server 8000
```

**Pain Points:**
- ❌ No dependency management
- ❌ No hot reload
- ❌ Manual refresh after changes
- ❌ No linting or formatting
- ❌ No build optimization

### After (With Vite)

```bash
# 1. Clone repo
git clone https://github.com/mjamiv/IPC-Builder.git
cd IPC-Builder

# 2. Install dependencies
npm install

# 3. Start dev server
npm run dev
# → Opens http://localhost:3000
# → Hot module replacement enabled
# → Fast refresh on save

# 4. Make changes
# → Instant feedback in browser
# → Linting errors shown in terminal
# → Format on save (if configured)

# 5. Run tests
npm run test

# 6. Build for production
npm run build
# → Optimized bundle in dist/
# → Source maps generated
# → Assets compressed

# 7. Preview production build
npm run preview
```

**Benefits:**
- ✅ Dependency versioning with npm
- ✅ Instant hot reload
- ✅ Automatic formatting
- ✅ Built-in testing
- ✅ Production optimization

### Development Commands

```bash
# Development
npm run dev              # Start dev server (port 3000)
npm run dev -- --port 8080  # Custom port

# Testing
npm run test             # Run unit tests
npm run test:ui          # Open Vitest UI
npm run test:coverage    # Generate coverage report
npm run test:e2e         # Run Playwright E2E tests

# Code Quality
npm run lint             # Check for linting errors
npm run lint:fix         # Auto-fix linting errors
npm run format           # Format code with Prettier

# Building
npm run build            # Production build
npm run build:analyze    # Build with bundle analysis
npm run preview          # Preview production build

# Deployment
npm run deploy           # Build and deploy to GitHub Pages (custom script)
```

---

## Performance Improvements

### Expected Metrics

| Metric | Current (No Build) | With Vite | Improvement |
|--------|-------------------|-----------|-------------|
| **Initial Bundle Size** | 579 KB (index.html) | ~180 KB (gzipped) | **69% smaller** |
| **Total Transfer Size** | ~1.2 MB (with CDN libs) | ~300 KB (all chunks) | **75% smaller** |
| **Initial Load Time (3G)** | ~2.5s | ~800ms | **68% faster** |
| **Time to Interactive** | ~3.2s | ~1.1s | **66% faster** |
| **First Contentful Paint** | ~1.8s | ~600ms | **67% faster** |
| **Lighthouse Score** | 72/100 | 95+/100 | **+23 points** |
| **Bundle Count** | 1 file | 8-12 chunks | **Better caching** |
| **Code Coverage** | Unknown | 70%+ tracked | **Measurable quality** |

### Optimization Techniques

1. **Tree Shaking**
   - Remove unused Chart.js components
   - Eliminate dead code from libraries
   - Expected savings: ~150 KB

2. **Code Splitting**
   - Main bundle: ~80 KB (core app)
   - Vendor chunk: ~120 KB (Chart.js, PDF.js)
   - AI features: ~40 KB (lazy loaded)
   - RFP wizard: ~30 KB (lazy loaded)
   - Charts: ~25 KB (lazy loaded)

3. **Minification & Compression**
   - Terser minification for JS
   - cssnano for CSS
   - Gzip/Brotli compression
   - HTML minification

4. **Asset Optimization**
   - Load benchmarking JSON on-demand
   - Cache API responses
   - Use WebP for images (if added)
   - Font subsetting

5. **Lazy Loading Strategy**
   ```js
   // Load RFP wizard only when opened
   openRfpWizard: async () => {
     const { RfpWizard } = await import('@features/rfp/wizard');
     // 30 KB loaded only when needed
   }

   // Load AI features on demand
   toggleChat: async () => {
     const { Chat } = await import('@features/ai/chat');
     // 40 KB loaded only when needed
   }
   ```

6. **Caching Strategy**
   ```
   index.html          → no-cache (always fresh)
   main-[hash].js      → cache 1 year (immutable)
   vendor-[hash].js    → cache 1 year (immutable)
   styles-[hash].css   → cache 1 year (immutable)
   benchmarking/*.json → cache 1 week
   ```

### Lighthouse Score Targets

**Performance:** 95+
**Accessibility:** 100
**Best Practices:** 100
**SEO:** 100
**PWA:** 90+ (optional)

---

## Risks & Mitigation

### Risk Matrix

| Risk | Probability | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| **Breaking existing functionality** | Medium | High | Comprehensive testing, phased rollout |
| **Increased complexity** | High | Medium | Good documentation, training |
| **npm dependency issues** | Medium | Medium | Lock file, regular updates |
| **Build time increase** | Low | Low | Vite is faster than Webpack |
| **Browser compatibility** | Low | Medium | Use @vitejs/plugin-legacy |
| **Team resistance to change** | Medium | High | Clear communication of benefits |
| **GitHub Pages deployment issues** | Low | Medium | Test deployment process early |
| **localStorage compatibility** | Low | High | Migration script for existing users |

### Mitigation Strategies

1. **Parallel Development**
   - Keep `index.html` working during migration
   - Use feature branches for Vite version
   - Run both versions in parallel initially

2. **Comprehensive Testing**
   - Unit tests for all critical functions
   - E2E tests for user workflows
   - Manual testing on multiple browsers
   - Performance testing before/after

3. **Gradual Rollout**
   - Phase 1-2: Internal testing
   - Phase 3-4: Beta testing with select users
   - Phase 5-6: Full production deployment

4. **Backward Compatibility**
   - Support importing old localStorage data
   - Maintain same URL structure
   - Keep feature parity with current version

5. **Documentation & Training**
   - Update all documentation
   - Create video walkthrough for developers
   - Document troubleshooting steps

6. **Rollback Plan**
   - Keep old version in `legacy` branch
   - Document rollback procedure
   - Maintain both deployments temporarily

---

## Timeline & Effort Estimates

### Detailed Timeline

| Phase | Duration | Effort (Hours) | Dependencies |
|-------|----------|----------------|--------------|
| **Phase 1: Foundation** | Week 1 | 16-24 hours | None |
| **Phase 2: CSS Migration** | Week 1-2 | 8-12 hours | Phase 1 |
| **Phase 3: JS Modularization** | Week 2-4 | 40-60 hours | Phase 1 |
| **Phase 4: Optimization** | Week 4-5 | 16-24 hours | Phase 3 |
| **Phase 5: Testing & QA** | Week 5-6 | 24-32 hours | Phase 4 |
| **Phase 6: Deployment** | Week 6-7 | 8-16 hours | Phase 5 |
| **Buffer & Iteration** | Week 7-9 | 16-24 hours | All |
| **TOTAL** | **7-9 weeks** | **128-192 hours** | - |

### Resource Allocation

**Primary Developer:** 1 full-time (40 hrs/week)
**Code Reviewer:** 1 part-time (8 hrs/week)
**QA Tester:** 1 part-time (8 hrs/week)

**Total Team:** 2-3 people

### Milestones

| Milestone | Date | Deliverable |
|-----------|------|-------------|
| **M1: Vite Setup** | Week 1 | Working dev server |
| **M2: CSS Complete** | Week 2 | Optimized CSS |
| **M3: Core Modules** | Week 3 | State, services extracted |
| **M4: Features Complete** | Week 4 | All features modularized |
| **M5: Optimization Done** | Week 5 | Code splitting implemented |
| **M6: Tests Passing** | Week 6 | 70%+ coverage |
| **M7: Production Ready** | Week 7 | Deployed to staging |
| **M8: Go Live** | Week 8-9 | Deployed to production |

---

## Success Criteria

### Technical Metrics

✅ **Bundle Size:** Total transfer size < 350 KB (gzipped)
✅ **Load Time:** FCP < 800ms on 4G
✅ **Lighthouse Score:** Performance 95+
✅ **Test Coverage:** Unit tests > 70%
✅ **Build Time:** Production build < 30 seconds
✅ **Dev Server Start:** < 2 seconds cold start

### Functional Requirements

✅ **Feature Parity:** All current features work identically
✅ **Data Migration:** Existing localStorage data imports successfully
✅ **Browser Support:** Chrome, Firefox, Safari, Edge (last 2 versions)
✅ **Responsive Design:** Works on mobile, tablet, desktop
✅ **Accessibility:** WCAG 2.1 AA compliant

### Developer Experience

✅ **Hot Reload:** Changes reflect instantly (< 100ms)
✅ **Error Messages:** Clear, actionable error messages
✅ **Documentation:** Complete setup and contribution guides
✅ **Linting:** All code passes ESLint
✅ **Testing:** Easy to run tests locally

### User Experience

✅ **No Regressions:** Zero bugs compared to current version
✅ **Performance:** Noticeably faster load and interaction
✅ **Reliability:** No console errors in production
✅ **Offline Ready:** (Optional) PWA with offline support

---

## Next Steps

### Immediate Actions (This Week)

1. **Review & Approve Plan**
   - Team review of migration plan
   - Stakeholder approval
   - Timeline confirmation

2. **Create Project Board**
   - Set up GitHub Projects board
   - Create issues for each phase
   - Assign initial tasks

3. **Set Up Development Environment**
   - Clone repo to new branch `vite-migration`
   - Initialize npm project
   - Install Vite

4. **Proof of Concept**
   - Build minimal Vite setup
   - Get one feature working
   - Validate approach

### Month 1 Goals

- ✅ Phase 1 complete: Vite foundation working
- ✅ Phase 2 complete: CSS optimized
- ✅ Phase 3 started: Core modules extracted
- ✅ Team trained on new workflow

### Month 2 Goals

- ✅ Phase 3 complete: All JS modularized
- ✅ Phase 4 complete: Optimization done
- ✅ Phase 5 started: Testing infrastructure
- ✅ Beta testing with internal users

### Month 3 Goals (If Needed)

- ✅ Phase 5 complete: Tests passing
- ✅ Phase 6 complete: Deployed to production
- ✅ Documentation updated
- ✅ Migration complete

---

## Appendix

### A. Alternative Build Tools Considered

| Tool | Pros | Cons | Decision |
|------|------|------|----------|
| **Vite** | Fast, modern, great DX | Newer, less mature ecosystem | ✅ **Selected** |
| **Webpack** | Mature, proven, plugins | Slow, complex config | ❌ Too complex |
| **Parcel** | Zero-config, fast | Less control, smaller ecosystem | ❌ Less flexible |
| **esbuild** | Extremely fast | Low-level, less features | ❌ Too minimal |
| **Rollup** | Great for libraries | Not ideal for apps | ❌ Wrong use case |

**Why Vite?**
- ⚡ Lightning-fast HMR
- 🎯 Optimized for modern browsers
- 🔌 Rich plugin ecosystem
- 📦 Built-in TypeScript support
- 🏗️ Rollup-based production builds
- 🎨 Great developer experience

### B. Breaking Changes

**None expected** - This is a behind-the-scenes migration that should not affect end users.

### C. Migration Checklist

**Pre-Migration:**
- [ ] Backup current codebase
- [ ] Document all current features
- [ ] Set up test environment
- [ ] Create rollback plan

**During Migration:**
- [ ] Phase 1: Vite setup
- [ ] Phase 2: CSS migration
- [ ] Phase 3: JS modularization
- [ ] Phase 4: Optimization
- [ ] Phase 5: Testing
- [ ] Phase 6: Deployment

**Post-Migration:**
- [ ] Update documentation
- [ ] Train team on new workflow
- [ ] Monitor performance metrics
- [ ] Gather user feedback
- [ ] Archive old codebase

### D. References

- [Vite Documentation](https://vitejs.dev/)
- [Vite Plugin Ecosystem](https://vitejs.dev/plugins/)
- [Rollup Options](https://rollupjs.org/configuration-options/)
- [Chart.js Tree Shaking](https://www.chartjs.org/docs/latest/getting-started/integration.html#bundlers-webpack-rollup-etc)
- [PDF.js Integration](https://mozilla.github.io/pdf.js/getting_started/)
- [Vitest Testing](https://vitest.dev/)
- [Playwright E2E](https://playwright.dev/)

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-09 | Claude | Initial migration plan created |

---

**End of Document**
