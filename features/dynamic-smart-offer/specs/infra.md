# Dynamic Smart Offer — Infrastructure Specs

Delta specifications for `expert-workspace-config`, `eds-platform-connectors`, and `enterprise-eventing` changes.
Format: [OpenSpec](https://github.com/Fission-AI/OpenSpec)

---

## ADDED Requirements

### Requirement: Logic Builder Kafka Sink CDK construct
The `createLogicBuilderKafkaSink` CDK construct MUST provision the Lambda with the correct IAM grants for K8s connector invocation and EventBridge publishing.

#### Scenario: K8s node group invoke permissions

- GIVEN the CDK construct is deployed to an environment
- WHEN Confluent K8s Lambda Sink Connectors attempt to invoke the Lambda
- THEN the K8s node group IAM roles have `lambda:InvokeFunction`, `lambda:InvokeAsync`, and `lambda:GetFunction` grants on the Lambda

#### Scenario: EventBridge publish permissions

- GIVEN the Lambda is invoked with a valid Kafka payload
- WHEN it calls `sendDynamicSmartOfferToFrontend`
- THEN the Lambda's execution role has `events:PutEvents` on the Expert Workspace EventBridge bus

#### Scenario: Lambda wired into sales stack

- GIVEN the `services/sales` CDK stack deploys
- WHEN `stack.ts` is applied
- THEN the `EWP-Sales-logic-builder-kafka-sink` Lambda is provisioned via `createLogicBuilderKafkaSink`

---

### Requirement: `isDynamicSmartOfferEnabled` feature flag JSON
The `isDynamicSmartOfferEnabled` feature flag MUST exist in all six environments with a default value of `false`.

#### Scenario: Flag exists in all environments

- GIVEN the feature flag JSON files are deployed
- WHEN any environment (nonprod, prod, gcp-nonprod, gcp-prod, saas-nonprod, saas-prod) evaluates the flag
- THEN `isDynamicSmartOfferEnabled` resolves to `false` by default (no override rules)

#### Scenario: Flag enabled in Nonprod for testing

- GIVEN `isDynamicSmartOfferEnabled` is set to `true` in the Nonprod environment config
- WHEN an expert loads the offer flow in Nonprod
- THEN dynamic smart offer behavior is active

---

### Requirement: Kafka connector configuration (Nonprod)
Nonprod Confluent K8s Lambda Sink Connectors MUST be configured and pointing to the correct Lambda ARN and AWS account.

#### Scenario: Nonprod connector configs merged

- GIVEN `ewp-sales-dynamic-smart-offer-att.json` and `ewp-sales-dynamic-smart-offer-verizon.json` are merged in `eds-platform-connectors`
- AND the Nonprod Lambda is deployed
- WHEN Logic Builder publishes to `paas.simplr.smart-offer-verizon-exwo` or `paas.simplr.smart-offer-att-exwo`
- THEN the connector invokes the Nonprod Lambda (`EWP-Sales-logic-builder-kafka-sink`) in account `322429462214`

---

### Requirement: Kafka connector configuration (Prod)
Prod Confluent K8s Lambda Sink Connectors MUST NOT be activated until nonprod end-to-end testing passes.

#### Scenario: Prod connectors gated on nonprod validation

- GIVEN nonprod end-to-end testing has not yet passed
- WHEN the prod connector PR (#770) is reviewed
- THEN it MUST remain as a draft and MUST NOT be merged

#### Scenario: Prod connectors activated after nonprod passes

- GIVEN nonprod testing passes
- AND the `enterprise-eventing` prod invoke grant PR (#9107) is merged and deployed
- WHEN the prod connector PR (#770) is merged
- THEN prod connectors activate on `paas.simplr.smart-offer-verizon-exwo` and `paas.simplr.smart-offer-att-exwo`
- AND the Lambda ARN is `arn:aws:lambda:us-east-1:737463266258:function:EWP-Sales-logic-builder-kafka-sink`

---

### Requirement: Kafka Connect Lambda invoke permissions (enterprise-eventing)
Kafka Connect MUST have IAM permission to invoke the Logic Builder Kafka Sink Lambda in both Nonprod and Prod.

#### Scenario: Nonprod invoke grant

- GIVEN the enterprise-eventing Nonprod CDK config includes the Lambda ARN pattern `EWP-Sales-*logic-builder-kafka-sink`
- WHEN the Kafka connector invokes the Lambda in Nonprod
- THEN the invocation is permitted by IAM

#### Scenario: Prod invoke grant

- GIVEN the enterprise-eventing Prod CDK config includes the exact function name `EWP-Sales-logic-builder-kafka-sink` (no wildcard)
- WHEN the Kafka connector invokes the Lambda in Prod
- THEN the invocation is permitted by IAM

---

## MODIFIED Requirements

### Requirement: Contentful `checklistSection` content type
The `checklistSection` content type in Contentful MUST include `sectionType` and `enableDynamicContent` fields.

#### Scenario: Fields available for content configuration

- GIVEN the updated content type is published in Contentful
- WHEN a content editor configures the Core Benefits section
- THEN `sectionType` can be set to `"coreBenefits"` and `enableDynamicContent` can be set to `true`
- AND the eligibility API includes both field values in its response
