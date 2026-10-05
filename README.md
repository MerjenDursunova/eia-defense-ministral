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

## Results (action_grounding / form_type0 / near_top_0, 177 instances)

| Condition            | ASR  | ASR_pt | Intercepts (attack) | Intercepts (benign, 177) |
|----------------------|------|--------|---------------------|--------------------------|
| No defense           | 0.54 | 0.26   | —                   | —                        |
| + v0.1 conservative  | 0.15 | 0.09   | 124                 | 80                       |
| + v0.2 PII-triggered | 0.17 | 0.07   | 110                 | 66                       |

- v0.2 (audit only PII-bearing TYPE actions) retains 69% of the attack-success
  reduction while firing less often than the conservative variant.
- In guarded runs, 56-63 instances never reached the attack step (guard-induced
  early termination; counted as non-success in the ASR denominator).
- Benign blocks are recoverable re-plans: trajectories complete at baseline no-op
  rates (~5%); the guard's cost is extra steps/latency, not task failure.

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
