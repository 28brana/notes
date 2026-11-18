# Q&A · Security

[← Back to Security](../README.md)

> [!tip] How to use
> Read the question, answer it **out loud**, then click to open the answer.
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

---

## 01 · SSL / TLS

[Read the notes →](../01-ssl-tls.md)

> [!question]- 🟢 Q1. What does TLS give you?
> Three things: encryption so nobody in the middle can read the data, integrity so any change is detected, and authentication — the certificate proves the server is who it claims to be. It protects data in transit only.

> [!question]- 🟡 Q2. Why does TLS use both asymmetric and symmetric encryption?
> Asymmetric crypto is slow, so it's used only in the handshake to agree on a shared session key, usually with Diffie-Hellman so the key never travels on the wire. All the real data is then encrypted with that fast symmetric key, like AES. TLS 1.3 does this in one round trip.

> [!question]- 🟡 Q3. Where would you terminate TLS, and what is mTLS?
> Usually at the load balancer or API gateway — certs live in one place and the LB can route on HTTP. If internal traffic must also be encrypted, re-encrypt to the backends or use passthrough. Between services I'd use mTLS, where both sides present certificates, often handled by a service mesh.

> [!question]- 🔴 Q4. At midnight every client suddenly fails to connect with TLS errors, but servers are healthy. What happened and how do you prevent it?
> Most likely a certificate expired, or a new cert was deployed without its intermediate chain. Fix by renewing or deploying the full chain. Prevent it by automating renewal, for example with Let's Encrypt or a managed cert service, and alerting 30 days before expiry.

---

## 02 · Encryption at Rest

[Read the notes →](../02-encryption-at-rest.md)

> [!question]- 🟢 Q5. What's the difference between disk encryption, TDE and field-level encryption?
> Disk encryption and TDE protect against stolen disks or copied files, but anyone who can query the DB sees plain data. Field-level encryption is done in the app, so the DB, its admins and any leaked dump only see ciphertext — at the cost of not being able to search or index those fields easily.

> [!question]- 🟡 Q6. Explain envelope encryption.
> Data is encrypted with a data key, and the data key is encrypted with a master key that never leaves the KMS. We store the encrypted data key next to the data. This keeps big data away from the KMS, makes master key rotation cheap, and gives an audit log of every key use.

> [!question]- 🔴 Q7. A DB backup was found in a public bucket. It was encrypted with a key stored in the app's config file in the same repo. Are you safe?
> No — if the repo or config is also exposed, the key is useless protection. Rotate the key and credentials, treat the data as leaked, and move keys into a KMS so the data and key are never stored together. Also lock down bucket policies.

---

## 03 · Secrets Management

[Read the notes →](../03-secrets-management.md)

> [!question]- 🟢 Q8. Where should a service's DB password live?
> In a secrets manager like Vault or AWS Secrets Manager, not in code or a `.env` file in git. The service authenticates with its own identity, like an IAM role, and gets only the secrets its policy allows, at runtime.

> [!question]- 🟡 Q9. What are short-lived or dynamic credentials and why use them?
> Instead of one long-lived password, the vault creates a fresh credential on request that expires in, say, an hour. If it leaks, it soon becomes useless, and each service has its own credential so access is easy to trace and revoke.

> [!question]- 🔴 Q10. A developer pushed an AWS key to a public GitHub repo and deleted the commit 10 minutes later. Is that enough?
> No. Bots scan GitHub within minutes, and history, forks and clones keep the key. Revoke and rotate the key immediately, check the audit logs for misuse, and add secret scanning in CI plus a vault so keys aren't in code at all.

---

## 04 · Password Storage

[Read the notes →](../04-password-storage.md)

> [!question]- 🟢 Q11. Why hash passwords instead of encrypting them?
> We never need the original password back — we only compare. Encryption is reversible, so if the key leaks with the DB every password is exposed. A hash is one-way, so there's no key to steal.

> [!question]- 🟡 Q12. Why not SHA-256 with a salt?
> The salt stops rainbow tables, but SHA-256 is designed to be fast — a GPU can try billions per second. bcrypt, scrypt or Argon2 are deliberately slow and memory-heavy, so each guess costs around 100 ms. That makes brute force practically impossible for decent passwords.

> [!question]- 🔴 Q13. Your login service CPU hits 100% after switching to Argon2. What do you do?
> That's the expected cost of slow hashing — don't make the hash fast again. Scale the auth service separately, tune the cost parameters to around 100–300 ms, add rate limiting and lockouts to stop credential-stuffing traffic, and use sessions or tokens so users don't re-enter passwords on every request.

---

## 05 · RBAC

[Read the notes →](../05-rbac.md)

> [!question]- 🟢 Q14. What is RBAC?
> Role-Based Access Control: permissions are grouped into roles like admin, editor, viewer, and users get roles. To change what someone can do, you change their role, not dozens of individual permissions. It's simple and easy to audit.

> [!question]- 🟡 Q15. RBAC vs ABAC vs ACL — when would you use each?
> RBAC for broad job-based access like admin panels. ABAC when rules depend on attributes or context, like "only the owner can edit this post" or "only from the office network". ACLs for per-object sharing, like sharing a single document with specific people. Real systems mix them.

> [!question]- 🔴 Q16. A fired admin could still call admin APIs for an hour. Why, and how do you fix it?
> Their role was probably inside a long-lived JWT, which stays valid until it expires. Use short-lived access tokens with refresh, check a revocation list or the role store for sensitive actions, and always enforce permissions on the server.

---

## 06 · SAML

[Read the notes →](../06-saml.md)

> [!question]- 🟢 Q17. How does SAML SSO work?
> The app (service provider) redirects the user to the company's identity provider. The user logs in there, and the IdP returns a signed XML assertion with their identity. The app verifies the signature with the IdP's certificate and logs them in.

> [!question]- 🟡 Q18. SAML or OIDC for a new product?
> OIDC for new apps — it's JSON and JWT-based, built on OAuth 2.0, and works well for mobile and APIs. But I'd still support SAML if we sell to enterprises, because many corporate identity providers and IT teams expect it.
