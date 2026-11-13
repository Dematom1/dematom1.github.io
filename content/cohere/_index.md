+++
title = 'For the Cohere team'
linkTitle = 'Overview'
robots = 'noindex, nofollow'
css = ['css/cohere.css']
outputs = ['html']
[build]
  list = 'never'
[sitemap]
  disable = true
# Never listed: Cohere pages stay out of every list, RSS feed and sitemap; the hub's tab bar fetches its pages by path.
[[cascade]]
  robots = 'noindex, nofollow'
  css = ['css/cohere.css']
  outputs = ['html']
  [cascade.build]
    list = 'never'
  [cascade.sitemap]
    disable = true
+++

Hello. I'm Laszlo Horanszky, a data platform engineer in Calgary, and this page goes with my application for **Member of Technical Staff, Training Infra Engineer**. This page shows the work behind my [resume](/resume/), with links, and where I stand against the posting. The other tabs follow [training-harness](/projects/training-harness/), the training project I am building now: [how I'm building it](building/), with the tradeoffs, and [my prompts](prompts/) to the coding agents that help.

## Why this role

The posting asks for someone to "improve our training setup from an infrastructure and codebase performance standpoint" and to "craft and implement tools to speed up our training cycles". That is the layer I work in: the clusters, jobs and pipelines that models run on, and the tooling around them. At Teamworks I ran the operations side of production models that data scientists own; I did not build the models. The step I am taking now is to do this work where training itself is the workload, and Cohere trains its own models.

## The work that maps to it

**Distributed compute on Kubernetes**

- In my first month at Teamworks I designed and shipped the system that runs a Monte Carlo roster model: a FastAPI service and a three-stage Celery chain fan out Kubernetes indexed jobs, up to 4,000 parallel workers on 16-CPU pods. A manual process became a 45-minute run.
- Co-led the move of 38 production models from a daily batch to event-driven Kubernetes jobs (GCP Workflows, per-model Pub/Sub fan-out): results in under 1 hour instead of 24+. I converted 8 myself.
- In an architecture review I found Kubernetes jobs sized and split wrongly on over-provisioned nodes; the redesigned layout saved $60K in compute, confirmed independently by a staff engineer.

**The infrastructure around models**

- Stood up an MLOps stack from scratch inside a client's own cloud as one of 2 engineers with production access: Kubernetes clusters, Terraform, Airflow workflows, model versioning APIs and model serving.
- Ran the operations side of 60+ production models (PyTorch and custom): Airflow orchestration, Kubernetes indexed jobs, training and retraining pipelines, on call.
- Designed the data platform's observability and ran its incidents, including an S3 bulk delete recovered with zero data loss and a production 404 traced across five systems to an upstream bug.

**Scheduling and reliability, in the open**

- [dibs](https://github.com/Dematom1/berth) (Go): a job admission and lease scheduler for machines many agents share. FIFO admission against CPU and memory budgets, no backfill so large jobs cannot starve, and 3 TLA+ specs model-checked in CI.
- [firstmate](https://github.com/kunchenguid/firstmate) and [no-mistakes](https://github.com/kunchenguid/no-mistakes): an orchestrator for crews of coding agents and a gate that reviews and tests their changes. I have 11 merged pull requests across them, including liveness detection for stalled workers.

**Training, now**

- [training-harness](/projects/training-harness/) ([repository](https://github.com/Dematom1/training-harness)): it will train a hand-written 124M-parameter GPT in PyTorch, data-parallel on 8 L4 spot GPUs (Slurm first, then Ray on Kubernetes, then a JAX port), and fine-tune it on postings from [Data First Jobs](https://datafirstjobs.com), my job board ([architecture](/posts/df-jobs-architecture/)), to score listing quality. It is in progress and nothing has run yet; the project page updates as each step completes.

## Against the posting, honestly

| The posting asks for | Where I stand |
|---|---|
| Extremely strong software engineering skills | Production distributed systems, above. I build with a fleet of coding agents that I direct; the design, review and verification are mine. The training-harness loop, launcher and checkpoint code will be written by hand. |
| Python and ML frameworks such as JAX, PyTorch and XLA/MLIR | Python in production (FastAPI, Celery, async SQLAlchemy). PyTorch: I have run the operations for PyTorch models in production; training-harness will be my first training code. JAX: its final step. XLA/MLIR: none yet. |
| Distributed training infrastructure (Kubernetes, Slurm) and frameworks (Ray) | Kubernetes in production (GKE, EKS, K3s). Slurm and Ray: training-harness, in progress. |
| Large-scale distributed training strategies | Not yet. Data parallel across 8 GPUs is the project's scope. |
| Hands-on training of large models at scale, and contributing to the tooling or setup of training infrastructure | The tooling and setup side, yes: the MLOps stack and the training and retraining pipelines above. Training a large model at scale, no. |

## Contact

laszlo@hey.com &middot; [LinkedIn](https://www.linkedin.com/in/laszlohoranszky/) &middot; [GitHub](https://github.com/Dematom1) &middot; [Resume](/resume/)
