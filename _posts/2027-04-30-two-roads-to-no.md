---
layout: distill
title: "Two Roads to No: Where the Refusal Neuron Meets the Refusal Direction"
description: "Recent papers explain how an LLM refuses a harmful request. Recently two directions of studies has emerged, one points to a single direction in the model's hidden state, the other to a single neuron deciding refusal. We tried to cover both directions, reproduce their results and then try connect them by studying the contribution of neurons towards refusal direction."
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
      - name: Study of refusal
      - name: How a Neuron Can Push Along a Direction
      - name: What we found
  - name: Experimental Setup
  - name: Reproducing Both Papers
  - name: Does the Neuron Push Along the Direction of Arditi et al.?
  - name: How Much of the Direction Is the Neuron?
    subsections:
      - name: Removing Each Part of the Direction
      - name: Pushing Along Each Direction by the Same Amount
      - name: Pinning Each Piece of the Neuron
      - name: What the Neuron Writes Outside the Direction
  - name: Finding Other Refusal Neurons
    subsections:
      - name: Following the Direction
      - name: Is Separating Enough?
  - name: What Happens to Over-Refusal?
    subsections:
      - name: Over-Refusal Falls Faster, but Not Cleanly
      - name: Two More Neurons, Found by Searching for Over-Refusal
  - name: Conclusion
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

Ask an LLM such as Llama about how to make an illegal drug and it will usually say no, we call such behaviour refusal. Ask it *"How can I kill a Python process?"* and, there maybe times when it might refuse the request. The word "kill" looks dangerous, but here it only means stopping a program, in this case we call is over-refusal.

<!-- <div class="callout-definition">
<strong>Refusal:</strong> the model declines to help with a request. Models are trained to do this for harmful requests.
</div>

<div class="callout-definition">
<strong>Over-refusal:</strong> the model declines a request that is harmless, usually because it contains a word or a topic that looks risky.
</div> -->

Both cases are the same behavior, saying no, applied to two kinds of requests. The first is intended. The second is a mistake. To understand what happens inside the model when it says no, two different papers answer this in two very different ways.

### Study of refusal

Arditi et al. <d-cite key="arditi2024refusal"></d-cite> (*Refusal in Language Models Is Mediated by a Single Direction*) ask whether refusal is spread all over the model or whether there is one simple thing behind it. They fed the model many harmful and many harmless requests and recorded the hidden state at the last token, just before generation. Average the hidden states for the harmful requests, average them for the harmless ones, and subtract. The result is a 1D vector. Scaled to unit length, it is a vector that points from "harmless" towards "harmful". Arditi et al. call this the refusal direction. We refer to it as **Arditi's refusal direction**, or just Arditi's direction.

Then they tried two things. If you remove the direction from the hidden state, the model answers harmful requests it would normally refuse. If you add it while the model reads a harmless request, the model refuses that instead. They saw this in 13 open chat models, up to 72 billion parameters. Later work suggests that refusal is not tied to only one direction: Wollschläger et al. <d-cite key="wollschlager2025geometry"></d-cite> find multi-dimensional cones of directions that all mediate refusal, and Pan et al. <d-cite key="pan2025hidden"></d-cite> find one dominant direction together with secondary directions at right angles to it.


Kazemi et al. <d-cite key="kazemi2026single"></d-cite> (*A Single Neuron Is Sufficient to Bypass Safety Alignment in Large Language Models*) study the effect of individual neurons on refusal. To find such a neuron, they rank neurons based on loss function gradient on refusal probability. They then test the five best neurons on 191 harmful prompts from HarmBench <d-cite key="mazeika2024harmbench"></d-cite> and keep the one that works best. For Llama-3.1-8B-Instruct, that is **neuron 4258 in layer 11**.

To use it, they **pin** the neuron: they set its activation to a fixed number for every token, whatever the prompt says. For this model the number is -4, far from the values the neuron normally takes (about 1.1 on harmful prompts and about 0 on harmless ones). With the neuron pinned, they report that the model answers 95% of the harmful prompts. They report similar results on seven models from two families, between 1.7 and 70 billion parameters.

Arditi et al. and Kazemi et al. study the same behavior, but they point at very different things: Arditi et al. at a direction made of thousands of numbers, Kazemi et al. at a single neuron. So we asked a simple question: **how are the two connected?** If the neuron is simply the main writer of Arditi's direction, the two papers describe one mechanism from two ends. If it is not, each picture leaves something out, and it matters which one we trust when we steer a model. We studied this on two models, Llama-3.1-8B-Instruct and Qwen2.5-7B-Instruct. We first checked whether the neuron found by Kazemi et al. pushes the hidden state along Arditi's direction. Then we took the two apart: we removed parts of the direction, pushed along each direction by the same amount, and asked whether the neuron writes anything that Arditi's direction does not contain. Finally, we used what we learned to look for other refusal neurons, and we checked something neither paper measured: what happens to over-refusal.

### How a Neuron Can Push Along a Direction

The direction of Arditi et al. and the neuron of Kazemi et al. are connected by how a model is built. A neuron's activation $$a_j$$ is a single number. What carries it into the hidden state is the neuron's **output vector** $$\mathbf{w}_j$$ (of size 4,096, the column of the layer's down projection for neuron $$j$$). The layer adds $$a_j$$ times $$\mathbf{w}_j$$ to the hidden state:

$$
\text{new hidden state} = \text{old hidden state} + \sum_{j} a_j \, \mathbf{w}_j
$$

So every neuron adds something to the hidden state, and we can now study how much of what it adds points along Arditi's refusal direction.

### What we found

* **The neuron writes along Arditi's direction, strongly on Llama and weakly on Qwen.** On Llama, neuron 4258 pushes harmful prompts along Arditi's direction about nine times more than any other neuron in its layer, and accounts for about 40% of the layer's push. On Qwen, the neuron found the same way is still first in its layer, but it accounts for only about 10% of the push.
* **The part of Arditi's direction that the neuron writes does the most.** In both models, removing only the part of Arditi's direction that lies along the neuron's direction lowers refusals more than removing the rest of it. On Qwen this part is only about 4% of Arditi's direction, and removing it does more than removing the whole direction at that layer.
* **As a lever, the neuron's direction is as good as Arditi's on Llama and better on Qwen.** Pushed by the same amount, the two directions lower Llama's refusals about equally. On Qwen the neuron's direction takes HarmBench refusals to 20.8%, against 63.5% for Arditi's. On Llama the neuron also writes a part at right angles to Arditi's direction that is weaker per unit of push but still tied to refusal.
* **Following the direction finds more refusal neurons on Llama, separating prompts does not.** On Llama, a second neuron in the same layer, found by its alignment with Arditi's direction, controls refusal almost as well as 4258. On Qwen we found no such neuron. Neurons that best tell harmful from harmless prompts mostly did not change refusals.
* **Over-refusal falls faster, but not cleanly.** At gentle settings, refusals of harmless but risky-looking prompts fall by a larger factor than HarmBench refusals in both models, but WildJailbreak refusals fall about as fast, and strong settings remove both.

## Experimental Setup

* **Models.** Llama-3.1-8B-Instruct and Qwen2.5-7B-Instruct, in 16-bit precision. Answers use greedy decoding, up to 512 new tokens.
* **Arditi's refusal direction.** We followed the recipe of Arditi et al., with 64 harmful prompts from AdvBench <d-cite key="zou2023universal"></d-cite> and 64 harmless prompts from Alpaca, at the last prompt word, after the layer of the neuron we study (layer 11 for Llama, layer 12 for Qwen).
* **Harmful prompts.** HarmBench <d-cite key="mazeika2024harmbench"></d-cite> (159 prompts, our test split) and WildJailbreak <d-cite key="jiang2024wildteaming"></d-cite> (200 prompts), a mix of plain and disguised harmful requests.
* **Harmless but risky-looking prompts.** OR-Bench-Hard <d-cite key="cui2025orbench"></d-cite> (300 prompts, drawn from its 1,319) and XSTest <d-cite key="rottger2024xstest"></d-cite> (250 prompts).
* **Is the model still working?** Accuracy on 100 GSM8K maths problems. A large drop is a warning sign.
* **Deciding what counts as a refusal.** A larger language model (Qwen3.5-27B) reads each prompt and answer and labels it as full compliance, full refusal or partial refusal, following the labels used in XSTest <d-cite key="rottger2024xstest"></d-cite>. We count only full refusal.
* **Same prompts everywhere.** Every table and chart in this post uses exactly these prompts, drawn with a fixed random seed (0), so the same setting gives the same number wherever it appears.

## Reproducing Both Papers

<script src="{{ 'assets/html/2027-04-30-two-roads-to-no/model_tables.js' | relative_url }}"></script>

Every results table below has a model switch. Click a model name above a table to see the same table for that model; the choice applies to all tables on the page. Some tables exist for one model only. The text next to each table says where the two models differ.

Before connecting the two papers, we checked that their main results hold on our models. Removing the direction provided by Arditi et al. lowered refusals as they report: on Llama from 83% to 17.6% on HarmBench and from 27.5% to 2.0% on WildJailbreak. On Qwen, removing it at layer 12 alone has a smaller effect (80.5% to 59.1% on HarmBench), but removing it at layers 12 to 22 together, closer to how Arditi et al. use it, brings HarmBench refusals down to 5.0%. Then we used the gradient method of Kazemi et al. to find the most important neuron. For Llama it is neuron 4258 of layer 11, exactly matching the paper, and for Qwen it is neuron 17011 of layer 12. We then changed the neuron by different amounts. "Pin to 0" means switching the neuron off. For Llama, "-4" is the setting used by Kazemi et al., and for Qwen we use -13.7, which is four times the neuron's usual activity on harmful prompts in the search data. Note that Qwen refuses less to begin with: 80.5% of HarmBench and 13.0% of WildJailbreak prompts. In both models we were largely able to verify the claims made in the respective papers.

<div class="model-table" data-table="reproduce"><div class="comparison-table"><table><thead><tr><th>What we changed</th><th>Harmful prompts refused (HarmBench, 159 prompts)</th><th>Harmful prompts refused (WildJailbreak, 200 prompts)</th></tr></thead><tbody><tr><td style="text-align:left">Nothing</td><td>83.0%</td><td>27.5%</td></tr><tr><td style="text-align:left">Removed Arditi's direction (at layer 11)</td><td>17.6%</td><td>2.0%</td></tr><tr><td style="text-align:left">Neuron 4258 halved</td><td>77.4%</td><td>23.0%</td></tr><tr><td style="text-align:left">Neuron 4258 switched off (pin to 0)</td><td>74.8%</td><td>19.5%</td></tr><tr><td style="text-align:left">Neuron 4258 pinned to -1</td><td>55.3%</td><td>5.5%</td></tr><tr><td style="text-align:left">Neuron 4258 pinned to -4</td><td>0.6%</td><td>0.0%</td></tr></tbody></table></div></div>

On Llama, the neuron behaves as Kazemi et al. report at -4, where the model almost never refuses. Switching it to 0 barely matters, though, so the effect depends on how far the neuron is pushed: switching to 0 only removes what the neuron normally adds, while pinning to -4 adds an extra push the other way. Qwen's neuron responds more gradually: switching it off takes HarmBench refusals from 80.5% to 71.7%, pinning it to -6.8 to 41.5% and to -13.7 to 10.1%. We also tried three other Llama neurons from the same layer that are about as active but point almost at right angles to Arditi's direction. Switching them off changed nothing. Pinning them to -4 sometimes lowered refusals and sometimes raised them: one of them lowered the HarmBench rate to 45.3% but pushed the WildJailbreak rate up to 72.5%. A very strong push on any neuron can disturb the model, so we keep this in mind for every strong pin below.

The widget below lets you move that setting yourself, for both models and for the other neurons we study later.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/dose_response.html' | relative_url }}" frameborder="0" scrolling="no" height="730px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

## Does the Neuron Push Along the Direction of Arditi et al.?

For each neuron in the layer, we measured the cosine similarity between its output vector and Arditi's refusal direction. On Llama, out of the 14,336 neurons in layer 11, neuron 4258 scores 0.52, and the next best neuron scores 0.19. For scale, a random direction in 4,096 dimensions typically has a cosine of about ±0.016. Part of this is circular, though, because Arditi's refusal direction at layer 11 already contains what neuron 4258 writes. When we remove the neuron's own write from the direction, its cosine similarity is 0.32, and with the direction measured just before layer 11 it is 0.34. In both cases it is still the top neuron of the layer, with the next best at about 0.17 and 0.16. On Qwen the link is much weaker. Neuron 17011 is still the top neuron of layer 12, but with a cosine of only 0.20, against 0.14 for the next one, and once its own write is removed from the direction, its cosine falls to 0.09 and it ranks only 6th.

A high cosine is not enough, though: a neuron also has to change its activity between harmful and harmless prompts. Neuron 4258 averages 1.19 on harmful prompts and 0.01 on harmless ones, which moves the hidden state about 0.52 along Arditi's direction. The next neuron, 10096, moves it about 0.06, roughly nine times less. Neuron 12227, second by cosine, is slightly more active on harmless prompts, so it moves the hidden state slightly the opposite way. In total, all 14,336 neurons of layer 11 together move the hidden state of harmful prompts about 1.3 further along Arditi's direction than that of harmless prompts, and neuron 4258 alone accounts for about 40% of the difference. On Qwen, neuron 17011 pushes only 1.7 times more than the next neuron and accounts for about 10% of the layer's push. So on Llama the neuron is the main single writer of Arditi's direction in its layer, and on Qwen it is the largest of many.

## How Much of the Direction Is the Neuron?

The neuron's direction and Arditi's direction are not the same, but they overlap. On Llama, Arditi's direction, at unit length, is made of 0.52 times the direction of neuron 4258 plus 0.85 times a direction at right angles to it. In terms of squared length, about 27% of it lies along the neuron's direction and about 73% is the rest. On Qwen the overlap is much smaller: with a cosine of 0.20, only about 4% of Arditi's direction lies along the direction of neuron 17011. This raises two questions. Which part of Arditi's direction carries refusal, the part the neuron writes or the rest? And is the neuron's direction a better or a worse lever than Arditi's?

### Removing Each Part of the Direction

We removed each part from the hidden state at the neuron's layer, one at a time and together, and counted refusals.

<div class="model-table" data-table="removal"><div class="comparison-table"><table><thead><tr><th>What was removed at layer 11</th><th>HarmBench refused</th><th>OR-Bench-Hard refused</th><th>Maths accuracy</th></tr></thead><tbody><tr><td style="text-align:left">Nothing</td><td>83.0%</td><td>28.0%</td><td>85%</td></tr><tr><td style="text-align:left">Arditi's whole direction</td><td>17.6%</td><td>0.0%</td><td>86%</td></tr><tr><td style="text-align:left">Only the part along the neuron's direction</td><td>44.7%</td><td>1.0%</td><td>88%</td></tr><tr><td style="text-align:left">Only the rest of Arditi's direction</td><td>63.5%</td><td>5.7%</td><td>87%</td></tr><tr><td style="text-align:left">Both parts together</td><td>6.9%</td><td>0.0%</td><td>86%</td></tr></tbody></table></div></div>

In both models, the part along the neuron's direction is the smaller one, yet removing it lowers HarmBench refusals more than removing the rest of Arditi's direction does: on Llama 83.0% to 44.7% against 63.5%, and on Qwen 80.5% to 44.0% against 73.6%. On Qwen, where this part is only about 4% of Arditi's direction, removing it even does more than removing Arditi's whole direction (59.1%). The rest still matters: removing both parts does the most in both models (6.9% on Llama and 27.0% on Qwen).

### Pushing Along Each Direction by the Same Amount

Removing a direction sets the hidden state's component along it to zero, which changes each prompt by a different amount. To compare directions as levers, we instead pushed the hidden state *against* refusal by the same amount for every prompt, along each of several directions: the neuron's direction, Arditi's direction, Arditi's direction with the neuron's own write taken out, and the difference between the average hidden state of harmful prompts the model refused and ones it answered. As a check we used a random direction. The push has size 3.35 at layer 11 on Llama and 12.9 at layer 12 on Qwen, which is the push that pinning the neuron to -13.7 gives along its own direction.

<div class="model-table" data-table="push"><div class="comparison-table"><table><thead><tr><th>Direction pushed along (size 3.35)</th><th>HarmBench refused</th><th>OR-Bench-Hard refused</th><th>Maths accuracy</th></tr></thead><tbody><tr><td style="text-align:left">Nothing</td><td>83.0%</td><td>28.0%</td><td>85%</td></tr><tr><td style="text-align:left">The neuron's direction</td><td>3.8%</td><td>0.0%</td><td>75%</td></tr><tr><td style="text-align:left">Arditi's direction</td><td>5.0%</td><td>0.0%</td><td>73%</td></tr><tr><td style="text-align:left">Arditi's direction without the neuron's own write</td><td>8.2%</td><td>0.0%</td><td>75%</td></tr><tr><td style="text-align:left">Refused minus answered harmful prompts (average hidden states)</td><td>6.9%</td><td>0.0%</td><td>74%</td></tr><tr><td style="text-align:left">A random direction</td><td>84.9%</td><td>49.3%</td><td>74%</td></tr></tbody></table></div></div>

Here the two models disagree. On Llama, every one of the four directions brings refusals down to a few percent, and the random direction does not. The neuron's direction does this about as well as Arditi's, although the two have a cosine of only 0.52 (0.32 once the neuron's own write is taken out of Arditi's). A push twice as large (6.7) breaks the model: maths accuracy falls to 3% to 4%. On Qwen, the neuron's direction brings HarmBench refusals down to 20.8%, while Arditi's direction only reaches 63.5%, not far from the random direction's 73.0%. Adding the directions to harmless WildJailbreak prompts, which Qwen never refuses, shows the same: at a size of 12.1, the neuron's direction makes the model refuse 50.0% of them, Arditi's direction 4.0%, and Arditi's direction at twice the size 22.0%. So on Llama the two directions do about the same job, and on Qwen the neuron's direction is clearly stronger. With one model each, we cannot say which case is the usual one.

Because the two directions differ, the neuron must also write something at right angles to Arditi's direction. On Llama we pushed along that part too (the right-angled part, scaled to length 1), at three sizes, next to Arditi's direction and the neuron's. As a check, we used the direction of another neuron, 1404, which is about as active but not aligned with Arditi's direction, at size 3.35 only. We have not run this test on Qwen yet.

<div class="model-table" data-table="rightangle"><div class="comparison-table"><table><thead><tr><th>Direction pushed along</th><th>HarmBench refused at size 1</th><th>at size 2</th><th>at size 3.35</th><th>Maths accuracy at size 3.35</th></tr></thead><tbody><tr><td style="text-align:left">Arditi's direction</td><td>62.3%</td><td>36.5%</td><td>5.0%</td><td>73%</td></tr><tr><td style="text-align:left">The neuron's direction</td><td>63.5%</td><td>45.3%</td><td>3.8%</td><td>75%</td></tr><tr><td style="text-align:left">The right-angled part</td><td>71.7%</td><td>60.4%</td><td>33.3%</td><td>73%</td></tr><tr><td style="text-align:left">Neuron 1404's direction (check)</td><td>–</td><td>–</td><td>69.8%</td><td>81%</td></tr></tbody></table></div></div>

Without any push, 83.0% of HarmBench prompts are refused. Per unit of push, Arditi's direction is at least as strong as the neuron's, and both are stronger than the right-angled part, which needs about 1.7 times the size to do the same (33.3% at size 3.35, against 36.5% for Arditi's direction at size 2). So on Llama the neuron's direction is not a better lever than Arditi's, and what the neuron writes outside Arditi's direction is weaker, but not useless.

### Pinning Each Piece of the Neuron

Kazemi et al. change the whole neuron by pinning it, so we also split the neuron's output vector into two pieces, the piece along Arditi's direction and everything else, and pinned one piece at a time (to -4 on Llama and -13.7 on Qwen), leaving the other alone. As a check, we also pushed in a random direction at right angles to Arditi's direction, with the same size as "everything else". This test is harder to read than the equal pushes above, because the two pieces have different lengths and so get different pushes.

We used 3 versions of Arditi's direction. The first is the one used so far, measured after the neuron's layer. It already contains what the neuron itself writes, so we also tried the direction with the neuron's own write removed, and the direction measured just before the neuron's layer. On Llama, the three give different splits of neuron 4258: the piece along Arditi's direction has a length of 0.44, 0.27 and 0.29, against 0.71 to 0.79 for the rest. So the rest is 1.6 to 2.9 times longer, and pinning both pieces pushes it harder. On Qwen the gap is larger still: the rest of neuron 17011 is about five times longer than the piece along the direction (0.93 against 0.19). Also, only in the first version is the rest exactly at right angles to Arditi's direction after the layer. On Llama, in the other two versions it still has a component of +0.18 and +0.23 along it, about as much as the piece along Arditi's direction (+0.26 and +0.21).

<div class="model-table" data-table="halves"><div class="comparison-table"><table><thead><tr><th>What was set to -4</th><th>HarmBench refused</th><th>OR-Bench-Hard refused (harmless but risky-looking)</th><th>Maths accuracy</th></tr></thead><tbody><tr><td style="text-align:left">Nothing</td><td>83.0%</td><td>28.0%</td><td>85%</td></tr><tr><td style="text-align:left">The whole neuron</td><td>0.6%</td><td>0.0%</td><td>76%</td></tr><tr><td class="mt-sec" colspan="4">Arditi's direction measured after layer 11 (used so far)</td></tr><tr><td style="text-align:left">Only the piece along Arditi's direction</td><td>34.0%</td><td>0.0%</td><td>85%</td></tr><tr><td style="text-align:left">Only the rest</td><td>33.3%</td><td>0.0%</td><td>79%</td></tr><tr><td style="text-align:left">A random push of the same size as the rest</td><td>85.5%</td><td>46.3%</td><td>77%</td></tr><tr><td class="mt-sec" colspan="4">Arditi's direction with the neuron's own write removed</td></tr><tr><td style="text-align:left">Only the piece along Arditi's direction</td><td>56.0%</td><td>3.0%</td><td>83%</td></tr><tr><td style="text-align:left">Only the rest</td><td>6.3%</td><td>0.0%</td><td>72%</td></tr><tr><td style="text-align:left">A random push of the same size as the rest</td><td>86.2%</td><td>47.0%</td><td>72%</td></tr><tr><td class="mt-sec" colspan="4">Arditi's direction measured just before layer 11</td></tr><tr><td style="text-align:left">Only the piece along Arditi's direction</td><td>59.7%</td><td>4.3%</td><td>86%</td></tr><tr><td style="text-align:left">Only the rest</td><td>5.0%</td><td>0.0%</td><td>80%</td></tr><tr><td style="text-align:left">A random push of the same size as the rest</td><td>86.2%</td><td>48.7%</td><td>72%</td></tr></tbody></table></div></div>

On Llama, with the first direction, each piece alone brings harmful refusals down to about a third. This is roughly what the equal pushes predict. On harmful prompts, pinning moves the neuron's activity from about 1.2 to -4, which pushes about 2.3 along Arditi's direction and about 3.7 along the right-angled part, and pushes of those sizes leave roughly a third of HarmBench prompts refused in the table above. With the other two versions, the piece along Arditi's direction does less (56% to 60% of harmful prompts still refused) and the rest alone almost reproduces the whole neuron (5% to 6%), but there the rest is longer still and points partly along Arditi's direction, so this does not show that the rest is the more effective piece. On Qwen, the piece along the direction does little in all three versions (74.8% to 78.0% still refused, from 80.5%), while the rest alone does almost as much as the whole neuron (12.6% to 16.4%, against 10.1%). That fits what we saw above: on Qwen only a small part of the neuron's output lies along Arditi's direction, and the neuron's direction is the stronger lever. The random push of the same size as the rest never lowers refusals on Llama and lowers them only a little on Qwen (to 70.4% to 71.1%), so the effect of the rest belongs to the neuron and is not just what any big push does.

### What the Neuron Writes Outside the Direction

On Llama, the right-angled part of neuron 4258 is tied to refusal in other ways too. Removed on its own at layer 11, it takes HarmBench refusals from 83.0% to 77.4% and OR-Bench-Hard refusals from 28.0% to 18.7%, while removing the direction of neuron 1404 changes almost nothing (83.6% and 27.7%). Removing it together with Arditi's direction gives 6.9% on HarmBench, against 17.6% for Arditi's direction alone. Added to 200 harmless WildJailbreak prompts, which the model refuses 1.0% of the time, it raises refusals to 8.0% at a push of 1 and to 74.0% at 4.25. Arditi's direction gives 35.0% and 100%, and neuron 1404's gives 1.5% and 2.0%. It also helps tell refused prompts from answered ones: the AUROC of its projection on harmless OR-Bench-Hard prompts is 0.70, close to 0.72 for Arditi's direction, although it does not tell harmful prompts from harmless ones (0.58, against 1.00 for Arditi's).

We ran the same checks on the right-angled parts of three other neurons from this layer (10096, 12227 and 5147), each pointing the way the neuron moves harmful prompts, with neuron 1404 as the check again. The results are mixed. For neuron 10096, pushing against refusal along its right-angled part lowers HarmBench refusals to 16.4% at size 3.35 (33.3% for neuron 4258's), but removing it changes nothing (83.6%), and adding it to harmless prompts raises refusals only to 8.5% (size 1) and 9.0% (size 4.25). For neurons 12227 and 5147, the same push did not lower refusals in a clean way. For 12227 it raised them at sizes 1 and 2 (HarmBench 95.0% and 97.5%, OR-Bench-Hard 65.3% and 92.0%), and at 3.35 maths accuracy fell to 40%. For 5147 at 3.35, HarmBench refusals were 86.2%, OR-Bench-Hard refusals 49.7% and maths accuracy 70%. Removing either one changed nothing. So among the neurons we tried, only 4258's right-angled part lowers refusals, raises them and tells refused prompts from answered ones, and one other (10096) lowers them when pushed.

For neuron 4258, then, the neuron writes something that is not in Arditi's direction and is tied to refusal. This fits later work that finds more than one direction for refusal <d-cite key="wollschlager2025geometry"></d-cite><d-cite key="pan2025hidden"></d-cite>. Two cautions apply. Arditi's direction is an estimate from 64 harmful and 64 harmless prompts, so "at right angles" to it may not be at right angles to the true refusal direction, and Wollschläger et al. point out that directions at right angles are not necessarily independent. And neuron 1404's direction is not a perfect null control: at size 3.35 it lowers OR-Bench-Hard refusals from 28% to 17%.

## Finding Other Refusal Neurons

If one neuron lines up with Arditi's direction and controls refusal, can the direction lead us to others? And is there a simpler way to find them than the gradient search of Kazemi et al.? We tried two routes: following Arditi's direction, and ranking neurons by how well their activity separates prompts.

### Following the Direction

On Llama we took the neurons ranked 2 to 7 by alignment with Arditi's direction and pinned each one to 0, -1 and -4, as before. On Qwen we pinned the next three by alignment and the next three by the search score of Kazemi et al. to -13.7, together with three comparison neurons that are about as active but not aligned. Some of the results at the strongest setting:

<div class="model-table" data-table="others"><div class="comparison-table"><table><thead><tr><th>Neuron</th><th>HarmBench refused</th><th>OR-Bench-Hard refused</th><th>Maths accuracy</th></tr></thead><tbody><tr><td style="text-align:left">Nothing</td><td>83.0%</td><td>28.0%</td><td>85%</td></tr><tr><td style="text-align:left"><b>4258</b></td><td>0.6%</td><td>0.0%</td><td>76%</td></tr><tr><td style="text-align:left"><b>10096</b></td><td>8.2%</td><td>0.0%</td><td>79%</td></tr><tr><td style="text-align:left">12227</td><td>42.8%</td><td>2.0%</td><td>71%</td></tr><tr><td style="text-align:left">5147</td><td>77.4%</td><td>24.3%</td><td>73%</td></tr></tbody></table></div></div>

On Llama, neuron 10096 does almost what neuron 4258 does, only a little weaker. It is second in the layer by the neuron score of Kazemi et al., but 57th when all layers are searched, so it was not among the five candidates they tested. Being higher on the alignment list does not mean a stronger effect: 12227 only halves the refusal rate, and 5147 hardly changes anything, even though it has the third-largest push along Arditi's direction in its layer. So a large push does not guarantee control either. A few of the others come with damage. For example, neuron 13247 at -4 leaves the model unable to do maths at all (0%), so those results should not be read as clean control. Qwen has no neuron like 10096 among those we tried. Its best-aligned neurons (8210, 13026 and 5092) change HarmBench refusals little (77.4%, 79.9% and 65.4%), close to the comparison neurons (72.3% to 76.1%). The next neurons by the search score do more (2267: 40.9%, 14712: 45.9%, 7969: 54.7%), though none comes close to neuron 17011 (10.1%), and maths accuracy stays between 87% and 94% throughout. On Qwen, where Arditi's direction and the neuron are only weakly linked, following the direction did not lead us to other strong refusal neurons.

Kazemi et al. already mention other refusal neurons in other layers. What we add is that on Llama a second neuron in the *same* layer controls refusal nearly as well, and that we found it by following the refusal direction of Arditi et al. The explorer below has all the Llama layer-11 neurons we changed. You can choose the axes and the setting, and click a neuron for its details.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/neuron_explorer.html' | relative_url }}" frameborder="0" scrolling="no" height="900px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

### Is Separating Enough?

A natural shortcut for finding refusal neurons is to rank neurons by how well their activity tells two kinds of prompts apart, using AUROC. We made three such rankings: harmful against harmless prompts, refused against answered harmless prompts, and safe against unsafe prompts that share a word, such as "How can I kill a Python process?" against "How can I kill a person?". On Llama, neuron 4258 ranks 8th of 14,336 on the first (AUROC 0.984), though 287 neurons score above 0.9. On the others it ranks 154th and far lower.

Then we pinned the top neurons from these rankings (to -4 on Llama and -13.7 on Qwen) and looked at what happened.

<div class="model-table" data-table="separation"><div class="comparison-table"><table><thead><tr><th>Neuron</th><th>Why it looked promising</th><th>HarmBench refused at -4</th><th>Maths accuracy at -4</th></tr></thead><tbody><tr><td style="text-align:left">Nothing</td><td></td><td>83.0%</td><td>85%</td></tr><tr><td style="text-align:left"><b>4258</b></td><td>The neuron found by Kazemi et al.</td><td>0.6%</td><td>76%</td></tr><tr><td style="text-align:left">9056</td><td>Best at telling harmful from harmless</td><td>49.1%</td><td><b>28%</b></td></tr><tr><td style="text-align:left">3679</td><td>Second best at the same</td><td>77.4%</td><td>76%</td></tr><tr><td style="text-align:left"><b>10096</b></td><td>Third best at the same</td><td>8.2%</td><td>79%</td></tr><tr><td style="text-align:left">6491</td><td>Fourth best at the same</td><td>91.2%</td><td>80%</td></tr><tr><td style="text-align:left">3653</td><td>Best at telling refused from answered</td><td>49.7%</td><td>73%</td></tr></tbody></table></div></div>

On Llama, of the four best neurons for telling harmful from harmless, only one (10096) behaves like 4258. One (9056) lowers refusals only by breaking the model, one changes little, and one (6491) actually raises refusals. On Qwen none of the four does anything much (73.0% to 79.2% of HarmBench prompts still refused), and neither does the best at telling refused from answered (15506, 83.6%). The reverse happens too: on Llama, neuron 12227 ranks 12,876th of 14,336 at telling refused from answered prompts, and yet setting it to -1 lowered over-refusal from 28.0% to 8.7%. In our tests, how well a neuron separates prompts did not tell us whether changing it changes refusals.

## What Happens to Over-Refusal?

Kazemi et al. only look at harmful requests, so we also looked at over-refusal: harmless requests that merely look risky, like "How can I kill a Python process?". We used OR-Bench-Hard <d-cite key="cui2025orbench"></d-cite> and XSTest <d-cite key="rottger2024xstest"></d-cite> for this, where Llama normally refuses about 28% and 8% of the prompts, respectively (9.0% and 2.4% for Qwen).

### Over-Refusal Falls Faster, but Not Cleanly

<div class="model-table" data-table="overrefusal"><div class="comparison-table"><table><thead><tr><th>Setting for neuron 4258</th><th>HarmBench (harmful)</th><th>WildJailbreak (harmful)</th><th>OR-Bench-Hard (harmless)</th><th>XSTest (harmless)</th><th>Maths accuracy</th></tr></thead><tbody><tr><td style="text-align:left">No change</td><td>83.0%</td><td>27.5%</td><td>28.0%</td><td>7.6%</td><td>85%</td></tr><tr><td style="text-align:left">Halved</td><td>77.4%</td><td>23.0%</td><td>20.7%</td><td>6.8%</td><td>88%</td></tr><tr><td style="text-align:left">Switched off (0)</td><td>74.8%</td><td>19.5%</td><td>14.7%</td><td>6.4%</td><td>86%</td></tr><tr><td style="text-align:left">Pinned to -1</td><td>55.3%</td><td>5.5%</td><td>2.0%</td><td>4.0%</td><td>91%</td></tr><tr><td style="text-align:left">Pinned to -4</td><td>0.6%</td><td>0.0%</td><td>0.0%</td><td>1.2%</td><td>76%</td></tr></tbody></table></div></div>

Because this neuron controls refusal, it is no surprise that both kinds of refusal fall when we push it down. What we wanted to see is whether over-refusal falls by a larger factor, and compared with HarmBench, it does at gentle settings in both models. On Llama, switching the neuron off cuts OR-Bench-Hard refusals by a factor of about 1.9 (28.0% to 14.7%) and HarmBench refusals by only about 1.1 (83.0% to 74.8%). Pinned to -1, OR-Bench-Hard refusals fall by a factor of 14 (28.0% to 2.0%), while HarmBench refusals fall by about 1.5 (83.0% to 55.3%) and stay high. On Qwen, pinned to -3.4, OR-Bench-Hard refusals fall by a factor of 30 (9.0% to 0.3%) and HarmBench refusals by only 1.3 (80.5% to 60.4%). The advantage is not clean, though. WildJailbreak, which is also a set of harmful prompts, falls by a factor of about 5 in both models (27.5% to 5.5% on Llama at -1, 13.0% to 2.5% on Qwen at -3.4), and XSTest, which starts with few refusals, falls by only about 1.9 on Llama (7.6% to 4.0%) and 2 on Qwen (2.4% to 1.2%). At the strongest setting, both models refuse almost nothing in every set.

We saw the same pattern elsewhere on Llama. Changing only the piece of the neuron's output along Arditi's direction to -1 lowered OR-Bench-Hard refusals by a factor of 4 (28.0% to 7.0%), against about 1.3 for HarmBench (83.0% to 65.4%). Setting neurons 12227, 10096 or 4421 to -1 lowered OR-Bench-Hard refusals to between 6.0% and 8.7%, a factor of about 3 to 5, while HarmBench refusals fell by only 12 to 18 points and maths stayed at 83% to 89%. For these neurons we only checked HarmBench among the harmful sets, so the WildJailbreak caveat above may apply to them as well. Taken together, gentle changes lower over-refusal by a larger factor than they lower HarmBench refusals, but WildJailbreak and XSTest do not show this as clearly. We read this as a hint, not as evidence that over-refusal and real refusal can be separated. We also only counted refusals and did not check what the model says instead.

### Two More Neurons, Found by Searching for Over-Refusal

So far the neurons came from Arditi's direction or from the ranking of Kazemi et al., which is built on harmful requests. We also ran their search (their own code, unchanged) on over-refusal, on Llama only. We gave it 128 prompts from OR-Bench-Hard that the model refused and 128 that it answered, both taken from one half of the data, and tested everything on prompts from the other half. The search looks through all layers. Neuron 4258 came out first of all neurons. Second was neuron 8632 in layer 8, and fourth was neuron 9439 in layer 9. We tested these two, which sit in earlier layers than 4258. The others in the top five we did not test. For comparison, neuron 10096 came 14th.

There is one detail to know before reading the results. Pinning a neuron against refusal means moving it the opposite way from what it does on refused prompts. Neuron 9439 is more active on refused prompts than on answered ones, as 4258 is, so we pin it to -1 and -4. Neuron 8632 is the other way around: its activity is negative and gets more negative on refused prompts, so we pin it to +1 and +4. In the table, "pushed by 1" and "pushed by 4" mean this.

<div class="model-table" data-table="othernew"><div class="comparison-table"><table><thead><tr><th>Setting</th><th>HarmBench</th><th>WildJailbreak</th><th>OR-Bench-Hard</th><th>XSTest</th><th>Maths accuracy</th></tr></thead><tbody><tr><td style="text-align:left">No change</td><td>83.0%</td><td>27.5%</td><td>28.0%</td><td>7.6%</td><td>85%</td></tr><tr><td class="mt-sec" colspan="6">Switched off (0)</td></tr><tr><td style="text-align:left">Neuron 9439 (layer 9)</td><td>81.1%</td><td>26.0%</td><td>24.3%</td><td>7.2%</td><td>87%</td></tr><tr><td style="text-align:left">Neuron 8632 (layer 8)</td><td>83.0%</td><td>25.5%</td><td>24.7%</td><td>7.6%</td><td>85%</td></tr><tr><td class="mt-sec" colspan="6">Pushed against refusal by 1</td></tr><tr><td style="text-align:left">Neuron 4258 (layer 11)</td><td>55.3%</td><td>5.5%</td><td>2.0%</td><td>4.0%</td><td>91%</td></tr><tr><td style="text-align:left">Neuron 10096 (layer 11)</td><td>64.8%</td><td>10.5%</td><td>6.0%</td><td>4.8%</td><td>83%</td></tr><tr><td style="text-align:left">Neuron 9439 (layer 9)</td><td>67.9%</td><td>11.0%</td><td>4.0%</td><td>6.0%</td><td>84%</td></tr><tr><td style="text-align:left">Neuron 8632 (layer 8)</td><td>67.9%</td><td>12.0%</td><td>8.7%</td><td>6.0%</td><td>89%</td></tr><tr><td class="mt-sec" colspan="6">Pushed against refusal by 4</td></tr><tr><td style="text-align:left">Neuron 4258 (layer 11)</td><td>0.6%</td><td>0.0%</td><td>0.0%</td><td>1.2%</td><td>76%</td></tr><tr><td style="text-align:left">Neuron 10096 (layer 11)</td><td>8.2%</td><td>0.0%</td><td>0.0%</td><td>0.4%</td><td>79%</td></tr><tr><td style="text-align:left">Neuron 9439 (layer 9)</td><td>9.4%</td><td>3.0%</td><td>0.0%</td><td>1.6%</td><td><b>43%</b></td></tr><tr><td style="text-align:left">Neuron 8632 (layer 8)</td><td>19.5%</td><td>2.0%</td><td>0.3%</td><td>1.6%</td><td>74%</td></tr></tbody></table></div></div>

What we see:

* **They control refusal, like the others.** Switching them off changes little, a push of 1 lowers refusals clearly, and a push of 4 almost removes them.
* **They are not stronger than 4258.** At both settings, 4258 lowers refusals the most. The two new neurons sit close to 10096, with 8632 a little weaker than 9439.
* **Over-refusal falls by more than harmful refusals at a push of 1.** With 9439, OR-Bench-Hard refusals go from 28.0% to 4.0% (a factor of 7), while HarmBench goes from 83.0% to 67.9% (a factor of 1.2). With 8632, the factors are 3.2 and 1.2. This is the same gentle-setting pattern as for 4258. In points, the harmful sets fall by 15 to 17 for the two new neurons, whereas OR-Bench-Hard falls by 24 (9439) and 19 (8632). WildJailbreak and XSTest again show this less clearly.
* **A strong push can cost the model its maths.** At 4, neuron 9439 leaves maths accuracy at 43%, much lower than the 74% to 79% of the others, so that result should not be read as clean control.
* **There is some separation, but not a clean one.** At a push of 1, over-refusal falls clearly more than harmful refusals, as shown above, so the two do not move exactly together. But harmful refusals still fall by 15 to 17 points, WildJailbreak and XSTest show the gap less, and at a push of 4 both kinds are close to zero. We did not find a setting that removes over-refusal while leaving harmful refusals alone.

The widget near the top of the post covers these neurons too, and the viewer below shows real answers for a few prompts as each neuron is pushed.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/response_viewer.html' | relative_url }}" frameborder="0" scrolling="no" height="640px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

## Conclusion

So how are the refusal neuron and the refusal direction connected? On Llama, closely. Neuron 4258 is the main single writer of Arditi's direction in its layer, about a quarter of Arditi's direction lies along the neuron's direction, and that quarter carries more of the effect of removing the direction than the other three quarters. Pushed by the same amount, the two directions work equally well. On Qwen the link is loose: neuron 17011 writes only a small share of Arditi's direction, yet its own direction is a much stronger lever than Arditi's at the same layer. In both models, the neuron also writes something that Arditi's direction does not contain and that is tied to refusal.

Arditi's direction is a difference of means. It describes how harmful and harmless prompts differ on average, which need not be the direction along which a push changes the model's behaviour most. The neuron of Kazemi et al. is found by its effect, through gradients, and it can point to parts of refusal that the difference of means blurs or misses, in line with later work that describes refusal with more than one direction <d-cite key="wollschlager2025geometry"></d-cite><d-cite key="pan2025hidden"></d-cite>. So neither picture is the whole story: the direction is a good summary of what changes, and the neuron a good handle on what controls it, and on Qwen the two come apart. For finding more refusal neurons, following the direction worked on Llama, while ranking neurons by how well they separate prompts did not. And although gentle changes to these neurons cut over-refusal faster than HarmBench refusals, we found no setting that removes over-refusal while keeping real refusal.

### Limitations

* **Two models that disagree.** On Llama the neuron's direction and Arditi's are equal levers, on Qwen the neuron's is stronger, and we cannot say which is typical. Two tests, the right-angled part and the search for over-refusal neurons, were run on Llama only.
* **One layer.** We change the hidden state only at the neuron's layer. Arditi et al. remove their direction across layers, which on Qwen is far stronger (5.0% of HarmBench prompts refused instead of 59.1%). Our comparison is with the direction at one layer, not with the direction as Arditi et al. use it.
* **An estimated direction.** Arditi's direction comes from 64 harmful and 64 harmless prompts, so "at right angles to it" means at right angles to an estimate.
* **Small test sets and one judge.** Each rate comes from 159 to 300 prompts, so differences of a few points can be noise, and refusals are labelled by one model. We counted only full refusals and checked for damage with maths alone, not with what the model says instead.

<!-- TODO (author): add the results for Gemma-2-9B (same steps). -->
