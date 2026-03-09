# Dynamic Smart Offer — UI Specs

Delta specifications for `expert-workspace` UI changes.
Format: [OpenSpec](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85)

---

## ADDED Requirements

### Tone Guidance section

#### Scenario: Expert enters offer flow — DS data already in store

WHEN the expert clicks into the offer flow
AND `useDynamicSmartOffer` completes its 2s `isPending` delay
AND `data.isValid()` returns true
THEN `DynamicToneGuidance` renders the generated `toneGuidance` string above the Accordion

#### Scenario: Expert enters offer flow — no DS data yet

WHEN the expert clicks into the offer flow
AND `useDynamicSmartOffer` completes its 2s `isPending` delay
AND `data.isValid()` returns false
THEN `DynamicToneGuidance` renders the awaiting state: neutral `dark.6` background, Sparkle icon, "Personalized guidance pending" copy

#### Scenario: Expert enters offer flow — isPending delay active

WHEN the 2s `isPending` delay is in progress
THEN `DynamicToneGuidance` renders a skeleton

#### Scenario: Newer DS data arrives while expert is viewing offer

WHEN a new `"dynamic_smart_offer"` WS event is received
AND the new data differs from what is currently displayed
THEN `hasNewerData` becomes true
AND `DynamicToneGuidance` renders a "Show latest" button alongside the current content

#### Scenario: Expert clicks "Show latest"

WHEN the expert clicks "Show latest"
THEN `isPending` is set to true for 2s
AND after the delay, the latest data from the store is displayed

#### Scenario: Expert dismisses Tone Guidance

WHEN the expert clicks the dismiss control on `DynamicToneGuidance`
THEN the Tone Guidance section is hidden for the remainder of the offer flow session

### Dynamic Core Benefits (Pitch Points)

#### Scenario: isEnabled and data is valid

WHEN `isEnabled` is true
AND `data.isValid()` returns true
AND the offer flow is not in `isPending`
THEN `DynamicCoreBenefits` renders the generated pitch points in place of the static Core Benefits section

#### Scenario: isEnabled and isAwaitingData

WHEN `isEnabled` is true
AND `data.isValid()` returns false
AND the offer flow is not in `isPending`
THEN `DynamicCoreBenefits` renders the static Core Benefits `ChecklistSection` as the fallback
AND the static section and the personalized section are never shown simultaneously

#### Scenario: isPending active

WHEN the 2s `isPending` delay is in progress
THEN `DynamicCoreBenefits` renders a skeleton with a Sparkle icon accordion header

#### Scenario: isEnabled is false

WHEN `isEnabled` is false (feature flag off or `enableDynamicContent` not set on the checklist section)
THEN the standard `ChecklistSection` renders for Core Benefits (no dynamic behavior)

### "Show latest" button (formerly "Try again")

#### Scenario: No newer data

WHEN `hasNewerData` is false
THEN no "Show latest" button is rendered

#### Scenario: Newer data available

WHEN `hasNewerData` is true
THEN "Show latest" is rendered
AND clicking it triggers a 2s `isPending` delay followed by displaying the latest store data

### Sparkle icon

#### Scenario: AI-generated content indicator

WHEN content is AI-generated (Tone Guidance or Pitch Points accordion header)
THEN a ✦ Sparkle icon appears as a consistent visual indicator of AI-generated content

---

## MODIFIED Requirements

### FullOfferChecklist wiring

#### Scenario: Section rendering delegation

WHEN `FullOfferChecklist` renders
THEN it consumes `useDynamicSmartOffer(sessionId, salesChecklist)`
AND passes `data`, `isPending`, `isAwaitingData`, `hasNewerData`, `showLatest` to both `DynamicToneGuidance` and `DynamicCoreBenefits`
AND identifies the Core Benefits slot by `section.sectionType === "coreBenefits" && section.enableDynamicContent`

---

## REMOVED Requirements

### "Example of Coverage" / "Personalize the Coverage" section

#### Scenario: Expert is anywhere in the offer flow

WHEN the expert is in the offer flow (any state: pending, awaiting, success)
THEN the "Example of Coverage" / "Personalize the Coverage" (`CoverageCarousel`) section is not rendered
AND this holds for all MVP states — it is hidden unconditionally in MVP
