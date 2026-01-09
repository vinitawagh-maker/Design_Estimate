# IPC Builder Reorganization Plan

## Executive Summary

This document outlines a comprehensive plan to reorganize the IPC Builder (WBS Terminal) codebase from a single 14,324-line `index.html` file into a modular, maintainable structure suitable for continued development.

---

## Part 1: Current State Analysis

### File Statistics
- **Total Lines:** 14,324
- **CSS:** ~2,900 lines (lines 11-2900)
- **HTML:** ~900 lines (lines 2900-3795)
- **JavaScript:** ~10,500 lines (lines 3797-14324)

### Current Code Sections (JavaScript)

| Section | Lines | Description |
|---------|-------|-------------|
| Data/State Objects | 3797-3823 | Global state (projectData, currentStep, chart) |
| Persistence System | 3824-4108 | localStorage autosave/recovery (~285 lines) |
| Multi-Project Manager | 4109-4453 | Save/load/duplicate projects (~345 lines) |
| Reports Panel | 4115-4155 | Report modal functions (~40 lines) |
| Core Data Constants | 4456-4533 | allDisciplines, exampleBudgets, claimingSchemes (~78 lines) |
| Project Templates | 4534-4751 | Template definitions and functions (~218 lines) |
| Cost Estimator Constants | 4752-4877 | complexityMap, industryDistribution, benchmarks (~126 lines) |
| MH Estimator Config | 4878-5329 | DISCIPLINE_CONFIG, HISTORICAL_BENCHMARKS (~452 lines) |
| MH Estimation Functions | 5330-5631 | Core MH calculation logic (~302 lines) |
| MH Estimator UI | 5632-6208 | MH estimator interface functions (~577 lines) |
| Core Wizard Functions | 6209-7013 | Navigation, disciplines, phases, packages (~805 lines) |
| Claiming Presets | 7014-7228 | Scheme distribution functions (~215 lines) |
| Schedule/Dates | 7229-7313 | Date table building (~85 lines) |
| WBS Generation | 7314-7541 | Core WBS table generation (~228 lines) |
| Chart/Filters | 7542-7988 | Chart.js integration (~447 lines) |
| WBS Editing Mode | 7989-9913 | Inline editing system (~1,925 lines) |
| Gantt Chart | 9914-10194 | Gantt visualization (~281 lines) |
| AI Chat Assistant | 10195-10823 | Chat interface (~629 lines) |
| AI Tool Calling | 10824-11383 | NLP tool execution (~560 lines) |
| AI Insights | 11384-11624 | Predictive analytics (~241 lines) |
| AI Schedule Gen | 11625-11983 | AI-powered scheduling (~359 lines) |
| RFP Wizard | 11984-13503 | PDF import/analysis (~1,520 lines) |
| Export Functions | Scattered | CSV, PDF export (~300 lines) |

### Current CSS Sections

| Section | Approximate Lines | Description |
|---------|-------------------|-------------|
| Base/Core | ~250 | Terminal, progress bar, inputs, buttons |
| Components | ~400 | Discipline grid, tables, KPIs, quick tags |
| Template Selector | ~80 | Template cards and grid |
| Cost Calculator | ~150 | Budget calculator styling |
| MH Estimator | ~350 | MH benchmark estimator |
| Claiming Presets | ~150 | Preset panel and preview |
| AI Chat | ~300 | Chat panel, messages, FAB |
| AI Insights | ~200 | Insights cards and grid |
| Gantt Chart | ~280 | Timeline and bars |
| WBS Editing | ~200 | Edit toolbar and inline styles |
| RFP Wizard | ~400 | Modal stages and quantities |
| Modals (shared) | ~200 | Recovery, reports, projects, API key |
| Responsive | ~100 | Media queries |

### Identified Dependencies

**Global State Objects:**
- `projectData` - Used in 398 places across all modules
- `currentStep` - Navigation state
- `chart` - Chart.js instance
- `chatState` - AI chat state
- `rfpState` - RFP wizard state
- `mhEstimateState` - MH estimator state
- `ganttState` - Gantt chart state
- `wbsEditMode` - Edit mode flag

**Shared Utility Functions:**
- `formatCurrency()` - Used in 30+ places
- `triggerAutosave()` - Used in 21 places
- `formatTimestamp()` - Used in persistence
- `calculateTotalBudget()` - Used in 15+ places
- `updateStatus()` - UI status updates

---

## Part 2: Proposed Architecture

### Option A: Zero-Build with ES6 Modules (Recommended for Simplicity)

Maintain the zero-build philosophy using native ES6 modules. Works in all modern browsers.

```
ipc-builder/
├── index.html              # Main HTML (slimmed down)
├── css/
│   ├── main.css            # Core styles
│   ├── components/
│   │   ├── terminal.css
│   │   ├── wizard.css
│   │   ├── tables.css
│   │   ├── modals.css
│   │   ├── buttons.css
│   │   └── cards.css
│   └── features/
│       ├── calculator.css
│       ├── mh-estimator.css
│       ├── chat.css
│       ├── insights.css
│       ├── gantt.css
│       ├── rfp-wizard.css
│       └── wbs-editor.css
├── js/
│   ├── app.js              # Main entry point
│   ├── state/
│   │   ├── store.js        # Central state management
│   │   └── persistence.js  # localStorage handling
│   ├── config/
│   │   ├── disciplines.js  # Discipline definitions
│   │   ├── templates.js    # Project templates
│   │   ├── benchmarks.js   # MH benchmarks data
│   │   └── constants.js    # App constants
│   ├── utils/
│   │   ├── formatting.js   # formatCurrency, formatMH, etc.
│   │   ├── validation.js   # Input validation
│   │   └── dom.js          # DOM utilities
│   ├── core/
│   │   ├── wizard.js       # Step navigation
│   │   ├── disciplines.js  # Discipline management
│   │   ├── phases.js       # Phase management
│   │   └── packages.js     # Package management
│   ├── features/
│   │   ├── budget/
│   │   │   ├── calculator.js
│   │   │   └── mh-estimator.js
│   │   ├── claiming/
│   │   │   ├── table.js
│   │   │   └── presets.js
│   │   ├── schedule/
│   │   │   ├── dates.js
│   │   │   └── ai-schedule.js
│   │   ├── wbs/
│   │   │   ├── generator.js
│   │   │   ├── table.js
│   │   │   └── editor.js
│   │   ├── visualization/
│   │   │   ├── chart.js
│   │   │   └── gantt.js
│   │   ├── ai/
│   │   │   ├── chat.js
│   │   │   ├── tools.js
│   │   │   └── insights.js
│   │   ├── rfp/
│   │   │   ├── wizard.js
│   │   │   ├── parser.js
│   │   │   └── analyzer.js
│   │   └── export/
│   │       ├── csv.js
│   │       ├── pdf.js
│   │       └── share.js
│   └── ui/
│       ├── modals.js       # Modal management
│       ├── projects.js     # Project manager
│       └── reports.js      # Reports panel
└── CLAUDE.md
```

**Pros:**
- No build step required
- Simple deployment (just copy files)
- Native browser support (Chrome, Firefox, Safari, Edge)
- Easy to debug (source maps not needed)

**Cons:**
- Many HTTP requests on load (can be mitigated with HTTP/2)
- No minification/bundling
- No TypeScript support

### Option B: Vite Build System (Recommended for Scale)

Use Vite for a modern development experience with minimal configuration.

```
ipc-builder/
├── index.html
├── package.json
├── vite.config.js
├── src/
│   ├── main.js             # Entry point
│   ├── styles/
│   │   ├── main.css
│   │   └── ... (same as Option A)
│   ├── lib/
│   │   └── ... (same as Option A js/)
│   └── assets/
└── dist/                   # Built output (single bundle)
```

**Pros:**
- Fast hot module replacement (HMR) during development
- Automatic bundling/minification for production
- TypeScript support if desired later
- Tree-shaking removes unused code
- Single file output for production

**Cons:**
- Requires Node.js for development
- Build step before deployment
- Slightly more complex setup

---

## Part 3: Recommended Approach - Hybrid Strategy

**Phase 1: Immediate - CSS Extraction (No Build Required)**
**Phase 2: Short-term - JavaScript Modularization (ES6 Modules)**
**Phase 3: Optional - Build Tool Integration**

This approach allows incremental progress while maintaining a working application at each step.

---

## Part 4: Detailed Implementation Plan

### Phase 1: CSS Extraction (~2,900 lines reduction)

**Goal:** Extract all CSS into organized external files.

**Step 1.1: Create CSS Directory Structure**
```
css/
├── main.css                 # Imports all other CSS
├── base/
│   ├── reset.css           # Base resets
│   ├── variables.css       # CSS custom properties (colors, sizes)
│   ├── typography.css      # Font styles
│   └── utilities.css       # Utility classes (.hidden, etc.)
├── layout/
│   ├── terminal.css        # Terminal window
│   ├── progress.css        # Progress bar
│   └── responsive.css      # Media queries
├── components/
│   ├── buttons.css
│   ├── inputs.css
│   ├── tables.css
│   ├── cards.css
│   ├── modals.css
│   ├── tooltips.css
│   └── tags.css
└── features/
    ├── wizard.css          # Step content
    ├── calculator.css      # Budget calculator
    ├── mh-estimator.css    # MH benchmark
    ├── claiming.css        # Claiming presets
    ├── schedule.css        # AI schedule
    ├── wbs.css             # WBS table and editor
    ├── chart.css           # Performance chart
    ├── gantt.css           # Gantt chart
    ├── chat.css            # AI chat assistant
    ├── insights.css        # AI insights
    └── rfp.css             # RFP wizard
```

**Step 1.2: Create CSS Variables File**
```css
/* css/base/variables.css */
:root {
  /* Colors */
  --color-bg-body: #0a0a0a;
  --color-bg-terminal: #0d0d0d;
  --color-bg-card: #1a1a1a;
  --color-primary: #ffd700;
  --color-text-primary: #ffd700;
  --color-text-white: #fff;
  --color-text-muted: #888;
  --color-text-dimmed: #666;
  --color-border: #333;
  --color-border-light: #444;
  --color-success: #00ff00;
  --color-error: #ff4444;
  --color-warning: #ff8800;

  /* Typography */
  --font-mono: 'JetBrains Mono', monospace;
  --font-size-xs: 10px;
  --font-size-sm: 11px;
  --font-size-base: 13px;
  --font-size-md: 14px;
  --font-size-lg: 16px;
  --font-size-xl: 18px;
  --font-size-2xl: 24px;

  /* Spacing */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 12px;
  --spacing-lg: 16px;
  --spacing-xl: 20px;
  --spacing-2xl: 24px;

  /* Borders */
  --border-radius-sm: 4px;
  --border-radius-md: 6px;
  --border-radius-lg: 8px;
  --border-radius-xl: 12px;

  /* Transitions */
  --transition-fast: 0.2s;
  --transition-normal: 0.3s;
}
```

**Estimated Reduction:** ~2,900 lines from index.html

---

### Phase 2: JavaScript Modularization (~10,500 lines)

**Step 2.1: Create State Management Module**

```javascript
// js/state/store.js
export const store = {
  projectData: {
    phases: [],
    disciplines: [],
    packages: [],
    budgets: {},
    claiming: {},
    dates: {},
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
  },

  ui: {
    currentStep: 1,
    wbsEditMode: false,
    chart: null
  },

  // Subscribe to state changes
  listeners: new Set(),

  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  },

  notify() {
    this.listeners.forEach(fn => fn(this.projectData));
  },

  update(path, value) {
    // Deep update helper
    const keys = path.split('.');
    let obj = this.projectData;
    for (let i = 0; i < keys.length - 1; i++) {
      obj = obj[keys[i]];
    }
    obj[keys[keys.length - 1]] = value;
    this.notify();
  }
};

// Convenience getters
export const getProjectData = () => store.projectData;
export const getCurrentStep = () => store.ui.currentStep;
export const setCurrentStep = (step) => { store.ui.currentStep = step; };
```

**Step 2.2: Create Utility Modules**

```javascript
// js/utils/formatting.js
export function formatCurrency(amount) {
  return '$' + Math.round(amount || 0).toLocaleString('en-US');
}

export function formatMH(mh) {
  if (mh >= 1000) {
    return (mh / 1000).toFixed(1) + 'K';
  }
  return Math.round(mh).toLocaleString();
}

export function formatTimestamp(ts) {
  const date = new Date(ts);
  const now = new Date();
  const diffMs = now - date;
  const diffMins = Math.floor(diffMs / 60000);
  const diffHours = Math.floor(diffMs / 3600000);
  const diffDays = Math.floor(diffMs / 86400000);

  if (diffMins < 1) return 'Just now';
  if (diffMins < 60) return `${diffMins} minute${diffMins > 1 ? 's' : ''} ago`;
  if (diffHours < 24) return `${diffHours} hour${diffHours > 1 ? 's' : ''} ago`;
  if (diffDays < 7) return `${diffDays} day${diffDays > 1 ? 's' : ''} ago`;
  return date.toLocaleDateString();
}

export function formatRate(rate, unit) {
  if (rate >= 1) {
    return `${rate.toFixed(2)} MH/${unit}`;
  }
  return `${rate.toFixed(3)} MH/${unit}`;
}

export function formatFileSize(bytes) {
  if (bytes < 1024) return bytes + ' B';
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB';
}
```

**Step 2.3: Create Configuration Modules**

```javascript
// js/config/disciplines.js
export const allDisciplines = [
  { name: 'Structures', selected: true },
  { name: 'Civil', selected: false },
  { name: 'Drainage', selected: false },
  // ... rest of disciplines
];

export const exampleBudgets = {
  'Structures': 450000,
  'Design Management': 180000,
  // ... rest of budgets
};

// js/config/templates.js
export const projectTemplates = {
  highway: {
    id: 'highway',
    name: 'Highway Reconstruction',
    icon: '🛣️',
    // ... rest of template
  },
  // ... rest of templates
};

// js/config/benchmarks.js
export const DISCIPLINE_CONFIG = { /* ... */ };
export const HISTORICAL_BENCHMARKS = { /* ... */ };
export const DIGITAL_DELIVERY_MATRIX = { /* ... */ };
```

**Step 2.4: Create Feature Modules**

```javascript
// js/features/budget/calculator.js
import { store, getProjectData } from '../../state/store.js';
import { formatCurrency } from '../../utils/formatting.js';
import { industryDistribution, projectComplexityMap } from '../../config/constants.js';
import { triggerAutosave } from '../../state/persistence.js';

export function initCalculator() { /* ... */ }
export function calculateBudgets() { /* ... */ }
export function updateIndustryIndicators() { /* ... */ }
export function updateCalculatorTotal() { /* ... */ }

// js/features/ai/chat.js
import { store, getProjectData } from '../../state/store.js';
import { formatCurrency } from '../../utils/formatting.js';
import { executeToolCall } from './tools.js';

export const chatState = {
  isOpen: false,
  messages: [],
  isLoading: false,
  isDragging: false,
  dragOffset: { x: 0, y: 0 }
};

export function toggleChat() { /* ... */ }
export function sendMessage() { /* ... */ }
export function addMessage(role, content) { /* ... */ }
```

**Step 2.5: Create Main Entry Point**

```javascript
// js/app.js
import { store } from './state/store.js';
import { initPersistence, checkForSavedData } from './state/persistence.js';
import { initWizard } from './core/wizard.js';
import { initDisciplines } from './core/disciplines.js';
import { initCalculator } from './features/budget/calculator.js';
import { initMHEstimator } from './features/budget/mh-estimator.js';
import { initChat } from './features/ai/chat.js';

// Initialize application
document.addEventListener('DOMContentLoaded', () => {
  initPersistence();
  initWizard();
  initDisciplines();
  initCalculator();
  initMHEstimator();
  initChat();

  // Check for saved data after brief delay
  setTimeout(checkForSavedData, 100);

  console.log('WBS Terminal v1.0 initialized');
});

// Export for global access (backwards compatibility)
window.WBSTerminal = {
  store,
  // Expose functions that need to be called from HTML onclick handlers
};
```

---

### Phase 3: HTML Cleanup (~900 lines)

**Step 3.1: Slim Down index.html**

After CSS and JS extraction, index.html will contain:
- DOCTYPE and head (meta, links, script imports)
- HTML structure/markup only
- No inline styles
- No inline scripts (except module imports)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>WBS Terminal v1.0</title>

  <!-- External Libraries -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>

  <!-- Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet">

  <!-- Application Styles -->
  <link rel="stylesheet" href="css/main.css">
</head>
<body>
  <!-- Application HTML markup -->
  ...

  <!-- Application Scripts -->
  <script type="module" src="js/app.js"></script>
</body>
</html>
```

---

## Part 5: Migration Strategy

### Approach: Incremental Migration with Feature Flags

To ensure the application remains functional throughout the migration:

1. **Create parallel structure** - New files alongside existing code
2. **Migrate one module at a time** - Start with utilities, then config, then features
3. **Test after each migration** - Run through testing checklist
4. **Keep fallbacks** - Original code can remain until new code is verified

### Migration Order (Recommended)

| Order | Module | Lines | Risk | Dependencies |
|-------|--------|-------|------|--------------|
| 1 | CSS Variables | ~100 | Low | None |
| 2 | CSS Base | ~250 | Low | Variables |
| 3 | CSS Components | ~400 | Low | Base |
| 4 | CSS Features | ~2,100 | Medium | Components |
| 5 | JS Utilities | ~150 | Low | None |
| 6 | JS Config/Data | ~900 | Low | None |
| 7 | JS State/Persistence | ~600 | Medium | Utilities |
| 8 | JS Core Wizard | ~800 | Medium | State |
| 9 | JS Budget/Calculator | ~900 | Medium | State, Config |
| 10 | JS MH Estimator | ~1,300 | Medium | State, Config |
| 11 | JS Claiming | ~300 | Low | State |
| 12 | JS Schedule | ~450 | Low | State |
| 13 | JS WBS Generation | ~250 | Medium | State, All Core |
| 14 | JS WBS Editor | ~1,900 | High | WBS Gen, State |
| 15 | JS Chart | ~450 | Medium | State |
| 16 | JS Gantt | ~280 | Medium | State |
| 17 | JS AI Chat | ~1,200 | High | State, All Features |
| 18 | JS AI Tools | ~560 | High | Chat, State |
| 19 | JS AI Insights | ~240 | Medium | State |
| 20 | JS AI Schedule | ~360 | Medium | State, AI |
| 21 | JS RFP Wizard | ~1,500 | High | State, MH Estimator |
| 22 | JS Export | ~300 | Low | State |
| 23 | JS Projects/Reports | ~400 | Medium | State, Persistence |

---

## Part 6: Final File Structure

After complete migration:

```
ipc-builder/
├── index.html              # ~200 lines (markup only)
├── css/
│   ├── main.css            # Imports all CSS (~20 lines)
│   ├── base/
│   │   ├── variables.css   # ~80 lines
│   │   ├── reset.css       # ~30 lines
│   │   ├── typography.css  # ~40 lines
│   │   └── utilities.css   # ~50 lines
│   ├── layout/
│   │   ├── terminal.css    # ~100 lines
│   │   ├── progress.css    # ~50 lines
│   │   └── responsive.css  # ~100 lines
│   ├── components/
│   │   ├── buttons.css     # ~80 lines
│   │   ├── inputs.css      # ~60 lines
│   │   ├── tables.css      # ~120 lines
│   │   ├── cards.css       # ~100 lines
│   │   ├── modals.css      # ~150 lines
│   │   ├── tooltips.css    # ~60 lines
│   │   └── tags.css        # ~40 lines
│   └── features/
│       ├── wizard.css      # ~100 lines
│       ├── calculator.css  # ~150 lines
│       ├── mh-estimator.css # ~350 lines
│       ├── claiming.css    # ~150 lines
│       ├── schedule.css    # ~80 lines
│       ├── wbs.css         # ~300 lines
│       ├── chart.css       # ~60 lines
│       ├── gantt.css       # ~280 lines
│       ├── chat.css        # ~300 lines
│       ├── insights.css    # ~200 lines
│       └── rfp.css         # ~400 lines
├── js/
│   ├── app.js              # ~50 lines (entry point)
│   ├── state/
│   │   ├── store.js        # ~100 lines
│   │   └── persistence.js  # ~300 lines
│   ├── config/
│   │   ├── disciplines.js  # ~80 lines
│   │   ├── templates.js    # ~200 lines
│   │   ├── benchmarks.js   # ~500 lines
│   │   └── constants.js    # ~200 lines
│   ├── utils/
│   │   ├── formatting.js   # ~80 lines
│   │   ├── validation.js   # ~60 lines
│   │   └── dom.js          # ~50 lines
│   ├── core/
│   │   ├── wizard.js       # ~300 lines
│   │   ├── disciplines.js  # ~150 lines
│   │   ├── phases.js       # ~80 lines
│   │   └── packages.js     # ~80 lines
│   ├── features/
│   │   ├── budget/
│   │   │   ├── calculator.js   # ~400 lines
│   │   │   └── mh-estimator.js # ~900 lines
│   │   ├── claiming/
│   │   │   ├── table.js        # ~150 lines
│   │   │   └── presets.js      # ~200 lines
│   │   ├── schedule/
│   │   │   ├── dates.js        # ~150 lines
│   │   │   └── ai-schedule.js  # ~350 lines
│   │   ├── wbs/
│   │   │   ├── generator.js    # ~250 lines
│   │   │   ├── table.js        # ~400 lines
│   │   │   └── editor.js       # ~1,200 lines
│   │   ├── visualization/
│   │   │   ├── chart.js        # ~450 lines
│   │   │   └── gantt.js        # ~280 lines
│   │   ├── ai/
│   │   │   ├── chat.js         # ~600 lines
│   │   │   ├── tools.js        # ~560 lines
│   │   │   └── insights.js     # ~240 lines
│   │   ├── rfp/
│   │   │   ├── wizard.js       # ~600 lines
│   │   │   ├── parser.js       # ~400 lines
│   │   │   └── analyzer.js     # ~500 lines
│   │   └── export/
│   │       ├── csv.js          # ~150 lines
│   │       ├── pdf.js          # ~500 lines
│   │       └── share.js        # ~50 lines
│   └── ui/
│       ├── modals.js           # ~100 lines
│       ├── projects.js         # ~250 lines
│       └── reports.js          # ~100 lines
├── CLAUDE.md
└── README.md
```

**Total Files:** ~45 files
**Average File Size:** ~150-300 lines
**Maximum File Size:** ~1,200 lines (wbs/editor.js)

---

## Part 7: Benefits Summary

### Before Reorganization
- **1 file** with 14,324 lines
- Difficult to navigate and maintain
- No code reusability
- Hard to test individual components
- Risk of merge conflicts in team settings

### After Reorganization
- **~45 files** with average ~200 lines each
- Clear separation of concerns
- Reusable modules
- Easier to test and debug
- Better collaboration potential
- CSS variables for consistent theming
- Modular JavaScript with clear dependencies

### Estimated Timeline

| Phase | Task | Estimated Time |
|-------|------|----------------|
| 1 | CSS Extraction | 2-3 hours |
| 2 | JS Utilities & Config | 1-2 hours |
| 3 | JS State Management | 2-3 hours |
| 4 | JS Core Wizard | 2-3 hours |
| 5 | JS Budget Features | 3-4 hours |
| 6 | JS Other Features | 4-6 hours |
| 7 | JS AI Features | 3-4 hours |
| 8 | JS RFP Wizard | 2-3 hours |
| 9 | Testing & Fixes | 2-3 hours |
| **Total** | | **~24-32 hours** |

---

## Part 8: Recommendations

### Immediate Actions
1. **Start with CSS extraction** - Lowest risk, highest visual impact
2. **Create CSS variables first** - Establishes theming foundation
3. **Migrate utilities next** - No dependencies, high reuse

### Development Environment
1. Use a local server (e.g., `python3 -m http.server 8000`)
2. Keep browser dev tools open for ES6 module debugging
3. Test in multiple browsers after each migration phase

### Future Considerations
1. **TypeScript** - Consider adding for larger features
2. **Vite** - Adopt for better DX if team grows
3. **Testing** - Add Jest/Vitest after modularization
4. **CI/CD** - GitHub Actions for automated testing

---

## Appendix: Quick Reference

### Module Import Pattern
```javascript
// Feature module
import { store, getProjectData } from '../../state/store.js';
import { formatCurrency } from '../../utils/formatting.js';
import { triggerAutosave } from '../../state/persistence.js';

export function myFeatureFunction() {
  const data = getProjectData();
  // ... implementation
  triggerAutosave();
}
```

### CSS Import Pattern
```css
/* css/main.css */
@import 'base/variables.css';
@import 'base/reset.css';
@import 'base/typography.css';
@import 'base/utilities.css';

@import 'layout/terminal.css';
@import 'layout/progress.css';
@import 'layout/responsive.css';

@import 'components/buttons.css';
/* ... */

@import 'features/wizard.css';
/* ... */
```

### HTML onclick Handler Pattern
```html
<!-- Instead of inline functions, use module exports -->
<button onclick="WBSTerminal.nextStep()">NEXT</button>
```

```javascript
// In app.js
window.WBSTerminal = {
  nextStep: () => import('./core/wizard.js').then(m => m.nextStep()),
  // ... other handlers
};
```

---

*This plan was generated based on analysis of the current codebase structure and best practices for JavaScript application architecture.*
