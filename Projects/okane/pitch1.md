## Project Pitch 

- I built a project called OKANE, which is a scalable digital wallet and peer-to-peer payment platform.
- it is a modern, production-ready financial management platform built with a Turborepo monorepo architecture
-  It enables users to track transactions, and it has automated backend and deployment pipelines.


## 1️⃣ What the project is about

OKANE allows users to authenticate, add money to their wallet, and transfer money peer-to-peer.
The core focus is financial correctness, concurrency safety, and performance optimization using caching.”

*Key capabilities:*

- Secure authentication
- Wallet balance management
- P2P transfers
- Transaction history
- Cache-optimized reads

## 2️⃣ Project structure & architecture

The project is structured as a Turborepo monorepo to cleanly separate concerns and allow shared logic.

*High level structure:*

- user-app → Next.js for frontend  and server actions

- bank-webhook → Express service simulating bank callbacks

- db package → Shared Prisma + PostgreSQL + Redis cache logic

- ui package → Shared UI components

This separation allows me to deploy services independently while reusing core logic safely.

## 3️⃣ How the system works (end-to-end flow)

**🔐 Authentication**

- Implemented using NextAuth
- Supports Google OAuth and credentials
- Session handling integrated directly into server actions

**💰 Add Money (Webhook Flow)**

- When a user adds money, the request is handled by a separate bank-webhook service.

The add-money flow is implemented using an on-ramp transaction model with a dedicated webhook service to mimic real payment gateways.

1. On-Ramp Transaction Creation (user-app)

When a user initiates an add-money action from the frontend, the request is handled by the user-app backend.

Backend steps:
- a new on-ramp transaction record is created in PostgreSQL via Prisma with:
  - userId
  - amount
  - status = PROCESSING
  - a random token

This token acts as:
- an idempotency key
- a secure reference for the bank callback

At this point, no balance mutation happens — we only register intent.

2. Bank Webhook Invocation

- After creating the on-ramp entry, the user-app makes a server-to-server POST request to the bank-webhook service, passing the generated token.

Why separate service?
 - Simulates real-world payment providers
 - Decouples payment confirmation from user-facing API
 - Prevents users from directly manipulating balances

3. Webhook Validation & Lookup (bank-webhook)

- The bank-webhook service validates the request by querying the on-ramp transaction using the provided token

Token lookup ensures:
- Request authenticity
- Idempotency (no duplicate credits)

If:
- transaction doesn’t exist → reject
- status ≠ PROCESSING → reject

4. Atomic Balance Update (Critical Section)

- Once validated, the webhook executes a database transaction to ensure atomicity.

Inside a single Prisma transaction:
 - Update on-ramp transaction:
 - status → SUCCESS
 - Increment user wallet balance
 - Insert a new transaction record (ledger entry)

This guarantees consistency — either all updates happen or none do.

5. Cache Invalidation & Write-Through Update

- After the database commit, cache consistency is handled explicitly.

  - Invalidate Redis key:
    - balance:{userId}

  - Push the new transaction into Redis:
  - keeps recent transaction history warm

- Balance cache will be:
  - lazily rehydrated
  - or proactively warmed via scheduled jobs

- The database remains the source of truth, Redis is purely an acceleration layer.

6. Idempotency & Safety Guarantees

- The token-based lookup ensures that even if the webhook is retried, the balance cannot be credited twice

Guarantees:
- Exactly-once crediting
- No race conditions
- No user-controlled balance mutation

* This mimics how real payment gateways like Stripe or Razorpay work.”

**🔁 P2P Transfers**

- P2P transfers are handled using database transactions and row-level locking to avoid race conditions.

 - Uses SELECT … FOR UPDATE

Ensures:
- No double spending
- Atomic balance updates
- Correct transaction state

“Even if two transfers happen at the same time, the balance remains consistent.”

## 4️⃣ Caching & performance optimization (Key Highlight)

- To optimize read-heavy operations and reduce cold-start latency, I integrated Redis using Upstash as a managed, serverless caching layer.

- What is cached:
  -  User balance
  -  Transaction history
  -  User profile
  -  Contacts

- Strategy:
  - Read from Redis first
  - Fallback to DB
  - Invalidate cache on writes
  - Cache Warming (Advanced)

- To avoid cold starts, I built an automated cache-warming system using GitHub Actions.
  - Runs every hour
  - Preloads Redis with:
    - balances
    - recent transactions
  - This significantly reduces latency and database load.

## 5️⃣ Why these tech choices?

Tech	                     Why I chose it
---
Next.js                 	Full-stack, server actions, great auth integration
TypeScript	                Type safety for financial data
PostgreSQL	                ACID compliance for money
Prisma	                    Safe transactions, schema control
Redis(Upstash)	            Ultra-fast reads, serverless compatible
Turborepo	                Scalable monorepo management
Docker	                    Consistent environments
GitHub Actions	            Automation + cache warming
Vercel / Render          	Best-fit hosting per service

- Every choice was made to match real-world production constraints


## Problems & Errors Faced


* ❌ Problem 1: Race conditions in P2P transfers

 Issue: 

- Simultaneous transfers could read stale balances.

 Solution: 

- Used database transactions
- Applied row-level locking
- Reduced transaction scope to avoid timeouts

✅ Result: No inconsistent balances

* ❌ Problem 2: Prisma transaction timeouts (P2028)

Issue:
- Long transactions caused failures

Solution:

- Minimized work inside transactions

- Moved reads outside

- Cached read-heavy data in Redis

* ❌ Problem 3: Monorepo deployment issues

Issue:

- Vercel couldn’t resolve shared packages

Solution:

- Proper package exports
- Correct workspace configuration
- Explicit build filters in CI

* ❌ Problem 4: Redis failures breaking flow

Issue: 

- Redis unavailable = request failure

Solution:

- Made Redis non-blocking
- DB remains source of truth
- Graceful fallback to DB

* ❌ Problem 5: Cold-start latency

Issue:

- First user request was slow

Solution: 

- GitHub Actions scheduled cache warmer
- Preloaded Redis hourly

---

- This project taught me how real systems are designed — not just writing APIs, but handling correctness, performance, failures, and scale.
- My focus was to think like a backend engineer, not just a feature developer.