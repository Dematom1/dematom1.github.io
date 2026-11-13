+++
title = 'Resume'
type = 'resume'
robots = 'noindex, nofollow'
css = ['css/resume.css']
[build]
  list = 'never'
[sitemap]
  disable = true
+++

# Laszlo Horanszky

Calgary, Canada | laszlo@hey.com | [LinkedIn](https://www.linkedin.com/in/laszlohoranszky/) | [GitHub](https://github.com/Dematom1)

I build the infrastructure that models train and run on: distributed compute, training pipelines and data platforms.

## Experience

### Teamworks (formerly Zelus Analytics) | Remote, Canada

#### Data Engineer II, tech lead of a 7-person data platform team | Aug 2025 - Present

- Re-architected vendor data integration from a single-tenant application into a multi-tenant, event-driven lakehouse (SQS, Kafka, Hudi; Iceberg and DuckDB for query): about 30 TB, growing 170 GB a day.
- Built the lakehouse-to-application sync on commit-time watermarks: 100M+ row changes a day from a 13B-row source without rescans; traced sync failures to 96% no-change writes and removed the write amplification.
- Identified that the legacy applications could not support the company's AI initiatives; pitched the VP and CTO, unprompted, the system to build 200+ integrations cheaply and quickly.
- Built that system: a registry-driven data model (onboarding fell from about 6 months to about an hour, 8 live), a build system with reproducible images and graph-generated CI, and a custom coding-agent distribution that generates each remaining integration for about $90.
- Co-led the move of 38 production models from a daily batch to event-driven Kubernetes jobs (GCP Workflows, per-model Pub/Sub fan-out): results in under 1 hour instead of 24+. Converted 8 myself.
- Designed the platform's observability and ran its incidents: an S3 bulk delete recovered with zero data loss; a production 404 traced across five systems (Hudi, XTable, Iceberg, Glue, DuckDB) to an upstream bug.
- Found Kubernetes jobs sized and split wrongly on over-provisioned nodes in an architecture review; redesigned the layout and saved $60K in compute, confirmed independently by a staff engineer.
- Lead the team's architecture reviews, about a dozen so far, and sat on hiring panels for 2 senior hires.

#### Data Engineer I | Apr 2024 - Aug 2025

- Designed and shipped, in my first month and ahead of the MLB trade deadline, the distributed system that runs a Monte Carlo roster model: a FastAPI service and three-stage Celery chain fan out Kubernetes indexed jobs, up to 4,000 parallel workers on 16-CPU pods. A manual process became a 45-minute run.
- Stood up an MLOps stack from scratch inside a client's own cloud as one of 2 engineers with production access: Kubernetes clusters, Terraform, Airflow workflows, model versioning APIs and model serving.
- Ran the operations side of 60+ production models owned by data scientists (PyTorch and custom): Airflow orchestration, Kubernetes indexed jobs, training and retraining pipelines, on call.

**Earlier (2018 - 2023):** Data Analyst, KOHO Financial; Senior Allocations Analyst, Loblaws; Data Developer, Luniko.

## Open Source and Projects

Built with a fleet of coding agents that I direct; the design, review and verification are mine.

- [training-harness](https://github.com/Dematom1/training-harness) (PyTorch, Slurm; in progress), 2026: trains a 124M-parameter GPT on postings from my job board to score listing quality in its pipeline; hand-written loop, data-parallel on 8 L4 spot GPUs; Ray and JAX next.
- [dibs](https://github.com/Dematom1/berth) (Go), 2026: a job admission and lease scheduler for machines many agents share: FIFO admission against CPU and memory budgets, no backfill so large jobs cannot starve; 3 TLA+ specs model-checked in CI.
- [firstmate](https://github.com/kunchenguid/firstmate) and [no-mistakes](https://github.com/kunchenguid/no-mistakes) (8.8K and 9.2K GitHub stars): an orchestrator for crews of coding agents and a gate that reviews and tests their changes; 11 merged pull requests, including liveness detection for stalled workers.

## Skills

**Platform:** Kubernetes (GKE, EKS, K3s), Airflow, Celery, Kafka, SQS, Pub/Sub, Terraform, Argo CD, Pants, GitLab CI

**Languages:** Python (FastAPI, async SQLAlchemy), Go, SQL, TypeScript

**Data:** Hudi, Iceberg, Databricks (Spark SQL), DuckDB, BigQuery, Postgres

## Education

#### BA, International Relations, University of Calgary, Canada | 2018
