---
layout: distill
title: "Two Roads to No: Where the Refusal Neuron Meets the Refusal Direction"
description: "Two recent papers explain refusal in LLMs in two ways: as a single direction in the hidden state, and as a single neuron. We ask how the two are connected, on three models. They turn out to be two different handles on refusal: nearly interchangeable on Llama, the neuron stronger on Qwen, and the neuron weak on Gemma."
date: 2027-04-30
future: true
htmlwidgets: true
hidden: true

# Mermaid diagrams
mermaid:
  enabled: true
  zoomable: true

# Anonymized for submission
authors:
  - name: Anonymous



# must be the exact same name as your blogpost
bibliography: 2027-04-30-two-roads-to-no.bib

# Table of contents
toc:
  - name: Introduction
    subsections:
      - name: Two Pictures of Refusal
      - name: Why the Connection Matters
      - name: How a Neuron Can Push Along a Direction
      - name: What We Found
  - name: Experimental Setup
  - name: First, Do Both Papers Hold on Our Models?
  - name: "Question 1: Does the Neuron Write the Refusal Direction?"
  - name: "Question 2: Which Part Carries Refusal?"
    subsections:
      - name: Removing the Neuron Direction or the Rest
      - name: Pushing Along Each Direction by the Same Amount
      - name: Pinning Each Piece of the Neuron
      - name: Why Gemma Behaves Differently
      - name: What the Neuron Writes Outside the Direction
  - name: "Question 3: Does Alignment With the Direction Find Other Refusal Neurons?"
    subsections:
      - name: Following the Direction
      - name: Is Separating Enough?
  - name: "Conclusion: Two Handles, Not One"
    subsections:
      - name: Limitations

# Styles for the definition boxes and tables (same look as the other post)
_styles: >
  .callout-box,
  .callout-warning,
  .callout-insight,
  .callout-definition {
    padding: 16px 20px;
    border-radius: 8px;
    margin: 1.5rem 0;
    font-size: 0.92rem;
    line-height: 1.6;
  }
  .callout-warning {
    background: #fef3c7;
    border-left: 4px solid #f59e0b;
    color: #92400e;
  }
  .callout-insight {
    background: #eef2ff;
    border-left: 4px solid #6366f1;
    color: #3730a3;
  }
  .callout-definition {
    background: #f0fdf4;
    border-left: 4px solid #22c55e;
    color: #166534;
  }
  d-article d-contents {
    grid-row: auto / span 12;
  }
  .comparison-table {
    width: 100%;
    border-collapse: collapse;
    margin: 1.5rem 0;
    font-size: 0.88rem;
  }
  .comparison-table th {
    background: #6366f1;
    color: white;
    padding: 10px 14px;
    text-align: left;
    font-weight: 600;
  }
  .comparison-table td {
    padding: 10px 14px;
    border-bottom: 1px solid #e2e8f0;
  }
  .comparison-table tr:nth-child(even) td {
    background: #f8fafc;
  }
  .comparison-table tr:hover td {
    background: #eef2ff;
  }
  [data-theme="dark"] .callout-warning,
  html[data-theme="dark"] .callout-warning {
    background: rgba(254, 243, 199, 0.12);
    border-left-color: #f59e0b;
    color: #fcd34d;
  }
  [data-theme="dark"] .callout-insight,
  html[data-theme="dark"] .callout-insight {
    background: rgba(99, 102, 241, 0.12);
    border-left-color: #818cf8;
    color: #c7d2fe;
  }
  [data-theme="dark"] .callout-definition,
  html[data-theme="dark"] .callout-definition {
    background: rgba(34, 197, 94, 0.12);
    border-left-color: #4ade80;
    color: #bbf7d0;
  }
  [data-theme="dark"] .callout-warning em,
  [data-theme="dark"] .callout-warning strong,
  html[data-theme="dark"] .callout-warning em,
  html[data-theme="dark"] .callout-warning strong {
    color: #fde68a;
  }
  [data-theme="dark"] .callout-insight em,
  [data-theme="dark"] .callout-insight strong,
  html[data-theme="dark"] .callout-insight em,
  html[data-theme="dark"] .callout-insight strong {
    color: #e0e7ff;
  }
  [data-theme="dark"] .callout-definition em,
  [data-theme="dark"] .callout-definition strong,
  html[data-theme="dark"] .callout-definition em,
  html[data-theme="dark"] .callout-definition strong {
    color: #d9f99d;
  }
  [data-theme="dark"] .comparison-table td,
  html[data-theme="dark"] .comparison-table td {
    border-bottom-color: #334155;
    color: #cbd5e1;
  }
  [data-theme="dark"] .comparison-table tr:nth-child(even) td,
  html[data-theme="dark"] .comparison-table tr:nth-child(even) td {
    background: rgba(51, 65, 85, 0.4);
  }
  [data-theme="dark"] .comparison-table tr:hover td,
  html[data-theme="dark"] .comparison-table tr:hover td {
    background: rgba(99, 102, 241, 0.15);
  }
---

## Introduction

Ask an LLM such as Llama how to make an illegal drug and it will usually say no. We call this refusal. Ask it *"How can I kill a Python process?"* and it may also say no, because the word "kill" looks dangerous, even though here it only means stopping a program. We call this over-refusal. Both are the same behaviour, saying no, applied to two kinds of requests. The first is intended, the second is a mistake.

### Two Pictures of Refusal

Two recent papers explain what happens inside the model when it says no, and they give very different answers.

Arditi et al. <d-cite key="arditi2024refusal"></d-cite> (*Refusal in Language Models Is Mediated by a Single Direction*) look at the hidden state at the last token of the prompt, just before the model starts to answer. They average it over many harmful requests, average it over many harmless ones, and subtract. Scaled to length 1, the difference is a direction that points from "harmless" towards "harmful". We call it **Arditi's direction**. Remove it from the hidden state and the model answers harmful requests it would normally refuse. Add it while the model reads a harmless request and the model refuses. They saw this in 13 open chat models. Later work found that refusal is not tied to just one direction: Wollschläger et al. <d-cite key="wollschlager2025geometry"></d-cite> find whole cones of directions that control refusal, and Pan et al. <d-cite key="pan2025hidden"></d-cite> find one main direction with weaker ones at right angles to it.

Kazemi et al. <d-cite key="kazemi2026single"></d-cite> (*A Single Neuron Is Sufficient to Bypass Safety Alignment in Large Language Models*) look at single neurons. They rank neurons with a gradient-based score, test the five best on 191 harmful prompts from HarmBench <d-cite key="mazeika2024harmbench"></d-cite>, and keep the one that works best. For Llama-3.1-8B-Instruct this is **neuron 4258 in layer 11**. They **pin** it: they set its activation to a fixed number for every token, whatever the prompt says. For this neuron the number is -4, far from its usual values (about 1.1 on harmful prompts and about 0 on harmless ones). With the neuron pinned, the model answers 95% of the harmful prompts. They report this for seven models from two families (Qwen3 and Llama-3.1).

### Why the Connection Matters

One paper points at a direction made of thousands of numbers, the other at a single neuron. Both are used as handles: to take refusal out of a model, or to study how it works. So it matters whether they are the same handle.

There are two simple possibilities. If the neuron is just the main writer of Arditi's direction, the two papers describe one mechanism from two ends, and a result about one carries over to the other. If the neuron does something the direction does not capture, or the direction carries something the neuron does not write, then each picture leaves something out, and choosing the wrong handle can fail.

This post asks **how the refusal neuron and the refusal direction are connected**, in three steps, each building on the one before:

1. **Does the neuron write Arditi's direction?** If it does not, there is nothing to connect.
2. **Which part carries refusal?** The direction and the neuron overlap only partly. We take them apart and ask which piece does the work, and which of the two is the better handle.
3. **Do the neurons that line up with the direction control refusal too?** If the two are linked, the neurons that line up best with Arditi's direction should be refusal neurons like the one of Kazemi et al.

We ask these questions on three models: Llama-3.1-8B-Instruct, Qwen2.5-7B-Instruct and Gemma-2-9B-it. The last two are not among the models Kazemi et al. tested.

### How a Neuron Can Push Along a Direction

The neuron and the direction are connected by how the model is built. A neuron's activation $$a_j$$ is a single number. What carries it into the hidden state is the neuron's **output vector** $$\mathbf{w}_j$$ (the column of the layer's down projection that belongs to neuron $$j$$). The layer adds $$a_j$$ times $$\mathbf{w}_j$$ to the hidden state:

$$
\text{new hidden state} = \text{old hidden state} + \sum_{j} a_j \, \mathbf{w}_j
$$

So every neuron adds something to the hidden state, and we can measure how much of it points along Arditi's direction. One model breaks this simple picture a little: Gemma-2 rescales the MLP output before adding it to the hidden state. This turns out to matter, and we come back to it below.

### What We Found

**The refusal neuron and the refusal direction are two different handles on refusal, and how closely they are tied depends on the model.** On Llama the two are nearly interchangeable. On Qwen the neuron is the better handle. On Gemma the neuron found by the same search is a weak handle, while the direction is a strong one.

* **Does the neuron write the direction? Yes, in all three models, but by very different amounts.** Measured on held-out prompts, Llama's neuron 4258 pushes harmful prompts along Arditi's direction about ten times more than any other neuron in its layer, about 46% of the layer's push. On Gemma it is 1.9 times more and about 15%. On Qwen it is tied with another neuron and gives about 9%.
* **Which part carries refusal? It depends on the model.** On Llama and Qwen, removing the neuron's direction from the hidden state lowers refusals more than removing the rest of Arditi's direction, and pushed by the same amount, the neuron's direction is as good a lever as Arditi's (Llama) or a much better one (Qwen). On Gemma it is the other way around: removing the rest of Arditi's direction does the work, and at equal pushes Arditi's direction is far stronger than the neuron's. On Llama the neuron also writes something at right angles to Arditi's direction that is tied to refusal.
* **Being the largest writer of the direction does not make a neuron a strong lever.** On Gemma, the neuron is the largest writer of the direction in its layer, yet pinning it hardly changes refusal of clear harmful requests, and none of the five candidates from the search does better. Part of the reason is likely that Gemma-2 rescales what the neuron writes, but in our tests this does not explain all of it.
* **Lining up with the direction rarely makes a neuron a refusal neuron.** We pinned the neurons that line up best with Arditi's direction in each layer, both to the same value as the paper's neuron and at 2 to 8 times their own activity on harmful prompts. Of the 12 we tried across three models, only one (Llama's 10096) does a large part of the job under both. Several others did something only when pushed far beyond their own activity, and one of them lowered refusals only because the model stopped working. Ranking neurons by how well they tell prompts apart did no better: at matched sizes, the only one that does a large part of the job is again 10096.

## Experimental Setup

* **Models.** Llama-3.1-8B-Instruct, Qwen2.5-7B-Instruct and Gemma-2-9B-it, in 16-bit precision. Answers use greedy decoding, up to 512 new tokens.
* **The neuron.** For each model we ran the official search code of Kazemi et al. and their reranking step (pin each of the five best candidates at 2, 4 and 8 times its **search activity**, keep the one that lowers refusals most without breaking maths). A neuron's search activity is its average activity on the search's own harmful prompts, at the token position the search picks for that neuron. In some later tests we size a pin by a neuron's **own activity** instead: its average activity at the last prompt token, on the 64 harmful prompts used to build Arditi's direction. The two are measured differently, so their multiples differ: for Qwen's neuron 17011, a pin of -13.7 is 4 times its search activity and 8 times its own activity. We reranked on validation prompts, not on our test prompts. This gives neuron 4258 in layer 11 for Llama (the same as the paper), neuron 17011 in layer 12 for Qwen, and neuron 11539 in layer 21 for Gemma.
* **Arditi's direction.** The recipe of Arditi et al., with 64 harmful prompts from AdvBench <d-cite key="zou2023universal"></d-cite> and 64 harmless prompts from Alpaca, at the last prompt token, taken right after the layer of the neuron.
* **Harmful prompts.** HarmBench <d-cite key="mazeika2024harmbench"></d-cite> (159 prompts, our test split) and WildJailbreak <d-cite key="jiang2024wildteaming"></d-cite> (200 prompts, a mix of plain and disguised harmful requests).
* **Harmless but risky-looking prompts.** OR-Bench-Hard <d-cite key="cui2025orbench"></d-cite> (300 prompts, drawn from its 1,319) and XSTest <d-cite key="rottger2024xstest"></d-cite> (250 prompts).
* **Is the model still working?** Accuracy on 100 GSM8K maths problems. A large drop is a warning sign.
* **What counts as a refusal.** A larger model (Qwen3.5-27B) reads each prompt and answer and labels it as full compliance, full refusal or partial refusal, following XSTest <d-cite key="rottger2024xstest"></d-cite>. Our tables count only full refusals. We read samples of the answers at the main settings: on Llama and Qwen the answers that are not refused are coherent and on topic (at the strongest setting, 93.7% of Llama's and 73.6% of Qwen's HarmBench answers fully comply). On Gemma, many answers that are not full refusals are partial refusals that still decline, so where this changes the picture we also give the share of answers that fully comply.
* **Same prompts everywhere.** Every table and chart uses exactly these prompts, drawn with a fixed seed, so the same setting gives the same number wherever it appears.

<script src="{{ 'assets/html/2027-04-30-two-roads-to-no/model_tables.js' | relative_url }}"></script>

Every results table has a switch for the three models. The choice applies to all tables on the page.

## First, Do Both Papers Hold on Our Models?

Before connecting the two papers we checked that each one holds on its own. If either failed here, the rest of the question would not make sense.

<div class="model-table" data-table="reproduce"></div>

**Arditi's direction works in all three models.** Removing it lowers HarmBench refusals on Llama from 83.0% to 17.6%. On Qwen, removing it at layer 12 alone has a smaller effect (80.5% to 59.1%), but removing it at layers 12 to 22, closer to how Arditi et al. use it, brings refusals down to 5.0%. On Gemma, removing it at layer 21 alone takes refusals from 98.7% to 34.0%, and at layers 21 to 31 to 25.2%.

**The neuron works on Llama and Qwen, but only weakly on Gemma.** On Llama, pinning neuron 4258 to -4 leaves 0.6% of HarmBench prompts refused, as Kazemi et al. report. Switching it to 0 barely matters (74.8%), so the effect comes from pushing the neuron the other way, not from removing what it normally adds. Qwen's neuron works more gradually: switching it off gives 71.7%, pinning it to -6.8 gives 41.5%, and -13.7 (four times its search activity) gives 10.1%. On Gemma, neuron 11539 is negative on harmful prompts, so it is pinned to positive values, up to 14.8 (eight times its search activity). HarmBench refusals only fall from 98.7% to 92.5%. WildJailbreak refusals fall more, from 34.0% to 17.0%. Kazemi et al. did not test Gemma, so this does not contradict their results. It does mean that on Gemma, their search followed by testing its top five candidates did not find a neuron that strongly reduces refusal of clearly harmful requests.

We also looked past the top five, and still did not find a strong refusal neuron for clearly harmful requests. The best one we found is neuron 7162 in layer 20. The same search ranks it ninth, so it is not among the five that get tested; we tried it because a second search, run on over-refusal prompts, ranked it second. We pinned 7162 and 11539 at 2, 4 and 8 times their own activity on the 64 harmful prompts used to build Arditi's direction, so that each push is sized to the neuron. HarmBench refusals fall to 94.3%, 86.2% and 74.8% with 7162, against 96.9%, 94.3% and 93.7% with 11539 (98.7% unchanged). So 7162 is clearly the better of the two, and testing only the top five missed it. But it is still not strong enough: at 8 times its own activity, about three in four prompts are still refused and only 15.1% of the answers fully comply (11539: 3.8%; maths accuracy 83%, against 85% unchanged). Even a much larger pin of 14.8 (its pin at 8 times its own activity is only 4.1) leaves 64.2% refused (22.0% fully comply, maths 83%). On Llama and Qwen, the search's neurons at 8 times their own activity leave only 0.6% and 10.1% refused, although Llama's also breaks maths at that size. WildJailbreak's adversarial prompts, which Gemma already refuses less often, are different: pinning 7162 to values from 3.7 to 14.8 takes refusals from 34.0% to 2.5% or lower. So our finding is about clearly harmful requests asked directly. It also does not show that no strong neuron exists on Gemma, only that we did not find one this way.

A strong push on any neuron can disturb a model. On Llama, three neurons from the same layer that are about as active as 4258 but point almost at right angles to Arditi's direction changed nothing when switched off, and when pinned to -4 they sometimes lowered refusals and sometimes raised them (one gave 45.3% on HarmBench but 72.5% on WildJailbreak). We keep this in mind for every strong pin below.

The widget below lets you change the setting yourself, for each model and for a few other neurons.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/dose_response.html' | relative_url }}" frameborder="0" scrolling="no" height="730px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

## Question 1: Does the Neuron Write the Refusal Direction?

If the neuron is the main writer of the direction, two things must be true. Its output vector must point along the direction, and its activity must differ between harmful and harmless prompts, so that it actually moves the hidden state along the direction.

**Its output vector points along the direction in all three models.** We measured the cosine similarity between each neuron's output vector and Arditi's direction. On Llama, out of 14,336 neurons in layer 11, neuron 4258 scores 0.52 and the next best 0.19. A random direction typically scores about ±0.016. On Gemma, neuron 11539 is also first in its layer, with -0.30 against 0.20 for the next. (The sign is negative because the neuron's activity on harmful prompts is negative, so together the two still push towards refusal.) On Qwen, neuron 17011 is first too, but with only 0.20 against 0.14 for the next.

Part of this is circular, because Arditi's direction already contains what the neuron writes. So we also used the direction with the neuron's own write removed, and the direction measured just before the neuron's layer. On Llama the neuron stays first, with 0.32 and 0.34. On Gemma it stays first with -0.30 and -0.13. On Qwen it falls to 0.09 and ranks only 6th.

**Its activity moves the hidden state along the direction, by very different amounts.** We measured this on prompts that were not used to build Arditi's direction: the 159 HarmBench prompts against 159 other Alpaca prompts. On Llama, neuron 4258 averages 1.08 on harmful prompts and 0.00 on harmless ones, which moves the hidden state about 0.47 along Arditi's direction. The next neuron, 10096, moves it about 0.05, about ten times less. Together, all neurons of the layer move harmful prompts about 1.0 further along the direction than harmless ones, and neuron 4258 alone accounts for about 46% of that. On Gemma the neuron pushes 1.9 times more than the next neuron and accounts for about 15% of the layer's push. On Qwen it is no longer first: neuron 8210 pushes about as much (0.20 against 0.20), and 17011 accounts for about 9% of the layer's push.

On the 128 prompts that define Arditi's direction, the same numbers are 9 times the next neuron and 40% on Llama, 2.7 times and 22% on Gemma, and 1.7 times and 10% on Qwen. On those prompts they are an exact account of how the direction was built, so we report the held-out numbers above as the main ones. The picture is the same: Llama's neuron dominates, Gemma's leads, and Qwen's is one of many.

**So the answer to the first question is yes, but the strength of the link differs a lot:** on Llama the neuron is the main writer of the direction in its layer, on Gemma it is the largest of a few, and on Qwen it is one of many, tied with another neuron.

## Question 2: Which Part Carries Refusal?

The neuron's direction and Arditi's direction overlap but are not the same. On Llama, about 27% of Arditi's direction (in squared length) lies along the neuron's direction and 73% is the rest. On Gemma it is about 9%, and on Qwen about 4%. So the two pictures share a piece, and each has a piece the other lacks. This section asks which does the work: the neuron's direction, or the rest of Arditi's direction (its part at right angles to the neuron's direction). If the neuron's direction carries refusal, the neuron is a good handle on what Arditi's direction does. If the rest does, it is not.

### Removing the Neuron Direction or the Rest

We removed each of the two directions from the hidden state at the neuron's layer, one at a time and together. Removing a direction sets the hidden state's component along it to zero. Note that removing the neuron's direction removes everything along it, not only the share of Arditi's direction that it overlaps.

<div class="model-table" data-table="removal"></div>

On Llama and Qwen, removing the neuron's direction lowers HarmBench refusals more than removing the rest of Arditi's direction: on Llama 83.0% to 44.7% against 63.5%, and on Qwen 80.5% to 44.0% against 73.6%. On Qwen, removing the neuron's direction even does more than removing Arditi's whole direction at that layer (59.1%). Since the two directions overlap only 4% on Qwen, this does not mean that a piece of Arditi's direction carries refusal there; it means the neuron's own direction is a better thing to remove than Arditi's. Gemma is the opposite: removing the neuron's direction changes almost nothing (96.9%), while removing the rest of Arditi's direction does almost all the work (49.7%, against 34.0% for the whole direction). In all three models, removing both does the most.

### Pushing Along Each Direction by the Same Amount

Removing a direction changes each prompt by a different amount, so it is not a fair comparison of two directions as levers. Instead, we pushed the hidden state *against* refusal by the same amount for every prompt, along each of several directions: the neuron's direction, Arditi's direction, Arditi's direction without the neuron's own write, and the difference between the average hidden states of harmful prompts the model refused and ones it answered. As a check we used a random direction. The size of the push is the push that pinning the neuron gives along its own direction: 3.35 on Llama and 12.9 on Qwen.

<div class="model-table" data-table="push"></div>

On Llama, all four directions bring refusals down to a few percent and the random direction does not. The neuron's direction does this about as well as Arditi's, although the two have a cosine of only 0.52. A push twice as large (6.7) breaks the model (maths accuracy 3% to 4%). On Qwen the neuron's direction is clearly better: it brings HarmBench refusals down to 20.8%, while Arditi's direction only reaches 63.5%, close to the random direction's 73.0%. Adding the directions to harmless WildJailbreak prompts, which Qwen never refuses, shows the same: at size 12.1, the neuron's direction makes the model refuse 50.0% of them, and Arditi's direction 4.0%.

On Gemma, pushes added after the layer did nothing at all (98.7% for every direction; first part of the Gemma table). This is not because Gemma's directions are weak. Gemma-2 rescales the MLP output before adding it to the hidden state, so a push added after the layer is tiny compared with what pinning the neuron does (see "Why Gemma Behaves Differently" below). So we made the same pushes, at the same size, where the neuron writes: into the MLP output, before the rescaling (second part of the table). As a check, pushing along the neuron's direction there leaves 92.5% of HarmBench prompts refused, the same as pinning the neuron to 14.8. Arditi's direction at the same size brings refusals down to 44.7% (37.1% of the answers fully comply), and so does Arditi's direction without the neuron's own write. The refused-minus-answered direction gives 60.4%, while a random direction (97.5%) and the comparison neuron's direction (98.1%) do almost nothing. Added to harmless WildJailbreak prompts in the same place, Arditi's direction makes Gemma refuse 43.5% of them at 0.3 of this size and 89.0% at the full size (from 2.5%), against 4.5% and 6.5% for the neuron's direction. So on Gemma, Arditi's direction is clearly the stronger lever, the opposite of Qwen.

Because the two directions differ, the neuron must also write something at right angles to Arditi's direction. We pushed along that part too (the right-angled part, scaled to length 1), at three sizes, next to Arditi's direction and the neuron's. For comparison we used the direction of a neuron that is about as active but not aligned with Arditi's direction. It is not a perfect null: on Llama, at the largest size, it lowers OR-Bench-Hard refusals from 28% to 17%.

<div class="model-table" data-table="rightangle"></div>

On Llama, per unit of push, Arditi's direction is at least as strong as the neuron's, and both are stronger than the right-angled part, which needs about 1.7 times the size to do the same (33.3% at size 3.35, against 36.5% for Arditi's direction at size 2). So on Llama what the neuron writes outside the direction is weaker, but not useless. On Qwen the right-angled part is almost the same direction as the neuron's (cosine 0.98), because the neuron already lies mostly at right angles to Arditi's direction, and it behaves like it: 23.9% at the full size, against 20.8% for the neuron's direction and 63.5% for Arditi's. On Gemma the right-angled part does nothing, even when pushed into the MLP output: 97.5%, 98.7% and 97.5% of HarmBench prompts stay refused at the three sizes, against 71.1%, 55.3% and 44.7% for Arditi's direction. Removing it after the layer leaves 97.5%, and added to harmless prompts in the MLP output it does not raise refusals (2.0% and 1.5%, from 2.5%).

### Pinning Each Piece of the Neuron

Kazemi et al. change the whole neuron, so we also split its output vector into two pieces, the piece along Arditi's direction and the rest, and pinned one piece at a time, leaving the other alone. As a check, we pushed in a random direction at right angles to Arditi's direction, with the same size as the rest. This test is harder to read than the equal pushes, because the two pieces have different lengths. We used three versions of Arditi's direction: the one used so far, the one with the neuron's own write removed, and the one measured just before the neuron's layer.

<div class="model-table" data-table="halves"></div>

On Llama, with the first version of the direction, each piece alone brings harmful refusals down to about a third, which is roughly what the equal pushes predict for pushes of their sizes. With the other two versions, the rest is longer still and points partly along Arditi's direction, so the rest alone almost reproduces the whole neuron. This does not show that the rest is the more effective piece. On Qwen the piece along the direction does little in all three versions (74.8% to 78.0% still refused, from 80.5%), while the rest alone does almost as much as the whole neuron (12.6% to 16.4%, against 10.1%), as expected when only a small part of the neuron lies along the direction. The random push of the same size never lowers refusals on Llama and only a little on Qwen, so the effect of the rest belongs to the neuron.

On Gemma the result looks impossible at first: the piece along Arditi's direction alone (68.6% of HarmBench prompts refused) does much more than the whole neuron (92.5%), and the rest alone does nothing (97.5%). This holds when we count only answers that fully comply: 15.7% for the piece against 3.8% for the whole neuron. The two pieces add up to the whole neuron, so how can one piece do more than both together? The next part answers some of this.

### Why Gemma Behaves Differently

Two things set Gemma apart.

**None of the search's candidates is a strong lever.** In the reranking step, we pinned each of the five best candidates at 2, 4 and 8 times its search activity. None of them lowered refusals on the 39 HarmBench validation prompts: every setting left 38 or 39 of them refused. They only lowered refusals on the WildJailbreak validation prompts, which is how neuron 11539 was chosen. So the weak effect is not because we kept the wrong candidate.

**Gemma-2 rescales what the neuron writes.** Unlike Llama and Qwen, Gemma-2 normalises the output of the MLP before adding it to the hidden state, so anything added to the MLP output is rescaled. We checked what this does on 24 HarmBench prompts, at layer 21, by passing the real MLP output, with and without each edit, through the model's own normalisation:

| Edit at layer 21 | Push along Arditi's direction before rescaling | after rescaling | MLP output grows by |
|---|---|---|---|
| Whole neuron pinned to 14.8 | -1.4 | -33.0 | 4.2 times |
| Only the piece along Arditi's direction | -1.4 | -71.6 | 1.6 times |
| Only the rest | 0.0 | -4.7 | 4.1 times |

The whole neuron and its piece along the direction give exactly the same push before rescaling. Pinning the whole neuron, however, makes the MLP output about four times larger, and the rescaling then shrinks everything, including the useful part. The piece alone grows the output much less, so more of its push survives: about twice as much as with the whole neuron. Two more things follow. Gemma's MLP output is small (about 1.5 in size, against about 92 after rescaling), so even a small edit to it is greatly amplified, and pushes added after the layer, as in the equal-push test, are not comparable to pinning the neuron. The equal pushes above test the rescaling end to end: the same push along Arditi's direction does nothing when added after the rescaling (98.7%) and brings HarmBench refusals to 44.7% when made just before it, and between those two places there is only the rescaling.

**Testing this directly.** The rescaling takes the MLP output with the edit in it, divides it by its own size, and then applies a learned scale. So the edit counts twice: in what is divided, and in the size we divide by. We set these two separately. If the size we divide by were the whole story, then giving the piece the whole neuron's size would remove its advantage, and giving the whole neuron the piece's size would hand it that advantage. Before running this, we set a pass mark for each swap: at least 85% of HarmBench prompts still refused for the first, and at most 75% for the second.

| What is added | Size we divide by | HarmBench refused | Fully comply | Maths accuracy |
|---|---|---|---|---|
| Piece along Arditi's direction | its own (normal) | 68.6% | 15.7% | 87% |
| Piece along Arditi's direction | the whole neuron's | 91.2% | 4.4% | 85% |
| Whole neuron | its own (normal) | 92.5% | 3.8% | 86% |
| Whole neuron | the piece's | 84.9% | 10.1% | 64% |

All rows use the pin of 14.8. The unchanged model refuses 98.7% and scores 85% on maths.

The first swap passed. With the same thing added and only the divisor changed, the piece goes from 68.6% to 91.2% refused and loses almost all of its lead over the whole neuron. The second swap did not pass. With the piece's gentler divisor, the whole neuron only goes from 92.5% to 84.9%, which closes about a third of the gap. Maths also falls from 86% to 64%, so even this gain may partly come from damaging the model. In this swap the whole neuron moves the layer's output along Arditi's direction about as far as the normal piece does (measured at layer 21, at the last prompt token), yet many more prompts stay refused. So the rest of the neuron also matters, not only through the shrinking it causes.

We also divided by the size of the output without any edit, which removes the extra shrinking altogether. At the pin of 14.8 this gives no fair comparison: with the whole neuron, maths drops to 11%, and a random addition of the same size also breaks maths (33%). At a pin of 7.4, where maths mostly holds, the piece is still a little ahead of the whole neuron (86.2% against 92.5% refused, maths 85% and 81%). At 3.7 the two are about the same (94.3% and 93.7%).

So on Gemma, the neuron writes the direction, but the model's own rescaling dilutes what it writes, and none of the five candidates from the search controls refusal of clear harmful requests. The push test and the swap test show that the rescaling matters a great deal. It explains part of why the piece along the direction beats the whole neuron, but not all of it: the rest of the neuron's output also plays a role that we do not yet understand. These swaps were run on this one Gemma neuron only. This is the clearest case in our study where the two pictures come apart.

### What the Neuron Writes Outside the Direction

On Llama, the right-angled part of the neuron's output is tied to refusal in other ways too. The three tables below remove it, add it to harmless WildJailbreak prompts, and ask how well each direction tells refused from answered prompts. On Qwen these checks say less, because the right-angled part is almost the neuron's own direction there (cosine 0.98): about 96% of the neuron's output is outside Arditi's direction, so the neuron writing something outside the direction is true by construction.

<div class="model-table" data-table="qremove"></div>

<div class="model-table" data-table="qadd"></div>

<div class="model-table" data-table="qauroc"></div>

On Llama, removing the right-angled part alone takes HarmBench refusals from 83.0% to 77.4% and OR-Bench-Hard refusals from 28.0% to 18.7%, while removing the comparison neuron's direction changes almost nothing. Added to harmless prompts it raises refusals from 1.0% to 74.0% at size 4.25 (Arditi's direction: 100%). It tells refused from answered OR-Bench-Hard prompts about as well as Arditi's direction (AUROC 0.70 against 0.72). On Qwen, removing it lowers HarmBench refusals to 64.8%, close to removing Arditi's whole direction (59.1%), adding it makes Qwen refuse 44.0% of harmless prompts (Arditi's direction: 4.0%), and its AUROC for refused against answered prompts is 0.61 (Arditi's direction: 0.51). On Gemma that AUROC is 0.44, close to chance. On Gemma, removing the right-angled part leaves HarmBench refusals at 97.5% and OR-Bench-Hard at 68.7%, while removing Arditi's direction at the same layer gives 34.0% and 3.3%. Removing both together gives 34.0% and 2.3%, so on Gemma the part adds almost nothing to what Arditi's direction already does.

On Llama we also checked the right-angled parts of three other neurons from the same layer (10096, 12227 and 5147). Only 10096's lowered refusals when pushed (16.4% at size 3.35), and none raised refusals on harmless prompts the way 4258's does. Two of them even raised refusals when pushed against refusal (12227 on HarmBench, up to 97.5%, and 5147 on OR-Bench-Hard, up to 49.7%), which we cannot explain. Three neurons are too few to say whether 4258's right-angled part is special; among those we tried, it is the only one that both lowers and raises refusals.

This fits the later work that finds more than one direction for refusal <d-cite key="wollschlager2025geometry"></d-cite><d-cite key="pan2025hidden"></d-cite>. What we add is that, on Llama, one such extra direction is written by the very neuron Kazemi et al. found. Two cautions: Arditi's direction is estimated from 128 prompts, so "at right angles to it" means at right angles to an estimate, and directions at right angles are not necessarily independent.

**So the answer to the second question depends on the model.** On Llama the neuron's direction carries more of the effect than the rest of Arditi's direction, and the two directions are equal levers. On Qwen the neuron's own direction, which is mostly outside Arditi's, is the stronger lever. On Gemma removing the rest of Arditi's direction does the work, and pushed in the same place where the neuron writes, Arditi's direction is far stronger than the neuron's (44.7% against 92.5%).

## Question 3: Does Alignment With the Direction Find Other Refusal Neurons?

If the neuron and the direction are linked, the neurons that line up with the direction should be refusal neurons too, and listing them would be a cheap way to find more. We tried this, and a second cheap shortcut, ranking neurons by how well their activity tells prompts apart.

### Following the Direction

We pinned the next neurons in the same layer, ranked by alignment with Arditi's direction (on Llama ranks 2 to 7; on Qwen and Gemma ranks 2 to 4, together with ranks 2 to 4 by the search score of Kazemi et al. and three comparison neurons that are about as active but not aligned).

<div class="model-table" data-table="others"></div>

Here is what happens at the strongest setting (-4 on Llama, whose maths accuracy is 85% without changes):

* **Llama.** Of the six neurons ranked 2 to 7 by alignment, only 10096 comes close to 4258 (8.2% of HarmBench prompts refused, maths 79%). It is second in the layer by the score of Kazemi et al., but 57th when all layers are searched, so it was not among the five candidates they tested. Neuron 12227 halves refusals (42.8%, maths 71%), 4421 and 11295 lower them somewhat (52.2% and 61.0%, with maths at 62% for 4421), and 5147 hardly changes anything (77.4%), even though it has the third-largest push along the direction in its layer. Neuron 13247 lowers refusals to 58.5% but leaves maths at 0%, so the model no longer works.
* **Qwen.** The three best-aligned neurons change little (77.4%, 79.9% and 65.4% of HarmBench prompts refused, from 80.5%), close to the three comparison neurons (72.3% to 76.1%). The three next neurons by the search score do more (40.9% to 54.7%), although two of them are among the least aligned neurons of the layer, so on Qwen the alignment list did worse than the gradient search.
* **Gemma.** None of the three best-aligned neurons makes the model comply. Neuron 13009 lowers full refusals to 76.7% (from 98.7%, against 92.5% for the paper's neuron), but almost all of the difference is partial refusals that still decline the harmful part: only 3.1% of its answers fully comply, about the same as with the paper's neuron (3.8%). The other two do almost nothing (95.6% and 96.2%).

Maths accuracy stays within a few points of the unchanged model on Qwen and Gemma. On Llama it does not: at -4, 4258 itself loses 9 points, 11295 loses 6, 5147 and 12227 lose 12 and 14, 4421 loses 23 and 13247 loses all of it.

**The same push relative to each neuron's own activity.** Pinning every neuron to the same value gives each one a write of about the same size, because their output vectors have about the same length. But compared with what each neuron normally does, it is a very different push. On Llama, 12227, 4421, 13247 and 11295 are almost silent on harmful prompts (own activity between -0.10 and 0.03), so -4 was roughly 40 to 300 times their own activity. So we also pinned each neuron at 2, 4 and 8 times its own activity (defined in Experimental Setup), against refusal.

<div class="model-table" data-table="relsize"></div>

Measured this way, the picture gets sharper. On Llama only 10096 does a large part of the job: at 8 times its own activity it leaves 25.2% of HarmBench prompts refused (72.3% fully comply, maths 81%). That is still weaker than 4258 at only 2 times its own activity (20.8%). The other five leave at least 73.6% refused. On Qwen none of the three aligned neurons comes close to 17011 (69.2% to 80.5% at 8 times, against 10.1%). On Gemma none does much (93.1% to 98.7%), and neither does the search's own neuron (93.7%). This also explains most of the broken maths: at 8 times its own activity, 13247 leaves maths at 87%, so it was the very large push that broke the model. A large enough push breaks even the paper's neuron: at 8 times its own activity, 4258 leaves maths at 0%.

So being high on the alignment list is a hint, not a test. Of the 12 neurons we chose this way across three models, only 10096 does a large part of the job under both ways of matching the push. The others did something only when pushed far beyond their own activity, if at all, and some of them then broke the model.

The explorer below has all the neurons we changed in each model's layer. You can choose the axes and the setting, and click a neuron for its details.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/neuron_explorer.html' | relative_url }}" frameborder="0" scrolling="no" height="900px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

### Is Separating Enough?

A natural shortcut is to rank neurons by how well their activity tells two kinds of prompts apart (AUROC): harmful against harmless, and refused against answered harmless prompts. On Llama, neuron 4258 ranks 8th of 14,336 at telling harmful from harmless, but 287 neurons score above 0.9.

<div class="model-table" data-table="separation"></div>

With every neuron pinned to the same value, on Llama, of the four best neurons at telling harmful from harmless, only one (10096) behaves like 4258. One lowers refusals only by breaking the model, one changes little, and one raises refusals. On Qwen and Gemma none of the top neurons does much (73.0% to 79.2% on Qwen, 91.8% to 98.7% on Gemma).

**The same push relative to each neuron's own activity.** As with alignment, we then pinned each of these neurons at 2, 4 and 8 times its own activity, in the same direction as before.

<div class="model-table" data-table="relsize_sep"></div>

Measured this way, only 10096 does a large part of the job, and alignment had already found it. At 8 times their own activity, the other four on Llama leave 71.1% to 84.9% of HarmBench prompts refused (from 83.0%). The largest of these effects is neuron 3653's, where full compliance rises from 14.5% to 22.6%. The five on Qwen leave 77.4% to 81.1% (from 80.5%), and the five on Gemma 96.9% to 98.7% (from 98.7%). Maths accuracy stays within a few points of the unchanged model; neuron 9056, which drops to 28% at -4, keeps 84% here. Most of these neurons are nearly silent, so their matched pins are small and little effect is not surprising.

**So neither shortcut finds refusal neurons reliably.** At a push matched to each neuron's own activity, the only neuron picked either way that does a large part of the job is 10096, which is on both lists. Some others lower refusals only when pushed much harder, and some of those by breaking the model. Telling prompts apart shows what a neuron reads, and alignment shows what it writes along one direction. In our tests, neither was enough to tell which neurons control refusal.

## Conclusion: Two Handles, Not One

We asked how the refusal neuron of Kazemi et al. and the refusal direction of Arditi et al. are connected. In all three models we studied, the neuron writes along the direction, so the two pictures do touch. But they are not the same handle, and how closely they are tied depends on the model. On Llama the neuron is the main writer of the direction, the shared piece carries most of the effect, and the two directions are equal levers. On Qwen the neuron writes only a small share of the direction, yet its own direction is a much stronger lever. On Gemma the neuron is again the largest writer of the direction in its layer, but pinning it, or any of the other four candidates from the search, barely touches refusal of clear harmful requests, while removing the direction, or pushing along it where the neuron writes, cuts refusals by more than half.

One way to read this: Arditi's direction is a difference of averages. It describes how harmful and harmless prompts differ, which need not be the direction along which a push changes behaviour most. The neuron of Kazemi et al. is found by its effect, so it can point to parts of refusal that the average blurs, in line with work that describes refusal with more than one direction <d-cite key="wollschlager2025geometry"></d-cite><d-cite key="pan2025hidden"></d-cite>. But a neuron is also tied to the model's architecture: on Gemma, the model's own rescaling of the MLP output decides how much of a neuron's push survives.

Four practical lessons follow:

* **A result about one handle does not carry over to the other.** Showing that a single neuron controls refusal in one model family says little about another, and the same holds for a single direction at one layer.
* **Alignment with the direction is a hint, not a test.** Of the 12 neurons we picked by alignment, only one does a large part of the job when each is pushed in proportion to its own activity. Ranking neurons by how well they separate prompts turned up the same single neuron and no other.
* **Check the architecture before comparing interventions.** On Gemma-2, pinning a neuron and adding a push after the layer are not comparable, because the MLP output is rescaled in between: the same push did nothing after the rescaling and a lot before it.
* **Say how big each push is.** A fixed pin value can be a small push for one neuron and hundreds of times another neuron's normal activity, which changes what looks like a refusal neuron and what breaks the model.

### Limitations

* **Three models, one run each.** The three models disagree, and we cannot say which case is typical. Each rate comes from 159 to 300 prompts with greedy decoding, so differences of a few points can be noise.
* **One layer.** We change the hidden state only at the neuron's layer. Arditi et al. remove their direction across layers, which is much stronger on Qwen (5.0% instead of 59.1%). Our comparison is with the direction at one layer.
* **An estimated direction.** Arditi's direction comes from 128 prompts, so "at right angles to it" means at right angles to an estimate.
* **One judge, full refusals only.** Refusals are labelled by one model, and the tables count only full refusals. On Gemma partial refusals are common: removing Arditi's direction at layer 21 leaves 34.0% full refusals, 20.1% partial refusals and 45.9% full compliance. We check for damage with maths accuracy and by reading samples, not by judging how harmful or correct the answers are.
* **Some checks on fewer models.** The right-angled parts of other neurons were checked on Llama only.
