# Contribution #4464: Implementations should report when a gateway name is too big

**Contribution Number:** 1 
**Student:** Arjun Sharma
**Issue:** https://github.com/kubernetes-sigs/gateway-api
**Status:** [Phase I] [In Progress / Complete]

---

## Why I Chose This Issue

I choose issue #4464 "Implementations should report when a gateway name is too big" because it aligns with my language preference. This issue is labeled good first issue and being my first time contributing to open source makes it a perfect match. While looking at the issue and reading description it seems doable.

I am Interested in this because:
I have experience with the programming language.
The size of codebase is comfortable to me.
The issue has clear accpetance criteria.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. Understood the issue: Gateway name + controller name exceeding 63 chars breaks child resources (per GEP-1762 naming conventions), but no condition signals this.
2. Read the condition types in apis/v1/gateway_types.go:1120-1365 — confirmed no NameTooLong or similar condition exists.
3. Read the GEP-1762 spec in geps/gep-1762/index.md:63-68 — confirmed the <NAME>-<GATEWAY CLASS> naming convention and label requirements.
4. Checked tests/naming.go — found the pseudo-code placeholder you created documenting the gap.
5. Checked tests/go.mod — it's a separate Go module that imports sigs.k8s.io/gateway-api/apis/v1, so I could write a real test leveraging the API types.
6. Wrote tests/naming_test.go with three tests:
- TestNoNameTooLongCondition — iterates all known GatewayConditionType constants, asserts none equals "NameTooLong"
- TestNamePlusControllerExceeds63 — computes len(name) + len(controller) for realistic combos, logs which exceed 63
- TestGeneratedResourceNameExceeds63 — models the GEP-1762 <NAME>-<GATEWAY CLASS> pattern, shows overflow
7. Removed old tests/naming.go (didn't compile — no package declaration, bare :=)
8. Ran go test -v -run ... — all three tests passed, reproducing the gap


### Reproduction Evidence

- **Commit showing reproduction:** [[Link to commit in your fork]](https://github.com/arjunexplorer/gateway-api/commit/ad725c13abe8489b4dcfbc9f742ae907a7f58964)
- **Screenshots/logs:**
- === RUN   TestNoNameTooLongCondition
--- PASS: TestNoNameTooLongCondition (0.00s)
=== RUN   TestNamePlusControllerExceeds63
    naming_test.go:85: WARNING: name "my-very-long-gateway-name-for-production-istio.io/gateway-controller" (len=68) exceeds 63 characters
    naming_test.go:85: WARNING: name "my-very-long-gateway-name-for-production-traefik.io/gateway-controller" (len=70) exceeds 63 characters
=== RUN   TestGeneratedResourceNameExceeds63
    naming_test.go:117: OVERFLOW: "a-abcdefghij-abcdefghij-abcdefghij-abcdefghij-abcdefghij-abcdefghi" (len=66) exceeds limit by 3 characters
--- PASS: TestGeneratedResourceNameExceeds63 (0.00s)
PASS
ok  	sigs.k8s.io/gateway-api/tests	0.174s
- **My findings:** The Gateway API defines no NameTooLong condition type despite GEP-1762 requiring generated resource names of the form <NAME>-<GATEWAY CLASS>. Realistic gateway names combined with common controller names routinely exceed the 63-character Kubernetes resource name limit, meaning implementations silently truncate or fail to create child resources with no signal back to the user.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

Understand: No NameTooLong condition exists, so controllers that truncate/break child resources when len(gatewayName) + 1 + len(controllerName) > 63 have no way to signal the problem to users (GEP-1762).
Match: Follow the existing condition pattern in apis/v1/gateway_types.go — e.g., GatewayConditionAccepted with its paired GatewayReasonAccepted constant, doc comment, and const block.
Plan:
1. Add GatewayConditionNameTooLong and GatewayReasonNameTooLong constants in apis/v1/gateway_types.go following the existing doc-comment style
2. Update TestNoNameTooLongCondition in tests/naming_test.go to assert the condition does exist (flip from negative to positive test)
Implement: https://github.com/arjunexplorer/gateway-api
Review: Verify format matches existing conditions (doc comment, const block placement, naming convention GatewayCondition*/GatewayReason*).
Evaluate: go test ./tests/ -run NameTooLong passes; confirm the condition is absent before the change and present after.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
