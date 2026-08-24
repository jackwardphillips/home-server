# Local personal-assistant feasibility findings

## Outcome

A read-only local assistant prototype was evaluated on Surface with Ollama and
`qwen3:4b-instruct-2507-q4_K_M`. The deployment was retired on 2026-08-23 after
testing showed that the available hardware and model were not a practical match
for the desired reasoning-oriented assistant.

## What worked

- A small FastAPI chat UI and Ollama ran reliably in bounded containers.
- Qwen could answer simple prompts and some narrowly constrained Observatory
  tool calls.
- Deterministic Python code could safely produce collector, trend, comparison,
  host-reachability, and HTTP-availability summaries.
- The design kept database credentials, SSH access, host mounts, and the Docker
  socket out of the assistant.

## Limiting findings

- The 4B model was inconsistent at selecting and parameterizing tools.
- Large tool schemas or report payloads could exceed the 120-second request
  timeout on CPU inference.
- Compact deterministic reports were fast, but did not provide the prioritization,
  causal reasoning, or follow-up investigation planning wanted from an assistant.
- Surface's NVIDIA GPU was using the `nouveau` driver and was unavailable to
  Ollama for CUDA acceleration.
- Six GiB of GPU memory could improve inference speed for a quantized 4B or
  marginal 7B/8B model, but was unlikely to provide the required increase in
  reasoning quality. Changing drivers on a service host was not justified by
  that limited expected benefit.
- Observatory presently contains environmental data, not personal-health data.
- Safe home-server checks could cover reachability and HTTP availability, while
  CPU, memory, disk, and container history required an authenticated read-only
  Beszel integration.

## Recommended future architecture

Keep raw data, deterministic calculations, anomaly detection, and all operational
access local. Send only a compact, sanitized evidence package to a stronger
reasoning model for prioritization and explanation. Infrastructure actions should
remain read-only by default and require explicit approval. Beszel is the preferred
source for future host-resource evidence; the Observatory API remains the source
for environmental evidence.
