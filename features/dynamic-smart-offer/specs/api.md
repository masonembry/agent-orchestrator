# Dynamic Smart Offer — API / Backend Specs

Delta specifications for `expert-workspace-api` and `expert-workspace-config` changes.
Format: [OpenSpec](https://dev.to/webdeveloperhyper/how-to-make-ai-follow-your-instructions-more-for-free-openspec-2c85)

---

## ADDED Requirements

### Kafka Lambda sink

#### Scenario: Logic Builder publishes a batch of events

WHEN the Confluent K8s Lambda Sink Connector invokes `handler.lambda.ts` with a `KafkaConnectLambdaEvent` (array of records)
THEN the Lambda takes only the last record in the batch (latest-value-wins)
AND validates the payload against `DynamicSmartOfferSinkEventSchema` (Zod)
AND calls `sendDynamicSmartOfferToFrontend` with the validated payload

#### Scenario: Payload validation fails

WHEN the Kafka record payload does not match the Zod schema
THEN the Lambda throws a validation error and does not forward to EventBridge

#### Scenario: Valid payload forwarded to frontend

WHEN payload is valid
THEN `sendEventToFrontend` is called with:
  - `messageType: 'dynamic-smart-offer-event'`
  - `expertIdentity: employeeId` (routing key)
  - `payload: { sessionId, toneGuidance, pitchPoints }`
AND the message is routed via EventBridge → websocket-service → expert's WS connection

### Feature flag

#### Scenario: Flag is off (default)

WHEN `isDynamicSmartOfferEnabled` is `false`
THEN `FullOfferChecklist` renders no dynamic behavior (standard checklist only)
AND no `useDynamicSmartOfferListener` is mounted

#### Scenario: Flag is on

WHEN `isDynamicSmartOfferEnabled` is `true`
THEN `isEnabled` gates activate and dynamic components receive live data

---

## MODIFIED Requirements

### Eligibility API — checklist section transform

#### Scenario: API returns checklist sections

WHEN `POST /v2/eligibility` is called
AND Contentful has `sectionType` and `enableDynamicContent` set on a `checklistSection`
THEN `buildChecklistSections.ts` maps both fields onto the returned `SalesChecklistSection` objects
AND the frontend receives `sectionType` and `enableDynamicContent` on each section

#### Scenario: Legacy checklist section without new fields

WHEN a `checklistSection` in Contentful does not have `sectionType` or `enableDynamicContent` set
THEN both fields are `undefined` on the returned `SalesChecklistSection`
AND the frontend treats the section as a standard (non-dynamic) section
