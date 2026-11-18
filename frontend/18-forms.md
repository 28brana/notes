# 18 · Forms in React

> [!info] Topic info
> **Subject:** Frontend · **Part:** React & Performance · **Priority:** ⚡ Medium · **Status:** ✅ done · **Last reviewed:** 2026-11-19
>
> [← Back to Frontend](README.md) · [Interview questions →](questions/react-and-performance-qa.md)

> [!abstract] In one line
> React forms are built with **controlled** or **uncontrolled** inputs; for real apps a library like **react-hook-form** plus a **schema validator** (Zod) handles values, validation, errors and submission with few re-renders.

---

## 1. The problem

A sign-up form with 10 fields. With one `useState` per field you write 10 `onChange` handlers, the whole form re-renders on every keystroke, and validation logic gets scattered everywhere.

---

## 2. Controlled vs uncontrolled — quick recap

```jsx
// Controlled: React state is the source of truth
<input value={email} onChange={e => setEmail(e.target.value)} />
// Uncontrolled: DOM keeps the value, read it on submit
<input name="email" defaultValue="" ref={emailRef} />
```

- **Controlled** — instant validation, formatting (e.g. card number spaces), disabling the button. Cost: re-render per keystroke.
- **Uncontrolled** — fewer re-renders, simpler; read values with `ref` or `new FormData(e.target)`.

See [React fundamentals](14-react-fundamentals.md).

---

## 3. react-hook-form + Zod

**react-hook-form** registers inputs as **uncontrolled** (refs), so typing doesn't re-render the whole form. **Zod** describes the shape once and gives you validation **and** TypeScript types.

```jsx
const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});
const { register, handleSubmit, formState: { errors, isSubmitting } } =
  useForm({ resolver: zodResolver(schema) });

<form onSubmit={handleSubmit(onSubmit)}>
  <input {...register('email')} />
  {errors.email && <p role="alert">{errors.email.message}</p>}
</form>
```

> [!example] When to validate
> - **On submit** — always.
> - **On blur** — friendly default: show the error after the user leaves the field.
> - **On change** — after the first error, so it clears as soon as it's fixed.

> [!important] Golden rule
> Client validation is for **UX**. The server must **validate again** — anyone can bypass the browser with curl.

---

## 4. Good form UX checklist

- Real `<label htmlFor>` for every input, errors linked with `aria-describedby`.
- Disable the submit button while `isSubmitting` to stop **double submits**.
- Keep the user's input on server errors; show field-level messages from the API.
- Use correct `type` (`email`, `tel`) for mobile keyboards and autofill.

> [!note] React 19
> React 19 adds **form Actions**: `<form action={fn}>` plus `useActionState` (pending state + result) and `useFormStatus`. Useful with Server Components / Next.js server actions.

> [!tip] 🎤 Say it like this
> "For small forms I use controlled inputs. For bigger ones I use react-hook-form, which keeps inputs uncontrolled so typing doesn't re-render the whole form, with a Zod schema for validation and types. I validate on blur and submit for UX, and the server always validates again."

> [!failure] Interview traps
> - Switching an input from uncontrolled to controlled (`value={undefined}` then a string) — React warns; start with `''`.
> - Relying only on client validation.
> - Forgetting `e.preventDefault()` in a plain `onSubmit` — the page reloads.

---

## Summary

- **Controlled** = control and instant feedback; **uncontrolled** = performance and simplicity.
- **react-hook-form + Zod** is the common production combo.
- Validate on blur/submit; **always re-validate on the server**.

> [!success] Key takeaway
> Use a form library for anything beyond a few fields — it solves validation, errors and re-render cost in one go.
