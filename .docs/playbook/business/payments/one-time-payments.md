---
kind: playbook
domain: business
topics: [business, payments, one, time]
authority: guidance
---

# One-Time Payments

One-time payment flows support digital downloads, licenses, tickets, courses, physical goods, donations, and fixed-price services. They are operationally simpler than subscriptions only when fulfillment, tax, refunds, disputes, and duplicate events are handled explicitly.

## Payment Flow

1. Create an internal pending order with an immutable line-item and price snapshot.
2. Create a hosted checkout session on the server and attach the internal order ID as metadata.
3. Redirect the buyer to the provider-hosted page.
4. Treat the return URL as presentation only; it does not prove payment.
5. Verify the signed provider webhook and transition the order idempotently.
6. Fulfill the order only after the provider reports a successful payment.
7. Send a receipt and make fulfillment status recoverable from the order record.

## Order States

Use explicit states rather than a single `paid` boolean:

```text
pending -> processing -> paid -> fulfilled
    |          |          |         |
 expired     failed    refunded   partially_refunded
                           |
                        disputed
```

Define allowed transitions and store provider event IDs. A duplicate webhook must return success without fulfilling twice. When fulfillment is asynchronous, enqueue it with the internal order ID and use an idempotency key at the job boundary as well.

## Fulfillment by Product Type

| Product | Fulfillment requirement |
|---|---|
| Digital download | Expiring, signed link; download and device limits if promised |
| License | Unique license record; activation and transfer policy |
| Ticket or booking | Atomic capacity reservation and cancellation policy |
| Physical goods | Inventory reservation, shipping address, tracking, returns |
| Fixed-price service | Scope, scheduling, acceptance, and refund milestones |
| Donation | Receipt language and tax-deductibility must match legal status |

## Refunds and Disputes

Store full and partial refunds as separate payment records instead of overwriting the original charge. Revoke access only when the stated terms allow it, and keep financial records required for reconciliation even if personal data is later anonymized. A dispute should freeze risky fulfillment, alert an operator, and preserve evidence such as the accepted terms and delivery record.

## Recommended Default

Use hosted checkout to keep card data off your servers. Keep your own order ledger as the business source of truth while treating provider events as authenticated inputs. For digital goods sold internationally by a small team, evaluate a merchant-of-record provider to reduce sales-tax and VAT operations.

## Security

- Calculate prices and discounts on the server; never trust client-submitted totals.
- Verify webhook signatures against the raw request body.
- Use idempotency keys when creating payment sessions and processing events.
- Never unlock a purchase based on query parameters or a client redirect.
- Keep card data out of application logs and databases.

## Related Documents

- [Payments provider options](provider-options.md)
- [One-time purchase strategy](../monetization/one-time-purchases.md)
- [Invoices and refunds](../../integrations/providers/stripe/invoices-refunds.md)
- [Webhook handling](../../backend/async/webhooks.md)
- [PCI DSS](../../compliance/pci-dss.md)
