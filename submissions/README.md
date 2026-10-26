# Model submissions

| Model | Edition | Decision Index | Complete results | Engine / commit | Hardware | Declared limits |
|---|---|---:|---|---|---|---|
| [StandardOne-35B-SH](https://huggingface.co/StandardThinking/StandardOne-35B-SH) | 0.3 | 66.26 | [scores.json](https://huggingface.co/datasets/StandardThinking/StandardOne-SH-decision-index/blob/4738e42c85439968274fb30fbed65c7287a1ed4d/runs/standardone-35b-sh-v1/scores.json) | `http` against the repo's `code/serve_head.py` @ `4e3b3a568f404b25ba9748d3933eb2c887214b2e` (our run: Standard One engine, ROCm build, bf16 backbone, fp32 head, context 131,072); kit `9eb2dbe` | 1 x AMD Instinct GPU, shared; serial latency separately measured on 1 x MI350X | 255 options, 10 score levels, 131,072 tokens; no truncation; 0 unsupported among scoreable |
