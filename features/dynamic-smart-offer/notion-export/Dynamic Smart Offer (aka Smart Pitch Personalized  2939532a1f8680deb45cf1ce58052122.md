# Dynamic Smart Offer (aka Smart Pitch/Personalized Offer)

Development Phase: Discovery & Design
Target Launch: 03/31/2026
Jira: https://solutonashville.atlassian.net/browse/RDESQ-1900
🏘️ Product Teams: Expert Product: Expert Workspace | Sell (https://www.notion.so/Expert-Product-Expert-Workspace-Sell-d3e4904706484adeb50e60243871adcd?pvs=21)
Estimated Timeline: 01/18/2026 → 03/31/2026
Sub-item: Discovery (https://www.notion.so/Discovery-2ea9532a1f8680bbae8edafe76cd5b1f?pvs=21), Data Science (https://www.notion.so/Data-Science-3049532a1f86801e8d27ddb1a53b7f28?pvs=21), Design (https://www.notion.so/Design-3049532a1f8680eba9b7c9a5a5ebc1cf?pvs=21)
Dev Weeks: 11
Dev Weeks per Engineer: 11
Client: All Clients
Include in Stakeholder Review: Yes
Status: Not started
Description: Evolve “Smart Offer” into a context-aware system that dynamically generates personalized offer guidance based on who the customer is and what we already know from their prior interactions.
Purpose: Sales

<aside>
<img src="https://www.notion.so/icons/clipping_gray.svg" alt="https://www.notion.so/icons/clipping_gray.svg" width="40px" />

**Project:** 

Team: Expert Workspace Sales

Author: Eliza Wall & Alicia James

Date: 1/16/26

</aside>

<aside>
<img src="https://www.notion.so/icons/link_gray.svg" alt="https://www.notion.so/icons/link_gray.svg" width="40px" />

**Links** 

[PRD: Dynamic Smart Offer](https://www.notion.so/PRD-Dynamic-Smart-Offer-3129532a1f8680ad8088e7390f4a8894?pvs=21) 

[Research: Generative Offer Support in Field & Alpha ](Dynamic%20Smart%20Offer%20(aka%20Smart%20Pitch%20Personalized%20/Research%20Generative%20Offer%20Support%20in%20Field%20&%20Alpha%202ea9532a1f8680aba0f1c3cf7d0f61b4.md)

[MVP Scope](Dynamic%20Smart%20Offer%20(aka%20Smart%20Pitch%20Personalized%20/MVP%20Scope%203059532a1f8680ffb925dc1dc98bc7e9.md)

[Dynamic Smart Offer - Engineering](Dynamic%20Smart%20Offer%20(aka%20Smart%20Pitch%20Personalized%20/Dynamic%20Smart%20Offer%20-%20Engineering%203179532a1f8680e59670d8bcfbec1d5a.md)

</aside>

### **Summary**

Evolve current “Smart Offer” into a context-aware system that dynamically generates personalized offer guidance based on who the customer is and what we already know from their prior interactions.

---

### **Problem Statement [WHY]**

The current “Smart Offer” experience does not leverage existing customer context from previous interactions. As a result:

- Messaging is often generic or misaligned with customer needs
- Known objections are not proactively addressed
- Tone can feel transactional or tone-deaf rather than responsive

We are underutilizing valuable customer signals that could materially improve how offers are positioned and received.

---

### **Hypothesis & Impact**

Context-aware guidance that leverages prior customer interactions (tone + benefit prioritization) will increase sales conversion and create stickier sales.

Context-aware offer guidance that incorporates prior customer interactions, specifically tone and benefit prioritization, will increase conversion rates and drive stickier sales.

**Field Results:**

- “Smart Pitch” ([more info here](Dynamic%20Smart%20Offer%20%28aka%20Smart%20Pitch%20Personalized%20/Research%20Generative%20Offer%20Support%20in%20Field%20%26%20Alpha%202ea9532a1f8680aba0f1c3cf7d0f61b4.md) demonstrated a meaningful lift (DiD +1.6 vs. +0.06), indicating that guided, context-driven selling improves outcomes.

---

### **Product Description [WHAT]**

Introduce a system that generates structured offer guidance by activating existing customer signals.

**Inputs:** Previous call history, customer attributes (collected historically or on current call), claims information 

**Outputs:** Context-aware guidance across tone, customer lifecycle, benefits, and objections

### Inputs

### 1. **Previous Call History**

- Call transcripts or summaries
- Documented decline reasons
- Language and sentiment signals indicating emotional state or preferences

### 2. **Customer Attributes (Collected historically or on current call)**

- Household context (e.g., presence of children)
- Device mix (phone, tablet, wearable, etc.)
- Number of devices
- Benefits previously presented, accepted, or declined

### **3. Claims Information**

- Device type
- Peril type
- Claim status (e.g., hold, backorder, unresolved, resolved)

### Outputs

### 1. **Tone Guidance**

Guidance on how the offer should be delivered, informed by prior customer sentiment and contextual signals.

- *Example:* “Empathize with busy family life. Focus on simplicity and future-proofing their growing tech needs.” “Customer appears rushed — keep offer concise and direct.”

### 2. Customer Lifecycle Guidance

Guidance that adapts offer framing based on where the customer is in their broader interaction and claims journey.

- Recent Interaction Awareness: Adjust offer framing to acknowledge recent callbacks and avoid repeating information already covered.
    - *Example:* Treat the offer as a follow-up rather than an initial pitch.
- Claims Awareness: Adjust offer presentation based on the customer’s current position in the claims process (e.g., claim holds, backorders, or resolution status).
    - *Example:* Acknowledge that while the claim could not be finalized today due to a specific constraint, the path forward is clear. Position the offer as future-proofing.

### 3. **Recommended Plan Benefits**

Personalized prioritization of plan benefits and examples of coverage based on a pool of pre-approved, marketing-written content.

- Benefits are selected based on customer relevance and to avoid repeating benefits emphasized in previously declined offers.
- *Note:* Field testing showed stronger performance when offers were anchored in an existing, pre-approved benefit set rather than fully generating net-new language. Additionally, prior Alpha experiments surfaced legal and compliance risks associated with dynamically generating new benefit copy.

### 4. **Objection-Aware Guidance**

Guidance tailored to known objections or historical decline reasons to help agents proactively address customer concerns.

- *Example:* Prior decline reason: “Too expensive” → Emphasize cost-of-repair comparisons and multi-device value.

### 5. Price & Ask for Sale

Structured prompts to ensure offers are compliant and consistent.

- Retain all required compliance language.
- Prompt experts to explicitly state the price and ask for the sale at the end of the offer.

---

### **Product requirements [HOW]**

|                                                    | **Product requirement**    | **User stories**                                                                                         | **Details**                                        | **Notes** |
|----------------------------------------------------|----------------------------|----------------------------------------------------------------------------------------------------------|----------------------------------------------------|-----------|
| **1**                                              | **Inputs**                 |                                                                                                          |                                                    |           |
|                                                    | Previous call history      | As an agent, I want prior interactions considered in the offer so it doesn’t feel repetitive or generic. | • Ingest transcripts or summaries from prior calls |           |
| • Extract decline reasons and key language signals | Utilize Customer 360 work. |                                                                                                          |                                                    |           |

Will need alternative experience when no call data available |
|  | Customer attributes | As an agent, I want offers tailored using household and device context so the pitch feels relevant. |   • Use attributes such as device types, device count, and presence of children
  • Support both historical and current-call data |  |
| **2** | **Outputs** |  |  |  |
|  | Context-aware tone guidance | As an agent, I want guidance on how to speak to the customer so I can match their emotional state and avoid sounding tone-deaf. |   • Detect sentiment and contextual cues (e.g., stress, frustration)
  • Output clear tone guidance (empathetic, concise, reassuring, etc.) |  |
|  | Recent interaction awareness (callbacks) | As an agent, I want awareness of recent callbacks so I can avoid repeating information and tailor the offer appropriately. |   • Detect recent callbacks or prior interactions
 • Adjust offer framing to acknowledge callback context | Callback awareness should focus on recent or repeated interactions (same day or same week), not full call history. |
|  | Benefit prioritization (Grounded in approved benefit set) | As an agent, I want the most relevant plan benefits highlighted so I can lead with what matters most to the unique customer. |   • Use pre-approved marketing material as the foundation, minimal generation
  • Prioritize benefits using customer attributes and call history to tailor relevance and avoid repetition | Field showed better performance and lower risk with pre-written marketing language. |
|  | Objection-aware guidance | As an agent, I want guidance on how to address known objections so I can proactively overcome prior concerns. |   • Surface previous decline reasons when available
  • Map decline reasons to approved objection-handling guidance
  • Display guidance contextually with the offer |  |
|  | Price & Ask for Sale | As an agent, I want a clear prompt to include price and an explicit ask for sale so I can offer compliantly and consistently. |   • Ensure price is always displayed using compliant language
  • Prompt the expert to explicitly ask for the sale at the end of the offer | Mirrors existing Smart Offer expectations rather than introducing new behavior. |

---

### MVP Scope

**Goal:** Refine the scope to a focused MVP that can be built and tested by the end of Q1. 

---

### **Success Metrics**

**Primary metrics**

- Offer usage
- NSP100

**Secondary metrics**

- NPS
- Compliance rate

---

### **Timeline [WHEN]**

- Targeting [MVP](Dynamic%20Smart%20Offer%20(aka%20Smart%20Pitch%20Personalized%20/MVP%20Scope%203059532a1f8680ffb925dc1dc98bc7e9.md) pilot by end of Q1 with iterations to MVP in Q2

---

### **Constraints**

- Prior call transcripts or summaries may be incomplete, delayed, or unavailable → Need to talk to C360 team on time of availability
- Decline reasons may not be consistently captured across channels

---

### **Non-Goals**

**Non-Requirements** (out of scope)

- 

**Future Requirements** (out of scope - ideas for future iterations)

- Dynamic objection handling (rebuttals) triggered during the call (vs. pre-call guidance)
- Personalized visual aids or pricing scenarios for multi-device households
- Logic to determine which product a customer is most likely to purchase

---