# Leaderboard

Generated 2026-09-21T11:34+00:00 from runs/*/result.json. Selection on development partitions only; the locked test is never read here.

- **Qwen/Qwen3-0.6B-Base** incumbent: transfer 0.620, dev 0.801, seeds [2] (v7-06b/02-trial-2)
- **Qwen/Qwen3-4B-Base** incumbent: transfer 0.775, dev 0.858, seeds [0] (v7-rc3/00-trial-0, v8-4b/00-trial-0)
- **Qwen/Qwen3-8B-Base** incumbent: transfer 0.796, dev 0.863, seeds [0] (v7-final/00-trial-0)

| study/trial | base | seed | dev acc | transfer acc | Brier | conf-err | held-out pairs | none_present | perm flip | gates | $ | knobs |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| night2-36b6/00-trial-0 | Qwen3.6-35B-A3B | 0 | 0.869 | 0.823 | 0.278 | 0.088 | 0.84 | 0.90 | 0.02 | pass | 10.42 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25, weights_dtype=bf16 |
| night2-9b-du/00-trial-0 | Qwen3.5-9B-Base | 1 | 0.872 | 0.822 | 0.286 | 0.087 | 0.83 | 0.90 | 0.00 | pass | 1.4 | accum=4, data=evals/night2/dates_unknowable.jsonl, init_from=jaredpalmer/kev-9b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| night2-36b6/01-trial-1 | Qwen3.6-35B-A3B | 0 | 0.869 | 0.819 | 0.289 | 0.091 | 0.88 | 0.85 | 0.08 | pass | 9.38 | accum=4, epochs=2, lr=2e-05, p_none_pair=0.25, weights_dtype=bf16 |
| night2-9b/00-trial-0 | Qwen3.5-9B-Base | 1 | 0.873 | 0.819 | 0.295 | 0.090 | 0.80 | 0.92 | 0.00 | pass | 1.1 | accum=4, data=evals/night2/dates.jsonl, init_from=jaredpalmer/kev-9b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| night2-9b/03-trial-3 | Qwen3.5-9B-Base | 1 | 0.875 | 0.819 | 0.291 | 0.082 | 0.81 | 0.92 | 0.00 | pass | 1.8 | accum=4, data=evals/night2/all.jsonl, init_from=jaredpalmer/kev-9b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| night2-9b/02-trial-2 | Qwen3.5-9B-Base | 1 | 0.870 | 0.816 | 0.303 | 0.088 | 0.80 | 0.90 | 0.00 | pass | 1.09 | accum=4, data=evals/night2/assertion.jsonl, init_from=jaredpalmer/kev-9b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| night2-9b/01-trial-1 | Qwen3.5-9B-Base | 1 | 0.870 | 0.814 | 0.300 | 0.088 | 0.78 | 0.92 | 0.02 | pass | 1.21 | accum=4, data=evals/night2/unknowable.jsonl, init_from=jaredpalmer/kev-9b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| q35-9b/01-trial-1 | Qwen3.5-9B-Base | 1 | 0.876 | 0.812 | 0.291 | 0.075 | 0.80 | 0.90 | 0.02 | pass | 6.6 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25 |
| q35-9b/00-trial-0 | Qwen3.5-9B-Base | 0 | 0.871 | 0.802 | 0.308 | 0.069 | 0.75 | 0.92 | 0.03 | pass | 5.78 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25 |
| q35-4b/01-trial-1 | Qwen3.5-4B-Base | 1 | 0.876 | 0.800 | 0.322 | 0.095 | 0.75 | 0.88 | 0.02 | pass | 3.99 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| night2-4b/01-trial-1 | Qwen3.5-4B-Base | 1 | 0.881 | 0.797 | 0.316 | 0.088 | 0.78 | 0.95 | 0.02 | pass | 0.98 | data=evals/night2/unknowable.jsonl, init_from=jaredpalmer/kev-4b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| night2-4b-du/00-trial-0 | Qwen3.5-4B-Base | 1 | 0.872 | 0.797 | 0.299 | 0.069 | 0.78 | 0.92 | 0.02 | pass | 0.85 | data=evals/night2/dates_unknowable.jsonl, init_from=jaredpalmer/kev-4b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| v7-final/00-trial-0 | Qwen3-8B-Base | 0 | 0.863 | 0.796 | 0.337 | 0.099 | 0.69 | 0.87 | 0.02 | fail | 5.78 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25 |
| q35-4b-s23/00-trial-0 | Qwen3.5-4B-Base | 2 | 0.877 | 0.794 | 0.316 | 0.082 | 0.78 | 0.93 | 0.02 | pass | 4.01 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| night2-4b/00-trial-0 | Qwen3.5-4B-Base | 1 | 0.871 | 0.793 | 0.333 | 0.105 | 0.77 | 0.93 | 0.02 | fail | 0.94 | data=evals/night2/dates.jsonl, init_from=jaredpalmer/kev-4b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| night2-4b/02-trial-2 | Qwen3.5-4B-Base | 1 | 0.877 | 0.791 | 0.337 | 0.110 | 0.78 | 0.95 | 0.02 | fail | 1.0 | data=evals/night2/assertion.jsonl, init_from=jaredpalmer/kev-4b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| night2-4b/03-trial-3 | Qwen3.5-4B-Base | 1 | 0.880 | 0.791 | 0.316 | 0.079 | 0.77 | 0.93 | 0.00 | pass | 1.29 | data=evals/night2/all.jsonl, init_from=jaredpalmer/kev-4b, lr=2e-05, p_none_pair=0.25, replay=2000 |
| v7-rc3/01-trial-1 | Qwen3-4B-Base | 1 | 0.854 | 0.790 | 0.328 | 0.082 | 0.73 | 0.87 | 0.02 | pass | 2.84 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| q35-4b/00-trial-0 | Qwen3.5-4B-Base | 0 | 0.877 | 0.788 | 0.339 | 0.096 | 0.72 | 0.92 | 0.03 | pass | 4.15 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| night2-4b-dense2/00-trial-0 | Qwen3.5-4B-Base | 1 | 0.877 | 0.780 | 0.333 | 0.085 | 0.73 | 0.87 | 0.02 | pass | 4.08 | epochs=2, lora_targets=dense, lr=5e-05, p_none_pair=0.25 |
| recipe-8b-r2/01-trial-1 | Qwen3-8B-Base | 1 | 0.868 | 0.779 | 0.336 | 0.081 | 0.67 | 0.87 | 0.05 | fail | 6.49 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25 |
| q35-4b-nopolicy/01-trial-1 | Qwen3.5-4B-Base | 1 | 0.872 | 0.777 | 0.327 | 0.075 | 0.69 | 0.87 | 0.02 | fail | 4.73 | epochs=2, lr=5e-05, p_none_pair=0.25, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,imdb,amazon,compositional |
| v8-4b/00-trial-0 | Qwen3-4B-Base | 0 | 0.858 | 0.777 | 0.338 | 0.076 | 0.69 | 0.85 | 0.03 | fail | 3.06 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| recipe-8b-r1/00-trial-0 | Qwen3-8B-Base | 0 | 0.869 | 0.774 | 0.339 | 0.082 | 0.61 | 0.88 | 0.02 | fail | 4.96 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25 |
| recipe-8b-s2/00-trial-0 | Qwen3-8B-Base | 2 | 0.866 | 0.774 | 0.348 | 0.085 | 0.59 | 0.88 | 0.02 | fail | 6.26 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25 |
| v7-final/01-trial-1 | Qwen3-8B-Base | 1 | 0.864 | 0.774 | 0.358 | 0.099 | 0.64 | 0.90 | 0.03 | fail | 5.98 | accum=4, epochs=2, lr=5e-05, p_none_pair=0.25 |
| anchor-4b-v6/00-trial-0 | Qwen3-4B-Base | 0 | 0.861 | 0.773 | 0.358 | 0.085 | 0.61 | 0.88 | 0.02 | fail | 3.4 | anchor=/runs/anchors/v6-qwen3-4b.json, anchor_sources=arc,openbookqa,csqa, anchor_w=0.5, epochs=2, lr=5e-05, p_none_pair=0.25 |
| v7-rc3/00-trial-0 | Qwen3-4B-Base | 0 | 0.858 | 0.773 | 0.359 | 0.098 | 0.62 | 0.87 | 0.05 | fail | 3.01 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| q35-4b-s23/01-trial-1 | Qwen3.5-4B-Base | 3 | 0.873 | 0.770 | 0.346 | 0.093 | 0.69 | 0.92 | 0.00 | fail | 4.17 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| recipe-8b-r2/00-trial-0 | Qwen3-8B-Base | 0 | 0.855 | 0.770 | 0.348 | 0.075 | 0.62 | 0.87 | 0.05 | fail | 5.92 | accum=4, epochs=2, lr=2e-05, p_none_pair=0.25 |
| v7-final/02-trial-2 | Qwen3-4B-Base | 2 | 0.849 | 0.770 | 0.371 | 0.105 | 0.67 | 0.87 | 0.02 | fail | 3.07 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| auto-4b-r0-91266/01-trial-1 | Qwen3-4B-Base | 0 | 0.840 | 0.767 | 0.355 | 0.062 | 0.61 | 0.87 | 0.07 | fail | 2.76 | epochs=2, lora_targets=all, lr=3e-05, p_none_pair=0.25 |
| auto-4b-r1/04-trial-4 | Qwen3-4B-Base | 0 | 0.843 | 0.767 | 0.350 | 0.059 | 0.59 | 0.88 | 0.05 | fail | 4.94 | epochs=2, lora_targets=all, lr=3e-05, p_none_pair=0.25, perm_kl=0.2 |
| auto-4b-r1/01-trial-1 | Qwen3-4B-Base | 0 | 0.849 | 0.765 | 0.369 | 0.078 | 0.61 | 0.83 | 0.05 | fail | 2.85 | epochs=2, lora_targets=all, lr=3e-05, p_none_pair=0.25, synthetic_repeat=2 |
| v3-8b-s0/00-trial-0 | Qwen3-8B-Base | 0 | 0.843 | 0.765 | 0.377 | 0.104 | 0.56 | 0.83 | 0.02 | fail | 1.45 | accum=4, epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,legacy_policy |
| auto-4b-r1/03-trial-3 | Qwen3-4B-Base | 0 | 0.835 | 0.762 | 0.347 | 0.062 | 0.61 | 0.85 | 0.00 | fail | 2.74 | epochs=2, head_dim=1024, lora_targets=all, lr=3e-05, p_none_pair=0.25 |
| anchor-4b-v6/01-trial-1 | Qwen3-4B-Base | 0 | 0.861 | 0.761 | 0.338 | 0.034 | 0.59 | 0.90 | 0.02 | fail | 3.38 | anchor=/runs/anchors/v6-qwen3-4b.json, anchor_w=0.3, epochs=2, lr=5e-05, p_none_pair=0.25 |
| auto-4b-r0-91266/02-trial-2 | Qwen3-4B-Base | 0 | 0.842 | 0.761 | 0.367 | 0.099 | 0.59 | 0.87 | 0.02 | fail | 4.06 | epochs=3, lr=3e-05, p_none=0.2, p_none_pair=0.25 |
| recipe-4b-r1/03-trial-3 | Qwen3-4B-Base | 0 | 0.849 | 0.761 | 0.384 | 0.105 | 0.56 | 0.85 | 0.05 | fail | 3.4 | epochs=2, lr=3e-05, p_none_pair=0.25 |
| auto-4b-r0-91266/06-trial-6 | Qwen3-4B-Base | 0 | 0.831 | 0.759 | 0.354 | 0.059 | 0.55 | 0.83 | 0.07 | fail | 2.6 | epochs=2, lr=2e-05, p_none=0.2, p_none_pair=0.25 |
| knowledge-4b-v6/01-trial-1 | Qwen3-4B-Base | 0 | 0.854 | 0.759 | 0.351 | 0.046 | 0.55 | 0.85 | 0.05 | fail | 1.93 | lr=0.0001, p_none_pair=0.25 |
| lowdrift-4b-v4/01-trial-1 | Qwen3-4B-Base | 0 | 0.843 | 0.759 | 0.346 | 0.055 | 0.62 | 0.85 | 0.00 | fail | 2.56 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| recipe-4b-v4-s2/00-trial-0 | Qwen3-4B-Base | 2 | 0.855 | 0.759 | 0.362 | 0.072 | 0.55 | 0.90 | 0.05 | fail | 2.83 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| v8-4b/01-trial-1 | Qwen3-4B-Base | 1 | 0.855 | 0.759 | 0.365 | 0.088 | 0.56 | 0.88 | 0.05 | fail | 3.1 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| recipe-4b-v4-s1/00-trial-0 | Qwen3-4B-Base | 1 | 0.853 | 0.758 | 0.371 | 0.062 | 0.58 | 0.88 | 0.05 | fail | 2.9 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| recipe-8b-r1/01-trial-1 | Qwen3-8B-Base | 0 | 0.861 | 0.758 | 0.370 | 0.090 | 0.56 | 0.85 | 0.00 | fail | 2.69 | accum=4, epochs=2, lr=5e-05, option_isolation=1, p_none_pair=0.25, public_frac=0.33, synthetic_repeat=2 |
| auto-4b-r0-91266/05-trial-5 | Qwen3-4B-Base | 0 | 0.828 | 0.756 | 0.370 | 0.064 | 0.56 | 0.83 | 0.08 | fail | 1.7 | epochs=2, lr=3e-05, p_none_pair=0.25, public_frac=0.5 |
| auto-4b-r0-91266/00-trial-0 | Qwen3-4B-Base | 0 | 0.843 | 0.755 | 0.360 | 0.056 | 0.59 | 0.88 | 0.07 | fail | 2.8 | epochs=2, lr=3e-05, p_none_pair=0.25 |
| auto-4b-r1/02-trial-2 | Qwen3-4B-Base | 0 | 0.839 | 0.755 | 0.355 | 0.049 | 0.56 | 0.87 | 0.05 | fail | 2.74 | epochs=2, lora=8, lora_targets=all, lr=3e-05, p_none_pair=0.25 |
| auto-4b-r1/05-trial-5 | Qwen3-4B-Base | 0 | 0.847 | 0.755 | 0.363 | 0.058 | 0.53 | 0.83 | 0.00 | fail | 3.53 | epochs=2, head_lr=0.0005, lora_targets=all, lr=3e-05, p_none_pair=0.25 |
| recipe-4b-r1/01-trial-1 | Qwen3-4B-Base | 0 | 0.861 | 0.755 | 0.370 | 0.091 | 0.58 | 0.87 | 0.02 | fail | 3.66 | epochs=2, lr=5e-05, p_none_pair=0.25 |
| mix-4b-knobs/03-trial-3 | Qwen3-4B-Base | 0 | 0.840 | 0.752 | 0.361 | 0.062 | 0.58 | 0.87 | 0.00 | fail | 1.94 | epochs=2, option_isolation=1, p_none_pair=0.25, public_frac=0.33, synthetic_repeat=2 |
| v3-data-capacity-s1/03-trial-3 | Qwen3-4B-Base | 1 | 0.840 | 0.750 | 0.357 | 0.052 | 0.62 | 0.82 | 0.00 | fail | 1.09 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| auto-4b-r0-91266/03-trial-3 | Qwen3-4B-Base | 0 | 0.842 | 0.748 | 0.369 | 0.056 | 0.56 | 0.85 | 0.05 | fail | 2.42 | epochs=2, lr=3e-05, p_none=0.2, p_none_pair=0.25 |
| auto-4b-r0-91266/04-trial-4 | Qwen3-4B-Base | 0 | 0.849 | 0.748 | 0.361 | 0.052 | 0.55 | 0.85 | 0.05 | fail | 2.55 | epochs=2, lr=3e-05, p_none_distract=0.25, p_none_pair=0.25 |
| recipe-4b-r1/00-trial-0 | Qwen3-4B-Base | 0 | 0.835 | 0.748 | 0.383 | 0.096 | 0.56 | 0.77 | 0.00 | fail | 1.46 | epochs=2, lr=5e-05, option_isolation=1, p_none_pair=0.25, public_frac=0.33, synthetic_repeat=2 |
| recipe-4b-r1/04-trial-4 | Qwen3-4B-Base | 0 | 0.835 | 0.748 | 0.402 | 0.125 | 0.53 | 0.80 | 0.00 | fail | 2.17 | epochs=3, lr=5e-05, option_isolation=1, p_none_pair=0.25, public_frac=0.33 |
| v3-data-capacity-s0/03-trial-3 | Qwen3-4B-Base | 0 | 0.825 | 0.747 | 0.382 | 0.088 | 0.52 | 0.83 | 0.08 | fail | 0.89 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| lowdrift-4b-v4/03-trial-3 | Qwen3-4B-Base | 0 | 0.842 | 0.742 | 0.366 | 0.066 | 0.55 | 0.80 | 0.08 | fail | 1.88 | epochs=2, lora=4, lora_targets=qv, p_none_pair=0.25 |
| anchor-4b-v6/02-trial-2 | Qwen3-4B-Base | 0 | 0.861 | 0.741 | 0.384 | 0.093 | 0.56 | 0.87 | 0.02 | fail | 3.51 | anchor=/runs/anchors/v6-qwen3-4b.json, anchor_sources=arc,openbookqa,csqa, anchor_w=1.0, epochs=2, lr=5e-05, p_none_pair=0.25 |
| mix-4b-knobs/01-trial-1 | Qwen3-4B-Base | 0 | 0.846 | 0.741 | 0.393 | 0.093 | 0.58 | 0.82 | 0.03 | fail | 1.16 | epochs=2, p_none_pair=0.25, public_frac=0.33 |
| v3-8b-s0/01-trial-1 | Qwen3-8B-Base | 0 | 0.840 | 0.741 | 0.375 | 0.082 | 0.61 | 0.83 | 0.02 | fail | 1.63 | accum=4, epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| lowdrift-4b-v4/00-trial-0 | Qwen3-4B-Base | 0 | 0.841 | 0.738 | 0.377 | 0.067 | 0.48 | 0.83 | 0.10 | fail | 2.02 | epochs=2, lora=8, lora_targets=attn, lr=0.0001, p_none_pair=0.25 |
| lowdrift-4b-v4/02-trial-2 | Qwen3-4B-Base | 0 | 0.839 | 0.738 | 0.369 | 0.038 | 0.52 | 0.83 | 0.08 | fail | 1.11 | lora_targets=attn, p_none_pair=0.25 |
| v4-4b-baseline/01-trial-1 | Qwen3-4B-Base | 1 | 0.834 | 0.735 | 0.373 | 0.052 | 0.47 | 0.85 | 0.02 | fail | 2.84 | epochs=2, p_none_pair=0.25 |
| mix-4b-knobs/02-trial-2 | Qwen3-4B-Base | 0 | 0.846 | 0.732 | 0.398 | 0.073 | 0.53 | 0.85 | 0.00 | fail | 3.23 | epochs=2, option_isolation=1, p_none_pair=0.25 |
| mix-4b-v4-b/01-trial-1 | Qwen3-4B-Base | 0 | 0.833 | 0.732 | 0.379 | 0.027 | 0.56 | 0.72 | 0.03 | fail | 1.54 | p_none_pair=0.25 |
| q35-4b-nopolicy/00-trial-0 | Qwen3.5-4B-Base | 1 | 0.844 | 0.730 | 0.423 | 0.134 | 0.52 | 0.90 | 0.02 | fail | 3.98 | epochs=2, lr=5e-05, p_none_pair=0.25, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,imdb,amazon |
| auto-4b-r1/00-trial-0 | Qwen3-4B-Base | 0 | 0.841 | 0.729 | 0.403 | 0.084 | 0.48 | 0.87 | 0.00 | fail | 2.69 | epochs=2, lora_targets=all, lr=3e-05, option_isolation=1, p_none_pair=0.25 |
| knowledge-4b-v6/00-trial-0 | Qwen3-4B-Base | 0 | 0.865 | 0.726 | 0.406 | 0.099 | 0.53 | 0.90 | 0.03 | fail | 3.17 | epochs=2, p_none_pair=0.25 |
| v3-data-capacity-s0/02-trial-2 | Qwen3-4B-Base | 0 | 0.824 | 0.721 | 0.409 | 0.088 | 0.45 | 0.82 | 0.03 | fail | 0.89 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,legacy_policy |
| v3-data-capacity-s1/02-trial-2 | Qwen3-4B-Base | 1 | 0.826 | 0.721 | 0.414 | 0.116 | 0.47 | 0.80 | 0.03 | fail | 0.94 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,legacy_policy |
| mix-4b-v4-b/00-trial-0 | Qwen3-4B-Base | 0 | 0.834 | 0.718 | 0.437 | 0.111 | 0.50 | 0.88 | 0.00 | fail | 2.98 | epochs=2, p_none_pair=0.25, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| recipe-4b-r1/02-trial-2 | Qwen3-4B-Base | 0 | 0.823 | 0.718 | 0.384 | 0.029 | 0.44 | 0.73 | 0.00 | fail | 1.07 | lr=0.0001, option_isolation=1, p_none_pair=0.25, public_frac=0.5 |
| mix-4b-v5/01-trial-1 | Qwen3-4B-Base | 0 | 0.858 | 0.713 | 0.408 | 0.072 | 0.50 | 0.93 | 0.00 | fail | 3.03 | p_none_pair=0.25, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| v4-4b-baseline/00-trial-0 | Qwen3-4B-Base | 0 | 0.849 | 0.704 | 0.463 | 0.113 | 0.58 | 0.93 | 0.02 | fail | 3.02 | epochs=2, p_none_pair=0.25 |
| mix-4b-knobs/00-trial-0 | Qwen3-4B-Base | 0 | 0.850 | 0.686 | 0.435 | 0.076 | 0.45 | 0.92 | 0.02 | fail | 3.09 | epochs=2, p_none_pair=0.25, synthetic_repeat=3 |
| night2-08b-du2/00-trial-0 | Qwen3.5-0.8B-Base | 1 | 0.825 | 0.652 | 0.499 | 0.099 | 0.42 | 0.83 | 0.02 | fail | 0.6 | data=evals/night2/dates_unknowable.jsonl, init_from=jaredpalmer/kev-0.8b, lr=4e-05, p_none_pair=0.25, replay=2000 |
| q35-08b/02-trial-2 | Qwen3.5-0.8B-Base | 2 | 0.829 | 0.643 | 0.513 | 0.096 | 0.38 | 0.83 | 0.02 | fail | 1.57 | epochs=2, lr=0.0001, p_none_pair=0.25 |
| q35-08b/01-trial-1 | Qwen3.5-0.8B-Base | 1 | 0.820 | 0.634 | 0.516 | 0.090 | 0.44 | 0.85 | 0.05 | fail | 1.48 | epochs=2, lr=0.0001, p_none_pair=0.25 |
| q35-08b/00-trial-0 | Qwen3.5-0.8B-Base | 0 | 0.817 | 0.622 | 0.508 | 0.084 | 0.27 | 0.88 | 0.00 | fail | 1.58 | epochs=2, lr=0.0001, p_none_pair=0.25 |
| ablation-v2/07-trial-7 | Qwen3-0.6B-Base | 1 | 0.793 | 0.621 | 0.532 |  | 0.07 | 0.74 | 0.04 | pass | 0.31 | epochs=2 |
| v7-06b/02-trial-2 | Qwen3-0.6B-Base | 2 | 0.801 | 0.620 | 0.536 | 0.108 | 0.08 | 0.80 | 0.07 | fail | 0.8 | epochs=2, lr=0.0001, p_none_pair=0.25 |
| ablation-v2/06-trial-6 | Qwen3-0.6B-Base | 0 | 0.816 | 0.620 | 0.525 |  | 0.03 | 0.76 | 0.06 | pass | 0.34 | epochs=2 |
| v7-06b/00-trial-0 | Qwen3-0.6B-Base | 0 | 0.797 | 0.613 | 0.555 | 0.111 | 0.05 | 0.82 | 0.02 | fail | 0.95 | epochs=2, lr=0.0001, p_none_pair=0.25 |
| arch-06b/06-trial-6 | Qwen3-0.6B-Base | 0 | 0.799 | 0.610 | 0.519 | 0.062 | 0.14 | 0.85 | 0.02 | fail | 0.31 | epochs=2, lr=0.0001, p_none_pair=0.25 |
| ablation-v2/04-trial-4 | Qwen2.5-0.5B | 0 | 0.742 | 0.605 | 0.501 |  | 0.05 | 0.49 | 0.12 | fail | 0.27 | epochs=2 |
| v4-06b-hardened/02-trial-2 | Qwen3-0.6B-Base | 2 | 0.800 | 0.605 | 0.545 | 0.066 | 0.06 | 0.85 | 0.02 | fail | 0.8 | epochs=2, p_none_pair=0.25 |
| v7-06b/01-trial-1 | Qwen3-0.6B-Base | 1 | 0.788 | 0.605 | 0.533 | 0.084 | 0.14 | 0.77 | 0.05 | fail | 0.83 | epochs=2, lr=0.0001, p_none_pair=0.25 |
| arch-06b/03-trial-3 | Qwen3-0.6B-Base | 0 | 0.799 | 0.599 | 0.538 | 0.050 | 0.19 | 0.82 | 0.00 | fail | 0.54 | epochs=2, option_isolation=1, p_none_pair=0.25, special_embeddings=1 |
| v3-data-capacity-s1/00-trial-0 | Qwen3-0.6B-Base | 1 | 0.760 | 0.599 | 0.549 | 0.082 | 0.06 | 0.68 | 0.10 | fail | 0.36 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,legacy_policy |
| ablation-v2/01-kev2 | legacy |  | 0.725 | 0.598 | 0.525 |  | 0.00 | 0.69 | 0.06 | pass | 0.07 | defaults |
| v4-06b-hardened/00-trial-0 | Qwen3-0.6B-Base | 0 | 0.805 | 0.598 | 0.521 | 0.052 | 0.11 | 0.78 | 0.02 | fail | 0.8 | epochs=2, p_none_pair=0.25 |
| auto-06b-r1/02-trial-2 | Qwen3-0.6B-Base | 0 | 0.807 | 0.596 | 0.542 | 0.067 | 0.03 | 0.83 | 0.00 | fail | 1.15 | epochs=2, p_none_pair=0.25, perm_kl=0.5 |
| v3-data-capacity-s1/01-trial-1 | Qwen3-0.6B-Base | 1 | 0.750 | 0.596 | 0.532 | 0.044 | 0.03 | 0.67 | 0.05 | fail | 0.36 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| arch-06b/05-trial-5 | Qwen3-0.6B-Base | 0 | 0.794 | 0.595 | 0.500 | 0.030 | 0.19 | 0.83 | 0.00 | fail | 0.31 | epochs=2, lr=0.0003, p_none_pair=0.25 |
| v4-06b-hardened/01-trial-1 | Qwen3-0.6B-Base | 1 | 0.793 | 0.595 | 0.579 | 0.082 | 0.09 | 0.80 | 0.03 | fail | 0.83 | epochs=2, p_none_pair=0.25 |
| v3-data-capacity-s0/01-trial-1 | Qwen3-0.6B-Base | 0 | 0.757 | 0.593 | 0.531 | 0.064 | 0.06 | 0.67 | 0.03 | fail | 0.31 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| arch-06b/01-trial-1 | Qwen3-0.6B-Base | 0 | 0.812 | 0.590 | 0.537 | 0.043 | 0.09 | 0.82 | 0.02 | fail | 0.53 | epochs=2, p_none_pair=0.25, special_embeddings=1 |
| arch-06b/02-trial-2 | Qwen3-0.6B-Base | 0 | 0.805 | 0.590 | 0.559 | 0.066 | 0.14 | 0.80 | 0.02 | fail | 0.32 | epochs=2, head_dim=1024, p_none_pair=0.25 |
| auto-06b-r1/01-trial-1 | Qwen3-0.6B-Base | 0 | 0.808 | 0.590 | 0.542 | 0.053 | 0.14 | 0.85 | 0.03 | fail | 0.81 | epochs=2, p_none_pair=0.25, special_embeddings=0 |
| auto-06b-r1/07-trial-7 | Qwen3-0.6B-Base | 0 | 0.806 | 0.588 | 0.563 | 0.081 | 0.06 | 0.87 | 0.00 | fail | 0.78 | epochs=2, p_none_distract=0.25 |
| auto-06b-r1/00-trial-0 | Qwen3-0.6B-Base | 0 | 0.820 | 0.587 | 0.560 | 0.078 | 0.06 | 0.83 | 0.00 | fail | 0.8 | epochs=2, p_none_pair=0.25 |
| arch-06b/00-trial-0 | Qwen3-0.6B-Base | 0 | 0.800 | 0.581 | 0.587 | 0.098 | 0.06 | 0.82 | 0.00 | fail | 0.33 | epochs=2, option_isolation=1, p_none_pair=0.25 |
| ablation-v2/00-kev-0.5b | legacy |  | 0.712 | 0.575 | 0.514 |  | 0.20 | 0.62 | 0.06 | pass | 0.07 | defaults |
| auto-06b-r1/06-trial-6 | Qwen3-0.6B-Base | 0 | 0.816 | 0.575 | 0.514 | 0.047 | 0.05 | 0.80 | 0.02 | fail | 1.13 | epochs=2, option_isolation=0, p_none_pair=0.25, perm_kl=0.2 |
| q35-smoke/00-trial-0 | Qwen3.5-0.8B-Base | 0 | 0.733 | 0.575 | 0.545 | 0.037 | 0.02 | 0.42 | 0.08 | fail | 0.51 | lr=0.0001, p_none_pair=0.25, public_frac=0.1 |
| auto-06b-r1/05-trial-5 | Qwen3-0.6B-Base | 0 | 0.771 | 0.572 | 0.535 | 0.034 | 0.02 | 0.72 | 0.08 | fail | 0.51 | p_none_pair=0.25 |
| auto-06b-r1/03-trial-3 | Qwen3-0.6B-Base | 0 | 0.768 | 0.569 | 0.515 | 0.020 | 0.14 | 0.77 | 0.03 | fail | 0.93 | epochs=2, lora=64, ord_w=0.5, p_none_pair=0.25 |
| ablation-v2/03-trial-3 | Qwen2.5-0.5B | 0 | 0.722 | 0.562 | 0.590 |  | 0.00 | 0.64 | 0.10 | fail | 0.2 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb |
| auto-06b-r1/04-trial-4 | Qwen3-0.6B-Base | 0 | 0.763 | 0.553 | 0.516 | 0.003 | 0.08 | 0.80 | 0.03 | fail | 0.9 | epochs=2, lr=0.0005, ord_w=0.25, p_none_pair=0.25 |
| v3-data-capacity-s0/00-trial-0 | Qwen3-0.6B-Base | 0 | 0.722 | 0.546 | 0.584 | 0.023 | 0.03 | 0.75 | 0.05 | fail | 0.29 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,legacy_policy |
| ablation-v2/05-trial-5 | Qwen2.5-0.5B | 1 | 0.657 | 0.482 | 0.580 |  | 0.05 | 0.44 | 0.14 | fail | 0.22 | epochs=2 |
| ablation-v2/02-trial-2 | Qwen2.5-0.5B | 0 | 0.552 | 0.466 | 0.614 |  | 0.00 | 0.22 | 0.25 | pass | 0.15 | epochs=2, train_sources=banking77,boolq,agnews,mnli,sst5,yelp |
| mix-4b-v5/00-trial-0 | Qwen3-4B-Base | 0 | 0.581 | 0.463 | 0.631 | 0.030 | 0.00 | 0.48 | 0.30 | fail | 6.02 | epochs=2, p_none_pair=0.25 |
| v3-smoke-02/01-trial-1 | Qwen3-4B-Base | 0 | 0.421 | 0.349 | 1.140 | 0.363 | 0.00 | 0.10 | 0.70 | fail | 0.1 | train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,legacy_policy |
| v3-smoke-02/00-trial-0 | Qwen3-0.6B-Base | 0 | 0.474 | 0.341 | 0.918 | 0.174 | 0.00 | 0.10 | 0.90 | fail | 0.05 | train_sources=banking77,boolq,agnews,mnli,sst5,yelp,trec,dbpedia14,amazon,imdb,compositional |
| backbone-v1/00-kev-0.5b | legacy |  | 0.797 |  |  |  |  | 0.78 | 0.03 | pass | 0.03 | defaults |
| backbone-v1/01-trial-1 | Qwen2.5-0.5B | 0 | 0.688 |  |  |  |  | 0.42 | 0.14 | pass | 0.09 | epochs=2 |
| backbone-v1/02-trial-2 | Qwen3-0.6B-Base | 0 | 0.725 |  |  |  |  | 0.42 | 0.06 | pass | 0.15 | epochs=2 |
| backbone-v1/03-trial-3 | Qwen2.5-0.5B | 1 | 0.697 |  |  |  |  | 0.36 | 0.11 | fail | 0.11 | epochs=2 |
| backbone-v1/04-trial-4 | Qwen3-0.6B-Base | 1 | 0.721 |  |  |  |  | 0.22 | 0.17 | fail | 0.15 | epochs=2 |
| gatecheck-trial4/00-checkpoint | legacy |  | 0.721 |  |  |  |  | 0.22 | 0.17 | pass | 0.03 | defaults |
| kev-4b-transfer-v9/00-kev-4b | legacy |  | 0.729 |  |  |  |  | 0.58 | 0.06 | pass | 0.08 | defaults |
| kev-8b-transfer-v9/00-kev-8b | legacy |  | 0.747 |  |  |  |  | 0.61 | 0.00 | pass | 0.11 | defaults |
| research-smoke-runner-v2/00-trial-0 | Qwen2.5-0.5B | 0 | 0.389 |  |  |  |  | 0.33 | 1.00 | pass |  | ord_w=0.1, perm_frac=1, perm_kl=0.1 |
| research-smoke-runner-verified/00-trial-0 | Qwen2.5-0.5B | 0 | 0.389 |  |  |  |  | 0.33 | 1.00 | pass |  | accum=3, ord_w=0.1, perm_frac=1, perm_kl=0.1 |
| smoke/00-trial-0 | Qwen2.5-0.5B | 0 | 0.389 |  |  |  |  | 0.33 | 1.00 | pass | 0.04 | ord_w=0.1, perm_frac=1, perm_kl=0.1 |
| smoke-anchor/00-trial-0 | Qwen2.5-0.5B | 0 | 0.389 |  |  |  |  | 0.33 | 0.83 | pass |  | anchor=/tmp/anchors-smoke.json, anchor_w=0.5 |
| smoke-arch/00-trial-0 | Qwen2.5-0.5B | 0 | 0.222 |  |  |  |  | 0.17 | 0.00 | pass |  | head_dim=512, option_isolation=1, p_none_pair=0.5, special_embeddings=1 |
| smoke-hl/00-trial-0 | Qwen2.5-0.5B | 0 | 0.389 |  |  |  |  | 0.33 | 0.83 | pass |  | head_lr=0.002, weight_decay=0.1 |
| smoke-mix/00-trial-0 | Qwen2.5-0.5B | 0 | 0.389 |  |  |  |  | 0.33 | 0.83 | pass |  | public_frac=0.5, synthetic_repeat=2 |


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/liuliang/integration-99101995.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/news/62141)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/jiaoliu/management-55756237.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/anli/careers-64952845.html)
* [全息网络通信节点白名单-#005](https://www.yx-sf.com/news/29066)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/tuiguang/api-23527438.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/yanjiu/about-14714302.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/wiki/47778)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/zhinan/link-27259567.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/wenzhang/accessibility-33079771.html)
* [全息网络通信节点白名单-#011](https://www.yx-sf.com/wiki/50742)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/jianzhan/plugin-75898937.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/chuangxin/media-73964810.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/wiki/54235)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/fuwu/sync-63716093.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/zhizhu/chapter-82483817.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/56522)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/jianzhan/chapter-69432840.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/zhinan/performance-60355123.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/62954)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/sheji/support-10178930.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/xitong/logo-97887412.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/tech/26016)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/keji/security-17907535.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/pingtai/innovation-69595977.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/wiki/16527)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/yunsuan/layout-18309176.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/zhinan/retention-25433955.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/news/64509)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/jishu/profit-01583054.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/yingxiao/login-76454482.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/news/83652)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/qiye/collaborate-05292530.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/yingyong/logo-48447184.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/tech/26760)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/suanfa/video-23643014.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/jishu/data-71122699.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/news/81755)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/anfang/solution-44727840.html)
* [异步事件循环架构设计规范-#003](https://www.mw-wm.com/yinqing/section-69485260.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/wiki/10285)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/gongju/brand-93464319.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/jianzhan/reminder-73503258.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/tech/12435)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/zixun/wellness-97066034.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/keji/advertising-21015278.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/news/12779)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/tuiguang/automation-53988831.html)
* [多协议互联数据格式规范-#012](https://www.mw-wm.com/anli/accessibility-60897270.html)
* [安全边界与可信凭证规约手册-#013](https://www.yx-sf.com/news/22512)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/zhinan/project-46202100.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/wendang/presentation-55840104.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/tech/91395)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/shangye/customization-64591938.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/shangye/schedule-98363849.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/news/31028)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/wendang/resource-78842819.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/gongxiang/course-93658674.html)
* [异步事件循环架构设计规范-#022](https://www.yx-sf.com/news/40574)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/anfang/workshop-12921378.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/yanjiu/online-87737501.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/news/69726)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/gongxiang/internet-81651217.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/xitong/milestone-73243132.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/tech/7080)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/anli/device-12283844.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/guanjianci/market-68635697.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/tech/79289)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/gongsi/document-20745042.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/ziyuan/project-87139173.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/tech/98866)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/fenxi/growth-75182455.html)
* [多协议互联数据格式规范-#036](https://www.mw-wm.com/anli/local-66275726.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/wiki/60165)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/kuangjia/update-76611425.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/xitong/database-66551302.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/news/17064)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/shuju/design-61166185.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/chanpin/success-81513093.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/news/43252)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/tuiguang/integration-25581194.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/xitong/machine-85866074.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/tech/98228)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/fenxi/cloud-91359104.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/wangluo/account-67474183.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/tech/80293)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/gongju/sale-07114932.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/jianzhan/page-10924470.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/news/57965)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/yingxiao/wellness-80717099.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/wangluo/message-54386507.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/tech/12366)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/liuliang/deal-90858257.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/baogao/revenue-50591766.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/news/99837)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/kaifa/resolution-68445815.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/zhizhu/economy-84096673.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/946)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/yingyong/schedule-71315049.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/anli/status-36769497.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/tech/32613)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/jiaoliu/document-37056176.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/huodong/software-83345573.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/wiki/53276)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/chanpin/digital-77337675.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/chuangxin/team-66603391.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/wiki/140)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/kuangjia/subject-37116266.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/wangluo/retention-76680927.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/wiki/80960)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/wendang/resource-77483916.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/baogao/community-01293802.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/tech/93837)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/jianzhan/collaborate-31984322.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/anli/sport-94101070.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/tech/67308)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/yingxiao/objective-93999509.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/paiming/brand-22380281.html)
* [权威网络权重与收录基准-#008](https://www.yx-sf.com/wiki/74165)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/yinqing/research-83011363.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/yingxiao/alert-22448889.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/tech/96463)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/liuliang/recipe-04747161.html)
* [节点连通性与存活探测准则-#013](https://www.mw-wm.com/anli/tactic-07500765.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/news/66820)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/gongsi/social-46617001.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/zixun/goal-36765150.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/wiki/17426)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/shangye/growth-55030546.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/hezuo/music-15923094.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/tech/73020)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/pingce/event-94420779.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/peixun/page-58468494.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/news/40660)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/peixun/satisfaction-89975600.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/fenxi/business-57698993.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/wiki/70967)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/zhizhu/calculator-42893534.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/ziyuan/achievement-10557572.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/news/37272)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/anfang/interface-00081037.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/zhinan/movie-97914113.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/news/13478)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/guanjianci/account-84554464.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/gongju/engagement-13101823.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/tech/7167)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/gongsi/upload-57676469.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/jiaoliu/widget-59708862.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/news/36613)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/ziyuan/software-57768332.html)

</details>

