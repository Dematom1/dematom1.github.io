+++
title = 'My prompts'
weight = 3
+++

What I actually said to the agents while building this application, quoted as typed, typos included. Each one is followed by what it changed.

### Starting the resume

> Refine the resume very, very well: review the current resume, review the job postings, and then grill the hell out of the candidate.

The agent questioned me claim by claim before drafting. Numbers I could not support were left off.

### Asking for a hard read

> Have another Fable 5.2 agent look at the resume as if they were Cohere filtering out applications to interview. Be critical.

The cold screen found that the page had no hands-on training on it. That became training-harness.

### Cutting my own summary

> Maybe this description is only going to hurt us and we should remove it altogether.

The summary I had written about coming to software from the humanities came off the page.

### Starting the project

> let's create a training project ... identify the scope and then we're gonna add it as an entry into the resume, probably under open source projects

### Asking what the training is for

> it would also help me to identify the goal. So what is the overall goal of training? Are we training for what? A more pleasant model around my repos?

The answer: the model is the load on the system; the measured infrastructure is the result.

### Choosing the tools

> [Kueue and TrainJob] are not one of the technology that's highlighted in the job posting. So I do want to use Slurm and or Ray. But you let me know what you think is best.

The plan became Slurm first, then Ray on Kubernetes, then JAX.

### Giving the model a job

> this doesnt tell me what the benefit its? for dfj, when I wanted to learn kafka and data streaming, I built click analytics for dfj as a use case. so whats the equiv here?

The model is fine-tuned on Data First Jobs postings to score listing quality in that pipeline.

### Keeping the claims honest

> I like the rec, but not machine learning, i was mlops not machine learning so careful there

The summary now describes infrastructure, not machine learning.

### Building these pages

> I would like to create a project page where I can show the team my work. I think it will be a good thing.

> looks good, but the cohere page is a hub, I want to there to be tabs and such since I want to show how I built it, my prompts with the ai, etc.
