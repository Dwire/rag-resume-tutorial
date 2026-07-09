# Payments platform migration (sample — safe to delete)

> This is a fictional example so you can see the expected format. Replace the
> `documents/` folder with your own write-ups.

At Acme Corp (2021–2023) I led the migration of the checkout service from a
monolith to three independently deployable services. Cut p95 checkout latency
from 820ms to 240ms and reduced payment-failure retries by 38%.

- Designed the idempotency-key scheme that eliminated duplicate charges during
  network retries (~1,200 avoided per month).
- Introduced contract tests between the cart and payments services, dropping
  cross-team integration bugs to near zero over two quarters.
- Tech: Python, PostgreSQL, Stripe API, Kafka, Docker, GitHub Actions.

## Skills demonstrated
Service decomposition, idempotency and exactly-once semantics, performance
profiling, cross-team API contracts, incremental migration under live traffic.
