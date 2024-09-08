---
layout: distill
title: Sequence classification expressivity
description: Comparing expressivity to learnability
img: # <!-- TODO tikz something! -->
importance: 2
category: Cambridge
bibliography: expressivity.bib
github: //www.github.com/Hjel2/dnn_miniproject
---

This is a writeup of my miniproject for the [DNN](https://www.cl.cam.ac.uk/teaching/2324/DNN/) module.
> It's great work, and you clearly went beyond the brief.
> -Ferenc Huszár

This project investigates the expressive power of sequence classification models by comparing them to formal grammars. Specifically, I lower-bounded the complexity of languages which different sequence classification architectures can theoretically recognise on the [Chomsky Hierarchy](https://en.wikipedia.org/wiki/Chomsky_hierarchy), and compared this to the languages which they empirically learnt to recognise.

<!--
## Work completed

This project consisted of the following work:
1. designed information-retrieval datasets corresponding to regular, context-free and context-sensitive languages
2. trained RNNs, GRNNs and transformers to recognise these languages
3. used [RASP](https://github.com/yashbonde/rasp) and weight hardcoding to semi-formally lower bound the expressivity of languages learnable by sequence classification models
4. investigated whether efficient transformer architectures limited the expressivity of transformers
5. evaluated the robustness of learned representations through adversarial attacks
-->

### Findings

Through the experiments in this project, I was able to discover:

1. Learnability is a subset of expressivity
2. Models *can* perfectly represent many languages
3. Models *do not* perfectly learn even simple languages
4. Recurrence as an inductive bias helps models approximate finite automata
5. Efficient transformers are less expressive than transformers

---

## Methodology

The primary contribution of this project is to demonstrate a separation of the grammars representable by models, and those which they learn. To achieve this, I train sequence to sequence models on datasets belonging to different formal grammar classes. I hardcode weights or use [RASPy](https://github.com/srush/raspy)<d-cite key='weiss2021thinking'> to theoretically demonstrate which languages can be represented.

During this project, I investigated the expressivity of RNN, GRNN and transformer architectures. Latter experiments also tested some efficient transformer architectures: linformer<d-cite key='wang2020linformer'></d-cite> and sparse transformer<d-cite key='child2019generating'>.

### Designing information retrieval datasets

The project must first empirically provide lower bounds on the expressivity of languages recognisable by sequence to sequence models. To this end, I constructed three synthetic information retrieval datasets. These datasets were dictionary-lookup tasks with keys in different grammar classes. Each dataset consisted of sequences of random symbols with a key immediately followed by a value.

- the regular dataset key was $$🔑$$
- the context-free dataset keys were of the form $$🗝️^n🔑^n$$
- the context-sensitive dataset keys were of the form $$🗝️^n🔑^n🗝️^n$$

I constructed these datasets by generating random sequences, inserting keys and then running a vectorised state machine to recognise and then remove any spurious keys. Models output whether the key had occurred, and if so what the value was. This 'essentially' turned sequence to sequence models into sequence classification models.

### How can we prove how expressive models are?

Let the dataset $$d_g$$ be a key-value retrieval dataset with keys drawn from grammar class $$g$$. If a model $$m$$ trained on the dataset $$d_g$$ is able to perform well, then the languages learnable by $$m$$ is not a subset of the grammar class $$g$$.

Based on this observation, I train RNNs, GRNNs and transformers on the datasets I have constructed. Through this, I am able to lower-bound the complexity of grammar classes which are representable by different models.

### RNNs struggle to recognise some regular languages

Recurrent neural networks (RNNs) are the oldest sequence to sequence models. They have an implicit bias towards recurrence: RNNs process data sequentially in order. This means RNNs mimic the operation of a finite automaton, and as such should theoretically be able to learn simpler grammars classes.

Unfortunately, my experiments demonstrated that recurrent neural networks do not consistently learn even simple regular languages. As seen in 

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/rnn_reglang.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Visualisation of the performance of an RNN when trained on the regular key-value retrieval dataset.
</div>

Critically, I observed that the RNN had great difficultly with the regular language dataset. Demonstrated by it typically converging to undesirable solutions: often getting stuck at 60-70% accuracy. During testing, it did converge to 95%+ accuracy sometimes, which demonstrated that it can learn to approximate the computation of a DFA.

Given that this is a comparatively simple sanity-check, it is surprising that the RNN is struggling. RNNs obsolescence in modern literature represents these limitations.

To determine whether the inability of RNNs to learn the regular language was due to a fundamental inability to represent the grammar, I set out to hardcode the weights for an RNN which recognised the language. After many hours of maths, I was able to explicitly construct RNN weights which caused the model to perfectly mimic the behaviour of a DFA, achieving 100\% accuracy. To demonstrate that all regular languages are representable by a DFA, I then hardcoded weighs for the context free dataset. Through this, I proved that RNNs are able to represent regular languages. 

The above theoretical results imply that the inability of RNNs to learn the simple regular language is due to the limitations in trainability rather than expressivity.

### GRNNs recognise many languages

My analysis of RNNS demonstrated that their inability to perform was largely due to training limitations rather than expressivity problems. Gated RNNs (GRNNs) improve upon the original RNN architecture and make them more trainable. To provide theoretical justification to this, I investigated whether GRNNs and RNNs are separable by formal grammars.

Empirically, I found that GRNNs were able to recognise regular and context free grammars to a high level. Furthermore, they took steps towards recognising context-sensitive grammars. However, they were unable to fully learn context-sensitive grammars.

Human grammar requires functionally all features of context-free grammars -- and some context-sensitive features. Since I have demonstrated that GRNNs struggle to learn context-sensitive features, it is unlikely that GRNNs would be able to fully represent human language. This means that they are unlikely to be able to form the foundation for the most powerful models.

### Transformers

Transformers emerged in 2019 and caused rapid improvements in SOTA across every major subdiscipline of ML. They are fundamentally different to the previous models and are based on the attention mechanism. Each token is able to directly attend to all previous tokens, bypassing the 'exponential forgetting problem' and removing the recursive inductive bias.

I trained the transformer all three datasets: regular, contxet free and context sensitive. The transformer was able to learn the regular and context free datasets. Furthermore, it was able to approximate context-sensitive languages. But they struggled to generalise to longer sequence lengths.

I validated that transformers are able to represent regular and context free grammars by using [RASPy](https://github.com/srush/raspy)<d-cite key='weiss2021thinking'> to implement programs which achieve perfect performance on the regular and context free datasets.

### Do models *recognise* context sensitivity or approximate them?

The lengths of the keys for the context sensitive dataset were drawn from an exponential with parameter $$\mu$$. By adjusting the value of $$\mu$$, I can adjust the distribution from which keys are drawn. This allows me to test whether the model is truly learning the grammar, or merely statistical relations. If the model has learnt the grammar, then adjusting the value of $$\mu$$ will have negligible effect on model behaviour. If, however, the model is merely approximating the grammar, then adjusting $$\mu$$ will move it into OOD and cause significant performance degradation.

<div class="col mt-2">
    <div class="row-sm mt-2 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/grnn_contextsenslang.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="row-sm mt-2 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/grnn_contextsenslang_mu.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Demonstration in the variation in performance for a GRNN as the distribution from which the key to the dataset is drawn. The model was trained with parameter $\mu=2$. Notice the significant performance degredation as $\mu$ varies. This demonstrates that GRNN is not learning the grammar, but only a weak approximation to it.
</div>
<div class="col mt-2">
    <div class="row-sm mt-2 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/transformer_contextsenslang.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="row-sm mt-2 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/transformer_contextsenslang_mu.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Demonstration in the variation in performance for a Transformer as the distribution from which the key to the dataset is drawn. The model was trained with parameter $\mu=2$. The transformer did not have significant performance degradation as the distribution from which the keys were drawn varied. This supports the hypothesis that transformers are more expressive than GRNNs, and that they are able to represent natural language.
</div>

From the above figures, we notice that transformers retained their performance when the distribution from which keys were drawn changed. This implies that they were able to better represent the context sensitive grammar.

### Efficient transformers are fundamentally less expressive than transformers

The above experiments support the hypothesis that transformers are fundamentally more expressive than other sequence to sequence models. However, transformers are also more computationally expensive. This means that much research has investigated efficient transformers. It is summarised below, in a venn diagram constructed by Efficient Transformers: A Survey<d-cite key="tay2022efficient">.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/Venn_Transformers.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Venn diagram demonstrating the types of efficient transformer.
</div>

This diagram shows that there are a wide range of transformers, whose methods of increasing efficiency can be split into a few categories. I continued my investigation by selecting and reimplementing two efficient transformer architectures. I chose Linformer and Sparse Transformer since they are different types of efficient transformer.

#### Linformer limits expressivity

Linformers are efficient transformers. They operate on constant-length datasets and have no causal mask. This means inference time is $$\mathcal{O}(n)$$ for sequence length $$n$$. As such, it is much more time efficient than the standard transformer architecture.

I replicated a Linformer and tested it on the datasets. It was unable to learn the language and constantly overfit. After significant experimentation, and repeating my experiments with the official implementation, I concluded that Linformer struggled to recognise even regular languages. Instead, Linformer consistently overfit to the training data and refused to approximate a DFA, even when provided extensive amounts of training data. I was unable to construct a situation where the Linformer approximated a DFA and did not overfit.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/linformer_reglang.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Diagram demonstrating the performance and generalisation of Linformer on a the regular langauge dataset.
</div>

Linformer does not have a causal mask. I hypothesised that the lack of a causal mask meant that Linformer struggled to approximate the behaviour of a finite automaton. To test this hypothesis, I removed the causal mask from a transformer and tested its performance. I found that it caused a significant performance drop, however not to the same extent as the Linformer. From this, I was able to conclude that there is a significant gap between the langauges learnable by a Linformer and by a Transformer.

#### Sparse Transformers

Sparse Transformers have time complexity of $$\mathcal{O}(n\cdot \lg n)$$ for sequence length $$n$$, which they achieve by limiting the flow of information.

I repeated the experiments above with Sparse Transformers. I found that due to the limited information flow, they required much deeper networks to propagate information in the same way as earlier. Transformers were able to achieve near-perfect accuracy on the regular language dataset with 3 layers. Sparse Transformers had such throttled information flow such that they would require 4 layers to propagate information to the next position. I was therefore only able to elicit performance out of 8-layer Sparse Transformers. This explicitly demonstrates how much less expressive Sparse Transformers (of fixed size) are than Transformers. Notably, I did not find Sparse Transformers to be fundamentally less expressive: they may be better suited for other tasks.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/sparsetransformer_reglang.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Diagram demonstrating the performance and generalisation of a very deep Sparse Transformer on a the regular langauge dataset.
</div>

## Sequence classification models learn robust approximations

I concluded by investigating the robustness of the approximations learnt on the regular language dataset. The theory is that if models learnt the actual function or a robust approximation, then adversarial attacks would be difficult to generate. However, if they learnt poor approximations, then adversarial attacks would be easy!

I found that black-box greedy attacks are easy. However, generating examples efficiently using white-box gradient-based methods proved ineffective. During my explorations, I implemented a greedy attack, Projected FGSM<d-cite key="papernot2016crafting">, Saliency FGSM<d-cite key="yang2020greedy">, an adaptation of Saliency FGSM using cosine similarity, and a greedy search for universal adversarial suffices.

I found that attacks which exploited gradient-information were unlikely to succeed. I attribute this to the discrete nature of the optimisation problem. Greedy attacks were able to succeed on both GRNNs and Transformers for longer context lengths. This implies that the models were able to learn the simple dataset and learnt robust approximations.

<div class="col mt-2">
    <div class="row-sm mt-2 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/grnn_reglang_adv.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="row-sm mt-2 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/transformer_reglang_adv.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Visualisation of the performance of greedy attacks on the regular langauge dataset on GRNNs and transformers.
</div>

## Conclusion

Overall, this project served to make the separation of theoretical expressivity and the learnable expressivity explicit. Additionally, it demonstrated how inductive biases help models to approximate finite automaton, which is demonstrated by GRNNs performing similarly to transformers in this situation, despite being outclassed by them in larger tasks.
