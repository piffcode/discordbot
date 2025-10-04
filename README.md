# Discord Sales Bot Workflow

This repository contains the exported n8n workflow that powers the Discord sales bot automation.

## Payment Verification Flow

When a customer replies in the ticket channel with `sent ...`, the **Capture Payment Confirmation** code node parses the message and normalizes the payment method. Variants such as `cashapp`, `cash`, `cashappff`, or `cashappfnf` are all collapsed to the canonical value `cashapp` so Cash App confirmations are recognized alongside PayPal, Venmo, and crypto options.

Next, the **Extract Payment Expectations** and **Build Verification Payload** nodes recover the invoice details from the earlier payment instructions and assemble the payload for the verification API. If the payment instructions embed listed Cash App, the canonical `cashapp` identifier is included in the `acceptedPaymentMethods` array that is passed to the downstream verification request. When no methods are explicitly recovered, the workflow defaults the accepted list to `['paypal', 'venmo', 'cashapp']` to ensure Cash App is always considered valid.

Finally, the **Verify Payment** HTTP Request node submits the assembled payload to the finance service, which confirms whether the funds arrived through any of the accepted methods. The response is merged back into the workflow context so later steps can tailor their messaging based on the verified payment method.
