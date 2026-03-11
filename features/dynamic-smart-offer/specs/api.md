# Dynamic Smart Offer — API / Backend Specs

Delta specifications for `expert-workspace-api` and `expert-workspace-config` changes.
Format: [OpenSpec](https://github.com/Fission-AI/OpenSpec)

---

## ADDED Requirements

### Requirement: Logic Builder Kafka Lambda sink
The `EWP-Sales-logic-builder-kafka-sink` Lambda MUST consume Kafka events from Logic Builder, validate them, and forward the latest event to the frontend.

#### Scenario: Batch received — latest value wins

- GIVEN the Confluent K8s Lambda Sink Connector invokes the Lambda with a `KafkaConnectLambdaEvent` (array of records)
- WHEN the Lambda handler executes
- THEN only the last record in the batch is processed (latest-value-wins)
- AND the payload is validated against `DynamicSmartOfferSinkEventSchema` (Zod)
- AND `sendDynamicSmartOfferToFrontend` is called with the validated payload

#### Scenario: Payload validation failure

- GIVEN the Kafka record payload does not conform to the Zod schema
- WHEN the Lambda processes the record
- THEN the Lambda throws a validation error
- AND the event is not forwarded to EventBridge or the frontend

#### Scenario: Valid payload forwarded to frontend

- GIVEN the payload passes Zod schema validation
- WHEN `sendDynamicSmartOfferToFrontend` is called
- THEN the event is published to EventBridge with `messageType: 'dynamic-smart-offer-event'`
- AND `employeeId` is used as the routing key (expert identity)
- AND the payload contains `{ sessionId, toneGuidance, pitchPoints }`
- AND the message is routed via EventBridge → websocket-service → expert's active WS connection

---

### Requirement: `isDynamicSmartOfferEnabled` feature flag
The `isDynamicSmartOfferEnabled` feature flag MUST exist in all environments, default to `false`, and gate all dynamic offer behavior.

#### Scenario: Flag is off

- GIVEN `isDynamicSmartOfferEnabled` is `false`
- WHEN the frontend renders the offer flow
- THEN `FullOfferChecklist` renders with no dynamic behavior (standard checklist only)
- AND no `useDynamicSmartOfferListener` is mounted

#### Scenario: Flag is on

- GIVEN `isDynamicSmartOfferEnabled` is `true`
- WHEN the frontend renders the offer flow
- THEN `isEnabled` gates activate
- AND dynamic components receive live Logic Builder data

---

## MODIFIED Requirements

### Requirement: Eligibility API — checklist section transform
The eligibility API MUST map `sectionType` and `enableDynamicContent` from Contentful onto each returned `SalesChecklistSection`.

#### Scenario: Contentful section has dynamic fields set

- GIVEN a `checklistSection` in Contentful has `sectionType` and `enableDynamicContent` configured
- WHEN `POST /v2/eligibility` is called
- THEN `buildChecklistSections.ts` maps both fields onto the returned `SalesChecklistSection` objects
- AND the frontend receives `sectionType` and `enableDynamicContent` on each applicable section

#### Scenario: Legacy section without dynamic fields

- GIVEN a `checklistSection` in Contentful does not have `sectionType` or `enableDynamicContent` set
- WHEN `POST /v2/eligibility` is called
- THEN both fields are `undefined` on the returned `SalesChecklistSection`
- AND the frontend treats the section as a standard non-dynamic section
