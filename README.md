# PACE: Perceptual Agentic Collaborative Evolution

### An Evolutionary Agentic Approach for Open-ended Image Quality Perception

[![Paper](https://img.shields.io/badge/Paper-PDF-B31B1B.svg)](docs/assets/pace.pdf)
[![Project Page](https://img.shields.io/badge/Project-Page-blue.svg)](https://2kxx.github.io/PACE/)
[![Code](https://img.shields.io/badge/Code-Releasing%20soon-yellow.svg)](#-code-and-data-release)
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)](LICENSE)

Zhenchen Tang<sup>1,2</sup>, Bo Peng<sup>1,2</sup>, Zichuan Wang<sup>1,2</sup>,
Songlin Yang<sup>3</sup>, Leilei Cao<sup>4</sup>, Fengjie Zhu<sup>4</sup>,
Jing Dong<sup>1</sup>

<sup>1</sup>New Laboratory of Pattern Recognition, Institute of Automation, Chinese Academy of
Sciences &nbsp;·&nbsp; <sup>2</sup>School of Artificial Intelligence, University of Chinese
Academy of Sciences &nbsp;·&nbsp; <sup>3</sup>The Hong Kong University of Science and
Technology &nbsp;·&nbsp; <sup>4</sup>Transsion

> **Paper:** [`docs/assets/pace.pdf`](docs/assets/pace.pdf) &nbsp;|&nbsp;
> **Project page:** https://2kxx.github.io/PACE/ &nbsp;|&nbsp;
> **Code:** will be released upon publication (see
> [Code and data release](#-code-and-data-release)).

---

## 🔎 Overview

Generative models are rapidly expanding image quality assessment (IQA) beyond traditional
fidelity factors (blur, noise, compression) to emerging dimensions such as physical
plausibility, text-rendering correctness, tactile satisfaction, or human vitality. Existing IQA
models rely on fixed quality definitions and heavy supervision, so they struggle with this
long tail of subjective, context-dependent dimensions.

We identify **holistic bias** as the key failure mode: when asked to score an unseen dimension,
MLLMs fall back to generic quality priors, so a globally attractive image receives a high
dimension-specific score even when it clearly fails that dimension. We quantify this effect with
the **Holistic Override Rate (HOR)** — the fraction of images rated *low* by humans on the
target dimension that receive incorrectly *high* predictions:

$$\mathrm{HOR} = \frac{\sum_i \mathbb{I}(y_i \le 3.0 \wedge \hat{y}_i \ge 4.0)}{\sum_i \mathbb{I}(y_i \le 3.0)} \times 100\%$$

**PACE** addresses this by treating open-ended IQA as **explicit protocol construction** instead
of direct score prediction: collaborative agents evolve verifiable VQA probes for the target
dimension, a one-time human-in-the-loop calibration fixes the human score range, and a dual-track
scoring mechanism produces continuous, human-aligned scores. The whole framework is
**training-free** — no parameters of the MLLM backbone are ever updated.

## ✨ Highlights

| | Direct MLLM scoring | **PACE** |
| --- | --- | --- |
| HOR, 6 open-ended dimensions | 44.4 % | **8.6 %** |
| HOR, *Tactile Satisfaction* / *Human Vitality* | 58.8 % | **2.9 %** |
| Tier-4 open-ended PLCC / SRCC | 0.493 / 0.501 (Qwen2.5-VL) | **0.714 / 0.666** |
| Human input per new dimension | large-scale annotation | **4 annotated images** |
| Inference cost after protocol reuse | 0.12 s / image | 0.67 s / image (**33.8×** faster than first-time evolution) |

* Training-free multi-agent framework: a **Planning** agent decomposes the dimension, a
  **Visualizer** agent writes VQA probes, a **Critic** agent filters them, and a **Generator**
  agent synthesizes calibration anchors.
* Reusable knowledge: evolved protocols are stored in a RAG memory bank, so familiar dimensions
  are served by a fast-thinking branch while only genuinely new dimensions trigger the slow
  evolutionary pipeline.
* Backbone-agnostic: Qwen2.5-VL (7B) by default, with consistent gains when the pipeline is
  re-run on mPLUG-Owl2.
* Evaluated across a four-tier hierarchy: traditional fidelity benchmarks, structural/logical
  integrity, context-aware aesthetics, and zero-training-data open-ended dimensions.

## 🧠 Method at a glance

1. **Fast-and-slow thinking routing.** The target dimension is embedded (all-MiniLM-L6-v2) and
   matched against the memory bank. Above $\theta = 0.8$ cosine similarity, PACE reuses existing
   experts or a previously evolved protocol; otherwise it constructs a new one.
2. **Multi-agent perceptual evolution.** The Planning Agent decomposes the concept into 3–5
   observable, minimally overlapping sub-dimensions; the Visualizer Agent turns each of them into
   verifiable VQA probes with discrete options and score mappings; the Critic Agent rejects
   subjective, redundant, non-observable, or inverted-mapping probes in up to three rounds.
3. **One-time HITL calibration.** The Generator Agent converts the finalized protocol into
   positive/negative prompts and synthesizes four candidate images. Human ratings of these images
   define the reusable high/low anchors and the output range $[Y_{low}, Y_{high}]$ of the new
   dimension — **four annotated images per dimension, no training.**
4. **Dual-track continuous scoring.** *Track 1 (absolute perception)* averages the logit-softmax
   expectations over all probes into $\alpha_1$; *Track 2 (relative perception)* places the image
   on a 5-rank scale defined by the calibrated anchors and converts the rank logits into
   $\alpha_2$. The fused coefficient $\alpha_{final} = \omega_1\alpha_1 + \omega_2\alpha_2$ (a
   single global setting, $\omega_1 = \omega_2 = 0.5$) is linearly projected onto the anchor range.
   Working with logits instead of emitted text keeps predictions continuous and preserves rank
   correlation with human MOS.
5. **Tool-augmented observation.** When a protocol refers to high-frequency evidence (noise, film
   grain, texture, edges), PACE automatically invokes **crop-and-zoom** so that local defects
   become visible to the MLLM.

## 📊 Main results

**Tier 4 — zero-training-data open-ended dimensions (PLCC / SRCC).** Six newly defined
dimensions: Text Rendering Fidelity, Lighting Consistency, Tactile Satisfaction, Surrealist
Coherence, Human Vitality, Cinematic Narrative.

| Methods | Typo | Light | Tactile | Dream | Portrait | Cine | OVERALL |
| --- | --- | --- | --- | --- | --- | --- | --- |
| CLIP-IQA | −0.142 / −0.240 | 0.128 / 0.051 | 0.095 / 0.090 | 0.430 / 0.363 | 0.423 / 0.416 | 0.624 / 0.637 | 0.260 / 0.219 |
| PickScore | −0.073 / −0.028 | 0.043 / 0.015 | 0.008 / −0.002 | 0.675 / 0.595 | 0.225 / 0.159 | 0.611 / 0.649 | 0.248 / 0.231 |
| ImageReward | −0.104 / −0.176 | −0.462 / −0.205 | 0.071 / 0.116 | 0.746 / 0.661 | −0.294 / −0.216 | 0.736 / 0.714 | 0.115 / 0.149 |
| HPSv2 | −0.013 / 0.062 | −0.257 / −0.161 | 0.388 / 0.437 | 0.697 / 0.725 | −0.010 / −0.038 | 0.617 / 0.609 | 0.237 / 0.272 |
| Q-Align | 0.461 / 0.608 | −0.187 / −0.211 | 0.312 / 0.386 | 0.009 / 0.155 | −0.058 / 0.125 | 0.487 / 0.519 | 0.170 / 0.264 |
| Qwen2.5-VL | 0.339 / 0.480 | −0.088 / −0.057 | 0.604 / 0.644 | **0.874** / 0.758 | 0.535 / 0.468 | 0.694 / 0.710 | 0.493 / 0.501 |
| **PACE (Slow)** | **0.684 / 0.673** | **0.348 / 0.204** | **0.838 / 0.796** | 0.849 / **0.785** | **0.806 / 0.790** | **0.759 / 0.745** | **0.714 / 0.666** |

Most baselines exhibit polarity inversion (negative SRCC) on at least one dimension — evidence
that generic quality priors override dimension-specific reasoning. PACE keeps continuous,
human-aligned scoring across all six unseen dimensions.

**Tier 1 — traditional IQA benchmarks (overall trends).**

| Methods | KonIQ | SPAQ | KADID | LIVE-Wild | AGIQA-3K | CSIQ |
| --- | --- | --- | --- | --- | --- | --- |
| Q-Align | 0.941 / 0.940 | 0.886 / 0.887 | 0.674 / 0.684 | 0.853 / 0.860 | 0.772 / 0.735 | 0.785 / 0.737 |
| DeQA-Score | 0.953 / 0.941 | 0.895 / 0.896 | 0.694 / 0.687 | 0.892 / 0.879 | 0.809 / 0.729 | 0.787 / 0.744 |
| Q-Scorer | 0.959 / 0.948 | 0.898 / 0.898 | 0.676 / 0.671 | 0.889 / 0.870 | 0.821 / 0.736 | 0.796 / 0.746 |
| Qwen2.5-VL (backbone) | 0.737 / 0.692 | 0.855 / 0.860 | 0.576 / 0.522 | 0.625 / 0.615 | 0.813 / 0.744 | 0.724 / 0.679 |
| **PACE (Slow)** | 0.841 / 0.783 | 0.900 / 0.893 | 0.782 / 0.776 | 0.807 / 0.760 | 0.821 / 0.758 | 0.735 / 0.697 |
| **PACE (Fast)** | **0.962 / 0.950** | **0.930 / 0.929** | **0.926 / 0.922** | **0.907 / 0.892** | **0.822 / 0.751** | **0.880 / 0.835** |

Table 2 (structural integrity: HandEval, PIPAL) and Table 3 (context-aware aesthetics: TAD66k
subsets) are reported in the paper; PACE reaches 0.572 / **0.575** on HandEval and improves every
TAD66k subset over the direct MLLM baseline.

Full tables, ablations, latency measurements, inter-annotator reliability (ICC(2,k) = 0.859 over
180 images), and the qualitative case studies are available in the
[paper](docs/assets/pace.pdf) and on the [project page](https://2kxx.github.io/PACE/).


## 🗂 Code and data release

**The source code and the Tier-4 data** will be released upon publication and will contain.

## 📝 Citation

```bibtex
@misc{tang2026pace,
  title  = {An Evolutionary Agentic Approach for Open-ended Image Quality Perception},
  author = {Tang, Zhenchen and Peng, Bo and Wang, Zichuan and Yang, Songlin and
            Cao, Leilei and Zhu, Fengjie and Dong, Jing},
  year   = {2026},
  note   = {Preprint}
}
```

## 📄 License and acknowledgements

Released under the [MIT License](LICENSE) (documentation and figures).

We thank the authors of **Qwen2.5-VL** and **mPLUG-Owl2** (multimodal backbones), **Q-Scorer**
and **DeQA-Score** (specialized IQA experts), and **Stable Diffusion** (anchor generation) for
making their models publicly available.
