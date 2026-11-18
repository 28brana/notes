# Security · Practice Questions

[← Back to Security](../README.md) · [Q&A with answers →](security-qa.md)

> [!tip] How to use
> Questions only — no answers. Answer each one **out loud**, then tick it off.
> Stuck? Check the [Q&A file](security-qa.md).
> 🟢 basic · 🟡 intermediate · 🔴 senior / scenario

**18 questions.**

---

## 01 · SSL / TLS

- [ ] 🟢 Q1. What does TLS give you?
- [ ] 🟡 Q2. Why does TLS use both asymmetric and symmetric encryption?
- [ ] 🟡 Q3. Where would you terminate TLS, and what is mTLS?
- [ ] 🔴 Q4. At midnight every client suddenly fails to connect with TLS errors, but servers are healthy. What happened and how do you prevent it?

---

## 02 · Encryption at Rest

- [ ] 🟢 Q5. What's the difference between disk encryption, TDE and field-level encryption?
- [ ] 🟡 Q6. Explain envelope encryption.
- [ ] 🔴 Q7. A DB backup was found in a public bucket. It was encrypted with a key stored in the app's config file in the same repo. Are you safe?

---

## 03 · Secrets Management

- [ ] 🟢 Q8. Where should a service's DB password live?
- [ ] 🟡 Q9. What are short-lived or dynamic credentials and why use them?
- [ ] 🔴 Q10. A developer pushed an AWS key to a public GitHub repo and deleted the commit 10 minutes later. Is that enough?

---

## 04 · Password Storage

- [ ] 🟢 Q11. Why hash passwords instead of encrypting them?
- [ ] 🟡 Q12. Why not SHA-256 with a salt?
- [ ] 🔴 Q13. Your login service CPU hits 100% after switching to Argon2. What do you do?

---

## 05 · RBAC

- [ ] 🟢 Q14. What is RBAC?
- [ ] 🟡 Q15. RBAC vs ABAC vs ACL — when would you use each?
- [ ] 🔴 Q16. A fired admin could still call admin APIs for an hour. Why, and how do you fix it?

---

## 06 · SAML

- [ ] 🟢 Q17. How does SAML SSO work?
- [ ] 🟡 Q18. SAML or OIDC for a new product?
