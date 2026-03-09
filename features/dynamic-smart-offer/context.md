# Dynamic Smart Offer — Engineering Context

> Living reference document for the Dynamic Smart Offer feature in `expert-workspace`.
> Source material: Notion PRD, MVP Scope doc, Engineering notes, Field/Alpha research.

---

## Overview

**What:** Evolve the existing "Smart Offer" into a context-aware system that dynamically generates personalized offer guidance based on who the customer is and what we know from their prior interactions.

**Jira:** [RDESQ-1900](https://solutonashville.atlassian.net/browse/RDESQ-1900)
**Target Launch:** Q1 pilot by March 31, 2026
**Authors:** Eliza Wall & Alicia James (PM/Product)
**Team:** Expert Workspace — Expert Product: Expert Workspace | Sell
**Estimated dev:** 11 weeks / 1 engineer

---

## Problem

The current Smart Offer is static and does not leverage existing customer context:
- Messaging is generic and misaligned to customer needs
- Known objections are not proactively addressed
- Tone is often transactional/tone-deaf

We are underutilizing customer signals that could materially improve offer conversion.

---

## Hypothesis & Evidence

Context-aware guidance (tone + benefit prioritization) derived from prior customer interactions will increase conversion rates and create stickier sales.

**Field "Smart Pitch" Pilot Results (DiD analysis):**
- Combined SP100: Test +1.60 vs Control +0.06
- HTP: Test +2.81 vs Control +1.08
- VHDP: Test +1.36 vs Control -0.82
- Statistical significance: 80% confidence level (1-in-5 chance of false positive)
- Note: Low click-through rate (~5%) limits signal strength

**Alpha "Personalized Offer" Results:**
- Positive for Cricket, negative for Verizon
- Key issues: inaccurate language ("all devices" vs "eligible devices"), compliance/trust concerns from over-claiming
- Learning: AI-generated copy underperformed marketing-written language → ground outputs in pre-approved pitch pool

---

## MVP Scope (Q1)

### What's In

**Data Science Inputs (real-time transcript only — MVP):**
- Real-time call transcript
  - Tone / emotional cues
  - Customer attributes inferred from transcript (tech savviness, presence of children, number/type of devices)

**Data Science Outputs:**
1. **Tone Guidance** — concise recommendation on how the expert should frame the offer
   - e.g., "Empathize with busy family life. Focus on simplicity and future-proofing their growing tech needs."
2. **Three Pitch Points** — selected from a pool of ~20 pre-approved, marketing-written pitch points, ranked dynamically by transcript signals; light contextual tailoring only (no net-new generation)

**UI Changes in Workspace:**
1. Tone Guidance section injected above offer scripting
2. Three Pitch Points replace the current "Core Benefits" section
3. "Example of Coverage" section removed (intent fulfilled by personalized pitch points)
4. Manual "Refresh" button (no auto-refresh once offer is visible)
5. Static fallback when DS output is unavailable (existing three static benefits shown; placeholder for tone guidance)

**Static sections remain unchanged:**
- Transition statement
- Discovery question
- Engagement question
- Price
- Ask for sale
- Ask to send link
- All existing detection models maintained

### What's Out (MVP Exclusions)

- Historical call transcripts
- Claims data (device type, peril, claim status)
- C360 / prior call history
- Callback awareness
- Objection-aware guidance (planned for post-MVP)
- Rebuttal/objection handling triggered live during call

### Post-MVP / Future Roadmap

- Dynamic objection handling (rebuttals) triggered during live call
- Personalized transition statements, discovery questions, engagement questions
- Expanded inputs: historical transcripts, claims data, C360
- Logic to determine most likely product to purchase
- Personalized visual aids / pricing scenarios for multi-device households

---

## Content Generation & Refresh Logic

| Trigger | Behavior |
|---|---|
| Expert clicks plan card (enters offer flow) | Attempt generation. If sufficient transcript data → generate Tone + Pitch Points. If not → show placeholder + static benefits. |
| Inside offer flow | No automatic refresh. Stable view. |
| Manual "Refresh" click | Regenerate on demand (e.g., if opened before enough transcript data was available) |
| Exit + re-enter offer flow | Regenerate on next entry |

**Open Decision:** How much transcript data (volume/time into call) is required before DS output is reliable? DS to specify a minimum threshold and return an "insufficient data" flag below it. DS should also estimate how many minutes into a call this threshold is typically reached.

---

## Fallback / Degraded State

When DS output is unavailable (insufficient transcript data or generation failure):
- Tone Guidance: show placeholder UI
- Pitch Points: show existing three static Smart Offer benefits:
  - "This plan covers breakdowns on all eligible products for one monthly price no matter when or where they were purchased!"
  - "The products you currently own are covered and future purchases are automatically included; No receipts or registration required"
  - "A service fee of no more than $99+ tax is required per claim, and your home tech will be repaired, replaced, or reimbursed."

The existing "Core Benefits" `SalesChecklistSection` is the static fallback. Never show both the static and personalized pitch points at once.

---

## Integration: Logic Builder → Kafka

Logic Builder is the backend service that drives content generation (confirmed over GAIA for MVP).

### Kafka Topics

Logic Builder publishes to 4 Kafka topics. ExWo consumes from 2:

| Topic (Prod) | Topic (Nonprod) | Owner |
|---|---|---|
| *(TBD)* | `paas.simplr.smart-offer-verizon-alpha` | Alpha — not consumed by ExWo |
| *(TBD)* | `paas.simplr.smart-offer-att-alpha` | Alpha — not consumed by ExWo |
| *(TBD)* | `paas.simplr.smart-offer-verizon-exwo` | **ExWo — Verizon** |
| *(TBD)* | `paas.simplr.smart-offer-att-exwo` | **ExWo — AT&T** |

### Frontend Data Model (Confirmed)

- **Push, not pull.** Logic Builder publishes to Kafka → backend bridges to WebSocket → frontend receives events. The frontend does NOT request data on-demand.
- **Multiple events per session.** As Logic Builder accumulates more transcript data, it re-sends updated events with better content. Each new event replaces the previous.
- **Persist in a Zustand store.** The WS event payload must be stored and updated as new events arrive. A re-render must never lose previously received data.
- **"Try again" = show latest persisted version.** No network call. "Try again" reads whatever is currently in the store and re-displays it, with a brief fake loading delay (~2s) to give the UX a sense of refresh.
- **No meaningful loading time in practice.** The "pending" UI state will be faked (~2s delay) rather than reflecting real network latency.

### Architectural Implications

| Concern | Approach |
|---|---|
| Data fetching | None — pure WS push, no React Query |
| Persistence | New Zustand store (per session, updates on each WS event) |
| `isPending` | Artificial ~2s delay after "Try again" or on initial entry to offer flow |
| `isError` | TBD — likely "entered offer flow and no event received within X seconds" |
| `handleTryAgain` | Read latest from store + trigger fake `isPending` delay |
| `handleDismiss` | Dismiss the tone guidance UI (no data effect) |

### Open Questions (Integration)

- **Kafka → Frontend bridge:** What sits between Kafka and the WS the frontend connects to? (ExWo API, a new service?) TBD.
- **Payload shape:** What does Logic Builder publish per event? Expected: `toneGuidance: string`, `pitchPoints: [string, string, string]` — not yet confirmed.
- **WS event type/name:** What is the event key the frontend listens for?
- **Error threshold:** How long do we wait before declaring `isError`? Who decides?
- **Who owns the pitch point pool?** Assumed Logic Builder sends rendered strings directly — not yet confirmed.
- **"Try again" signal:** Does ExWo need to signal Logic Builder when the expert clicks "Try again"? Or is Logic Builder already re-publishing on its own cadence?

---

## Key Engineering Notes

- The "Core Benefits" `SalesChecklistSection` is the **static fallback** for the new "Personalized Pitch Points" section. Never render both.
- The "Personalize the Coverage" (aka "Example of Coverage") section is hidden in MVP.
- Detection models for static sections (transition, discovery, engagement, price, ask for sale, ask to send link) are maintained as-is.
- No detection models for the new Pitch Points section in MVP.

---

## UI Reference

- Layout mirrors Field "Smart Pitch" UI pattern (see screenshot in `notion-export/Dynamic Smart Offer (aka Smart Pitch Personalized /MVP Scope/Screenshot_2026-02-12_at_6.55.56_PM.png`)
- Insert Tone Guidance above the offer scripting
- Reuse existing Smart Offer layout and scripting structure wherever possible

---

## Success Metrics

**Primary:**
- Offer usage (adoption rate)
- NSP100

**Secondary:**
- NPS
- Compliance rate

**Measuring usage of generated content:**
- DS: Retroactively compare full transcript to generated content to infer usage (~400–500 call batch)
- Quality team: Targeted manual review
- Future: Real-time detection models for expert mention of generated content

---

## Constraints & Risks

- Prior call transcripts may be incomplete, delayed, or unavailable (MVP sidesteps this by using real-time only)
- Decline reasons not consistently captured across channels
- AI-generated copy shown to underperform pre-approved marketing language (Alpha finding) — MVP uses pre-approved pool to mitigate
- Legal/compliance risk from dynamically generating new benefit copy — confirmed risk from Alpha, avoided by grounding in approved set
- GAIA event delivery reliability — need resilience strategy (see open questions)

---

## Codebase Context (expert-workspace)

**Repo:** `code/expert-workspace`
**App:** `apps/expert-ui` — React 19 SPA, Vite 7, TypeScript strict
**UI:** Mantine 8
**State:** Zustand 5 + immer
**Routing:** react-router 7
**Data fetching:** TanStack React Query 5
**Styling:** CSS Modules (no inline styles), Mantine layout components (`Flex`, `Stack`, `Group`)
**Integrations:** Amazon Connect (chat + voice streams), Twilio
**Testing:** Vitest (unit), Playwright (E2E)
**Linting/Formatting:** oxlint + oxfmt (4-space indent, double quotes, 120 char line width)
**Commits:** Conventional commits — `type(scope): description`

---

## Existing Smart Offer — Code Deep Dive

### Contentful Types

**File:** `packages/contentful/src/types.ts`

The Contentful content type `checklistSection` (raw CMS) maps to `SalesChecklistSection` (app TS type):

```typescript
// App-facing type (transformed)
export interface SalesChecklistSection {
    title: string;
    id: string;
    checklistItems: SalesChecklistItem[];
}

// Contentful raw skeleton
interface ChecklistSection {
    title: EntryFieldTypes.Text;
    id: string;
    checklistItems: EntryFieldTypes.Array<
        EntryFieldTypes.EntryLink<EntrySkeletonType<OfferChecklistItem, "checklistItem">>
    >;
}
```

The Contentful content type `checklistItem` maps to `SalesChecklistItem`:

```typescript
export type SalesChecklistItem = {
    phrase: string;                             // Main display text
    subheading: string | undefined;
    highlightColor: string | undefined;         // Background color when unchecked
    popupLinkText: string | undefined;
    popupDescriptionText: string | undefined;
    key: string;                                // Detection key — must match OfferChecklistKey
    defaultExample: SalesChecklistItemDefaultExample;
    examples: string[];
    icon: string | undefined;
};
```

The checklist lives inside `ProductInfo`, which is embedded in the eligibility API response:

```typescript
export interface SalesChecklist {
    description?: string;
    checklistSections: SalesChecklistSection[];
}

export interface ProductInfo {
    salesOfferChecklist?: SalesChecklist;
    additionalHelpSteps?: SalesChecklist;
    chatSmartOfferMessage?: string;
    chatEnrollmentMessage?: string;
    // ... more fields
}
```

The incoming Contentful JSON for the current "Core Benefits" `checklistSection`:
- `sectionName`: "Smart Offer Core Benefits"
- `title`: "Core benefits"
- `checklistItems`: 4 linked entries (the static benefits)
- Content type ID: `checklistSection`

### Contentful → App Data Flow

Contentful is **not fetched directly by the frontend**. The backend eligibility API (`POST /v2/eligibility`) fetches from Contentful and transforms it into `ProductInfo`, which is embedded in the eligibility response.

```
POST /v2/eligibility
  → CheckEligibilityResult
      → EligibleProduct[]
          → productInfo: Record<LocaleCode, ProductInfo>
              → salesOfferChecklist: SalesChecklist
                  → checklistSections: SalesChecklistSection[]
```

Hook that drives the fetch: `apps/expert-ui/src/sales-tools/home-product/hooks/useEligibility.ts`
Store that holds products: `apps/expert-ui/src/sales-tools/home-product/stores/useProductOfferStore.ts`

### Component Tree (Traditional / Workspace Flow)

```
IntegratedFlow
  └─ FullOfferChecklistStep  (step === "SuggestedOfferStep")
      └─ FullOfferChecklist
          └─ Accordion
              └─ ChecklistSectionWithErrorBoundary  (per section)
                  └─ ChecklistSection
                      ├─ CompletionBadge  (X/Y completed)
                      └─ ChecklistItem  (per item)
                          ├─ GreenCheckMark | UncheckedRadioIcon  (based on detection)
                          ├─ BillingDetailsToolTip  (optional)
                          └─ Carousel (conditional on item.key):
                              ├─ RapportCarousel         (key === "RAPPORT")
                              ├─ CoverageCarousel        (key === "COVERAGE")
                              └─ EngagementCarousel      (key === "DISCOVERY_QUESTION" | "ENGAGEMENT_QUESTION")
```

**Key files:**
```
apps/expert-ui/src/sales-tools/home-product/components/
  IntegratedFlow.tsx
  StepSpecific/FullOfferChecklistStep.tsx
  FullOfferChecklist/FullOfferChecklist.tsx
  FullOfferChecklist/components/ChecklistSection.tsx
  FullOfferChecklist/components/ChecklistItem.tsx
```

The parallel **Chat Smart Offer** flow (`apps/expert-ui/src/components/chat/ChatSmartOffer/`) is **out of scope for MVP** — Workspace only.

### Detection Model Integration

GAIA sends WebSocket events → `useSalesChecklistEventListener` processes them → updates `useOfferChecklistStore`.

**File:** `apps/expert-ui/src/sales-tools/sales-checklist/useSalesChecklistEventListener.ts`

```typescript
// Listens for GAIA WebSocket events
gaiaWsEventBus.on("gaia_ws_message-suggestion", ({ body }) => {
    if (body.suggestion.responseType !== "sales-checklist-mentions") return;

    body.suggestion.sales_checklist_mentions?.forEach((mention) => {
        useOfferChecklistStore
            .getState()
            .setData(sessionGroupId, { [mention.name]: mention.value });
        // mention.name = "RAPPORT" | "COVERAGE" | "BILLING" | etc.
    });
});
```

**File:** `apps/expert-ui/src/sales-tools/home-product/stores/useOfferChecklistStore.ts`

```typescript
// All valid detection keys
type OfferChecklistKey =
    | "PROTECTION" | "NON_PARTNER_DEVICES" | "TECH_SUPPORT" | "FUTURE_DEVICES"
    | "IN_HOME_INSTALLATION" | "IN_STORE_SERVICES" | "ASK_SALE" | "BILLING"
    | "TRANSITION" | "COVERAGE" | "RAPPORT" | "DEVICE_CUE"
    | "ENGAGEMENT_QUESTION" | "DISCOVERY_QUESTION";
```

`ChecklistItem` reads from this store using `item.key` to determine `isDetected`. When `isDetected === true`, the item shows a green checkmark and mutes visually.

**MVP note:** No detection models for the new Pitch Points section in MVP. The detection model architecture stays in place for all static sections (transition, discovery, engagement, price, ask for sale, ask to send link).

### Zustand Stores Summary

| Store | File | Purpose |
|---|---|---|
| `useProductOfferStore` | `stores/useProductOfferStore.ts` | Products, eligibility, `ProductInfo` (contains checklist) |
| `useOfferChecklistStore` | `stores/useOfferChecklistStore.ts` | Detection state per item key per session |
| `useIntegratedFlowStore` | `stores/useIntegratedFlowStore.ts` | Current step in traditional flow |
| `useSmartOfferFlowStore` | `chat/ChatSmartOffer/useSmartOfferFlowStore.ts` | Current step + used actions in chat flow |

### Display Cues (Device Personalization)

```typescript
export interface DisplayCue {
    name: string;
    detected: boolean;
    isDetectedOnPreviousCall: boolean;
    rank?: number;
    type: "DEVICE" | "SOFTWARE";
}
```

Display cues come from the eligibility response and drive the `CoverageCarousel` (the "Example of Coverage" / "Personalize the Coverage" section). **This section is being hidden in MVP** — its personalization intent is superseded by the generated pitch points.

---

## Dynamic Smart Offer — State & Rendering Model

### Data Model

```typescript
// apps/expert-ui/src/sales-tools/dynamic-smart-offer/DynamicSmartOffer.ts
class DynamicSmartOffer {
    toneGuidance: string;
    pitchPoints: string[];
    isValid(): boolean; // true if toneGuidance non-empty and at least one pitchPoint non-empty
}
```

Default/empty instance: `new DynamicSmartOffer("", [])` — `isValid()` returns `false`, which drives the `isError` state.

### State Architecture

Implemented across three layers:

**1. `useDynamicSmartOfferStore`** — Zustand store (persist + immer + devtools), per-session:
```typescript
interface SessionState { latestData: DynamicSmartOffer; }
// Actions: getSession(sessionId), receiveData(sessionId, data), reset(sessionId)
// Default: latestData = new DynamicSmartOffer("", [])
```
The store is purely a data cache. No `isPending` or status tracking — that belongs in the hook.

**2. `useDynamicSmartOfferListener`** — subscribes to `logicBuilderEventBus` for `"dynamic_smart_offer"` events, converts payload to `DynamicSmartOffer`, writes to store via `receiveData(payload.sessionId, ...)`.

**3. `useDynamicSmartOffer(sessionId, salesChecklist)`** — the main consumption hook. Returns:
```typescript
{ data: DynamicSmartOffer, isPending: boolean, isError: boolean, isEnabled: boolean, tryAgain: () => void }
```
- Mounts `useDynamicSmartOfferListener` internally
- On mount (and on `tryAgain`): sets `isPending = true`, waits 2s (`FETCH_DELAY_MS`), then reads current store state and sets `data`
- `isEnabled = isDynamicSmartOfferEnabled && hasCoreBenefitsWithDynamicContent(salesChecklist)`
- `isError = !isPending && !data.isValid()` — error means: not loading, and no valid data yet

### What Each State Renders

| Section | `isPending` | `isError = false, data.isValid()` | `isError = true` |
|---|---|---|---|
| Tone Guidance | Skeleton | Generated string | Placeholder UI |
| Pitch Points slot | Skeleton | Generated pitch points | Static Core Benefits (`ChecklistSection` fallback) |
| Example of Coverage | Hidden (always, MVP) | Hidden | Hidden |
| All other sections | Render normally | Render normally | Render normally |

### Layout Order (from design)

1. Tone Guidance (above all checklist sections)
2. Discovery question
3. Engagement question
4. Transition statement
5. Pitch Points slot (where Core Benefits currently lives)
6. Pricing transparency

### Visual Design Notes (from design screenshot)

- ✦ sparkle icon marks both the global "Generating..." label and the pitch points accordion row header — consistent AI-generated content indicator
- Tone Guidance loading: full-width dark empty box with purple/violet border
- Pitch Points loading: accordion row with ✦ sparkle + skeleton bar where section title would be; no "Done" badge; chevron present for expand/collapse

### Wiring (in `FullOfferChecklist`)

`FullOfferChecklist` now delegates entirely to `useDynamicSmartOffer`:

```tsx
const { data, isPending, isError, isEnabled, tryAgain } = useDynamicSmartOffer(sessionId, salesChecklist);

// Above the Accordion:
<DynamicToneGuidance isEnabled={isEnabled} isPending={isPending} isError={isError}
    toneGuidance={data.toneGuidance} onTryAgain={tryAgain} />

// Inside the Accordion map, for sectionType === "coreBenefits" && enableDynamicContent:
<DynamicCoreBenefits isEnabled={isEnabled} isPending={isPending} isError={isError}
    pitchPoints={data.pitchPoints} fallbackSlot={<ChecklistSectionWithErrorBoundary ... />} />
```

**Identifying sections by `sectionType`** — swap logic uses `section.sectionType === "coreBenefits" && section.enableDynamicContent` rather than matching on title. Both fields are optional on `SalesChecklistSection` for backwards compatibility.

---

## Backend Implementation Plan — Kafka → WebSocket Lambda

### Overview

A new Lambda in the `sales` service will act as a **Kafka Connect Lambda Sink**. It consumes Logic Builder events from Kafka and forwards them to the frontend via the existing `sendEventToFrontend` WebSocket infrastructure.

```
Logic Builder → Kafka (ExWo_VZ / ExWo_ATT topics)
                  ↓
          [new Lambda sink — sales service]
                  ↓
          sendEventToFrontend (EventBridge → websocket-service)
                  ↓
          Frontend WS → logicBuilderEventBus → useDynamicSmartOfferStore
```

---

### Repo & Ownership

- **Repo:** `expert-workspace-api`
- **Service:** `services/sales/`
- **Team:** sales (matches `readonly team = 'sales'` in `services/sales/cdk/stack.ts`)

---

### Files (Implemented)

#### `services/sales/lib/logic-builder-kafka-sink/handlers/handler.lambda.ts`

Lambda entrypoint. Invoked by Confluent's K8s Lambda Sink Connector as a **batch** (`KafkaConnectLambdaEvent = Array<KafkaConnectRecord>`). Takes only the **last record** in the batch (latest value wins), validates with Zod, calls `sendDynamicSmartOfferToFrontend`.

**Kafka input schema (Zod-validated):**
```typescript
// DynamicSmartOfferSinkEventSchema
{
    sessionId: string,      // frontend store key
    employeeId: string,     // routes WS message to the correct expert
    toneGuidance: string,
    pitchPoints: string[],
}
```

#### `services/sales/lib/logic-builder-kafka-sink/src/services/sendDynamicSmartOfferToFrontend.ts`

Calls `sendEventToFrontend` (EventBridge → websocket-service → expert's WS connection):

```typescript
const MESSAGE_TYPE = 'dynamic-smart-offer-event';

await sendEventToFrontend<DynamicSmartOfferPayload>({
    correlationId: sessionId,
    expertIdentity: employeeId,   // employeeId is the routing key
    messageType: MESSAGE_TYPE,
    payload: { sessionId, toneGuidance, pitchPoints },
});
```

**WS `messageType` is `'dynamic-smart-offer-event'`** (note the `-event` suffix — frontend listener must match this exactly).

#### `services/sales/cdk/constructs/lambdas/createLogicBuilderKafkaSink.ts`

Creates the Lambda with `createLambda()`. **Kafka is not wired via an EventSourceMapping** — the Confluent K8s Lambda Sink Connector invokes the Lambda directly. The CDK construct grants invoke permissions to the K8s node group IAM roles:

```
nonprod: arn:aws:iam::807093730725:role/eventing-nonprod-event-streaming-cl1-connect-ng-role
prod:    arn:aws:iam::031019796475:role/eventing-prod-event-streaming-cl1-connect-ng-role
```

Grants: `lambda:InvokeFunction`, `lambda:InvokeAsync`, `lambda:GetFunction`.

Also grants the Lambda `events:PutEvents` on the ExWo EventBridge bus (`exwoEventBusName` from config).

#### `services/sales/cdk/stack.ts`

`createLogicBuilderKafkaSink` instantiated and added to the stack. Already wired in.

---

### Frontend — New WS Message Handler

Frontend needs to listen for `messageType: 'dynamic-smart-offer-event'` and pipe it into `logicBuilderEventBus`:

```typescript
logicBuilderEventBus.emit('dynamic_smart_offer', {
    sessionId: payload.sessionId,
    toneGuidance: payload.toneGuidance,
    pitchPoints: payload.pitchPoints,
});
```

---

### Remaining (backend)

- [ ] **Kafka-Connect Lambda Sink Connector setup** — Event Streaming team is currently configuring the Confluent K8s connector to invoke our Lambda. [Slack thread](https://asurion-teams.slack.com/archives/C0166CTKQET/p1772822416750369?thread_ts=1772821725.447529&cid=C0166CTKQET)
- [ ] **Data shape validation** — Once the connector is set up, send test Kafka events and confirm the payload matches our Lambda's Zod schema.
- [ ] **E2E integration test** — Send test Kafka events end-to-end using the [Event Streaming Portal (nonprod)](https://event-streaming-portal-nonprod.edpapi.npr.aws.asurion.net/kafka-tester) and verify the full path: Kafka → Lambda → EventBridge → WebSocket → frontend store.

---

## Implementation Tracker

### Done

- [x] `DynamicToneGuidance` component — loading / error / success states, stories, tests
- [x] `DynamicCoreBenefits` component — loading / error / success / fallback states, stories, tests
- [x] Shared `Sparkle` component with `color` and `size` props
- [x] `@tabler/icons-react` added to `sales-tools` package
- [x] `SalesChecklistSection` type updated with `sectionType?` and `enableDynamicContent?`
- [x] Contentful CMS: `sectionType` and `enableDynamicContent` fields added to `checklistSection` content type
- [x] `@expert/sales-tools` package wired up with `main` entry and barrel `index.ts`
- [x] `@expert/sales-tools` added as dependency in `expert-ui`
- [x] `DynamicToneGuidance` wired into `FullOfferChecklist` above the `<Accordion>`
- [x] `DynamicCoreBenefits` wired into `FullOfferChecklist` swap logic (`sectionType === "coreBenefits" && enableDynamicContent`)
- [x] Module constant `DYNAMIC_SMART_OFFER_STATE` used as placeholder data pending real GAIA integration
- [x] `isDynamicSmartOfferEnabled` feature flag added to `WorkspaceFeatures` type and defaults (`packages/features`) — default `false`, no `FLAG_NAME_MAP` entry needed
- [x] `isDynamicSmartOfferEnabled` wired into `FullOfferChecklist` via `useFeatures<WorkspaceFeatures>()`, gates `isEnabled` on both `DynamicToneGuidance` and `DynamicCoreBenefits`
- [x] Feature flag JSON files created in all 6 environments in `expert-workspace-config` (`nonprod`, `prod`, `gcp-nonprod`, `gcp-prod`, `saas-nonprod`, `saas-prod`) — default `false`, no rules — draft PR #589
- [x] Eligibility API transform: `sectionType` and `enableDynamicContent` mapped in `buildChecklistSections.ts`; types updated in `shared-types` and `services/sales` — draft PR #3221 in `expert-workspace-api`

### Frontend — To Do

- [ ] Emit `offerGenerated: boolean` analytics event when first event is received
- [ ] Remove debug `console.log("MASONLOG", ...)` from `FullOfferChecklist.tsx`

### Frontend — Done

- [x] `DynamicSmartOffer` class — `toneGuidance`, `pitchPoints`, `isValid()`, `equals()`; with unit tests
- [x] `useDynamicSmartOfferStore` — per-session Zustand store (persist + immer + devtools), `getSession` / `receiveData` / `reset`; with unit tests
- [x] `useDynamicSmartOfferListener` — subscribes to `logicBuilderEventBus`, converts payload → `DynamicSmartOffer`, writes to store
- [x] `useDynamicSmartOffer` hook — wraps store + listener, manages `isPending` / `isAwaitingData` / `isEnabled` / `hasNewerData` / `showLatest`; subscribes to store reactively so `hasNewerData` updates immediately on new WS events; with unit tests
- [x] `logicBuilderEventBus` — `TinyEmitter` for Logic Builder WS events; `__logicBuilder.emitDynamicSmartOffer()` exposed on `globalThis` in non-prod for console testing
- [x] `FullOfferChecklist` — uses `useDynamicSmartOffer`, passes `data`/`isPending`/`isAwaitingData`/`hasNewerData`/`showLatest` to `DynamicToneGuidance` and `DynamicCoreBenefits`
- [x] Dev tool `EventEmitter` — draggable popover with GAIA tab + Logic Builder tab (merged from two separate tools)
- [x] `isError` replaced with `isAwaitingData` throughout — no error state; UI either has data (success) or is waiting (awaiting)
- [x] "Try again" button renamed to "Show latest"; only renders when `hasNewerData` is true (store has valid data differing from what's displayed)
- [x] `DynamicToneGuidance` awaiting state — neutral `dark.6` bg, Sparkle icon, "Personalized guidance pending"; copy: "Guidance will be available soon." / "Guidance is available. Click 'Show latest' to view." when `hasNewerData`
- [x] `handleDismiss` — dismisses tone guidance in both awaiting and success states

### Backend / API

- [x] Eligibility API transform: `sectionType` and `enableDynamicContent` mapped in `buildChecklistSections.ts`; types updated in `packages/shared-types/contentful-product-info.ts` and `services/sales/lib/shared/utils/contentful/types.ts` — branch `RDESQ-1900-dynamic-smart-offer` in `expert-workspace-api`
- [x] `services/sales/lib/logic-builder-kafka-sink/handlers/handler.lambda.ts` — Lambda handler (batch-aware, latest-record-wins, Zod validation)
- [x] `services/sales/lib/logic-builder-kafka-sink/handlers/types.ts` — Zod schemas: `DynamicSmartOfferSinkEventSchema`, `DynamicSmartOfferPayloadSchema`, `KafkaConnectLambdaEvent`
- [x] `services/sales/lib/logic-builder-kafka-sink/src/services/sendDynamicSmartOfferToFrontend.ts` — sends to frontend via EventBridge (`messageType: 'dynamic-smart-offer-event'`, routed by `employeeId`)
- [x] `services/sales/cdk/constructs/lambdas/createLogicBuilderKafkaSink.ts` — CDK construct; grants invoke permissions to K8s node group roles (Confluent sink connector); grants EventBridge `PutEvents`
- [x] `services/sales/cdk/stack.ts` — Lambda wired into sales stack

---

## Open Questions (Tracker)

- [ ] What is the minimum transcript data threshold for DS generation? When in a typical call is it reached?
- [ ] GAIA event delivery resilience strategy for checklist detection events (missed events, re-send, scheduled re-send)?
  - Can all checked items be sent on every event so a missed one is corrected on the next?
  - Can there be a scheduled re-send even when no new item is checked?
- [ ] Who is responsible for compiling success metrics downstream?
- [ ] Single, public communication forum for this effort (all stakeholders)?
- [ ] What is needed from C360 team re: historical transcript availability timing (post-MVP)?

### Resolved

- ✅ **Backend service:** Logic Builder (not GAIA) drives content generation for MVP.
- ✅ **Pitch point pool ownership:** Logic Builder owns the pool and sends rendered content to the UI. ExWo just renders what it receives. No Contentful entries for pitch points.
- ✅ **Pitch points are not checked off live:** No detection models for the pitch points section. No live check-off behavior.
- ✅ **Generated offer does not mutate during a call:** Content is "dynamic" at fetch time only. No mid-call updates unless the expert explicitly clicks "Try again" / Refresh.
- ✅ **Success metrics — ExWo instrumentation:** ExWo needs to emit an `offerGenerated: boolean` event so downstream can track whether the dynamic offer was shown vs. the static fallback.
- ✅ **Chat is out of scope:** This is a Voice feature. Chat will handle its own implementation separately.
- ✅ **Kafka topic config:** Nonprod topic names confirmed (`paas.simplr.smart-offer-verizon-exwo`, `paas.simplr.smart-offer-att-exwo`). Connector setup delegated to Event Streaming team.
- ✅ **WS message type registration:** `messageType: 'dynamic-smart-offer-event'` is dispatched generically — no explicit registration needed on the frontend dispatcher.
