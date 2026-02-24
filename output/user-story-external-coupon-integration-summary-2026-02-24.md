# User Story: External Coupon Integration (Webhook-Based Coupon Validation)

## Source
- **Channel:** Internal Product Idea
- **Requester:** Assy

## Feature Description
Introduce a new coupon source type — "External / API-connected" — alongside the existing
CSV upload option. Instead of requiring merchants to upload a CSV of pre-generated coupon
codes, merchants configure a webhook endpoint on their side. When a customer is eligible
to receive a coupon reward, Gameball calls that endpoint in real-time to request a valid
coupon code, then issues it to the customer — identical behavior to CSV-sourced coupons
but sourced dynamically from the client's own infrastructure.

## Strategic Fit
| Criteria | Result | Reasoning |
|----------|--------|-----------|
| Aligns with OKRs | Yes | Fulfills customer requests (OKR #3); removes a deal blocker for mid-market/enterprise clients driving revenue (OKR #1) |
| Matches Strategy | Yes | Consistent with Gameball's deep customizability positioning; lets clients plug in their own systems rather than being forced into Gameball's CSV model |
| Supports Revenue/Growth | Yes | Unlocks mid-market and enterprise deals where clients have existing coupon infrastructure; potential deal-closer for that segment |

## Product Fit
- **Scope:** Niche — targets merchants with existing external coupon/promo systems; not
  relevant to smaller merchants who rely entirely on Gameball for coupon generation.
  High-value for the segment it serves.

### Plan Availability
| Platform | Free | Starter | Pro | Guru |
|----------|------|---------|-----|------|
| Shopify | ❌ | ❌ | ✅ | ✅ |
| Self-Serve | — | — | Growth: ✅ | Enterprise: ✅ |
| Salla | ❌ | ❌ | ✅ (الفضية) | ✅ (الذهبية) |

## Market Research
### Competitor Comparison
| Competitor | External Coupon Support | Architecture | Pricing Level |
|-----------|------------------------|-------------|---------------|
| Talon.One | ✅ Native webhook-based real-time pre-validation | Self-serve, embedded in rule engine | All tiers (limits scale up) |
| Capillary Tech | ⚠️ Partial — post-event notification, not pre-validation | Requires support ticket to configure | Enterprise only, custom pricing |

### Pricing Model Analysis
- Talon.One is the most capable competitor here — real-time pre-validation via webhooks
  available across all tiers, self-serve
- Capillary's model is post-event and requires support intervention — not true real-time
- Neither competitor offers this natively embedded in a loyalty + gamification platform
  at Gameball's price point in a self-serve way

## Competitor Analysis
### vs Talon.One
- Talon.One supports real-time webhook callbacks before discount application
- Their implementation is a separate system not native to a loyalty platform
- Gameball wins on: unified experience (loyalty + coupon in one platform), mid-market
  pricing, and the natural fit within the existing reward/redemption flow

### vs Capillary Tech
- Capillary's approach is post-event notifications, not pre-validation
- Requires raising a support ticket to configure — not self-serve
- Gameball wins on: true real-time validation, self-serve dashboard configuration,
  accessible to Pro/Guru (not just enterprise)

### Gaps to Fill
- No loyalty platform offers real-time external coupon validation natively embedded
  in the redemption flow at mid-market price points
- Competitors require external system setup; Gameball can make this self-serve from
  the dashboard

### Our Differentiators
- **Loyalty-native integration:** External coupon is a first-class option inside the
  existing reward/redemption flow — not a bolt-on
- **Self-serve config:** Dashboard form for endpoint, auth, timeout, and fallback —
  no support ticket needed
- **Mid-market accessible:** Available at Pro/Guru, not enterprise-only

## User Story
**As a** Gameball merchant with an existing coupon or promotions system,
**I want** to connect my external coupon system to Gameball via a webhook/API endpoint,
**So that** Gameball can validate and retrieve coupons from my system in real-time when
a customer is about to redeem one — without requiring me to upload coupon CSVs manually.

### Acceptance Criteria
- [ ] Merchants can select "External Coupon Source" as a new option when configuring a
      coupon-type reward (alongside existing CSV upload)
- [ ] Configuration form collects: Endpoint URL (HTTPS only), Auth method (API Key /
      Bearer Token), token value (encrypted), timeout (1–10s, default 3s), fallback
      behavior (block redemption / skip silently)
- [ ] Gameball POSTs to the configured endpoint at reward issuance time with context
      payload: event, customer_id, customer_email, reward_id, reward_name, requested_at
- [ ] External endpoint responds with: coupon_code (required), expires_at (optional),
      label (optional)
- [ ] On success: coupon issued and displayed to customer identically to CSV-sourced coupons
- [ ] On failure/timeout with Block fallback: redemption paused, error shown to customer
- [ ] On failure/timeout with Skip fallback: reward marked issued, no code shown
- [ ] "Send Test Request" button fires sample payload and shows raw response + pass/fail
- [ ] All webhook calls logged: timestamp, payload, response, HTTP status, latency
      (30-day retention, visible in dashboard)
- [ ] Circuit breaker: after 10 consecutive failures → alert email to merchant +
      temporarily disable external source
- [ ] Auth token masked in UI after save; never returned in plaintext
- [ ] Outbound calls signed with X-Gameball-Signature (HMAC-SHA256)
- [ ] SSRF protection: HTTPS only, private/internal IP ranges blocked

### Azure DevOps
- **Work Item ID:** #23417
- **Board:** Kanban Q3 — New column (top)
- **Link:** https://dev.azure.com/gameballers/Gameball%20Main%20Project/_workitems/edit/23417

## UX Notes
### User Flow
1. Merchant opens Rewards → Add/Edit coupon reward
2. "Coupon Source" selector: [Upload CSV] | [Connect External System]
3. Selecting "Connect External System" reveals the config form
4. Merchant fills in endpoint, auth, timeout, fallback → clicks "Send Test Request"
5. Test shows raw response + pass/fail → merchant saves configuration
6. At redemption time: customer clicks Redeem → Gameball silently calls endpoint →
   coupon code displayed to customer (success) or fallback behavior applied (failure)

### Wireframe
```
┌─────────────────────────────────────────────┐
│ Coupon Source                               │
│                                             │
│  ○  Upload CSV                              │
│  ●  Connect External System                 │
│                                             │
│ ┌─────────────────────────────────────────┐ │
│ │ Endpoint URL                            │ │
│ │ https://your-system.com/coupons/issue   │ │
│ ├─────────────────────────────────────────┤ │
│ │ Auth Method      [API Key       ▼]      │ │
│ │ API Key Value    [●●●●●●●●●●●●  👁]     │ │
│ ├─────────────────────────────────────────┤ │
│ │ Timeout          [3] seconds            │ │
│ │ Fallback         [Block redemption ▼]   │ │
│ ├─────────────────────────────────────────┤ │
│ │        [Send Test Request]              │ │
│ │  ✅ Response: 200 OK — coupon_code      │ │
│ │     received: "SAVE20-XYZ"             │ │
│ └─────────────────────────────────────────┘ │
│                          [Cancel]  [Save]   │
└─────────────────────────────────────────────┘
```

### UI Consistency
- Coupon source toggle follows existing radio button pattern in reward configuration
- Masked token field (eye icon + regenerate) consistent with Integration settings page
- Test request result panel uses same success/error styling as other inline feedback
- Webhook call log table follows Customer Activity Log table component pattern

## Tech Notes
### Architecture Impact
- **Webhook Dispatcher Service:** New module in redemption flow for outbound HTTP calls
- **External Coupon Config Store:** New DB entity (URL, auth method, encrypted token,
  timeout, fallback) per merchant
- **Webhook Call Log Store:** Separate log table, TTL-purged at 30 days
- **Signature Generator:** HMAC-SHA256 signing of outbound payloads
- **Circuit Breaker:** Consecutive failure detection + merchant email alert + auto-disable
- **Dashboard API:** New CRUD endpoints for config + log retrieval

### Dependencies & Risks
| Item | Risk | Notes |
|------|------|-------|
| Merchant endpoint latency | Medium-High | Strict configurable timeout; async fallback |
| Secrets storage | Low | Encrypt at rest using existing encryption service |
| Malformed external response | Medium | Strict schema validation; treat non-conforming as failure |
| SSRF via merchant-controlled URL | Low-High | HTTPS only; block private/internal IP ranges |
| Circuit breaker false positives | Low | Threshold tuning; manual re-enable from dashboard |
| Log table growth | Low | TTL purge at 30 days; separate from core tables |

### Effort Estimation
| Component | Effort |
|-----------|--------|
| Backend — Webhook Dispatcher + config store | M |
| Backend — Signature generation + security | S |
| Backend — Circuit breaker + alert | S |
| Backend — Webhook call log + TTL purge | S |
| Dashboard UI — Config form + test request | M |
| Dashboard UI — Webhook call log view | S |
| Dashboard API — Config CRUD + log endpoints | S |
| Plan gating (Pro/Guru/Growth/Enterprise) | XS |
| Developer docs + merchant help article | S |
| **Total** | **L** |

> Recommend two milestones: M1 — Core integration (config, dispatcher, coupon issuance,
> basic fallback); M2 — Observability (webhook log UI, circuit breaker, merchant alerts).
