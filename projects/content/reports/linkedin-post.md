# LinkedIn Post: The PFF Case Study

A real case study from PFF, a 200-person sports data company. Two engineers. Two months. 25x more deploys. 10x output. Customer satisfaction up from a 7-7.5 baseline to 8.6 out of 10.

The reframe that drove it:

Stop asking how to make engineers faster. Start asking how to make agents faster.

The agile manifesto, the foosball tables, the sleeping pods, every perk that other industries do not have, all of it exists because engineers were the bottleneck. They are not the bottleneck anymore. Optimizing for the new bottleneck (agent throughput) is the work most engineering orgs in 2026 have not done yet.

What PFF cut:

- Sprint planning
- Daily standups
- Sprint refinement
- Project manager role

What survived:

- Half-hour huddles every other day
- Customer satisfaction surveys
- Deployment metrics
- Retrospectives

The pipeline replaced the ceremony.

The workflow is four autonomous stages plus two verification stages: spec, lightweight design document, auto-generated tickets, auto-generated PRs, auto-deploy to staging, QA agent that checks acceptance criteria. Self-healing loop coming next.

The single highest-return move was the LDD Skill. A Skill calibrated against PFF's existing design documents that keeps every new feature in the same architectural shape as everything else in the codebase. Without it, every PR is generic.

Full case study with the numbers, the diagrams, the operational specifics:

[link to Medium article]

Three things to know up front in the comments. What is the engineering ceremony you would have the hardest time cutting?
