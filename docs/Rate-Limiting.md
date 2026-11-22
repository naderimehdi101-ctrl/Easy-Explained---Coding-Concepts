# Rate Limiting

**Rate limiting** is a way to control how many requests a **user** or **client** can send to a server within a specific time window.

---

## Real-World Analogy

Imagine a café with a strict rule: **maximum 2 coffees per hour per person.**

If one customer walks in and tries to order 50 coffees in a row, the barista gets overwhelmed and every other customer in line is stuck waiting. Capping each person at 2 coffees per hour keeps the café running smoothly and fair for everyone.

---

## In Real Software

In a backend API, the server is the barista and HTTP requests are the coffee orders. Without rate limiting, a single buggy client, scraper, or attacker can flood an endpoint and slow down or crash the entire system.

When a client crosses the allowed limit, the server blocks the extra requests and responds with:
- **HTTP `429 Too Many Requests`**
- A **`Retry-After`** header telling the client how long to wait before trying again

In practice, APIs usually track limits **per User ID** (for logged-in users) or **per IP address** (for anonymous traffic). Common use cases include:
- **Login endpoints:** Stopping brute-force password guessing.
- **Public/read endpoints:** Preventing scrapers or accidental infinite loops from overloading the database.

---

## Types of Rate Limiting

### 1. Fixed Window

<img src="https://github.com/user-attachments/assets/59a198d0-6873-49eb-9e40-4c1744497145" alt="Fixed Window Rate Limiting" width="600" />


Time is split into rigid, fixed intervals (like `0:00–1:00`, `1:00–2:00`). Every time the clock hits the next minute, the counter resets to zero.

- **Pros:** Simple, fast, and uses very little memory (one counter per user).
- **Cons:** Vulnerable to a **boundary burst**—a client can send 10 requests at `0:59` and 10 more at `1:01`, squeezing 20 requests into 2 seconds.

---

### 2. Sliding Window

![Http Request_20261122_151729_0000.png](https://github.com/user-attachments/assets/d9fd1aa1-6a03-4437-9032-2730f42a4272)

Instead of resetting on the clock, the server checks: **"How many requests did this client send in the last 60 seconds from right now?"**

- **Pros:** Eliminates the 2× boundary spike. Requests sent at `0:59` still count at `1:01` and don't expire until `1:59`.
- **How frameworks optimize it:** Storing the exact timestamp of every request takes too much memory at scale. Frameworks like ASP.NET Core split the window into smaller segments (for example, `6 × 10s` buckets) and only refund the oldest segment as time moves forward.
