# API Specification — Expert Workspace API

Source of truth for `expert-workspace-api` behavior.
Format: [OpenSpec](https://github.com/Fission-AI/OpenSpec)

This spec reflects the system as of the Dynamic Smart Offer (RDESQ-1900) implementation.

---

## Purpose

The Expert Workspace API provides eligibility evaluation and real-time data pipeline for the expert-facing sales tools. It processes Kafka events from Logic Builder and routes generated content to the frontend via WebSocket.

---

## Requirements

### Requirement: Eligibility API — checklist section fields

The eligibility API (`POST /v2/eligibility`) MUST return `sectionType` and `enableDynamicContent` on each checklist section when set in Contentful.

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

---

### Requirement: Logic Builder Kafka Lambda sink

The `EWP-Sales-logic-builder-kafka-sink` Lambda MUST consume events from the Kafka topic, validate them, and forward the latest event to the frontend via EventBridge.

#### Scenario: Batch received — latest value wins

- GIVEN the Confluent K8s Lambda Sink Connector invokes the Lambda with a `KafkaConnectLambdaEvent` (array of records)
- WHEN the Lambda handler executes
- THEN only the last record in the batch is processed (latest-value-wins)
- AND the payload is validated against `DynamicSmartOfferSinkEventSchema` (Zod)

#### Scenario: Valid payload — forwarded to frontend

- GIVEN the Kafka record payload passes Zod schema validation
- WHEN the Lambda processes the record
- THEN `sendDynamicSmartOfferToFrontend` is called with the validated payload
- AND the event is forwarded to EventBridge with `messageType: 'dynamic-smart-offer-event'`
- AND routing uses `employeeId` as the expert identity key
- AND the payload contains `{ sessionId, toneGuidance, pitchPoints }`

#### Scenario: Payload validation failure

- GIVEN the Kafka record payload does not conform to the Zod schema
- WHEN the Lambda processes the record
- THEN the Lambda throws a validation error
- AND the event is not forwarded to EventBridge or the frontend

---

### Requirement: WebSocket event routing

The WebSocket service MUST route `dynamic-smart-offer-event` messages to the correct expert's active WebSocket connection.

#### Scenario: Event dispatched to connected expert

- GIVEN an expert is connected via WebSocket
- AND a `dynamic-smart-offer-event` message is published to EventBridge with their `employeeId`
- WHEN the websocket-service processes the EventBridge event
- THEN the message is delivered to the expert's active WebSocket connection
- AND the frontend dispatches the event to `logicBuilderEventBus`
