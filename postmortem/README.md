# Postmortem Index

This directory contains postmortem analyses of significant issues discovered and resolved in the Jira Execution Planner project.

## Purpose

Postmortems serve to:
- Document root causes and resolutions
- Share lessons learned across the team
- Prevent similar issues in the future
- Build institutional knowledge
- Improve development practices

## Postmortem List

### 2026-01-27 Session: Scenario Planner Improvements

| # | Title | Severity | Status | Summary |
|---|-------|----------|--------|---------|
| [001](./001-performance-degradation-page-load.md) | Performance Degradation on Page Load | High | ✅ Resolved | Page load time increased to 3-5 seconds due to missing early return guards and incorrect memoization dependencies in 12 React useMemo hooks |
| [002](./002-false-conflict-detection.md) | False Conflict Detection | Medium | ✅ Resolved | Conflict detection algorithm flagged correctly scheduled tasks as conflicts due to: excluded tasks being checked, O(n²) all-pairs comparison, and improper date comparison |
| [003](./003-lane-stacking-assignee-interleaving.md) | Lane Stacking Assignee Interleaving | High | ✅ Resolved | Tasks from different assignees appeared joined on same visual row due to assignee-agnostic lane stacking algorithm |

## Postmortem Template

Each postmortem follows this structure:

```markdown
# Postmortem #XXX: [Title]

**Date**: YYYY-MM-DD
**Severity**: [Critical/High/Medium/Low]
**Status**: [Resolved/In Progress/Monitoring]
**Author**: [Name]

## Summary
Brief description of the incident

## Impact
- Users affected
- Duration
- Symptoms

## Root Cause
Technical details of what went wrong

## Timeline
Chronological events

## Resolution
How it was fixed

## Verification
How the fix was validated

## Lessons Learned
- What went well
- What could be improved

## Action Items
- [x] Completed items
- [ ] Pending items

## Prevention
How to avoid similar issues

## Related Issues
Links to related postmortems

## References
Commits, files, documentation
```

## Statistics

### By Severity
- **High**: 2 postmortems (67%)
- **Medium**: 1 postmortem (33%)
- **Critical**: 0 postmortems
- **Low**: 0 postmortems

### By Status
- **Resolved**: 3 postmortems (100%)
- **In Progress**: 0 postmortems
- **Monitoring**: 0 postmortems

### By Category
- **Performance**: 1 postmortem
- **Frontend Logic**: 2 postmortems
- **Backend**: 0 postmortems
- **Infrastructure**: 0 postmortems

## Common Themes

### Issues Found
1. **Testing Gaps**: Insufficient testing with empty/edge case data
2. **Algorithm Validation**: Need peer review for complex algorithms
3. **Backend/Frontend Alignment**: Frontend didn't match backend logic
4. **Performance**: Missing optimization guards in React hooks

### Action Items Summary
Across all postmortems, key actions needed:

**Immediate** (Already Done):
- ✅ Add early return guards to expensive computations
- ✅ Fix missing memoization dependencies
- ✅ Optimize conflict detection from O(n²) to O(n)
- ✅ Implement assignee-aware lane stacking

**Short Term** (TODO):
- [ ] Add ESLint rule: `react-hooks/exhaustive-deps` enforcement
- [ ] Add performance tests for empty data states
- [ ] Add integration tests with real Jira data
- [ ] Add visual regression tests
- [ ] Document algorithm design decisions

**Medium Term** (TODO):
- [ ] Performance budget metrics in CI
- [ ] Conflict detection accuracy metric
- [ ] Consider moving conflict detection to backend
- [ ] Add assignee labels to lane rows

## Contributing

When creating a new postmortem:

1. **Number it sequentially**: `00X-short-title.md`
2. **Use the template** above
3. **Be blameless**: Focus on systems, not people
4. **Be specific**: Include code snippets, data, screenshots
5. **Be actionable**: List concrete action items
6. **Update this README**: Add entry to the table

## Related Documentation

- [SCENARIO_PLANNER_ANALYSIS.md](../SCENARIO_PLANNER_ANALYSIS.md): Original feature analysis
- [SCENARIO_BUG_ANALYSIS.md](../SCENARIO_BUG_ANALYSIS.md): Detailed bug analysis for #003

## Questions?

For questions about postmortems or to discuss issues, contact the development team.

---

*Last Updated: 2026-01-27*
*Total Postmortems: 3*
