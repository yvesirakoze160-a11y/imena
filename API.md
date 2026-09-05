# IMENA API Documentation

**Base URL:** `https://api.imena.rw/api` (production) | `http://localhost:3000/api` (development)

**Authentication:** JWT Bearer token in `Authorization` header

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 1. Authentication Endpoints

### 1.1 Register User

```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!@",
  "firstName": "Jane",
  "lastName": "Doe",
  "phoneNumber": "+250791234567",
  "preferredLanguage": "rw"
}
```

**Response (201 Created):**
```json
{
  "id": "user-uuid",
  "email": "user@example.com",
  "firstName": "Jane",
  "lastName": "Doe",
  "phoneNumber": "+250791234567",
  "status": "active",
  "createdAt": "2024-09-05T22:50:00Z"
}
```

**Validation:**
- Email: valid email format, unique
- Password: min 12 chars, uppercase, lowercase, number, special char
- Phone: must be +250XXXXXXXXX (Rwanda format)
- Language: en, rw, or fr

---

### 1.2 Login

```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123!@"
}
```

**Response (200 OK):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "user-uuid",
    "email": "user@example.com",
    "roles": ["community_member"],
    "scope": {
      "scope_type": null,
      "scope_id": null
    }
  }
}
```

**Notes:**
- Access token: 15-minute expiry
- Refresh token: 7-day expiry (httpOnly cookie)
- Return refresh token in response for mobile apps

---

### 1.3 Refresh Token

```http
POST /auth/refresh
Cookie: refreshToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response (200 OK):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "user-uuid",
    "email": "user@example.com"
  }
}
```

---

### 1.4 Get Current User

```http
GET /auth/me
Authorization: Bearer <access-token>
```

**Response (200 OK):**
```json
{
  "id": "user-uuid",
  "email": "user@example.com",
  "firstName": "Jane",
  "lastName": "Doe",
  "phoneNumber": "+250791234567",
  "preferredLanguage": "rw",
  "roles": [
    {
      "role": "district_coordinator",
      "scope_type": "district",
      "scope_id": "kigali-city-uuid"
    }
  ],
  "createdAt": "2024-09-05T22:50:00Z"
}
```

---

### 1.5 Logout

```http
POST /auth/logout
Authorization: Bearer <access-token>
```

**Response (200 OK):**
```json
{
  "message": "Logged out successfully"
}
```

---

## 2. Geographic Endpoints

### 2.1 List Provinces

```http
GET /geographic/provinces
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "province-uuid-1",
      "name": "City of Kigali",
      "code": "KGL",
      "districtCount": 3
    },
    {
      "id": "province-uuid-2",
      "name": "Eastern Province",
      "code": "EST",
      "districtCount": 7
    }
  ],
  "total": 5
}
```

---

### 2.2 List Districts

```http
GET /geographic/districts?provinceId=province-uuid-1&page=1&limit=20
```

**Query Parameters:**
- `provinceId` (optional): Filter by province
- `page` (optional, default 1): Pagination
- `limit` (optional, default 20): Items per page
- `search` (optional): Search district name

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "district-uuid-1",
      "name": "Gasabo",
      "code": "GSB",
      "provinceId": "province-uuid-1",
      "sectorCount": 14
    },
    {
      "id": "district-uuid-2",
      "name": "Kicukiro",
      "code": "KCK",
      "provinceId": "province-uuid-1",
      "sectorCount": 9
    }
  ],
  "total": 3,
  "page": 1,
  "pages": 1
}
```

---

### 2.3 List Sectors

```http
GET /geographic/sectors?districtId=district-uuid-1&page=1&limit=20
```

**Query Parameters:**
- `districtId` (required): Filter by district
- `page` (optional, default 1)
- `limit` (optional, default 20)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "sector-uuid-1",
      "name": "Kicukiro",
      "code": "KCK-001",
      "districtId": "district-uuid-1",
      "cellCount": 8
    }
  ],
  "total": 14,
  "page": 1,
  "pages": 1
}
```

---

### 2.4 Geographic Search

```http
GET /geographic/search?query=kigali&limit=10
```

**Query Parameters:**
- `query` (required): Search term
- `limit` (optional, default 10)

**Response (200 OK):**
```json
{
  "results": [
    {
      "id": "kigali-city-uuid",
      "name": "City of Kigali",
      "type": "province",
      "path": "Rwanda > City of Kigali"
    },
    {
      "id": "gasabo-uuid",
      "name": "Gasabo",
      "type": "district",
      "path": "Rwanda > City of Kigali > Gasabo"
    }
  ]
}
```

---

### 2.5 Get Map GeoJSON

```http
GET /geographic/map?type=districts&provinceId=province-uuid-1
```

**Query Parameters:**
- `type` (required): provinces | districts | sectors
- `provinceId` (optional): Filter by province (for districts/sectors)

**Response (200 OK):**
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "district-uuid-1",
      "properties": {
        "name": "Gasabo",
        "code": "GSB",
        "activitiesCount": 42,
        "opportunitiesCount": 8
      },
      "geometry": {
        "type": "Polygon",
        "coordinates": [[[29.5, -1.9], [29.6, -1.9], ...]]
      }
    }
  ]
}
```

---

## 3. Opportunities Endpoints

### 3.1 List Opportunities

```http
GET /opportunities?districtId=district-uuid&category=agriculture&status=active&page=1
```

**Query Parameters:**
- `districtId` (optional): Filter by district
- `category` (optional): agriculture, education, environment, skills, etc.
- `status` (optional): active, closed, draft
- `organizationId` (optional): Filter by organization
- `page` (optional, default 1)
- `limit` (optional, default 20)
- `search` (optional): Search title/description

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "opportunity-uuid-1",
      "title": "Coffee Cooperative Support",
      "description": "Help local farmers improve coffee farming techniques",
      "category": "agriculture",
      "sector": "agribusiness",
      "district": {
        "id": "district-uuid-1",
        "name": "Gatsibo"
      },
      "organizationId": "org-uuid-1",
      "status": "active",
      "participantCount": 12,
      "rewardAmount": 5000,
      "rewardPerActivity": 5000,
      "startDate": "2024-09-01",
      "endDate": "2024-12-31",
      "verificationRequired": true,
      "createdAt": "2024-09-01T10:00:00Z"
    }
  ],
  "total": 45,
  "page": 1,
  "pages": 3
}
```

---

### 3.2 Get Opportunity Details

```http
GET /opportunities/opportunity-uuid-1
```

**Response (200 OK):**
```json
{
  "id": "opportunity-uuid-1",
  "title": "Coffee Cooperative Support",
  "description": "Help local farmers improve coffee farming techniques",
  "category": "agriculture",
  "sector": "agribusiness",
  "district": {
    "id": "district-uuid-1",
    "name": "Gatsibo"
  },
  "sector": {
    "id": "sector-uuid-1",
    "name": "Kigali"
  },
  "organization": {
    "id": "org-uuid-1",
    "name": "Coffee Cooperative Network",
    "type": "cooperative"
  },
  "createdBy": {
    "id": "user-uuid",
    "firstName": "John",
    "lastName": "Doe"
  },
  "status": "active",
  "participantCount": 12,
  "rewardAmount": 5000,
  "rewardPerActivity": 5000,
  "startDate": "2024-09-01",
  "endDate": "2024-12-31",
  "verificationRequired": true,
  "createdAt": "2024-09-01T10:00:00Z",
  "updatedAt": "2024-09-05T15:00:00Z"
}
```

---

### 3.3 Create Opportunity

```http
POST /opportunities
Authorization: Bearer <access-token>
Content-Type: application/json

{
  "title": "Coffee Cooperative Support",
  "description": "Help local farmers improve coffee farming techniques",
  "category": "agriculture",
  "sector": "agribusiness",
  "districtId": "district-uuid-1",
  "sectorId": "sector-uuid-1",
  "organizationId": "org-uuid-1",
  "rewardAmount": 5000,
  "rewardPerActivity": 5000,
  "startDate": "2024-09-01",
  "endDate": "2024-12-31",
  "verificationRequired": true
}
```

**Response (201 Created):**
```json
{
  "id": "opportunity-uuid-1",
  "title": "Coffee Cooperative Support",
  "status": "active",
  "createdAt": "2024-09-05T22:50:00Z"
}
```

**Authorization:**
- Requires: organization_admin or national_admin role
- Scope: Must be in organization (for org admin) or national (for admin)

---

### 3.4 Apply to Opportunity

```http
POST /opportunities/opportunity-uuid-1/apply
Authorization: Bearer <access-token>
Content-Type: application/json

{
  "motivationStatement": "I am interested in helping improve coffee farming techniques in my community."
}
```

**Response (200 OK):**
```json
{
  "applicationId": "application-uuid-1",
  "opportunityId": "opportunity-uuid-1",
  "status": "applied",
  "appliedAt": "2024-09-05T22:50:00Z"
}
```

---

## 4. Activities Endpoints

### 4.1 Submit Activity

```http
POST /activities
Authorization: Bearer <access-token>
Content-Type: multipart/form-data

opportunityId: opportunity-uuid-1
description: "Trained 5 farmers on sustainable coffee farming practices"
activityDate: 2024-09-05
districtId: district-uuid-1
sectorId: sector-uuid-1
latitude: -1.9536
longitude: 29.8739
evidencePhotos: [file1.jpg, file2.jpg]
evidenceDocuments: [report.pdf]
```

**Response (201 Created):**
```json
{
  "id": "activity-uuid-1",
  "opportunityId": "opportunity-uuid-1",
  "status": "submitted",
  "rewardAmount": 5000,
  "createdAt": "2024-09-05T22:50:00Z"
}
```

---

### 4.2 List Activities

```http
GET /activities?opportunityId=opportunity-uuid-1&status=submitted&page=1
```

**Query Parameters:**
- `opportunityId` (optional): Filter by opportunity
- `status` (optional): submitted, verified, rejected
- `districtId` (optional): Filter by district
- `userId` (optional): Filter by user (requires authorization)
- `page` (optional, default 1)
- `limit` (optional, default 20)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "activity-uuid-1",
      "opportunityId": "opportunity-uuid-1",
      "participantId": "user-uuid",
      "description": "Trained 5 farmers on sustainable coffee farming practices",
      "activityDate": "2024-09-05",
      "districtId": "district-uuid-1",
      "sectorId": "sector-uuid-1",
      "status": "submitted",
      "rewardAmount": 5000,
      "createdAt": "2024-09-05T22:50:00Z"
    }
  ],
  "total": 12,
  "page": 1,
  "pages": 1
}
```

---

### 4.3 Verify Activity (Admin)

```http
POST /activities/activity-uuid-1/verify
Authorization: Bearer <access-token>
Content-Type: application/json

{
  "approved": true,
  "feedback": "Well documented activity. Reward approved."
}
```

**Response (200 OK):**
```json
{
  "id": "activity-uuid-1",
  "status": "verified",
  "verifiedAt": "2024-09-05T23:00:00Z",
  "rewardAmount": 5000
}
```

**Authorization:**
- Requires: district_coordinator or higher role
- Scope: Must have access to activity's district

---

## 5. Wallets & Payments Endpoints

### 5.1 Get User Wallet

```http
GET /wallets/me
Authorization: Bearer <access-token>
```

**Response (200 OK):**
```json
{
  "id": "wallet-uuid-1",
  "userId": "user-uuid",
  "balanceRwf": 15000,
  "currency": "RWF",
  "status": "active",
  "lastUpdated": "2024-09-05T23:00:00Z"
}
```

---

### 5.2 Get Transaction History

```http
GET /wallets/me/transactions?status=completed&page=1&limit=20
```

**Query Parameters:**
- `status` (optional): pending, completed, failed
- `page` (optional, default 1)
- `limit` (optional, default 20)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "transaction-uuid-1",
      "transactionType": "reward",
      "amountRwf": 5000,
      "status": "completed",
      "paymentProvider": "mtn_momo",
      "description": "Reward for activity-uuid-1",
      "completedAt": "2024-09-05T23:00:00Z"
    },
    {
      "id": "transaction-uuid-2",
      "transactionType": "withdrawal",
      "amountRwf": 10000,
      "status": "completed",
      "paymentProvider": "mtn_momo",
      "description": "Withdrawal to +250791234567",
      "completedAt": "2024-09-04T15:30:00Z"
    }
  ],
  "total": 12,
  "page": 1,
  "pages": 1
}
```

---

### 5.3 Initiate Payment

```http
POST /payments/initiate
Authorization: Bearer <access-token>
Content-Type: application/json

{
  "amount": 5000,
  "paymentProvider": "mtn_momo",
  "phoneNumber": "+250791234567",
  "reference": "activity-uuid-1-reward"
}
```

**Response (202 Accepted):**
```json
{
  "transactionId": "transaction-uuid-1",
  "status": "pending",
  "amountRwf": 5000,
  "paymentProvider": "mtn_momo",
  "message": "Payment initiated. Please complete the transaction on your phone.",
  "initiatedAt": "2024-09-05T22:50:00Z"
}
```

---

### 5.4 Check Payment Status

```http
GET /payments/transaction-uuid-1/status
Authorization: Bearer <access-token>
```

**Response (200 OK):**
```json
{
  "transactionId": "transaction-uuid-1",
  "status": "completed",
  "amountRwf": 5000,
  "paymentProvider": "mtn_momo",
  "completedAt": "2024-09-05T23:00:00Z",
  "walletBalance": 15000
}
```

---

## 6. Dashboard Endpoints

### 6.1 National Dashboard

```http
GET /dashboards/national
Authorization: Bearer <access-token>
```

**Response (200 OK):**
```json
{
  "level": "national",
  "statistics": {
    "totalOpportunities": 245,
    "activeOpportunities": 180,
    "totalActivities": 5420,
    "verifiedActivities": 5100,
    "totalParticipants": 3200,
    "totalRewardDistributedRwf": 27150000,
    "averageRewardRwf": 5325
  },
  "byProvince": [
    {
      "province": "City of Kigali",
      "opportunities": 45,
      "activities": 1200,
      "participants": 850
    },
    {
      "province": "Eastern Province",
      "opportunities": 60,
      "activities": 1500,
      "participants": 1050
    }
  ],
  "topCategories": [
    {
      "category": "agriculture",
      "count": 2100,
      "rewardDistributedRwf": 10500000
    },
    {
      "category": "environment",
      "count": 1850,
      "rewardDistributedRwf": 9250000
    }
  ],
  "generatedAt": "2024-09-05T23:00:00Z"
}
```

---

### 6.2 District Dashboard

```http
GET /dashboards/district/district-uuid-1
Authorization: Bearer <access-token>
```

**Response (200 OK):**
```json
{
  "level": "district",
  "districtId": "district-uuid-1",
  "districtName": "Gasabo",
  "statistics": {
    "totalOpportunities": 15,
    "activeOpportunities": 12,
    "totalActivities": 250,
    "verifiedActivities": 240,
    "totalParticipants": 180,
    "totalRewardDistributedRwf": 1250000,
    "averageRewardRwf": 5208
  },
  "bySector": [
    {
      "sector": "Kicukiro",
      "opportunities": 5,
      "activities": 85,
      "participants": 60
    }
  ],
  "generatedAt": "2024-09-05T23:00:00Z"
}
```

**Authorization:**
- Requires: district_coordinator (for own district) or national_admin

---

### 6.3 Geographic Map Statistics

```http
GET /dashboards/geographic-map
```

**Response (200 OK):**
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "district-uuid-1",
      "properties": {
        "name": "Gasabo",
        "opportunities": 15,
        "activities": 250,
        "participants": 180,
        "rewardDistributedRwf": 1250000
      },
      "geometry": {
        "type": "Polygon",
        "coordinates": [...]
      }
    }
  ]
}
```

---

## 7. Organizations Endpoints

### 7.1 List Organizations

```http
GET /organizations?verified=true&type=ngo&page=1
```

**Query Parameters:**
- `verified` (optional): true | false
- `type` (optional): ngo, cooperative, company, school, etc.
- `districtId` (optional): Filter by district
- `page` (optional, default 1)
- `limit` (optional, default 20)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "org-uuid-1",
      "name": "Coffee Cooperative Network",
      "type": "cooperative",
      "district": { "id": "district-uuid-1", "name": "Gatsibo" },
      "description": "Supporting small-scale coffee farmers",
      "verificationStatus": "verified",
      "contactEmail": "info@coffeecoop.rw",
      "website": "https://coffeecoop.rw",
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ],
  "total": 42,
  "page": 1,
  "pages": 3
}
```

---

### 7.2 Get Organization Details

```http
GET /organizations/org-uuid-1
```

**Response (200 OK):**
```json
{
  "id": "org-uuid-1",
  "name": "Coffee Cooperative Network",
  "type": "cooperative",
  "description": "Supporting small-scale coffee farmers",
  "district": { "id": "district-uuid-1", "name": "Gatsibo" },
  "website": "https://coffeecoop.rw",
  "contactEmail": "info@coffeecoop.rw",
  "contactPhone": "+250791234567",
  "verificationStatus": "verified",
  "verifiedAt": "2024-03-10T15:00:00Z",
  "members": [
    { "id": "user-uuid-1", "firstName": "John", "role": "admin" },
    { "id": "user-uuid-2", "firstName": "Jane", "role": "staff" }
  ],
  "opportunities": 8,
  "totalRewardDistributed": 1250000,
  "createdAt": "2024-01-15T10:00:00Z"
}
```

---

## 8. Admin Endpoints

### 8.1 Import Geographic Data

```http
POST /admin/import
Authorization: Bearer <access-token>
Content-Type: multipart/form-data

{
  "dataType": "districts",
  "file": districts.csv,
  "institution": "Government of Rwanda",
  "datasetName": "Administrative Districts 2024",
  "sourceUrl": "https://www.gov.rw/government/directory/local-government",
  "dateCollected": "2024-09-01",
  "verificationStatus": "authoritative"
}
```

**Response (202 Accepted):**
```json
{
  "importId": "import-uuid-1",
  "status": "processing",
  "dataType": "districts",
  "recordsProcessed": 30,
  "importedAt": "2024-09-05T22:50:00Z"
}
```

---

### 8.2 Assign User Role

```http
PUT /admin/users/user-uuid-1/role
Authorization: Bearer <access-token>
Content-Type: application/json

{
  "role": "district_coordinator",
  "scopeType": "district",
  "scopeId": "district-uuid-1"
}
```

**Response (200 OK):**
```json
{
  "userId": "user-uuid-1",
  "role": "district_coordinator",
  "scopeType": "district",
  "scopeId": "district-uuid-1",
  "assignedAt": "2024-09-05T22:50:00Z"
}
```

---

## 9. Error Responses

### Standard Error Format

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Email is required",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  }
}
```

### Common Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 202 | Accepted (async processing) |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (missing/invalid token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found |
| 429 | Too Many Requests (rate limited) |
| 500 | Server Error |

---

**Last Updated:** 2026-09-05  
**Version:** 1.0  
**Contact:** dev@imena.rw
