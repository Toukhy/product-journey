# External Coupon Integration (Webhook-Based Coupon Validation)

**Source:** Internal Product
**Requester:** Assy
**Date:** 2026-02-24

---

## User Story
**As a** Gameball merchant with an existing coupon or promotions system,
**I want** to connect my external coupon system to Gameball via a webhook/API endpoint,
**So that** Gameball can validate and retrieve coupons from my system in real-time when a customer is about to redeem one — without requiring me to upload coupon CSVs manually.

---

## Description

**Current situation:**
Merchants who want to offer coupons as a redemption reward inside Gameball must upload a CSV file containing pre-generated coupon codes through the dashboard. Gameball stores these codes internally and serves them to customers upon redemption. This model works for merchants with a fixed, pre-known coupon set, but breaks down for merchants who:
- Generate coupon codes dynamically in an external system
- Have a large or frequently-changing coupon catalog
- Need Gameball to check coupon validity against their own source of truth before issuing

**Proposed change:**
Introduce a new coupon source type — **"External / API-connected"** — alongside the existing CSV upload option. Merchants configure an endpoint URL (webhook) on their side. When a customer is eligible to receive a coupon reward, Gameball calls that endpoint in real-time to request a valid coupon code. Gameball then issues the returned code to the customer, exactly as it would with a CSV-sourced coupon.

**Impacted modules:**
- Rewards / Redemption Options (new coupon source type)
- Admin Settings → Integration (new External Coupon configuration section)
- Redemption flow (outbound webhook call at reward issuance time)
- Dashboard UI (configuration form for endpoint, auth, and fallback behavior)

---

## Acceptance Criteria

### Functional Requirements
- Merchants can select **"External Coupon Source"** as a new option when configuring a coupon-type reward, in addition to the existing CSV upload option
- Merchants configure the following in the dashboard:
  - **Endpoint URL** — the webhook URL Gameball will call to request a coupon
  - **Authentication method** — API Key (header-based) or Bearer Token
  - **Secret / Token value**
  - **Timeout threshold** — max wait time before fallback behavior kicks in (default: 3 seconds)
  - **Fallback behavior** — either block the redemption or skip coupon issuance silently
- Gameball sends a `POST` request to the configured endpoint at the moment of coupon reward issuance, with a payload containing relevant context (customer ID, reward ID, timestamp, etc.)
- The external system responds with a coupon code (and optionally: expiry, label)
- Gameball issues the returned coupon code to the customer, logs it, and displays it in the widget exactly as CSV-sourced coupons are shown today
- The configuration can be tested from the dashboard with a "Send Test Request" button that fires a sample payload and shows the response

---

### Behavior & Logic
- The webhook call is made **synchronously at reward issuance time** — Gameball waits for a response before confirming the reward to the customer
- If the external endpoint returns a valid coupon code within the timeout window, the coupon is issued and the redemption proceeds normally
- If the endpoint returns an error or times out:
  - If fallback = **block**: redemption is paused and an error message is shown to the customer (e.g., "Coupon unavailable, please try again")
  - If fallback = **skip**: reward is marked as issued but no coupon code is given; the customer sees a generic success message
- Each coupon code returned by the external system is treated as single-use by Gameball (same behavior as CSV coupons)
- Gameball logs every outbound webhook call: timestamp, payload sent, response received, HTTP status, latency
- If the merchant has both CSV and External configured on different reward options, both can coexist independently

---

### Validation & Error Handling
- Endpoint URL must be a valid HTTPS URL — HTTP endpoints are rejected
- Auth token/key field must not be empty if an auth method is selected
- If the external endpoint returns a non-200 HTTP status, Gameball treats it as a failure and applies fallback behavior
- If the response body does not contain a coupon code in the expected format, Gameball treats it as a failure
- Timeout must be configurable between 1–10 seconds; values outside this range are rejected with a validation error
- "Send Test Request" must show the merchant the raw response and whether it would be treated as a success or failure
- Dashboard must show a warning if the endpoint URL is saved without being tested

---

### Data Accuracy & Consistency
- Every coupon code issued via external source is stored in Gameball's transaction log with source = `external_webhook`, associated reward ID, customer ID, and timestamp
- Coupon codes received from external endpoints are not stored in Gameball's coupon pool — they are fetched on demand and logged post-issuance
- Webhook call logs (request/response) are retained for 30 days for debugging purposes and visible to the merchant in the dashboard
- If the same coupon code is returned twice by the external system (duplicate), Gameball flags it in the log but still issues it — deduplication is the external system's responsibility

---

### UI / API Behavior
- In the reward configuration UI, the coupon source selector shows two options: **"Upload CSV"** (existing) and **"Connect External System"** (new)
- Selecting "Connect External System" reveals the configuration form (endpoint, auth, timeout, fallback)
- The webhook payload sent by Gameball to the merchant's endpoint follows this structure:

```json
{
  "event": "coupon_requested",
  "customer_id": "abc123",
  "customer_email": "user@example.com",
  "reward_id": "rwd_456",
  "reward_name": "Exclusive Discount",
  "requested_at": "2026-02-24T10:00:00Z"
}
```

- The expected response from the merchant's endpoint:

```json
{
  "coupon_code": "SAVE20-XYZ",
  "expires_at": "2026-12-31T23:59:59Z",
  "label": "20% Off Your Next Order"
}
```

- `expires_at` and `label` are optional; only `coupon_code` is required
- API documentation for the webhook contract must be published in Gameball's developer docs

---

### Backward Compatibility
- Existing CSV-based coupon rewards are completely unaffected
- Merchants not using external coupon integration see no changes to their current workflow
- No changes to how issued coupons appear in the customer widget — behavior is identical regardless of source

---

### Performance & Scalability
- Gameball's outbound webhook call must not add more than the configured timeout to the redemption response time
- Webhook calls must be made with proper connection pooling to avoid resource exhaustion under concurrent redemptions
- If a merchant's endpoint consistently fails (e.g., >10 consecutive failures), Gameball sends an alert email to the merchant and temporarily disables the external source, falling back to the configured fallback behavior
- Webhook call logs must not impact dashboard query performance — stored separately from core transaction data

---

### Security & Permissions
- Only merchants on **Shopify Pro, Shopify Guru, Self-Serve Growth, Self-Serve Enterprise, Salla Pro, Salla Guru** can configure External Coupon Integration
- Within the dashboard, only users with **Admin** role can create or modify the external coupon configuration
- The auth token/secret value is stored encrypted at rest and never returned in plaintext via the dashboard UI after saving (masked, with a "regenerate" option)
- All outbound webhook calls from Gameball include a signature header (`X-Gameball-Signature`) so the merchant can verify the request originated from Gameball

---

### Documentation
- Developer docs: webhook contract (request payload schema, response schema, error codes)
- Merchant-facing help article: how to set up External Coupon Integration, with a step-by-step guide and example endpoint implementation
- Dashboard inline help text on each configuration field
- Changelog entry on release

---

### Plans

**Shopify:** Pro, Guru
**Salla:** Pro (الفضية), Guru (الذهبية)
**Self-Serve:** Growth, Enterprise

---

## Notes / Assumptions
- The feature assumes the merchant's endpoint is always available and performant — Gameball is not responsible for downtime on the merchant's side
- Deduplication of coupon codes across redemptions is the merchant's responsibility
- Initial release targets webhook/API key auth only; OAuth-based external system auth is out of scope for v1
- The signature verification mechanism (`X-Gameball-Signature`) spec to be defined by the backend team (e.g., HMAC-SHA256 of payload + shared secret)
- Open question: should Gameball support a "coupon pool" fallback where if the external endpoint fails, it falls back to a CSV pool? Could be a v2 consideration
- Open question: should there be a rate limit on how many webhook calls Gameball makes per merchant per minute?

---

## Tech Review Summary
- **Effort:** Large (L)
- **Recommended milestones:**
  - M1: Core integration (config, dispatcher, coupon issuance, basic fallback)
  - M2: Observability (webhook log UI, circuit breaker, merchant alerts)
- **Key risks:** endpoint latency, SSRF via merchant-controlled URL, secrets storage
