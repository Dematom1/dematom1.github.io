+++
title = "How I'm building it"
weight = 2
+++

This tab follows [training-harness](/projects/training-harness/) as I build it. **It is in progress: nothing has trained yet.** Below are the decisions made so far, why, and what each one gives up. As each step runs, what broke and what I changed gets added here, and the prompts I used go on the [prompts](../prompts/) tab.

## How the work is split

The training program (model, loop, DDP setup, data loader, checkpoints), the launcher and the Slurm config will be written by hand. That is the part the project exists to prove. Coding agents that I direct will write the rest: Terraform, data preparation, plotting and CI. I run them with [firstmate](https://github.com/kunchenguid/firstmate), which gives each agent one task in its own copy of the repository, and every change they make will pass [no-mistakes](https://github.com/kunchenguid/no-mistakes) (review, tests, lint, CI) before it is merged. The repository README will say which files are which.

## Decisions so far

### Slurm first, then Ray on Kubernetes, then JAX

The first plan ran on Kubernetes with Kueue and Kubeflow Trainer. They are good tools, but your posting names Kubernetes, Slurm and Ray, so I rebuilt the plan on those.

Slurm comes first because under Slurm I write the launcher and the recovery myself. Ray Train then supplies those same parts as a framework, on the same hardware, so the project can compare my requeue-and-resume with Ray's recovery after the same node kill. Doing Ray first would hand the hard parts to the framework before I had written them. Kubernetes comes last because it is the tool I already use most.

**Gives up:** Kueue's quota and priority scheduling. It stays an optional later step, queueing the Ray jobs.

### A small model from scratch, data parallel

A GPT-2-small-shaped model (about 124M parameters) on public text is the most reproduced small language-model setup there is, so anyone can check my loss curve against public baselines and see the training is real. It fits on one 24 GB GPU, which makes one GPU a fair baseline for the scaling table, and a failed run costs about a dollar.

Because the model fits on one GPU, plain data parallel (DDP) is the right strategy. Sharding it with FSDP would add machinery the model does not need.

**Gives up:** the alternative was fully fine-tuning an open 1-2B model with FSDP across nodes. That is closer to "distributed training strategies", but it costs several times more per run and gives coarser scaling steps. One strategy on a small model is the honest limit of this project.

### GCP spot L4s, and spot on purpose

Two `g2-standard-48` nodes with 4 NVIDIA L4 GPUs each, 8 GPUs in total, at about US$0.60 per GPU-hour on spot (GCP's price page, 2026-11-10). Spot matters beyond price: preemption is a real failure to recover from, not one I stage.

What I ruled out:

| Option | Why not |
|---|---|
| Lambda VMs | Full root, but no spot, so every node loss would be self-inflicted |
| RunPod pods | Containers with no kubelet, so no Kubernetes; single node only |
| Vast.ai | Cheapest raw GPUs, but separate hosts share no private network, so multi-node numbers would not be fair |
| Modal | Serverless; it hides the scheduler and the node, which are the things being shown |

**Gives up:** spot capacity may not be there when I want it; a new GCP project needs a GPU quota increase before the first GPU VM starts; and L4s over ordinary cloud networking are far from a training cluster's interconnect.

### Checkpoints that a torn upload cannot corrupt

Every K steps, rank 0 will save weights, optimizer, scaler, step, data cursor and RNG state, upload them to Cloud Storage, and write the manifest last. Resume will read only the manifest, so a node that dies mid-upload leaves the previous checkpoint in force. The first step will check this on a laptop CPU: a killed and resumed run must reproduce the loss curve of an uninterrupted one. Without that check, a recovery number only shows that something restarted.

**Gives up:** up to K steps of work on every kill. Measuring that loss against the cost of checkpointing more often is one of the results.

### Backfill or strict FIFO

My scheduler [dibs](https://github.com/Dematom1/berth) never admits a job ahead of the head of its queue, so a large job cannot be starved by small ones. Slurm's default scheduler backfills instead: it starts smaller jobs early when that will not delay the job at the head. On my own two nodes I will submit an 8-GPU job behind a running 4-GPU job and some short 1-GPU jobs, run it under both scheduler types, and record what each does to the large job's start time and to GPU utilisation, including when the jobs' time limits are wrong.

### A real job for the model

The model is mostly the load on the system: the result is how it trains, scales and recovers. To give it a use as well, it will be fine-tuned on postings from [Data First Jobs](https://datafirstjobs.com), my job board, to score listing quality as a filter in the board's ingestion pipeline.

**Gives up:** a 124M model's judgement is limited. The filter will be reported as the share of postings it flags and how many of those were really junk, not as a quality claim.

## What comes next

The [project page](/projects/training-harness/) has the steps in order and their status. First comes the training loop, written by hand and tested on CPU, then one GPU, then Slurm across two nodes.
