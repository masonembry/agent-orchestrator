# Dynamic Smart Offer — Design

Technical decisions, architecture, and open questions.

---

## Integration Architecture

```
Logic Builder → Kafka (ExWo_VZ / ExWo_ATT topics)
                  ↓
          [Lambda sink — services/sales]
                  ↓
          sendEventToFrontend (EventBridge → websocket-service)
                  ↓
          Frontend WS → logicBuilderEventBus → useDynamicSmartOfferStore
```

**Key decisions:**
- Push, not pull — frontend does not request data; Logic Builder publishes and frontend receives
- Multiple events per session — Logic Builder re-sends as transcript accumulates; each event replaces the previous in the store
- No detection models for pitch points in MVP

### Kafka Topics

| Topic | Env | Owner |
|---|---|---|
| `paas.simplr.smart-offer-verizon-exwo` | Nonprod + Prod | ExWo — Verizon |
| `paas.simplr.smart-offer-att-exwo` | Nonprod + Prod | ExWo — AT&T |

Alpha topics (`paas.simplr.smart-offer-verizon-alpha`, `paas.simplr.smart-offer-att-alpha`) are out of scope for MVP.

### WS Message Type

`messageType: 'dynamic-smart-offer-event'` — dispatched generically, no explicit frontend registration needed.

---

## Frontend State Model

### Data class

```typescript
// apps/expert-ui/src/sales-tools/dynamic-smart-offer/DynamicSmartOffer.ts
class DynamicSmartOffer {
    toneGuidance: string;
    pitchPoints: string[];
    isValid(): boolean; // true if toneGuidance non-empty and at least one pitchPoint non-empty
}
```

Default/empty: `new DynamicSmartOffer("", [])` — `isValid()` returns `false`.

### Three-layer architecture

**1. `useDynamicSmartOfferStore`** — Zustand store (persist + immer + devtools), per-session:
- `getSession(sessionId)`, `receiveData(sessionId, data)`, `reset(sessionId)`
- Purely a data cache. No status tracking — that belongs in the hook.

**2. `useDynamicSmartOfferListener`** — subscribes to `logicBuilderEventBus` for `"dynamic_smart_offer"` events, converts payload → `DynamicSmartOffer`, writes to store via `receiveData`.

**3. `useDynamicSmartOffer(sessionId, salesChecklist)`** — main consumption hook (note: `pitchPoints` stays `string[]` in the event/store; MVP sends one item, future expansion requires no frontend changes):
```typescript
{ data: DynamicSmartOffer, isPending: boolean, isAwaitingData: boolean, isEnabled: boolean, hasNewerData: boolean, showLatest: () => void }
```
- Mounts `useDynamicSmartOfferListener` internally
- On mount and on `showLatest`: sets `isPending = true`, waits 2s (`FETCH_DELAY_MS`), then reads current store state
- `isEnabled = isDynamicSmartOfferEnabled && hasCoreBenefitsWithDynamicContent(salesChecklist)`
- `isAwaitingData = !isPending && !data.isValid()`
- `hasNewerData = true` when store has valid data differing from what's currently displayed

### State rendering matrix

| Section | `isPending` | `isAwaitingData = false, data.isValid()` | `isAwaitingData = true` |
|---|---|---|---|
| Tone Guidance | Skeleton | Generated string | Placeholder UI |
| Personalized benefit (above Core Benefits) | Skeleton | Generated benefit (with Sparkle icon) | Not rendered |
| Static Core Benefits | Rendered (always) | Rendered (always) | Rendered (always) |
| Example of Coverage | Hidden | Hidden | Hidden |
| All other sections | Render normally | Render normally | Render normally |

### Section identification

Swap logic uses `section.sectionType === "coreBenefits" && section.enableDynamicContent` rather than matching on title. Both fields are optional on `SalesChecklistSection` for backwards compatibility.

---

## Component Tree

```
FullOfferChecklist
  ├─ DynamicToneGuidance           (above Accordion)
  └─ Accordion
      └─ [per section]
          ├─ DynamicCoreBenefits   (when sectionType === "coreBenefits" && enableDynamicContent)
          │     ├─ personalized benefit row (Sparkle icon, when data.isValid())
          │     │     OR skeleton (when isPending)
          │     │     OR nothing (when isAwaitingData)
          │     └─ ChecklistSectionWithErrorBoundary  (static Core Benefits — always rendered)
          └─ ChecklistSectionWithErrorBoundary  (all other sections)
                └─ ChecklistSection
                    └─ ChecklistItem  (per item)
```

**Key files:**
```
apps/expert-ui/src/sales-tools/dynamic-smart-offer/
  DynamicSmartOffer.ts
  useDynamicSmartOfferStore.ts
  useDynamicSmartOfferListener.ts
  useDynamicSmartOffer.ts
  logicBuilderEventBus.ts

packages/sales-tools/src/modules/dynamic-smart-offer/
  DynamicToneGuidance/
  DynamicCoreBenefits/
  Sparkle/

apps/expert-ui/src/sales-tools/home-product/components/
  FullOfferChecklist/FullOfferChecklist.tsx
```

### Layout order (from design)

1. Tone Guidance (above all checklist sections)
2. Discovery question
3. Engagement question
4. Transition statement
5. Pitch Points slot (where Core Benefits currently lives)
6. Pricing transparency

---

## Backend Lambda

**Repo:** `expert-workspace-api` / **Service:** `services/sales/`

Lambda is invoked by Confluent K8s Lambda Sink Connector as a batch (`KafkaConnectLambdaEvent = KafkaConnectEvent[]`). Takes only the last record (latest value wins).

**Confirmed connector shape (observed in nonprod):**
- `event[n].payload.value` — JSON-encoded string of the event data directly (not an inner-records array)
- Parser: `JSON.parse(event[last].payload.value)` → Zod validate

**Kafka input schema (Zod-validated):**
```typescript
{ sessionId: string, employeeId: string, toneGuidance: string, pitchPoints: string[] }
```

IAM: K8s node group roles granted `lambda:InvokeFunction`, `lambda:InvokeAsync`, `lambda:GetFunction`.
Lambda granted `events:PutEvents` on the ExWo EventBridge bus.

---

## Contentful → App Data Flow

Contentful is not fetched directly by the frontend. The backend eligibility API (`POST /v2/eligibility`) fetches from Contentful and transforms into `ProductInfo`, embedded in the eligibility response.

```
POST /v2/eligibility
  → CheckEligibilityResult
      → EligibleProduct[]
          → productInfo: Record<LocaleCode, ProductInfo>
              → salesOfferChecklist: SalesChecklist
                  → checklistSections: SalesChecklistSection[]  ← includes sectionType + enableDynamicContent
```

---

## Zustand Stores

| Store | File | Purpose |
|---|---|---|
| `useProductOfferStore` | `stores/useProductOfferStore.ts` | Products, eligibility, `ProductInfo` |
| `useOfferChecklistStore` | `stores/useOfferChecklistStore.ts` | Detection state per item key per session |
| `useDynamicSmartOfferStore` | `sales-tools/dynamic-smart-offer/useDynamicSmartOfferStore.ts` | Latest DS data per session |
| `useIntegratedFlowStore` | `stores/useIntegratedFlowStore.ts` | Current step in traditional flow |

---

## Open Questions

- [ ] What is the minimum transcript data threshold for DS generation? When in a typical call is it reached?
- [ ] GAIA event delivery resilience strategy for checklist detection events (missed events, re-send)?
  - Can all checked items be sent on every event so a missed one is corrected on the next?
  - Can there be a scheduled re-send even when no new item is checked?
- [ ] Who is responsible for compiling success metrics downstream?
- [ ] Single, public communication forum for this effort (all stakeholders)?
- [ ] What is needed from C360 team re: historical transcript availability timing (post-MVP)?

### Resolved

- ✅ **Backend service:** Logic Builder (not GAIA) drives content generation for MVP.
- ✅ **Pitch point pool ownership:** Logic Builder owns the pool and sends rendered content. ExWo just renders.
- ✅ **No detection models for pitch points:** Static detection architecture maintained for all other sections.
- ✅ **Generated offer does not mutate during a call:** Content is dynamic at fetch time only. No mid-call updates unless expert clicks "Show latest" (only visible when newer data is in store).
- ✅ **Success metrics instrumentation:** ExWo emits `offerGenerated: boolean` so downstream can track dynamic vs. static fallback.
- ✅ **Chat is out of scope:** Voice feature only. Chat handles its own implementation separately.
- ✅ **Kafka topic config:** Nonprod topic names confirmed. Connector setup delegated to Event Streaming team.
- ✅ **WS message type registration:** `'dynamic-smart-offer-event'` dispatched generically — no explicit registration needed.
