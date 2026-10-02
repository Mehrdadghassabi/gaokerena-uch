# Single-Pass Uncertainty Heads for Claim-Level Hallucination Detection in Persian Medical Language Models

Code, data, and notebooks accompanying the paper **"Single-Pass Uncertainty Heads for Claim-Level Hallucination Detection in Persian Medical Language Models"**.

We adapt the [LLM Uncertainty Head (LUH)](https://arxiv.org/abs/2505.08200) framework to Aya-Expanse-8B-based Persian medical models (**Gaokerena-V** and **Gaokerena-R**). Two lightweight claim-level heads are trained on frozen backbone attention maps and token probabilities, using claim-level hallucination datasets built **directly in Persian**. At inference time the heads need **no retrieval and no repeated sampling**: one forward pass yields a hallucination score for every extracted claim.

> **Note:** the test splits are small (50 responses each) and the labels are produced by an automatic annotator (DeepSeek-V4-Flash), not by humans. Please read the Limitations section before relying on these results.

---

## Highlights

- **Native Persian supervision.** 1,600 Persian medical questions answered by each backbone, with atomic claims extracted, labeled, and aligned to the Aya-Expanse tokenization (no translation step).
- **Paired datasets.** Both backbones answer the same questions in the same order (greedy decoding), so differences between the two heads come from the backbones, not the question set.
- **Open heads and data.** Both trained heads ([V](https://huggingface.co/gaokerena/gaokerena-V-uhead), [R](https://huggingface.co/gaokerena/gaokerena-R-uhead)) and both claim-level datasets ([LUH-V](https://huggingface.co/datasets/gaokerena/LUH_Gaokerena_V), [LUH-R](https://huggingface.co/datasets/gaokerena/LUH_Gaokerena_R)) are on Hugging Face.
- **Two trained heads.** Claim-level ROC-AUC of 0.7852 (Gaokerena-V) and 0.7810 (Gaokerena-R); PR-AUC of 2.30x and 2.66x the random baselines.
- **Reproducible training notes.** Includes LoRA adapter merging, tokenizer checks, validation/test isolation, and two fixes to the public LUH code (see below).

## Released resources

All released artifacts live under the [`gaokerena`](https://huggingface.co/gaokerena) Hugging Face organization.

**Trained uncertainty heads**

| Head | Backbone | Link |
|---|---|---|
| Gaokerena-V head | Gaokerena-V | [gaokerena/gaokerena-V-uhead](https://huggingface.co/gaokerena/gaokerena-V-uhead) |
| Gaokerena-R head | Gaokerena-R | [gaokerena/gaokerena-R-uhead](https://huggingface.co/gaokerena/gaokerena-R-uhead) |

**Claim-level hallucination datasets (native Persian)**

| Dataset | Backbone | Link |
|---|---|---|
| LUH-V | Gaokerena-V | [gaokerena/LUH_Gaokerena_V](https://huggingface.co/datasets/gaokerena/LUH_Gaokerena_V) |
| LUH-R | Gaokerena-R | [gaokerena/LUH_Gaokerena_R](https://huggingface.co/datasets/gaokerena/LUH_Gaokerena_R) |

**Backbones** (LoRA adapters on top of [Aya-Expanse-8B](https://huggingface.co/CohereLabs/aya-expanse-8b); the heads are trained on the merged models)

| Model | Link |
|---|---|
| Gaokerena-V | [gaokerena/gaokerena-v1.0](https://huggingface.co/gaokerena/gaokerena-v1.0) |
| Gaokerena-R | [gaokerena/gaokerena-r1.0](https://huggingface.co/gaokerena/gaokerena-r1.0) |

## Results

### Claim-level performance on the held-out test splits (%)

| Metric | Gaokerena-V | Gaokerena-R |
|---|---|---|
| Accuracy | 76.24 | 62.96 |
| Precision | 44.85 | 29.52 |
| Recall | 58.17 | 80.84 |
| F1 | 50.65 | 43.24 |
| ROC-AUC | 78.52 | 78.10 |
| PR-AUC | 48.20 | 46.52 |
| Random PR-AUC | 20.95 | 17.46 |
| PR-AUC / random | 2.30 | 2.66 |
| Test claims | 1,951 | 1,687 |
| Selected epoch | 3 | 4 |

Accuracy alone is not informative here (a classifier that never flags a claim reaches 79.05% and 82.54%). Compare PR-AUC against the random baseline, which equals the positive base rate.

### Response variability (motivation experiment)

Five chain-of-thought runs per model on the 168-question September 2023 Iranian Basic Medical Sciences Entrance Examination (IBMSEE).

| | Gaokerena-V | Gaokerena-R | Aya-Expanse | Med-Gemma |
|---|---|---|---|---|
| Accuracy (ensemble answer, %) | 29.76 | 38.69 | 35.71 | 37.50 |
| Questions with a single option taken | 14 | 37 | 36 | 168 |
| Questions with all options taken | 10 | 4 | 6 | 0 |
| Entropy (bits) | 1.11 | 0.80 | 0.84 | 0 |
| Parameters | 8B | 8B | 8B | 4B |

**Metric definitions** (identical to the paper):

- *Valid run:* a run from which an option letter (A-D) can be extracted. Runs such as `none` or `multiple choice` are invalid.
- *Ensemble answer / accuracy:* the option chosen by at least 3 of the 5 runs, otherwise `not_agreed`. Accuracy is the percentage of questions whose ensemble answer equals the correct option; `not_agreed` counts as incorrect.
- *Single option taken:* all five runs give the same valid letter (a question with an invalid run is not counted).
- *All options taken:* A, B, C, and D all appear among the valid answers.
- *Entropy:* per-question Shannon entropy in bits over the valid runs, averaged over the 168 questions. The maximum with five runs and four options is about 1.92 bits.

## Datasets

| | LUH-V | LUH-R |
|---|---|---|
| Annotated responses | 1,600 | 1,600 |
| Extracted claims | 60,434 | 54,209 |
| Supported claims | 48,991 | 43,305 |
| Hallucinated claims | 10,557 | 9,528 |
| Undecided claims | 886 | 1,376 |
| Hallucination rate | 17.73% | 18.03% |
| Supervised tokens | 61.4% | 54.9% |
| Positive class weight | 4.64 | 4.53 |

**Construction pipeline**

1. Curate 1,600 Persian medical questions deterministically (at most one question per base medical entity, synthetic entity variants capped at about 6%, even-interval selection over an alphabetically ordered entity pool).
2. Generate answers with Gaokerena-V and Gaokerena-R (greedy decoding, same questions, same order).
3. Extract atomic, verifiable claims from the Persian responses with Persian-language prompting; discard sentences shorter than 40 characters after markup removal.
4. Label each claim as *supported*, *hallucinated*, or *undecided* with DeepSeek-V4-Flash. Undecided claims are kept in the released data but excluded from the loss.
5. Map each supported/hallucinated claim to its token span in the Aya-Expanse tokenization. Tokens inside supervised claims get the binary label; all other tokens get the masking value `-100`.

**Splits:** 1,458 train / 92 validation (`eval`) / 50 test responses per dataset, sampled at even intervals over the ordered entity pool.

## Method

For each frozen backbone, the head receives:

1. the backbone's attention history from all layers (history size 3, no pooling at feature extraction), and
2. the top-4 token probabilities at each position.

These are concatenated and passed through a two-layer Transformer encoder (hidden size 768, 8 heads, dropout 0.1). Token-level signals are pooled per claim, producing one hallucination score per claim. Training uses weighted binary cross-entropy with positive-class weights 4.64 (V) and 4.53 (R).

### Training configuration (both heads)

| Setting | Value |
|---|---|
| Maximum epochs | 10 |
| Learning rate | 1e-4 |
| Warmup ratio | 0.05 |
| Weight decay | 0.1 |
| Per-device batch size | 8 |
| Gradient accumulation | 1 |
| Max gradient norm | 1.0 |
| Head dimension / layers / heads | 768 / 2 / 8 |
| Head dropout | 0.1 |
| Backbone precision | fp16 |
| Checkpoint metric | validation PR-AUC |
| Early stopping | 3 epochs |

No hyperparameter search was performed. Training used a single CUDA GPU with about 40 GB of memory at batch size 8. Backbone parameters stay frozen; only the head is updated.


### 1. Get the data and heads

```python
from datasets import load_dataset
from huggingface_hub import snapshot_download

luh_v = load_dataset("gaokerena/LUH_Gaokerena_V")   # claim-level data for Gaokerena-V
luh_r = load_dataset("gaokerena/LUH_Gaokerena_R")   # claim-level data for Gaokerena-R

head_v = snapshot_download("gaokerena/gaokerena-V-uhead")   # trained heads
head_r = snapshot_download("gaokerena/gaokerena-R-uhead")
```

### 2. Merge the Gaokerena LoRA adapters

Gaokerena-V and Gaokerena-R are distributed as LoRA adapters, and the public LUH trainer does not load PEFT adapters directly. Load Aya-Expanse-8B, apply the adapter with PEFT, merge it into the base model, and train the head against the resulting standalone model:

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

base_id = "CohereLabs/aya-expanse-8b"
adapter_id = "gaokerena/gaokerena-v1.0"   # or "gaokerena/gaokerena-r1.0"

base = AutoModelForCausalLM.from_pretrained(base_id, torch_dtype=torch.float16)
model = PeftModel.from_pretrained(base, adapter_id)
model = model.merge_and_unload()

model.save_pretrained("merged-gaokerena-v")          # standalone backbone for LUH training
AutoTokenizer.from_pretrained(base_id).save_pretrained("merged-gaokerena-v")
```

Before training, check tokenizer compatibility: compare the adapter and base tokenizers, and verify that chat-template tokenization reproduces the stored `input_ids` for the test split. The notebooks do this check.

### 3. Train a head

Run the training notebook for the backbone you want (`Gaokerena-V` or `Gaokerena-R`). Configure the 92-response `eval` split as the validation set; the 50-response `test` split must not be exposed to checkpoint selection.

### 5. Evaluate on the test split

Evaluation is a separate run: load the selected head and change the framework's validation argument to `test`. This keeps the final test metrics out of training and model selection. To evaluate the released heads, point this run at the checkpoints downloaded from `gaokerena-V-uhead` / `gaokerena-R-uhead`.

## Implementation corrections to the public LUH code

Two issues in the public LUH code were corrected before training:

1. **Chat-template output.** In recent Transformers versions, `apply_chat_template` returns a mapping rather than a plain list. The original code used its length as a token count, which can shift the reply boundary and place claim masks on prompt positions. We explicitly extract the `input_ids` sequence before computing the boundary.
2. **Precision resolution.** The original model loading did not resolve the configured PyTorch dtype, so a configuration requesting `fp16` could still load the 8B backbone in `fp32`. We use an explicit lookup of the requested torch dtype.

Both fixes affect preprocessing and model loading, not the head architecture. Without them, training runs can look valid while using incorrect supervision or unnecessary memory.

## Citation

```bibtex
@inproceedings{ghassabi2026uncertaintyheads,
  title     = {Single-Pass Uncertainty Heads for Claim-Level Hallucination Detection in Persian Medical Language Models},
  author    = {Ghassabi, Mehrdad and Rostami, Pedram and Kashani, Hamidreza Baradaran and Hakim, Sadra and Ebrahimi, Audrina},
  year      = {2026}
}
```

Please also cite the underlying works:

- LUH: Shelmanov et al., *A Head to Predict and a Head to Question: Pre-trained Uncertainty Quantification Heads for Hallucination Detection in LLM Outputs*, EMNLP 2025 (arXiv:2505.08200).
- Gaokerena-V: Ghassabi et al., *Leveraging Online Data to Enhance Medical Knowledge in a Small Persian Language Model* (arXiv:2505.16000); model: [gaokerena/gaokerena-v1.0](https://huggingface.co/gaokerena/gaokerena-v1.0).
- Gaokerena-R: Ghassabi et al., *Enhancing Reasoning Skills in Small Persian Medical Language Models Can Outperform Large-Scale Data Training* (arXiv:2510.20059); model: [gaokerena/gaokerena-r1.0](https://huggingface.co/gaokerena/gaokerena-r1.0).

## Authors

Mehrdad Ghassabi, Pedram Rostami, Hamidreza Baradaran Kashani, Sadra Hakim,Audrina Ebrahimi
