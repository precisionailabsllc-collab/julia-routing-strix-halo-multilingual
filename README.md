---
license: apache-2.0
base_model: SupersonicLabs/Julia-1
library_name: julia
pipeline_tag: text-classification
tags:
  - julia
  - strix-halo
  - multilingual
  - bounded-decisions
  - routing
  - cpu
  - gpu
---

# Julia Routing Strix Halo Multilingual Experimental

Experimental multilingual fine-tune of Julia-1 for fast bounded-decision routing on AMD Strix Halo-class local machines.

Runtime status:

- CPU: tested
- GPU: expected to work through the Julia/PyTorch runtime, not separately benchmarked for this release
- NPU: not validated yet; AMD/XDNA NPU packaging and execution are future work

This is an **experimental v0.2 multilingual candidate**, not a replacement for the English-focused v0.1 checkpoint.

## Intended use

Good candidates:

- task routing over fixed options
- eval-case routing: category and deterministic grader type
- documentation-update triage
- context-management metadata
- non-authoritative tool-policy hints

Do **not** use this model as the final authority for destructive commands, credential handling, production deploys, security replay safety, durable memory writes, summarization, or final prose.

## Training summary

- Base model: `SupersonicLabs/Julia-1`
- Starting checkpoint: `julia-routing-strix-halo` v0.1 tuned routing model
- Fine-tuning method: native partial fine-tune, no LoRA
- Trainable parameters: `7,634,563`
- Unfrozen modules: decision head plus last 2 encoder layers
- Learning rate: `1e-5`
- Epochs: `1`
- Languages: Spanish, French, German, Portuguese, Chinese, Japanese, Arabic
- Labels remain canonical English option IDs

## Local eval summary

| Eval | Score | Accuracy |
|---|---:|---:|
| Multilingual holdout small | `187/224` | `83.5%` |
| Multilingual holdout large | `574/672` | `85.4%` |
| English holdout | `98/126` | `77.8%` |

By language on the large multilingual holdout:

| Language | Accuracy |
|---|---:|
| Portuguese | `93.8%` |
| French | `90.6%` |
| Spanish | `88.5%` |
| German | `85.4%` |
| Japanese | `82.3%` |
| Chinese | `79.2%` |
| Arabic | `78.1%` |

All latency numbers in development were measured on CPU. NPU acceleration has not been validated.

## Usage

Install the Julia runtime from the base model package, then load this checkpoint:

```python
from julia import load_model

engine = load_model(
    "./julia-routing-strix-halo-multilingual",
    device="cpu",
    strict_encoding=True,
    max_length=1024,
    head_length=512,
)

result = engine.predict(
    state={"language": "es", "request": "Corrige la prueba TypeScript que falla y ejecuta la suite."},
    questions={
        "route": {
            "type": "choice",
            "instructions": "Which worker route should handle this request?",
            "criteria": {
                "code_edit": "modify repository files",
                "browser_qa": "interact with a web UI",
                "research": "gather information only",
                "chat": "answer conversationally",
            },
        }
    },
)
print(result)
```

## Files

- `model.safetensors` — fine-tuned Julia decision weights
- `julia_config.json` — Julia runtime configuration
- `encoder/config.json` — encoder configuration
- `tokenizer/` — tokenizer files

## Limitations

- Experimental multilingual candidate.
- Specialized for bounded decisions and fixed option sets.
- Not a general-purpose embedding model.
- Not a generative model.
- Can be confidently wrong outside the routing distributions represented in training.
- NPU execution has not been validated yet.
