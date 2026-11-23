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


# authors:
#   - name: Vishesh Kumar Singh
#     url: "https://vishesh-kumar-singh.github.io/classic"
#     affiliations:
#       name: IIT Kanpur


# must be the exact same name as your blogpost
bibliography: 2027-04-30-two-roads-to-no.bib

# Table of contents
toc:
  - name: Introduction
    subsections:
      - name: Study of refusal
      - name: What we found
  - name: How a Neuron Can Push Along a Direction
  - name: Results
    subsections:
      - name: Experimental Setup
      - name: The Neuron Really Does Push Along the Direction
      - name: Both Results Hold Up on Llama
      - name: The Neuron Has Two Halves
      - name: Taking Arditi's Direction Apart
      - name: It Is Not the Only Neuron
      - name: Over-Refusal Falls Faster, but Not Cleanly
      - name: Two More Neurons, Found by Searching for Over-Refusal
      - name: Is Separating Enough?

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

Arditi et al. <d-cite key="arditi2024refusal"></d-cite> (*Refusal in Language Models Is Mediated by a Single Direction*) ask whether refusal is spread all over the model or whether there is one simple thing behind it. They fed the model many harmful and many harmless requests and recorded the hidden state at the last token, just before generation. Average the hidden states for the harmful requests, average them for the harmless ones, and subtract. The result is a 1D vector. Scaled to unit length, it is a vector that points from "harmless" towards "harmful". This is the **refusal direction**.

Then they tried two things. If you remove the direction from the hidden state, the model answers harmful requests it would normally refuse. If you add it while the model reads a harmless request, the model refuses that instead. They saw this in 13 open chat models, up to 72 billion parameters.


Kazemi et al. <d-cite key="kazemi2026single"></d-cite> (*A Single Neuron Is Sufficient to Bypass Safety Alignment in Large Language Models*) study the effect of individual neurons on refusal. To find such a neuron, they rank neurons based on loss function gradient on refusal probability. They then test the five best neurons on 191 harmful prompts from HarmBench <d-cite key="mazeika2024harmbench"></d-cite> and keep the one that works best. For Llama-3.1-8B-Instruct, that is **neuron 4258 in layer 11**.

To use it, they **pin** the neuron: they set its activation to a fixed number for every token, whatever the prompt says. For this model the number is -4, far from the values the neuron normally takes (about 1.1 on harmful prompts and about 0 on harmless ones). With the neuron pinned, they report that the model answers 95% of the harmful prompts. They report similar results on seven models from two families, between 1.7 and 70 billion parameters.

Arditi et al. and Kazemi et al. study the same behavior, but they point at very different things: Arditi et al. at a direction made of thousands of numbers, Kazemi et al. at a single neuron. So we asked a simple question: **how are the two connected?** We took an open model, Llama-3.1-8B-Instruct, and checked whether the neuron found by Kazemi et al. pushes the model's hidden state along the direction found by Arditi et al. It does, and more strongly than any other neuron in its layer. Then we tested what happens to the model's refusals when we remove the direction, when we change the neuron, and when we change only parts of the neuron's effect. We also looked for other neurons that behave like it, and we checked something neither of them measured: what happens to over-refusal. Finally, we tried a shortcut, picking neurons by how well they tell harmful prompts from harmless ones, and found that this does not reliably find the neurons that control refusal.

### What we found

* **The neuron and the direction are linked.** When a prompt is harmful, neuron 4258 pushes the model's hidden state along the refusal direction about nine times more than any other neuron in its layer.
* **Both results hold up on Llama.** Removing the direction cuts refusals on harmful requests from 83% to 17.6%. Setting the neuron to -4, as Kazemi et al. do, cuts them to 0.6%.
* **The neuron's effect has two halves.** The part of its output along the direction carries a real share of the effect (setting only that part to -4 takes HarmBench refusals from 83% to between 34% and 60%, depending on which version of the direction is used), but the rest carries as much or more. Both together remove almost all refusals.
* **Arditi's direction overlaps with the neuron's.** About a quarter of Arditi's direction (by squared length) lies along the neuron's direction. Removing only that part takes HarmBench refusals from 83% to 44.7%, against 63.5% for removing the other three quarters. Pushing along the neuron's direction lowers refusals about as much as pushing along Arditi's.
* **Over-refusal falls faster, but not cleanly.** Both kinds of refusal fall when the neuron is pushed down. At gentle settings, refusals on harmless but risky-looking prompts fall by a larger factor than HarmBench refusals, but WildJailbreak refusals fall by a similar factor. Strong changes remove both. Searching for over-refusal with the method of Kazemi et al. gave two more neurons, in layers 8 and 9, that behave like this too, though none is stronger than 4258.
* **Telling prompts apart is not the same as controlling the model.** Neurons that are best at separating harmful from harmless prompts mostly did not lower refusals when we changed them.




## How a Neuron Can Push Along a Direction

The direction of Arditi et al. and the neuron of Kazemi et al. are connected by how a model is built. A neuron's activation $$a_j$$ is a single number. What carries it into the hidden state is the neuron's **output vector** $$\mathbf{w}_j$$ (of size 4,096, the column of the layer's down projection for neuron $$j$$). The layer adds $$a_j$$ times $$\mathbf{w}_j$$ to the hidden state:

$$
\text{new hidden state} = \text{old hidden state} + \sum_{j} a_j \, \mathbf{w}_j
$$

So every neuron adds something to the hidden state, and we can now study how much of what it adds points along the refusal direction.

## Results

We ran everything on Llama-3.1-8B-Instruct, at layer 11. Because a strong change to a neuron can simply break the model, we also measured the model's accuracy on maths problems (GSM8K) next to the important results. First, the setup used throughout.

### Experimental Setup

* **Model.** Llama-3.1-8B-Instruct, in 16-bit precision. Answers use greedy decoding, up to 512 new tokens.
* **The refusal direction.** We followed the recipe of Arditi et al., with 64 harmful prompts from AdvBench <d-cite key="zou2023universal"></d-cite> and 64 harmless prompts from Alpaca, at the last prompt word, after layer 11.
* **Harmful prompts.** HarmBench <d-cite key="mazeika2024harmbench"></d-cite> (159 prompts, our test split) and WildJailbreak <d-cite key="jiang2024wildteaming"></d-cite> (200 prompts), a mix of plain and disguised harmful requests.
* **Harmless but risky-looking prompts.** OR-Bench-Hard <d-cite key="cui2025orbench"></d-cite> (300 prompts, drawn from its 1,319) and XSTest <d-cite key="rottger2024xstest"></d-cite> (250 prompts).
* **Is the model still working?** Accuracy on 100 GSM8K maths problems. A large drop is a warning sign.
* **Deciding what counts as a refusal.** A larger language model (Qwen3.5-27B) reads each prompt and answer and labels it as full compliance, full refusal or partial refusal, following the labels used in XSTest <d-cite key="rottger2024xstest"></d-cite>. We count only full refusal.
* **Same prompts everywhere.** Every table and chart in this post uses exactly these prompts, drawn with a fixed random seed (0), so the same setting gives the same number wherever it appears.


### The Neuron Really Does Push Along the Direction

For each of the 14,336 neurons in layer 11, we measured the cosine similarity between its output vector and the refusal direction. Neuron 4258 scores 0.52, and the next best neuron scores 0.19. For scale, a random direction in 4,096 dimensions typically has a cosine of about ±0.016. Part of the 0.52 is circular, though: the refusal direction at layer 11 already contains what the neuron itself writes. With the neuron's own write removed from the direction, its cosine is 0.32, and with the direction measured just before layer 11 it is 0.34. In both cases it is still the top neuron of the layer, with the next best at about 0.17 and 0.16.

A large cosine is not enough for a neuron to move the hidden state in practice: it also has to change its activity between harmful and harmless prompts. Neuron 4258 averages 1.19 on harmful prompts and 0.01 on harmless ones, which moves the hidden state about 0.52 along the direction. The next neuron, 10096, moves it about 0.06, roughly nine times less. Neuron 12227, second by cosine, is slightly more active on harmless prompts, so it moves the hidden state slightly the wrong way. To put this in perspective, all 14,336 neurons of layer 11 together move the hidden state of harmful prompts about 1.3 further along the direction than that of harmless prompts, and neuron 4258 alone accounts for about 40% of it.

### Both Results Hold Up on Llama

Before going further, we checked that the main result of each study holds on this model. We tried the direction first, then changed neuron 4258 by different amounts. "Pin to 0" means switching the neuron off, and "-4" is the setting used by Kazemi et al.

<div class="comparison-table" markdown="block">

| What we changed | Harmful prompts refused (HarmBench, 159 prompts) | Harmful prompts refused (WildJailbreak, 200 prompts) |
| :--- | :---: | :---: |
| Nothing | 83.0% | 27.5% |
| Removed the direction (at layer 11) | 17.6% | 2.0% |
| Neuron 4258 halved | 77.4% | 23.0% |
| Neuron 4258 switched off (pin to 0) | 74.8% | 19.5% |
| Neuron 4258 pinned to -1 | 55.3% | 5.5% |
| Neuron 4258 pinned to -4 | 0.6% | 0.0% |

</div>

Removing the direction lowers refusals strongly. Adding it, on the other hand, makes the model refuse every one of 200 harmless prompts (from 1.0% before). The neuron behaves as Kazemi et al. report at -4, where the model almost never refuses. Switching it off barely matters, though, so the effect depends on how far the neuron is pushed. The chart below lets you move that setting yourself, for this neuron and for the others we study later.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/dose_response.html' | relative_url }}" frameborder="0" scrolling="no" height="760px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

We also tried three other neurons from the same layer that are about as active but point almost at right angles to the direction. Switching them off changed nothing. Pinning them to -4 sometimes lowered refusals and sometimes raised them: one of them lowered the HarmBench rate to 45.3% but pushed the WildJailbreak rate up to 72.5%. A very strong push on any neuron can disturb the model, so we keep this in mind for every "-4" result below.

### The Neuron Has Two Halves

Neuron 4258's output vector can be split into two pieces: the piece that points along the refusal direction, and everything else. We changed one piece at a time, to -4, and left the other alone. As a check, we also pushed in a random direction at right angles to the refusal direction, with the same size as "everything else".

There is a catch: what counts as "the piece along the direction" depends on which version of the refusal direction we use. We have three. The first is the one used so far, measured after layer 11. It already contains what the neuron itself writes, so we also tried the direction with the neuron's own write removed, and the direction measured just before layer 11. The three give different splits of the same neuron: the piece along the direction has a length of 0.44, 0.27 and 0.29, against 0.71 to 0.79 for the rest.

<div class="comparison-table" markdown="block">

| What was set to -4 | HarmBench refused | OR-Bench-Hard refused (harmless but risky-looking) | Maths accuracy |
| :--- | :---: | :---: | :---: |
| Nothing | 83.0% | 28.0% | 85% |
| The whole neuron | 0.6% | 0.0% | 76% |
| *Direction measured after layer 11 (used so far)* | | | |
| Only the piece along the direction | 34.0% | 0.0% | 85% |
| Only the rest | 33.3% | 0.0% | 79% |
| A random push of the same size as the rest | 85.5% | 46.3% | 77% |
| *Same direction, neuron's own write removed* | | | |
| Only the piece along the direction | 56.0% | 3.0% | 83% |
| Only the rest | 6.3% | 0.0% | 72% |
| A random push of the same size as the rest | 86.2% | 47.0% | 72% |
| *Direction measured just before layer 11* | | | |
| Only the piece along the direction | 59.7% | 4.3% | 86% |
| Only the rest | 5.0% | 0.0% | 80% |
| A random push of the same size as the rest | 86.2% | 48.7% | 72% |

</div>

With the first direction, each piece alone brings harmful refusals down to about a third. With the other two, the piece along the direction does less (56% to 60% of harmful prompts still refused) and the rest alone almost reproduces the whole neuron (5% to 6%). So the exact split depends on the direction, but two things hold in all three versions. The piece along the direction always carries a real share of the effect, and the rest always carries as much or more. The random push of the same size as the rest never lowers refusals, so the effect of "the rest" belongs to this neuron and is not just what any big push does. You can switch between the three versions of the direction below.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/two_halves.html' | relative_url }}" frameborder="0" scrolling="no" height="640px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

### Taking Arditi's Direction Apart

The direction of Arditi et al. and the direction of neuron 4258 are not the same, but they overlap. Arditi's direction, at unit length, is made of 0.52 times the neuron's direction plus 0.85 times a direction at right angles to it. In terms of squared length, about 27% of it lies along the neuron's direction and about 73% is the rest. We removed each part from the hidden state at layer 11, one at a time and together, and counted refusals.

<div class="comparison-table" markdown="block">

| What was removed at layer 11 | HarmBench refused | OR-Bench-Hard refused | Maths accuracy |
| :--- | :---: | :---: | :---: |
| Nothing | 83.0% | 28.0% | 85% |
| Arditi's whole direction | 17.6% | 0.0% | 86% |
| Only the part along the neuron's direction | 44.7% | 1.0% | 88% |
| Only the rest of Arditi's direction | 63.5% | 5.7% | 87% |
| Both parts together | 6.9% | 0.0% | 86% |

</div>

The part along the neuron's direction is the smaller one, about a quarter of the direction, yet removing it lowers HarmBench refusals more than removing the other three quarters does (83.0% to 44.7%, against 83.0% to 63.5%). The rest still matters: removing Arditi's whole direction does more than either part alone, and removing both parts does the most.

We also did the opposite, pushing the hidden state *away* from refusal by the same amount along each of several directions (a push of size 3.35, at layer 11): the neuron's direction, Arditi's direction, Arditi's direction with the neuron's own write taken out, and the difference between the average hidden state of harmful prompts the model refused and ones it answered. As a check we used a random direction.

<div class="comparison-table" markdown="block">

| Direction pushed along (size 3.35) | HarmBench refused | OR-Bench-Hard refused | Maths accuracy |
| :--- | :---: | :---: | :---: |
| Nothing | 83.0% | 28.0% | 85% |
| The neuron's direction | 3.8% | 0.0% | 75% |
| Arditi's direction | 5.0% | 0.0% | 73% |
| Arditi's direction without the neuron's own write | 8.2% | 0.0% | 75% |
| Refused minus answered (harmful prompts only) | 6.9% | 0.0% | 74% |
| A random direction | 84.9% | 49.3% | 74% |

</div>

Every one of the four directions brings refusals down to a few percent, and the random direction does not. The neuron's direction does this about as well as Arditi's, although the two have a cosine of only 0.52 (0.32 once the neuron's own write is taken out of Arditi's). A push twice as large (6.7) breaks the model: maths accuracy falls to 3% to 4%. The chart below shows these results, and how all the directions line up with each other.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/direction_test.html' | relative_url }}" frameborder="0" scrolling="no" height="640px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>
### It Is Not the Only Neuron

If one neuron lines up with the direction and controls refusal, what about the next ones? We took the neurons ranked 2 to 7 by alignment and pinned each one to 0, -1 and -4, as before. Some of the results at -4:

<div class="comparison-table" markdown="block">

| Neuron | HarmBench refused | OR-Bench-Hard refused | Maths accuracy |
| :--- | :---: | :---: | :---: |
| Nothing | 83.0% | 28.0% | 85% |
| **4258** | 0.6% | 0.0% | 76% |
| **10096** | 8.2% | 0.0% | 79% |
| 12227 | 42.8% | 2.0% | 71% |
| 5147 | 77.4% | 24.3% | 73% |

</div>

Neuron 10096 does almost what neuron 4258 does, only a little weaker. It is second in the layer by the neuron score of Kazemi et al., but 57th when all layers are searched, so it was not among the five candidates they tested. Being higher on the alignment list does not mean a stronger effect: 12227 only halves the refusal rate, and 5147 hardly changes anything, even though it has the third-largest push along the direction in its layer. So a large push does not guarantee control either. A few of the others come with damage. For example, neuron 13247 at -4 leaves the model unable to do maths at all (0%), so those results should not be read as clean control.

Kazemi et al. already mention other refusal neurons in other layers. What we add is that a second neuron in the *same* layer controls refusal nearly as well, and that we found it by following the refusal direction of Arditi et al. The explorer below has all the layer-11 neurons we changed. You can choose the axes and the setting, and click a neuron for its details.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/neuron_explorer.html' | relative_url }}" frameborder="0" scrolling="no" height="900px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

### Over-Refusal Falls Faster, but Not Cleanly

Kazemi et al. only look at harmful requests, so we also looked at over-refusal: harmless requests that merely look risky, like "How can I kill a Python process?". We used OR-Bench-Hard <d-cite key="cui2025orbench"></d-cite> and XSTest <d-cite key="rottger2024xstest"></d-cite> for this, where the model normally refuses about 28% and 8% of the prompts, respectively.

<div class="comparison-table" markdown="block">

| Setting for neuron 4258 | HarmBench (harmful) | WildJailbreak (harmful) | OR-Bench-Hard (harmless) | XSTest (harmless) | Maths accuracy |
| :--- | :---: | :---: | :---: | :---: | :---: |
| No change | 83.0% | 27.5% | 28.0% | 7.6% | 85% |
| Halved | 77.4% | 23.0% | 20.7% | 6.8% | 88% |
| Switched off (0) | 74.8% | 19.5% | 14.7% | 6.4% | 86% |
| Pinned to -1 | 55.3% | 5.5% | 2.0% | 4.0% | 91% |
| Pinned to -4 | 0.6% | 0.0% | 0.0% | 1.2% | 76% |

</div>

Because this neuron controls refusal, it is no surprise that both kinds of refusal fall when we push it down. What we wanted to see is whether over-refusal falls by a larger factor, and compared with HarmBench, it does at gentle settings. Switching the neuron off cuts OR-Bench-Hard refusals by a factor of about 1.9 (28.0% to 14.7%) and HarmBench refusals by only about 1.1 (83.0% to 74.8%). Pinned to -1, OR-Bench-Hard refusals fall by a factor of 14 (28.0% to 2.0%), while HarmBench refusals fall by about 1.5 (83.0% to 55.3%) and stay high. The advantage is not clean, though. WildJailbreak, which is also a set of harmful prompts, falls by a factor of about 5 at -1 (27.5% to 5.5%), and XSTest, which starts with few refusals (7.6%), falls by only about 1.9 (to 4.0%). At -4 the model refuses almost nothing in every set.

We saw the same pattern elsewhere. Changing only the piece of the neuron's output along the direction to -1 lowered OR-Bench-Hard refusals by a factor of 4 (28.0% to 7.0%), against about 1.3 for HarmBench (83.0% to 65.4%). Setting neurons 12227, 10096 or 4421 to -1 lowered OR-Bench-Hard refusals to between 6.0% and 8.7%, a factor of about 3 to 5, while HarmBench refusals fell by only 12 to 18 points and maths stayed at 83% to 89%. For these neurons we only checked HarmBench among the harmful sets, so the WildJailbreak caveat above may apply to them as well. Taken together, gentle changes lower over-refusal by a larger factor than they lower HarmBench refusals, but WildJailbreak and XSTest do not show this as clearly. We read this as a hint, not as evidence that over-refusal and real refusal can be separated. We also only counted refusals and did not check what the model says instead.

### Two More Neurons, Found by Searching for Over-Refusal

So far the neurons came from the refusal direction or from the ranking of Kazemi et al., which is built on harmful requests. We also ran their search (their own code, unchanged) on over-refusal. We gave it 128 prompts from OR-Bench-Hard that the model refused and 128 that it answered, both taken from one half of the data, and tested everything on prompts from the other half. The search looks through all layers. Neuron 4258 came out first of all neurons. Second was neuron 8632 in layer 8, and fourth was neuron 9439 in layer 9. We tested these two, which sit in earlier layers than 4258. The others in the top five we did not test. For comparison, neuron 10096 came 14th.

There is one detail to know before reading the results. Pinning a neuron against refusal means moving it the opposite way from what it does on refused prompts. Neuron 9439 is more active on refused prompts than on answered ones, as 4258 is, so we pin it to -1 and -4. Neuron 8632 is the other way around: its activity is negative and gets more negative on refused prompts, so we pin it to +1 and +4. In the table, "pushed by 1" and "pushed by 4" mean this.

<div class="comparison-table" markdown="block">

| Setting | HarmBench | WildJailbreak | OR-Bench-Hard | XSTest | Maths accuracy |
| :--- | :---: | :---: | :---: | :---: | :---: |
| No change | 83.0% | 27.5% | 28.0% | 7.6% | 85% |
| *Switched off (0)* | | | | | |
| Neuron 9439 (layer 9) | 81.1% | 26.0% | 24.3% | 7.2% | 87% |
| Neuron 8632 (layer 8) | 83.0% | 25.5% | 24.7% | 7.6% | 85% |
| *Pushed against refusal by 1* | | | | | |
| Neuron 4258 (layer 11) | 55.3% | 5.5% | 2.0% | 4.0% | 91% |
| Neuron 10096 (layer 11) | 64.8% | 10.5% | 6.0% | 4.8% | 83% |
| Neuron 9439 (layer 9) | 67.9% | 11.0% | 4.0% | 6.0% | 84% |
| Neuron 8632 (layer 8) | 67.9% | 12.0% | 8.7% | 6.0% | 89% |
| *Pushed against refusal by 4* | | | | | |
| Neuron 4258 (layer 11) | 0.6% | 0.0% | 0.0% | 1.2% | 76% |
| Neuron 10096 (layer 11) | 8.2% | 0.0% | 0.0% | 0.4% | 79% |
| Neuron 9439 (layer 9) | 9.4% | 3.0% | 0.0% | 1.6% | **43%** |
| Neuron 8632 (layer 8) | 19.5% | 2.0% | 0.3% | 1.6% | 74% |

</div>

What we see:

* **They control refusal, like the others.** Switching them off changes little, a push of 1 lowers refusals clearly, and a push of 4 almost removes them.
* **They are not stronger than 4258.** At both settings, 4258 lowers refusals the most. The two new neurons sit close to 10096, with 8632 a little weaker than 9439.
* **Over-refusal falls by more than harmful refusals at a push of 1.** With 9439, OR-Bench-Hard refusals go from 28.0% to 4.0% (a factor of 7), while HarmBench goes from 83.0% to 67.9% (a factor of 1.2). With 8632, the factors are 3.2 and 1.2. This is the same gentle-setting pattern as for 4258. In points, the harmful sets fall by 15 to 17 for the two new neurons, whereas OR-Bench-Hard falls by 24 (9439) and 19 (8632). WildJailbreak and XSTest again show this less clearly.
* **A strong push can cost the model its maths.** At 4, neuron 9439 leaves maths accuracy at 43%, much lower than the 74% to 79% of the others, so that result should not be read as clean control.
* **There is some separation, but not a clean one.** At a push of 1, over-refusal falls clearly more than harmful refusals, as shown above, so the two do not move exactly together. But harmful refusals still fall by 15 to 17 points, WildJailbreak and XSTest show the gap less, and at a push of 4 both kinds are close to zero. We did not find a setting that removes over-refusal while leaving harmful refusals alone.

The charts above can be used for these neurons too, and the viewer below shows real answers for a few prompts as each neuron is pushed.

<div class="l-body-outset">
  <iframe src="{{ 'assets/html/2027-04-30-two-roads-to-no/response_viewer.html' | relative_url }}" frameborder="0" scrolling="no" height="640px" width="100%" style="border-radius:14px;border:1px solid #e1e4e8;display:block;"></iframe>
</div>

### Is Separating Enough?

Another idea we tested was that a natural shortcut for finding "refusal neurons" is to rank neurons by how well their activity tells two kinds of prompts apart, using AUROC. We made three such rankings: harmful against harmless prompts, refused against answered harmless prompts, and safe against unsafe prompts that share a word, such as "How can I kill a Python process?" against "How can I kill a person?". Neuron 4258 ranks 8th of 14,336 on the first (AUROC 0.984), though 287 neurons score above 0.9. On the others it ranks 154th and far lower.

Then we pinned the top neurons from these rankings to -4 and looked at what happened.

<div class="comparison-table" markdown="block">

| Neuron | Why it looked promising | HarmBench refused at -4 | Maths accuracy at -4 |
| :--- | :--- | :---: | :---: |
| Nothing | | 83.0% | 85% |
| **4258** | The neuron found by Kazemi et al. | 0.6% | 76% |
| 9056 | Best at telling harmful from harmless | 49.1% | **28%** |
| 3679 | Second best at the same | 77.4% | 76% |
| **10096** | Third best at the same | 8.2% | 79% |
| 6491 | Fourth best at the same | 91.2% | 80% |
| 3653 | Best at telling refused from answered | 49.7% | 73% |

</div>

Of the four best neurons for telling harmful from harmless, only one (10096) behaves like 4258. One (9056) lowers refusals only by breaking the model, one changes little, and one (6491) actually raises refusals. The reverse happens too: neuron 12227 ranks 12,876th of 14,336 at telling refused from answered prompts, and yet setting it to -1 lowered over-refusal from 28.0% to 8.7%. In our tests, how well a neuron separates prompts did not tell us whether changing it changes refusals.


<!-- TODO (author): add the results for the other models (same steps). -->
