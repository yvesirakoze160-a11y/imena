# IMENA Data Sources & Rwanda Research

## Overview

This document tracks all external data sources used in IMENA, with emphasis on Rwanda-specific information sourced from authoritative institutions.

**Principle:** Every Rwanda-specific dataset includes source attribution, access date, and verification status.

---

## 1. Rwanda Geographic Data

### 1.1 Administrative Hierarchy (Provinces & Districts)

**Source:** Government of Rwanda - Local Government Directory  
**URL:** https://www.gov.rw/government/directory/local-government  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Authoritative Government Source  
**Data Format:** Hierarchical listing  
**Last Updated by Source:** Ongoing  

**Contents:**
- 5 Provinces/Kigali divisions
- 30 Districts
- Hierarchical relationships

**Districts by Province (2024):**

**City of Kigali (3 Districts):**
- Gasabo
- Kicukiro
- Nyarugenge

**Eastern Province (7 Districts):**
- Bugesera
- Gatsibo
- Kayonza
- Kirehe
- Ngoma
- Nyagatare
- Rwamagana

**Northern Province (5 Districts):**
- Burera
- Gakenke
- Gicumbi
- Musanze
- Rulindo

**Southern Province (8 Districts):**
- Gisagara
- Huye
- Kamonyi
- Muhanga
- Nyamagabe
- Nyanza
- Nyaruguru
- Ruhango

**Western Province (7 Districts):**
- Karongi
- Ngororero
- Nyabihu
- Nyamasheke
- Rubavu
- Rusizi
- Rutsiro

**Database Status:** Ready for seeding into geographic tables

---

### 1.2 District Land Use & Master Plans

**Source:** Urban Planning Institute (UPI) Rwanda  
**URL:** https://upi.rw/master-plans  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Government Planning Source  
**Data Format:** GeoJSON, shapefiles, PDF reports  
**Update Frequency:** Ongoing  

**Contents:**
- District boundary GeoJSON
- Sector boundaries where available
- Land use classifications
- Development plans

**Usage in IMENA:**
- Map rendering (district/sector boundaries)
- Geographic queries (point-in-polygon)
- Administrative filtering

---

### 1.3 Geographic Coordinates & Gazetteer

**Source:** National Institute of Statistics Rwanda (NISR)  
**URL:** https://statistics.gov.rw  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Statistics Source  
**Data Format:** CSV, GIS layers  
**Update Frequency:** Census cycles + ongoing updates  

**Contents:**
- Province/district coordinate data
- Population centers
- Administrative center locations

**Database Status:** To be imported after NISR API documentation review

---

## 2. Rwanda Payment Ecosystem

### 2.1 MTN Mobile Money (MoMo)

**Provider:** MTN Rwanda  
**Service:** MTN Mobile Money (MoMo)  
**Market Share:** ~70% of Rwanda mobile money market  
**URL:** https://www.mtnonline.com/rw  
**API Documentation:** Available via merchant portal  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Active Commercial Service  

**Integration Requirements:**
- Merchant account
- API credentials (API key, secret)
- Webhook endpoint configuration
- Transaction fee structure (RWF 600-1000 per transaction)

**Implementation Status:** Adapter pattern created, awaiting credentials

**Ledger Requirements:**
- Transaction ID
- Status (pending → completed/failed)
- Provider response webhook
- User wallet update

---

### 2.2 Airtel Money

**Provider:** Airtel Rwanda  
**Service:** Airtel Money  
**Market Share:** ~25-30% of Rwanda mobile money market  
**Interoperability:** Full interoperability with MTN MoMo  
**URL:** https://www.airtelrwanda.rw  
**API Documentation:** Available via merchant portal  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Active Commercial Service  

**Integration Requirements:**
- Merchant account
- API credentials
- Webhook configuration
- Transaction fee structure

**Implementation Status:** Adapter pattern created, awaiting credentials

---

### 2.3 Payment Aggregators

**Provider:** RwandaPay  
**URL:** https://rwandapay.rw/  
**Accessed:** 2026-09-05  
**Service:** Payment gateway aggregating MTN MoMo & Airtel Money  
**Verification Status:** ✅ Active Commercial Service  

**Alternative:** PaySnapper  
**URL:** https://paysnapper.com/country/rwanda/  
**Service:** Multi-provider payment routing  

**Implementation Status:** Consider if direct provider integration requires additional overhead

---

## 3. Rwanda Business & Legal Environment

### 3.1 Business Registration

**Source:** Rwanda Revenue Authority (RRA)  
**URL:** https://www.rra.gov.rw  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Tax Authority  

**Requirements for IMENA:**
- No automatic business verification (mark as pending)
- Verification workflow requires official RRA integration (future)
- Organizations table includes verification_status field

**Current Implementation:** Manual verification by administrators

---

### 3.2 Data Protection Requirements

**Source:** National Data Protection Authority (Rwanda)  
**Framework:** Rwanda Data Protection Law  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Regulatory Source  

**Key Requirements:**
- User consent for data collection
- Right to access/delete personal data
- Data breach notification procedures
- Cross-border data transfer restrictions

**Implementation Status:**
- Privacy policy required
- Data deletion endpoints (GDPR-style)
- Audit logging for compliance

---

### 3.3 Financial Services Regulation

**Source:** National Bank of Rwanda (BNR)  
**URL:** https://www.bnr.rw  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Central Bank  

**Key Distinction:**
- IMENA is NOT a bank, payment institution, or financial service provider
- IMENA operates a ledger/reward system tied to activities
- All payments go through licensed payment providers (MTN, Airtel)
- IMENA holds no user funds (payments direct to provider)

**Legal Risk Assessment:**
- ✅ Safe: Reward system for verified activities
- ✅ Safe: Integration with licensed payment providers
- ⚠️ Review if adding savings/lending features

---

## 4. Rwanda Environment & Climate Data

### 4.1 Climate & Environmental Programs

**Source:** Rwanda Environment Management Authority (REMA)  
**URL:** https://www.rema.gov.rw  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Environmental Authority  

**Relevant Programs:**
- National reforestation initiatives
- Wetland restoration programs
- Waste management regulations
- Environmental impact assessment requirements

**IMENA Integration:**
- Opportunity categories aligned with REMA programs
- Activity verification based on environmental standards
- Impact measurement requires baseline/methodology

**Carbon Credit Disclaimer:**
- ⚠️ Internal IMENA impact points ≠ carbon credits
- Carbon credit claims require official methodology (VCS, Gold Standard)
- Requires separate verification registry
- Not implemented in Phase 1

---

### 4.2 Agriculture & Soil Data

**Source:** Rwanda Agriculture Board (RAB)  
**URL:** https://www.rab.gov.rw  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Agricultural Authority  

**Data Available:**
- Crop suitability by district
- Soil classifications
- Agricultural extension programs
- Cooperative networks

**IMENA Integration:**
- Agriculture opportunity category
- District-specific agricultural programs
- Activity geo-tagging for agricultural work

---

## 5. Rwanda Development Programs

### 5.1 Rwanda Development Board

**Source:** Rwanda Development Board (RDB)  
**URL:** https://www.rdb.rw  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Development Authority  

**Programs Relevant to IMENA:**
- Youth entrepreneurship initiatives
- Skills development programs
- SME support programs
- Digital economy initiatives

---

### 5.2 Government Partnerships

**Current Status:** No partnerships claimed without verification

**Process for Future Partnerships:**
1. Official partnership agreement signed
2. Document stored securely (access-controlled)
3. Clearly marked in UI/marketing (not implicit)
4. Regular verification with partner institution

---

## 6. Rwanda Language & Terminology

### 6.1 Official Languages

**Kinyarwanda (rw):**
- Official national language
- Primary language in rural areas
- UI full support required

**English (en):**
- Business/international standard
- Secondary in urban areas
- Full UI support

**French (fr):**
- Historical administrative language
- Regional use in East Africa
- Full UI support

**Resource:** Localization files in `/locales/` directory

---

### 6.2 Terminology Standards

**Geographic Names:**
- Use official spellings from Government of Rwanda
- Example: "Kigali City" not "Kigali" (for the entity)
- Preserve district names exactly

**Currency:**
- RWF (Rwandan Franc)
- Symbol: FRw or Frw
- No automatic currency conversion without verification

**Administrative Terms:**
- Sector (umurenge in Kinyarwanda)
- Cell (akagari)
- Village (ikaramu)

---

## 7. Rwanda Statistics & Demographic Data

### 7.1 National Census

**Source:** National Institute of Statistics Rwanda (NISR)  
**URL:** https://statistics.gov.rw  
**Latest Census:** 2023  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Statistics Source  

**Available Data:**
- Population by district
- Age distribution
- Employment statistics
- Household composition

**IMENA Usage:**
- Dashboard statistics only if marked with source
- No auto-derived statistics without census basis
- Always include "Source: NISR 2023" in displays

---

### 7.2 Economic Indicators

**Source:** National Bank of Rwanda (BNR)  
**URL:** https://www.bnr.rw/statistics  
**Accessed:** 2026-09-05  
**Verification Status:** ✅ Official Central Bank  

**Available Data:**
- Exchange rates (RWF/USD, RWF/EUR, etc.)
- Inflation rates
- Employment statistics
- Economic growth rates

**IMENA Usage:**
- Exchange rates: Use configurable API source (not hard-coded)
- Employment data: Reference in dashboard context

---

## 8. Data Import & Management

### 8.1 Import Process

**Admin Import Workflow:**

```
1. Admin selects dataset (CSV/JSON/GeoJSON)
2. System validates schema
3. System detects duplicates
4. Admin reviews preview (first 100 rows)
5. Admin confirms source metadata:
   - Institution
   - Dataset name
   - Date collected
   - License/usage rights
6. System validates geographic relationships
7. System performs import (transactional)
8. Admin logs import event
9. Import summary email sent
```

**No overwrite without explicit confirmation.**

---

### 8.2 Data Source Registry

**Database Table:** `data_sources`

```sql
{
  id: UUID,
  name: "Rwanda Districts 2024",
  institution: "Government of Rwanda",
  dataset_name: "Administrative Divisions",
  source_url: "https://www.gov.rw/government/directory/local-government",
  description: "Official list of 30 districts across 5 provinces",
  date_collected: "2024-01-15",
  last_updated: "2024-06-30",
  verification_status: "authoritative",
  license_notes: "Public government data",
  created_at: "2026-09-05T22:50:00Z"
}
```

**All external data includes this metadata.**

---

## 9. Demo vs Real Data

### 9.1 Distinction

**Real Data:**
- Imported from verified sources
- Marked with source attribution
- Used in production
- Audit-logged

**Demo Data:**
- Development/testing only
- Clearly labeled "DEMO"
- Never sent to production
- Destroyed after testing

**Example UI Label:**
```
✅ Real Data          | ⚠️ Demo Data (Development Only)
Source: NISR 2023     | Testing Purposes Only
```

---

### 9.2 Demo Dataset

**Included for Development:**

```
Province: Eastern Province
├── District: Gatsibo (DEMO)
│   └── Sector: Kigali (DEMO)
│       └── Cell: Kimuri (DEMO)
│           └── Village: Muhura (DEMO)

Opportunity: Coffee Cooperative Support (DEMO)
├── Location: Gatsibo District
├── Reward: 5,000 RWF (test)
└── Participants: 3 (test users)

Activity: Coffee Processing (DEMO)
├── Submitted by: demo@example.com
├── Verified: No
└── Reward: 5,000 RWF (unpaid, test)
```

---

## 10. Rwanda Map Data & GIS

### 10.1 GeoJSON Sources

**Rwanda Boundaries (GeoJSON):**
- Source: OpenStreetMap / Natural Earth
- License: ODbL (OpenData Commons Database License)
- Format: GeoJSON (FeatureCollections)
- Update Frequency: Regular community updates
- Files Location: `/public/geojson/`

**Files:**
- `provinces.geojson` — Provincial boundaries
- `districts.geojson` — District boundaries
- `sectors.geojson` — Sector boundaries (if available)

**Usage:**
- Leaflet.js map rendering
- Geographic filtering
- Spatial queries (database-side)

---

### 10.2 Leaflet Configuration

```javascript
// Map initialization
const map = L.map('map').setView([latitude, longitude], zoom);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: '© OpenStreetMap contributors',
  maxZoom: 19,
}).addTo(map);

// Load Rwanda districts
fetch('/geojson/districts.geojson')
  .then(r => r.json())
  .then(data => {
    L.geoJSON(data, {
      style: { color: '#3388ff', weight: 2 },
      onEachFeature: (feature, layer) => {
        layer.on('click', () => filterByDistrict(feature.properties.id));
      }
    }).addTo(map);
  });
```

---

## 11. Research Workflow Going Forward

### 11.1 When Implementing New Features

**Before coding any Rwanda-specific feature:**

1. ✅ **Identify** what Rwanda information is needed
2. ✅ **Search** authoritative sources (gov.rw, NISR, etc.)
3. ✅ **Compare** multiple sources if available
4. ✅ **Document** source metadata in this file
5. ✅ **Store** data in database (not hard-coded)
6. ✅ **Implement** feature with source attribution
7. ✅ **Test** geographic relationships and filtering

**Never:**
- ❌ Invent Rwanda administrative data
- ❌ Use random blogs for official information
- ❌ Hard-code district/sector names
- ❌ Claim partnerships without verification
- ❌ Present demo data as real

---

### 11.2 Conflict Resolution

**If sources conflict:**

| Scenario | Resolution |
|----------|-----------|
| Two sources differ | Prefer official government source |
| Newer vs. older data | Prefer most recent official source |
| Missing data | Create database structure; leave empty; document gap |
| Outdated source | Note update needed; continue with best available |

---

## 12. Regulatory Compliance

### 12.1 Rwanda Data Protection

**Requirements:**
- User consent for data collection
- Access to personal data on request
- Right to erasure ("right to be forgotten")
- Breach notification within 72 hours
- Data processing transparency

**Implementation:**
- Privacy policy (Kinyarwanda, English, French)
- User consent checkboxes at registration
- Data export endpoint (GDPR-style)
- Data deletion endpoint
- Audit log of all data processing

---

### 12.2 Financial Compliance

**Rwanda Revenue Authority (RRA) Requirements:**
- All transactions logged with reference
- Monthly transaction reports (if required)
- No unregistered financial services
- Transparent fee structures

**IMENA Compliance:**
- All reward transactions logged in ledger
- Payment provider handles regulatory requirements
- IMENA provides transaction reference only
- No banking/financial regulation applies (not a bank)

---

## 13. Security & Credentials

### 13.1 Credential Management

**Never:**
- ❌ Hard-code API keys in source code
- ❌ Commit `.env` files to Git
- ❌ Store credentials in database
- ❌ Display credentials in logs

**Always:**
- ✅ Store credentials in environment variables
- ✅ Use `.env.example` with placeholder values
- ✅ Rotate credentials regularly
- ✅ Log only credential references (not values)

**Example `.env.example`:**
```
# Payment Providers
MTN_MOMO_API_KEY=your-mtn-api-key-here
MTN_MOMO_API_SECRET=your-mtn-api-secret-here
AIRTEL_MONEY_API_KEY=your-airtel-api-key-here
AIRTEL_MONEY_API_SECRET=your-airtel-api-secret-here

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/imena

# Email/SMS
TWILIO_ACCOUNT_SID=your-twilio-sid
TWILIO_AUTH_TOKEN=your-twilio-token
```

---

## 14. Data Sourcing Checklist

Use this checklist before adding any Rwanda-specific feature:

- [ ] Authoritative source identified
- [ ] Source URL documented
- [ ] Access date recorded
- [ ] Data imported to database
- [ ] Source metadata stored in `data_sources` table
- [ ] Geographic relationships validated
- [ ] Admin import tested
- [ ] Demo data clearly labeled
- [ ] UI displays source attribution
- [ ] No hard-coded values
- [ ] Tests include geographic filtering
- [ ] Documentation updated

---

## 15. Future Data Integrations

### 15.1 Potential (Post-Phase 1)

- Rwanda Identity/verification system
- Government open data portal
- SMS payment confirmation (Twilio/local provider)
- Email notifications
- Environmental impact verification
- Carbon credit registry integration

### 15.2 Not Implementing

- ❌ Government employee directory (privacy)
- ❌ Private citizen contact info (privacy)
- ❌ Automatic carbon credit issuance (requires verification)
- ❌ Direct bank account access (security, regulation)

---

**Last Updated:** 2026-09-05  
**Next Review:** When implementing new Rwanda-specific features  
**Maintained By:** Development Team
