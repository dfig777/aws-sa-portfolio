# Lab B — Cognito User Pool + Hosted UI (Domain 1)

## Objective
Implement user authentication using Cognito User Pool and validate login via Hosted UI with an OAuth 2.0 authorization code flow.

## What I built
- Cognito **User Pool**: `<USER_POOL_NAME>`
- **App client** for SPA: `<APP_CLIENT_NAME>`
- Hosted UI **domain** configured
- Callback/sign-out URLs for local dev: `http://localhost:3000`

## Architecture context
User → Cognito Hosted UI → (tokens) → SPA (localhost) → (next) API Gateway authorizer → Lambda → DynamoDB

## Key decisions (what recruiters care about)
- **Hosted UI**: reduces risk vs building auth screens/flows manually; standard OAuth flow.
- **Authorization code grant**: preferred for SPAs over legacy implicit flow.
- **User Pool vs Identity Pool**:
  - User Pool = authentication (who the user is)
  - Identity Pool = AWS credentials (temporary STS) when a client must access AWS resources directly (later, if needed)

## Configuration (public-safe)
- OAuth flow: Authorization code
- Scopes: `openid`, `email`, `profile`
- Callback URL: `http://localhost:3000`
- Sign-out URL: `http://localhost:3000`

## Validation
- Hosted UI loads
- Can sign in with a test user
- Redirect back to `http://localhost:3000` succeeds

## Common errors / fixes
- Redirect mismatch: callback URL must match exactly (scheme/host/port/path)
- HTTPS vs HTTP locally: use `http://localhost:3000` for dev unless you set up HTTPS
- Nothing running on localhost: start the dev server (`npm start`) before testing

## Evidence
Screenshots stored locally only (not published).
