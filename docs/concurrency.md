# Concurrency Control

**Concurrency control** prevents data from being accidentally overwritten or corrupted when multiple users try to edit the same record at the same time.

---

## Real-World Analogy

Imagine two gym admins managing the **Basic** membership plan (currently **$29.99**) at the exact same time.

Both open the plan's pricing sheet simultaneously. Admin A decides to run a discount, updates the price to **$20**, and saves it at **3:00 PM**. One minute later at **3:01 PM**, Admin B—who is still looking at the old **$29.99** sheet—raises the price to **$35** and hits save. Admin A's discount is **completely wiped out**, and neither admin realizes it happened.

---

## In Real Software

In a backend API, this is called a **Lost Update**.

<img src="PASTE_NO_CONCURRENCY_IMAGE_URL_HERE" alt="No Concurrency Control - Lost Update" width="600" />

When two requests read the same database row at the same time and both try to save changes, the last write silently wins—overwriting the first user's update.

Common places this breaks real apps:
- **Admin Dashboards:** Two admins editing the same plan price or user profile simultaneously.
- **E-commerce & Inventory:** Two people buying the last item in stock at the exact same second.
- **Banking:** Two withdrawals reading the same balance before either deducts money.

---

## Types of Concurrency Control

### 1. Optimistic Concurrency

<img src="PASTE_OPTIMISTIC_CONCURRENCY_IMAGE_URL_HERE" alt="Optimistic Concurrency Control" width="600" />

Assumes conflicts are **rare**, so it **doesn't lock** the database. Instead, every row carries a **version tag** (like a number, timestamp, or GUID).

When saving, the database checks: *"Is the version still the same as when this user read it?"*
- **Same version:** Save succeeds, and the version updates (`v1` → `v2`).
- **Different version:** Someone else edited it first. The save is rejected, and the API returns **`409 Conflict`** so the user can reload and retry.

- **Pros:** Fast, no blocking, great for web APIs with lots of reads.
- **Cons:** Users occasionally have to retry if a conflict happens.

---

### 2. Pessimistic Concurrency

<img src="PASTE_PESSIMISTIC_CONCURRENCY_IMAGE_URL_HERE" alt="Pessimistic Concurrency Control" width="600" />

Assumes conflicts are **likely**, so the database **locks the row** the moment Admin A starts editing. Admin B is forced to wait until Admin A saves and releases the lock.

- **Pros:** Guarantees no conflicts or retries—changes happen strictly one at a time.
- **Cons:** Slows down traffic when multiple users wait on the same lock, and can cause **deadlocks** if two operations wait on each other.
