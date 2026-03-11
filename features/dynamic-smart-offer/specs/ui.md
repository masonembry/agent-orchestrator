# Dynamic Smart Offer — UI Specs

Delta specifications for `expert-workspace` UI changes.
Format: [OpenSpec](https://github.com/Fission-AI/OpenSpec)

---

## ADDED Requirements

### Requirement: Tone Guidance section
`DynamicToneGuidance` MUST render above the offer accordion when the feature is enabled, in one of three states: loading skeleton, awaiting data, or loaded AI-generated content.

#### Scenario: Loaded state — DS data already in store

- GIVEN the expert opens the offer flow
- AND `useDynamicSmartOffer` completes its 2s `isPending` delay
- AND `data.isValid()` returns `true`
- WHEN the offer flow renders
- THEN `DynamicToneGuidance` renders the AI-generated `toneGuidance` string above the Accordion

#### Scenario: Awaiting state — no DS data yet

- GIVEN the expert opens the offer flow
- AND `useDynamicSmartOffer` completes its 2s `isPending` delay
- AND `data.isValid()` returns `false`
- WHEN the offer flow renders
- THEN `DynamicToneGuidance` renders the awaiting state: animated Lottie spark icon, bold "Listening for customer cues" title, and body copy in a neutral rounded card

#### Scenario: Loading skeleton

- GIVEN the expert opens the offer flow
- WHEN the 2s `isPending` delay is in progress
- THEN `DynamicToneGuidance` renders a skeleton placeholder

#### Scenario: Auto-refresh when new data arrives during awaiting state

- GIVEN the expert is in the awaiting state (no valid data displayed)
- WHEN a new `dynamic_smart_offer` WebSocket event is received with valid data
- THEN `useDynamicSmartOffer` automatically triggers the 2s `isPending` delay
- AND after the delay, the latest store data is displayed with no user action required

#### Scenario: "Show latest" offered when new data arrives during loaded state

- GIVEN valid data is already displayed
- WHEN a new `dynamic_smart_offer` WebSocket event is received
- THEN `hasNewerData` becomes `true`
- AND `DynamicToneGuidance` renders a "Show latest" button alongside the current content

#### Scenario: Expert clicks "Show latest"

- GIVEN `hasNewerData` is `true` and "Show latest" is visible
- WHEN the expert clicks "Show latest"
- THEN a 2s `isPending` delay begins
- AND after the delay, the latest store data is displayed

---

### Requirement: Dynamic Core Benefits (Personalized Pitch Point + Static Core Benefits)
The AI-generated pitch point(s) MUST render as the first item(s) **inside** the Core Benefits accordion section — not as a separate accordion section. The static Core Benefits always render after the pitch point (they are required disclosures). When no pitch point is available, only the static Core Benefits render.

#### Scenario: Dynamic data available

- GIVEN `isEnabled` is `true`
- AND `data.isValid()` returns `true`
- AND the offer flow is not in `isPending`
- WHEN the offer flow renders
- THEN `DynamicCoreBenefits` renders the AI-generated pitch point(s) (with Sparkle icon, dark pill styling) as the first item(s) inside the Core Benefits accordion panel
- AND the static Core Benefits items follow immediately after

#### Scenario: Awaiting data

- GIVEN `isEnabled` is `true`
- AND `data.isValid()` returns `false`
- AND the offer flow is not in `isPending`
- WHEN the offer flow renders
- THEN no personalized pitch point slot is shown; only the static Core Benefits render inside the accordion

#### Scenario: Loading skeleton

- GIVEN the 2s `isPending` delay is in progress
- WHEN the offer flow renders
- THEN a skeleton row (Sparkle icon + skeleton bar) renders as the first item inside the Core Benefits accordion panel, followed by the static Core Benefits

#### Scenario: Feature disabled

- GIVEN `isEnabled` is `false`
- WHEN the offer flow renders
- THEN the standard `ChecklistSection` renders for Core Benefits with no dynamic behavior

---

### Requirement: "Show latest" button
The "Show latest" button MUST only be visible when newer Logic Builder data is available in the store.

#### Scenario: No newer data

- GIVEN `hasNewerData` is `false`
- WHEN the expert views the offer flow
- THEN no "Show latest" button is rendered

#### Scenario: Newer data available

- GIVEN `hasNewerData` is `true`
- WHEN the expert views the offer flow
- THEN "Show latest" is rendered
- AND clicking it triggers a 2s `isPending` delay followed by displaying the latest store data

---

### Requirement: AI content indicator
A Sparkle (✦) icon MUST appear as a consistent visual indicator on all AI-generated content in the offer flow.

#### Scenario: AI-generated content rendered

- GIVEN AI-generated content is displayed (Tone Guidance or Pitch Points accordion header)
- WHEN the expert views the offer flow
- THEN a Sparkle icon appears as a consistent visual indicator of AI-generated content

---

## MODIFIED Requirements

### Requirement: FullOfferChecklist wiring
`FullOfferChecklist` MUST consume `useDynamicSmartOffer` and delegate rendering to `DynamicToneGuidance` and `DynamicCoreBenefits`.

#### Scenario: Section rendering delegation

- GIVEN `FullOfferChecklist` is rendered with a valid session and sales checklist
- WHEN the component renders
- THEN it consumes `useDynamicSmartOffer(sessionId, salesChecklist)`
- AND passes `data`, `isPending`, `isAwaitingData`, `hasNewerData`, `showLatest` to both `DynamicToneGuidance` and `DynamicCoreBenefits`
- AND identifies the Core Benefits slot by `section.sectionType === "coreBenefits" && section.enableDynamicContent`

---

## REMOVED Requirements

### Requirement: Dismiss button on Tone Guidance
The dismiss (close) button on Tone Guidance MUST NOT be rendered — the section is always visible while the feature is enabled.

#### Scenario: Expert is viewing Tone Guidance

- GIVEN the expert is viewing any Tone Guidance state (awaiting or loaded)
- WHEN the offer flow renders
- THEN there is no close/dismiss button on `DynamicToneGuidance`

---

### Requirement: "Example of Coverage" / "Personalize the Coverage" section
The "Personalize the Coverage" checklist item (key: `COVERAGE`) MUST NOT render when the Dynamic Smart Offer feature is enabled. The AI pitch point replaces this manual step. This is implemented via `hiddenItemKeys={["COVERAGE"]}` passed to `ChecklistSection` for any coreBenefits section with `enableDynamicContent`.

#### Scenario: Feature enabled — any offer flow state

- GIVEN `isEnabled` is `true`
- WHEN the expert is in the offer flow (any state: pending, awaiting, or loaded)
- THEN the `COVERAGE` checklist item is not rendered in the Core Benefits section
