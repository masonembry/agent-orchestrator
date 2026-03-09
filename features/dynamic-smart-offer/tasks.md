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
- [x] `handleDismiss` — dismisses tone guidance in awaiting and success states
- [x] Dev tool `EventEmitter` — draggable popover with GAIA tab + Logic Builder tab

### To Do

- [ ] Emit `offerGenerated: boolean` analytics event when first WS event is received
- [ ] Remove debug `console.log("MASONLOG", ...)` from `FullOfferChecklist.tsx`
- [ ] Frontend WS handler for `messageType: 'dynamic-smart-offer-event'` → emit to `logicBuilderEventBus`

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

- [ ] Kafka-Connect Lambda Sink Connector setup — Event Streaming team configuring Confluent K8s connector to invoke Lambda ([Slack thread](https://asurion-teams.slack.com/archives/C0166CTKQET/p1772822416750369?thread_ts=1772821725.447529&cid=C0166CTKQET))
- [ ] Data shape validation — confirm Kafka payload matches Lambda Zod schema once connector is live
- [ ] E2E integration test — send test events via [Event Streaming Portal (nonprod)](https://event-streaming-portal-nonprod.edpapi.npr.aws.asurion.net/kafka-tester); verify Kafka → Lambda → EventBridge → WebSocket → frontend store

---

## expert-workspace-config (Infra)

### Done

- [x] `isDynamicSmartOfferEnabled` feature flag JSON created in all 6 environments (nonprod, prod, gcp-nonprod, gcp-prod, saas-nonprod, saas-prod) — default `false`, no rules — draft PR #589
- [x] Contentful CMS: `sectionType` and `enableDynamicContent` fields added to `checklistSection` content type

### To Do

- [ ] Enable `isDynamicSmartOfferEnabled` flag in nonprod once integration test passes
