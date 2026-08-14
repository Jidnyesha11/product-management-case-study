# Section 5 — Product Requirements Document (PRD)
### Jentara | MVP & V1 Platform Requirements

**Document Owner:** Product Management
**Status:** Draft for Engineering Review
**Related Sections:** 04-Product-Strategy, 06-Feature-Breakdown*, 07-Architecture

---

## 5.1 Goals

| Goal | Description |
|---|---|
| G1 | Ship a production-ready D2C storefront supporting browsing, cart, checkout, and order management |
| G2 | Support drop-based commerce mechanics: waitlists, countdowns, limited inventory, per-account purchase limits |
| G3 | Provide an admin dashboard for order, inventory, and catalog management from day one |
| G4 | Integrate reliable India-first payments via **Razorpay** (UPI, cards, netbanking, wallets) |
| G5 | Build a foundation extensible to loyalty, AI styling, and creator marketplace features (post-MVP) |

## 5.2 Scope (MVP — V1)

**In Scope:**
- Customer-facing storefront (Next.js/TypeScript/Tailwind)
- Supabase Auth (email/OTP + social login)
- Product catalog with variants (size/color), collections, drops
- Cart, wishlist, checkout, Razorpay payment integration
- Order management (place, track, cancel, return request)
- Coupons/discount codes
- Reviews & ratings
- Search & filters
- Basic recommendation engine (rule-based, not AI, for MVP)
- Admin dashboard: order management, inventory management, basic analytics
- Notifications: order status (email), restock alerts (email)
- Waitlist/pre-launch signup mechanism

**Explicitly Out of Scope for MVP (see Section 13 — Future Roadmap):**
- AI Styling Assistant (ML-based)
- Virtual Try-On
- Creator Marketplace & affiliate dashboard
- Loyalty program (points/tiers)
- Gift cards
- Vendor multi-seller dashboard
- Native mobile app (mobile-web only for MVP)
- International shipping/currency support
- WhatsApp marketing automation (email only for MVP)

## 5.3 Functional Requirements

| ID | Requirement | Priority |
|---|---|---|
| FR-01 | Users can register/login via email+password, OTP, or social login (Supabase Auth) | Must |
| FR-02 | Users can browse products by collection, category, and drop | Must |
| FR-03 | Users can view a Product Detail Page (PDP) with images, size guide, story/lore content, and stock status | Must |
| FR-04 | Users can add/remove products (with size/color variant) to cart | Must |
| FR-05 | Users can add/remove products to a wishlist | Must |
| FR-06 | Users can apply a coupon code at checkout | Must |
| FR-07 | Users can complete checkout via **Razorpay** (UPI, Cards, Netbanking, Wallets) | Must |
| FR-08 | System reserves inventory for a limited window (e.g., 10 minutes) once checkout begins, to prevent overselling during drops | Must |
| FR-09 | Users can view order history and real-time order status | Must |
| FR-10 | Users can request a return/exchange from order history | Must |
| FR-11 | Users can submit a rating and written review for purchased products | Should |
| FR-12 | Users can search products with keyword and filter by size/color/collection/price | Must |
| FR-13 | Users receive email notifications for order confirmation, shipping, delivery, and restocks | Must |
| FR-14 | Users can join a pre-launch waitlist and receive drop-notification emails | Must |
| FR-15 | Admins can create/edit/archive products, variants, and collections | Must |
| FR-16 | Admins can view and update order status (processing, shipped, delivered, returned) | Must |
| FR-17 | Admins can manage inventory levels per SKU/variant, including low-stock alerts | Must |
| FR-18 | Admins can view a basic analytics dashboard (sales, top products, traffic) | Should |
| FR-19 | System enforces per-account purchase limits during limited drops (e.g., max 2 units per SKU) | Must |
| FR-20 | System supports scheduled "drop" publishing (product goes live at a specific timestamp) | Must |

## 5.4 Non-Functional Requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-01 | Page load performance (LCP) | < 2.5s on 4G mobile |
| NFR-02 | Storefront uptime | ≥ 99.9% monthly |
| NFR-03 | Checkout success rate (excluding user payment failures) | ≥ 98% |
| NFR-04 | System must handle drop-day traffic spikes | Support 10x baseline concurrent traffic without degradation |
| NFR-05 | Data protection | Compliant with India's DPDP Act; Supabase RLS policies enforced on all customer data tables |
| NFR-06 | Payment security | Razorpay integration must be PCI-DSS compliant via Razorpay's hosted checkout/SDK — no raw card data touches Jentara servers |
| NFR-07 | Accessibility | WCAG 2.1 AA compliance on core storefront flows |
| NFR-08 | Mobile responsiveness | Fully responsive across breakpoints 320px–1920px |
| NFR-09 | Scalability | Architecture must support horizontal scaling of catalog/traffic without re-architecture (see Section 7) |
| NFR-10 | Observability | All checkout and payment events logged with correlation IDs for support/debugging |

## 5.5 Acceptance Criteria (Sample — Checkout Flow)

**Feature: Razorpay Checkout**

```
Given a user has items in their cart
And the items are still in stock
When the user proceeds to checkout and selects a payment method (UPI/Card/Netbanking/Wallet)
Then Razorpay's secure checkout modal/SDK is invoked
And on successful payment, an order is created with status "Confirmed"
And inventory is decremented for the purchased variants
And the user receives an order confirmation email within 2 minutes
And the payment is verified server-side via Razorpay webhook signature validation before the order is marked "Confirmed"
```

```
Given a user's Razorpay payment fails or is cancelled
When the checkout session ends without a successful payment
Then the reserved inventory hold is released after the reservation window expires
And the user is shown a clear retry option
And no order is created in the system
```

## 5.6 Edge Cases

| Edge Case | Expected Behavior |
|---|---|
| Two users add the last unit of a size to cart simultaneously during a drop | Inventory hold is granted on a first-checkout-initiated basis; second user is notified the item is no longer available before payment |
| Razorpay webhook is delayed or fails to reach the server | System polls Razorpay payment status API as a fallback reconciliation job within 5 minutes |
| User closes browser mid-payment after Razorpay debits but before redirect | Webhook-driven confirmation ensures order is still created; user sees status update on next login |
| Coupon code is applied but becomes invalid before payment completes (e.g., expired) | Checkout re-validates coupon at payment initiation; user is notified and total is recalculated |
| Admin marks a product out of stock while it's in a user's cart | Cart shows "no longer available" at checkout review step, preventing payment attempt |
| User requests a return outside the return window | System blocks return request submission and displays the policy-based cutoff date |
| Drop goes live but traffic exceeds server capacity | Queueing/waiting-room mechanism activates (see Section 7 — Architecture) instead of hard failure |

## 5.7 Epics, Features & User Stories

### Epic 1: Authentication & Account

| Story ID | User Story | Priority |
|---|---|---|
| US-101 | As a new user, I want to sign up with email or OTP, so I can create an account quickly | Must |
| US-102 | As a returning user, I want to log in via social login, so I don't need to remember a password | Should |
| US-103 | As a user, I want to reset my password securely, so I can regain account access | Must |

### Epic 2: Catalog & Discovery

| Story ID | User Story | Priority |
|---|---|---|
| US-201 | As a shopper, I want to browse by collection/drop, so I can find themed products easily | Must |
| US-202 | As a shopper, I want to filter by size/color/price, so I can narrow results to what fits me | Must |
| US-203 | As a shopper, I want to see a countdown for upcoming drops, so I know when to return | Must |
| US-204 | As a shopper, I want to read the story/lore behind a collection on the PDP, so I understand its meaning | Should |

### Epic 3: Cart, Wishlist & Checkout

| Story ID | User Story | Priority |
|---|---|---|
| US-301 | As a shopper, I want to add items to my cart with size/color selected, so I can purchase multiple items together | Must |
| US-302 | As a shopper, I want to save items to a wishlist, so I can purchase them later or get notified of restocks | Must |
| US-303 | As a shopper, I want to apply a coupon code at checkout, so I can redeem a discount | Should |
| US-304 | As a shopper, I want to pay via UPI/Card/Netbanking through Razorpay, so I can use my preferred payment method | Must |
| US-305 | As a shopper, I want my cart items held briefly during checkout, so I don't lose them to another buyer mid-payment | Must |

### Epic 4: Orders & Post-Purchase

| Story ID | User Story | Priority |
|---|---|---|
| US-401 | As a customer, I want to view my order status in real time, so I know when to expect delivery | Must |
| US-402 | As a customer, I want to request a return/exchange, so I can resolve sizing/quality issues | Must |
| US-403 | As a customer, I want to leave a review and rating, so I can share my experience with others | Should |

### Epic 5: Admin & Operations

| Story ID | User Story | Priority |
|---|---|---|
| US-501 | As an admin, I want to create and edit products/variants, so I can manage the catalog | Must |
| US-502 | As an admin, I want to update order statuses, so customers see accurate tracking info | Must |
| US-503 | As an admin, I want to view low-stock alerts, so I can restock before selling out unexpectedly (outside intended drops) | Should |
| US-504 | As an admin, I want a sales/traffic analytics dashboard, so I can make data-informed decisions | Should |

### Epic 6: Drop Mechanics

| Story ID | User Story | Priority |
|---|---|---|
| US-601 | As a shopper, I want to join a waitlist for an upcoming drop, so I get early notification | Must |
| US-602 | As the system, I need to enforce per-account purchase limits during drops, so inventory is fairly distributed | Must |
| US-603 | As a shopper, I want a fair queueing experience if traffic exceeds capacity, so I'm not met with a broken page | Must |

## 5.8 MoSCoW Prioritization Summary

| Priority | Count | Examples |
|---|---|---|
| **Must Have** | 15 stories | Auth, catalog browsing, cart, Razorpay checkout, order tracking, admin product/order management, drop queueing & purchase limits |
| **Should Have** | 6 stories | Social login, coupons, reviews, low-stock alerts, analytics dashboard, lore content on PDP |
| **Could Have** | Deferred | Wishlist-to-restock auto-notify refinements, advanced filters (fit type, fabric) |
| **Won't Have (this release)** | Deferred to roadmap | AI styling, virtual try-on, creator marketplace, loyalty program, gift cards, native app |

## 5.9 Product Backlog (Prioritized — Top of Backlog)

| Rank | Item | Epic | Priority | Est. Story Points |
|---|---|---|---|---|
| 1 | Supabase Auth integration (email + OTP) | Auth | Must | 5 |
| 2 | Product catalog data model & PDP | Catalog | Must | 8 |
| 3 | Cart & inventory hold logic | Cart/Checkout | Must | 8 |
| 4 | Razorpay checkout integration + webhook verification | Cart/Checkout | Must | 13 |
| 5 | Order management (customer + admin) | Orders/Admin | Must | 8 |
| 6 | Drop scheduling & countdown | Drop Mechanics | Must | 5 |
| 7 | Per-account purchase limit enforcement | Drop Mechanics | Must | 5 |
| 8 | Waiting-room/queueing for high-traffic drops | Drop Mechanics | Must | 8 |
| 9 | Admin inventory management | Admin | Must | 5 |
| 10 | Waitlist signup + email notification | Drop Mechanics | Must | 3 |
| 11 | Coupon engine | Cart/Checkout | Should | 5 |
| 12 | Reviews & ratings | Post-Purchase | Should | 5 |
| 13 | Search & filters | Catalog | Must | 5 |
| 14 | Return/exchange request flow | Orders | Must | 5 |
| 15 | Basic admin analytics dashboard | Admin | Should | 8 |

*(Full backlog with all remaining Should/Could items continues in Section 9 — Jira/Sprint Board templates.)*

---
**Previous section:** [`04-Product-Strategy`](../04-Product-Strategy/Product-Strategy.md)
**Next section:** [`06-UX`](../06-UX/UX.md) *(Note: Feature Breakdown detail — Section 6 in the original outline — is folded into the UX & Architecture sections' feature-level specs to avoid duplication; a dedicated Feature Breakdown matrix will be included as an appendix.)*
