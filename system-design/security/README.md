# System Design · Security

[← Back to System Design](../README.md) · [Interview Q&A →](questions/security-qa.md) · [Practice questions 📝](questions/practice-questions.md)

How to protect data in transit and at rest, keep secrets and passwords safe, and control who can do what. Interviewers expect you to mention these in every design, even briefly.

| # | Topic | In one line | Priority |
|---|---|---|---|
| 01 | [SSL / TLS](01-ssl-tls.md) | Encrypts data in transit, proves server identity; termination at LB vs end-to-end, mTLS | 🔥 High |
| 02 | [Encryption at Rest](02-encryption-at-rest.md) | Disk vs TDE vs field-level encryption, envelope encryption with KMS | ⚡ Medium |
| 03 | [Secrets Management](03-secrets-management.md) | Vaults, least-privilege access, rotation, short-lived credentials | ⚡ Medium |
| 04 | [Password Storage](04-password-storage.md) | Slow salted hashes (Argon2, bcrypt), pepper, why not SHA-256 | ⚡ Medium |
| 05 | [RBAC](05-rbac.md) | Permissions via roles; RBAC vs ABAC vs ACL, least privilege | ⚡ Medium |
| 06 | [SAML](06-saml.md) | XML-based enterprise SSO; SAML vs OIDC | 💤 Low |
