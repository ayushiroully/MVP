# MVP - Technical Design & Architecture

## 🏗️ System Architecture (4-Layer Design)

```
┌─────────────────────┐ ┌─────────────────────────────┐
│ BOUTIQUE OWNER      │───▶│ FRONTEND LAYER              │
│                     │    │ Next.js 15 + TypeScript     │
│ - CSV Upload        │    │ TailwindCSS (Glassmorphic)  │
│ - Mobile-first      │    │ Responsive Dashboard        │
└──────────┬──────────┘    └──────────┬──────────────────┘
           │                          │
           ▼                          ▼
┌─────────────────────┐    ┌─────────────────────────────┐
│ FILE SYSTEM         │    │ API LAYER                   │
│ sales_data.csv      │◄───▶│ Next.js API Routes          │
│ (10K rows max)      │    │ POST /api/forecast          │
└──────────┬──────────┘    └──────────┬──────────────────┘
           │                          │
           ▼                          ▼
           ┌─────────────────────────────┐
           │ PROCESSING LAYER            │
           │ - PapaParse (CSV → JSON)    │
           │ - FestivalMultiplier        │
           │ - SimpleRegressionEngine    │
           │ - BusinessImpactCalculator  │
           └──────────┬──────────────────┘
                      │
                      ▼
           ┌─────────────────────────────┐
           │ RESULTS JSON                │
           │ {trend: "Red Anarkali",     │
           │  demandIncrease: 35, ...}   │
           └─────────────────────────────┘
```

## 🔧 Component Breakdown

### 1. **Frontend Layer** (`src/app/page.tsx`)
Components:
├── HeroSection.tsx (MVP + 3 feature cards)
├── CSVUploadForm.tsx (Drag-drop + progress)
├── ResultsDashboard.tsx (KPI cards + alerts)
└── FestivalCalendar.tsx (Oct 2.8x | Dec 1.5x)

Styling:
- Glassmorphic cards (backdrop-blur-xl)
- Gradient: Rose-400 → Amber-500
- Mobile-first (375px breakpoints)

### 2. **API Layer** (`src/app/api/forecast/route.ts`)
POST /api/forecast
├── Input: FormData (CSV file)
├── Processing: <3 seconds guaranteed
└── Output: JSON forecast response

Error Handling:
- 400: Invalid CSV format
- 413: File > 10MB
- 500: Processing timeout

### 3. **Processing Layer** (`lib/forecast-engine.ts`)
Core Functions:
1. parseCSV(file: File) → SalesRow[]
2. calculateBaseline(history: SalesRow[]) → number
3. applyFestivalMultiplier(month: string) → number
4. generateBusinessImpact(forecast: Forecast) → Impact
5. validateConfidence(forecast: Forecast) → boolean

## 💾 Data Models (TypeScript)

```typescript
interface SalesRow {
  date: string        // "2024-10-15"
  sku: string         // "Red_Anarkali"
  quantity: number    // 140
  price: number       // 2800
  festival?: string   // "Diwali"
}

interface Forecast {
  topTrend: string
  demandIncrease: number      // 35
  confidence: number          // 85
  restockQuantity: number     // 200
  suggestedPrice: number      // 2800
  festivalMultiplier: number  // 2.8
}

interface BusinessImpact {
  revenuePotential: string    // "₹5.6L"
  overstockRisk: string       // "15%"
  accuracyImprovement: string // "+25%"
}
```

## 🎨 Design System

### Color Palette (Ethnic Fusion)
Primary: #E2725B (Terracotta)
Secondary: #0F4C5C (Deep Teal)
Accent: #D4A574 (Gold)
Background: linear-gradient(135deg, #F59E0B → #EF4444)
Glass: rgba(255,255,255,0.2) + backdrop-blur-xl

### Typography
Headings: Playfair Display (Google Fonts)
Body: Inter (system font fallback)
Sizes: 6xl (hero) → xs (labels)

### Key Components (Figma-ready)
1. KPI Card: "85% Accuracy (+25% vs manual)"
2. Trend Alert: "+35% Red Anarkalis"
3. Restock Card: "200 units @ ₹2,800"
4. Festival Multiplier: "Oct: 2.8x Diwali"

## ⚙️ Festival Multiplier Engine

```typescript
const FESTIVAL_MULTIPLIERS: Record<string, number> = {
  'Diwali': 2.8,     // October
  'Wedding': 1.5,    // December
  'Valentine': 2.2,  // February
  'Holi': 1.8,       // March
  'Eid': 2.1,        // Variable
  'Regional': 1.3    // Auto-detect
}

function getMultiplier(month: number, festival: string): number {
  // Logic: Festival > Month pattern > Baseline
  return FESTIVAL_MULTIPLIERS[festival] || 1.0
}
```

## 📊 Sequence Diagram (CSV → Forecast)

```
Boutique Owner    Frontend       API Route      ML Engine
      |               |               |               |
      |---CSV Upload→ |               |               |
      |               |---POST------→ |               |
      |               |               |---parse()→→   |
      |               |               |←-SalesRow--   |
      |               |               |---forecast→   |
      |               |               |←-JSON------   |
      |               |←---JSON------  |               |
      |←---Results--- |               |               |
```

## 🚀 API Contract

```
POST /api/forecast
Content-Type: multipart/form-data

Request:
├── file: sales_data.csv (max 10MB)

Response (200):
{
  "success": true,
  "forecast": { ...Forecast },
  "impact": { ...BusinessImpact }
}

Response (400):
{
  "error": "Invalid CSV format: missing 'date' column"
}
```

## 📱 Responsive Design

Mobile (375px+): 1-column results
Tablet (768px+): 2-column KPI cards
Desktop (1024px+): Full dashboard + charts

Key Breakpoints:
- CSV Upload: Full-width → Compact
- Results: Stacked cards → Grid layout
- Festival Calendar: List → 3-column grid

## 🧪 ML Engine (Simple Regression)

```typescript
// Pseudo-algorithm (85% accuracy target)
function forecastDemand(history: SalesRow[], targetMonth: number): Forecast {
  const baseline = calculateMonthlyAverage(history)
  const multiplier = getFestivalMultiplier(targetMonth)
  const trend = detectTopSKU(history)
  const confidence = calculateConfidence(history.length)
  
  return {
    topTrend: trend,
    demandIncrease: Math.round(multiplier * 100 - 100),
    confidence: Math.min(95, confidence),
    restockQuantity: Math.round(baseline * multiplier),
    suggestedPrice: calculateOptimalPrice(history, trend),
    festivalMultiplier: multiplier
  }
}
```

## 🚀 Deployment Architecture

Vercel (Free Tier):
├── next build (30s)
├── next start (serverless functions)
├── Custom domain: mvp.vercel.app
├── Environment: NODE_ENV=production

Static Export (₹99 hosting):
├── next export
├── Netlify/CDN deployment
├── Works offline (client-side CSV parsing)

## 📈 Performance Targets

- CSV Parsing: 10K rows < 500ms
- ML Processing: <2 seconds total
- API Response: <3 seconds end-to-end
- Bundle Size: <500KB (gzip)
- Lighthouse Score: 95+ (mobile)

## 🎯 Key Design Decisions

1. Offline-first: No APIs = instant demo
2. CSV universal: Every boutique uses Excel
3. Festival rules > complex ML: Week 1 delivery
4. Glassmorphic UI: Matches ethnic luxury aesthetic
5. Mobile-first: 80% boutique owners use phones
6. Serverless: Scale to 10K users Day 1

## 🔍 Validation Strategy

1. Historical Accuracy: Backtest vs 12 months data
2. Festival Simulation: Test Diwali 2.8x multiplier
3. Edge Cases: Empty CSV, malformed data, outliers
4. Performance: Load test 10K row CSVs
5. Mobile Testing: iPhone/Android responsiveness
