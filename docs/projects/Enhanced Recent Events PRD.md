# **Enhanced Recent Events**

## **Context**

The Review AI Orchestration Rule workflow surfaces a panel of Recent CEF Events alongside the condition editor. This panel is designed to give engineers a real-time reference point while authoring orchestration rules, enabling them to verify that the conditions they define will match events that exist in their environment.

Currently, the panel always shows all recent events regardless of the condition being built. There is no way to filter the list to events that match the condition currently being edited, and there is no explicit action required before a condition query is applied. This means users cannot easily confirm whether an incomplete or in-progress condition would match expected events, and there is a risk that a partially written condition is inadvertently used to query events before the author intended.

This PRD describes enhancements to the Recent Events panel that give users explicit control over what events are displayed and when a filter is applied.

## **Persona**

**Buyer Persona: CIO, Head of IT, Head of Operations, VP Infrastructure**

Buyers are interested in reducing operational overhead and maximising the ROI of the PagerDuty platform. Higher-quality orchestration rules translate directly to fewer false-positive incidents, reduced FTE hours, and a lower MTTR.

**User Persona: SRE Engineer, DevOps Engineer, SRE Manager, NOC Manager**

The primary user builds and maintains event orchestration rules. They need confidence that the conditions they configure accurately target real event payloads before committing a rule to production. They operate in environments where a misconfigured rule can cause noise or suppress critical alerts.

## **Problem**

Users authoring orchestration rules have no reliable way to validate that their conditions match the right events before publishing. The current Recent Events panel is always populated with all recent events, providing a reference but no filtering capability. This creates two compounding issues:

* Lack of condition-based filtering: Engineers must mentally compare condition logic against event payloads row by row, which is error-prone and time-consuming, especially for complex multi-field conditions.  
* No explicit apply action: Because the panel is always in a live “all events” view, there is no clear mechanism to signal “I am ready to test this condition.” Users who want to validate a condition mid-edit risk applying an incomplete condition query, producing misleading results that may cause them to make incorrect changes to their rule logic.

These gaps reduce confidence in the authoring workflow, increase the chance of misconfigured rules reaching production, and slow down the overall time to create a well-tuned orchestration.

## **Value Proposition**

Enhanced Recent Events solves the validation gap in the orchestration authoring workflow. By giving users the ability to view either all recent events or only events matching their current condition — and by requiring an explicit action to apply the filter — the feature delivers:

* Confidence before publishing: Engineers can verify that their conditions behave as expected against real data before a rule goes live.  
* Protection against accidental queries: The explicit Apply Condition button prevents a half-finished condition from being used to filter events, eliminating a common source of confusion.  
* Faster rule authoring: Immediate visual feedback on condition coverage reduces the back-and-forth iteration cycle and lowers the skill floor for authoring accurate rules.

**Key Metrics:**

* Reduction in time to complete a rule authoring session  
* Increase in % of published rules that match events on first deployment  
* Adoption rate of the “Matching Events” view toggle  
* Number of Apply Condition button interactions per session  
* Reduction in support tickets related to misconfigured conditions

## **Features & Requirements**

### **High Level User Stories**

| High Level User Story | Priority |
| :---- | :---- |
| As a User, I want to toggle between viewing all recent events and only events that match my current condition, so that I can understand the coverage of my rule against real event data. | P1 |
| As a User, I want an explicit “Apply Condition” button to trigger the matching-events filter, so that I do not accidentally query events with an incomplete or in-progress condition. | P1 |
| As a User, I want the Recent Events panel to remain in the “All Events” view by default when I open the rule editor, so that I always have a reference set of events visible without taking any action. | P1 |
| As a User, I want the “Apply Condition” button to be visually disabled or hidden when the condition editor is empty or invalid, so that I am not misled into running a filter against a blank condition. | P1 |
| As a User, I want the matching-events view to automatically refresh when I click Apply Condition again after modifying my condition, so that the results always reflect my latest changes. | P1 |
| As a User, I want to see a count of how many events match my applied condition compared to total recent events, so that I can quickly assess the scope and impact of my rule. | P1 |
| As a User, I want a clear visual indicator in the panel when the matching-events view is active, so that I always know which view mode I am in. | P2 |
| As a User, I want the panel to display a helpful empty state when no recent events match my applied condition, so that I understand the result is valid and not an error. | P2 |
| As a User, I want the selected view mode (all events vs. matching events) to persist within my current editing session, so that I do not need to re-select it every time I make a change to the condition. | P2 |
| As a PM, I want Segment analytics events emitted for ‘Toggle to Matching Events’, ‘Apply Condition Clicked’, and ‘Toggle to All Events’, so that I can measure adoption and identify friction points. | P2 |
| As a User, I want the “Apply Condition” button to show a loading state while the matching events are being fetched, so that I know the system is working and do not click the button repeatedly. | P2 |
| As a User, I want an error state in the matching-events panel if the condition query fails, so that I understand the issue without losing my work. | P3 |

## **Solution**

* 

## **Designs**

[https://www.figma.com/design/S1Wg2r4tKjvqGXnF0H87VF/AI-orchestrations?node-id=2161-16119\&t=7LLVSEwr4bo9nXJ6-0](https://www.figma.com/design/S1Wg2r4tKjvqGXnF0H87VF/AI-orchestrations?node-id=2161-16119&t=7LLVSEwr4bo9nXJ6-0)

## **Success Metrics**

* 

**Adoption Tiers:**

* Configured — User opens the panel and clicks Apply Condition at least once  
* Adopted — User uses the Matching Events view in ≥ 25% of their authoring sessions  
* Engaged — User uses the Matching Events view in ≥ 50% of their authoring sessions

