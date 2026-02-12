# Topic B — Cognito Decision Rules

- Use **Cognito User Pool** when you need user sign-in and tokens (authentication).
- Use **Cognito Identity Pool** when you need to exchange a user identity for **temporary AWS credentials** (STS) to call AWS services directly.
- Prefer **Authorization Code Grant** for modern web apps; avoid implicit unless you have a specific reason.
- Callback/sign-out URLs must match exactly; treat them as part of the security boundary.