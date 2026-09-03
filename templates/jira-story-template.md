# Jira Story Template

## Epic
**Title**: {Epic Title}
**Description**: {High-level description of the modernization epic}
**Business Value**: {Why this matters to the business}
**Acceptance Criteria**:
- [ ] {Epic-level acceptance criteria}

---

## Story
**Title**: As a {role}, I want {goal} so that {benefit}
**Epic Link**: {Parent Epic}
**Priority**: {Critical / High / Medium / Low}
**Story Points**: {estimate}

### Description
{Detailed description of what needs to be built/migrated}

### Technical Notes
- **Legacy System Reference**: {COBOL program / batch job / interface}
- **Source Documentation**: {BRD section / spec reference}
- **Impacted APIs/Services**: {list}
- **Database Changes**: {schema changes needed}

### Acceptance Criteria
- [ ] {Given/When/Then format criterion 1}
- [ ] {Given/When/Then format criterion 2}
- [ ] {Given/When/Then format criterion 3}

### Test Scenarios
| Scenario | Input | Expected Output | Priority |
|----------|-------|-----------------|----------|
| {name}   | {in}  | {out}           | {P1-P3}  |

### Dependencies
- **Blocked By**: {other stories/tasks}
- **Blocks**: {downstream stories/tasks}

### Definition of Done
- [ ] Code complete and peer reviewed
- [ ] Unit tests passing (>80% coverage)
- [ ] Integration tests passing
- [ ] API contract validated
- [ ] Documentation updated
- [ ] Deployed to staging and smoke tested
