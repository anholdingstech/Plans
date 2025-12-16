# Smart Use-Case VPN – MVP Build Plan

Version: 1.0 
Date: 2025-12-14 
Company: US-based 
MVP Timeline: < 1 month 
Target: Web app MVP (desktop & mobile web), future native iOS/Android

---

## 1. Product Overview

### 1.1 Core Idea

A **web-based VPN app** with:

- **Use-case presets** ("Flights", "Streaming Services", "Social Media", "Privacy/Anonymity", etc.).
- **Smart location recommendations** (country/city/IP type) tailored to each use case.
- **One-click connect**, simple UX, with an optional **advanced guide** that remains friendly and short.
- Focus on **high-quality, clean IPs**, including **static/dedicated and persistent IPs**, and **optionally higher-priced “clean” IP tiers**.

### 1.2 Primary User Segments

1. **General consumers**
   - Want to unblock content, get deals, feel “safer” online.
2. **Content creators**
   - Need consistent IPs, “clean” IPs for platforms (e.g., Social Media).
3. **Travelers / digital nomads**
   - Access home-region services (streaming, banking).
   - Find better flight deals.

---

## 2. Feature Scope (MVP)

### 2.1 Core Features (User-Facing)

1. **Authentication**
   - Email + password signup/login.
   - Password reset (email-based).

2. **Subscription & Payments**
   - **Stripe** integration.
   - **Paid-only** access (no free plan).
   - **Monthly and yearly** plans, yearly discounted.
   - Basic subscription management (view/cancel plan).

3. **VPN Connection UX**
   - One-click **Connect / Disconnect**.
   - **Use-case selection**:
     - Flights
     - Streaming Services
       - YouTube TV
       - Netflix
       - Prime Video
     - Social Media
       - X/Twitter
       - TikTok
     - Privacy/Anonymity
     - (Room to add more use cases later)
   - **Location selection**:
     - Default: “Best location for this use case.”
     - Advanced: choose from a curated list of **Top ~10 countries** (with some city-level options where needed).
   - **Connection status** indicator (connected / connecting / error).
   - **Speed & latency indicators**:
     - Basic: ping + estimated bandwidth class (e.g., “Fast”, “OK”, “Slow”).

4. **Use-Case Settings Guide**
   - **Short, friendly “recipes”** for each use case:
     - E.g., “Flights: Try Country X or Y, clear cookies, use incognito, don’t log in until you’ve found a price you like.”
   - Option to **expand details**:
     - Collapsible sections: “Learn more”.
   - Guides for:
     - Flights
     - Streaming:
       - YouTube TV (location-first)
       - Netflix
       - Prime Video
     - Social Media:
       - X/Twitter
       - TikTok
     - Privacy/Anonymity (best practices, but still simple language).
   - Future: dynamic/AB tested recommendations based on metrics and policies.

5. **Settings / Controls**
   - Kill switch toggle (conceptually: “Block traffic if VPN disconnects” – implemented via OS/browser instructions or helper app later; see MVP note under Tech).
   - Show:
     - Current selected use case.
     - Current selected country/city.
     - IP type: shared vs dedicated vs “clean” tier (if applicable).
   - Optional advanced toggles:
     - DNS leak protection (conceptually; real implementation may be through provider).
     - Auto-connect on login (web session-based).

6. **Use Cases List UI**
   - A "Use Cases" tab or section:
     - Grid or list of use cases with icons.
     - Each item opens a pane with:
       - Description
       - Recommended locations
       - Key tips
       - “Connect with these settings” button (one-click apply & connect).

### 2.2 Admin / Internal Features

1. **Admin Dashboard (Web)**
   - Authentication with role “admin”.
   - **Visual metrics** (initially simple charts) for:
     - Use-case usage stats (e.g., how many sessions use Flights vs Streaming vs Privacy).
     - Location popularity.
     - Connection success/failure rate.
     - Basic anonymized stats for:
       - Flights use case.
       - Streaming use case.
       - Privacy/Anonymity use case.
   - Ability to:
     - Add/edit/delete **use-case recipes** (title, description, recommended countries/cities, recommended IP type).
     - Configure which countries/cities are visible to end users.
     - Flag some locations as “blocked due to sanctions” (not selectable in UI).

2. **Basic Logging (Minimal)**
   - Store:
     - User ID
     - Timestamp
     - Chosen use case
     - Chosen location
     - Connection status (success/failure)
   - No activity/content logs (no per-site browsing history).
   - Aggregate metrics in admin view; preserve privacy.

---

## 3. Non-Functional Requirements

### 3.1 Compliance & Legal

- **Company is US-based.**
- **Host on SOC2-compliant provider** (e.g., AWS, GCP, Azure).
  - **Benefits**:
    - Strong baseline security controls.
    - Easier to pass due diligence (B2B, investors).
    - Clear documentation for data handling, backups, etc.
  - **Drawbacks / tradeoffs**:
    - Potentially higher hosting costs than some non-compliant providers.
    - Vendor lock-in patterns may be stronger.
    - Limited choice if a smaller provider offers cheaper/closer VPN servers but isn’t SOC2.
- **Sanctions-aware**:
  - Block access to some locations due to US legal sanctions (e.g., OFAC lists).
  - Frontend should hide these from users; API should also reject requests.
- **Terms of Service**:
  - Cover:
    - Misuse (illegal activities, harassment, etc.).
    - No guarantee of price manipulation for flights or content availability.
    - Logging policy (minimal logs).
    - Privacy policy disclaimers.
- **Logging**:
  - Minimal technical logs as described.
  - Avoid PII beyond what’s necessary (email, billing, and minimal connection metadata).

### 3.2 Security

- HTTPS everywhere.
- Strong password storage (bcrypt/argon2).
- JWT or secure session cookies for auth.
- Stripe webhooks verified via secret.
- IP address handling:
  - Ensure static IPs and “clean IPs” classification come from provider, not ad-hoc heuristics.
- Multi-tenant safe DB design.

### 3.3 Performance & Reliability

- Provider choice: **Use a reputable VPN/proxy provider** for infrastructure.
  - Requirements:
    - Offers:
      - Static/dedicated IPs.
      - Optionally “residential” or higher-quality IP pools for social media / streaming.
      - City-level selection where needed.
      - API for programmatic control.
    - Strong reliability and throughput.
    - Ability to get “unused / low-abuse history” IPs.
  - Strategy:
    - Start with a single primary provider with strong global coverage.
    - Negotiate for:
      - Clean IP pools.
      - Specific city endpoints for streaming and flights where required.
- Backend must be able to handle:
  - Quick connection negotiation.
  - 0–1000 early users easily (for small paid beta).

---

## 4. Tech Stack & Architecture

### 4.1 Frontend

- Framework: **React** (SPA).
- Styling:
  - Clean, minimal, generic.
  - Follow **Apple Weather app** design cues:
    - Soft gradients or subtle color backgrounds.
    - Card-based layout for each use case.
    - Large, readable temperature-like “connection” indicator.
- Key components:
  - `Auth` pages (Login, Signup, Forgot Password).
  - `Dashboard`:
    - Header: status (Connected/Disconnected), current IP, location.
    - Use Case selector (cards).
    - Quick connect button.
    - Speed/latency meter.
  - `UseCaseDetail`:
    - Short description.
    - Recommended location(s).
    - “Quick tips” with “More details” expand.
    - “Apply & Connect” button.
  - `Settings`:
    - Kill switch toggle (for now: logical/UX-level explanation).
    - Auto-connect toggle.
    - Basic account settings.
  - `Billing`:
    - Plan overview.
    - Link to Stripe customer portal (if used).
  - `AdminDashboard`:
    - Charts, tables for stats.
    - Recipes CRUD.

### 4.2 Backend

- Language: **Node.js** or **Python** (you’re open to both).
  - For fastest MVP under 1 month with many existing libraries, lean slightly toward **Node.js** with Express/Nest for tight integration with React dev stack.
- DB: **PostgreSQL** (fully managed).
- Hosting:
  - Fully-managed (e.g., AWS RDS for Postgres, managed Node.js hosting, or similar PaaS).
  - SOC2-compliant cloud (AWS/GCP/Azure).
- Key services/modules:
  1. Auth service:
     - Email/password registration.
     - JWT-based sessions.
  2. Subscription service:
     - Integrates with Stripe.
     - Webhook listener: updates user subscription status.
  3. VPN session controller:
     - Interacts with external VPN/proxy provider’s API:
       - Create sessions.
       - Select IP, location, city.
       - Configure static/dedicated IP mapping.
     - Handles:
       - `startSession(userId, useCase, location)` → returns VPN config.
       - `endSession(sessionId)`.
  4. Use-case recommender:
     - Maps use case → recommended locations & IP types.
     - Supports dynamic rules (config in DB, editable from admin).
  5. Logging / Metrics:
     - Writes anonymized usage records to DB.
     - Aggregate queries for admin charts.
  6. Access control:
     - Role-based (user vs admin).
     - Sanction/country blocking.

### 4.3 High-Level Architecture

- Client (React) ⇄ API Gateway/Backend (Node/Python) ⇄
  - PostgreSQL (data)
  - Stripe (billing)
  - VPN/Proxy Provider API (infrastructure)
- Admin dashboard reuses same backend with admin-only routes.

---

## 5. VPN / Proxy Layer Design

### 5.1 Provider Strategy

- Use a **single, strong VPN/proxy provider** in MVP that offers:
  - REST API or SDK.
  - Multiple POPs across **Top ~10 countries** (with city options).
  - Static/dedicated IPs.
  - Clean IP pools (residential or premium datacenter).
- Requirements:
  - Ability to tag or differentiate:
    - **“Clean” IPs** (low historical abuse, not flagged for spam/automation).
    - **Standard shared IPs**.
  - Ability to obtain **persistent IPs** per user (for an additional fee).
- MVP decision:
  - Basic mapping:
    - Tier 1: Standard shared IP.
    - Tier 2: Clean IP (add-on fee).
    - Tier 3: Dedicated IP per user (optional add-on).
  - Actual tiering may be purely internal at first (all users on same tier) while pricing is tested.

### 5.2 City-Level & Static IPs

- For **Streaming**:
  - City-level is often preferred (e.g., US – specific city endpoints known to work well).
  - Provider needs to guarantee capacity and low blacklisting.
- For **Flights**:
  - Country-level may be enough but city-level can be used for specific hacks (e.g., booking from city where airline is based).
- For **Static Dedicated IPs**:
  - Assign per-user IP(s) from provider.
  - Persist mapping in DB:
    - `user_id` → `dedicated_ip_id` / `provider_endpoint`.
  - Allow simple UI indicator:
    - “Persistent IP enabled” vs not.

---

## 6. Use Cases & Recipe Details

### 6.1 Use Cases (MVP)

Top-level use cases (can map to multiple countries):

1. **Flights**
   - Goal: find cheaper flights.
   - UX:
     - Suggested locations: e.g., user’s home region + alternate “deal-finder” regions.
     - Tips:
       - Clear cookies and cache.
       - Use incognito/private browsing.
       - Avoid logging in until after price exploration.
       - Try multiple locations and compare.
   - Recipe style:
     - Short card:
       - “Connect to [Country A] or [Country B]. Open your flight site in an incognito window. Don’t log in until you see a price you like. Compare a few regions.”
     - Expand:
       - More detail / step-by-step.

2. **Streaming Services**
   - Sub-use cases:
     1. YouTube TV (location-first).
     2. Netflix.
     3. Prime Video.
   - UX:
     - For each streaming provider, recommended countries/cities.
     - Some with city-level endpoints for reliability.
   - Tips:
     - For YouTube TV:
       - Connect to a US city compatible with home subscription region (if US-based user).
       - Refresh page after connecting.
     - For Netflix / Prime Video:
       - Connect to target country (where catalog you want is available).
       - If blocked, suggest switching to another city/country within same region.

3. **Social Media (Clean IPs)**
   - Sub-use cases:
     - X/Twitter.
     - TikTok.
   - Goal:
     - Stable, clean IPs to reduce risk of friction, captchas, or account flags.
   - UX:
     - Show dedicated/clean IP indicator.
     - Recommended: user’s home region or neighbor region with clean IPs.
   - Tips:
     - Avoid rapidly switching regions.
     - Use persistent IP for main accounts.

4. **Privacy / Anonymity**
   - Goal:
     - Obscure user’s IP and location from everyday websites.
   - UX:
     - “Best privacy” default region (could be privacy-friendly countries).
     - Offer random region selection.
   - Tips:
     - Explain concept in simple terms: “This doesn’t make you invisible, but it helps hide your IP from sites.”
     - Suggest using secure browsers, avoiding personal logins for max anonymity.

5. **(Future use cases)** – placeholder in DB:
   - Sports streaming.
   - Gaming/low latency.
   - Remote work / corporate.

### 6.2 Dynamic Rules

- Use-case rules stored in DB:
  - Example fields:
    - `use_case_id`
    - `name`
    - `description_short`
    - `description_long`
    - `recommended_countries` (JSON list)
    - `recommended_cities` (JSON list, optional)
    - `ip_type_preference` (shared/clean/dedicated)
    - `blocked_countries` (derived from sanctions + provider support)
- Backend rule engine:
  - `getRecommendedConfig(useCase, userProfile)`:
    - Uses DB rules.
    - Excludes sanction-blocked locations.
    - May adjust based on provider availability.

---

## 7. Logging & Privacy

### 7.1 What We Log (MVP)

- On each connection attempt:
  - `user_id`
  - `timestamp`
  - `use_case`
  - `country` / `city` chosen
  - `ip_type` (shared/clean/dedicated)
  - `success` / `failure` + basic error code
- No browsing activity or content logs.
- Aggregation only for:
  - Use-case popularity.
  - Location performance.
  - Connection reliability.

### 7.2 Admin Metrics

- **Example charts**:
  1. Connections by use case (Flights/Streaming/Social/Privacy).
  2. Success rate by country/city.
  3. Number of sessions using “clean IPs” vs standard.
  4. For the 3 example major categories (Flights, Streaming, Privacy):
     - Daily/weekly active sessions.
     - Average session length (approx).
- Displayed in a simple admin dashboard.

---

## 8. UX & Visual Design

### 8.1 Design Language

- Style: **fun travel-oriented** with a **power user** undercurrent.
- Visual metaphors:
  - **Travel**: cards like “Flight Mode”, “Beach & Streams”, “Stealth Mode”.
  - Soft gradients reminiscent of sky/sea (Apple Weather style).
- Layout guidelines:
  - Home/dashboard:
    - Top: big status indicator (similar to temperature: “Connected to: Paris, France”).
    - Middle: Use case cards horizontally scrollable or grid.
    - Bottom: connection details + tips.

### 8.2 Interaction Patterns

- **One-click connect**:
  - User chooses use case → defaults to best suggested location → click “Connect”.
- **Guided help**:
  - Under each use case, show “Quick tips (15–30 words)” then “Show details” to expand.
- **Minimal friction**:
  - Only show advanced toggles after user clicks “Advanced”.

---

## 9. Pricing & Plans (MVP Logic)

- **Paid-only SAAS**.
- Plans:
  1. **Monthly**:
     - Base price (TBD).
  2. **Yearly**:
     - Equivalent to e.g., ~2 months free vs monthly.
  3. Optional add-ons (later or v1 if time permits):
     - “Clean IP boost” (access to higher-quality IP pools).
     - “Dedicated IP” per region.
- Stripe:
  - Create products/prices.
  - Integrate checkout.
  - After payment, redirect back to app to activate subscription.
  - Stripe Customer Portal for billing management.

---

## 10. Data Model (High-Level)

### 10.1 Main Tables (Postgres)

- `users`
  - `id`, `email`, `password_hash`, `role` (user/admin), `created_at`.
- `subscriptions`
  - `id`, `user_id`, `stripe_customer_id`, `stripe_subscription_id`, `status`, `plan`, `current_period_end`.
- `vpn_sessions`
  - `id`, `user_id`, `use_case_id`, `country`, `city`, `ip_type`, `provider_session_id`, `status`, `started_at`, `ended_at`.
- `use_cases`
  - `id`, `key`, `name`, `description_short`, `description_long`, `created_at`.
- `use_case_rules`
  - `id`, `use_case_id`, `recommended_countries` (JSON), `recommended_cities` (JSON), `ip_type_preference`, `notes`.
- `clean_ip_assignments`
  - `id`, `user_id`, `ip_address`, `provider_endpoint_id`, `status`, `created_at`.
- `provider_endpoints`
  - `id`, `provider`, `country`, `city`, `ip_type`, `sanction_blocked`, `status`.
- `metrics_logs`
  - `id`, `user_id`, `use_case_id`, `country`, `city`, `ip_type`, `success`, `error_code`, `timestamp`.

---

## 11. MVP Implementation Phases (<= 1 Month)

### Week 1: Foundations

1. Finalize provider choice (VPN/proxy).
2. Set up:
   - Repo, CI/CD pipeline.
   - Basic React app scaffold.
   - Backend scaffold (Node.js/Express or NestJS).
   - Managed Postgres.
3. Implement:
   - User auth (email/password, JWT).
   - Basic Stripe integration (test mode).
4. Begin schema definition & migrations.

### Week 2: VPN Integration & Core UX

1. Implement VPN provider API client.
2. Implement:
   - `startSession` / `endSession` endpoints.
   - Mapping use case → provider endpoint (basic rules).
3. Build frontend:
   - Dashboard layout.
   - Connect/Disconnect flow.
   - Show connection status & simple metrics (latency, IP).
4. Seed DB with initial countries/cities and use cases.

### Week 3: Use-Case Recipes & Admin

1. Implement **Use Case Settings Guide** on frontend:
   - Short tips + expandable details per use case.
2. Admin dashboard:
   - Basic login.
   - CRUD for:
     - Use cases.
     - Rules / recommended locations.
   - Simple metrics charts (e.g., line/bar charts).
3. Logging in backend:
   - Save `metrics_logs` on connection attempts.

### Week 4: Polish, Compliance, and Beta Prep

1. Final pass on Legal:
   - Terms of Service, Privacy Policy text.
   - Sanctions-affected countries flagged & tested.
2. UI polish to align with:
   - Travel-inspired, Apple Weather-like styling.
3. Testing:
   - Functional tests for:
     - Subscription flow.
     - Connection success/failure.
     - Kill-switch messaging (even if full OS implementation is v2).
4. Beta onboarding:
   - Create a simple landing page summarizing:
     - Use cases.
     - Features.
     - Pricing.
   - Enable production Stripe keys & provider production endpoints.

---

## 12. Known Tradeoffs & Future Enhancements

- **Kill Switch**:
  - Full system-level kill switch not fully doable in a pure web app (requires OS-level or VPN client).
  - MVP can:
    - In browser: cut off app’s requests and show user instructions for system-level settings.
  - Future: native apps that implement real kill switch.

- **Quality vs Cost**:
  - Clean IPs and static IPs will cost more from providers.
  - MVP should likely:
    - Start everyone on the same quality tier.
    - Later introduce pricing segmentation.

- **Device Coverage**:
  - MVP: primarily browser-based.
  - Later:
    - Native iOS/Android clients.
    - Browser extensions.
    - Desktop apps.

- **Privacy Guarantees**:
  - Communicate clearly that:
    - No activity logs are stored.
    - Only connection metadata for uptime/quality purposes is stored.
  - Future: external audits, privacy certifications.

---

End of `vpn_build_plan.md`