# PBL #41 - Show/Hide Confidential User Information in the UI

Woxsen University - Industry SDLC PBL (React & Node.js internship assignments)
Student: Vidurshan S - 24WU0101140 - B.Tech CSE, Panthers

A small Customer Data Console in which confidential customer data (email, phone, PAN, Aadhaar, card number) is
**masked on the server by default**, can be revealed per field with an eye icon **only by authorised roles**,
needs **password re-authentication** for the most sensitive fields (Aadhaar, card), **hides again after 30 s**,
and writes **every reveal decision to an audit table**.

All data is synthetic (invented names, public test card numbers).

## Run

```bash
npm install
npm run seed      # creates data/pbl41.db with 4 users and 8 synthetic customers
npm run build     # builds the React app into dist/
npm run server    # http://localhost:4200
npm run test:api  # 16 API checks (role matrix, masking, re-auth, audit)
npm run test:ui   # 15 browser checks + screenshots (needs: npx playwright install chromium)
```

Requires Node.js 22+ (uses the built-in `node:sqlite`).

## Test accounts (local, synthetic)

| username   | password       | role       |
|------------|----------------|------------|
| admin      | Admin@Pbl41    | admin      |
| compliance | Comply@Pbl41   | compliance |
| support    | Support@Pbl41  | support    |
| viewer     | Viewer@Pbl41   | viewer     |

## Masking rules

| Field   | Example raw -> masked |
|---------|-----------------------|
| Email   | `ravi.kumar@gmail.com` -> `r****@gmail.com` |
| Phone   | `9876543210` -> `XXXXXX3210` |
| PAN     | `ABCDE1234F` -> `XXXXXX234F` |
| Aadhaar | `234567890123` -> `XXXX-XXXX-0123` |
| Card    | `4111 1111 1111 1111` -> `XXXX-XXXX-XXXX-1111` |

## Role matrix

| Role       | Email  | Phone  | PAN    | Aadhaar           | Card              | Audit log |
|------------|--------|--------|--------|-------------------|-------------------|-----------|
| admin      | reveal | reveal | reveal | reveal + password | reveal + password | yes |
| compliance | reveal | reveal | reveal | reveal + password | 403               | yes |
| support    | reveal | reveal | 403    | 403               | 403               | 403 |
| viewer     | 403    | 403    | 403    | 403               | 403               | 403 |

## API

| Method | Path | Notes |
|--------|------|-------|
| POST | `/api/login` | returns a session token and the role's permissions |
| GET  | `/api/customers` | **masked values only**, for every role |
| POST | `/api/customers/:id/reveal` | body `{ field, password? }`; 200 value + `visibleForSeconds: 30`, 401 re-auth required / wrong password, 403 role not allowed |
| GET  | `/api/audit` | admin and compliance only |
| GET  | `/api/insecure/customers` | **anti-pattern demo, only with `ALLOW_INSECURE_DEMO=1`** - sends raw values to show why CSS-only hiding fails |

## Limitations

- Re-authentication uses the password (no OTP); no lockout / rate limit on wrong passwords.
- Sessions are in server memory; the 30 s auto-hide is enforced in the browser.
- Not production code; demo data and accounts only.
