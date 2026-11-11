+++
title = 'training-harness'
date = 2026-11-11
draft = false
+++

**Status: in progress.** The [repository](https://github.com/Dematom1/training-harness) is public. No training run has happened yet; this page says what is planned, and each step below moves to "done" only after it has run, with its numbers and logs.

## What it is

A distributed training stack built as infrastructure. It will train a hand-written, GPT-2-small-shaped language model (about 124M parameters) in PyTorch:

1. **Pretrain** on a subset of [FineWeb-Edu](https://huggingface.co/datasets/HuggingFaceFW/fineweb-edu), public educational web text.
2. **Fine-tune** on job postings from [Data First Jobs](https://datafirstjobs.com), my job board, so the model can score listing quality as a filter in the board's ingestion pipeline.

The training loop, the launcher and the checkpoint logic will be written by hand. Coding agents may write the surrounding Terraform, data preparation and plotting, and the repository README says which files are which.

## Why

My work so far is the infrastructure models run on: Kubernetes batch systems, an MLOps stack, the operations side of production models. I had not run distributed GPU training myself. This project closes that gap at small scale and in the open.

The model is mostly the load on the system. What matters is how fast it trains, how it scales as GPUs are added, what was slow and how I found it, and what happens when a machine dies mid-run. The listing filter gives the trained model a real job on a system I already run.

## Architecture

One hand-written training program (model, loop, DDP setup, data loader, checkpoints) that knows nothing about its scheduler, launched three ways in turn. Every run reads token shards from and writes checkpoints to the same Cloud Storage bucket, and every rank logs one JSON line every N steps.

- **Slurm (steps 2 and 3).** A small controller VM and two spot GPU nodes with 4 NVIDIA L4 GPUs each, 8 GPUs in total. A launcher renders and submits the `sbatch` script; `srun` starts one rank per GPU; DDP across both nodes. A checkpoint (weights, optimizer, step, data cursor, RNG state) is uploaded every K steps and its manifest is written last, so a torn upload is never picked up. When a spot node is preempted, Slurm requeues the job and it resumes from the last checkpoint.
- **Ray on Kubernetes (step 4).** The same loop as a Ray Train job on KubeRay, on a GKE spot GPU node pool of the same machines. This compares throughput under `srun` with Ray Train on identical hardware, and my own requeue-and-resume with Ray's recovery after the same node kill.
- **JAX (step 5).** The model and training step ported to JAX, with throughput and compile time beside the PyTorch version.

## Progress

| Step | What it proves | Status |
|---|---|---|
| Repository public | - | Done (README only) |
| 0. Training loop on CPU, with a resumed run that matches an uninterrupted one | Correct checkpoint and resume | Not started |
| 1. One GPU, then 2 and 4 GPUs on one node | PyTorch data-parallel training | Not started |
| 2. Slurm across 2 nodes, 8 GPUs | Multi-node training, scaling, profiling | Not started |
| 3. Kill nodes mid-run | Recovery from preemption | Not started |
| 4. Ray on Kubernetes | Same loop under Ray Train | Not started |
| 5. JAX port | Same model in JAX | Not started |
| Fine-tune on Data First Jobs postings | The listing-quality filter | Not started |

## Results

Nothing has run yet, so there are no numbers. Each result below will be added with its raw logs as its step completes.

- **Throughput and scaling efficiency** at 1, 2, 4 and 8 GPUs (tokens per second, median of three runs).
- **One profiled bottleneck**, before and after the fix.
- **Work lost and time to recover** when a node is killed between checkpoints.
- **Cost per run**, reconciled against the GCP bill.
- **The listing filter**: the share of postings it flags, and how many of those were really junk.

## What it will not show

A 124M-parameter model on 8 small GPUs is small-scale training. It uses one strategy (data parallel) on a model that fits on one GPU, over ordinary cloud networking. It shows that I can build, run, measure and repair a distributed training job end to end; it is not experience training a large model at scale.
