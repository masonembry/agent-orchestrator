# Infrastructure Specification — Expert Workspace Platform

Source of truth for `expert-workspace-config`, `eds-platform-connectors`, and `enterprise-eventing` infrastructure.
Format: [OpenSpec](https://github.com/Fission-AI/OpenSpec)

This spec reflects the system as of the Dynamic Smart Offer (RDESQ-1900) implementation.

---

## Purpose

Infrastructure specs cover CDK constructs, feature flag configuration, Kafka connector configuration, and IAM grants that support the Expert Workspace platform.

---

## Requirements

### Requirement: Dynamic Smart Offer feature flag

The `isDynamicSmartOfferEnabled` feature flag MUST exist in all six environments and default to `false`.

#### Scenario: Default state across all environments

- GIVEN the `isDynamicSmartOfferEnabled` flag JSON exists in all environments (nonprod, prod, gcp-nonprod, gcp-prod, saas-nonprod, saas-prod)
- WHEN no override rules are configured
- THEN the flag evaluates to `false` for all experts

#### Scenario: Enabled in Nonprod

- GIVEN `isDynamicSmartOfferEnabled` is set to `true` in the Nonprod environment config
- WHEN an expert loads the UI in Nonprod
- THEN dynamic smart offer behavior is active

---

### Requirement: Logic Builder Kafka Sink CDK construct

The `EWP-Sales-logic-builder-kafka-sink` Lambda MUST be provisioned via CDK with the correct IAM grants for Kafka connector invocation and EventBridge publishing.

#### Scenario: K8s node group invoke permissions

- GIVEN the CDK construct `createLogicBuilderKafkaSink` is deployed
- WHEN Confluent K8s Lambda Sink Connectors attempt to invoke the Lambda
- THEN the K8s node group IAM roles have `lambda:InvokeFunction`, `lambda:InvokeAsync`, and `lambda:GetFunction` grants on the Lambda

#### Scenario: EventBridge publish permissions

- GIVEN the Lambda is invoked with a valid payload
- WHEN it calls `sendDynamicSmartOfferToFrontend`
- THEN the Lambda's execution role has `events:PutEvents` on the Expert Workspace EventBridge bus

---

### Requirement: Kafka connector configuration (eds-platform-connectors)

Confluent K8s Lambda Sink Connectors MUST be configured for each carrier topic and point to the correct Lambda ARN and AWS account.

#### Scenario: Nonprod connectors active

- GIVEN the Nonprod Lambda (`EWP-Sales-logic-builder-kafka-sink`) is deployed
- AND the Nonprod connector configs are merged (`ewp-sales-dynamic-smart-offer-att.json`, `ewp-sales-dynamic-smart-offer-verizon.json`)
- WHEN a Logic Builder event is published to `paas.simplr.smart-offer-verizon-exwo` or `paas.simplr.smart-offer-att-exwo`
- THEN the connector invokes the Nonprod Lambda with the event payload

#### Scenario: Prod connectors — gated on nonprod validation

- GIVEN nonprod end-to-end testing has passed
- WHEN the prod connector PR (#770) is merged
- THEN prod connectors activate on `paas.simplr.smart-offer-verizon-exwo` and `paas.simplr.smart-offer-att-exwo`
- AND the Lambda ARN is `arn:aws:lambda:us-east-1:737463266258:function:EWP-Sales-logic-builder-kafka-sink`

---

### Requirement: Kafka Connect Lambda invoke permissions (enterprise-eventing)

Kafka Connect MUST have IAM permission to invoke the Logic Builder Kafka Sink Lambda in both Nonprod and Prod.

#### Scenario: Nonprod invoke grant

- GIVEN the enterprise-eventing Nonprod CDK config includes the Lambda ARN pattern `EWP-Sales-*logic-builder-kafka-sink`
- WHEN the Kafka connector attempts to invoke the Lambda in Nonprod
- THEN the invocation is permitted

#### Scenario: Prod invoke grant

- GIVEN the enterprise-eventing Prod CDK config includes the exact Lambda function name `EWP-Sales-logic-builder-kafka-sink`
- WHEN the Kafka connector attempts to invoke the Lambda in Prod
- THEN the invocation is permitted

---

### Requirement: Contentful content model

The Contentful `checklistSection` content type MUST include `sectionType` and `enableDynamicContent` fields.

#### Scenario: Fields available for configuration

- GIVEN the `checklistSection` content type in Contentful has `sectionType` (short text) and `enableDynamicContent` (boolean) fields
- WHEN a content editor configures the Core Benefits section
- THEN `sectionType` can be set to `"coreBenefits"` and `enableDynamicContent` can be set to `true`
- AND the eligibility API includes these values in the response
