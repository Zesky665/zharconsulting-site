---
dg-publish: true
tags:
  - QA
  - Bugs
  - Triage
  - Process
---

## What is it?

Triage is a system which enables people to quickly assign priority to problems of different severity. It originates from a medical practice where priority for treatment is assigned to victims in mass casualty events. 

The point is to empower individuals to make quick decisions about priority.

How this applies to bugs is fairly self-evident, different bugs have different severity. 
Some need immediate attention while others can wait for the next sprint.

## How to?

First, it's important to have a good description of the bug available, if a ticket is missing information make sure it gets it.

Then look over the bug report and see which of the following categories fit, if it fits multiple assigns the highest priority. 

#### Priority 4:
What is it?
- Blocks functionality.
How to handle it?
- Add to iteration immediately.
- Work on it immediately.

#### Priority 3:
What is it?
- Blocks functionality, but has a workaround.
- Blocks functionality but is an edge case.
- Looks bad in a prominent part of the app.
How to handle it?
- Add to iteration immediately.
- Work on its current sprint if there is a free dev, leave it for the next iteration otherwise.

#### Priority 2:
What is it?
- Annoying but doesn't block functionality.
- Looks bad in a not-so-prominent part of the app. 
How to handle it?
- Add to bug column.
- Add it in subsequent iterations if time allows.

#### Priority 1:
What is it?
- Annoying but doesn't block functionality, also niche. 
- A minor visual issue, only designers seem to notice it.
How to handle it?
- Add to bug column.
- Add it in subsequent iterations if time allows.
