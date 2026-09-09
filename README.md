# Karmic Kuisine

**Full-stack food ordering platform** — pizza restaurant with a public-facing marketing site and a server-rendered ordering flow. Two components in two repos: a Next.js marketing site, and a Node.js + Express + MongoDB backend that serves the menu, handles auth, checkout, and pushes real-time order updates over Socket.io.

---

## Architecture

```
                              ┌───────────────────────────────────────┐
                              │   Marketing (karmic-kuisine-web)      │
                              │  Next.js 16 + React 19 + Tailwind v4  │
                              │                                        │
                              │  Hero / Signature Dishes / Our Story  │
                              │  How It Works / Reserve a Table       │
                              │  Framer Motion scroll reveals         │
                              └────────────────┬──────────────────────┘
                                               │
                                               │  "Order Now" → /menu
                                               ▼
                              ┌───────────────────────────────────────┐
                              │    Ordering App (karmic-kuisine)      │
                              │  Node.js + Express + EJS (SSR)        │
                              │                                        │
                              │  - Menu / Cart / Checkout             │
                              │  - Passport (local) auth              │
                              │  - Sessions in Mongo (persistent cart)│
                              │  - Admin order dashboard              │
                              │  - Socket.io: admin status update →   │
                              │    push to customer's order page      │
                              └────────────────┬──────────────────────┘
                                               │
                                               ▼
                                     ┌──────────────────┐
                                     │   MongoDB        │
                                     │   Mongoose ODM   │
                                     │                  │
                                     │   Menu / Users / │
                                     │   Orders /       │
                                     │   Sessions       │
                                     └──────────────────┘
```

## The two repos

| Repo | Purpose | Tech |
|---|---|---|
| [**`karmic-kuisine-web`**](https://github.com/Dev-Harsh0218/karmic-kuisine-web) | Public marketing site — hero, signature dishes, story, reserve-a-table CTA. Deploys to Vercel. | Next.js 16 (App Router, Turbopack), React 19, TypeScript, Tailwind v4, Framer Motion, lucide-react |
| [**`karmic-kuisine`**](https://github.com/Dev-Harsh0218/karmic-kuisine) | The ordering app itself — server-rendered menu, cart, checkout, order tracking, admin dashboard. | Node.js, Express 4, EJS, MongoDB (Mongoose 8), Passport, connect-mongo (session store), Socket.io, bcrypt, Laravel Mix |

## The flow

1. Customer lands on **karmic-kuisine-web** (marketing site) → clicks **Order Now**
2. Redirected to **karmic-kuisine** (`/menu`) → browses dishes served from MongoDB
3. Adds items to cart → cart stored in Mongo-backed session (survives restart)
4. Registers / logs in via Passport local strategy → bcrypt-hashed password
5. `POST /orders` → order written to Mongo, tied to user, redirect to `/orders/:id`
6. Order page opens a **Socket.io** connection subscribed to that order's channel
7. Admin views `/admin/orders`, changes status → `POST /admin/orders/status` emits over Socket.io → customer's page updates live (no polling, no refresh)

## Design decisions worth defending

| Decision | Why |
|---|---|
| **Two repos, not one monorepo** | Marketing is deploy-on-push to Vercel (edge, static, no runtime deps). Ordering app is a stateful Node process behind a load balancer. Different deploy targets → cleaner as separate repos. |
| **SSR (EJS) for the ordering flow, not an SPA** | Menu + cart + checkout doesn't need client-side routing. SSR = simpler ops, faster first paint, better SEO on menu pages, no hydration overhead. |
| **Next.js for the marketing site** | The marketing surface DOES need SEO + fast page loads globally. Next on Vercel gives you edge caching + image optimization without config. |
| **MongoDB over Postgres** | Menu items have variable shape (toppings, size options, allergens per item); orders reference a snapshot of the dish at time-of-order. Document model fits without schema-migration churn per menu change. |
| **Mongo-backed sessions** | Cart survives restart + can scale horizontally behind a load balancer with no sticky sessions. |
| **Passport local strategy** | Well-audited auth surface. Adding Google/Facebook OAuth later is a strategy plugin, not a rewrite. |
| **Socket.io for order tracking** | Order status changes are push-driven from the admin. Polling would be wasteful and add latency. WebSockets give sub-second updates. |

## Live URLs

| Component | URL | Status |
|---|---|---|
| Marketing (web) | _pending deploy_ | Building |
| Ordering app | _localhost only_ | Runs cleanly locally; deploy target TBD (Render / Railway / Fly.io — needs Mongo Atlas) |

## Build status

- ✅ **Platform meta-repo** (this repo)
- ✅ **karmic-kuisine** — ordering app is complete: menu, cart, register/login, order placement, admin dashboard, Socket.io order tracking. History cleaned (secrets purged, junk files removed), proper README + .env.example + .gitignore added.
- 🟡 **karmic-kuisine-web** — Next.js marketing site (in progress)

## Related

Built by [Harsh Bhardwaj](https://github.com/Dev-Harsh0218) — full-stack engineer working across Node.js, Django, React, and TypeScript.
