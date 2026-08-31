---
dg-publish: true
tags:
  - Process
  - Bugs
  - DoD
  - Teamwork
  - Team_Building
---



Work process document or Definition of Done is a document developed by a team which lays out the agreed upon terms within that team. 

Having a DoD document is important for team building and for maintaining team cohesion in times of high turnover.

Below is an example of a DoD document I wrote while working in previous job

__________________________________________________________

— Start

📐 Ready for Dev: (What needs to be done before a ticket gets added to the interaction/epic)

1.  Discover requirements.
2.  Create ticket. (Link to acceptance criteria)
3.  Refine/Groom ticket.
4.  Assign an estimation point value.(user Fibonacci numbers)(watch [video](https://www.youtube.com/watch?v=vvr-Fd1xYCI))
5.  Add to Iteration.
6.  Assign developer. (optional)(take task order into consideration)

For Epics:

1.  List Data Needs (possibly during discovery)
2.  Add test cases. (Regression test cases)
3.  Data Request (if needed) has been created and sent.

— At this point, the development has started.

_The ticket gets developed._

— At this point, the development is mostly done.

🔍 Ready for Review:

1.  PR has been created. (make sure to fill out the template description)
2.  PR description has been filled in.
3.  Relevant GitHub/Bitbucket links have been added to the ticket.
4.  Unit tests have passed.(successfully 😅)
5.  PR has been assigned to the relevant reviewers.

👮🏻‍♂️ Ready for QA:

1.  Create Test Env.
2.  Any comments have been resolved.
3.  Dev + QA sync on test scenarios / Acceptance criteria.(optional)
4.  Write down and send to dev a list of things to test / test scenarios. (more detailed than the epic test cases)

🧐 Ready for Deployment to Staging:

1.  Manual Tests.
2.  Any bugs have been fixed. (Make sure any bugs are related to the PR and not the app in general)
3.  Run E2E tests.
4.  Update any e2e broken by the new changes.
5.  Create new e2e tests if needed. (optional)
6.  Squash commit history into readable chunks.
7.  Announce to Customer Support / Customer Success / Sales / ETC
8.  Make sure all of the translations are completed.
9.  Ask for a knowledge base article to be written. (Use this google form to request)
10.  Exploratory test session. (attendance optional, except for John Smith (UX Designer))

— At this point, the ticket is Done.

🦸 Ready for Deployment to Production:

1.  Merge to staging. (unless the DO NOT MERGE label is applied)
2.  Run e2e tests against staging. (they should be 🟢)
3.  Announce to Customer Support / Customer Success / Sales / ETC that the feature is being deployed to production.
4.  Data tracking set up and working (Pendo, Metabase, etc).
5.  Create PR for Production Branch.
6.  Ask for approval from all the other stakeholders.
7.  Make sure it’s not Friday.

🚀 Deploy to prod

1.  Merge the PR to Prod Branch.
2.  Celebrate 🎉.
3.  Announce that the Feature has been Released to Production - OvercommunicationFTW!!!✊
4.  Make sure the Prod app is not on 🔥.
5.  Put out any fires 🚒.

— At this point, the Epic is Done.

