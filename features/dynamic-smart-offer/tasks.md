# Dynamic Smart Offer — Tasks

Grouped by repo. Check items off as PRs land.

---

## expert-workspace (Frontend)

### Done

- [x] `DynamicSmartOffer` class — `toneGuidance`, `pitchPoints`, `isValid()`, `equals()`; with unit tests
- [x] `useDynamicSmartOfferStore` — per-session Zustand store (persist + immer + devtools), `getSession` / `receiveData` / `reset`; with unit tests
- [x] `useDynamicSmartOfferListener` — subscribes to `logicBuilderEventBus`, converts payload → `DynamicSmartOffer`, writes to store
- [x] `useDynamicSmartOffer` hook — wraps store + listener, manages `isPending` / `isAwaitingData` / `isEnabled` / `hasNewerData` / `showLatest`; with unit tests
- [x] `logicBuilderEventBus` — `TinyEmitter` for Logic Builder WS events; `__logicBuilder.emitDynamicSmartOffer()` on `globalThis` for console testing (non-prod)
- [x] `DynamicToneGuidance` component — loading / awaiting / success states, stories, tests
- [x] `DynamicCoreBenefits` component — loading / awaiting / success / fallback states, stories, tests
- [x] Shared `Sparkle` component with `color` and `size` props
- [x] `@tabler/icons-react` added to `sales-tools` package
- [x] `SalesChecklistSection` type updated with `sectionType?` and `enableDynamicContent?`
- [x] `@expert/sales-tools` package wired up with `main` entry and barrel `index.ts`
- [x] `@expert/sales-tools` added as dependency in `expert-ui`
- [x] `DynamicToneGuidance` wired into `FullOfferChecklist` above the `<Accordion>`
- [x] `DynamicCoreBenefits` wired into `FullOfferChecklist` swap logic (`sectionType === "coreBenefits" && enableDynamicContent`)
- [x] `isDynamicSmartOfferEnabled` feature flag added to `WorkspaceFeatures` type and defaults (`packages/features`) — default `false`
- [x] `isDynamicSmartOfferEnabled` wired into `FullOfferChecklist` via `useFeatures<WorkspaceFeatures>()`, gates `isEnabled` on both components
- [x] `isError` replaced with `isAwaitingData` throughout — no error state; UI either has data or is waiting
- [x] "Try again" button renamed to "Show latest"; only renders when `hasNewerData` is true
- [x] `DynamicToneGuidance` awaiting state — neutral `dark.6` bg, Sparkle icon, "Personalized guidance pending"
- [x] Dev tool `EventEmitter` — draggable popover with GAIA tab + Logic Builder tab
- [x] Close button removed from `DynamicToneGuidance` — tone guidance is always visible while enabled
- [x] Sparkle icon in "Guidance" pill is white — CSS `brightness(0) invert(1)` filter on the white Sparkle variant
- [x] Auto-fetch on `isAwaitingData && hasNewerData` — `useDynamicSmartOffer` triggers the 2s pending delay automatically when an event arrives during the awaiting state; no "Awaiting Data With Newer Available" UI state

### To Do

- [x] Emit `DynamicSmartOfferShown` analytics event when dynamic content is first rendered — dispatched once per session in `useDynamicSmartOffer` when `data` first becomes valid
- [x] Remove debug `console.log("MASONLOG", ...)` from `FullOfferChecklist.tsx`
- [x] Frontend WS handler for `messageType: 'dynamic-smart-offer-event'` → emit to `logicBuilderEventBus` — generic dispatch in `useWebsocket.ts` (line 235); `DynamicSmartOfferEvent` type + payload registered in `expert-workspace-websocket/src/types.ts`

---

## expert-workspace-api (Backend)

### Done

- [x] Eligibility API: `sectionType` and `enableDynamicContent` mapped in `buildChecklistSections.ts`
- [x] Types updated in `packages/shared-types/contentful-product-info.ts` and `services/sales/lib/shared/utils/contentful/types.ts` — branch `RDESQ-1900-dynamic-smart-offer`
- [x] `services/sales/lib/logic-builder-kafka-sink/handlers/handler.lambda.ts` — Lambda handler (batch-aware, latest-record-wins, Zod validation)
- [x] `services/sales/lib/logic-builder-kafka-sink/handlers/types.ts` — Zod schemas
- [x] `services/sales/lib/logic-builder-kafka-sink/src/services/sendDynamicSmartOfferToFrontend.ts` — EventBridge dispatch (`messageType: 'dynamic-smart-offer-event'`, routed by `employeeId`)
- [x] `services/sales/cdk/constructs/lambdas/createLogicBuilderKafkaSink.ts` — CDK construct; K8s node group invoke grants; EventBridge PutEvents grant
- [x] `services/sales/cdk/stack.ts` — Lambda wired into sales stack

### To Do

- [ ] E2E integration test — can run NOW against PR environment Lambda (already deployed, Kafka integration live); send test events via [Event Streaming Portal (nonprod)](https://event-streaming-portal-nonprod.edpapi.npr.aws.asurion.net/kafka-tester); verify Kafka → Lambda → EventBridge → WebSocket → frontend store
- [ ] Data shape validation — confirm Kafka payload matches Lambda Zod schema (do as part of E2E test above)
- [ ] Deploy Nonprod Lambda (`EWP-Sales-logic-builder-kafka-sink`) — PR environment Lambda is live but Nonprod Lambda is not yet deployed
- [ ] After Nonprod Lambda deploy: trigger eds-platform-connectors deploy to activate the Nonprod connector config

---

## expert-workspace-config (Infra)

### Done

- [x] `isDynamicSmartOfferEnabled` feature flag JSON created in all 6 environments (nonprod, prod, gcp-nonprod, gcp-prod, saas-nonprod, saas-prod) — default `false`, no rules — draft PR #589
- [x] Contentful CMS: `sectionType` and `enableDynamicContent` fields added to `checklistSection` content type
- [x] `isDynamicSmartOfferEnabled` enabled in Nonprod (`true`)

### To Do

- [ ] Prod: enable `isDynamicSmartOfferEnabled` for a cohort of agents (TBD) as part of prod rollout

---

## eds-platform-connectors (Kafka Connector)

### Done

- [x] Nonprod connector configs merged (dev testing + Nonprod): `ewp-sales-dynamic-smart-offer-att.json`, `ewp-sales-dynamic-smart-offer-verizon.json` — dev testing connector is live; Nonprod connector is merged but inactive until Nonprod Lambda is deployed
- [x] Prod connector configs — draft PR #770, do not merge until nonprod testing passes
  - `paas.simplr.smart-offer-verizon-exwo` / `paas.simplr.smart-offer-att-exwo` (ExWo)
  - `paas.simplr.smart-offer-verizon-alpha` / `paas.simplr.smart-offer-att-alpha` (Alpha — prod only)
  - Lambda ARN: `arn:aws:lambda:us-east-1:737463266258:function:EWP-Sales-logic-builder-kafka-sink`

### To Do

- [ ] Deploy eds-platform-connectors (Nonprod) after Nonprod Lambda is live — activates the merged connector config
- [ ] Merge and deploy eds-platform-connectors prod PR #770 after nonprod testing passes

---

## enterprise-eventing (Kafka Connect Permissions)

### Done

- [x] Nonprod: `EWP-Sales-*logic-builder-kafka-sink` Lambda ARN added to Kafka Connect invoke grants (`infra/cdk/configs/non-prod/us-east-1/non-prod-connect.ts`) — PR open on branch `RDESQ-1900-dynamic-smart-offer`
- [x] Prod: `EWP-Sales-*logic-builder-kafka-sink` Lambda ARN added to Kafka Connect invoke grants (`infra/cdk/configs/prod/us-east-1/prod-connect.ts`) — draft PR #9107, do not merge until nonprod testing passes

### To Do

- [ ] Merge and confirm deployment of prod PR #9107 as part of prod rollout — must land before prod eds-platform-connectors connector is activated
