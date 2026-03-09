# Research: Generative Offer Support in Field & Alpha

# Alpha

***TL;DR** Limited personalization. Device-based personalization largely mirrored what Workspace already supports via “Example of Coverage.” Alpha lacked this capability at the time.*

### **Overview**

- “Personal Offer” launched across Cricket and piloted with Verizon for ~6 weeks
    - Positive results for Cricket
    - Negative results for Verizon
- Experience used a full offer checklist as a baseline, with personalization applied only to the plan benefits section

### **Personalization Approach**

- Based on:
    - Devices collected
    - Number of devices
    - Presence of children in household
- Key issues observed:
    - Inaccurate language (e.g., “all devices protected” vs. “eligible devices”)
    - Over-claiming coverage created trust and compliance concerns

### **Learnings & Current State**

- All clients (including Cricket) are now 100% using “Smart Offer”
- Preliminary analysis (Venkat):
    - Biggest drivers in sales decision:
        - Core benefits
        - Example of coverage
        - Price
    - Implication: Personalization should focus on core benefits and example of coverage
    - Gap: *Example of coverage is not currently available in Alpha*

### **Alpha Results**

- See visual summary:

![image (129).png](Research%20Generative%20Offer%20Support%20in%20Field%20&%20Alpha/image_(129).png)

- Supporting materials:

[Sales for Protection_Alpha Readout 2.18.2025.pptx](Research%20Generative%20Offer%20Support%20in%20Field%20&%20Alpha/Sales_for_Protection_Alpha_Readout_2.18.2025.pptx)

[Personalized Offers in Alpha - Cricket.pptx](Research%20Generative%20Offer%20Support%20in%20Field%20&%20Alpha/Personalized_Offers_in_Alpha_-_Cricket.pptx)

---

# Field

***TL;DR** Although Smart Pitch usage cannot be identified with certainty due to lack of transcript for field visits, results are promising. This approach demonstrates a higher degree of personalization than Alpha testing.*

### **Context**

- ~40–50% of customers call in before a field visit
    - Information from these calls is used to generate a GenAI-powered offer they call “Smart Pitch”
- They were not pushed by legal/reg to move to “Smart Offer” due to:
    - Lack of field transcripts
    - Observed positive impact from Smart Pitch as a standalone experience

### **Pilot Results (High-Level)**

- Test group showed +2 bps lift in Verizon and AT&T
- Plan to scale once migrated to GAIA
- Smart Pitch page load time is a few seconds
    - Usage inferred when experts allow the page to fully load

### **Inputs Used**

- Decline reason from prior call (if applicable)
- Tone derived from prior interaction
- Customer attributes:
    - Tech savviness
    - Presence of children
    - Devices owned

### **Smart Pitch Structure**

1. **Customer Overview**
2. **Tone Guidance**
    - Example: Customer mentioned a chaotic life → emphasize empathy
3. **Three Pitch Points**
    - Selected from a pool of ~20 options
    - Never identical wording
    - More generic phrasing performed better
        - Hypothesis: Marketing-written language outperforms AI-generated copy
4. **Decline Reason + Objection Handling**
    - Built but not launched
    - Disabled due to concerns it could discourage sales

### **Field Results**

![image.png](Research%20Generative%20Offer%20Support%20in%20Field%20&%20Alpha/image.png)

- **Overall, the test group’s SP100 increased in Pilot 2 vs the pre-pilot period by 1.6 compared to the control group’s increase of 0.06.** By product (HTP and VHDP), these changes were 2.81 vs 1.08 and 1.36 vs (0.82), respectively. The table below shows these changes.
- The DiD in the combined SP100 are statistically significant at the 80% confidence level.
    - **80% confidence means that there is a 20% chance of concluding the results were statistically significant when that is not the case.**
- On the product level, the change in SP100 between the two groups in the Pilot 2 period is larger for VHDP than HTP.
- Conclusions:
    - There is an indication that the impact of the Smart Pitch pilot is positive across the two measured products as the test group improved more over time than the control group, but that increase is only statistically significant at the 80% confidence level (1 in 5 odds of being wrong).
    - This means that the effect of the Smart Pitch is small enough (combination of the impact of the feature along with its minimal 5% click-through rate) that the impact to the overall metric is difficult to identify when comparing the two groups.
    - The fact that the product level DiD is only statistically significant (even at the 80% confidence level) for VHDP and not HTP implies that the pitch feature may be less effective with the HTP product compared to the VHDP.
    - External processes like sales propensity routing combined with overstaffing could impact the validity of these results.
- Analysis Type: Difference-in-Difference (DiD)
    - Compares the test and control groups to see if the normal variances in performance were impacted by the pilot treatment. For example, the Control and Test groups in an experiment usually have a gap in a key KPI of 2% with Control group performing better than the Test group. After the introduction of a new pilot feature for the Test group, that difference in performance has shrunk to 0% indicating that the new feature has had a positive impact on the performance of the test group.
- Analysis Groups and Pilot Periods:
    - Test Group: 12 markets (Atlanta GA, Buffalo NY, Cedar Rapids IA, Charleston SC, Honolulu HI, Memphis TN, Minneapolis MN, San Jose CA, Shreveport LA, Tallahassee FL, Washington DC, Wichita KS)
    - Control Group: Used all 82 remaining markets per feedback and guidance from Senior Leadership.
    - Pre-Pilot: 7/1 through 8/14 (VHDP) or 8/18 (HTP)
    - Pilot 1: 8/14 (VHDP) or 8/18 (HTP) through 9/14
    - Pilot 2: 9/15 through 11/11

| **Experiment Group** | **SP100** | **Pre-Pilot** | **Pilot 1** | **Pilot 2** | **Pre-Pilot and Pilot 2 Delta** |
| --- | --- | --- | --- | --- | --- |
| **Control** | **Combined** | 21.56 | 22.00 | 21.62 | 0.06 |
|  | **HTP** | 24.07 | 25.01 | 25.15 | 1.08 |
|  | **VHDP** | 18.18 | 18.09 | 17.36 | -0.82 |
| **Test** | **Combined** | 18.51 | 18.41 | 20.12 | 1.60 |
|  | **HTP** | 21.24 | 21.83 | 24.05 | 2.81 |
|  | **VHDP** | 14.44 | 14.11 | 15.80 | 1.36 |