# Scenario Planner Bug Analysis

## Issue Description
Two tasks assigned to different people ("Bidcast haproxy" and "Merge rep") appear on the same visual row in team mode, causing user confusion.

## Data from Screenshots and API

### Tasks Involved (R&D Reliability team, Feb-Apr 2026):

**Nikolai Baltsevich:**
1. TECH-25516: "Bidcast haproxy autoscalling preparation"
   - Dates: 2026-02-12 to 2026-03-12
   - SP: 2.0

2. TECH-26132: "Migrate from n2d in gce-or and gce-nl"
   - Dates: 2026-03-12 to 2026-03-26
   - SP: Unknown (need to check)

3. TECH-26133: "Run c4a Haproxy"
   - Dates: 2026-03-26 to 2026-04-09
   - SP: 1.0

**Dmytro Hopkalo:**
1. TECH-25313: "Merge repositories for gitlab-ci in deploy process for bswx"
   - Dates: 2026-03-12 to 2026-03-26
   - SP: 1.0

## Root Cause Analysis

### Backend Scheduler Behavior
The backend scheduler (planning/scheduler.py) correctly serializes tasks for the same assignee:
- Nikolai's tasks are scheduled consecutively: Task 1 ends when Task 2 starts, Task 2 ends when Task 3 starts
- Dmytro's task runs in parallel with Nikolai's Task 2 (different assignees = allowed to overlap)

**This is CORRECT behavior** - different people can work in parallel.

### Frontend Lane Stacking Issue
The lane stacking algorithm (jira-dashboard.html lines 6510-6530) places tasks on rows based ONLY on time:

```javascript
let rowIndex = rowEnds.findIndex(rowEnd => (
    isUnscheduled ? start > rowEnd : start >= rowEnd
));
```

This algorithm:
1. Places "Bidcast" (Nikolai) on Row 0, ending 2026-03-12
2. Sees Row 0 ends on 2026-03-12
3. Places "Merge" (Dmytro, starts 2026-03-12) on Row 0 because `start >= rowEnd`
4. Also needs to place "Migrate from n2d" (Nikolai, starts 2026-03-12) somewhere

**THE BUG**: Both "Merge" (Dmytro) and "Migrate from n2d" (Nikolai) start on the same day (2026-03-12). The lane stacking algorithm places them on different rows, but visually it looks like:
- Row 0: [Bidcast (Nikolai)] [Merge (Dmytro)] [Run c4a (Nikolai)]
- Row 1: [Migrate from n2d (Nikolai)]

This creates visual confusion because:
1. Nikolai's tasks appear non-consecutive visually
2. Different assignees' tasks appear "joined together" on the same row
3. User expects to see clear assignee separation in team mode

## The Actual Problem

The lane stacking algorithm is **assignee-agnostic**. It should:
1. Keep same-assignee tasks visually consecutive when possible
2. Prevent "interleaving" of different assignees' tasks on the same row

## Proposed Fix

Modify the lane stacking algorithm to be assignee-aware in team mode:

```javascript
const assignRows = (issueList, rowEnds, baseOffset, rowAssignees) => {
    issueList.forEach((issue) => {
        if (!issue?.key) return;
        const assignee = issue.assignee || 'Unassigned';
        const isUnscheduled = !issue.start || !issue.end;
        const start = parseScenarioDate(issue.start) || fallbackStart;
        const end = parseScenarioDate(issue.end) || start;
        const normalizedEnd = end < start
            ? start
            : (isUnscheduled ? new Date(start.getTime() + DAY_MS) : end);

        // Find a row that:
        // 1. Has time available (start >= rowEnd), AND
        // 2. Either has no assignee yet, OR has the same assignee
        let rowIndex = rowEnds.findIndex((rowEnd, idx) => {
            const timeAvailable = isUnscheduled ? start > rowEnd : start >= rowEnd;
            const assigneeMatch = !rowAssignees[idx] || rowAssignees[idx] === assignee;
            return timeAvailable && assigneeMatch;
        });

        if (rowIndex === -1) {
            rowIndex = rowEnds.length;
            rowEnds.push(normalizedEnd);
            rowAssignees[rowIndex] = assignee;
        } else {
            rowEnds[rowIndex] = normalizedEnd;
            rowAssignees[rowIndex] = assignee;
        }
        rowIndexByKey.set(issue.key, baseOffset + rowIndex);
    });
};
```

This ensures:
- Same assignee tasks stay on the same row when sequential
- Different assignees get separate rows within a team
- Visual clarity: each row represents one person's timeline
