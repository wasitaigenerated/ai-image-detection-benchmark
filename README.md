# AI Image Detection Benchmark

An open benchmark for **AI-generated image detectors**: 1,269 images from 19 current image generators
(GPT-Image-2, Nano Banana Pro, FLUX.3, Seedream 5, Ideogram, Grok Imagine, Qwen Image and more) and free-licensed real photos, each also as a
WhatsApp, Instagram and re-shared copy, because that is how images reach moderation and investigation teams.
Run by [WasItAIGenerated](https://www.wasitaigenerated.com).

- Write-up with full tables: https://www.wasitaigenerated.com/research/ai-image-video-detector-benchmark-2026
- Public split on Hugging Face: https://huggingface.co/datasets/wasitaigeneratedcom/frontier-image-video-bench-2026-10

## Headline (hidden split, October 2026)

Files are assigned by hash: 60% public, 40% hidden. These numbers come from the hidden 40%: 208 AI and
314 real images. "Detected @ 1% FP" is the share of AI images caught when the threshold is set so that 1%
of real images are flagged.

| Detector | Detected @ 1% FP, originals | Detected @ 1% FP, social copies |
|---|---:|---:|
| WasItAI V3 | 99.5 % | 96.2 % |
| ZeroGPT (TruthScan engine) | 97.2 % | not run |
| Best open-weights detector | 48.1 % | 37.3 % |

**Caveat:** we built this set and our models have seen similar content in training, so read our numbers
as a home-field result. That is why the public split exists.

## Data

Real photos are under free licenses (CC0, public domain, CC BY, CC BY-SA), with author and source for every
file. Metadata (EXIF, XMP, C2PA) was stripped so a detector cannot read the answer from the file header;
pixels are unchanged.

```
images/<id>.<ext>                                     originals
images_social/{whatsapp,instagram,reshared}/<id>.jpg  social-media copies
metadata.csv   file, id, media, label (ai/real), variant, generator, license, author, source_url
```

| | Public (60%) | Hidden (40%) |
|---|---:|---:|
| AI images | 261 | 208 |
| Real images | 486 | 314 |

## Evaluate your own detector

```bash
pip install requests numpy
cd 2026-10
python eval.py --adapter my_detector.py:predict
```

```python
# my_detector.py
from PIL import Image

model = load_my_model()

def predict(path: str) -> float:
    return float(model(Image.open(path).convert("RGB")))
```

Write a function that takes a file path and returns the probability that the file is AI-generated
(0 to 1), then point `eval.py` at it. If you already have scores, pass a CSV with columns `id,score`
(add a `variant` column for the copies; without it every row counts as an original).

Scores are appended to `scores_<adapter>.csv` one file at a time, so a run can be stopped and restarted.
Other options: `--copies original|social|all`, `--workers N`, `--threshold T` (also report rates at a fixed
score), `--revision` (pin a dataset commit).

The script reports:

- **AUROC**: how well the scores separate AI from real files across all thresholds.
- **Detected @ 1% FP**: the threshold is set on the real files of the same row so that at most 1% of them
  are above it; the number is the share of AI files above it.

To run the WasItAIGenerated API on the same files (uses your API credits; the API takes uploads up to 4 MB,
larger files are skipped and reported as unscored):

```bash
WAI_API_KEY=wai_... python eval.py --adapter wai
```

## Hidden split integrity

The hidden 40% stays private so nobody can tune a detector on it. To show it does not change after the
fact, [`2026-10/hidden-split-sha256.txt`](2026-10/hidden-split-sha256.txt) lists the SHA-256 of every hidden
file (2,079 images, originals and copies), sorted. The SHA-256 of that list is

```
d4fca0f0d754f493cad7aebe25804f485bc7d917d88f9b28903665712ff44ec2
```

If we ever add, remove or swap a hidden file, this hash changes.

## Submit your detector

Run your detector on the public split and email **patrick@wasitaigenerated.com** with your results and a way
for us to run it (weights, code or an API). We score it on the hidden split and list it on the write-up
page, including results that beat ours.

## Related benchmarks

- [ai-text-detection-benchmark](https://github.com/wasitaigenerated/ai-text-detection-benchmark): AI text detectors (GPTZero, Pangram, ZeroGPT, tropa)
- [ai-video-detection-benchmark](https://github.com/wasitaigenerated/ai-video-detection-benchmark): AI video detectors, incl. messenger copies
- Data on Hugging Face: [text, September 2026](https://huggingface.co/datasets/wasitaigeneratedcom/ai-detector-benchmark-2026-09) · [images and video, October 2026](https://huggingface.co/datasets/wasitaigeneratedcom/frontier-image-video-bench-2026-10)
- Want to audit a detector, ours included? [Call for benchmarks](https://www.wasitaigenerated.com/call-for-benchmarks) (free API credits for researchers)

## License

AI-generated files: CC BY 4.0 (WasItAIGenerated). Real files keep their original license, listed per
file in `metadata.csv` with author and source; CC BY and CC BY-SA files require attribution to the listed
author when reused. Code (`eval.py`): MIT (see `LICENSE`). Results and text in this repository: CC BY 4.0 (see `LICENSE-DATA`).
