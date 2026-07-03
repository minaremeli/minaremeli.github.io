---
layout: post
categories: posts
title: "Correct Looks Better"
subtitle: "Pairwise Comparisons Reveal Accuracy Rankings"
featured-image: /images/post_pics/2026-06-16/maella.jpg
tags: [LLMs, pairwise comparisons, model evaluation, ranking, ICML]
date-string: JUNE 16, 2026
---

<center style="display: flex; justify-content: center; gap: 20px;">
<a href="https://arxiv.org/abs/2606.09409">Paper</a>
<span>·</span>
<a href="https://github.com/socialfoundations/correct-looks-better">Code</a>
<span>·</span>
<span>ICML 2026</span>
</center>

> **TL;DR:** Pairwise comparison-based ranking reflects the accuracy-based ranking of models on verifiable tasks.

## Motivation

When we evaluate large language models, we often care about ranking them in one way or another. If the task is well-defined and we have ground truth labels, we can rank them by accuracy, which reflects how well they perform on the task. However, benchmarks measuring accuracy are not our only option for ranking models. In [Chatbot Arena](https://lmarena.ai/)[^5] models are evaluated on a diverse set of open-ended tasks using pairwise comparisons. The concern is that pairwise judgments are heavily biased to select the output that *appears* to be correct rather than selecting the output that *is* correct [^1][^2][^3][^4]. So the question arises:

> **Can pairwise comparison-based ranking reflect task performance?**

We set out to answer this question by applying pairwise comparisons on a range of benchmarks, and checking whether the resulting ranking aligns with the accuracy-based ranking of the models.

## What we found

- **High alignment with accuracy-based ranking**: high rank and score correlation across 5 benchmarks.
- **Outperforms baseline in weak model setting**: using a weak model (a model that's not good at the task) directly as a judge (*LLM-as-judge*) is significantly worse than using it to collect pairwise comparisons.
- **Rankings robust to judge biases**: correcting for style [^1][^2] and self-preference bias [^3] had minor impact on the ranking.
- **Echo influences judgment on non-discriminative pairs**: on pairs where both candidate answers are correct (or both are incorrect), the judge tends to choose answers that do not have any sequence repetition (*echo*). 

<figure>
  <img src="/images/post_pics/2026-06-16/fig1.png" style="display:block; margin-left:auto; margin-right:auto; width:70%">
  <figcaption><center><i>Accuracy (x-axis) vs. Bradley-Terry score (y-axis). Each point represents a model evaluated on a specific benchmark. Bottom: judge model accuracy.</i></center></figcaption>
</figure>


### Main results

Preference-based rankings align closely with accuracy-based rankings across all five benchmarks.
We observe both high rank correlation ($$ρ > 0.9$$) and high score correlation ($$r > 0.87$$) across all judge models.
What this means in practice is that

> on 4 out of 5 benchmarks, only **6-9%** of model pairs need to be swapped to recover the accuracy-based ranking.

<figure>
    <img src="/images/post_pics/2026-06-16/rank-distance.png" style="display:block; margin-left:auto; margin-right:auto">
    <figcaption><center><i>Accuracy-based ranking (left) vs. Bradley-Terry based ranking (right). Bottom: Kendall's distance.</i></center></figcaption>
</figure>

### Direct Judge Baseline

*Is it better to use the model directly as a judge?*

A natural alternative to pairwise comparisons is to use the judge to directly classify each answer as correct or incorrect — this is what we call the *Direct Judge* baseline. This is a strictly harder task: the judge must assess answers in isolation, without the relative signal a comparison provides. A weak judge that struggles with individual classification may still reliably pick the better of two answers.
When the selected judge does well on the task, both the Bradley-Terry and the Direct Judge approach perform well. However,

> when the judge is weak, the Direct Judge *doubles* the rank distance on SimpleQA ($$K_D=0.2 \rightarrow 0.382$$).

<figure>
    <img src="/images/post_pics/2026-06-16/baseline.png" style="display:block; margin-left:auto; margin-right:auto; width:60%">
    <figcaption><center><i>Rank correlation of "strong judge" (03, 59.3% acc.) vs "weak judge" (gpt-oss-20b, 4.9% acc.) on SimpleQA.</i></center></figcaption>
</figure>

In other words, Bradley-Terry outperforms the Direct Judge when the judge model has low accuracy on the task. This robustness may prove useful in frontier settings, where it's difficult to say whether the selected judge is "weak" or "strong".

### Judge bias

*Does rank correlation improve after bias correction?*

Style and self-preference biases are well-documented in pairwise evaluation — judges tend to favour longer, more formatted answers (*style bias*), or outputs from their own model family (*self-preference bias*). While such biases may affect absolute scores, their effect on relative model rankings is less clear. We corrected for both and found that

> bias correction has a minor effect on rank correlation.

<figure>
    <img src="/images/post_pics/2026-06-16/bias.png" style="display:block; margin-left:auto; margin-right:auto; width:60%">
    <figcaption><center><i>Rank correlation after bias correction (original in red). m = number of ranked models. </i></center></figcaption>
</figure>

Importantly, this does not imply the *absence* of bias — only that such biases have limited impact on model ranking in our evaluation.

### Echo

*What drives the signal in non-discriminative pairs?*

On 60% of pairs, correctness provides no discriminating signal. We call these *non-discriminative* pairs, where both answers are either correct or incorrect. The judge has to rely on superficial cues. And yet, we find that on such pairs rank correlation is still surprisingly high. One strong cue we identified is *echo*: a failure mode where a model fails to stop after its final answer and repeats content, such as phrase or sequence repetition.

> We found echo to be a *causal driver* of judge preference: $$P(A>B)$$ drops from 0.81 to 0.29 ($$\Delta=−0.52$$) depending on which answer has echo. This effect *disappears* on discriminative pairs ($$\Delta=−0.07$$).


<figure>
    <img src="/images/post_pics/2026-06-16/echo.png" style="display:block; margin-left:auto; margin-right:auto; width:60%">
    <figcaption><center><i>Judge preference conditional on echo on BBH. Non-discriminative (left) vs discriminative pairs (right). </i></center></figcaption>
</figure>

## Closing thoughts

Of course, it's important that our model rankings are not influenced by subjective biases. On questions where some answers are objectively better, we found that pairwise comparisons (and the resulting rankings) often indeed reflect the objective choice (picking the model with higher accuracy). This means that on *discriminative* pairs the judge picks the correct answer. On pairs where this objective choice is not possible, however, the judge falls back to appearance-based cues — it avoids picking answers that contain *echo* — which are also correlated with higher model accuracy. This doesn't mean the biases can be ignored: they could still be exploited adversarially to inflate a model's ranking.

---
_**Acknowledgements**_

_Cover image [source](https://artvee.com/dl/design-for-a-ceiling-3#00). Drawing by Mariano Salvador Maella._

[^1]: Y. Dubois, P. Liang, T. Hashimoto. "Length-controlled AlpacaEval: A simple debiasing of automatic evaluators." ICML 2024.
[^2]: T. Li, A. Angelopoulos, W. Chiang. "Does style matter? Disentangling style and substance in Chatbot Arena." LMSYS Blog, 2024.
[^3]: K. Wataoka, T. Takahashi, R. Ri. "Self-preference bias in LLM-as-a-judge." NeurIPS Safe Generative AI Workshop, 2024.
[^4]: G.H. Chen, S. Chen, Z. Liu, F. Jiang, B. Wang. "Humans or LLMs as the judge? A study on judgement bias." EMNLP 2024.
[^5]: W. Chiang, L. Zheng, Y. Sheng, et al. "Chatbot Arena: An open platform for evaluating LLMs by human preference." ICML 2024.
