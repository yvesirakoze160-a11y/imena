# IMENA Project Specification

**IMENA — Rwanda Real-World Community Impact and Opportunity Platform**

## Overview

IMENA is a full-stack web application designed for Rwanda that enables:
- Community members to discover and engage with opportunities
- Organizations to post and manage community activities
- Government bodies to track and verify community impact
- Geographic-based opportunity discovery and impact reporting

## Core Principles

1. **Real Rwanda Data** — All Rwanda-specific information sourced from authoritative institutions
2. **Verified Sources** — Geographic data, administrative hierarchies, and statistics documented with source attribution
3. **No Fake Integration** — External services (payments, SMS, maps) fully implemented or clearly marked as unconfigured
4. **Real Currency (RWF)** — All financial transactions in Rwandan Francs
5. **Security First** — Server-side authorization, encrypted credentials, no sensitive data exposure

## Rwanda Administrative Hierarchy

```
Rwanda (Country)
├── City of Kigali (Special Status)
│   ├── Gasabo (District)
│   ├── Kicukiro (District)
│   └── Nyarugenge (District)
├── Eastern Province
│   ├── Bugesera
│   ├── Gatsibo
│   ├── Kayonza
│   ├── Kirehe
│   ├── Ngoma
│   ├── Nyagatare
│   └── Rwamagana
├── Northern Province
│   ├── Burera
│   ├── Gakenke
│   ├── Gicumbi
│   ├── Musanze
│   └── Rulindo
├── Southern Province
│   ├── Gisagara
│   ├── Huye
│   ├── Kamonyi
│   ├── Muhanga
│   ├── Nyamagabe
│   ├── Nyanza
│   ├── Nyaruguru
│   └── Ruhango
└── Western Province
    ├── Karongi
    ├── Ngororero
    ├── Nyabihu
    ├── Nyamasheke
    ├── Rubavu
    ├── Rusizi
    └── Rutsiro
```

**Source:** Government of Rwanda, Local Government Directory (gov.rw)

## Core Features

### 1. Geographic Management
- Rwanda province, district, sector, cell, village hierarchy
- Database-driven geographic search
- Interactive Rwanda map with district boundaries
- District-level opportunity and activity filtering

### 2. Opportunity Management
- Post, discover, and manage community opportunities
- Configurable opportunity categories (agriculture, environment, education, skills, etc.)
- Sector-based filtering
- Application/participant tracking
- Verification workflow

### 3. Activity Tracking
- Record community activities and impacts
- Link activities to opportunities and geographic locations
- Real-time and historical activity logging
- Impact verification workflow

### 4. Payment & Rewards
- RWF-based reward system
- MTN MoMo and Airtel Money integration
- Ledger-based financial tracking
- Transaction history and settlement

### 5. Geographic Dashboards
- National, provincial, district, and sector-level impact summaries
- Opportunity statistics by region
- Participant and activity counts
- Geographic filtering and drill-down

### 6. Government Authorization
- Role-based access control (national, provincial, district, sector levels)
- Scope-based data visibility
- Verification and approval workflows
- Administrative reporting

### 7. Organization Management
- NGO, cooperative, company, and community organization profiles
- Verification status tracking
- Impact area specification
- Contact and project linking

### 8. Multi-Language Support
- Kinyarwanda (rw)
- English (en)
- French (fr)
- Full UI localization
- No machine-translated official content

## Technology Stack

### Backend
- **Framework:** Node.js + Express.js
- **Database:** PostgreSQL with PostGIS for geographic queries
- **Authentication:** JWT with secure refresh tokens
- **ORM:** Prisma
- **API:** RESTful with detailed documentation

### Frontend
- **Framework:** React 18+
- **Styling:** Tailwind CSS
- **State:** Redux Toolkit
- **Maps:** Leaflet.js with GeoJSON for Rwanda boundaries
- **Mobile:** Fully responsive design

### Infrastructure
- **Database:** PostgreSQL with geographic support
- **Storage:** Environment-based (local development, cloud for production)
- **Payment Gateway:** Adapter pattern for MTN MoMo / Airtel Money
- **Email/SMS:** Provider abstraction (Twilio, local, etc.)

## Data Security & Privacy

- No citizen location data exposed publicly
- Server-side authorization enforcement
- Encrypted credential storage
- Audit logging for financial transactions
- GDPR-compatible data retention policies
- Rwanda data protection requirements compliance

## Development Phases

### Phase 1: Foundation
- Repository structure and tooling
- Database schema and migrations
- Authentication and authorization
- Geographic data model

### Phase 2: Core Features
- Opportunity management
- Activity tracking
- Geographic search
- Basic dashboards

### Phase 3: Integration
- Payment provider integration (MTN MoMo, Airtel Money)
- Map rendering
- Geographic filtering
- Administrative workflows

### Phase 4: Polish & Testing
- Comprehensive API testing
- UI/UX refinement
- Performance optimization
- Production readiness

## Data Sources

All Rwanda-specific data sourced from:

| Institution | Dataset | Source URL |
|---|---|---|
| Government of Rwanda | Local Government Directory | gov.rw/government/directory/local-government |
| National Institute of Statistics Rwanda | Geographic & Census Data | statistics.gov.rw |
| Rwanda Information Society Authority | ICT Infrastructure | risa.rw |
| Rwanda Revenue Authority | Tax & Business Info | rra.gov.rw |

## Success Criteria

✅ All Rwanda districts and sectors in database
✅ Geographic search functional and fast
✅ Payment integration non-fake (credentials required or clearly marked)
✅ Dashboard filters by region
✅ Mobile-responsive across all screens
✅ No horizontal scrolling
✅ Real PostgreSQL persistence
✅ Server-side authorization enforcement
✅ Multi-language UI
✅ Comprehensive test coverage

---

**Status:** Specification Phase  
**Last Updated:** 2026-09-05  
**Next:** Database Schema & Architecture
