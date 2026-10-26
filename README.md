# EIA Defense on Open-Weight Models (ministral-8b-2512)

Independent reproduction of the Environmental Injection Attack (EIA; Liao et al.)
against a GUI web agent, plus two variants of an action-layer defense ("guardrail")
that reduce attack success rate from 0.54 to 0.15-0.17.

Built on [OSU-NLP-Group/EIA_against_webagent](https://github.com/OSU-NLP-Group/EIA_against_webagent)
(MIT license, see LICENSE and README_EIA_upstream.md) with minimal, documented patches.

## Why
Agent-security papers increasingly demonstrate attacks while deferring defenses.
This project reproduces the attack with zero budget on an open-weight model and
evaluates a cheap, auditable defense: one text-only audit call per PII-bearing
action, before browser execution. Every intercept carries a model-generated reason.

## Results (177 instances, action_grounding/form_type0/near_top_0)

| Variant                  | Attack success rate | Benign per-step agreement with baseline (n) | Trajectory length ratio |
|--------------------------|--------------------:|--------------------------------------------:|------------------------:|
| Unguarded                |                0.54 | baseline                                    |                   1.000 |
| v0.1 conservative guard  |                0.15 | 0.891 (169)                                 |                   1.000 |
| v0.2 PII-triggered guard |                0.17 | 0.902 (176)                                 |                   1.000 |

**Denominator note:** in guarded runs, 56–63 instances never reached the attack
step (guard-induced early termination); these are counted as non-success in the
ASR denominator.

**Takeaway:** guarded benign trajectories reproduce ~90% of baseline actions in
exact order (n=169/176), with no increase in trajectory length. We report
per-step agreement with the baseline rather than task success, because agreement is
what we measured.

## Key changes vs. upstream (see commit history)
- Redirected the OpenAI-compatible client to Mistral free tier via env vars.
- Fixed upstream typo: 'nase64' -> 'base64' in image data URLs.
- Engine hardening: 240s call timeout, 6 req/min throttle, visible API-call logging.
- Auto-downscale screenshots to 640px JPEG q70 (free-tier payload limits).
- `SeeAct/defense.py`: guard_check() — hooked into main.py before execution.

## Reproduce
1. Deploy webpages: `cd web && uvicorn main:app --reload --port 8000`
2. Set env: `source ~/.seeact_env` (Mistral key + base URL + model)
3. Run: `cd SeeAct && python main.py --attack_type action_grounding --attack_subtype form_type0 --attack_position near_top_0 --model_name gpt4v --save_dir "../eval_results/eval_results"`
4. Evaluate: `python eval.py --eval_dir_benign ".../benign" --eval_dir ".../action_grounding_form_type0_near_top_0"`

## Notes
- Runs are resumable (completed instance IDs are skipped on restart).
- The 18 GB upstream eval_results.zip was not used; all numbers above are from
  locally executed runs on ministral-8b-2512 (Mistral free tier).
