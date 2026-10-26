# Standard One 35B SH (v1)

[StandardThinking/StandardOne-35B-SH](https://huggingface.co/StandardThinking/StandardOne-35B-SH) (tag `v1`, commit `4e3b3a568f404b25ba9748d3933eb2c887214b2e`) is Qwen3.6-35B-A3B (a mixture-of-experts model with about 3B active parameters per token) fine-tuned with LoRA and merged into bfloat16 weights, plus a float32 joint schema head (68.5M parameters) that reads the final layer and layer 29. Apache-2.0.

- It reads a request once and scores every option of every question from the hidden states in one prefill; nothing is generated.
- Prompt format: the Qwen chat template with thinking disabled and no system message (the repo default).

| | |
|---|---|
| Decision Index 0.3 public index | 66.26 (raw 74.68) |
| Requests | 140,178 of 140,178 scoreable, all answered |
| Request time in `scores.json` | median 559.9 ms, mean 1185.4 ms, p95 2516.8 ms; several client shards in parallel on a shared AMD GPU, so it includes queueing |
| Sequential request time | median 365 ms, mean 415 ms, p80 456 ms, p95 646 ms over 120 public requests, one at a time, with the reference server (`code/serve_head.py`, `--max-batch-requests 1`) on one AMD Instinct MI350X; not measured on an RTX PRO 6000 |
| Results | [`StandardThinking/StandardOne-SH-decision-index`](https://huggingface.co/datasets/StandardThinking/StandardOne-SH-decision-index/tree/main/runs/standardone-35b-sh-v1) `runs/standardone-35b-sh-v1` |

Areas (skill x100, coverage 100 % in every area): Knowledge & Reasoning 52.15, Language Understanding 66.03, Retrieval & Classification 72.49, Tools & Automation 86.09, Arts & Human Taste 54.61.

## Run

The model repository ships a plain PyTorch reference server (`code/serve_head.py`) that answers the kit's `http` engine directly; no proxy is needed.

```sh
hf download StandardThinking/StandardOne-35B-SH --revision v1 --local-dir StandardOne-35B-SH
python -m pip install -r StandardOne-35B-SH/code/requirements.txt
python StandardOne-35B-SH/code/serve_head.py --snapshot StandardOne-35B-SH --head StandardOne-35B-SH/head --port 30171
python -m decision_index pipeline --engine http --option base_url=http://127.0.0.1:30171 \
  --option model=standardthinking/standard-one-35b-sh --out runs/standardone-35b-sh-v1
```

It needs one GPU with at least 96 GB of memory. `--max-batch-requests` and `--max-batch-tokens` control batching.

Our run used the ROCm build of the Standard One engine (our SGLang fork with the schema-head readout, image `st-sglang:decision-qwen35-776e539961-60dac5b`) at a 131,072-token context and the kit at `9eb2dbe`. Its Qwen3.5/3.6 support is not yet in the public engine repository, so we checked the released files with the reference server before publishing: JevBench and our long-document set gave identical accuracy to the engine run, our internal held-out set differed by one item in 600, and a kit spot check of 15 Decision Index 0.3 rows (18 questions) gave the same answers. The results README describes the run.

## Settings and limits

- bfloat16 backbone, float32 head, no system message, thinking disabled.
- Context 131,072 tokens for this run; the model was trained on inputs up to 32,768 tokens.
- Declared limits: 2-255 options per choice, 2-10 score levels.

## Training data

Qwen3.6-35B-A3B fine-tuned in several rounds of head and LoRA training on decision data, each LoRA merged into the weights. We do not publish further details of the training data.
