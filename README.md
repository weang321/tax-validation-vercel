# Tax Validation Vercel App v2.0

A modern, secure Vercel app for validating Indonesian corporate tax invoices through dual-agent verification.

## What's New in v2.0

### 🛡️ Security Hardening
- **Rate Limiting**: Per-action rate limits (10 validations/min, 5 uploads/min, 20 feedback/min)
- **CSRF Protection**: Token-based CSRF mitigation for all mutating requests
- **Input Validation**: Comprehensive sanitization for strings, numbers, file paths
- **File Validation**: Type checking, size limits (8MB), extension whitelisting
- **XSS Prevention**: HTML escaping for all dynamic content
- **Security Headers**: CSP, X-Frame-Options, X-Content-Type-Options in vercel.json

### 🎨 UI/UX Improvements
- **Dark Mode**: Full dark theme with system preference detection
- **Progress Indicators**: Visual progress bars with loading states
- **Mobile Responsive**: Bottom navigation, responsive grid layouts
- **Toast Notifications**: Non-blocking success/error/warning notifications
- **Glass Morphism**: Modern translucent panel design

### 📊 Dashboard
- Real-time validation statistics
- Accuracy tracking metrics
- Recent activity feed
- Quick-start validation buttons

### 🔔 Notification System
- Toast notifications for all actions
- Activity log with timestamps
- Error handling with user-friendly messages

### 🧪 Testing
- Unit tests for security utilities (rate limiting, CSRF, sanitization)
- Unit tests for API client error handling
- Vitest configuration with coverage reporting

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Frontend (React + TypeScript + Tailwind CSS)       │
│  ┌─────────┐ ┌──────────┐ ┌────────┐ ┌──────────┐ │
│  │Dashboard│ │Validation│ │Reports │ │  Admin   │ │
│  └─────────┘ └──────────┘ └────────┘ └──────────┘ │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  API Layer (Rate Limited + CSRF Protected)          │
│  ┌───────────────┐ ┌──────────────┐ ┌────────────┐│
│  │validate_tax   │ │drive_pickup  │ │export_report││
│  └───────────────┘ └──────────────┘ └────────────┘│
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  Validation Engine                                  │
│  ┌────────────────┐     ┌──────────────────┐       │
│  │ Tax Agent 1    │────▶│ Tax Agent 2      │       │
│  │ (First Pass)   │     │ (Independent)    │       │
│  └────────────────┘     └──────────────────┘       │
└─────────────────────────────────────────────────────┘
```

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, TypeScript, Tailwind CSS |
| State | Zustand with persistence |
| Build | Vite |
| Testing | Vitest |
| Backend | Python (Vercel Serverless) |
| Database | Supabase (optional) |
| Storage | Google Drive API |

## Getting Started

### Prerequisites
- Node.js 18+
- npm or yarn

### Installation

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Run tests
npm test

# Run tests with coverage
npm run test:coverage
```

### Environment Variables

Set these in Vercel or `.env.local`:

```env
# Supabase (optional, for persistence)
SUPABASE_URL=
SUPABASE_SERVICE_ROLE_KEY=

# Google Drive Auto Pickup
GOOGLE_DRIVE_API_KEY=
# or
GOOGLE_SERVICE_ACCOUNT_JSON=

# Security Tokens
SUPER_ADMIN_UPLOAD_TOKEN=
TAX_REVIEWER_FEEDBACK_TOKEN=
DRIVE_PICKUP_TOKEN=
CRON_SECRET=
```

## Security Features

### Rate Limiting

| Action | Limit | Window |
|--------|-------|--------|
| Validation | 10 requests | 1 minute |
| File Upload | 5 requests | 1 minute |
| Feedback | 20 requests | 1 minute |
| Report Export | 3 requests | 1 minute |
| Default | 30 requests | 1 minute |

### CSRF Protection
- Token generated per session using `crypto.getRandomValues()`
- Token attached to all mutating API requests via `X-CSRF-Token` header
- 64-character hex token validated server-side

### Input Validation
- Strings: HTML escaping, length limits
- Numbers: Range validation, overflow protection
- Files: Extension whitelist, size limits (8MB)
- Paths: Path traversal prevention

### Security Headers
```json
{
  "X-Content-Type-Options": "nosniff",
  "X-Frame-Options": "DENY",
  "X-XSS-Protection": "1; mode=block",
  "Referrer-Policy": "strict-origin-when-cross-origin",
  "Permissions-Policy": "camera=(), microphone=(), geolocation=()"
}
```

## Features

### Tax Agent 1 (First Pass)
- Validates invoice completeness against master knowledge base
- Checks PPN/VAT calculations
- Verifies PPh withholding tax
- Flags duplicates and invalid invoices
- Risk assessment (Low/Medium/High/Critical)

### Tax Agent 2 (Independent Re-check)
- Cross-checks Agent 1 results
- Verifies rates against DJP official guidance
- Compares against uploaded tax tables
- Produces compliance report

### Auto Pickup
- Scans Google Drive every 5 minutes
- Processes folders starting with "tax invoice"
- Skips folders with "_closed" suffix
- Supports nested month folders

### Report Export
- Excel workbook with multiple sheets
- PDF management summary
- Activity log included

### Human Review
- Tax team feedback with accuracy tracking
- DJP re-check on corrections
- Final decision workflow (agree/disagree)

## Project Structure

```
tax-validation-vercel/
├── api/                    # Vercel serverless functions
│   ├── validate_tax.py     # Main validation logic
│   ├── tax_invoice_extract.py
│   ├── drive_auto_pickup.py
│   ├── reviewer_feedback.py
│   ├── tax_team_decision.py
│   ├── export_report.py
│   └── refresh_djp_rates.py
├── src/                    # React frontend
│   ├── components/         # React components
│   │   ├── Dashboard.tsx
│   │   ├── ValidationPanel.tsx
│   │   ├── ReportCentre.tsx
│   │   ├── AdminPanel.tsx
│   │   ├── Sidebar.tsx
│   │   ├── ThemeToggle.tsx
│   │   └── NotificationCenter.tsx
│   ├── store/              # Zustand state
│   ├── types/              # TypeScript types
│   ├── utils/              # Utilities
│   │   ├── api.ts          # API client
│   │   ├── security.ts     # Security functions
│   │   ├── api.test.ts     # API tests
│   │   └── security.test.ts # Security tests
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
└── vercel.json
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/validate_tax` | POST | Run validation (Agent 1 or 2) |
| `/api/tax_invoice_extract` | POST | Extract invoice data from files |
| `/api/drive_auto_pickup` | POST | Scan Google Drive for invoices |
| `/api/reviewer_feedback` | POST | Submit tax team feedback |
| `/api/tax_team_decision` | POST | Submit final decision |
| `/api/export_report` | POST | Export management report |
| `/api/tax_table_upload` | POST | Upload tax table (admin) |
| `/api/refresh_djp_rates` | POST/GET | Refresh DJP rates |

## Cron Jobs

| Schedule | Endpoint | Description |
|----------|----------|-------------|
| Every 2 days | `/api/refresh_djp_rates` | Auto-refresh DJP tax rates |

## Testing

```bash
# Run all tests
npm test

# Run with coverage
npm run test:coverage

# Watch mode
npm test -- --watch
```

Test coverage includes:
- Rate limiting logic
- CSRF token generation/validation
- Input sanitization (strings, numbers, paths)
- File validation
- Tax data validation
- Token/email validation
- HTML escaping

## Deployment

### Quick Deploy

1. **Fork/Clone** this repository
2. **Create Supabase project** and run `supabase/schema.sql`
3. **Deploy to Vercel**:
   ```bash
   npm install
   ./scripts/deploy-vercel.sh
   ```

### Environment Variables

Required in Vercel Dashboard:

| Variable | Description |
|----------|-------------|
| `VITE_SUPABASE_URL` | Supabase project URL |
| `VITE_SUPABASE_ANON_KEY` | Supabase public key |

See [DEPLOYMENT.md](./DEPLOYMENT.md) for full guide.

### GitHub Actions

Auto-deploys on push to `main`:
- Runs tests
- Builds frontend
- Deploys to Vercel

Setup secrets: `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`

## License

Proprietary - TSH Resources
