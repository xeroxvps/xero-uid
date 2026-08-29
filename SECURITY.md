# Security Policy

## Supported version

Security fixes are applied to the latest commit on the `main` branch.

## Reporting a vulnerability

Do not open a public issue containing credentials, cookies, personal data, or exploit details. Contact the repository owner privately through GitHub instead. Include the affected endpoint, reproduction steps, expected impact, and a suggested fix if available.

## Secret handling

- Never commit Facebook cookies, access tokens, passwords, `.env` files, databases, or private keys.
- Store `FB_DEFAULT_COOKIE` only as a secret environment variable in Render.
- Rotate a credential immediately if it is accidentally exposed.
- Use only accounts and data you are authorized to access.

## Deployment safeguards

The production container runs as a non-root user. API requests have a size limit, UID batches are bounded and validated, outbound HTTPS certificates are verified, CORS is restricted to configured origins, and responses include baseline browser security headers.
