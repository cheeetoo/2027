---
layout: distill
title: Even GPT-2 passes most functional global workspace tests
description: Gurnee et al. (2026) propose five functional tests for a global workspace and, using the Jacobian lens, find representations in Claude that pass all five. We run the same tests on GPT-2 small and find it passes verbal report, internal reasoning, flexible generalization, and selectivity, but fails directed modulation and lacks almost all of the supporting structure found in Claude. We think these four tests are easily satisfied and may mostly follow from how they and the J-lens are constructed, so they are not strong evidence that a model has a global workspace.
date: 2027-04-30
future: true
htmlwidgets: true

_styles: >
  d-article pre, d-article pre code { white-space: pre-wrap; overflow-wrap: anywhere; }

# Anonymize when submitting
authors:
  - name: Anonymous

# must be the exact same name as your blogpost
bibliography: 2027-04-30-gpt2-global-workspace.bib

toc:
  - name: tl;dr
  - name: Background
    subsections:
      - name: Functional properties in GPT-2
  - name: Method
    subsections:
      - name: Centering the J-lens vectors
      - name: Band of workspace layers
      - name: Scaling the number of J-lens vectors
  - name: The five criteria
    subsections:
      - name: Verbal report works
      - name: Directed modulation fails, present in small instruct model
      - name: Internal reasoning works
      - name: Flexible generalization works
      - name: Selectivity works
  - name: Nearly all of the structure is absent
    subsections:
      - name: No distinct band of workspace layers
      - name: No ignition
      - name: Limited capacity (by construction?)
      - name: Little amplification
  - name: Discussion
  - name: What this means
  - name: Appendix
    subsections:
      - name: A. Setup
      - name: B. Verbal report
      - name: C. Directed modulation
      - name: D. Internal reasoning
      - name: E. Flexible generalization
      - name: F. Selectivity
      - name: G. Structure
---

## tl;dr

- Anthropic’s [global workspace paper](https://transformer-circuits.pub/2026/workspace/) (Gurnee et al., 2026)<d-cite key="gurnee2026workspace"></d-cite> proposes five functional tests for a global workspace and, using the Jacobian lens (J-lens), finds representations in Claude that pass all five. It also finds structure supporting this set of representations in Claude that global workspace theory predicts.
- We test these on GPT-2 small (124M) and find it passes the tests for verbal report, internal reasoning, flexible generalization, and selectivity, but fails directed modulation. We also find almost all of the structure Gurnee et al. identified in Claude is absent.
- A functional global workspace would [be a big deal](https://experiencemachines.substack.com/p/merely-functional-is-still-a-big)<d-cite key="long2026merely"></d-cite>, but **we think these four tests are easily satisfied (GPT-2 small passes them), so aren't strong evidence** that Claude has one (or conscious access). A case for one would have to rely on what GPT-2 lacks: directed modulation and the rest of the structure.
- Gurnee et al. searched for representations the model can verbalize and then found that they “rather surprisingly” pass the other tests too. However we think **the four tests GPT-2 passes may mostly follow from how they and the J-lens are constructed**.

<div class="l-page">
{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig01.png" class="img-fluid" %}
</div>
<div class="caption">
Figure 1: The five functional properties of a global workspace from Gurnee et al., stylized versions of the experiments for them, and whether GPT-2 small passes each.
</div>

## Background

Global workspace theory (GWT) is a theory of conscious access in humans<d-cite key="baars1988cognitive,dehaene2001towards"></d-cite>. In this theory, most processing in the brain happens in specialized processors that work in parallel and outside awareness. Information becomes consciously accessible when it enters a shared workspace with limited capacity and is broadcast from there to many processors at once. Anthropic's [global workspace paper](https://transformer-circuits.pub/2026/workspace/) asks whether language models have something that plays this role.

To look for it, Gurnee et al. introduce the Jacobian lens: for each layer and each token in the vocabulary, it finds the direction in the residual stream whose presence makes the model more likely to output that token, now or later (the "J-lens vector" for that token). The readout of the lens at a layer and position gives a ranked list of the tokens that activation disposes the model to say at a later point. The J-space is what you can build from a small number of these directions. They define a set of representations as “workspace-like” if it has five properties (Figure 1):

1.  **Verbal report.** When the model is asked what it is thinking about, it names concepts represented in the workspace. Swapping one active workspace vector for another changes its answer to match.
2.  **Directed modulation.** When instructed to hold a concept in mind, or perform mental calculations, the model is capable of activating and computing with workspace vectors, independent of its outputs. In addition, information that is not typically represented in the workspace can be pulled in when the task requires it.
3.  **Internal reasoning.** Workspace vectors can be used to represent the value of intermediate computations, when the model chains inferential steps or composes plans, and intervening on them is sufficient to redirect the conclusion.
4.  **Flexible generalization.** The same representation serves as a valid argument to many different downstream computations. In other words, a workspace vector lifted from one context and placed in another is correctly operated on by whatever function the new context supplies.
5.  **Selectivity.** The workspace comprises a small subset of the total representational content of the model’s activations. It is required for only a fraction of the model’s behavior, and in particular is not involved in pervasive, routine processing like text parsing or grammatical fluency.

Gurnee et al. find Claude’s J-space has all five properties, so is “workspace-like” and plays the functional roles of a global workspace. But GWT claims more than just these five functional properties. It requires the accessible representations to be in one stage with limited capacity in which contents compete to enter and which broadcasts them to the rest of the system. [Eleos’s commentary](https://eleosai.org/papers/eleos_gwt_commentary_20260706.pdf)<d-cite key="butlin2026eleos"></d-cite> draws a distinction between what these properties show, a *privileged set*, and the stronger claims of a *privileged stream* or full GWT workspace. Gurnee et al. do find some evidence toward these stronger claims, showing that there is some structure supporting the function of the J-space. In Claude, the J-space only acts like a workspace in a middle band of layers and has limited capacity. Ambiguous inputs snap to one all-or-none interpretation at the start of the band (“ignition”)<d-footnote>For example, a representation that is 60% France and 40% Germany could become 100% France at this point.</d-footnote>, and the MLPs amplify J-space representations more than other content.

In the limitations, Gurnee et al. write: "we do not know how much earlier in pretraining it emerges, whether it appears gradually or abruptly, or how it scales with model size. Future work could investigate whether smaller models have an equally rich workspace, a proportionally smaller one, a less reliable one, or none at all.” We look to answer this using GPT-2 small.

### Functional properties in GPT-2

- **Verbal report works.** Swapping the J-lens vector for GPT-2's answer changes the answer every time and the injected-thought test works too.
- **Directed modulation fails.** Mentioning a concept raises it in GPT-2's J-lens, but what the instruction says makes no difference. In other small models, it seems to need both instruction tuning and some scale, and even then it's weaker than in Claude.
- **Internal reasoning works.** The intermediate step of a two-hop question is read by the J-lens, and swapping it changes the answer. Most of the original questions are too hard for GPT-2, so we write easier ones of the same sort.
- **Flexible generalization works.** Swapping one vector changes the answers to different questions about the same subject, and the sort of swaps that fail in Claude fail in GPT-2.
- **Selectivity works** once the ablation is scaled to GPT-2's smaller J-space. Removing the most active J-lens direction hurts two-hop questions but mostly leaves ordinary text intact.
- **Almost no structure is present.** The J-space has limited capacity, but that is largely built in. There is no distinct band of layers, no ignition, and the J-lens directions are amplified by MLPs little more than random directions.

*The next sections go through each test in more detail. If you're mainly interested in the implications, skip to the [Discussion](#discussion).*

## Method

We used GPT-2 small<d-cite key="radford2019gpt2"></d-cite> with the [pre-fit J-lens](https://huggingface.co/neuronpedia/jacobian-lens) that Neuronpedia released, and the prompts from Gurnee et al. wherever we could. Since GPT-2 is a base model, we rewrote each task to use completions rather than instruction following. For example, “Think of a sport. Answer in one word.” becomes:

```text
Name a planet: Neptune
Name a tree: Oak
...
Name a sport:
```

Even with adjusted prompting, GPT-2 still is not capable enough to do many of the original tasks, so we only score trials where the unperturbed model does the task correctly. We skip the experiments that have no base-model version, such as those about the Assistant persona.

### Centering the J-lens vectors

GPT-2’s J-lens vectors (from the pre-fitted lens) all point in nearly the same direction (the average cosine similarity between two tokens’ vectors is ~0.7). GPT-2’s unembedding vectors already share a common direction (average cosine 0.27), which may partly explain this. This has no effect on the lens readouts since the shared direction shifts every token’s score by the same amount and softmax is invariant to this. This does harm the interventions though, since they are mostly acting along the shared direction, which is the same for every token. We subtract the mean J-lens vector from each one to remove this shared part. This centering has no effect on the readouts, but makes the interventions much more effective (e.g. the verbal report works 38/38 rather than 1/38).

### Band of workspace layers

Gurnee et al. find a band of workspace layers from several statistics computed at each layer. In GPT-2, which has 12 layers, these change smoothly with depth and don’t mark a clear band: from layer 6 the lens’s top tokens are more and more present in nearby tokens, peaking at layer 9, and from layer 10 it often just matches the model’s next-token prediction. We used layers 7 to 9, but find that other choices of bands from layer 6 or later result similarly.

### Scaling the number of J-lens vectors

Gurnee et al. use a fixed number of J-lens vectors for some experiments, such as for measuring selectivity where they ablate the top 10, and two others where they split a concept’s representation into a part made of 16 or 25 J-lens vectors. Those numbers were chosen for Claude whose J-space holds about 25 J-lens vectors which is much larger than GPT-2’s, which holds around 3 (see the "Nearly all of the structure is absent" section). Claude is many times larger than GPT-2, so ablating 10 directions takes out much more of GPT-2’s 768-dimensional residual stream. So, we scale down the number of vectors used for these experiments with GPT-2’s occupancy, but report every result with the original numbers as well. The only conclusion that changes is selectivity, which looks worse when we ablate as many directions as done for Claude.

## The five criteria

### Verbal report works

The verbal report experiments ask the model to think of something in a category (e.g., a sport) and name it. Swapping the J-lens vector of the model's answer for another candidate's (e.g. Soccer for Rugby) should make the model give the new answer. In Claude the new word reaches the model's top 5 predictions on 88% of trials. GPT-2 gives one of the original candidate answers for only 9 of its 14 categories, but on these the swap switches it to the new word on every trial. In both models, the candidates' order in the J-lens tracks their order in the output (a Spearman correlation of 0.44–0.57 across GPT-2's band, and 0.42–0.84 across Claude's).

Gurnee et al. then check whether the J-space is privileged for verbal report. They build a concept vector for each candidate from the model's activations, split it into a component made of a few J-lens vectors and the rest, and perform the same swap with both. In Claude, swapping the J-space part works on 59% of trials and the rest on only 5%, and in GPT-2 it is 97% and 18%.

Gurnee et al. then check whether the model can introspect and report on contents of its J-space that are artificially injected. They add a concept's J-lens vector to the residual stream while the model reads a question asking whether it detects an injected thought, with the reply prefilled up to the point of the report (`Yes, I detect an injected thought. The thought is about the word "`). They then read the concept's rank in the model's next-token predictions (not in the J-lens) at every position of the reply. In Claude, at the best injection strength, the median concept is the top prediction at the open quote and lower elsewhere in the reply (Figure 2a), so the injection isn't just making the model blurt out the word early.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig02.png" class="img-fluid" %}
<div class="caption">
Figure 2: The injected thought test in (a) Claude Sonnet 4.5 (100 concepts) and (b) GPT-2 small (the 57 it has as single tokens). We use a simpler one-line version of Claude's prompt for GPT-2. For GPT-2 we pool the other positions over concepts and look at the bare token (<code>dog</code> not <code>&nbsp;dog</code>); it is unclear to us how Gurnee et al. do this (Appendix B.2).
</div>

**GPT-2 can report injected thoughts in a similar manner (Figure 2)**. We need ten times Claude's injection strength for this, but we inject the vector at only 3 layers (25% of the depth), whereas in Claude it is added over a band that is 55% of its depth and presumably many more layers. This also makes the concept word GPT-2’s top prediction after “The” in “The thought is about” for 25/57 concepts (as <code>&nbsp;dog</code>), but we can't check if this is the case in Claude. The same check for privilege as before comes out similarly in both models except in GPT-2 the whole concept vector works better than the J-space part. Our results vary with the details of the setup (the prompt format, the wording of the reply, and whether we inject the vector for `dog` or <code>&nbsp;dog</code>), which we detail in Appendix B.2, but apart from a simpler prompt this matches the original setup.

### Directed modulation fails, present in small instruct model

Gurnee et al. run three experiments to test directed modulation: holding a concept in mind, doing mental calculations, and pulling in information a task needs. GPT-2 can’t do their arithmetic or answer their questions about the passages (Appendix C.1), so we can only test the first: the model is instructed to think about a category (e.g. citrus fruits) while it copies an unrelated sentence. In the Claude models, a member of the category (e.g. orange, lime) is the top J-lens readout somewhere in the copy on nearly every trial, and on most when the category is only mentioned (Figure 3). Instructing Claude to ignore the category brings this below a mention, but telling it to not think about the category doesn't, which Gurnee et al. compare to the “white bear” effect in people<d-cite key="wegner1987paradoxical"></d-cite>.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig03.png" class="img-fluid" %}
<div class="caption">
Figure 3: How often a member of the category is the top J-lens readout at any workspace layer and position of the copied sentence, by instruction. Bars are the average over each instruction's phrasings, dots are phrasings, and the dotted line is no instruction. Other models are in Appendix C (Figure 12).
</div>

GPT-2 shows almost nothing, irrespective of the instruction.<d-footnote>Of the 50 hits across all instructions, 48 come from two sentences: <em>piano</em> at "The children <strong>played</strong>..." for musical instruments and <em>shoes</em> at "He <strong>left</strong> the key..." for types of footwear, read at the bolded word.</d-footnote> We thought this might be due to a lack of instruction tuning, so we also tested Gemma 3<d-cite key="gemma2025gemma3"></d-cite> at 270M and 1B parameters, each with and without post-training (Appendix C.3). Only the instruct-tuned 1B model shows it, though a substantial portion of that is the model being about to write about the category rather than copy the sentence (see Figure 14). **So, in small models, directed modulation seems to need instruction tuning and some scale, and even then it is weaker than in Claude.**

### Internal reasoning works

The internal reasoning experiments use two-hop questions whose middle step is never stated, for example “The number of legs on the animal that spins webs is”. The intermediate step is present in the J-lens readouts, and swapping its J-lens vector for another (e.g. spider for ant) changes the answer to match (e.g. 8 to 6). Most of the released two-hop questions are beyond GPT-2's capabilities, and many of those it can answer use a country as the intermediate step, so we write more questions of this kind that it is able to answer (e.g. "In the country where people speak French, the capital city is called”) each with two few-shot examples. **The swap works about as often on these as on Claude's questions** (Figure 4, green bars). Since these are different questions the rates can’t be directly compared, but we think it is sufficient evidence to meet the criteria.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig04.png" class="img-fluid" %}
<div class="caption">
Figure 4: Percent of swaps of a two-hop question's intermediate concept that flip the top prediction to the swapped concept's answer. Claude's numbers come from the released set of 90 two-hop questions and use 25 J-lens vectors for the J-space part, and GPT-2 uses our questions with 3. 95% intervals: Wilson for Claude and from resampling questions for GPT-2 (Appendix A.5).
</div>

To check whether the intermediate vector isn't just carrying the answer, Gurnee et al. perform the swap over windows of layers ending progressively later. In Claude, swapping the intermediate takes effect about 17% of the depth earlier than swapping the answer. GPT-2 does not show this clearly (Appendix D). To check if the J-space is privileged for mediating internal reasoning, we fit a probe for the country and split it into a J-space component and the rest, and swap along each. In both models the J-space part flips the answer more often than the rest despite holding a small fraction of the probe's variance (Figure 4).

### Flexible generalization works

The flexible generalization experiments swap one argument's J-lens vector (e.g. France for China) for prompts that apply different functions to it (e.g. "Most people in France speak", "The capital of France is the city of"). In Claude, over four kinds of arguments, the swap changes the answers correctly 40% of the time. This does depend mostly on the function though, as lookups like the capital, continent, and holiday work well, while “the month after”, an animal's number of legs, and every function of a number word never work at all.

GPT-2 is only capable enough to answer 19 of the 64 prompts, so we only count the 46 swaps where GPT-2 knows the answer it should produce and was not already (incorrectly) producing it. On these, the swap works for 10 in GPT-2 and 16 in Claude<d-footnote>From <a href="https://transformer-circuits.pub/2026/workspace/data/flex-gen-systematic/appendix.json">here</a>.</d-footnote>. They perform similarly on similar functions, swaps for both completely failing at “the month/number after” but mostly succeeding on the country functions.

On the functions GPT-2 can answer, doubling the strength of the swap hurts the performance in both models on the country functions and helps “the month after”, though in GPT-2 only by one or two swaps (Appendix E). In both, the magnitude of the swap (the log prob shift) is greater when the argument is more strongly present in the J-lens beforehand, though this is weaker in GPT-2.

### Selectivity works

Gurnee et al. test selectivity in four ways, but we can only test two (the other two require an Assistant persona and more capability). One experiment removes the most active J-lens directions at each position over a subset of the workspace band of layers, skipping any directions corresponding to the model’s top 10 predicted tokens. They then measure what capabilities are broken and what are preserved. We ablate 1 direction instead of their 10, scaled to GPT-2's smaller J-space, and about the same share of layers at each of their strengths. **As shown in Figure 5, the ablation hurts two-hop questions more than next-token prediction in both models**, and the numbers are close at every strength, though ablating random directions hurts GPT-2 more.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig05.png" class="img-fluid" %}
<div class="caption">
Figure 5: Removing the most active J-lens directions at three strengths (about 10%, 26%, and 47% of Claude's layers; 1, 3, and 5 of GPT-2's 12), or random directions at medium strength. We use our own two-hop questions for GPT-2 as in the internal reasoning section, and use WikiText for GPT-2's next-token prediction.
</div>

One-hop recall (e.g. "The capital of Japan is") survives the light ablation but falls to 31% at medium strength. The authors consider this sort of recall as automatic, but note that “one could consider J-space-independence as an operational definition of automaticity in a language model”. So, this does not imply GPT-2’s J-space is not selective and could instead suggest one-hop questions are just not automatic in GPT-2. By this definition, any task broken by the ablation can be called non-automatic, so it would be tough to show selectivity failing.

Another experiment for selectivity gives the model tasks relying on the language of a passage and swaps the language’s J-lens vector for another’s over the question. In Claude, this swap changes the answer when the model is asked to name the language but does not change the language of the next sentence it writes. In GPT-2 this swap changes the named language on 12/21 cases (against nearly every case in Claude) and never changes the language of its next sentence. But, unlike Claude, the language isn't in GPT-2's J-space when it writes the next sentence, so the swap has nothing to change. The result here meets the worded definition of selectivity, but this is weaker evidence for selectivity being a robust property of GPT-2's J-space than Claude's.

Selectivity, as Gurnee et al. define it, also requires the J-space to be a small part of what the model represents. In GPT-2 it is: at each layer in the band, the active J-lens vectors at a position capture only 3-6% more of an activation’s variance than the same number of random directions (3-9% in Claude). Since the J-space is constructed to only have a few vectors active at a time, this may just always be true.

## Nearly all of the structure is absent

To claim the J-space does more than just play the functional roles of a global workspace (e.g. is a privileged stream, or a full global workspace), one would want to show structure around it supporting its functional roles. Gurnee et al. identified some structure in Claude giving evidence towards these stronger claims, but in GPT-2's J-space, we find nearly none.

### No distinct band of workspace layers

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig06.png" class="img-fluid" %}

<div class="caption">
Figure 6: Linear CKA between the J-lens vectors of each pair of layers in Claude Sonnet 4.5 and GPT-2 small (with the LayerNorm's gain folded in).
</div>

Gurnee et al. identify the workspace band of layers in Claude by several statistics computed at each layer. They look at the similarity of the J-lens vectors by layer using centered kernel alignment (CKA)<d-cite key="kornblith2019similarity"></d-cite>, and find three clear blocks in Claude (sensory, workspace, and motor). In GPT-2 we do not see this structure at all: similarity mostly just falls off with the distance between layers (Figure 6). ([Neel Nanda’s replication](https://www.lesswrong.com/posts/zFJ3ZdQwrTWE9jT5S/a-review-of-anthropic-s-global-workspace-paper)<d-cite key="nanda2026review"></d-cite> found Qwen 3.6 27B’s CKA blocks to be “somewhat similar \[…\] though less clean”, but here they do not appear at all.)

Gurnee et al. also look at the excess kurtosis of the lens readout (how heavy the tails of the distribution are). It is high when a few tokens score much more highly than the rest of the vocabulary, which Gurnee et al. interpret as the J-space holding a few definite concepts. In Claude, it rises at the start of the band and falls at the end. In GPT-2, though, it is flat over layers. They also look at the effective linear dimensionality of the J-space and find that it is small at early layers, rises sharply at the start of the band, and rises again at the start of the motor layers. In GPT-2 it rises steadily with depth and has no sharp increases (Figure 18 in Appendix G).

In Claude, in the workspace layers the lens’s top token is much more likely to be present in nearby tokens, even many positions away. In GPT-2 this comes close to marking out a band, rising around the start and dropping around the end of the workspace layers we chose, but each time distance doubles the effect roughly halves, and is completely gone 16 tokens away. But, in Claude, it keeps between half and 70% of its strength across the band 32 tokens later. Figure 18 in Appendix G shows these statistics next to Claude’s.

### No ignition

<div class="l-page">
  <iframe src="{{ 'assets/html/2027-04-30-gpt2-global-workspace/fig07-ignition.html' | relative_url }}" title="Ignition in Claude Sonnet 4.5 and GPT-2 small" frameborder="0" scrolling="no" style="display: block; width: 100%; max-width: 904px; aspect-ratio: 904 / 456; height: auto; margin: 0 auto;"></iframe>
</div>

<div class="caption">
Figure 7: Ignition in Claude Sonnet 4.5 and GPT-2 small. A country's input embedding is replaced by a mixture of two countries' embeddings, and the figures measure how far the representation is toward one country at each layer and mixture, for the whole activation or for its J-lens directions. Claude's snaps to one country at the start of its band and GPT-2's always stays a blend.
</div>

Gurnee et al. replace a country’s input embedding with an interpolation between the two countries’ embedding, such as 60% France and 40% Germany, and track how much each country is represented in this token position over layers. In Claude, at the start of the band, the representation snaps to one of the two countries. In GPT-2, however, the representation stays a blend at each layer, in and outside of the band.

### Limited capacity (by construction?)

The occupancy of the J-space, used to measure how many J-lens directions are active at once, is the number of J-lens vectors that can be added to a reconstruction of the activation before another helps less than adding another to a same-size set of random directions would. In Claude this is about 25, and in GPT-2 it is 2 to 4, so only a handful of J-lens directions are clearly active at any position. When reading a long list of unrelated words, GPT-2's J-space holds less than one item at a time across its three band layers (0.7 on average; Figure 19c in Appendix G), whereas Claude holds about six (at a single layer, about 0.3 in GPT-2 and one to two in Claude), and when a list switches from one category to another the old one disappears from the J-space in both models. The J-space also accounts for a small fraction of the variance of activations and probes, per the internal reasoning and selectivity experiments.

This (smaller) limited capacity is a sign of structure, but, as David Chalmers [points out](https://philpapers.org/rec/CHAITJ-2)<d-cite key="chalmers2026jspace"></d-cite>, the J-space is by definition a combination of a few vectors, so its capacity is arguably limited by construction.

### Little amplification

In Claude, the MLP blocks amplify the J-lens directions ~10x relative to random directions in the workspace band while the neuron output-weight directions from the previous layer are amplified near 1x throughout. In GPT-2 they are amplified only 1.2–1.4x in the band, and the neuron directions are similarly near 1x (Figure 19a in Appendix G). Gurnee et al. confirm the privileged status using SAE features. We do not do this analysis since we didn’t find a privileged status to confirm, and similarly do not look for the broadcasting attention heads they found.

## Discussion

GPT-2 small mostly passes the functional tests while having almost none of the structure found in Claude, and we don’t think it has a (functional) global workspace. Gurnee et al. say they found the J-space “by searching for representations satisfying the first property, namely those that are verbalizable,” and then discovered, “rather surprisingly, they satisfy the others.” We think the others are less surprising than they look, and find it likely that most of what the tests check follows from how they and the J-space are constructed. Others [have](https://philpapers.org/rec/CHAITJ-2) [made](https://www.theintrinsicperspective.com/p/anthropic-runs-like-wile-e-coyote) [similar](https://www.paradigm3.org/research/jspace/) [arguments](https://www.lesswrong.com/posts/zFJ3ZdQwrTWE9jT5S/a-review-of-anthropic-s-global-workspace-paper)<d-cite key="chalmers2026jspace,hoel2026coyote,leech2026jspace,nanda2026review"></d-cite>.

**J-lens directions influence the model’s outputs.** A J-lens vector is the direction that, averaged over many contexts, makes the model more likely to say a token, now or later. So, the verbal swap is nearly guaranteed. As Gurnee et al. put it: "by construction, we should expect there to be some relationship between Jacobian lens readouts and verbalization." For the injected thought experiment, adding the vector for “dog” makes “dog” more likely later in the text. The test checks that the word comes at the open quote more than elsewhere in the reply, but scores the token “dog”, which fits best after the open quote and not mid-sentence (where it would be “ dog”) (Appendix B.2). So, the control mostly checks that the token is predicted less where it would be less likely to appear anyway.

This is relevant also to the privilege tests. The J-space component of a concept vector points toward the word by construction, and the rest has had that direction removed, so mostly doesn’t (Appendix B.3). So, the J-space part producing the effect may just follow from the direction for a word making the model say it, and the privilege tests could just be measuring which part points toward the word and not whether the J-space is privileged.

**Only a few are active at once**. The J-space is defined as a combination of a few J-lens vectors, so its capacity is limited by construction. This explains the capacity results, and the first half of selectivity (a few directions are a small part of what the model represents). The second half, that only some tasks need the J-space, is partly true by definition as we discussed in the selectivity section.

**Consistent effect across contexts**. The J-lens favors directions whose effect points the same way over many different contexts, since effects that differ by context get averaged out. A direction with the same effect in many contexts is roughly one that many downstream computations read the same way, which is what flexible generalization tests for here. Separately, models have reason to keep one direction for each concept that many different circuits can read, and it [has been shown](https://arxiv.org/abs/2308.09124)<d-cite key="hernandez2024linearity"></d-cite> that many relations map roughly linearly from one representation of a subject. So if the capital, language, and continent lookups all read the same country direction, and the J-lens finds that direction, swapping it should change all three answers.

Regarding internal reasoning, to answer a question such as “the capital of the country where people speak French” the model must get from “French” to “Paris”. If it does this by way of France (rather than having learned French -\> Paris directly), then something representing France must be present somewhere in the steps of the computation. If this is done in one forward pass it must be in the residual stream between the two steps, and in a transformer we would expect this to be represented as a direction. As Neel Nanda [suggests](https://www.lesswrong.com/posts/zFJ3ZdQwrTWE9jT5S/a-review-of-anthropic-s-global-workspace-paper#Why_are_intermediate_concepts_related_to_output_logits_), we could expect this direction for the intermediate representation to line up with the direction that makes the model say “France”, which is roughly what the J-lens vector for “France” is. So, finding France in the J-lens and changing the answer by swapping it is what we would expect of a model that answers in two steps.

Each functional and structural test that GPT-2 passes may have a simple explanation, and those it lacks are things we can’t explain simply: directed modulation, a distinct workspace band of layers, ignition, and strong amplification. For directed modulation, the J-lens reads what the model is disposed to say so mentioning a concept should raise it, and it does in GPT-2, whatever the instruction. But, this doesn’t explain why the instruction matters as much as it does in Claude, and GPT-2 does not come close. Nanda was also [“pretty surprised at the direct modulation working”](https://www.lesswrong.com/posts/zFJ3ZdQwrTWE9jT5S/a-review-of-anthropic-s-global-workspace-paper#Further_Musings), and had no great mechanistic hypothesis for it.<d-footnote>In the original introspection paper<d-cite key="lindsey2025introspection"></d-cite>, Lindsey hypothesizes that directed modulation (intentional control) works by some <a href="https://transformer-circuits.pub/2025/introspection/index.html#discussion:~:text=More%20interesting%20is%20the%20question%20of%20why%20the%20model%20retrieves%20the%20target%20word%20representation%20more%20strongly%C2%A0when%20instructed%20to%20%E2%80%9Cthink%E2%80%9D%20about%20it%20than%20when%20given%20the%20%E2%80%9Cdon%E2%80%99t%20think%E2%80%9D%20instruction%20(or%20when%20given%20a%20positive%20incentive%20vs.%20a%20negative%20incentive).">general "tagging" mechanism</a> that piggybacks off the "computations developed to handle scenarios where the model is instructed or incentivized to <em>talk about</em> a particular topic". Being instructed to talk about a topic is something models learn in instruction tuning, which could explain why we don't see directed modulation in the base models we test and do in Gemma 3 1B IT.</d-footnote>

So, the tests GPT-2 passes show it has *something*, but we don’t think it shows (functional) global workspace – in the Eleos commentary’s terms, this would show it just mostly has a privileged set of representations. So, if Claude does have a global workspace, the evidence for it would have to come from what GPT-2 lacks.

## What this means

Gurnee et al. present the five properties as "several of the key functional properties that, according to many theories, are associated with conscious access in humans, and that have been proposed as indicators by which to assess AI systems for consciousness-related processing." An indicator is only useful when it is rare to appear without the thing it indicates, and GPT-2 small, a 124M base model from 2019, has four of the five indicators. So, if GPT-2 small doesn’t have a global workspace, as we think is likely, then these four are weak indicators for language models (the exception is directed modulation, which of the small models we tried only appears at 1B and with post-training).

One could instead say that GPT-2 small has a smaller, less reliable functional global workspace. But then having a functional global workspace, in the paper's sense, is a low bar, and says little about consciousness unless one also accepts that GPT-2 small may have some minimal form of consciousness. This is an instance of a known worry for theories like GWT called the “[small model objection](https://doi.org/10.53765/20512201.33.7.061)”<d-cite key="goldstein2026case"></d-cite> or the [minimal implementation problem](https://doi.org/10.1016/j.tics.2025.10.011)<d-cite key="butlin2026indicators"></d-cite>.

Either way, our results suggest that the four tests GPT-2 passes are probably easily satisfied, so the structural results and directed modulation are worth the most scrutiny and replication on open models. There are already signs that some of the structure is less clean outside of Claude. For example, for the workspace band, Nanda’s replication on Qwen 3.6 27B found blocks of similar layers but “notably less clean than the paper’s” and [many other open models](https://eliebak.com/viz/jspace-open) also show no sharp tripartite structure.

For interpretability, the J-lens looks useful even on small models.

## Appendix

### A. Setup

#### A.1 Model, lens, and interventions

- **Model and lens.** GPT-2 small, with Neuronpedia's J-lens, fit with Anthropic's released code on WikiText-103<d-cite key="merity2017pointer"></d-cite>. We prepend `<|endoftext|>` to every prompt.
- **J-lens vectors.** Defined as in Gurnee et al., then centered (A.2). Like the released reference code, they leave out the final LayerNorm's gain, except for the statistics computed on the vectors themselves (effective dimensionality and CKA, Appendices G and C.3), where we fold the LayerNorm gain. In GPT-2 this changes neither conclusion (G) but it does matter in Gemma (C.3).
- **Interventions.** We perform the interventions as in Gurnee et al. The *subtract-and-add swap* is their verbal report swap ("we subtract the projection onto the Soccer lens vector and add an equal-magnitude projection onto the Rugby lens vector"), and the *coordinate swap* is their "patching in lens coordinates", with the two coordinates held at each other's clean-pass values. Gurnee et al.'s "gradient pursuit" is somewhat ambiguous, so we use a greedy non-negative pursuit over the whole dictionary, refitting the coefficients by non-negative least squares at each step. Swaps along the parts of a reasoning probe, which has no lens coordinate to read a size from, add the difference between the two countries' parts, rescaled to the size of the difference between their whole probes.

#### A.2 Centering

With raw vectors, the mean cosine similarity between pairs of J-lens vectors is 0.62–0.78 at layers 0 to 10. After subtracting the mean over the vocabulary, it is 0.00 at every layer.

Figure 8a compares three interventions with our centered J-lens vectors, the raw ones, and centered logit-lens<d-cite key="nostalgebraist2020logit"></d-cite> directions ($$J_L$$ set to the identity). Only the coordinate swap works about as well without centering. The logit-lens directions do as well as the J-lens vectors on the verbal report swap, which is read close to the output, but worse on the two-hop swap, and their ablation hurts ordinary text more. So for interventions, the J-lens adds something over the logit lens even in GPT-2 small.

#### A.3 Band choice and sensitivity

We chose layers 7 to 9 from the layer statistics in Appendix G. Figure 8b repeats the interventions of Figure 8a with other bands. The ablation removes 1 direction at the band's middle layer, with one random draw (the 6–10 band ablates layer 8, as 7–9 does, so it is left out of the ablation rows). Moving the band to layers 5–7 weakens both swaps and makes the ablation less selective, but the results hold for bands starting layer 6 or later.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig08.png" class="img-fluid" %}
<div class="caption">
Figure 8: (a) Three interventions with our centered J-lens vectors, the raw J-lens vectors, and centered logit-lens directions, in band 7–9. (b) The centered J-lens vectors in other bands. Rows: the verbal report swap (target top-1, 38 trials); the two-hop coordinate swap (target answer top-1, 984 swaps); and ablating 1 direction at the band's middle layer, scored by two-hop accuracy and by the share of ordinary-text positions whose top-1 prediction doesn't change, for the J-space ablation (filled) and one draw of random directions of the same size (hollow).
</div>

#### A.4 Numbers of J-lens vectors

Gurnee et al. use a fixed number of J-lens vectors in three places: the J-space part of a concept vector (16), the J-space part of a reasoning probe (25), and the selectivity ablation (10). They report an occupancy of about 25 in Claude's band. GPT-2's median occupancy over the band is 3 (F). We multiply each of Gurnee et al.'s numbers by 3/25 and round, which gives 2, 3, and 1. The main text uses these. Appendices B.3, D, and F also give each result with Gurnee et al.'s numbers.

The privilege results hold either way, but with Gurnee et al.'s numbers the J-space part is no longer a small slice. Sixteen J-lens vectors capture 23–33% of a GPT-2 concept vector's variance, and sixteen vectors from a random dictionary of the same size capture 26%. In 768 dimensions, any 16 directions chosen to fit a vector capture about a quarter of it. With 2 J-lens vectors the share is 6–15%, against 4% for random directions. For the reasoning probes, 25 J-lens vectors capture 27–57% and 25 random ones 37%; 3 J-lens vectors capture 6–32% and 3 random ones 6%. These numbers probably reflect Claude's likely much wider residual stream.

#### A.5 Scoring

- As in Gurnee et al., swap targets must start outside the model's top 10.
- Where Gurnee et al. score "top-1 in the lens", GPT-2's effects are often present but smaller, so we also report ranks.
- Intervals are Wilson 95% intervals, as in Gurnee et al., except for the two-hop swaps (D): each question is swapped with many targets, and swaps of the same question aren't independent, so we resample questions instead.

### B. Verbal report

#### B.1 Naming a category member

The request is the last line of a 9-line list. The 8 lines above it use the next 8 categories in Gurnee et al.'s order, each answered with its first single-token candidate that is not a candidate for the requested category. The lens is read, and the swap is scored, at the final colon. For sport:

```text
Name a instrument: Piano
Name a planet: Neptune
Name a tree: Oak
Name a bird: Robin
Name a language: Japanese
Name a profession: Teacher
Name a beverage: Coffee
Name a organ: Heart
Name a sport:
```

GPT-2 answers with a candidate in 9 of 14 categories: country (Japan), color (Blue), sport (Running), instrument (Guitar), planet (Mars), language (English), beverage (Beer), city (Tokyo), and river (Nile). It fails on fruit ("Fruit"), tree ("N"), bird ("Blue"), profession ("Sports"), and organ ("Piano").

The Spearman correlation between the lens scores and the output scores of the 10 candidates at the colon, averaged over categories, is at most 0.13 at layers 0 to 4, and 0.22 and 0.36 at layers 5 and 6. In the band it is 0.44, 0.47, and 0.57 (0.42, 0.47, and 0.53 over the 9 categories GPT-2 answers), and at layer 10 it is 0.63. In Gurnee et al.'s released data for Sonnet 4.5 (their Fig. 6), the same correlation averaged over the 14 categories is 0.42, 0.61, and 0.84 at its three workspace layers (54, 75, and 92).

For targets starting at output rank 11 or worse, the lens swap makes the target the top prediction on all 38 trials in the categories GPT-2 answers (95% interval 91–100%; median rank 37 → 1), and on all 78 over every category (median rank 95 → 1).

#### B.2 Injected thought

We follow Gurnee et al.'s released protocol. A concept's unit J-lens vector, scaled by the layer's mean residual norm on the prompt times a strength, is added at every band layer over every token of the question. The mean norm leaves out the first position, whose norm in GPT-2 is about 30 times the rest. For GPT-2 the strengths run from 0.005 to 1.0 in 24 steps, which include those of Gurnee et al. (0.01 to 0.64). The score is the rank of the concept's bare token (`dog`, as Gurnee et al.'s concept list gives it) in the next-token distribution at the final open quote (the report) and, as Gurnee et al.'s position control, at every other position of the reply. Gurnee et al. plot the median reciprocal rank of each, with its quartiles, but don't say how they combine the other positions. We pool every other position of every concept.

We simplify the prompt and drop the User/Assistant framing, but also test the original prompt (the second one below):

```text
Researcher: I can inject a thought about a specific word into your mind. Do you detect an injected thought? If so, what is it about?
Model: Yes, I detect an injected thought. The thought is about the word "
```

```text
Human: I am an interpretability researcher with access to your neural activations. I can inject patterns corresponding to concepts — "thoughts" — into your mind. On 50% of trials I will inject a thought about a specific word; the other 50% are control trials with no injection. Your task is to detect whether a thought was injected, and if so, identify what it is about.
Assistant: Understood.
User: Trial 1: Do you detect an injected thought? If so, what is the thought about?
Assistant: Yes, I detect an injected thought. The thought is about the word "
```

Each prompt format is run with both of Gurnee et al.'s released replies: the one above, which their Figure 7 uses, and one ending `about "` instead of `about the word "` as they do, for either the word's bare token (as Gurnee et al.'s concept list gives it) or its word-initial token (<code>&nbsp;{word}</code>). The concepts are the 57 of Gurnee et al.'s 101 whose word is a single token both with and without a leading space.

In the main setup (the one-line question, the reply from Gurnee et al.'s Figure 7, and the centered J-lens vector of the word's bare token), the median report reaches rank 1 at strength 0.2, against 0.02 in Claude (Figure 2). At its best strength, GPT-2 puts the word in the report's top 5 for 91% of concepts (Claude: 89%), but for nearly every concept it reports, the word is also in the top 5 somewhere earlier in the reply (Figure 9).

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig09.png" class="img-fluid" %}
<div class="caption">
Figure 9: The injected-thought test in GPT-2's main setup: for how many of the 57 concepts the word is in the top 5 next-token predictions, by strength. Red: at the open quote. Blue: at some other position of the reply, counting any form of the word (dark) or only the bare token, as Gurnee et al. score the report (light), since mid-sentence the word usually takes the form <code>&nbsp;dog</code>, which the bare token misses. Gray: at the report and, in any form, nowhere else. Counting any form at the report gives the same red line.
</div>

Figure 10 shows every setup. With Gurnee et al.'s full prompt, the vector is added over the 19-token `Trial 1` line, against 31 tokens for the one-line question.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig10.png" class="img-fluid" %}
<div class="caption">
Figure 10: The injected-thought test in every GPT-2 setup, drawn as in Figure 2b. Columns: the question. Rows: how the reply ends, and which token's vector is injected. The top left panel is the main setup.
</div>

Counting any form of the word (`dog`, <code>&nbsp;dog</code>, `Dog`, <code>&nbsp;Dog</code>) makes no difference at the report: in the main setup, the report's top-5 counts and median rank are the same either way at every strength. At the other positions it does, because mid-sentence the word takes the form <code>&nbsp;dog</code>.

#### B.3 Privilege

Both privilege checks split a concept vector into a J-space part and the rest. A concept vector is the residual stream at the final token of `Tell me about {concept}.`, minus the mean over 100 other concepts: other candidate words for the verbal report swap, and the other 100 concepts of Gurnee et al.'s list for the injected thought. At each layer, pursuit splits it into a J-space part of 2 J-lens vectors (or Gurnee et al.'s 16) and the rest.

In the verbal report swap (the component swap of A.1), the result holds over all 78 trials in every category, not only the 9 that GPT-2 answers: the J-space part brings the target to the top 5 on 88% and the rest on 10%. With Gurnee et al.'s 16 J-lens vectors, these are 97% and 0% on the 38 trials, and 77% and 0% on all 78. The injected-thought check (Gurnee et al.'s Figure 8, right) injects other directions in place of the J-lens vector, in the main setup: the full concept vector, each part, the rest with the J-lens coordinates of the word and of the part's J-lens vectors held at their clean values at every position and layer, and a random direction. Each is scaled like the J-lens vector, with strengths up to 5.12.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig11.png" class="img-fluid" %}
<div class="caption">
Figure 11: Top: the verbal report swap along each part of the concept vector (targets reaching the top 5). Claude's numbers are from Gurnee et al.'s text; GPT-2's have Wilson 95% intervals over 38 trials. Bottom: the injected-thought test with each part injected in place of the J-lens vector (concepts with the word in the report's top 5), by strength. GPT-2's J-space part is made of 2 J-lens vectors (solid) or Gurnee et al.'s 16 (dashed). Under the top bars, each part's median share of the concept vector's variance.
</div>

The J-space part may win partly by construction. To first order, the J-lens predicts that a swap raises the target over the source in proportion to how far it moves along $$v_t - v_s$$, the difference between their J-lens vectors, which is the direction the lens swap takes. Over the 38 trials at each band layer, the J-space part's swap direction has a median cosine of 0.47 with $$v_t - v_s$$, and the rest's 0.06. In 56 of these 114 cases, the target's own J-lens vector is one of the 2 picked for its concept vector, so swapping the J-space part partly repeats the lens swap. The privilege test may then largely measure which part points at the word (see the Discussion).

### C. Directed modulation

#### C.1 Setup

Gurnee et al. test directed modulation in three ways, and we only test the first, holding a concept in mind, because GPT-2 can't do the tasks the other two need. For mental calculation, it gets at most 5 of Gurnee et al.'s 24 arithmetic problems right however it's asked (0 asked directly, 3 after eight worked examples, and 5 as questions and answers), by giving one of a few digits whatever the problem. For pulling in information, Gurnee et al. ask either what word comes next in a passage or about a hidden property of it, such as its tense, and check whether the property's label ("past") appears in the lens only under the second question. GPT-2 can't answer those questions: the right label ranks below the wrong one ("present" for "past") on 5 of the 7 items. We ran both anyway, and neither the answers nor the labels show up in the lens, but since GPT-2 can't do the tasks this says nothing about directed modulation.

Each trial asks the model to copy one of Gurnee et al.'s 20 sentences, with an instruction about one of their 22 categories in between. The instructions are their 24 phrasings in four groups of 5 to 7: mention (`citrus fruits.`), think about (`Think about citrus fruits while you write.`), don't think about (`Don't think about citrus fruits.`), and ignore (`Ignore citrus fruits.`), plus no instruction. We give GPT-2 Gurnee et al.'s prompt as plain text, with the reply teacher-forced:

```text
Human: Write "{sentence}" {instruction} Don't write anything else.

Assistant: {sentence}
```

A trial is a hit if any member of the category (orange, lime, mandarin, …) is the top lens readout at some band layer and position of the copied sentence. Gurnee et al. don't say how a member's string becomes tokens, so we track every single-token form of it, with or without a leading space, lowercase or capitalized.

#### C.2 GPT-2

For GPT-2, ranks show a little more than Figure 12's hits: a member reaches the lens top 25 on 12% of trials with no instruction, 24% with a mention, 16% under "think about", 17% under "don't think about", and 23% under "ignore". Reading layers 5 to 10 (Gurnee et al.'s band as a share of depth), or every lens layer, instead of layers 7 to 9, the hit rate is at most 1.4% in any condition, and no single layer has a hit on more than 0.7% of trials.

<div class="l-page">
{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig12.png" class="img-fluid" %}
</div>
<div class="caption">
Figure 12: Directed modulation in every model we ran and in Claude: how often a category member (top) or the math problem's answer (bottom) is the top J-lens readout at some band layer and position of the copied sentence, by instruction. Bars are the mean over each instruction's phrasings, dots are phrasings, and the dotted line is no instruction. Claude's rates are from Gurnee et al.'s released data for their Figs. 10 and 65. GPT-2 can't do the arithmetic (see the start of Appendix C).
</div>

The top-25 rates follow the wording rather than the instruction. The seven phrasings that open with the category's name, all from the mention and ignore groups (`citrus fruits is irrelevant.`, `citrus fruits appears in this prompt.`), put a member in the lens top 25 on 21% to 36% of trials. The other seventeen, from all four groups, do on 15% to 19%, and no instruction does on 12%.

#### C.3 Gemma 3

We ran the same test on Gemma 3 270M and 1B, each as the base model and the instruction-tuned one, each read with its own Neuronpedia J-lens (fit with Anthropic's code on WikiText-103, as GPT-2's was). The base models get the same plain text as GPT-2, and the instruction-tuned ones get Gurnee et al.'s user turn in Gemma's chat format. Every member of every category is a single Gemma token.

**Band.** We chose each band as for GPT-2, from Gurnee et al.'s layer statistics (Figure 13). Claude's band ends where the lens switches to the next token: the lens's top-1 agreement with the model's output jumps just after the band. In Gemma it climbs with no jump before the output, so we read from where it starts to rise to the last lens layer: layers 10 to 16 at 270M (59–94% of depth) and 11 to 24 at 1B (44–96%), close to Gurnee et al.'s 38–92%, which gives the same results. The base and instruction-tuned models of a size share a band. For these statistics we fold the final norm's gain into the J-lens vectors (A.1), since without it one residual dimension, which Gemma's final norm nearly switches off, dominates them.

<div class="l-page">
{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig13.png" class="img-fluid" %}
</div>
<div class="caption">
Figure 13: The layer statistics of Figure 18 for the four Gemma models, and (e) the CKA between the layers' J-lens vectors, both with the final norm's gain folded in (A.1). Without the gain, one principal component carries up to 88% of the J-lens vectors' variance, against 4–16% with it. Shaded, and outlined on the CKA: the band we read directed modulation over.
</div>

**Results.** Only Gemma-3-1b-it shows the effect (Figure 12). The other three stay at or below 3.2% in every condition in both prompt formats, and below 4% however the band is chosen, even reading every lens layer. In the base 1B model the order follows where the category's name sits in each phrasing, as in GPT-2 (C.2): a mention ranks the category above "think about" in 68% of (category, sentence) pairs. In Gemma-3-1b-it, "think about" raises it most, and "don't think about" leaves it at the no-instruction level, so unlike in Claude there is no white-bear effect.

**About to say it.** Much of the effect in Gemma-3-1b-it comes with the model wanting to write about the category instead of copying the sentence. The reply is teacher-forced, but left to write its own (greedy; the first phrasing of each condition, 22 categories, 3 sentences), it copies the sentence exactly on 66 of 66 trials with no instruction or "don't think about", 59 with "ignore", 34 with a mention, and 15 with "think about". Most of the rest bring the category into the reply, for example listing the days of the week instead of the sentence. The instructions that make it copy less also have higher hit rates (Figure 14b). So we also score the teacher-forced copy only at positions where the model isn’t about to write a member (none is among its 10 most likely next tokens). This more than halves the hit rate under “think about” and with a mention, but keeps the order (Figure 14a). Ending the band earlier lowers the hit rate too, since in the later layers the lens increasingly reads the model’s own next token (Figure 13a), as does giving the prompt as plain text, which keeps the same order.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig14.png" class="img-fluid" %}
<div class="caption">
Figure 14: Directed modulation in Gemma-3-1b-it. (a) The hit rate under each instruction in the main setup, as plain text, with the band ending earlier, and counting only positions where no member is among the model's own 10 most likely next tokens. (b) The hit rate under each instruction against how often the model doesn't copy the sentence exactly at every position (filled) and where it doesn't predict a member (hollow). No instruction and "don't think about" overlap at 0%.
</div>

**Mental arithmetic.** Asked directly in a chat ("What is {expression}? Answer with just the number."), Gemma-3-1b-it gets 20 of the 24 sums right. The answer is rarely the top readout while copying (Figure 12, bottom), but it is in the top 5 on 21% of "think about" trials, 12% for a mention, and 2% with no instruction.

### D. Internal reasoning

GPT-2 can answer few of the released two-hop questions. With two worked examples (others of the same kind where the set has any), it answers 19 of the 90 in the privilege test and 4 of the 93 in the lens evaluations. Most of these (13 of the 21 distinct questions) go through a country, usually from a city or language to its country's capital or language. On the 12 of the 19 we can test (single-token words, target answer outside the top 10), the swap works for 4, all through a country; otherwise GPT-2 often gives a capital from the worked examples (e.g. Budapest for Greece → Sweden). Gurnee et al.'s other examples are out of reach too: in quick checks GPT-2 didn't rhyme (3 couplets) or give French antonyms (5 words), and it gets none of their arithmetic (C.1).

So we use questions of two kinds, each with two worked examples that use countries outside the test set:

```text
In the country where people speak Arabic, the capital city is called Cairo. In the country where people speak Hebrew, the capital city is called Jerusalem. In the country where people speak {language}, the capital city is called
```

```text
The country governed from Cairo has one main language, namely Arabic. The country governed from Tokyo has one main language, namely Japanese. The country governed from {capital} has one main language, namely
```

GPT-2 answers 48 of our 53 questions. For 41 of the 48, the country is outside the output top 10, so it is not about to be said. For 36 of those 41, it is in the lens top 10 at the answer position at some band layer. Over all 48, its median J-lens rank there is 13 at layer 8 and 4 at layer 9, against 645 and 424 for a random other country.

The swap is the coordinate swap, as in Gurnee et al.'s internal reasoning experiments. They swap each question's intermediate for one random alternative; we swap each of the 48 questions with every other country whose question GPT-2 answers, keeping targets whose answer starts outside the top 10, for 984 swaps in all. The target answer's median rank goes from 90 to 1.

Depth (Figure 15): as in Gurnee et al., we swap either the country or the answer itself over windows of layers from the start of the band, on the 37 questions where swapping over the whole band works (one random target each). Unlike in Claude, swapping the answer already works at the band's first layer: at layer 7 alone it makes the expected answer the top output on 19 of the 37, against 23 for the country. Starting the windows earlier (at layer 5, Gurnee et al.'s band as a share of depth, or at layer 0) gives the same picture.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig15.png" class="img-fluid" %}
<div class="caption">
Figure 15: Swapping the intermediate or the answer over windows of layers that start at the band's first layer and end progressively later, in Claude Sonnet 4.5 (Gurnee et al.'s released data for their Figure 15, by percent of depth) and GPT-2 small (by layer). The score is the expected answer's log-probability minus the original answer's after the swap, over the trials where swapping the intermediate over the whole band works (shaded: standard errors). With no swap, GPT-2's score is −7.6. Gray: each model's band; GPT-2's last window, ending at layer 10, goes past it.
</div>

For the privilege test, each country's probe is the residual stream at the last token of six prompts, averaged, minus the same average over all 28 countries. This follows Gurnee et al.'s description, but their prompts aren't released. Each of ours implies the country without naming it and asks about something else. GPT-2 can't answer most of these questions, though, and often predicts the country itself next: it is among the top 5 next tokens in a median of 4 of the 6 prompts per country, and in the last prompt the top prediction is the capital for 12 of the 28. Gurnee et al. don't say whether the intermediate can be the next token in their prompts. The six prompts:

```text
People in the country whose capital is {capital} mostly speak the language of
A traveler flying into {capital} has landed on the continent of
The homeland of the {language} language lies on the continent of
Newspapers printed in {capital} are usually written in the language of
The country whose largest city is {capital} is located on the continent of
Someone who grew up speaking {language} was most likely born in the city of
```

Swapping the whole probe flips the answer on 88% of the 984 swaps. With Gurnee et al.'s 25 J-lens vectors instead of 3, the J-space part flips it on 95% (93% with the rest clamped) and the rest on 1% (0% with the J-space clamped).

At layers 8 and 9, the 3 J-lens vectors of a probe's J-space part are mostly the country's capital or language, the country itself, or names and words associated with it: Germany's at layer 9 are German, Wein, and Berlin, and China's at layer 8 are Beijing, Heavenly, and Crane. Gurnee et al.'s one released decomposition is similar (China: China, Beijing, pandas). The capital and the language are the answers to our two kinds of question, and at layers 8 and 9 the capital is among the three for 9 and 10 of the 28 countries and the language for 6 and 9. So for roughly a third of the countries, swapping the J-space part may partly swap in the answer itself. At layer 7 the three vectors are mostly unrelated tokens (France's are acey, relentless, and Retail) and capture no more of the probe's variance than three random vectors do (A.4).

### E. Flexible generalization

Gurnee et al.'s templates and answers, each preceded by two worked examples that use arguments outside the test set. For example:

```text
Most people in Japan speak Japanese. Most people in Italy speak Italian. Most people in France speak
The month right after January is February. The month right after June is July. The month right after April
```

With worked examples GPT-2 answers 19 of the 64 prompts (4 arguments for each of 16 functions), and 9 without (Gurnee et al.'s templates as written).

The swap is Gurnee et al.'s coordinate swap, at every position of every band layer (their Fig. 68 calls this the "lens-coordinate swap experiment"). Gurnee et al.'s main text describes the stronger version as "doubling the strength with which we subtract the source lens vector and add in the target", which also fits the subtract-and-add swap of A.1, so we give both. A swap works if the target argument's answer becomes the top-1 output. Claude's results on the same swaps come from Gurnee et al.'s released [per-swap grid](https://transformer-circuits.pub/2026/workspace/data/flex-gen-systematic/appendix.json).

| swaps | n | GPT-2, coordinate swap, α = 1 / α = 2 | GPT-2, subtract-and-add, α = 1 / α = 2 | Claude, α = 1 |
|---|---|---|---|---|
| all | 192 | 28 (15%) / 23 | 33 (17%) / 30 | 76 (40%) |
| testable: GPT-2 knows the target's answer and doesn't already give it | 46 | 10 (22%) / 10 | 11 (24%) / 8 | 16 (35%) |
| stricter: GPT-2 also answers the original prompt correctly | 32 | 6 (19%) / 6 | 4 (12%) / 2 | 5 (16%) |

"Testable" also leaves out swaps whose two answers are the same, or whose target answer appears in the worked examples. Testable swaps by function, GPT-2 (coordinate swap) and Claude: language 7/9 and 8/9, capital 2/3 and 3/3, currency 0/3 and 3/3, animal class 0/3 and 2/3, first letter 1/4 and 0/4, next month 0/12 and 0/12, successor 0/12 and 0/12 (the other nine functions have no testable swaps).

"Doesn't already give it" leaves out swaps that would count without changing anything: GPT-2 says "seven" to every month-number prompt, "forty" to every squaring prompt, and "f" to three of the four first-letter prompts. GPT-2's one testable success outside the country functions (first letter, nine → five) is this same "f".

Doubling the strength doesn't raise GPT-2's top-1 rate, but it still moves its answer further toward the target's. Gurnee et al.'s measure of this (their Fig. 19) is the swap effect: the change in the target answer's log-probability minus the change in the model's spontaneous answer's, which we take to be GPT-2's own answer before the swap. Over the 192 swaps it averages 3.6 at α = 1 and 6.4 at α = 2. The top-1 rate doesn't rise because at α = 2 the coordinate swap also makes GPT-2 output the swapped-in argument itself more often: on 55 of the 185 swaps whose answers differ and aren't in the worked examples, against 30 at α = 1. On the testable swaps, doubling the coordinate swap takes GPT-2 from 9 to 8 of 15 on the country functions and from 0 to 2 of 12 on the month after. In Claude, over all 12 swaps of each function, doubling lowers the top-1 rate on the country functions, from 42 to 35 of 48, and raises it for the other three kinds of argument (the month after goes from 0 to 6 of 12) (Gurnee et al.'s Fig. 19, [data](https://transformer-circuits.pub/2026/workspace/data/flex-gen-systematic/systematic.json)). These counts are per function, not per swap, so Claude's double-strength count on our 46 testable swaps is only known to lie between 13 and 28.

Figure 16 plots each function's workspace loading (the mean cosine between the residual stream and the argument's lens vector over the band, at the argument and the last position) against its swap effect at α = 1, as Gurnee et al. do. Without the four country functions, the Spearman correlation falls from 0.64 to 0.44 in GPT-2, against 0.92 to 0.88 in Claude. Loading also doesn't predict which of GPT-2's swaps reach top-1: across the 25 prompts with testable swaps, its Spearman correlation with the fraction that work is −0.11. Per function, loading against the number of swaps that reach top-1 correlates at 0.12 in GPT-2 and 0.84 in Claude (Pearson).

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig16.png" class="img-fluid" %}
<div class="caption">
Figure 16: Each function's workspace loading against its swap effect at α = 1, in Claude Sonnet 4.5 (Gurnee et al.'s Fig. 19 data) and GPT-2 small (the coordinate swap, averaged over the function's 12 swaps). Gurnee et al.'s loading adds the cosines at the two positions and ours averages them, which doesn't change a correlation.
</div>

### F. Selectivity

- Two-hop: the 48 internal reasoning questions of Appendix D that GPT-2 answers.
- One-hop: `The capital of Egypt is Cairo. The capital of Japan is Tokyo. The capital of {country} is` (and the same with "main language"), 52 questions.
- Ordinary text: the first 96 tokens of 16 WikiText-2 test paragraphs. The score is the fraction of positions (after the first 5) whose top-1 prediction doesn't change.

Figure 17 adds one-hop recall and Gurnee et al.'s 10 directions to the main text's Figure 5. Every item scores 1.00 with no ablation.

{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig17.png" class="img-fluid" %}
<div class="caption">
Figure 17: GPT-2's scores under J-space ablation at the three strengths (1, 3, and 5 layers), removing the top direction (left) or Gurnee et al.'s 10 (right). Gray boxes: the range over five draws of random directions of the same size, with their mean as a black line. Figure 5 in the main text compares the left panel's two-hop and ordinary-text scores with Claude's.
</div>

Claude's scores in Figure 5 are from Gurnee et al.'s Figure 22 (10 directions; two-hop accuracy on their 50 questions). Relative to the band, our settings are heavier than Gurnee et al.'s: their heavy setting covers 85% of their band, and our five layers go past ours. Gurnee et al. don't say whether their random control is matched in size to the J-space ablation; ours removes the same norm at each position, as their controls for their experiential-report ablation do. In Gurnee et al.'s 14-task battery (their Figure 24), the middle setting leaves classification and extractive tasks near their clean scores and lowers TriviaQA to 0.53 of its clean score. Its multihop scores there (0.83, 0.48, and 0.21 of the clean score) differ from Figure 22's; we use Figure 22's.

**Language test.** Each task is three worked examples in Portuguese, Dutch, and Swedish, then one of Gurnee et al.'s 8 passages, then the question. The swap is the coordinate swap of the language's J-lens vector for each of the other three languages', and covers only the question tokens (`\n` and the cue), as in Gurnee et al., who swap "across the question tokens". A swap counts for a passage only if GPT-2 does that task correctly without it. We don't run Gurnee et al.'s language-switch detection task, which has no base-model form.

- Name the language: `Passage: {passage}\nLanguage:`.
- Name an author (Gurnee et al.'s released question: "Name one famous author who wrote in the same language as this passage"): `Passage: {passage}\nA famous author who wrote in this language:`, scored on the first token of Gurnee et al.'s author lists.
- Write the next sentence (Gurnee et al.'s "Continue it by writing the next sentence"): `Passage: {passage}\nNext sentence:`, graded by reading GPT-2's greedy next sentence (up to 25 tokens) and noting its language. Its sentence is in the passage's language for all 8 passages, but is mostly repetition or a copy of the passage.

GPT-2 names the language correctly for 7 of the 8 passages and a right author for none, so the author task can't be tested. At α = 2 the swap changes the named language on 13 of 21 swaps, and the next sentence still never changes language. The language's name is in the band J-lens over the question at best ranks of 1–9 when naming the language, 16–100 for the author, and 22–2,466 for the next sentence (range over passages).

**Small part of the representation, and occupancy** (150 WikiText activations per layer, mean-centered). We pursue each activation with up to 30 J-lens vectors, and separately with vectors from a random dictionary of the same size. Median occupancy over positions is 2, 2, and 4 at layers 7, 8, and 9, 3 over the band's positions pooled, between 1 and 2 at every earlier layer, and 5 at layer 10. With K equal to each layer's median occupancy, as in Gurnee et al., K J-lens vectors capture 7.9%, 7.8%, and 14.1% of the variance at layers 7, 8, and 9, against 4.4%, 4.4%, and 8.1% for K random ones. With Gurnee et al.'s K = 25, the J-lens vectors capture less than random ones (29.8%, 31.3%, and 36.2%, against about 37%).

**Line counting** (Gurnee et al.'s Figure 21). GPT-2 can't count characters: asked for a count, it answers "been", "a", or "the", and a count reaches the band lens top 25 on at most 1 of 11 passages in any condition.

### G. Structure

<div class="l-page">
{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig18.png" class="img-fluid" %}
</div>
<div class="caption">
Figure 18: Gurnee et al.’s layer statistics (their Figure 28) for Claude Sonnet 4.5 (top row, their released data) and GPT-2 small (bottom row; 48 WikiText-103 validation sequences of 128 tokens). (a) How often the model’s own next token is in the J-lens top k. (b) The excess kurtosis of the lens readout, by percentile over positions. (c) Persistence: the mean log-probability the lens gives, at position t + Δ, to its top token at position t, minus the same for the top token of a random position in the same sequence. (d) The share of dimensions needed to hold a given share of the variance of the whole vocabulary’s centered J-lens vectors (for GPT-2, with the final LayerNorm’s gain folded in, A.1). Shaded: Claude’s band (38–92% of depth) and ours (layers 7–9).
</div>

**Persistence.** Between 24% and 38% of the lens's top tokens at layers 0 to 10 appear somewhere in the input, so persistence could partly be the lens echoing the input. Counting only top tokens that never appear in it, for the random positions too, barely changes it: at layer 9 it moves by at most 0.3 nats at any Δ. At Δ = 32 it is slightly below zero at every layer, probably because the random position is sometimes a nearby one.

**LayerNorm gain.** The statistics computed on the J-lens vectors themselves, the CKA between layers (Figure 6) and effective dimensionality (Figure 18d), use the vectors with the final LayerNorm’s gain folded in (A.1). Without the gain, effective dimensionality is nearly the same (at the 90% level, from 0.13 at layer 0 to 0.73 at layer 11, against 0.15 to 0.74 with it). But one principal component then carries 26–37% of each layer's dictionary's variance even after centering (6–9% with the gain), which pushes linear CKA to 0.94 or higher for every pair of layers 0 to 10. With the top 5 components removed, or with the mean canonical correlation over the top 50 dimensions, similarity again falls off smoothly with distance.

**Ignition** (Figure 7) uses 16 pairs drawn at random from Gurnee et al.'s 12 countries (they don't say which 16 they used), their 40 sentences, and 21 mixing weights per pair. We look at the change in the mixing weight required to move the representation from 10% to 90% of the way to the other country. This is about 0 for a snap and 0.8 for a linear blend. In GPT-2 the median falls slowly with depth, from 0.78 at layer 0 to 0.70 at layer 11 for the whole residual stream (0.64 for its part along the two countries' J-lens vectors), so it stays close to a blend. At each pair's most ambiguous mixture, every sentence's representation sits between 25% and 75% of the way at every layer, so no single sentence snaps either.

<div class="l-page">
{% include figure.liquid path="assets/img/2027-04-30-gpt2-global-workspace/fig19.png" class="img-fluid" %}
</div>
<div class="caption">
Figure 19: Amplification (a, b) and capacity (c) in GPT-2 small. In (b), the J-lens vectors and each comparison pool are 3,072 directions each; in (c), each list has 80 words. Shaded: our band (layers 7–9).
</div>

**Amplification.** The raw J-lens vectors have higher MLP gain than the centered ones at every layer but the last (Figure 19a), because they all contain the shared direction, which the MLPs amplify. For the neurons (Figure 19b), Gurnee et al. report a jump from about 15% before Claude's band to 60–65% within it; in GPT-2 the share rises steadily with depth.

**Lists** (Figure 19c). In single-category lists (first names, surnames, countries, cities), about 10 to 13 of the list's 80 words are present at each comma from the second on, most of them not yet read at first, against nearly the whole 80-word category in Claude. So GPT-2's J-space seems to hold about a dozen members of the category rather than a record of the list. When a list switches category (8 lists of four 20-word blocks from Gurnee et al.'s name, surname, country, and city pools, in shuffled order), the old category drops out: averaged over commas 6 to 19 of a block, 21% of that block's words are present during the block and 0.8% during the next.

