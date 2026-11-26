# Architecture Decision Record Template

## 1. Context

[Describe the issue or problem that needs to be addressed by this architecture decision.]

## 2. Options

[List all viable alternatives that were considered for addressing the context problem.]

## 3. Decision

[State which option was chosen and why it was selected over others.]

## 4. Tradeoffs

[Detail the pros and cons of the chosen approach, including any compromises made.]

## 5. Tests

[Specify how the decision will be tested or validated, including negative test cases.]

## 6. Supersession

[List any previous decisions that this one supersedes or modifies.]

## 7. Authority Rule

[Define the rule that determines who has authority to make or approve this decision.]
- Only project owners may approve ADRs
- Agents may propose ADRs but may not mark them accepted or change an accepted contract on their own
- All changes to accepted contracts must be documented in a corresponding ADR and coordinated with all stakeholders
- ADRs transition from "Proposed" to "Accepted" only when approved by project owners

## 8. Validation

[Describe how we validate that this decision is working as intended.]

## 9. Related Documents

[Link to any related documents, contracts, or decisions.]

## 10. Example Empty ADR

### ADR-000: Template Example

**Status:** Proposed  
**Date:** 2026-11-26  
**Owner/Approver:** [Project Owner Name]

This is an example of a blank ADR using the template format. It demonstrates the structure and metadata required for a valid ADR.

## 11. Negative Test

Reject undocumented changes to accepted contracts.
- If a PR changes an accepted contract/invariant without linking a corresponding proposed-and-approved ADR and updating references, it must be rejected
- This can be validated by ensuring all contract changes are properly referenced in the ADR and that the PR includes the necessary documentation of the change