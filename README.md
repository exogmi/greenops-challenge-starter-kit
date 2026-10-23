# GreenOps Challenge starter

Build a program that chooses when and where cloud workloads run, and reduce their
electricity emissions. **Your code runs on your computer and controls our simulator through
APIs. The website shows its progress and results.**

Start with this demo: it schedules one CPU workload and one GPU workload. Then
extend it to schedule all 15 workloads within 120 simulated hours.

## 1. Download your credentials

Sign in at [greenops.containerstack.net](https://greenops.containerstack.net),
create or join a team, then click **New simulation run**.

Open that run’s **API credentials → Download CLI config** to download
`greenops-exo-<run-id>.json`. The ID is the first 10 characters of your run.
Python, Go and Terraform examples all use this file.
Keep it private; it contains your run’s access keys.

## 2. Get the starter

You need Git and Docker installed on your computer.

```sh
git clone https://github.com/exogmi/greenops-challenge-starter-kit.git
cd greenops-challenge-starter-kit
```

Move the downloaded JSON file into this folder and rename it to
`greenops-exo.json`. It is ignored by Git.

## 3. Open the development environment

Run this in the starter folder:

```sh
docker run --rm -it -v "$PWD:/workspace" \
  ghcr.io/exogmi/greenops-challenge-starter-kit:latest
```

This opens a terminal inside a container with Python, Go and Terraform ready to use.
Your checkout is mounted at `/workspace`; edits on your computer appear there.

## 4. Run the demo

Inside the container, run:

```sh
python examples/python/static.py run --config greenops-exo.json
```

Keep that terminal open and open your run’s dashboard in the website. You should
see one CPU VM and one GPU VM, followed by workload progress. The clock starts
when the first VM is created. The example deletes its VMs after their outputs
are delivered. It schedules **two of the 15 workloads**.

Prefer another tool? Use the same setup and choose the
[Go or Terraform example](docs/EXAMPLES.md) on a fresh run.

## 5. Build your solution

Start by editing `examples/python/static.py` or `examples/go-static/`. The demo’s
fixed placements are in `examples/static-plan.json`; all 15 workload requirements
are in `challenge/workloads.json`.

First schedule all workloads and meet their deadlines. Then use electricity
forecasts to improve when and where they run. Read the
[cloud, work-status and electricity API reference](docs/API.md) as you need it.

For each new attempt, create a fresh run and download its credentials. Python and
Terraform state belongs to one run: use a fresh checkout or
[separate receipt/state files](docs/DEVELOPMENT.md#start-another-attempt).

Use only observations and forecasts published by the run you control, and respect
each workload’s allowed locations. Read the [competition rules](docs/RULES.md)
before building your optimizer. Submit your GitHub repository and reproducible
run instructions through the website when you have a result to submit.

[Native installation and troubleshooting](docs/DEVELOPMENT.md) ·
[Verification](docs/VERIFICATION.md) · [Third-party software](docs/THIRD_PARTY.md)
