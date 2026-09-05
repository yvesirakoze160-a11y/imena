# IMENA Security & Compliance

## 1. Authentication & Authorization

### 1.1 User Authentication

**JWT-Based Authentication:**

```
User Login
    ↓
Validate credentials (bcrypt hash comparison)
    ↓
Generate JWT token (15-min expiry)
    ↓
Generate refresh token (7-day expiry, httpOnly cookie)
    ↓
Return access token to client
    ↓
Client stores access token in memory
    ↓
Client includes token in Authorization header
```

**JWT Payload:**

```javascript
{
  sub: "user-uuid",
  email: "user@example.com",
  roles: ["community_member"],
  scope_type: null,
  scope_id: null,
  iat: 1694000000,
  exp: 1694000900  // 15 minutes
}
```

**Password Requirements:**
- Minimum 12 characters
- Mix of uppercase, lowercase, numbers, special characters
- Not dictionary words
- Hashed with bcrypt (12 rounds)
- Never stored in plain text
- Never logged

**Refresh Token Flow:**

```
Access token expires
    ↓
Client sends refresh token (from httpOnly cookie)
    ↓
Backend validates refresh token
    ↓
Generate new access token
    ↓
Return new access token (refresh token reused)
    ↓
If refresh token invalid/expired:
    └─ Force re-login
```

---

### 1.2 Role-Based Access Control (RBAC)

**Roles:**

| Role | Scope | Permissions |
|------|-------|-------------|
| `national_admin` | national | All operations, user management, data import |
| `province_coordinator` | province | Dashboard, opportunity approval, activity verification in province |
| `district_coordinator` | district | Dashboard, opportunity approval, activity verification in district |
| `sector_coordinator` | sector | Activity verification in sector |
| `organization_admin` | organization | Manage organization opportunities, view team members |
| `community_member` | none | Discover opportunities, submit activities, view own wallet |

**Scope Model:**

```javascript
// User with district-level access
{
  userId: "uuid",
  roles: [
    {
      role: "district_coordinator",
      scope_type: "district",
      scope_id: "uuid-of-kigali-city-district"
    }
  ]
}

// User with national access
{
  userId: "uuid",
  roles: [
    {
      role: "national_admin",
      scope_type: null,  // null = national scope
      scope_id: null
    }
  ]
}
```

---

### 1.3 Authorization Middleware

**Every API endpoint enforces authorization:**

```javascript
// Example: Get district dashboard
router.get('/dashboards/district/:districtId', 
  authenticate(),  // Verify JWT
  authorize(['district_coordinator', 'province_coordinator', 'national_admin']),
  validateScope('district', ':districtId'),  // User's scope must match
  async (req, res) => {
    // Only executed if user is authorized for this district
  }
);

// Authorization middleware checks:
1. User is authenticated (valid JWT)
2. User has required role
3. User's scope matches requested resource
4. Log authorization attempt

// If unauthorized:
└─ Return 403 Forbidden
└─ Log security event
└─ No data leaked in response
```

**Scope Validation:**

```javascript
// Only allow users to see data in their scope
// National admin: sees all
// District coordinator: sees only their district
// Community member: sees only their own data (activities, wallet)

function validateScope(scopeType, resourceId) {
  return (req, res, next) => {
    const userScope = req.user.scope;
    
    if (!userScope || scopeType !== userScope.scope_type) {
      return res.status(403).json({ error: 'Access denied' });
    }
    
    if (req.params[resourceId] !== userScope.scope_id) {
      return res.status(403).json({ error: 'Access denied' });
    }
    
    next();
  };
}
```

---

## 2. Data Protection

### 2.1 Sensitive Data Classification

**Sensitive Data:**
- User passwords
- Payment provider API keys
- User phone numbers
- User email addresses
- Financial transaction details
- Citizen location data
- Organization financial information

**Public Data:**
- District names
- Opportunity titles
- Verified activity counts
- Community impact statistics

**Data Handling:**

| Data Type | Storage | Access | Logging |
|-----------|---------|--------|---------|
| Password | Bcrypt hash | Never returned | Not logged |
| API Keys | Environment variables | Backend only | Reference only |
| Email | Encrypted at rest | Authorized users | Minimally logged |
| Transaction | Encrypted at rest | Audit trail | Full transaction logged |
| Location | Not stored (activities only) | Sector-level only | Never in logs |

---

### 2.2 Encryption

**At Rest:**
- Database: Use PostgreSQL encryption (pgcrypto extension)
- Sensitive fields: AES-256 encryption

```sql
-- Encrypt sensitive data
ALTER TABLE users ADD COLUMN phone_encrypted BYTEA;

-- Store encrypted phone
UPDATE users SET phone_encrypted = pgp_sym_encrypt(phone, 'secret-key');
DROP COLUMN phone;

-- Query encrypted data (in application code, not SQL)
SELECT pgp_sym_decrypt(phone_encrypted, 'secret-key') FROM users;
```

**In Transit:**
- HTTPS enforced (TLS 1.2+)
- No sensitive data in URLs
- No credentials in logs

---

### 2.3 No Location Data Exposure

**Private by Design:**

```javascript
// ❌ NEVER expose citizen locations
GET /api/activities/activity-123
// BAD response:
{
  id: 'activity-123',
  userId: 'user-123',
  latitude: -1.9536,  // ❌ Exposes exact location
  longitude: 29.8739,
  description: 'Coffee farming'
}

// ✅ ALWAYS aggregate or anonymize
GET /api/dashboards/district/kigali-city
{
  districtId: 'kigali-uuid',
  totalActivities: 42,
  activitiesByCategory: { agriculture: 15, education: 27 },
  // No individual location data exposed
}

// ✅ OK: Sector-level filtering only
GET /api/dashboards/sector/kicukiro-sector
{
  sectorId: 'sector-uuid',
  activitiesInSector: 8
}
```

---

## 3. Financial Security

### 3.1 Payment Integration Safety

**Never:**
- ❌ Store payment provider credentials in code
- ❌ Accept real payment in test mode
- ❌ Hard-code transaction responses
- ❌ Store user payment details
- ❌ Expose payment provider internal references

**Always:**
- ✅ Validate webhook signatures from payment provider
- ✅ Verify payment status before updating wallet
- ✅ Log all payment attempts (provider reference only)
- ✅ Use encrypted credentials from environment
- ✅ Implement retry logic with exponential backoff

---

### 3.2 Transaction Ledger

**Ledger Design (Immutable):**

```sql
CREATE TABLE ledger_entries (
  id UUID PRIMARY KEY,
  wallet_id UUID REFERENCES wallets(id),
  transaction_id UUID REFERENCES transactions(id),
  
  debit_rwf DECIMAL(12, 2) DEFAULT 0,
  credit_rwf DECIMAL(12, 2) DEFAULT 0,
  balance_after_rwf DECIMAL(15, 2),
  
  entry_type VARCHAR(50), -- reward, withdrawal, refund
  reference TEXT, -- transaction reference
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Immutable: No updates, only inserts
-- Audit trail: Every transaction creates entry
-- Reconciliation: Balance verified on each entry
```

**Sample Ledger:**

| ID | Wallet ID | Debit | Credit | Balance | Type | Reference | Date |
|----|-----------|-------|--------|---------|------|-----------|------|
| 1 | wallet-1 | 0 | 5000 | 5000 | reward | activity-123 | 2024-01-01 |
| 2 | wallet-1 | 2000 | 0 | 3000 | withdrawal | mtn-momo-tx-456 | 2024-01-02 |
| 3 | wallet-1 | 0 | 3000 | 6000 | refund | activity-789-rejected | 2024-01-03 |

**Validation:**
- Each entry must have debit OR credit (not both)
- Balance integrity check: previous_balance + credit - debit = new_balance
- No negative balances (enforce in application)

---

### 3.3 Payment Provider Integration

**Architecture:**

```javascript
// PaymentProvider interface
class PaymentProvider {
  async initiateTransaction(params) {
    // Returns: { transactionId, status, providerReference }
  }
  
  async verifyTransaction(providerReference) {
    // Returns: { status, amount, timestamp }
  }
  
  async handleWebhook(payload, signature) {
    // Verify signature
    // Parse payload
    // Return transaction status
  }
}

// Implementation
class MTNMoMoProvider extends PaymentProvider {
  constructor(apiKey, apiSecret, merchantAccountId) {
    this.apiKey = apiKey; // From environment
    this.apiSecret = apiSecret; // From environment
    this.merchantAccountId = merchantAccountId; // From environment
  }
  
  async initiateTransaction({ amount, phoneNumber, reference }) {
    const payload = {
      amount,
      currency: 'RWF',
      externalId: reference,
      partyId: phoneNumber,
      partyIdType: 'MSISDN',
      description: 'IMENA Community Activity Reward'
    };
    
    const response = await fetch('https://api.mtn-momo.rw/v1/requesttopay', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${await this.getToken()}`,
        'X-Reference-Id': generateUUID(),
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    });
    
    if (response.status !== 202) {
      throw new PaymentInitiationError('MTN MoMo API error');
    }
    
    return {
      transactionId: generateUUID(),
      status: 'pending',
      providerReference: response.headers['X-Reference-Id']
    };
  }
}

// Usage
const paymentProvider = new MTNMoMoProvider(
  process.env.MTN_MOMO_API_KEY,
  process.env.MTN_MOMO_API_SECRET,
  process.env.MTN_MOMO_MERCHANT_ID
);

// Initiate payment
const transaction = await paymentProvider.initiateTransaction({
  amount: 5000,
  phoneNumber: '+250791234567',
  reference: 'activity-123-reward'
});

// Store transaction with provider reference (for webhook matching)
await db.transactions.create({
  wallet_id: walletId,
  amount_rwf: 5000,
  transaction_type: 'reward',
  status: 'pending',
  provider_reference: transaction.providerReference,
  provider_response: { initiationId: transaction.transactionId }
});
```

**Webhook Verification:**

```javascript
// Payment provider sends webhook
app.post('/webhooks/mtn-momo', async (req, res) => {
  const signature = req.headers['X-Signature'];
  const payload = req.body;
  
  // Verify signature
  const expectedSignature = crypto
    .createHmac('sha256', process.env.MTN_MOMO_API_SECRET)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  if (signature !== expectedSignature) {
    logger.warn('Invalid webhook signature', { payload });
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Find transaction by provider reference
  const transaction = await db.transactions.findOne({
    provider_reference: payload.externalId
  });
  
  if (!transaction) {
    logger.error('Transaction not found', { externalId: payload.externalId });
    return res.status(404).json({ error: 'Transaction not found' });
  }
  
  // Update transaction status
  const status = payload.status === 'successful' ? 'completed' : 'failed';
  await db.transactions.update(transaction.id, { status });
  
  // Update wallet if successful
  if (status === 'completed') {
    await db.wallets.increment(transaction.wallet_id, {
      balance_rwf: transaction.amount_rwf
    });
    
    // Create ledger entry
    await db.ledger_entries.create({
      wallet_id: transaction.wallet_id,
      transaction_id: transaction.id,
      credit_rwf: transaction.amount_rwf,
      balance_after_rwf: updatedWallet.balance_rwf,
      entry_type: transaction.transaction_type,
      reference: payload.externalId
    });
  }
  
  // Log for audit
  logger.info('Payment webhook processed', {
    transactionId: transaction.id,
    status,
    amount: transaction.amount_rwf
  });
  
  res.json({ success: true });
});
```

---

## 4. Audit Logging

### 4.1 Security Events

**Log Every:**
- User login/logout
- Failed authentication attempts
- Authorization failures (access denied)
- Password changes
- Role/permission changes
- Data export requests
- Admin actions (data import, user management)
- Payment attempts and status changes
- Sensitive data access

**Example Audit Log:**

```javascript
{
  timestamp: '2024-09-05T22:50:00Z',
  event_type: 'payment_initiated',
  user_id: 'user-123',
  user_email: 'jane@example.com',
  action: 'initiate_payment',
  resource_id: 'activity-456',
  amount_rwf: 5000,
  payment_provider: 'mtn_momo',
  provider_reference: 'mtn-ref-789',
  status: 'success',
  ip_address: '192.168.1.1',
  user_agent: 'Mozilla/5.0...'
}
```

**Never Log:**
- ❌ Passwords
- ❌ API keys
- ❌ Payment provider secrets
- ❌ Sensitive personal data (full phone, full SSN)
- ❌ Citizen exact locations

---

### 4.2 Audit Table

```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY,
  timestamp TIMESTAMP NOT NULL DEFAULT NOW(),
  
  event_type VARCHAR(100),
  action VARCHAR(100),
  
  user_id UUID,
  user_email VARCHAR(255),
  
  resource_type VARCHAR(100),
  resource_id UUID,
  
  status VARCHAR(50), -- success, failure
  error_message TEXT,
  
  ip_address INET,
  user_agent TEXT,
  
  details JSONB,
  
  created_at TIMESTAMP DEFAULT NOW()
);

-- Index for fast queries
CREATE INDEX idx_audit_timestamp ON audit_logs(timestamp DESC);
CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_event ON audit_logs(event_type);
```

---

## 5. Credential Management

### 5.1 Environment Variables

**.env File (Development Only):**

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/imena_dev

# JWT
JWT_SECRET=your-secret-key-min-32-chars
REFRESH_TOKEN_SECRET=your-refresh-secret-key-min-32-chars

# Payment Providers
MTN_MOMO_API_KEY=your-mtn-api-key
MTN_MOMO_API_SECRET=your-mtn-api-secret
MTN_MOMO_MERCHANT_ID=your-merchant-id

AIRTEL_MONEY_API_KEY=your-airtel-api-key
AIRTEL_MONEY_API_SECRET=your-airtel-api-secret

# Email/SMS
EMAIL_PROVIDER=sendgrid
SENDGRID_API_KEY=your-sendgrid-key

SMS_PROVIDER=twilio
TWILIO_ACCOUNT_SID=your-twilio-sid
TWILIO_AUTH_TOKEN=your-twilio-token

# Encryption
AES_ENCRYPTION_KEY=your-32-char-encryption-key

# Environment
NODE_ENV=development
PORT=3000
```

**.env.example (Committed to Git):**

```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/imena_dev

# JWT
JWT_SECRET=your-secret-key-min-32-chars
REFRESH_TOKEN_SECRET=your-refresh-secret-key-min-32-chars

# Payment Providers
MTN_MOMO_API_KEY=your-mtn-api-key
MTN_MOMO_API_SECRET=your-mtn-api-secret
MTN_MOMO_MERCHANT_ID=your-merchant-id

AIRTEL_MONEY_API_KEY=your-airtel-api-key
AIRTEL_MONEY_API_SECRET=your-airtel-api-secret

# Email/SMS
EMAIL_PROVIDER=sendgrid
SENDGRID_API_KEY=your-sendgrid-key

SMS_PROVIDER=twilio
TWILIO_ACCOUNT_SID=your-twilio-sid
TWILIO_AUTH_TOKEN=your-twilio-token

# Encryption
AES_ENCRYPTION_KEY=your-32-char-encryption-key

# Environment
NODE_ENV=development
PORT=3000
```

**In Production:**
- Use secret manager (AWS Secrets Manager, HashiCorp Vault)
- Rotate credentials regularly
- Never commit `.env` to Git
- Use separate credentials per environment

---

### 5.2 Credential Validation

```javascript
// Startup validation
function validateEnvironment() {
  const required = [
    'DATABASE_URL',
    'JWT_SECRET',
    'REFRESH_TOKEN_SECRET',
    'AES_ENCRYPTION_KEY',
    'NODE_ENV'
  ];
  
  const optional = [
    'MTN_MOMO_API_KEY',  // May not be configured in development
    'AIRTEL_MONEY_API_KEY'
  ];
  
  // Check required
  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
  
  // Warn if optional not configured
  for (const key of optional) {
    if (!process.env[key]) {
      logger.warn(`Optional environment variable not configured: ${key}`);
    }
  }
  
  // Validate values
  if (process.env.JWT_SECRET.length < 32) {
    throw new Error('JWT_SECRET must be at least 32 characters');
  }
  
  logger.info('Environment validation passed');
}

validateEnvironment();
```

---

## 6. HTTPS & Transport Security

### 6.1 SSL/TLS Configuration

**Production:**
- HTTPS enforced (redirect HTTP → HTTPS)
- Certificate from trusted CA (Let's Encrypt)
- TLS 1.2+ only
- HSTS header enabled

```javascript
// Force HTTPS
app.use((req, res, next) => {
  if (process.env.NODE_ENV === 'production' && req.header('x-forwarded-proto') !== 'https') {
    return res.redirect(`https://${req.header('host')}${req.url}`);
  }
  next();
});

// HSTS header
app.use((req, res, next) => {
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  next();
});
```

---

## 7. Data Privacy (GDPR-like compliance)

### 7.1 User Data Export

**User can request data export:**

```javascript
POST /api/users/me/export
// Request all personal data in machine-readable format (JSON)

// Response:
{
  profile: {
    id, email, phone, firstName, lastName, ...
  },
  activities: [
    { id, title, status, rewards, ... }
  ],
  transactions: [
    { id, amount, status, date, ... }
  ],
  auditLog: [
    { timestamp, event, ... }
  ]
}
```

---

### 7.2 User Data Deletion

**User can request deletion:**

```javascript
POST /api/users/me/delete
// Request account deletion

// Process:
1. Anonymize personal data (name, email, phone → hash)
2. Keep activities/contributions (not associated with user)
3. Delete auth credentials
4. Send confirmation email
5. Log deletion event
6. Retain transaction history (financial compliance)
```

---

## 8. API Security

### 8.1 Rate Limiting

```javascript
// Authentication endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts, please try again later'
});

app.post('/api/auth/login', authLimiter, loginHandler);

// Payment endpoints
const paymentLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 1, // 1 payment per minute per user
});

app.post('/api/payments/initiate', paymentLimiter, initiatePaymentHandler);

// General API
const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100 // 100 requests per minute
});

app.use('/api', apiLimiter);
```

---

### 8.2 Input Validation

```javascript
// All inputs validated and sanitized
const Joi = require('joi');

const registerSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string()
    .min(12)
    .pattern(/[A-Z]/).pattern(/[a-z]/).pattern(/[0-9]/).pattern(/[^A-Za-z0-9]/)
    .required(),
  firstName: Joi.string().max(100).required(),
  lastName: Joi.string().max(100).required(),
  phoneNumber: Joi.string().pattern(/^\+250[0-9]{9}$/).required(), // Rwanda format
  preferredLanguage: Joi.string().valid('en', 'rw', 'fr').default('en')
});

app.post('/api/auth/register', async (req, res) => {
  const { error, value } = registerSchema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }
  // Process validated input
});
```

---

## 9. Compliance Checklist

### 9.1 Data Protection

- [ ] HTTPS enforced in production
- [ ] Passwords hashed with bcrypt
- [ ] Sensitive data encrypted at rest
- [ ] No citizen location data exposed
- [ ] Audit logging implemented
- [ ] User data export enabled
- [ ] User data deletion enabled
- [ ] Privacy policy published (KN, EN, FR)

### 9.2 Financial Security

- [ ] Payment credentials in environment variables only
- [ ] Webhook signatures verified
- [ ] Ledger immutable and audited
- [ ] Transaction status verified before wallet update
- [ ] No fake payment responses in production
- [ ] Rate limiting on payment endpoints
- [ ] RRA compliance (if applicable)

### 9.3 Authorization

- [ ] Authentication required for all endpoints
- [ ] Scope-based access control enforced
- [ ] Authorization logged
- [ ] Admin actions logged
- [ ] Regular security audit planned

---

**Last Updated:** 2026-09-05  
**Next Review:** Before production deployment  
**Security Officer:** Development Team Lead
