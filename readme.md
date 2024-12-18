Forecasting the progress of research is an elusive and important goal.
Here, we take a toy step towards this goal by exploring generating new scientific paper titles given past titles on arXiv:
1. We generate titles conditioned on a specific author (using GPT-3 without finetuning)
2. We generate titles conditioned on their publication year (using GPT-Neo with finetuning)
3. We evaluate the generated titles to see how well they match new, recent paper titles

## 1  &nbsp; Author-specific paper titles (prompting gpt3)
To generate author-specific titles, we take the five most recent titles from each author with atleast 3 arXiv AI papers (cs.ML, cs.LG, stat.ML).
We then format the papers using the following template and query for a new title using GPT-3:

```
Here is a list of related machine-learning papers:

> [title 1]
> [title 2]
...
> [title 5]
> ____
```



Here's a concrete example -- when prompting with these 5 recent titles:

```
> Hierarchical Shrinkage: improving the accuracy and interpretability of tree-based methods
> Fast Interpretable Greedy-Tree Sums (FIGS)
> Adaptive wavelet distillation from neural networks through interpretations
> Emb-GAM: an Interpretable and Efficient Predictor using Pre-trained Language Models
> Explaining Patterns in Data with Language Models via Interpretable Autoprompting
> ____
```

We get these 5 (independent) random generations for the blank:

```
1. Towards Interpretable Natural Language Processing: A Survey
2. A Unified Framework for Interpretable Machine Learning
3. Compositional Attention Networks for Machine Reasoning
4. Achieving Open Vocabulary Neural Machine Translation
5. A Deep Understanding of Neural Networks through Deep Visualization
```

The results are often interesting but fall into failure modes where they generate irrelevant titles for an author, often leaning towards popular topics such as deep learning, multi-task learning, and reinforcement learning.
Note: the model used was GPT-3 `text-davinci-002` from the OpenAI API on Oct 14 2022. It likely was not up to date with the most current advances and could be improved with finetuning on more recent titles.
During preprocessing, paper titles with irregular formatting were removed and distinct authors with exactly the same name were not differentiated.

## 2  &nbsp; Finetuned paper title generation (gptneo)

To improve the model's ability to generate cogent titles, we finetune it on a large corpuse of titles. We start from the [gpt-neo-2.7B checkpoint](https://huggingface.co/EleutherAI/gpt-neo-2.7B). We finetune on all [paper titles on arXiv](https://www.kaggle.com/datasets/Cornell-University/arxiv) in the categories cs.AI, cs.LG, stat.ML up to Oct 13, 2022. We exclude all papers after Apr 1, 2022 (to test the ability to forecast new papers) and an additional random 5\% of titles.
We also exclude titles with a length of less than 6 words or greater than 20 words. This results in 98,388 papers for finetuning:

![](https://csinva.io/gpt-paper-title-generator/figs/paper_metadata.svg)


**Samples**
After finetuning, here are some example titles generated by the model, conditioned on different years (see a large dump in [this folder](https://github.com/Palaksha1510/generating-research-paper-title/tree/master/samples/gptneo)):

**2022**

- Diverse Datasets for Learning to Rank via Knowledge Graph Embedding
- Machine learning-driven method for high-throughput single-cell analysis of differentiation and lineage commitment
- On the Sample Complexity of Differentially Private Learning
- Data-Dependent Weight Normalization for Improved Image Resolution
- Adaptive Densely Connected Networks for the Generation and Visualization of Object Deformations
- Exploring the Implicit Bias in Transfer Learning using Imitation Learning

**2023** (These samples tend to just be similar to 2021/2022 where the majority of the training data lies)

- An Interpretable Dynamic Network for Spatiotemporal Pattern Prediction in High-Dimensional Time Series Data
- Multimodal Deep Learning for Automated Cancer Histopathology Analysis
- Reinforced Learning for Robust and Accurate Object Detection
- A Machine Learning Approach to High Sensitivity Data Processing
- Adversarial Robustness for Graph Neural Networks in Network Intrusion Detection
- Reinforcement Learning via Exploration and Rehearsal for Learning from Demonstrations

**2010** (Seems to properly generate older titles)

- Learning in a Dynamic, Clustered and Homogeneous Heterogeneous Markov Decision Process
- An Empirical Analysis of the Regularization of the Gaussian Process Regression
- A Scalable Clustering Algorithm under Heterogeneous Data
- A Unified Representation for Probabilistic Time Series Forecasting
- A Hybrid Approach to Automatic Alignment and Localization of Digital Object Platforms
- Bayesian nonparametric modeling of random fields


During finetuning each paper title was given in the format `<year>\n\n <title>\n` (e.g. `2020\n\n Interpretations are useful: penalizing explanations to align neural networks with prior knowledge\n`). The same format should be used for inference. These samples are considerably better than the samples we made with GPT2.

## 3  &nbsp; Generated paper evaluation

We now evaluate whether the generated titles for 2022 match the real paper titles from the test set (April 1 - Oct 13 2022). Note that the model has never seen any papers from this time period and it's pre-training corpus also only contained text from before 2022. We generate 5,000 titles and find for the closest match for each of them in the test set (which contains ~15,000 titles). The resulting BLEU scores are shown in this figure:

![](https://csinva.io/gpt-paper-title-generator/figs/bleu.svg)

Here's a table of the first 5 matches. See if you can guess which are the real titles and which are generated (answers below):

  | A                                                            | B                                                            |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Understanding the effect of data augmentation in generative adversarial networks | Understanding the effect of data augmentation in self-supervised anomaly detection |
| Adversarial attacks on graph neural networks                 | Sparse vicious attacks on graph neural networks              |
| Differentiable reinforcement learning for continuous control | Normality-guided distributional reinforcement learning for continuous control |
| Multilevel representation learning for time series forecasting | Out-of-distribution representation learning for time series classification |
| Unsupervised feature learning for medical image segmentation | Distributed contrastive learning for medical image segmentation |

ㅤ

**Answers** 
The generated titles often seem to be overly general, missing the detailed specificity of the real titles (e.g. "Sparse vicious attacks" rather than "Adversarial attacks").

**Some possible followups**
This post was very limited, but there are a bunch of directions to explore to see how well language models can really forecast scientific titles (and scientific progress in general). Here are some straightforward followups:

- Use information about abstracts instead of just titles
- Get the language model to explain why it generated a particular title (probably grounded in abstract)
- Build a model to classify year given paper title and then use [iPrompt](https://arxiv.org/abs/2210.01848) to describe the year-to-year differences
- Improve author-specific title generation with finetuning (some authors have *a lot* of papers)

![](https://csinva.io/gpt-paper-title-generator/figs/author_counts.svg)

## Reference

- Code [here](https://github.com/csinva/gpt-paper-title-generator)
- arXiv dataset from [here](https://www.kaggle.com/datasets/Cornell-University/arxiv)
