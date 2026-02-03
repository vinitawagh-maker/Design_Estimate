# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**WBS Terminal v2.0** is a Vite-powered web application for generating Work Breakdown Structures (WBS) for engineering projects. It provides a terminal-themed interface for project planning, budgeting, and schedule management with AI-powered features.

| | |
|---|---|
| **GitHub** | https://github.com/mjamiv/IPC-Builder |
| **Live URL** | https://mjamiv.github.io/IPC-Builder/ |
| **Build Tool** | Vite 5.x |
| **Node Version** | 18+ |

## Tech Stack

| Category | Technology |
|----------|------------|
| **Build** | Vite 5.x with HMR, PostCSS, Autoprefixer, cssnano |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript (ES Modules) |
| **Charts** | Chart.js 4.x (npm package) |
| **PDF Parsing** | PDF.js 3.11.174 (npm + CDN worker) |
| **PDF Export** | html2pdf.js (npm package) |
| **AI** | OpenAI API (user-provided key) |
| **CI/CD** | GitHub Actions (lint, build, deploy) |

## Quick Start

```bash
# Install dependencies
npm install

# Start dev server (http://localhost:3000)
npm run dev

# Production build
npm run build

# Preview production build
npm run preview

# Build with bundle size analysis
npm run build:analyze

# Run linting
npm run lint

# Fix linting issues automatically
npm run lint:fix

# Format code with Prettier
npm run format

# Lint + build check
npm run check
```

### Test Commands (Configured but not yet implemented)
```bash
npm run test              # Run unit tests with Vitest
npm run test:ui           # Vitest UI
npm run test:coverage     # Coverage report
npm run test:e2e          # Playwright E2E tests
```

## Project Structure

```
IPC-Builder/
├── index.html              # Main HTML entry point
├── package.json            # Dependencies and scripts
├── vite.config.js          # Vite configuration
├── postcss.config.js       # CSS processing
├── .eslintrc.json          # Linting rules
├── .prettierrc.json        # Code formatting
├── CLAUDE.md               # This file
├── README.md               # Project readme
│
├── src/
│   ├── main.js             # Entry point - imports deps, loads legacy code
│   ├── App.js              # Main app controller (scaffolded)
│   ├── core/
│   │   ├── constants.js    # App constants
│   │   └── state.js        # Global state
│   ├── utils/
│   │   ├── format.js       # Formatting helpers
│   │   └── dom.js          # DOM utilities
│   ├── services/
│   │   ├── api/openai.js   # OpenAI integration
│   │   ├── storage/localStorage.js
│   │   └── export/         # CSV, URL exports
│   ├── legacy/
│   │   └── app-legacy.js   # Legacy code (~12,000 lines)
│   └── styles/
│       ├── main.css        # CSS entry (imports all)
│       ├── base/           # variables, reset, typography
│       ├── components/     # buttons, cards, modals, tables
│       ├── features/       # calculator, chat, gantt, rfp
│       └── layout/         # terminal, progress, responsive
│
├── public/data/benchmarking/  # Historical MH data (14 JSON files)
├── docs/                      # Documentation
├── .github/workflows/         # CI/CD pipelines
└── dist/                      # Production build output
```

## Architecture Notes

### ES Modules + Legacy Code

The app uses ES modules (`type="module"`) but the bulk of the application logic is in `src/legacy/app-legacy.js` (~12,000 lines). This file exports 100+ functions to `window.*` for HTML `onclick` handlers.

**Critical Pattern for HTML Event Handlers:**

When adding new functions called by HTML onclick attributes, you MUST add them to the global exports at the end of `app-legacy.js`:

```javascript
// At end of src/legacy/app-legacy.js
window.myNewFunction = myNewFunction;
```

**Why this is required:** The HTML in `index.html` uses inline event handlers like `onclick="myFunction()"`. These require global scope access. ES modules are scoped by default, so explicit window exports are necessary.

**Early Loading Pattern:** Critical functions like `discardRecovery`, `restoreRecovery`, `nextStep`, `prevStep`, `toggleDisc` are immediately exposed as placeholders at the top of `app-legacy.js` (lines 11-16) to prevent "function not yet loaded" errors during page initialization.

### Entry Point Flow

```
index.html
  └── <script type="module" src="/src/main.js">
        ├── Import CSS: ./styles/main.css
        ├── Import npm packages: chart.js, pdfjs-dist, html2pdf.js
        ├── Configure PDF.js worker (CDN jsDelivr v3.11.174)
        ├── Expose libs to window: Chart, pdfjsLib, html2pdf
        ├── Import modular services (state, constants, utils, services)
        ├── Create window.WBS namespace with modular architecture
        └── Import ./legacy/app-legacy.js
              └── Attaches 100+ functions to window.* (gradual migration target)
```

**Modular Architecture (window.WBS):**

The new modular code is available at `window.WBS`:

```javascript
window.WBS = {
    state: { projectData, currentStep },
    constants,
    utils: { ...formatUtils, ...domUtils },
    services: {
        storage: storageService,
        openai: openaiService,
        csv: csvExport,
        url: urlService
    }
};
```

**Migration Strategy:** New code should use the WBS namespace. Legacy code will gradually be refactored into these modules.

### CSS Architecture

Modular CSS following SMACSS/BEM principles:

```
src/styles/main.css (imports in order)
  ├── base/      → variables.css, reset.css, typography.css
  ├── layout/    → terminal.css, progress.css
  ├── components/→ buttons.css, cards.css, modals.css, tables.css...
  ├── features/  → calculator.css, chat.css, gantt.css, rfp.css...
  └── layout/responsive.css (must be last)
```

## Data Model

The `projectData` object is the central data structure (defined in src/legacy/app-legacy.js:19-85):

```javascript
projectData = {
    // Core WBS data
    phases: [],                     // ["Base", "ESDC", "TSCD"]
    disciplines: [],                // ["Structures", "Civil", "Traffic"]
    packages: [],                   // ["Preliminary", "Interim", "Final"]
    budgets: {},                    // { "Structures": 150000 }
    claiming: {},                   // { "Structures-Preliminary": 30 }
    dates: {},                      // { "Structures-Preliminary": { start, end } }

    // Unique identifiers
    disciplineIds: {},              // { "Roadway": "DISC-001", ... }
    packageIds: {},                 // { "Preliminary": "PKG-001", ... }
    activities: {},                 // { "Roadway-Preliminary": { id, description }, ... }

    // Design review steps
    reviewSteps: {},                // { "Roadway-Preliminary": [{ step, start, end }, ...] }
    rfpReviewSteps: [],             // Review steps extracted from RFP

    // Calculator data
    calculator: {
        totalConstructionCost: 0,
        designFeePercent: 15,
        projectType: 'Highway/Roadway',
        totalDesignFee: 0,
        complexityOverrides: {},    // Per-discipline complexity adjustments
        isCalculated: false,
        manualEdits: {}             // Track manual budget overrides
    },

    // RFP-extracted data
    projectScope: '',
    scheduleNotes: '',
    disciplineScopes: {},

    // Chapter 1 - Project Information
    projectInfo: {
        projectName: '',
        projectLocation: '',
        leadDistrict: '',
        partneringDistricts: '',
        kieNonSpPercentage: '',
        technicalProposalDue: '',
        priceProposalDue: '',
        interviewDate: '',
        contractAward: '',
        noticeToProceed: '',
        stipendAmount: '',
        ownerContractType: '',
        kegEntity: '',
        evaluationCriteria: '',
        dbeGoals: ''
    },

    // Chapter 2 - Commercial Terms
    commercialTerms: {
        client: '',
        projectValue: '',
        projectStatus: '',
        waiverConsequentialDamages: '',
        limitationOfLiability: '',
        professionalLiability: '',
        insuranceRequirements: '',
        standardOfCare: '',
        reliedUponInformation: '',
        thirdPartyDelays: '',
        thirdPartyContractorImpacts: '',
        indemnification: ''
    },

    // Chapter 3 - Project Organization
    projectOrganization: ''
}
```

**Storage:** Persisted to localStorage with key `'wbs_project_autosave'` (STORAGE_KEY constant)

## Key Features

### 6-Step Wizard
1. **PHASES** - Project phases with templates
2. **DISCIPLINES** - Engineering discipline selection (18+ options)
3. **PACKAGES** - Deliverable milestones
4. **BUDGET** - Cost estimator with industry benchmarks
5. **CLAIMING** - Distribution schemes (linear, front/back-loaded, bell curve)
6. **SCHEDULE** - AI-powered schedule generation

### AI Features (requires OpenAI API key)
- **Chat Assistant** - Natural language WBS editing
- **Schedule Generation** - AI-optimized timelines
- **Insights Panel** - Risk scoring, forecasting
- **RFP Wizard** - PDF import with quantity extraction

### Export Options
- **Design Fee Book** - Professional PDF with all project data (B/W, print-ready)
- Import from CSV
- Shareable URL

## Key Functions

### Navigation
- `goToStep(step)` - Jump to step
- `nextStep()` / `prevStep()` - Navigate
- `showStep(step)` - Display step content

### Persistence
- `saveToLocalStorage()` - Auto-save
- `loadFromLocalStorage()` - Load saved
- `showRecoveryModal()` - Recovery prompt

### Cost Estimator
- `initCalculator()` - Initialize
- `calculateBudgets()` - Main calculation
- `updateIndustryIndicators()` - Variance display

### MH Estimator
- `loadBenchmarkData()` - Load JSON files
- `estimateMH()` - Calculate man-hours
- `applyMHEstimate()` - Apply to budget

### AI Features
- `toggleChat()` - Open/close chat
- `sendMessage()` - Send to AI
- `generateAISchedule()` - AI schedule
- `openRfpWizard()` - RFP import

### Export
- `generateDesignFeeBook()` - Professional PDF report (Design Fee Book)
- `shareProjectUrl()` - URL sharing
- `importData()` - Import from CSV

## Development Guidelines

### Making Changes

1. **Styling** - Edit files in `src/styles/` based on component/feature (CSS is modular and organized by base/components/features/layout)
2. **Business Logic** - Modify `src/legacy/app-legacy.js` (legacy code being gradually refactored)
3. **New Functions** - If called from HTML onclick, MUST export to window.* in app-legacy.js
4. **New Modular Code** - Add to appropriate module in `src/core/`, `src/utils/`, or `src/services/` and expose via window.WBS
5. **New Modals** - Use `modal-base` class pattern for consistency
6. **New Features** - Group with section comments (`// ============================================`), add persistence to localStorage

### Path Aliases (vite.config.js)

Available import aliases:
```javascript
import { foo } from '@/bar';           // src/
import styles from '@styles/foo.css';   // src/styles/
import Comp from '@components/Foo';     // src/ui/components/
import util from '@utils/format';       // src/utils/
import svc from '@services/api/openai'; // src/services/
import { constants } from '@core/constants'; // src/core/
```

### Code Conventions

- **Indentation:** 4 spaces (enforced by .prettierrc.json)
- **Naming:**
  - `kebab-case` for CSS classes/IDs
  - `camelCase` for JavaScript functions and variables
  - `UPPERCASE` for constants
- **Section Markers:** `// ============================================` (section dividers in app-legacy.js)
- **Linting:** ESLint configured (.eslintrc.json) - excludes `src/legacy/*` from strict checks
- **Formatting:** Prettier configured (.prettierrc.json) with ignores in .prettierignore

### Testing Checklist

- [ ] All 6 wizard steps navigate correctly
- [ ] Budget totals calculate properly
- [ ] Claiming percentages validate to 100%
- [ ] Dates calculate duration correctly
- [ ] WBS table generates with correct numbering
- [ ] Chart displays and filters work
- [ ] CSV/PDF exports download correctly
- [ ] Responsive breakpoints work (768px, 480px)
- [ ] Auto-save triggers on changes
- [ ] Recovery modal appears with saved data
- [ ] AI features work with valid API key

## Design System

### Colors

| Token | Value | Usage |
|-------|-------|-------|
| `--bg-body` | `#0a0a0a` | Body background |
| `--bg-terminal` | `#0d0d0d` | Terminal background |
| `--bg-card` | `#1a1a1a` | Card backgrounds |
| `--color-primary` | `#ffd700` | Gold accent |
| `--color-success` | `#00ff00` | Success states |
| `--color-error` | `#ff4444` | Error states |

### Typography

- **Font:** JetBrains Mono (monospace)
- **Gold on dark** for terminal aesthetic

## Benchmarking Data

Located in `public/data/benchmarking/`:

| File | Metric |
|------|--------|
| `benchmarking-bridges.json` | Bridge deck area (SF) |
| `benchmarking-roadway.json` | Alignment length (LF) |
| `benchmarking-drainage.json` | Project area (AC) |
| `benchmarking-traffic.json` | Alignment length (LF) |
| `benchmarking-utilities.json` | Relocation count (EA) |
| ... | (14 files total) |

**JSON Structure:**
```json
{
  "discipline": "Bridges",
  "eqty_metric": { "name": "Bridge Deck Area", "uom": "SF" },
  "projects": [
    { "project": "IH 820", "fct_mhrs": 12345, "eqty": 100, "production_mhrs_per_ea": 123.45 }
  ]
}
```

## CI/CD

### GitHub Actions

| Workflow | Trigger | Actions |
|----------|---------|---------|
| `ci.yml` | Push, PR | Lint, Build |
| `deploy.yml` | Push to main | Build, Deploy to GH Pages |

### Build Output

Production build creates optimized bundles in `dist/`:
- **Vendor chunks:** `vendor-charts.js` (Chart.js), `vendor-pdf.js` (pdfjs-dist, html2pdf.js)
- **Feature chunks:** Legacy app code and application logic
- **Minified CSS:** Autoprefixed and optimized with cssnano
- **Asset organization:** `assets/js/`, `assets/styles/`, `assets/images/`, `assets/fonts/`

### Base Path Configuration (IMPORTANT)

The Vite config uses a **conditional base path** (vite.config.js:9):
- **Development:** `base: '/'`
- **Production:** `base: '/vinitaschangestomikesorig/'` (GitHub Pages subpath)

**When deploying to a different subdomain or custom domain, you MUST update this base path in vite.config.js.**

## Known Limitations

1. Client-side only (localStorage, no server)
2. Single-user (no collaboration)
3. API key in localStorage (security consideration)
4. No undo/redo functionality
5. Legacy monolithic JS file (gradual modularization in progress)

## Troubleshooting

### Common Issues

**"Function not yet loaded" errors on page load:**
- Ensure critical functions are exposed early in app-legacy.js (lines 11-16)
- Check that window.functionName assignments exist at bottom of app-legacy.js

**PDF.js worker errors:**
- Worker URL is set to jsDelivr CDN (v3.11.174) in main.js:17
- Standard fonts URL configured at main.js:21
- If offline, PDF features will fail (no local worker fallback)

**Build fails or assets 404 in production:**
- Check `base` path in vite.config.js:9 matches deployment URL
- GitHub Pages requires `/repo-name/` base path for project sites

**Styles not loading or HMR not working:**
- Ensure CSS imports maintain order (responsive.css must be last)
- Check that main.css imports are in correct sequence (base → layout → components → features → layout/responsive)

**localStorage data loss:**
- Data is stored client-side only, cleared on browser cache clear
- Use "Save Project" feature for persistent backups
- Recovery modal appears if unsaved changes detected

## Future Improvements

- [ ] Complete modularization of `app-legacy.js`
- [ ] Add TypeScript for type safety
- [ ] Add unit tests with Vitest (configured but not implemented)
- [ ] Add E2E tests with Playwright (configured but not implemented)
- [ ] Server-side persistence option
- [ ] Real-time collaboration
- [ ] Undo/redo functionality

