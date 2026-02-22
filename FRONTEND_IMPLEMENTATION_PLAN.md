# Smart Building Platform Frontend-First Implementation Plan

This document resets delivery order to **frontend first** so we can lock UX, role-based flows, and offline behavior before deep backend buildout.

## 1) Product & UX Discovery (Week 1)
1. Confirm primary personas: Admin, Facility Manager, Occupant.
2. Map top 5 user journeys end-to-end (first login, access request, submit vote, FM alert review, schema change rollout).
3. Define UI principles: clean, low-cognitive-load, mobile-first, accessible.
4. Produce information architecture for navigation by role.
5. Agree on MVP scope boundaries for the first release.

## 2) Design System Foundation (Week 1-2)
1. Define brand tokens:
   - Color palette (primary, secondary, neutral, semantic states).
   - Typography scale.
   - Spacing, radius, elevation.
2. Build a component inventory:
   - Buttons, inputs, cards, tabs, modal, toasts, tables, empty/error states.
3. Add accessibility constraints:
   - WCAG color contrast targets.
   - Keyboard navigation and focus states.
   - Screen-reader labels and landmarks.
4. Create role-aware layout templates (Admin/FM/Occupant).
5. Set motion guidelines (subtle transitions, reduced motion support).

## 3) Frontend Codebase Planning (Week 2)
1. Choose framework and app architecture (e.g., Next.js + TypeScript).
2. Define folder/module structure by domain:
   - `features/voting`
   - `features/presence`
   - `features/analytics`
   - `features/config`
3. Define state strategy:
   - Server state (React Query/SWR).
   - Client state (Zustand/Redux as needed).
4. Define API client standards:
   - Tenant-aware auth headers.
   - Typed request/response contracts.
   - Global error handling and retry policy.
5. Define form strategy (schema-driven forms + validation).

## 4) Offline-First UX & Data Sync Contract (Week 2-3)
1. Define local queue model for votes (IndexedDB/local storage strategy).
2. Define sync states visible to users:
   - Pending sync.
   - Synced.
   - Duplicate accepted.
   - Validation failed.
3. Define conflict UX copy and retry behavior.
4. Define network-awareness behavior (online/offline banners, manual sync trigger).
5. Create telemetry events for sync reliability and user-facing failure rates.

## 5) Screen-by-Screen Delivery Plan (Week 3-6)
1. Auth & session screens (login, token refresh behavior, role routing).
2. Occupant experience:
   - Presence context indicator.
   - Voting flow (fast submit, offline queue handling).
   - “Feedback received” confirmation.
3. Facility Manager dashboard:
   - Live zone health cards.
   - Trend charts (using mocked + staged data first).
   - Alert inbox and broadcast composer.
4. Admin workspace:
   - Tenant/building management.
   - Role assignment tools.
   - Audit viewer shell.
5. Settings and notification preference center.

## 6) Frontend Quality Gates (Week 4-7)
1. Storybook coverage for all shared components.
2. Unit tests for critical hooks/utilities.
3. E2E tests for top user journeys across roles.
4. Performance budget checks (LCP/CLS/TTI targets).
5. Accessibility audit before release candidate.

## 7) Backend Integration Sequencing (Frontend-Driven)
1. Integrate Auth API first (role/tenant claims).
2. Integrate Building/Config APIs next (required for rendering dynamic forms).
3. Integrate Vote batch endpoint and offline sync responses.
4. Integrate Presence and Analytics streams.
5. Integrate Notification and Audit APIs.

## 8) Non-Delegable Decisions You Must Make
1. Final visual direction and color palette approval.
2. Accessibility compliance target (WCAG AA vs AAA by surface).
3. Supported device/browser matrix and minimum OS versions.
4. Localization scope (English-only vs multilingual launch).
5. UX trade-off decisions (speed vs depth of data on dashboards).

## 9) 30/60/90-Day Frontend Milestones
- **Day 30**: Design system + role-based wireframes + prototype validated.
- **Day 60**: Occupant and FM core flows production-ready with mocked fallback data.
- **Day 90**: Full role experiences integrated with backend APIs + quality gates passed.

---

## Frontend Discovery Questions (Please answer these first)
1. What **color palette** do you want (or brand examples you like)?
2. Do you want a **light-only UI** or **light + dark mode** at launch?
3. What tone should the UI feel like: **corporate**, **modern minimal**, or **friendly consumer**?
4. Which platforms are launch-critical: **mobile web**, **desktop web**, and/or **native wrapper**?
5. What accessibility bar do you want for MVP (WCAG AA minimum?)
6. Which role should we optimize first for UX polish: **Occupant**, **Facility Manager**, or **Admin**?
7. Should dashboards prioritize **live data visibility** or **historical analysis depth** for v1?
8. Any reference products whose UX you admire?
