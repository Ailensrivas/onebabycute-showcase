# One Baby Cute — Showcase

A personalized-products shop for children, built end to end as a solo project with [Claude Code](https://claude.com/claude-code): custom storybooks, stickers, digital cards and invitations, and coloring books, where a child's illustrated avatar is placed into ready-made artwork.

Live: **[onebabycute.com](https://onebabycute.com)** · App: **[app.onebabycute.com](https://app.onebabycute.com)**

> The source code is private because this is a production product. This repository documents what it does, how it is built, and the decisions behind it. I'm happy to walk through the code on a call.

---

## What it does

1. **Create an avatar.** The customer picks a face (or uploads an optional photo) and the app generates a cartoon avatar of their child.
2. **Preview for free.** A short sample of the book is rendered with that avatar so the customer sees the result before paying.
3. **Pay.** Stripe Checkout creates the order; a signed webhook marks it paid.
4. **Compose and deliver.** Only after payment, the full product is composed and delivered by email as a secure, expiring link (digital) or sent to a print-on-demand partner (physical).

Products: personalized storybooks (digital and print), custom stickers, digital cards and invitations, and coloring books. A second sales channel (Etsy) is served by a small admin tool that produces the same files without the web checkout.

## Architecture

Two separate services, on purpose:

```
┌──────────────────────────┐        ┌────────────────────────────────┐
│  Web  (Next.js 16)       │  HTTPS │  API  (Express, Node 24)       │
│  React 19 · TypeScript   │ ─────► │  Orders · Stripe · AI images   │
│  Tailwind 4              │        │  PDF / print files · Email     │
│  Book viewer, shop, SEO  │        │  SQLite (node:sqlite)          │
└──────────────────────────┘        └───────────────┬────────────────┘
                                                    │
              OpenAI · Gemini · Stripe · Resend · print-on-demand API
```

- **Web:** storefront, story pages, a page-flip book viewer, the checkout flow and order pages.
- **API:** everything that costs money or needs secrets — image generation, payments, order state, file composition, email, print files.
- Each service has its own Dockerfile and is deployed from GitHub to a VPS.

## Tech stack

| Area | Tools |
| --- | --- |
| Frontend | Next.js 16 (App Router), React 19, TypeScript, Tailwind CSS 4, page-flip |
| Backend | Node.js 24, Express, `node:sqlite`, multer |
| Payments | Stripe Checkout + signature-verified webhooks |
| AI | OpenAI (text, images) and Gemini image models for the child's avatar |
| Images & print | sharp, pdfkit, opentype.js (title lettering), ffmpeg for book videos |
| Email | Resend (verified domain) |
| Deploy | Docker, VPS, DNS split between landing page and app subdomains |

## Engineering decisions worth a look

- **Draw once, personalize many times.** Books are illustrated a single time. For each customer only the child's avatar is composited into the existing artwork, which keeps AI cost per order low and the output consistent.
- **Free previews can't drain the AI budget.** Generation endpoints are rate-limited per IP, per hour and per day, with a global cap, because a per-IP limit alone doesn't protect against traffic from many addresses.
- **Payment is trusted only from Stripe.** Orders are marked paid by the signature-verified webhook, never by the browser's redirect. The expensive full-book composition runs only after that.
- **Expiring, signed links.** Previews and downloads use HMAC-signed URLs with an expiry instead of guessable IDs.
- **Recoverable fulfilment.** Order dispatch can be retried from an admin route, so a failed email or print step doesn't lose a paid order.
- **Print-ready output.** Files are generated at a fixed print resolution and page size, with cover title lettering kept inside the printer's safe zone by code.
- **Secrets stay out of git.** Configuration is through environment variables with a documented `.env.example`; test-mode Stripe keys are used in development.

## Built with Claude Code

I used Claude Code as the main development tool: planning features, writing and refactoring both services, debugging deploy problems, and generating validation scripts (for example checks that every story and catalog entry is complete before it ships). Project conventions live in instruction files in the repos, and long-lived decisions are recorded so each session starts with the right context.

## Scale

Roughly 11,000 lines of JavaScript in the API and 12,000 lines of TypeScript in the web app, plus build and print-pipeline scripts.

## Contact

[github.com/Ailensrivas](https://github.com/Ailensrivas)
