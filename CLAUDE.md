# CLAUDE.md - IPC Builder

This file provides guidance to Claude Code (or any AI assistant) when working with the IPC Builder (WBS Terminal) codebase.

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

The app uses ES modules (`type="module"`) but the bulk of the application logic is in `src/legacy/app-legacy.js`. This file exports functions to `window.*` for HTML `onclick` handlers.

**Important:** When adding new functions called by HTML onclick attributes, you MUST add them to the global exports at the end of `app-legacy.js`:

```javascript
// At end of src/legacy/app-legacy.js
window.myNewFunction = myNewFunction;
```

### Entry Point Flow

```
index.html
  └── <script type="module" src="/src/main.js">
        ├── Import CSS: ./styles/main.css
        ├── Import npm packages: chart.js, pdfjs-dist, html2pdf.js
        ├── Configure PDF.js worker (CDN)
        ├── Expose libs to window: Chart, pdfjsLib, html2pdf
        └── Import ./legacy/app-legacy.js
              └── Attaches 100+ functions to window.*
```

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

```javascript
projectData = {
    phases: [],           // ["Base", "ESDC", "TSCD"]
    disciplines: [],      // ["Structures", "Civil", "Traffic"]
    packages: [],         // ["Preliminary", "Interim", "Final"]
    budgets: {},          // { "Structures": 150000 }
    claiming: {},         // { "Structures-Preliminary": 30 }
    dates: {},            // { "Structures-Preliminary": { start, end } }
    calculator: {
        totalConstructionCost: 0,
        designFeePercent: 15,
        projectType: 'Highway/Roadway',
        totalDesignFee: 0,
        complexityOverrides: {},
        isCalculated: false,
        manualEdits: {}
    },
    projectScope: '',
    scheduleNotes: '',
    disciplineScopes: {}
}
```

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

1. **Styling** - Edit files in `src/styles/` based on component/feature
2. **Business Logic** - Modify `src/legacy/app-legacy.js`
3. **New Functions** - Add to global exports if used in HTML onclick
4. **New Modals** - Use `modal-base` class pattern
5. **New Features** - Group with section comments, add persistence

### Code Conventions

- **Indentation:** 4 spaces
- **Naming:** 
  - `kebab-case` for CSS classes/IDs
  - `camelCase` for JavaScript
  - `UPPERCASE` for constants
- **Section Markers:** `// ============================================`

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
- Vendor chunks (chart.js, pdfjs-dist)
- Feature chunks (legacy app code)
- Minified CSS with autoprefixer

## Known Limitations

1. Client-side only (localStorage, no server)
2. Single-user (no collaboration)
3. API key in localStorage (security consideration)
4. No undo/redo functionality
5. Legacy monolithic JS file (gradual modularization in progress)

## Future Improvements

- [ ] Complete modularization of `app-legacy.js`
- [ ] Add TypeScript for type safety
- [ ] Add unit tests with Vitest
- [ ] Add E2E tests with Playwright
- [ ] Server-side persistence option
- [ ] Real-time collaboration
- [ ] Undo/redo functionality

