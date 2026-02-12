# Domain 1 — Topic B (Cognito): SPA Login + JWT-Protected API (HTTP API) + Lambda

## Outcome (what I proved)
I implemented an end-to-end authentication flow that mirrors a real-world pattern:

**Browser SPA → Cognito Hosted UI → OAuth2 Authorization Code → JWT tokens → API Gateway HTTP API (JWT Authorizer) → Lambda**

This proves I can design a secure authentication boundary where:
- Users authenticate through a managed identity provider (Cognito).
- The frontend never needs long-lived AWS credentials.
- The backend only executes if a valid JWT is presented.
- The API is protected by a centralized authorization layer (API Gateway authorizer), not by “trusting the client.”

---

## Why this matters in Domain 1 (Design Secure Architectures)
Domain 1 focuses heavily on:
- **Identity and access management design**
- **Secure authentication and authorization patterns**
- **Least privilege**
- **Avoiding long-lived credentials**
- **Defense in depth (multiple layers enforce security)**

This build demonstrates those by separating concerns cleanly:
- **AuthN (authentication)**: Cognito verifies user identity.
- **AuthZ (authorization)**: API Gateway validates tokens and blocks unauthenticated callers.
- **Execution permissions**: Lambda uses an IAM execution role (temporary credentials) to access AWS services (later).

---

## High-level architecture
### Request flow (what happens at runtime)
1) User clicks **Login** in the SPA.
2) Browser is redirected to **Cognito Hosted UI**.
3) User signs in (Cognito validates identity).
4) Cognito redirects back to SPA with an **authorization code**.
5) SPA exchanges the **code** for tokens:
   - **access_token** (for calling APIs)
   - **id_token** (identity claims, UI-level)
   - **refresh_token** (optional, session continuation)
6) SPA calls `GET /whoami` with:
   - `Authorization: Bearer <access_token>`
7) API Gateway validates JWT (issuer + audience). If valid → invokes Lambda.
8) Lambda returns a small response proving the request was authenticated.

---

## Build process (start → finish), with reasoning

### 1) Create Cognito User Pool (the identity system)
**What I did**
- Created a Cognito **User Pool**: `<user_pool_name>`
- Chosen because I need a managed user directory and token issuance.

**Why**
- User Pools provide authentication features (sign-in, MFA options, password policy, hosted login).
- In Domain 1 terms: I’m using a managed identity service to reduce risk and centralize auth controls.

**Domain 1 tie-in**
- “Choose appropriate authentication mechanisms.”
- “Reduce operational risk by using managed services for identity.”

---

### 2) Create an App Client for the SPA (who is allowed to request tokens)
**What I did**
- Created an **App Client** representing the SPA: `<app_client_name>`
- Selected **SPA / Single-page app** type (public client pattern).

**Why**
- OAuth requires the identity provider to know “which application is asking for tokens.”
- The App Client controls:
  - Allowed OAuth flows (grant types)
  - Allowed redirect/callback URLs
  - Scopes available to the client
- This reduces the chance of token issuance to unknown or misconfigured apps.

**Domain 1 tie-in**
- “Establish least privilege and controlled access boundaries.”
- “Use managed identity controls (scopes, callback restrictions).”

---

### 3) Configure Hosted UI + OAuth settings (how login happens)
**What I did**
- Enabled **Managed login pages / Hosted UI** for the App Client.
- Set:
  - **Callback URL(s)**: `http://localhost:3000` (dev-only)
  - **Sign-out URL(s)**: `http://localhost:3000` (dev-only)
- Chosen OAuth grant:
  - **Authorization Code Grant** (recommended)
- Scopes:
  - `openid` (required for OIDC)
  - `email` (optional; used only if needed)

**Why**
- Authorization Code flow is a standard secure pattern because:
  - The SPA does not handle user passwords directly.
  - Tokens are issued only after successful auth and correct redirect validation.
- Restricting callback URLs prevents open redirect/token leakage to unknown domains.

**Domain 1 tie-in**
- “Prevent common misconfigurations (open redirects).”
- “Use secure OAuth flow choices.”

---

### 4) Create the API boundary: API Gateway HTTP API + JWT Authorizer
**What I did**
- Created an **HTTP API** in API Gateway.
- Created a **JWT authorizer** using:
  - **Issuer**: `https://cognito-idp.<region>.amazonaws.com/<user_pool_id>`
  - **Audience**: `<app_client_id>`
- Attached the authorizer to a protected route:
  - `GET /whoami`

**Why it works (mechanics)**
- When the SPA calls `/whoami` with a Bearer token:
  - API Gateway validates the JWT signature against Cognito’s public keys (JWKS).
  - Checks token `iss` matches the configured issuer.
  - Checks token `aud` matches the configured audience (app client).
- If any check fails → request is blocked before Lambda.

**Domain 1 tie-in**
- “Use centralized enforcement points for authorization.”
- “Implement defense in depth: validation at the edge (API Gateway) rather than trusting backend blindly.”

---

### 5) Create Lambda backend: `/whoami` function
**What I did**
- Created a Lambda function: `<lambda_name>`
- Integrated API Gateway route `GET /whoami` to Lambda.

**Why**
- Lambda is a simple backend compute target for protected APIs.
- The authorizer ensures only authenticated requests reach Lambda.

**Domain 1 tie-in**
- “Compute behind secure front doors.”
- “Don’t expose direct service-to-client access without auth controls.”

---

### 6) IAM: Lambda execution role (ties to Topic A lessons)
**What I did (earlier)**
- Created / used a Lambda execution role (trust + permissions).
- Trust policy allows:
  - `lambda.amazonaws.com` to assume the role via `sts:AssumeRole`

**Why this matters here**
- Even though Topic B focuses on Cognito, the backend still needs AWS permissions to do real work.
- Lambda never uses long-lived access keys.
- Lambda assumes a role and gets **temporary credentials via STS**.

**This is the direct bridge from Topic A → Topic B**
- Topic A taught:
  - Trust policy = **who** can assume a role
  - Permission policy = **what** can be done
  - Temporary creds reduce exposure vs long-lived keys
- Topic B uses that by ensuring the backend execution environment follows the same “no long-lived keys” rule.

---

### 7) Frontend SPA: React app local dev + login + token usage
**What I did**
- Created a local React app that:
  - Redirects user to Hosted UI login
  - Handles redirect back with `?code=...`
  - Exchanges code → tokens
  - Calls `/whoami` with `Authorization: Bearer <access_token>`

**Why**
- This proves the *real* integration, not just console configuration.
- It demonstrates the exact behavior recruiters expect:
  - Frontend authenticates via OIDC
  - Backend protected route requires JWT

**Security hygiene applied**
- Did not commit tokens.
- Did not print tokens in UI.
- Used redaction / avoided exposing claims in repo artifacts.
- Kept sensitive runtime artifacts local.

**Domain 1 tie-in**
- “Prevent credential leakage.”
- “Use secure client-to-API auth patterns.”

---

### 8) Hardening (security posture improvements)
**What I tightened**
- **CORS** restricted to local dev origin:
  - `http://localhost:3000`
- Kept allowed methods/headers minimal:
  - Methods: `GET`, `OPTIONS`
  - Headers: `Authorization`, `Content-Type`
- Reduced what `/whoami` returns (no PII, no full claims).

**Why**
- CORS restriction reduces browser-based abuse from untrusted origins.
- Minimal response prevents accidental data exposure in logs/screenshots.

**Domain 1 tie-in**
- “Minimize exposure.”
- “Least privilege, least disclosure.”

---

## Verification checklist (what I tested and expected outcomes)
1) **Unauthenticated request**
   - Calling `/whoami` without token → `401 Unauthorized`
2) **Authenticated request**
   - Calling `/whoami` with valid access token → `200 OK`
3) **Tampered token**
   - Modified token → `401 Unauthorized`
4) **CORS**
   - Browser call from non-allowed origin blocked (expected in browser context)

These tests confirm:
- AuthN is working (Cognito issues tokens)
- AuthZ is enforced (API Gateway validates JWT)
- Backend is protected and not directly exposed

---

## Domain 1 concepts demonstrated (mapped)
- **Authentication vs Authorization**
  - Cognito = authentication
  - API Gateway JWT authorizer = authorization gate
- **Defense in Depth**
  - Tokens validated at API Gateway before Lambda runs
- **Least Privilege / Least Disclosure**
  - Minimal claims exposure; minimal API surface (single protected route)
- **No long-lived credentials**
  - SPA never uses AWS keys
  - Lambda uses IAM role + STS temporary creds (Topic A linkage)
- **Secure OAuth selection**
  - Authorization Code grant is standard and reduces risk vs weaker patterns

---

## What this enables next (how it evolves into a “real” app)
This Topic B foundation is required before adding real application behavior:
- Add `POST /users` protected route
- Lambda writes to DynamoDB using least-privilege IAM policy
- Encrypt data with KMS (Topic C)
- Store config in Parameter Store/Secrets Manager securely (Topic C)
- Add monitoring/alerts for auth anomalies (later domains)

