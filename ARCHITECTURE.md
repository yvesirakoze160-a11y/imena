# IMENA Architecture

## System Design Overview

IMENA is a full-stack, geographically-aware platform built with:
- **Backend:** Node.js + Express.js + PostgreSQL
- **Frontend:** React 18 + Tailwind CSS + Leaflet.js
- **Database:** PostgreSQL with PostGIS for geographic queries
- **Authentication:** JWT + refresh tokens
- **Payments:** Adapter pattern for MTN MoMo / Airtel Money

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT LAYER (React)                      │
│  ┌──────────────┬──────────────┬──────────────────────────┐ │
│  │   Web UI     │  Mobile UI   │    Geographic Map        │ │
│  │  Dashboard   │  (Responsive)│    (Leaflet + GeoJSON)   │ │
│  └──────────────┴──────────────┴──────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                           ↓ (REST API)
┌─────────────────────────────────────────────────────────────┐
│                   API LAYER (Express.js)                     │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Authentication   │  Authorization  │  Middleware     │  │
│  │  (JWT)            │  (RBAC + Scope) │  (Logging, etc) │  │
│  └───────────────────────────────────────────────────────┘  │
│  ┌───────────────���───────────────────────────────────────┐  │
│  │ Routes:                                                 │  │
│  │ • /api/auth          (login, register, refresh)        │  │
│  │ • /api/users         (user profile, preferences)       │  │
│  │ • /api/geographic    (provinces, districts, sectors)   │  │
│  │ • /api/opportunities (CRUD, search, filter)            │  │
│  │ • /api/activities    (log, verify, report)             │  │
│  │ • /api/payments      (initiate, verify, ledger)        │  │
│  │ • /api/organizations (profile, verification)           │  │
│  │ • /api/dashboards    (statistics, impact reports)      │  │
│  │ • /api/admin         (data import, configuration)      │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           ↓ (SQL + Transactions)
┌─────────────────────────────────────────────────────────────┐
│               DATABASE LAYER (PostgreSQL)                    │
│  ┌──────────────┬──────────────────┬──────────────────────┐ │
│  │ Geographic   │ Business Logic   │ Financial Ledger     │ │
│  │ Tables       │ Tables           │ Tables               │ │
│  │ (Hierarchy)  │ (Opportunities,  │ (Transactions,       │ │
│  │              │  Activities,     │  Wallets, Rewards)   │ │
│  │ • Countries  │  Organizations)  │                      │ │
│  │ • Provinces  │                  │ • Transactions       │ │
│  │ • Districts  │ • Users          │ • Ledger Entries     │ │
│  │ • Sectors    │ • Participants   │ • Wallets            │ │
│  │ • Cells      │ • Opportunities  │ • Settlements        │ │
│  │ • Villages   │ • Activities     │                      │ │
│  │              │ • Verifications  │ PostGIS Extensions   │ │
│  │ PostGIS      │ • Feedback       │ for geographic       │ │
│  │ enabled for  │ • Organizations  │ queries              │ │
│  │ spatial ops  │ • Roles & Perms  │                      │ │
│  └──────────────┴──────────────────┴──────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
         ↓ (Async queues, webhooks)
┌─────────────────────────────────────────────────────────────┐
│                  EXTERNAL INTEGRATIONS                       │
│  ┌──────────────┬──────────────┬──────────────────────────┐ │
│  │ MTN MoMo API │ Airtel Money │  Email/SMS Provider      │ │
│  │ (Payments)   │ (Payments)   │  (Notifications)         │ │
│  └──────────────┴──────────────┴──────────────────────────┘ │
│  ┌──────────────┬──────────────┬──────────────────────────┐ │
│  │ Map Provider │ File Storage │  Monitoring/Logging      │ │
│  │ (Leaflet)    │ (S3/Local)   │  (Sentry, etc)           │ │
│  └──────────────┴──────────────┴──────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Database Schema

### 1. Geographic Hierarchy

```sql
-- Countries
CREATE TABLE countries (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  code VARCHAR(2) UNIQUE,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Provinces
CREATE TABLE provinces (
  id UUID PRIMARY KEY,
  country_id UUID REFERENCES countries(id),
  name VARCHAR(255) NOT NULL,
  code VARCHAR(10),
  geometry GEOGRAPHY(POLYGON, 4326),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Districts
CREATE TABLE districts (
  id UUID PRIMARY KEY,
  province_id UUID REFERENCES provinces(id),
  name VARCHAR(255) NOT NULL,
  code VARCHAR(10),
  geometry GEOGRAPHY(POLYGON, 4326),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Sectors
CREATE TABLE sectors (
  id UUID PRIMARY KEY,
  district_id UUID REFERENCES districts(id),
  name VARCHAR(255) NOT NULL,
  code VARCHAR(10),
  geometry GEOGRAPHY(POLYGON, 4326),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Cells
CREATE TABLE cells (
  id UUID PRIMARY KEY,
  sector_id UUID REFERENCES sectors(id),
  name VARCHAR(255) NOT NULL,
  code VARCHAR(10),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Villages
CREATE TABLE villages (
  id UUID PRIMARY KEY,
  cell_id UUID REFERENCES cells(id),
  name VARCHAR(255) NOT NULL,
  code VARCHAR(10),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Index for fast hierarchical queries
CREATE INDEX idx_provinces_country ON provinces(country_id);
CREATE INDEX idx_districts_province ON districts(province_id);
CREATE INDEX idx_sectors_district ON sectors(district_id);
CREATE INDEX idx_cells_sector ON cells(sector_id);
CREATE INDEX idx_villages_cell ON villages(cell_id);

-- Geographic indexes for spatial queries
CREATE INDEX idx_provinces_geometry ON provinces USING GIST(geometry);
CREATE INDEX idx_districts_geometry ON districts USING GIST(geometry);
CREATE INDEX idx_sectors_geometry ON sectors USING GIST(geometry);
```

### 2. User & Authorization

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  phone VARCHAR(20),
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  avatar_url VARCHAR(500),
  preferred_language VARCHAR(5) DEFAULT 'en',
  status VARCHAR(50) DEFAULT 'active', -- active, suspended, deleted
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE roles (
  id UUID PRIMARY KEY,
  name VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE permissions (
  id UUID PRIMARY KEY,
  name VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  resource VARCHAR(100),
  action VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE role_permissions (
  id UUID PRIMARY KEY,
  role_id UUID REFERENCES roles(id),
  permission_id UUID REFERENCES permissions(id),
  UNIQUE(role_id, permission_id)
);

CREATE TABLE user_roles (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  role_id UUID REFERENCES roles(id),
  scope_type VARCHAR(50), -- 'national', 'province', 'district', 'sector'
  scope_id UUID, -- province_id, district_id, sector_id, or NULL for national
  assigned_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, role_id, scope_type, scope_id)
);
```

### 3. Opportunities & Activities

```sql
CREATE TABLE opportunities (
  id UUID PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  category VARCHAR(100), -- agriculture, education, environment, etc.
  sector VARCHAR(100),
  district_id UUID REFERENCES districts(id),
  sector_id UUID REFERENCES sectors(id),
  organization_id UUID REFERENCES organizations(id),
  creator_id UUID REFERENCES users(id),
  status VARCHAR(50) DEFAULT 'active', -- active, closed, draft
  participant_count INT DEFAULT 0,
  reward_amount_rwf DECIMAL(12, 2),
  reward_per_activity_rwf DECIMAL(12, 2),
  start_date DATE,
  end_date DATE,
  verification_required BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE activities (
  id UUID PRIMARY KEY,
  opportunity_id UUID REFERENCES opportunities(id),
  participant_id UUID REFERENCES users(id),
  description TEXT,
  activity_date DATE NOT NULL,
  location_district_id UUID REFERENCES districts(id),
  location_sector_id UUID REFERENCES sectors(id),
  location_lat DECIMAL(10, 8),
  location_lng DECIMAL(11, 8),
  evidence_photos TEXT[], -- JSON array of photo URLs
  evidence_documents TEXT[], -- JSON array of doc URLs
  status VARCHAR(50) DEFAULT 'submitted', -- submitted, verified, rejected
  verified_by UUID REFERENCES users(id),
  verified_at TIMESTAMP,
  reward_amount_rwf DECIMAL(12, 2),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_opportunities_district ON opportunities(district_id);
CREATE INDEX idx_opportunities_status ON opportunities(status);
CREATE INDEX idx_activities_opportunity ON activities(opportunity_id);
CREATE INDEX idx_activities_status ON activities(status);
```

### 4. Financial Ledger

```sql
CREATE TABLE wallets (
  id UUID PRIMARY KEY,
  user_id UUID UNIQUE REFERENCES users(id),
  balance_rwf DECIMAL(15, 2) DEFAULT 0,
  currency VARCHAR(3) DEFAULT 'RWF',
  status VARCHAR(50) DEFAULT 'active',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE transactions (
  id UUID PRIMARY KEY,
  wallet_id UUID REFERENCES wallets(id),
  transaction_type VARCHAR(50), -- reward, withdrawal, refund, settlement
  amount_rwf DECIMAL(12, 2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'RWF',
  status VARCHAR(50) DEFAULT 'pending', -- pending, completed, failed
  payment_provider VARCHAR(100), -- mtn_momo, airtel_money, bank_transfer
  provider_reference VARCHAR(255),
  provider_response JSONB,
  description TEXT,
  activity_id UUID REFERENCES activities(id),
  initiated_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE ledger_entries (
  id UUID PRIMARY KEY,
  wallet_id UUID REFERENCES wallets(id),
  transaction_id UUID REFERENCES transactions(id),
  debit_rwf DECIMAL(12, 2) DEFAULT 0,
  credit_rwf DECIMAL(12, 2) DEFAULT 0,
  balance_after_rwf DECIMAL(15, 2),
  entry_type VARCHAR(50),
  reference TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_transactions_wallet ON transactions(wallet_id);
CREATE INDEX idx_transactions_status ON transactions(status);
CREATE INDEX idx_ledger_wallet ON ledger_entries(wallet_id);
```

### 5. Organizations

```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  organization_type VARCHAR(100), -- ngo, cooperative, company, school, etc.
  description TEXT,
  district_id UUID REFERENCES districts(id),
  website VARCHAR(500),
  contact_email VARCHAR(255),
  contact_phone VARCHAR(20),
  verification_status VARCHAR(50) DEFAULT 'pending',
  verified_by UUID REFERENCES users(id),
  verified_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE organization_members (
  id UUID PRIMARY KEY,
  organization_id UUID REFERENCES organizations(id),
  user_id UUID REFERENCES users(id),
  role VARCHAR(100), -- admin, staff, volunteer
  joined_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(organization_id, user_id)
);
```

### 6. Data Sources (for Rwanda info tracking)

```sql
CREATE TABLE data_sources (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  institution VARCHAR(255),
  dataset_name VARCHAR(255),
  source_url VARCHAR(500),
  description TEXT,
  date_collected DATE,
  last_updated DATE,
  verification_status VARCHAR(50),
  license_notes TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

## API Structure

### Authentication Endpoints

```
POST   /api/auth/register        -- Register new user
POST   /api/auth/login           -- Login with email/password
POST   /api/auth/refresh         -- Refresh JWT token
POST   /api/auth/logout          -- Logout user
GET    /api/auth/me              -- Get current user profile
```

### Geographic Endpoints

```
GET    /api/geographic/countries -- List all countries
GET    /api/geographic/provinces -- List provinces (with optional country filter)
GET    /api/geographic/districts -- List districts (with optional province filter)
GET    /api/geographic/sectors   -- List sectors (with optional district filter)
GET    /api/geographic/cells     -- List cells (with optional sector filter)
GET    /api/geographic/villages  -- List villages (with optional cell filter)
GET    /api/geographic/search    -- Search geographic locations
GET    /api/geographic/map       -- Get GeoJSON for mapping
```

### Opportunities Endpoints

```
GET    /api/opportunities                 -- List opportunities (with filters)
POST   /api/opportunities                 -- Create opportunity
GET    /api/opportunities/:id             -- Get opportunity details
PUT    /api/opportunities/:id             -- Update opportunity
DELETE /api/opportunities/:id             -- Archive opportunity
POST   /api/opportunities/:id/apply       -- Apply to opportunity
GET    /api/opportunities/:id/participants -- List participants
```

### Activities Endpoints

```
POST   /api/activities                    -- Submit activity
GET    /api/activities                    -- List activities (with filters)
GET    /api/activities/:id                -- Get activity details
PUT    /api/activities/:id                -- Update activity
POST   /api/activities/:id/verify         -- Verify activity (admin)
POST   /api/activities/:id/reject         -- Reject activity (admin)
```

### Payments Endpoints

```
POST   /api/payments/initiate             -- Initiate payment/reward
GET    /api/payments/:id/status           -- Check payment status
GET    /api/wallets/me                    -- Get user's wallet
GET    /api/wallets/me/transactions       -- Get transaction history
POST   /api/wallets/me/withdraw           -- Request withdrawal
```

### Dashboards Endpoints

```
GET    /api/dashboards/national           -- National statistics
GET    /api/dashboards/province/:id       -- Province statistics
GET    /api/dashboards/district/:id       -- District statistics
GET    /api/dashboards/sector/:id         -- Sector statistics
GET    /api/dashboards/geographic-map     -- Map statistics
```

### Admin Endpoints

```
POST   /api/admin/data-sources            -- Create/update data source
GET    /api/admin/data-sources            -- List data sources
POST   /api/admin/import                  -- Import CSV/JSON data
GET    /api/admin/users                   -- List users
PUT    /api/admin/users/:id/role          -- Assign user role/scope
```

## Authorization Model

**Scope-Based RBAC:**

```javascript
// User with national admin role
{
  role: "admin",
  scope_type: "national",
  scope_id: null,
  permissions: ["*"]
}

// User with district-level role
{
  role: "district_coordinator",
  scope_type: "district",
  scope_id: "uuid-of-kigali-city-district",
  permissions: [
    "opportunities:read",
    "opportunities:create",
    "activities:read",
    "activities:verify",
    "dashboards:read"
  ]
}

// Regular community member
{
  role: "community_member",
  scope_type: null,
  scope_id: null,
  permissions: [
    "opportunities:read",
    "activities:create",
    "wallets:read"
  ]
}
```

**Authorization Middleware:**

```javascript
// Example: Only district admin can see/modify data in their district
middleware.requireScope('district', (req) => {
  return req.params.districtId; // Validate against user's scope_id
});
```

## Frontend Architecture

### Directory Structure

```
imena-frontend/
├── src/
│   ├── pages/              -- Page components
│   ├── components/         -- Reusable components
│   ├── features/           -- Feature modules (opportunities, activities, etc.)
│   ├── services/           -- API clients
│   ├── store/              -- Redux state management
│   ├── hooks/              -- Custom React hooks
│   ├── locales/            -- Translation files (en, rw, fr)
│   ├── styles/             -- Global styles
│   ├── utils/              -- Helper functions
│   └── App.jsx
├── public/
│   └── geojson/            -- Rwanda GeoJSON boundary files
└── package.json
```

### State Management (Redux)

```javascript
// Store structure
{
  auth: {
    user: null,
    token: null,
    roles: [],
    isLoading: false
  },
  geographic: {
    selectedCountry: null,
    selectedProvince: null,
    selectedDistrict: null,
    selectedSector: null,
    mapData: {},
    isLoading: false
  },
  opportunities: {
    list: [],
    currentOpportunity: null,
    filters: {
      category: null,
      district_id: null,
      status: 'active'
    },
    isLoading: false
  },
  activities: {
    list: [],
    currentActivity: null,
    filters: {},
    isLoading: false
  },
  payments: {
    wallet: null,
    transactions: [],
    isLoading: false
  },
  dashboards: {
    statistics: {},
    mapStatistics: {},
    isLoading: false
  },
  ui: {
    sidebarOpen: true,
    theme: 'light',
    language: 'en'
  }
}
```

## Security Considerations

1. **Authentication:**
   - JWT with 15-minute expiry
   - Refresh tokens stored securely (httpOnly cookies)
   - Password hashing with bcrypt (12 rounds)

2. **Authorization:**
   - Scope-based access control
   - Every API endpoint validates user's scope
   - Database queries filtered by scope automatically

3. **Data Protection:**
   - No citizen location data exposed in public APIs
   - Financial data encrypted at rest
   - Audit logging for all admin actions
   - HTTPS enforced in production

4. **External Integrations:**
   - Payment credentials stored in environment variables (never in code)
   - Webhook signature verification for payment confirmations
   - Rate limiting on payment endpoints
   - No test payments in production

## Deployment Architecture

```
┌─────────────────┐
│ GitHub Repo     │
│ (Main branch)   │
└────────┬────────┘
         │
         ↓
┌─────────────────────────────────────┐
│ CI/CD Pipeline (GitHub Actions)     │
│ ├─ Lint & Format                    │
│ ├─ Run Tests                        │
│ ├─ Build Docker Images              │
│ └─ Push to Registry                 │
└────────┬────────────────────────────┘
         │
         ↓
┌─────────────────────────────────────┐
│ Cloud Environment (e.g., Heroku)    │
│ ├─ PostgreSQL Database              │
│ ├─ Express.js Backend               │
│ ├─ React Frontend (Static)          │
│ └─ Nginx Reverse Proxy              │
└─────────────────────────────────────┘
```

## Performance Optimization

1. **Database:**
   - Geographic indexes with GIST
   - Pagination for large result sets
   - Query optimization with EXPLAIN ANALYZE

2. **Frontend:**
   - Code splitting with React.lazy()
   - Image optimization
   - Memoization for expensive computations
   - Service worker for offline capabilities

3. **API:**
   - Response caching (Redis optional)
   - Compression with gzip
   - Connection pooling for database
   - Rate limiting

---

**Status:** Architecture Phase  
**Last Updated:** 2026-09-05  
**Next:** Database Setup & Migrations
