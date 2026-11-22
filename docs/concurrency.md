# Concurrency Control

**Concurrency control** prevents data from being accidentally overwritten or corrupted when multiple users try to edit the same record at the same time.

---

## Real-World Analogy

Imagine two gym admins managing the **Basic** membership plan (currently **$29.99**) at the exact same time.

Both open the plan's pricing sheet simultaneously. Admin A decides to run a discount, updates the price to **$20**, and saves it at **3:00 PM**. One minute later at **3:01 PM**, Admin B—who is still looking at the old **$29.99** sheet—raises the price to **$35** and hits save. Admin A's discount is **completely wiped out**, and neither admin realizes it happened.

---

## In Real Software

In a backend API, this is called a **Lost Update**.

![Http Request_20261122_184423_0000.png](https://github.com/user-attachments/assets/27263453-b38c-4ae9-a350-52e7d5e00d84)

When two requests read the same database row at the same time and both try to save changes, the last write silently wins—overwriting the first user's update.

Common places this breaks real apps:
- **Admin Dashboards:** Two admins editing the same plan price or user profile simultaneously.
- **E-commerce & Inventory:** Two people buying the last item in stock at the exact same second.
- **Banking:** Two withdrawals reading the same balance before either deducts money.

---

## Types of Concurrency Control

### 1. Optimistic Concurrency

![Http Request_20261122_190748_0000.png](https://github.com/user-attachments/assets/48dfbe65-e087-444d-ae1a-65052a023582)

Nobody gets locked out. Instead, every piece of data has a **version number** (like `v1`).

When someone tries to save a change, the database asks one simple question: **"Is this still `v1`?"**
- **Yes (`v1`):** Save goes through, and the version bumps to `v2`.
- **No (it's already `v2`):** Someone else beat you to it. The save is blocked with **`409 Conflict`**, so you just refresh and try again.

- **Pros:** Super fast—no waiting in line.
- **Cons:** If two people edit at the exact same second, the second person has to retry.

---

### 2. Pessimistic Concurrency

<img src="PASTE_PESSIMISTIC_CONCURRENCY_IMAGE_URL_HERE" alt="Pessimistic Concurrency Control" width="600" />

Assumes conflicts are **likely**, so the database **locks the row** the moment Admin A starts editing. Admin B is forced to wait until Admin A saves and releases the lock.

- **Pros:** Guarantees no conflicts or retries—changes happen strictly one at a time.
- **Cons:** Slows down traffic when multiple users wait on the same lock, and can cause **deadlocks** if two operations wait on each other.
