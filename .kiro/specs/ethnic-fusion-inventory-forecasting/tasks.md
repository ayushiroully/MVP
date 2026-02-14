# Implementation Plan: Ethnic Fusion Inventory Forecasting

## Overview

This implementation plan breaks down the ethnic fusion inventory forecasting MVP into discrete, incremental coding tasks. The approach follows a bottom-up strategy: starting with core data models and processing logic, then building the API layer, and finally implementing the frontend dashboard. Each task builds on previous work, ensuring no orphaned code and continuous integration.

## Tasks

- [ ] 1. Set up project structure and dependencies
  - Initialize Next.js 15 project with TypeScript and Tailwind CSS
  - Install required dependencies: PapaParse, date-fns, and testing libraries
  - Configure TypeScript with strict mode and path aliases
  - Set up project directory structure: `lib/`, `src/app/`, `src/components/`
  - _Requirements: Technical Requirements (Tech Stack)_

- [ ] 2. Implement core data models and types
  - [ ] 2.1 Create TypeScript interfaces for data models
    - Define `SalesRow`, `Forecast`, `BusinessImpact`, and `FestivalMultiplier` interfaces
    - Add validation types for CSV column requirements
    - Create error types for processing failures
    - _Requirements: Feature 1 (CSV Sales Forecasting)_
  
  - [ ]* 2.2 Write unit tests for type validation
    - Test valid and invalid SalesRow objects
    - Test edge cases for date formats and numeric values
    - _Requirements: Feature 1 (CSV Sales Forecasting)_

- [ ] 3. Implement Festival Multiplier Engine
  - [ ] 3.1 Create festival multiplier configuration and lookup logic
    - Implement `FESTIVAL_MULTIPLIERS` constant with all festival values
    - Write `getMultiplier(month: number, festival: string)` function
    - Add month-to-festival mapping logic for automatic detection
    - _Requirements: Feature 2 (Festival Multiplier Engine)_
  
  - [ ]* 3.2 Write property test for festival multiplier consistency
    - **Property 1: Festival multiplier bounds**
    - **Validates: Requirements Feature 2**
    - Test that all multipliers are between 1.0 and 3.0
    - Test that Diwali always returns 2.8x, Wedding 1.5x, Valentine 2.2x
  
  - [ ]* 3.3 Write unit tests for edge cases
    - Test unknown festival fallback to 1.0
    - Test month-only detection without festival name
    - Test regional festival auto-detection
    - _Requirements: Feature 2 (Festival Multiplier Engine)_

- [ ] 4. Implement CSV parsing and validation
  - [ ] 4.1 Create CSV parser with PapaParse integration
    - Write `parseCSV(file: File)` function returning `SalesRow[]`
    - Implement column validation (date, sku, quantity, price, festival)
    - Add error handling for malformed CSV files
    - Ensure processing completes in <500ms for 10K rows
    - _Requirements: Feature 1 (CSV Sales Forecasting), Performance (<3 seconds)_
  
  - [ ]* 4.2 Write property test for CSV parsing
    - **Property 2: CSV parsing preserves data integrity**
    - **Validates: Requirements Feature 1**
    - Test that parsed row count matches input row count
    - Test that numeric values are correctly converted
  
  - [ ]* 4.3 Write unit tests for CSV validation
    - Test missing required columns error handling
    - Test invalid date format handling
    - Test negative quantity/price rejection
    - Test file size limit (10MB) enforcement
    - _Requirements: Feature 1 (CSV Sales Forecasting), Technical Requirements (10K rows)_

- [ ] 5. Implement ML forecasting engine
  - [ ] 5.1 Create baseline calculation functions
    - Write `calculateMonthlyAverage(history: SalesRow[])` function
    - Write `detectTopSKU(history: SalesRow[])` function for trend identification
    - Write `calculateConfidence(dataPoints: number)` function
    - _Requirements: Feature 1 (CSV Sales Forecasting), Acceptance Criteria (85% confidence)_
  
  - [ ] 5.2 Implement demand forecasting algorithm
    - Write `forecastDemand(history: SalesRow[], targetMonth: number)` function
    - Integrate festival multiplier logic
    - Calculate restock quantity based on baseline × multiplier
    - Implement optimal price calculation from historical data
    - Ensure 85%+ accuracy target through validation
    - _Requirements: Feature 1 (CSV Sales Forecasting), Performance (85%+ accuracy)_
  
  - [ ]* 5.3 Write property test for forecast accuracy
    - **Property 3: Forecast confidence bounds**
    - **Validates: Requirements Feature 1, Acceptance Criteria**
    - Test that confidence is always between 0 and 95
    - Test that more data points increase confidence
  
  - [ ]* 5.4 Write property test for demand calculation
    - **Property 4: Demand increase matches multiplier**
    - **Validates: Requirements Feature 2**
    - Test that demandIncrease = (multiplier × 100) - 100
    - Test that restockQuantity = baseline × multiplier
  
  - [ ]* 5.5 Write unit tests for forecasting edge cases
    - Test empty history handling
    - Test single-row history (low confidence warning)
    - Test outlier detection and handling
    - Test Diwali scenario: 12 months data → +35% demand, 85% confidence
    - _Requirements: Acceptance Criteria (Diwali Forecasting, Low Confidence Handling)_

- [ ] 6. Checkpoint - Ensure core processing logic works
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 7. Implement business impact calculator
  - [ ] 7.1 Create business impact calculation functions
    - Write `generateBusinessImpact(forecast: Forecast, history: SalesRow[])` function
    - Calculate revenue potential from restock quantity × price
    - Calculate overstock risk percentage
    - Calculate accuracy improvement (+25% vs manual baseline)
    - Format currency values in Indian Rupees (₹)
    - _Requirements: Feature 3 (Impact Dashboard), Target KPIs_
  
  - [ ]* 7.2 Write property test for business calculations
    - **Property 5: Revenue calculation consistency**
    - **Validates: Requirements Feature 3**
    - Test that revenuePotential = restockQuantity × suggestedPrice
    - Test that currency formatting includes ₹ symbol
  
  - [ ]* 7.3 Write unit tests for impact metrics
    - Test overstock reduction calculation (30% → 15%)
    - Test accuracy improvement display (+25%)
    - Test large number formatting (₹5.6L, ₹2.5 Cr)
    - _Requirements: Target KPIs (Business KPIs)_

- [ ] 8. Implement API route for forecasting
  - [ ] 8.1 Create POST /api/forecast endpoint
    - Set up Next.js API route at `src/app/api/forecast/route.ts`
    - Handle multipart/form-data file upload
    - Integrate CSV parsing, forecasting, and impact calculation
    - Return JSON response with forecast and business impact
    - Ensure total processing time <3 seconds
    - _Requirements: Feature 1 (CSV Sales Forecasting), Performance (<3 seconds)_
  
  - [ ] 8.2 Implement comprehensive error handling
    - Return 400 for invalid CSV format with descriptive message
    - Return 413 for files exceeding 10MB
    - Return 500 for processing timeout or unexpected errors
    - Add low confidence warning (<70%) in response
    - _Requirements: Acceptance Criteria (Low Confidence Handling), Technical Requirements_
  
  - [ ]* 8.3 Write integration tests for API endpoint
    - Test successful forecast with valid CSV
    - Test error responses for invalid inputs
    - Test performance with 10K row CSV (<3 seconds)
    - Test Diwali scenario end-to-end
    - _Requirements: Acceptance Criteria (Diwali Forecasting), Performance_

- [ ] 9. Checkpoint - Ensure API layer works correctly
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. Implement frontend components
  - [ ] 10.1 Create HeroSection component
    - Build hero section with gradient background (Rose-400 → Amber-500)
    - Add MVP headline and 3 feature cards
    - Implement glassmorphic styling with backdrop-blur-xl
    - Ensure mobile-first responsive design (375px breakpoint)
    - _Requirements: Feature 3 (Impact Dashboard), Usability (Mobile-first)_
  
  - [ ] 10.2 Create CSVUploadForm component
    - Implement drag-and-drop file upload with visual feedback
    - Add file validation (CSV only, max 10MB)
    - Show upload progress indicator
    - Handle file selection and form submission
    - Display error messages for invalid files
    - _Requirements: Feature 1 (CSV Sales Forecasting), Usability (1-click upload)_
  
  - [ ] 10.3 Create ResultsDashboard component
    - Build KPI cards for accuracy, confidence, and demand increase
    - Display top trend alert with restock recommendation
    - Show business impact metrics (revenue potential, overstock risk)
    - Implement glassmorphic card styling
    - Add responsive grid layout (1-column mobile, 2-column tablet, full desktop)
    - _Requirements: Feature 3 (Impact Dashboard), Target KPIs_
  
  - [ ] 10.4 Create FestivalCalendar component
    - Display festival multipliers in calendar format
    - Show Oct (2.8x Diwali), Dec (1.5x Wedding), Feb (2.2x Valentine)
    - Implement 3-column grid for desktop, list for mobile
    - Use accent colors for festival highlights
    - _Requirements: Feature 2 (Festival Multiplier Engine), Feature 3 (Impact Dashboard)_
  
  - [ ]* 10.5 Write component unit tests
    - Test HeroSection renders correctly
    - Test CSVUploadForm file validation
    - Test ResultsDashboard displays forecast data
    - Test FestivalCalendar shows all multipliers
    - _Requirements: Feature 3 (Impact Dashboard)_

- [ ] 11. Implement main page integration
  - [ ] 11.1 Create main page with component orchestration
    - Set up `src/app/page.tsx` with all components
    - Implement state management for upload and forecast results
    - Connect CSVUploadForm to API endpoint
    - Handle loading states and error display
    - Show ResultsDashboard and FestivalCalendar after successful forecast
    - _Requirements: Feature 1, Feature 3 (Impact Dashboard)_
  
  - [ ] 11.2 Add design system and styling
    - Configure Tailwind with custom color palette (Terracotta, Deep Teal, Gold)
    - Add Google Fonts: Playfair Display (headings), Inter (body)
    - Implement gradient backgrounds and glassmorphic effects
    - Ensure responsive breakpoints (375px, 768px, 1024px)
    - _Requirements: Usability (Mobile-first), Technical Requirements_
  
  - [ ]* 11.3 Write end-to-end integration tests
    - Test complete user flow: upload → processing → results
    - Test error handling flow with invalid CSV
    - Test responsive design at different breakpoints
    - Test Diwali scenario: upload → +35% demand → 200 units @ ₹2,800
    - _Requirements: Acceptance Criteria (Diwali Forecasting)_

- [ ] 12. Implement performance optimizations
  - [ ] 12.1 Optimize bundle size and loading performance
    - Configure Next.js for optimal code splitting
    - Lazy load heavy components (ResultsDashboard)
    - Optimize images and fonts
    - Ensure bundle size <500KB gzipped
    - _Requirements: Performance (Bundle Size <500KB)_
  
  - [ ] 12.2 Add performance monitoring
    - Implement timing measurements for CSV parsing (<500ms)
    - Add API response time tracking (<3 seconds)
    - Log performance metrics to console in development
    - _Requirements: Performance (<3 seconds processing)_
  
  - [ ]* 12.3 Run Lighthouse performance audit
    - Test mobile performance score (target: 95+)
    - Test desktop performance score
    - Verify accessibility and best practices scores
    - _Requirements: Performance (Lighthouse Score 95+)_

- [ ] 13. Final checkpoint - End-to-end validation
  - Ensure all tests pass, ask the user if questions arise.
  - Test complete Diwali forecasting scenario
  - Verify <3 second processing time with 10K rows
  - Confirm 85%+ accuracy on validation dataset
  - Test mobile responsiveness on actual devices

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation at key milestones
- Property tests validate universal correctness properties (multiplier bounds, calculation consistency)
- Unit tests validate specific examples and edge cases (Diwali scenario, error handling)
- The implementation follows a bottom-up approach: data models → processing → API → frontend
- All performance targets (<3s processing, <500KB bundle, 95+ Lighthouse) are validated in tasks
- Festival multiplier engine is implemented early to support forecasting logic
- Mobile-first design is enforced throughout frontend implementation
