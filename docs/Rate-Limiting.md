# Rate Limiting

**Rate limiting** caps how many requests a **user** or **client** can send to a server within a set time window.

---

## Real-World Analogy

Imagine a café with one rule: **max 2 coffees per hour, per person.**

If one customer orders 50 coffees back-to-back, the barista gets overwhelmed and everyone else is stuck waiting. Capping orders keeps the line fair and moving.

---

## In Real Software

Your **server** is the barista, and **HTTP requests** are coffee orders. Without rate limiting, a buggy client, scraper, or attacker can flood your API and slow down or crash the system.

When a client hits the limit, the server blocks extra requests and sends back:
- **`429 Too Many Requests`** — status code saying "slow down"
- **`Retry-After`** — header telling the client how long to wait

Limits are tracked by **User ID** (logged-in users) or **IP address** (anonymous traffic). Common uses:
- **Login endpoints:** Stop brute-force password guessing.
- **Public/read endpoints:** Stop scrapers or accidental infinite loops from overloading the database.

---

## Types of Rate Limiting

### 1. Fixed Window

<img src="https://github.com/user-attachments/assets/59a198d0-6873-49eb-9e40-4c1744497145" alt="Fixed Window Rate Limiting" width="600" />

Splits time into rigid blocks (e.g., `0:00–1:00`, `1:00–2:00`). When the clock hits the next minute, the counter resets to `0`.

- **Pros:** Simple, fast, tiny memory footprint (1 counter per user).
- **Cons:** **Boundary bursts** — a client can send 10 requests at `0:59` and 10 more at `1:01` (**20 requests in 2 seconds**).

---

### 2. Sliding Window

![Http Request_20261122_151729_0000.png](https://github.com/user-attachments/assets/d9fd1aa1-6a03-4437-9032-2730f42a4272)

Instead of resetting on the clock, the server asks: **"How many requests happened in the last 60 seconds from right now?"**

- **Pros:** Eliminates the 2× boundary spike — requests from `0:59` still count at `1:01` and don't expire until `1:59`.
- **Smart optimization:** Saving every exact timestamp burns too much memory at scale. Frameworks like **ASP.NET Core** split the window into smaller segments (e.g., `6 × 10s` buckets) and only refund the oldest bucket as time moves forward.
