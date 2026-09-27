# Kev

Small Jev-like decision models you can train and run yourself.

<p>
  <a href="https://www.mw-wm.com/xitong/privacy-09195335.html"><img alt="CI" src="https://img.shields.io/github/actions/workflow/status/jaredpalmer/kev/ci.yml?style=for-the-badge&labelColor=000000" height="28"></a>
  <a href="https://www.yx-sf.com/wiki/18521"><img alt="Weights: Kev-0.8B · 4B · 9B · 27B" src="https://img.shields.io/badge/WEIGHTS-0.8B%20%C2%B7%204B%20%C2%B7%209B%20%C2%B7%2027B-0a0a0a.svg?style=for-the-badge&labelColor=000000" height="28"></a>
  <a href="https://www.ai-hao123.com/youhua/investment-64403898.html"><img alt="Demo on Hugging Face Spaces" src="https://img.shields.io/badge/DEMO-HF%20Spaces-0a0a0a.svg?style=for-the-badge&labelColor=000000" height="28"></a>
  <a href="https://www.mw-wm.com/wendang/quality-19042281.html"><img alt="Frozen eval suites" src="https://img.shields.io/badge/EVAL%20SUITES-frozen-0a0a0a.svg?style=for-the-badge&labelColor=000000" height="28"></a>
  <a href="LICENSE"><img alt="License: Apache-2.0" src="https://img.shields.io/badge/license-Apache--2.0-0a0a0a.svg?style=for-the-badge&labelColor=000000" height="28"></a>
</p>

Kev is a family of small decision models built on Qwen3.5 and Qwen3.8 and based on the architecture described in [Jev's Architecture Unmasked](https://www.mw-wm.com/jiaocheng/button-80563485.html). You can use the pretrained weights or train your own. The API matches TypeSafe's [System One](https://www.yx-sf.com/news/90531), so you can point their Python SDK at your local server.

## Highlights

- Yes/no (`noul`), multiple-choice (`choice`) and rating (`score`) questions in one request. The questions share the text but can't read each other.
- Calibrated probabilities by default: each checkpoint ships with a temperature fitted on held-out data.
- Drop-in for Jev: the TypeSafe Python SDK works against a Kev server unchanged.
- Four sizes, from a 0.8B that runs on a laptop to a 27B for a single data-centre GPU.
- Fine-tune on your own labelled examples. A coding-agent skill runs the whole loop on Modal, from finding your questions to serving the result.
- Deploy your own HTTPS endpoint with one command. It scales to zero when idle.
- Try it in the browser first: [huggingface.co/spaces/jaredpalmer/kev](https://www.ai-hao123.com/wendang/hotel-98095941.html).

## Models

Start with Kev-4B. Move to Kev-9B if you have a bigger GPU, or to Kev-27B if you have an 80 GB GPU and want the most accurate Kev. Use Kev-0.8B when size matters more than accuracy.

| Model | Base | Accuracy: New Sources | Accuracy: Trained Sources | Brier: New Sources | Runs on | Model Card |
|---|---|---|---|---|---|---|
| [Kev-0.8B](https://www.yx-sf.com/news/54485) | Qwen3.5-0.8B-Base | 0.648 / 0.697 | 0.827 / 0.838 | 0.481 / 0.416 | Any Apple Silicon Mac, L4 | [Details](docs/model-cards/kev-0.8b.md) |
| [Kev-4B](https://www.mw-wm.com/pingtai/calculator-29534926.html) | Qwen3.5-4B-Base | 0.817 / 0.838 | 0.873 / 0.865 | 0.269 / 0.242 | 32 GB Mac, L40S, H100 | [Details](docs/model-cards/kev-4b.md) |
| [Kev-9B](https://www.ai-hao123.com/zhinan/resolution-47453236.html) | Qwen3.5-9B-Base | 0.822 / 0.852 | 0.872 / 0.874 | 0.286 / 0.237 | 32 GB Mac, L40S, H100 | [Details](docs/model-cards/kev-9b.md) |
| [Kev-27B](https://www.mw-wm.com/xitong/course-14204719.html) | Qwen3.8-27B (post-trained) | **0.848 / 0.896** | 0.866 / 0.870 | **0.236 / 0.164** | B200, H200, H100 80 GB | [Details](docs/model-cards/kev-27b.md) |
| Jev | Hosted | 0.857 / – | 0.845 / – | 0.211 / – | TypeSafe's API | – |

Each cell is **development / test**. "New sources" means datasets and policy rules Kev never saw during training. It is the closest thing here to your own questions. "Trained sources" means held-out examples from the datasets Kev was trained on. We pick checkpoints using the development sets and read each test set only once per released model. Jev has only been run on the development sets. Brier scores the whole probability distribution, not just the top answer; lower is better.

On new sources Kev-27B is within a point of Jev (0.848 vs 0.857), and Kev-4B and Kev-9B are within four points. We don't know what Jev was trained on, so this isn't a controlled comparison of the two architectures. [What to Expect](#what-to-expect) says where Kev is as good as Jev and where it isn't.

Kev-0.8B, 4B and 9B start from Qwen base models and share one training recipe. Kev-27B starts from Qwen's post-trained release, and we don't know what that was trained on. Each model card has the full recipe, all results, and the earlier versions kept as Hub tags. The weights are also in the [GitHub release](https://www.ai-hao123.com/pingce/story-12675492.html), with SHA-256 checksums.

## Quick Start

### Try It in the Browser

The [Hugging Face Space](https://www.mw-wm.com/tuiguang/status-17735565.html) runs Kev-4B and Kev-0.8B, with nothing to install.

### Run It Locally

You'll need Python 3.12 or 3.13 and [uv](https://www.yx-sf.com/tech/75281). The repo's `.python-version` makes `uv sync` use 3.13; torch has no wheels for 3.14 yet.

```bash
git clone https://github.com/jaredpalmer/kev.git && cd kev
uv sync --extra serve
uv run --extra serve python -m kev.serve --run jaredpalmer/kev-4b --port 8009
```

This starts Kev-4B on your machine: CUDA or ROCm if you have a GPU, MLX on Apple Silicon. The first run downloads the adapter and the base model. `--run` also accepts a local checkpoint directory or a Hub revision like `jaredpalmer/kev-4b@qwen3`.

In another terminal, send it a ticket:

```bash
curl -s localhost:8009/v1/systemone -H 'content-type: application/json' -d '{
  "state": "Shoes arrived two weeks late and in the wrong size. Also I see two charges on my card.",
  "model": "kev-latest",
  "questions": {
    "department":  {"type": "choice", "instructions": "Which team should handle this?",
                    "criteria": {"returns": "Exchanges, refunds, wrong or damaged items",
                                 "shipping": "Delivery status, delays, lost packages",
                                 "billing": "Charges, invoices, payment problems"}},
    "escalate":    {"type": "noul",  "instructions": "Does this need urgent human attention?"},
    "frustration": {"type": "score", "instructions": "How frustrated is the customer?",
                    "criteria": ["Calm", "Frustrated", "Very angry"]}
  }}'
```

Example response from Kev-4B, running in bf16 on an Apple M5:

```json
{
  "model": "kev-latest",
  "answers": {
    "department":  { "type": "choice", "choice": "returns", "confidence": 0.21,
                     "probabilities": { "returns": 0.47, "shipping": 0.28, "billing": 0.25 } },
    "escalate":    { "type": "noul", "noul": 0.93 },
    "frustration": { "type": "score", "score": 1.44, "confidence": 0.34,
                     "legend": { "0": "Calm", "1": "Frustrated", "2": "Very angry" },
                     "probabilities": { "0": 0.00, "1": 0.56, "2": 0.44 } }
  },
  "usage": { "input_tokens": 101, "output_tokens": 161 },
  "latency_ms": 495
}
```

The ticket mentions a return, a late delivery and a billing problem, and the department probabilities say so. That's why Kev returns probabilities instead of a single label: your code can route the confident cases and send the rest to a person.

### Use It From Python

If you already call Jev, point your client at Kev and keep the rest of your code. The TypeSafe SDK is included in `uv sync --extra serve`:

```python
from typesafe_sdk import Choice, Noul, Score, TypeSafeClient

client = TypeSafeClient(
    api_key="local",
    base_url="http://127.0.0.1:8009",
    model="kev-latest",
)
response = client.system_one(
    state="I was charged twice. Please fix this ASAP.",
    questions={
        "billing": Noul(instructions="Is this ticket about billing?"),
        "tone": Choice(
            instructions="What is the customer's tone?",
            criteria={"calm": None, "frustrated": None, "angry": None},
        ),
        "urgency": Score(
            instructions="How urgent is this ticket?",
            criteria=["can wait", "this week", "today"],
        ),
    },
)
print(response.nouls["billing"].noul)
print(response.choices["tone"].choice)
print(response.scores["urgency"].score)
```

## Fine-Tune on Your Own Data

The released models were trained on public datasets and generated policy examples. If your questions look different, like your own routing categories, your own escalation rules or another language, a short fine-tune usually helps more than any prompt change. It also fits the temperature to your data, so the confidence you set thresholds on is measured on your own labels.

What to expect: on an example support workload (three questions, 1,050 generated records, 15 minutes on an H100), fine-tuning took Kev-4B from 67.7% to 73.6% accuracy, and from automating 34% of decisions at a 5% error budget to 48% ([details](skills/kev-finetune/README.md#why-fine-tune-at-all)). On real data, one epoch on 5,219 labelled consumer-finance complaints took Kev-4B from 0.804 to 0.904 accuracy on complaints it had never seen. Gains like these are in distribution: they tell you how well Kev learns your task, not how it does on everything else. Size your dataset first. With 400 records, the gain on the example workload was inside the noise.

### With a Coding Agent

```bash
npx skills add jaredpalmer/kev@kev-finetune
```

Then ask your agent to "fine-tune Kev on my support tickets". The [`kev-finetune` skill](skills/kev-finetune/) interviews you, finds the questions your code already asks Jev or TypeSafe, converts the labels you have or generates enough with any LLM to measure a gain, fine-tunes from a released checkpoint on Modal, fits the temperature on a held-out slice, scores the result against the untouched model, deploys an endpoint, and tears everything down at the end. You don't need a local GPU or a clone of this repo. A Kev-4B training run costs about $1 on an H100.

### By Hand

The skill's [README](skills/kev-finetune/README.md) is the same recipe for people: six short standard-library scripts and one Modal app. To train from this repo instead, put your examples in a JSONL file, one request per line. It's the same shape as an API request, plus a `label` on every question:

```jsonl
{"state": {"subject": "Charged twice", "body": "I see two charges for order #4411. Please refund one."},
 "questions": {
   "team":     {"type": "choice", "instructions": "Which team should handle this ticket?",
                "criteria": {"billing": "Payments and refunds", "shipping": "Delivery problems", "access": "Login and account access"}, "label": "billing"},
   "angry":    {"type": "noul",   "instructions": "Is the customer angry?", "label": false},
   "priority": {"type": "score",  "instructions": "How urgent is this ticket?", "criteria": ["low", "normal", "high"], "label": 1}}}
```

For `choice` the label is the option name, for `noul` it's `true` or `false`, and for `score` it's the level's position starting at 0. Keep 10–20% of the file aside for evaluation.

Then start from a released checkpoint with `--init_from`:

```bash
uv run python -m kev.train --data train.jsonl --base Qwen/Qwen3.5-4B-Base --init_from jaredpalmer/kev-4b \
    --epochs 2 --lr 2e-5 --batch 1 --accum 8 --dtype bf16 --checkpointing 1 --device cuda --out runs/mine

uv run python -m kev.benchmark --run runs/mine --data heldout.jsonl --out runs/mine-eval
uv run --extra serve python -m kev.serve --run runs/mine --port 8009
```

`--init_from` loads the adapter and pointer head from the released model before training, so you keep what Kev already knows and add your domain on top. Starting from the base model instead throws that away: in one user's test on 836 support-tool decisions, a fine-tune from the base scored 0.33 on Kev's own evaluation set, against 0.84 for the released model; the same data with `--init_from` kept 0.83 there and reached 0.88 on the new domain. Use a smaller learning rate than the from-scratch recipe (`2e-5` is a good start), and pick `--base` to match the checkpoint you start from; the trainer checks that the base, revision, LoRA rank and head size agree before it loads anything.

`--batch 1 --accum 8` in bf16 fits the 0.8B model on a 4 GB GPU. The benchmark reports accuracy, Brier score and calibration per question type, so you can see which of your questions the fine-tune helped. The checkpoint you started from is recorded in `runs/mine/training_config.json`. On a Mac, run one training job at a time; two jobs on the same Apple GPU are much slower.

## Deploy Your Own Endpoint

To get an HTTPS endpoint instead of a local server, you don't need this repo, just a [Modal](https://www.yx-sf.com/tech/33415) account:

```bash
pip install modal && modal setup
curl -LO https://raw.githubusercontent.com/jaredpalmer/kev/main/skills/kev-deploy/scripts/kev_serve.py
KEV_API_KEY=$(openssl rand -hex 24) modal deploy kev_serve.py
```

That serves Kev-4B on an L40S at `https://<your-workspace>--kev-api.modal.run`, with the same API as above behind `Authorization: Bearer <key>`. It scales to zero when idle, so an unused endpoint costs nothing. The first request after idle waits about 35 seconds for a container to start. `KEV_MODEL=jaredpalmer/kev-9b` serves another model on the GPU that suits it; Kev-27B goes to a B200, falling back to an H200 or H100. If you use a coding agent, `npx skills add jaredpalmer/kev@kev-deploy` does the same and wires the URL into your code. [skills/kev-deploy](skills/kev-deploy/) has the GPU and cost table.

A model you fine-tuned with the `kev-finetune` skill deploys the same way from its own Modal app (`KEV_SERVE_SECRET=kev-serve-key KEV_SERVE_RUN=<run> modal deploy scripts/kev_modal.py`; see [its deploy guide](skills/kev-finetune/references/deploy.md)). To host Kev on your own machines instead, run `kev.serve` from [Run It Locally](#run-it-locally) on a GPU box with `--host 0.0.0.0` and put it behind your own proxy; [Serving Performance](#serving-performance) says which GPU to pick.

## What to Expect

**Accuracy.** Kev-27B is within three points of Jev, or ahead of it, on 9 of the 11 new-source categories in the chart below. Kev-4B and Kev-9B are about as close on classification-shaped sources like routing, entailment and science questions. All of them trail on knowledge questions, which depend mostly on the base model (MMLU: Kev-9B 0.74, Kev-27B 0.84, Jev 0.90), and the smaller models also trail on day-precision date arithmetic.

![Accuracy by source for Kev and Jev](docs/kev-family.png)

**Confidence.** Each checkpoint ships with a fitted temperature, so its probabilities are calibrated by default. As served, Kev-9B puts at least 0.9 probability on a wrong answer for 4.0% of new-source questions, against Jev's 3.7%. Jev still ranks its answers better: at a 5% error budget, Kev can automate 0.45–0.57 of decisions and Jev 0.70. Check a threshold on your own data before you rely on it.

**Speed.** Kev-4B answers six questions about a new short text in 18.1 ms of model time on an H100 and 41.5 ms on an L40S, and a container serves around 101 requests per second on an H100. On an Apple M5, Kev-4B takes 721 ms for five questions, or 136 ms when the text repeats and comes from the cache. [Serving Performance](#serving-performance) has every GPU and batch size.

**Length.** Training used states of up to 384 tokens. The server accepts states of up to 65,536 tokens, and 8,192 more for each question. Longer inputs work, but accuracy drops on long documents. Kev-27B holds up much better: on a panel of questions buried in 1k–6k tokens of unrelated text it scores 0.833, against Kev-9B's 0.556.

## Playground

With the server running, open another terminal. You'll need Node 20.9+:

```bash
cd playground
npm install
npm run dev -- -p 3001
```

Open [localhost:3001](https://www.ai-hao123.com/zhizhu/collaboration-95168073.html), load a preset, and edit the text and questions. Press `⌘↵` to run it. "Packed vs separate" compares asking all questions at once with asking them one at a time. "Permute" runs a Choice question with six option orders. There are also presets for testing question isolation and fake delimiter tokens.

![Kev playground](docs/playground.png)

There's a [chess demo](https://www.ai-hao123.com/yingxiao/subject-62889341.html), too. The board is the input, legal moves are Choice options, and a Score question rates the position. You can play against Kev or let it play itself. Games are saved in `localStorage`.

## API

### `POST /v1/systemone`

`state` is the text to evaluate. Each question has instructions and, where needed, a set of answers to choose from.

```jsonc
{
  "state": "…",                          // string | object | array — the content to evaluate
  "model": "kev-latest",
  "questions": {
    "<id>": {                            // you choose the id; the model never sees it
      "type": "noul" | "choice" | "score",
      "instructions": "…",               // string | object | array, optional
      "criteria": …                      // noul: {true?, false?}  choice: {option: description|null}  score: [level, …]
    }
  }
}
```

| Type | Criteria | Answer |
|---|---|---|
| `noul` | Optional descriptions for `true` and `false` | `noul`: probability of yes |
| `choice` | 1–255 option names, each with a description or `null` | `choice`: most likely option; `probabilities` and `confidence` |
| `score` | 1–255 descriptions, ordered from lowest to highest | `score`: mean level index, starting at 0; `legend`, `probabilities`, and `confidence` |

For Choice with `K > 1` options, confidence is `(p_max − 1/K) / (1 − 1/K)`. A single option has confidence 1. Score confidence is `max(0, 1 − E|level − mode| / D)`: `mode` is the most likely level and `D` is the mean distance of a uniform distribution over the levels from its middle (2/3 for three levels), so all probability on one level gives 1 and a uniform or wider spread gives 0. Both formulas are the ones in TypeSafe's reference adapter ([`system-one-adapter`](https://www.mw-wm.com/shangye/hosting-53608870.html) 0.2.1). Neither field is a measured accuracy rate.

Objects and arrays are converted to labeled text. Delimiter-like strings in user input are escaped before tokenization. Invalid requests return `422`. `usage.output_tokens` counts tokens in the serialized answers, not generated tokens.

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/models` | Model cards (`name`, `description`, `release_date`) plus the loaded checkpoint's details |
| `POST` | `/v1/systemone/permute` | Run one Choice question with different option orders (`n_perm` 1 to 64, default 6) |
| `POST` | `/v1/systemone/separate` | Run each question in its own forward pass |

A request may carry any number of questions. The server runs them a token budget at a time (one maximal row of 16,384 tokens per forward pass, counting the cached document once per question in that pass), so memory does not grow with the question count and the answers do not depend on the split. Every response carries an `x-typesafe-request-id` header. The server binds to `127.0.0.1` (`--host 0.0.0.0` to accept other machines) and is open by default; set `KEV_API_KEY` to require `Authorization: Bearer <key>` on `/v1/*`, as the TypeSafe clients always send it.

| Variable | Effect |
|---|---|
| `KEV_TEMPERATURE=1.0` | Return raw probabilities instead of the calibrated ones |
| `KEV_DATE_FACTS=1` | Append the number of days between any two dates in the state (see [Benchmarks](#benchmarks)) |
| `KEV_DTYPE=fp32` | Serve the exact fp32 path the evaluations use (bf16 is the default on GPUs) |
| `KEV_API_KEY` | Require a bearer key |

## How It Works

Each checkpoint is a rank-16 LoRA adapter and a small pointer head on a Qwen base model. On an attention-only base (Qwen3), the state and questions go into one token sequence:

```text
<state> …state…
<q> instructions <opt> option 1 </opt> <opt> option 2 </opt> … <decide>
<q> instructions <opt> option 1 </opt> <opt> option 2 </opt> … <decide>
```

The attention mask lets a token read the state and its own question, but not other questions or future tokens. Each question's position IDs restart just after the state. This lets the model process the state once and answer each question independently.

Qwen3.5 and Qwen3.8 mix attention layers with Gated DeltaNet layers, which are recurrent and ignore attention masks. For those models, which is every current Kev, each question runs as its own row: the state followed by that question, with the same positions as above. The rows are independent, so isolation is exact, and the server and `DecisionModel.probs()` compute the state once and reuse its cache for every row. `forward()`, which `kev.benchmark` scores and every published number comes from, keeps the plain rows and runs the state once per question; the two agree to fp32 rounding. On attention-only models the rows and the mask above give identical probabilities (`tests/test_model.py`).

Kev-27B uses the same design on `Qwen/Qwen3.8-27B`, with two differences. Its base is Qwen's post-trained release rather than a `-Base` checkpoint, and we don't know what it was post-trained on. And its frozen weights are held in bf16 (`--weights_dtype bf16`), because fp32 weights don't fit next to the optimizer on one GPU. It therefore serves in bf16 only, with 55 GB of weights (about 66 GB resident with the serving buffers), which is why it needs an 80 GB card and has no Mac path. Serving folds the adapter into those bf16 weights, as for the other Kevs; its served probabilities stay within 0.009 of the evaluation path on an H200 (`runs/fused-27b-h200`).

The pointer head scores each option's `</opt>` hidden state against the question's `<decide>` hidden state. A softmax turns those scores into probabilities. Because `<decide>` comes last, it can attend to the full option list.

Training uses cross-entropy on the correct answer. The adapter and head are trained together; the rest of the base weights stay fixed. Training examples and API requests use the same text format. No Jev outputs were used for training.

Asking questions together or separately produces probabilities within 4e-6 in the fp32 tests. This does **not** mean option order is irrelevant: options within a question can still affect one another. See [the model code](kev/model.py) and [parity tests](tests/test_model.py).

## Training

The released models share one base training set, `decision-v7`: 10,000 examples from ten public datasets, 896 generated policy examples, and 1,680 examples from 60 generated rule structures. Kev-0.8B, 4B and 9B train on it for two epochs with LoRA rank 16 and cross-entropy. The learning rate is `1e-4` for 0.8B and `5e-5` for 4B and 9B. On these hybrid bases the adapter covers the attention, MLP and DeltaNet projections; `kev.train` picks the right targets from the model config.

Kev-0.8B, 4B and 9B then get short follow-up fine-tunes from their released checkpoints, through the same `--init_from` path you'd use for your own data: generated cases that state day counts or have the deciding evidence removed (all three), then real documents and generated skill data (4B and 0.8B). Kev-27B trains in a single one-epoch run on one H200 (learning rate `5e-5`, the same adapter targets): Kev-9B's data, plus 1,400 records with a question buried in 1k–6k tokens of unrelated text, and soft targets instead of one-hot labels on records whose answer is genuinely ambiguous. The buried-question records target long documents, where it holds up much better than the smaller models. The model cards list every stage with its data and cost.

```bash
# sanity run, ~1 minute
uv run python -m kev.train --n_per_source 40 --accum 4 --out runs/smoke

# the first stage of Kev-0.8B (~20 min on one H100; the Mac path works but is slow for Qwen3.5 bases)
uv run python -m kev.train --suite evals/v7/decision-v7 --base Qwen/Qwen3.5-0.8B-Base --base_revision dc7cdfe2ee4154fa7e30f5b51ca41bfa40174e68 \
    --epochs 2 --lr 1e-4 --batch 8 --dtype bf16 --p_none_pair 0.25 --device cuda --out runs/kev-0.8b

# the first stage of Kev-4B (one H100 via Modal, ~1 h; see below). Swap in Qwen/Qwen3-4B-Base for the previous generation.
uv run python -m kev.train --suite evals/v7/decision-v7 --base Qwen/Qwen3.5-4B-Base --base_revision 1001bb4d826a52d1f399e183466143f4da7b741b \
    --epochs 2 --lr 5e-5 --batch 4 --accum 2 --dtype bf16 --checkpointing 1 --p_none_pair 0.25 --device cuda --out runs/kev-4b
```

Use `uv run python -m kev.train --help` for all training options. The released models don't use the optional `--perm_kl` or `--ord_w` losses. [PLAN.md](PLAN.md) records what was tried, what helped, and what didn't.

### Modal

Each trial gets its own H100. The study keeps running if you disconnect, and you can download the results when it finishes:

```bash
uv run modal token new                                    # once; opens the browser
KEV_GPU=T4 uv run modal run modal_app.py::smoke           # end-to-end check, ~1 minute of GPU

uv run modal deploy modal_app.py                          # once; studies run on the deployed app and survive disconnects
uv run modal run modal_app.py::study \
    --suite evals/v7/decision-v7 --plan experiments/v7-final.json \
    --name my-study --transfer evals/v4/transfer-v4 --budget 30 --timeout 7200
uv run modal run modal_app.py::pull --name my-study       # results -> runs/my-study, ranked
```

[Study plans](experiments/v7-final.json) list training settings. Each trial saves the settings, code hashes, dataset hashes, and results. Choose models using the development results, not the locked test. After choosing a final candidate, you can read its test results once:

```bash
uv run modal run modal_app.py::locked_test --trial my-study/00-trial-0 --name my-candidate   # one read, ever
```

## Benchmarks

The evaluation data under `evals/` is frozen: dataset versions and file checksums are recorded in each manifest. Large files are downloaded from [the Hub mirror](https://www.yx-sf.com/tech/63320) and checked against those hashes. Every model in the tables above is scored on the same items. The numbers in this README and the model cards are checked in CI against the committed reports they come from (`docs/claims.json`, `uv run python scripts/verify_claims.py`).

| Suite | What it measures |
|---|---|
| `decision-v7` | Held-out examples from the ten training datasets, the generated policies and the rule structures ("trained sources") |
| `transfer-v4` | 764 records from datasets and policy and rule types Kev never trained on: QNLI, SciQ, PAWS, MMLU, Emotion, TweetEval, held-out policies and rules ("new sources") |
| `transfer-v9` | `transfer-v4` plus 10-way MMLU-Pro, records buried in unrelated text, and "unknowable" records whose deciding evidence was removed |

```bash
uv run python -m kev.benchmark --run jaredpalmer/kev-4b --suite evals/v4/transfer-v4 --out runs/my-eval      # new sources
uv run python -m kev.benchmark --run jaredpalmer/kev-4b --suite evals/v9/transfer-v9 --out runs/my-eval-v9   # + MMLU-Pro, buried states, unknowable items
uv run python -m kev.benchmark --run jaredpalmer/kev-4b --suite evals/v7/decision-v7 --out runs/my-eval-id   # trained sources
uv run python -m kev.benchmark --remote http://127.0.0.1:8009 --suite evals/v4/transfer-v4 --out runs/my-remote   # any System One endpoint, Jev included
```

These commands use development data. Test data requires `--allow-test`. The benchmark reports accuracy, Brier score, calibration error, the share of decisions you could automate at a 5% error budget, option-order changes, and question isolation. On the unknowable records it reports how often the model still answers with at least 0.9 confidence (Kev-9B 0%, Jev 9%). Published accuracy numbers use fp32 evaluation, not the bf16 serving path. `kev.jev` runs the same questions against Jev through Vercel AI Gateway, and `kev.compare` compares two saved runs with paired bootstrap confidence intervals.

**Calibration.** Each checkpoint stores a temperature (Kev-27B 1.38, Kev-9B 2.30, Kev-4B 2.41, Kev-0.8B 2.35) fitted on its in-distribution development set, and the pointer head applies it when the model is loaded. It never changes which answer wins. On new sources it takes Kev-9B's calibration error from 0.106 to 0.042 and its confident errors (wrong answers with probability ≥ 0.9) from 8.7% to 4.0%, about Jev's 3.7%. The accuracy numbers above are the same either way; the Brier numbers are for the raw probabilities. `scripts/calibrate_checkpoint.py` also reports an out-of-fold estimate, so the in-sample fit can be checked against records it didn't see.

**Dates.** Kev can't subtract dates reliably, but it can use a day count it's given. `KEV_DATE_FACTS=1` appends one sentence per pair of dates in the state ("June 26, 2026 is 8 days before July 4, 2026"). On the deadline policy questions this takes Kev-9B from 0.80 to 0.90 (Jev 0.93). None of the tables use it.

**Other people's test sets.** `evals/external/` holds test sets from other projects, converted to this format, with their published live Jev results. [SemIf](https://www.ai-hao123.com/anfang/team-40271891.html) uses the last two to compare its own models, and they are rebuilt from the same hash-verified sources with `scripts/freeze_semif_external.py`. Some were scored on earlier versions of the Kev weights, which the Kev column names.

| Suite | What it is | Jev | Kev |
|---|---|---|---|
| [SemIf](https://www.mw-wm.com/jishu/forecast-60734392.html) | 144 authored decisions | 0.965 | 0.917 (Kev-9B at `v7-base`) |
| [scienthoon](https://www.mw-wm.com/huodong/recipe-96357024.html) | 900 support tickets, routing / tone | 0.897 / 0.914 | 0.952 / 0.911 (Kev-9B at `v7-base`) |
| `wanli-v1` | 256 WANLI test pairs: supported, insufficient or contradicted | 0.758 | 0.703 (Kev-9B), 0.695 (Kev-4B at `night2-du-release`) |
| `typesafe-v1` | The 102 public evals.typesafe.ai questions over 20 cases: agreement / distance on the 89 that fit | 0.891 / 0.125 | 0.809 / 0.226 (Kev-9B), 0.856 / 0.231 (Kev-4B at `night2-du-release`) |

Kev trails Jev on WANLI and TypeSafe's evals. SemIf reports 0.637 balanced accuracy on WANLI for untrained Qwen3.5-4B. TypeSafe's cases are scored the way SemIf scores them (`scripts/compare_typesafe.py`): agreement with the reference answer and total-variation distance to the reference distribution, averaged within each case and then over cases. The published TypeSafe answers score 0.883 / 0.127 on the same rows. The documents are long, and 13 of them exceeded the 8,192-token serving context these were scored under; counting those as wrong, Kev-9B scores 0.728 / 0.304 and Kev-4B 0.770 / 0.308 over all 102. Document length explains part of the gap: Kev-9B answers 0.92 of the 26 documents inside its 384-token training context and 0.75 to 0.79 of the longer ones (Kev-4B 0.88, then 0.81 to 0.88).

## Serving Performance

Pick the GPU by the model:

| Model | GPU ($/h) | 6 questions, short text | 5 questions, 2,200-token text | Requests/s, 64 clients |
|---|---|---|---|---|
| Kev-0.8B | L4 (0.80) | 22.7 / 16.1 ms | 108.6 / 32.3 ms | 62.8 |
| Kev-4B | L40S (1.95) | 41.5 / 27.7 ms | 145.2 / 43.0 ms | 51.4 |
| Kev-4B | H100 (3.95) | 18.1 / 12.9 ms | 89.4 / 22.5 ms | 100.8 |
| Kev-9B | L40S (1.95) | 66.4 / 42.7 ms | 235.6 / 57.5 ms | 32.7 |
| Kev-9B | H100 (3.95) | 24.0 / 16.6 ms | 88.5 / 26.4 ms | 79.5 |
| Kev-27B | B200 (6.25) | 46.5 / 32.2 ms | 178.0 / 52.1 ms | 44.2 |
| Kev-27B | H200 (4.54) | 65.5 / 48.0 ms | 267.9 / 71.9 ms | 30.3 |
| Kev-27B | H100 (3.95) | 75.0 / 52.0 ms | 277.5 / 79.3 ms | 28.9 |

Times are model time per request (the `latency_ms` the API returns), median of 20, for a new text / the same text again. The server caches the text, so asking more questions about a document you've already sent only pays for the questions. Requests per second are for 64 concurrent clients sending six questions about a new short text each; the server batches them. Network time is extra: about 65 ms per round trip through a Modal web endpoint in the same region.

An L4 is enough for Kev-0.8B but too slow for Kev-4B. The A100 is slower than the L40S here and costs more. Kev-9B needs about 17 GB of GPU memory and Kev-27B 55 GB of weights (about 66 GB with the batching buffers); under load Kev-27B is compute-bound, and a B200, H200 or H100 costs about the same per request. On CUDA, install `flash-linear-attention` for the Qwen3.5 models (`kev_serve.py` and the Modal images already do).

On Apple Silicon, `uv sync --extra serve` installs [MLX](https://www.ai-hao123.com/yingxiao/creative-93556524.html) and the server uses it automatically. Five questions about a ~270-token text on an M5 (32 GB):

| Model | New text | Same text again |
|---|---|---|
| Kev-0.8B | 149 ms | 28 ms |
| Kev-4B | 721 ms | 136 ms |

The server runs in bf16 on GPUs and Macs. Its probabilities differ from the fp32 path the published evaluations use by at most about 0.03 on a GPU and 0.05 on a Mac, and the top answer changes on about one question in 300. Set `KEV_DTYPE=fp32` for the exact path. `/v1/models` reports the backend and precision in use. `uv run modal run modal_app.py::serving --run jaredpalmer/kev-4b --gpu L40S --name <name>` measures a row of the table on your own account (the rows above: `runs/serve-*`, `runs/grouping-4b-h100`, `runs/fused-27b-*`).

## Limitations

- Calibration is a single temperature fitted in distribution. It can't reorder confidences, so the share of decisions you can automate at a 5% error budget (0.45–0.57) is still below Jev's 0.70. Test a probability threshold on your own data before you rely on it.
- Knowledge questions are set by the base model. MMLU is 0.74 for Kev-9B against Jev's 0.90, and MMLU-Pro 0.52 against 0.84.
- Fine-tuning can make the base model worse at individual tasks. Date arithmetic was the clearest case ([issue #8](https://www.mw-wm.com/qiye/analytics-81311902.html)); training on stated day counts plus `KEV_DATE_FACTS=1` recovers it.
- Changing option order can change an answer. Question isolation doesn't prevent this.
- Training used at most 384 state tokens and 1,024 tokens for the state plus one question. Serving allows a 65,536-token state; longer context wasn't covered by training.
- On a Mac, answers take hundreds of milliseconds, not tens. Kev-27B needs an 80 GB GPU and has no Mac path.
- Kev-27B starts from a post-trained model whose training data we don't know.

## Development

```bash
uv run --extra serve python -m pytest tests/test_unit.py tests/test_research.py tests/test_generators.py tests/test_conventions.py \
    tests/test_documents_tools.py tests/test_hard_v1.py tests/test_devtools_v1.py tests/test_breadth_v1.py tests/test_rounds.py tests/test_skill_scripts.py -q   # no weights, no server; what CI runs
KEV_BASE_URL=http://127.0.0.1:8009 uv run --extra serve python -m pytest tests/test_api.py -q   # against a running server
cd playground && npm run lint && npx next typegen && npx tsc --noEmit -p .
```

The API tests run TypeSafe's example requests and the official SDK against your local server. [PLAN.md](PLAN.md) is the research plan: what we have learned, the rules every experiment follows, and one line per round. The full log (every experiment, the criteria set before it ran, and how it came out) is at the git tag `research-archive-2026-09-24`.

<details>
<summary>Previous generation (Qwen3) and the prototype</summary>

The first Kev family used Qwen3 bases with the same data and settings. Those weights stay published and run on plain PyTorch on a Mac, but they are no longer developed.

| Model | Base | Accuracy: Trained Sources | Accuracy: New Sources | Brier: New Sources | Model Card |
|---|---|---|---|---|---|
| Kev-0.6B (Qwen3) — `jaredpalmer/kev-0.6b` | Qwen3-0.6B-Base | 0.801 / 0.808 | 0.620 / 0.642 | 0.536 / 0.483 | [Details](docs/model-cards/kev-0.6b-qwen3.md) |
| Kev-4B (Qwen3) — `jaredpalmer/kev-4b@qwen3` | Qwen3-4B-Base | 0.854 / 0.856 | 0.790 / 0.806 | 0.328 / 0.294 | [Details](docs/model-cards/kev-4b-qwen3.md) |
| Kev-8B (Qwen3) — `jaredpalmer/kev-8b` | Qwen3-8B-Base | 0.863 / 0.870 | 0.796 / 0.780 | 0.337 / 0.327 | [Details](docs/model-cards/kev-8b-qwen3.md) |

The original [Kev-0.5B](https://www.yx-sf.com/tech/20980) used Qwen2.5-0.5B and is kept for reference; see its [model card](docs/model-cards/kev-0.5b.md).

</details>

<details>
<summary>Troubleshooting</summary>

- If MPS runs out of memory during training, check that you're running only one job. Don't enable `output_hidden_states` or add tokens with peft's `trainable_token_indices`; both have caused memory problems here.
- If the playground loads but buttons don't work, use `localhost:3001`. Next.js checks development hostnames. Other hosts need an entry in `allowedDevOrigins` in `playground/next.config.ts`.
- If dataset loading reports `Dataset scripts are no longer supported`, use `legacy-datasets/banking77`. This repo already uses it.

</details>

## Authors

- Jared Palmer ([@jaredpalmer](https://www.mw-wm.com/kaifa/url-21531398.html))

Built with [Devin](https://www.mw-wm.com/kuangjia/system-37030123.html). Thanks to [Archer Hume](https://www.yx-sf.com/news/8111) for the architecture write-up, [TypeSafe](https://www.yx-sf.com/wiki/43363) for the API design, [Qwen](https://www.yx-sf.com/tech/60220) for the base models, [3x3xX3N0N](https://www.ai-hao123.com/liuliang/technology-90681413.html) for showing where the date-arithmetic failure really is, and [Radexito](https://www.yx-sf.com/news/47727) for `--init_from`.

Related work: [Hydragen](https://www.mw-wm.com/zixun/enterprise-19580485.html), [DeFT](https://www.mw-wm.com/yingxiao/theme-13187713.html), [FIRST](https://www.mw-wm.com/guanjianci/workshop-99202841.html).

## License

[Apache-2.0](LICENSE). The Qwen3, Qwen3.5 and Qwen3.8 base models are also Apache-2.0. Training datasets have their own licenses; see the [model cards](docs/model-cards/).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/qiye/income-20909367.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/tech/88036)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/kaifa/budget-64514123.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/zhizhu/resource-84169426.html)
* [全息网络通信节点白名单-#005](https://www.yx-sf.com/news/75428)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/yanjiu/creative-26689948.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/tuiguang/security-34022413.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/wiki/2317)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/yunying/responsive-27451943.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/xinwen/backup-49054206.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/news/4057)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/anli/reporting-30685070.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/chanpin/terms-30977385.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/82508)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/tuiguang/analytics-48910875.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/yinqing/terms-34222327.html)
* [全息网络通信节点白名单-#017](https://www.yx-sf.com/tech/37909)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/chanpin/business-29971403.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/wenzhang/automation-20311968.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/news/15677)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/suanfa/course-07214699.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/peixun/forum-58704325.html)
* [高韧性数据交换通道规约-#023](https://www.yx-sf.com/news/88874)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/yingyong/domain-18920409.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/huodong/business-65505055.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/tech/5197)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/baogao/schedule-72241594.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/anli/services-25207363.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/tech/77839)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/huodong/affordable-09026641.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/chanpin/settings-49106364.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/news/74317)
* [高韧性数据交换通道规约-#033](https://www.ai-hao123.com/xuexi/enterprise-87976265.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/qiye/investment-19986890.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/wiki/43932)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/baogao/cheap-51736243.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/kaifa/ai-34209249.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://www.yx-sf.com/wiki/95126)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/zixun/luxury-07610747.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/xinwen/lead-38098009.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/tech/45157)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/jiaoliu/module-40320595.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/zixun/topic-26433191.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/tech/48337)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/shangye/navigation-99933273.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/jiaocheng/experience-57031021.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/tech/50025)
* [异步事件循环架构设计规范-#011](https://www.ai-hao123.com/gongju/theme-99189455.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/zhizhu/device-86958533.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/news/25757)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/gongsi/products-46698984.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/zhineng/interface-17742491.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/tech/72823)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/liuliang/upload-74468079.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/huodong/expensive-38235825.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/news/30506)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/yunying/event-74894671.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/qiye/account-93860157.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/news/40987)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/shangye/page-34736555.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/pingce/study-84000397.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/tech/59159)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/yunying/community-31206981.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/yunsuan/achievement-72930187.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/tech/29337)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/keji/lead-38105115.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/jiaocheng/user-89866026.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/news/94158)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/pingtai/networking-20052386.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/shichang/dashboard-82016105.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/tech/69734)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/wendang/register-07669440.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/guanjianci/chapter-27022221.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/wiki/88199)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/huodong/vendor-70351312.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/peixun/button-81071095.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/news/27153)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/anli/message-20626792.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/yinqing/satisfaction-34471651.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/wiki/86896)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/fuwu/saving-50490298.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/jiaoliu/roi-07263174.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/wiki/95320)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/sheji/revenue-07651347.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/sheji/hosting-42605045.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/tech/39849)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/peixun/finance-56377529.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/chanpin/profile-19423660.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/tech/56650)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/tuiguang/identity-55738700.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/jiaoliu/global-54164712.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/news/17288)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/peixun/food-94306207.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/peixun/tag-24557223.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/tech/45910)
* [北美与欧洲边缘备份节点-#022](https://www.ai-hao123.com/paiming/app-08285569.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/jiaocheng/machine-57770625.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/51047)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/shuju/responsive-45510709.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/jianzhan/photo-98525710.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/wiki/79098)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/zhizhu/analysis-31102318.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/yunying/expensive-54947468.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/tech/18891)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/zhizhu/engagement-74462318.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/guanjianci/theme-12432645.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/tech/55651)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/huodong/coupon-64681178.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/youhua/tag-78313501.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/news/5220)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/fuwu/success-82471758.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/zixun/partner-33916988.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/wiki/51578)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/kaifa/link-47686029.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/yinqing/personalization-63363535.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/news/94451)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/jiaoliu/tool-49641531.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/tuiguang/course-14801850.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/news/53603)
* [权威网络权重与收录基准-#009](https://www.ai-hao123.com/qiye/automation-19523043.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/guanjianci/social-23717948.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/wiki/83776)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/kaifa/experience-57179621.html)
* [节点连通性与存活探测准则-#013](https://www.mw-wm.com/anli/course-19028752.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/news/23745)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/suanfa/shopping-49729722.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/paiming/like-46478391.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/news/25532)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/xinwen/price-05888038.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/kaifa/hosting-74065395.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/90383)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/shichang/engagement-12802052.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/zhineng/traffic-17633997.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/news/77349)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/xinwen/creative-42744318.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/tuiguang/feedback-36506169.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/news/6837)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/tuiguang/domain-34603877.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/shangye/project-96089219.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/tech/57383)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/zhinan/travel-32273461.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/paiming/value-79644099.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/wiki/37841)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/jianzhan/image-41255317.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/wendang/responsive-54076183.html)
* [权威网络权重与收录基准-#035](https://www.yx-sf.com/tech/12386)
* [权威网络权重与收录基准-#036](https://www.ai-hao123.com/shichang/price-91145669.html)
* [防重放安全验证与校验哈希-#037](https://www.mw-wm.com/jiaocheng/conversion-41306078.html)
* [防重放安全验证与校验哈希-#038](https://www.yx-sf.com/news/1207)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/keji/cloud-73547968.html)

</details>

