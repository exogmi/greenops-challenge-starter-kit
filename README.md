# Follow the Green Energy — participant starter

Exoscale × [The Energy Bridge Hackathon](https://www.theenergybridge.eu/events/teb-critical-infrastructure-hackathon-2026). Build a **reactive** orchestrator for a simulated European cloud: finish CPU/GPU work within 120 simulated hours while reducing VM operational carbon impact.

**[Read the competition rules](docs/RULES.md) before building an optimizer.** Unlimited attempts, two unfinished runs per team. The winning orchestrator must be submitted as a GitHub repository with an exact commit and reproducible command. Electricity decisions may use only the current run's published observations/forecasts. Harvesting future values from other runs or external historical datasets is prohibited.

Source: [exogmi/greenops-challenge-starter-kit](https://github.com/exogmi/greenops-challenge-starter-kit). Toolchain image: `ghcr.io/exogmi/greenops-challenge-starter-kit:latest`. Both architecture builds and the anonymous pull/runtime check passed on GitHub Actions; see [verification evidence and immutable digest](docs/VERIFICATION.md).

## What's here

| Path | Purpose |
|---|---|
| `Dockerfile` | Reusable development image: exo 1.95.6, Terraform 1.13.4, Go, Python, official bindings, curl/jq/git and examples |
| `examples/go-static/` | Go/egoscale v3.1.40: fixed CPU/GPU deployment, list and destroy |
| `examples/python/static.py` | Official Python `exoscale` 0.16.3: same fixed deployment, local receipts and cleanup |
| `examples/python/grid.py` | Read a forecast from the matching run's grid endpoint |
| `examples/terraform/` | VM-only static baseline via Terraform's built-in resource and the Python binding |
| `examples/static-plan.json` | Fixed legal CPU and GPU placements; no impact-dependent scheduling |
| `challenge/` | Public catalog, 15 workloads, matching schemas and asset hashes |
| `docs/API.md` | Supported cloud/electricity routes, auth, limits, fictional samples and official links |
| `scripts/export_public.py` | Organizer-only public catalog exporter; never reads a private scenario file |
| `.github/workflows/ci.yml` | Tests and multi-architecture GHCR build/publish |

The examples allocate **two workloads**, not the complete 15-job competition pack. They establish a simple deployment-impact baseline and show API mechanics. They do not claim full-pack completion or optimized scheduling. Use a **fresh run for each example**. The examples share deterministic VM names and must not be run concurrently against the same run. Repeating the same deployment recovers existing names instead of intentionally adding workers. Preserve the Python/Terraform receipt when cleaning up.

## 1. Get a run

Sign in to the participant portal with GitHub, create/join a team and create a run. In **API credentials**, select the run and download `greenops-exo.json`. Every teammate can retrieve the same saved credentials later; reading does not rotate keys. Save the file in this repository's root (gitignored). Explicit rotation and team departure revoke prior keys.

Check that your run uses the scenario ID in `challenge/manifest.json`. The public catalog is also available from the portal's `/api/catalog`. Keep the run configuration outside Git. No real Exoscale account or cloud key is required. The first accepted VM starts time, even if its workload has a later release time; boot/setup/work/output progress belongs to the simulator.

The current workload pack contains **14,600 CPU vCPU-hours** plus four GPU jobs,
with room for carbon-aware timing and pauses. Read exact per-job requirements
from `challenge/workloads.json` and use a run matching its scenario identity.

## 2. Use the toolchain image

Clone the starter and use the published toolchain:

```sh
git clone https://github.com/exogmi/greenops-challenge-starter-kit.git
cd greenops-challenge-starter-kit
docker pull ghcr.io/exogmi/greenops-challenge-starter-kit:latest
docker run --rm -it -v "$PWD:/workspace" ghcr.io/exogmi/greenops-challenge-starter-kit:latest
```

Or build locally:

```sh
make image
docker run --rm -it -v "$PWD:/workspace" greenops-toolchain:local
```

Inside the container, tools and bindings are ready:

```sh
exo version
exo -C greenops-exo.json -O json zone
python /opt/greenops-starter/examples/python/static.py deploy --config /workspace/greenops-exo.json --state /workspace/.local/python-receipt.json
python /opt/greenops-starter/examples/python/static.py list --config /workspace/greenops-exo.json
python /opt/greenops-starter/examples/python/static.py destroy --config /workspace/greenops-exo.json --state /workspace/.local/python-receipt.json
```

To edit code, the mounted checkout is under `/workspace`; use its examples instead of the baked copies under `/opt/greenops-starter`. For Go:

```sh
go run ./examples/go-static -config greenops-exo.json -action deploy
go run ./examples/go-static -config greenops-exo.json -action list
go run ./examples/go-static -config greenops-exo.json -action destroy
```

The baked Go command is also available as `greenops-static`; supply `-plan /opt/greenops-starter/examples/static-plan.json -catalog /opt/greenops-starter/challenge/catalog.json` when running outside the checkout.

A run's URL must be reachable from the container. `127.0.0.1` in a downloaded URL means the container itself, not the host. For a local portal with loopback URLs, run the native examples below, or have the organizer configure a container-reachable public origin before creating the test run. Do not rewrite only the discovery URL: the discovered zone endpoints must use that same origin and run prefix. Hosted challenge URLs need no such adjustment.

A participant Dockerfile can start with `FROM ghcr.io/exogmi/greenops-challenge-starter-kit:latest`. Pin the verified release/commit digest for the winning submission; `latest` can change as the starter is improved.

## 3. Native Go or Python

Go 1.25+ is required by the pinned Go binding. From this checkout:

```sh
go run ./examples/go-static -config greenops-exo.json -action deploy
```

Python 3.10+:

```sh
python3 -m venv .venv
.venv/bin/pip install --require-hashes -r requirements.txt
.venv/bin/python examples/python/static.py deploy --config greenops-exo.json
.venv/bin/python examples/python/static.py list --config greenops-exo.json
.venv/bin/python examples/python/static.py destroy --config greenops-exo.json
```

These clients discover zone URLs, look up type/template IDs, send the workload binding and stop after admission. They do not wait for work completion or automatically remove idle VMs. Inspect progress/impact in the portal; destroy later to stop idle costs. The Python receipt is written after each admitted/recovered VM, including partial deployments. If a response is lost, inspect the run before retrying; the cloud API does not promise durable idempotent create receipts.

The Python grid example requires the matching run's grid URL and secret (download/copy them from the portal):

```sh
export GREENOPS_CONFIG="$PWD/greenops-exo.json"
export GREENOPS_GRID_ENDPOINT='COPY_RUN_GRID_ENDPOINT'
export GREENOPS_GRID_SECRET='COPY_RUN_GRID_SECRET'
.venv/bin/python examples/python/grid.py
```

For a container, pass those exported values with `-e GREENOPS_GRID_ENDPOINT -e GREENOPS_GRID_SECRET`, and set the config path inside the mount. Grid reads do not advance the run clock. Use only this run's currently published data when extending the orchestrator.

## 4. Terraform baseline

In the container, Terraform and the Python binding are already installed:

```sh
terraform -chdir=examples/terraform init
terraform -chdir=examples/terraform apply -var="config_path=$PWD/greenops-exo.json"
terraform -chdir=examples/terraform destroy -var="config_path=$PWD/greenops-exo.json"
```

For native use, install Terraform **1.13.4**, then include `-var="python=$PWD/.venv/bin/python"` in apply/destroy. The config path is a path, never a secret value in HCL. Keep state and `.local/terraform-receipt.json` until destroy succeeds.

This example uses **`terraform_data` plus the official Python SDK**. It does not use or claim support for the Exoscale Terraform provider, whose simulator endpoint routing remains unverified. It has create/destroy hooks, not full native provider refresh/import/drift reconciliation. If its receipt is lost, cleanup fails visibly: list and remove the VMs explicitly. Changing credentials in the same config file is sufficient after a deliberate rotation; use a separate config/state/receipt for another run.

## Development and publication

```sh
PATH="$PWD/.venv/bin:$PATH" make test
```

For a live lifecycle check, create a fresh empty run and use:

```sh
.venv/bin/python scripts/live_smoke.py --config greenops-exo.json --language python
```

Choose `--language go` or `terraform` on a different fresh run; `--terraform` can point to a pinned binary. The check creates the two VMs, verifies repeat deployment preserves their IDs, then destroys them and checks empty inventories. It intentionally does not claim workload completion.

CI runs Go/Python tests and builds the container for Linux amd64 and arm64. CLI and Terraform archives are checksum pinned; Python dependencies are hash locked; Go dependencies use `go.sum`; base image manifests are digest pinned. Upstream tools keep their licenses; see [third-party notes](docs/THIRD_PARTY.md).

The workflow targets `exogmi/greenops-challenge-starter-kit`. The workflow uses `GITHUB_TOKEN` with package-write permission and publishes `latest` on the default branch, commit tags and release tags. Make the GHCR package public if anonymous participant pulls are required. Run the `Verify anonymous image pull` workflow to pull with an empty Docker credential config and execute CLI, Terraform, Go and Python checks. Verify this workflow before announcing anonymous image availability. Anonymous pull and tool checks passed for the initial image; exact evidence and digest are in the verification document. Rebuild or edit the examples by deriving an image or mounting the checkout; do not bake run credentials or electricity histories into it.

Public assets come from the same platform public projection, with hashes in `challenge/manifest.json`. Organizer refresh:

```sh
python3 scripts/export_public.py --portal https://YOUR_CHALLENGE_ORIGIN
```

The origin above is a placeholder. This exports only public workload/resource metadata. Never add the platform's private `runtime.json`, actuals, complete forecast vintages or balancing/oracle reports.

To deploy, poll for successful final outputs and delete this example's completed
VMs, use `greenops-static -action run` or `python3 examples/python/static.py run`.
See [work telemetry](docs/API.md#batched-work-status-and-explicit-cleanup) and
[frozen ranking and official judging](docs/RULES.md#frozen-v1-ranking-and-official-judging).
