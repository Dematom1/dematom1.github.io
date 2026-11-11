+++
title = 'How I built this'
weight = 2
+++

I build with a crew of coding agents that I direct. [firstmate](https://github.com/kunchenguid/firstmate) runs the crew: it hands each agent one task in its own copy of the repository and reports back to me. [no-mistakes](https://github.com/kunchenguid/no-mistakes) is the gate every change passes before it ships: review, tests, lint and CI. The agents write most of the code and drafts. The design, the decisions, the corrections and the final approval are mine.

This application was built the same way, in November 2026. My own words to the agents are on the [prompts](../prompts/) tab.

## The trail

1. **The resume.** An agent read every Member of Technical Staff posting on your job board, picked the roles that fit, and then questioned me on my own work, claim by claim, before it wrote a line. Numbers I could not back up stayed off the page.
2. **A cold screen.** A second agent read the result the way your screeners would: a recruiter with 30 seconds, then a training infrastructure hiring manager. Its verdict: the page showed no hands-on training. That was true, and it changed what I did next.
3. **Twelve rounds of review.** Each draft went onto an annotated review board. I marked lines and answered questions, the agent revised, and the page went back up until I approved it.
4. **The project.** A design agent turned the gap into [training-harness](/projects/training-harness/): a map of the posting's tools, GPU prices, and a plan in steps. I moved it from Kueue to Slurm and Ray because those are the tools your posting names, and gave the model a job on my own job board.
5. **These pages.** An agent built this hub, the [resume page](/resume/) and the [project page](/projects/training-harness/) in this site's repository and showed me desktop and mobile previews on a board. I asked for this hub with tabs. The change went through no-mistakes before it was merged.

## Who does what

| | Agents | Me |
|---|---|---|
| Research | Read postings, my notes and repositories; price GPUs | Choose what to pursue |
| Writing | Draft every line and revision | Correct facts and wording, cut what is not true |
| Code | Write it, test it, review each other's changes | Design it, review it, verify it runs |
| Shipping | Run the gate: review, tests, lint, CI | Approve the merge |

## What I correct

The agents are fast and confident, so most of my work is catching where a draft says more than is true. Two examples from this application:

- A proposed summary described my work as machine learning. I ran the operations side of production models; data scientists built them. The line was rewritten to describe the infrastructure.
- An early project plan used Kueue and Kubeflow Trainer. They are good tools, but your posting names Slurm and Ray, so the plan changed to those.
