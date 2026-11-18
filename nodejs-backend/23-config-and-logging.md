# 23 · Config and Logging

> [!info] Topic info
> **Subject:** Node.js Backend · **Part:** Building Production Backends · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Node.js Backend](README.md) · [Interview questions →](questions/building-backends-qa.md)

> [!abstract] In one line
> Read config from **environment variables** (12-factor), **validate it at startup**, keep secrets out of git, and write **structured JSON logs** (with **pino**) to stdout with **levels** and a **request ID**.

---

## 1. The problem

The DB password is hard-coded in `config.js` and pushed to GitHub. In production, logs are `console.log('here 2')` lines that nobody can search when checkout breaks at 3 a.m.

---

## 2. Config — the 12-factor way

**12-factor app** = a well-known set of rules for cloud apps. For config: the **same build** runs in dev, staging and prod; only **env vars** change.

```js
const { z } = require('zod');
const env = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
}).parse(process.env);          // missing var → crash at boot, not at 3 a.m.
module.exports = env;
```

- **Local:** `.env` file loaded by `dotenv` (or `node --env-file=.env` on Node 20+). Commit `.env.example`, **never** `.env`.
- **Prod:** env vars injected by the platform; real secrets from a secrets manager (AWS Secrets Manager, Vault, K8s Secrets). See [secrets management](../system-design/security/03-secrets-management.md).
- Read `process.env` in **one** config module, not all over the code.
- `NODE_ENV=production` matters — Express and many libs turn off debug overhead.

---

## 3. Structured logging with pino

`console.log` is synchronous to a terminal, has no levels and prints plain text. **pino** writes fast JSON lines that log tools (ELK, Loki, Datadog) can filter. Background: [logging](../system-design/observability/02-logging.md).

```js
const logger = require('pino')({ level: process.env.LOG_LEVEL || 'info', redact: ['req.headers.authorization', '*.password'] });
app.use(require('pino-http')({ logger, genReqId: (req) => req.headers['x-request-id'] || crypto.randomUUID() }));

req.log.info({ orderId: 42, amount: 500 }, 'order created');
// {"level":30,"time":...,"reqId":"abc-123","orderId":42,"amount":500,"msg":"order created"}
```

- **Levels:** `fatal` > `error` > `warn` > `info` > `debug`. Prod runs at `info`.
- **Request ID** on every log line (and passed to downstream services) so you can follow one request — the base of [distributed tracing](../system-design/observability/05-distributed-tracing.md).
- **Log to stdout**; let Docker / K8s / the agent ship logs. Don't manage files in the app.
- **Redact** passwords, tokens, card numbers.

> [!tip] 🎤 Say it like this
> "All config comes from environment variables, validated with a schema at startup so the app fails fast if something is missing. Secrets come from a secrets manager, never git. For logs I use pino: JSON to stdout, proper levels, a request ID on every line and redaction of sensitive fields, then the platform ships them to something like Loki or Datadog."

> [!failure] Interview traps
> - Committing `.env` or secrets.
> - Different builds per environment instead of different config.
> - Logging full request bodies with passwords or tokens.
> - Writing log files inside containers — lost on restart, fills the disk.

---

## Summary

| Area | Practice |
|---|---|
| **Config** | Env vars, one module, validated at boot |
| **Secrets** | Secrets manager, `.env` only locally |
| **Logs** | pino JSON → stdout, levels, request ID, redaction |

> [!success] Key takeaway
> Fail fast on bad config, and log as if you'll be debugging prod at 3 a.m. — because you will.
