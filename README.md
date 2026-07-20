<div align="center">

# SeismicAgent: Multi-Scale Structural Damage Assessment

### Multi-Agent Large Language Model Framework for Multi-Scale Post-Earthquake Structural Damage Assessment

**Ye Yuan<sup>1,2</sup> · Yuzhou Qie<sup>2</sup> · Long Chen<sup>1,*</sup> · Qiuchen Lu<sup>3</sup> · Nan Li<sup>4</sup> · Dong Yang<sup>5</sup> · Mou Ben<sup>6</sup>**

<sup>1</sup> City University of Hong Kong · <sup>2</sup> The Hong Kong Polytechnic University · <sup>3</sup> University College London  
<sup>4</sup> Tsinghua University · <sup>5</sup> Guangzhou University · <sup>6</sup> Central South University

[![Project](https://img.shields.io/badge/Project-SeismicAgent-181717?logo=github)](https://github.com/FredyHKU/SeismicAgent)
[![Dataset](https://img.shields.io/badge/Dataset-%CE%A6--Net-2C7FB8)](https://apps.peer.berkeley.edu/phi-net/)
[![Backbone](https://img.shields.io/badge/Vision%20Backbone-ViT--B%2F16-6A5ACD)](#specialist-visual-perception)
[![Code](https://img.shields.io/badge/Source%20Code-Coming%20Soon-F0AD4E)](#source-code)

**Accurate perception · Scene-adaptive orchestration · Evidence-grounded reporting**

> **This repository accompanies a paper under review at the ASCE Journal of Computing in Civil Engineering. The source code is being prepared and will be uploaded here.**

</div>

<p align="center">
  <img src="assets/overall_architecture.svg" width="100%" alt="Overall architecture of SeismicAgent">
</p>

<p align="center"><em>Overall architecture of SeismicAgent. An LLM moderator coordinates the scene-level agent and specialist ViT agents, while single-image and multi-image findings are synthesized into image-supported assessment reports.</em></p>

## Overview

Post-earthquake structural assessment requires reasoning across multiple visual scales. Wide-view images reveal the overall building condition and collapse status, object-level images support component identification and severity assessment, and close-up images expose local damage patterns such as cracking and spalling. Existing single-task classifiers can provide strong recognition performance but do not form a complete assessment workflow. End-to-end multimodal approaches such as **SDA-Chat** provide flexible language output, but asking one model to perform both visual perception and language generation can weaken fine-grained damage recognition.

**SeismicAgent** addresses this trade-off by separating **visual perception**, **task orchestration**, and **report synthesis**. Seven specialist Vision Transformer (ViT) agents provide task-specific predictions, an LLM moderator selects scale-appropriate agents, and a separate multimodal summary agent transforms image-linked evidence into structured engineering reports. This separation improves task-level recognition over SDA-Chat on the shared Φ-Net assessment tasks while retaining flexible, evidence-grounded report generation.

## Main Contributions

1. **Perception-reasoning separation**  
   Seven specialist ViT classifiers act as visual experts, while the LLM focuses on task orchestration and evidence synthesis rather than replacing task-specific perception. On the six tasks shared with SDA-Chat, this design improves average accuracy by **6.84 percentage points**.

2. **Multi-scale and multi-image assessment**  
   The moderator first identifies the spatial scale of each image and then invokes only the relevant assessment agents. Multiple images are analyzed independently before their evidence is aggregated into a site-level report.

3. **Traceable report generation**  
   Report statements are grounded in the original image, the corresponding specialist-agent output, and confidence information, allowing conclusions to be traced back to their supporting evidence.

## Method

### Scene-Adaptive Orchestration and Report Synthesis

<p align="center">
  <img src="assets/scene_adaptive_workflow.svg" width="100%" alt="Scene-adaptive workflow for single-image and multi-image assessment">
</p>

<p align="center"><em>Scene-adaptive workflow for single-image and multi-image assessment.</em></p>

For each input image, the moderator first calls the **scene-level agent** to determine whether the image is structural-, object-, or pixel-level. The predicted scale then determines which specialist agents are appropriate:

<div align="center">
<table>
  <thead>
    <tr>
      <th align="center">Spatial scale</th>
      <th align="center">Selected assessment tasks</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center"><strong>Structural level</strong></td>
      <td align="center">Damage state, Collapse mode</td>
    </tr>
    <tr>
      <td align="center"><strong>Object level</strong></td>
      <td align="center">Damage state, Spalling, Component type, Damage level</td>
    </tr>
    <tr>
      <td align="center"><strong>Pixel level</strong></td>
      <td align="center">Damage state, Spalling, Damage level, Damage type</td>
    </tr>
  </tbody>
</table>
</div>

The resulting labels and confidence scores are stored as **structured evidence records**. For multi-image inputs, each image is processed independently through the single-image pipeline. The image-level reports and evidence records are then integrated by a summary agent into one site-level assessment, preserving the source image associated with each finding.

### Specialist Visual Perception

<p align="center">
  <img src="assets/specialist_vit_architecture.svg" width="100%" alt="Architecture and inference pipeline of specialist ViT classification agents">
</p>

<p align="center"><em>Architecture and inference pipeline of the specialist ViT classification agents.</em></p>

The specialist perception layer contains seven task-specific agents for:

- scene level;
- damage state;
- spalling condition;
- collapse mode;
- component type;
- damage level; and
- damage type.

All agents adopt an ImageNet-pretrained **ViT-B/16** backbone and return both a predicted label and a confidence score. Nominal tasks are trained with cross-entropy loss, while **damage level** and **collapse mode** use conditional ordinal regression (CORN) to preserve the ordering of severity categories.

## Representative Assessment Outputs

<p align="center">
  <img src="assets/representative_outputs.svg" width="100%" alt="Representative single-image and multi-image assessment outputs">
</p>

<p align="center"><em>Representative dialogue-based assessment outputs for single-image and multi-image inputs.</em></p>

The generated reports combine visible image evidence with structured specialist-agent predictions. Single-image reports describe the condition shown in one photograph, while multi-image reports integrate complementary observations from different viewpoints and spatial scales into an overall site-level assessment. The report generator is instructed to use structural-engineering terminology, avoid unsupported extrapolation, and qualify uncertain findings.

## Key Results

### Comparison with SDA-Chat

At the perception layer, SeismicAgent is compared with **SDA-Chat**, which fine-tunes a single multimodal VQA model to perform both visual recognition and language generation. SeismicAgent instead assigns visual recognition to specialist ViT agents and reserves the LLM for orchestration and report synthesis.

<div align="center">
<table>
  <thead>
    <tr>
      <th align="center">Shared assessment task</th>
      <th align="center">SeismicAgent (%)</th>
      <th align="center">SDA-Chat (%)</th>
      <th align="center">Difference (pp)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center">Damage state</td>
      <td align="center"><strong>86.60</strong></td>
      <td align="center">81.24</td>
      <td align="center"><strong>+5.36</strong></td>
    </tr>
    <tr>
      <td align="center">Spalling condition</td>
      <td align="center"><strong>84.70</strong></td>
      <td align="center">78.36</td>
      <td align="center"><strong>+6.34</strong></td>
    </tr>
    <tr>
      <td align="center">Collapse mode</td>
      <td align="center">79.00</td>
      <td align="center"><strong>82.32</strong></td>
      <td align="center">−3.32</td>
    </tr>
    <tr>
      <td align="center">Component type</td>
      <td align="center"><strong>88.30</strong></td>
      <td align="center">86.01</td>
      <td align="center"><strong>+2.29</strong></td>
    </tr>
    <tr>
      <td align="center">Damage level</td>
      <td align="center">71.90</td>
      <td align="center"><strong>73.86</strong></td>
      <td align="center">−1.96</td>
    </tr>
    <tr>
      <td align="center">Damage type</td>
      <td align="center"><strong>66.90</strong></td>
      <td align="center">34.58</td>
      <td align="center"><strong>+32.32</strong></td>
    </tr>
    <tr>
      <td align="center"><strong>Average on six common tasks</strong></td>
      <td align="center"><strong>79.57</strong></td>
      <td align="center">72.73</td>
      <td align="center"><strong>+6.84</strong></td>
    </tr>
  </tbody>
</table>
</div>

> **SeismicAgent improves the average accuracy on the six common tasks from 72.73% to 79.57%. The largest gain occurs in damage-type recognition, where accuracy increases from 34.58% to 66.90%.**

The comparison should be interpreted with the dataset protocols in mind: SDA-Chat uses a more rigorously filtered and corrected Φ-Net-derived VQA dataset, whereas this study applies only the minimum necessary cleaning and retains more of the original benchmark heterogeneity. Even under this setting, the specialist-agent design shows a clear advantage for fine-grained visual recognition.

### Report-Level Comparison with Direct MLLMs

<div align="center">
<table>
  <thead>
    <tr>
      <th align="center">Evaluation setting</th>
      <th align="center">Direct MLLM accuracy</th>
      <th align="center">SeismicAgent accuracy</th>
      <th align="center">Hallucination reduction</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center">Single image · GPT-5.5</td>
      <td align="center">82.85%</td>
      <td align="center"><strong>92.87%</strong></td>
      <td align="center">9.52% → <strong>1.40%</strong></td>
    </tr>
    <tr>
      <td align="center">Single image · Claude Sonnet 4.6</td>
      <td align="center">85.75%</td>
      <td align="center"><strong>91.43%</strong></td>
      <td align="center">8.71% → <strong>0.82%</strong></td>
    </tr>
    <tr>
      <td align="center">Multi image · GPT-5.5</td>
      <td align="center">88.40%</td>
      <td align="center"><strong>95.10%</strong></td>
      <td align="center">9.30% → <strong>2.33%</strong></td>
    </tr>
    <tr>
      <td align="center">Multi image · Claude Sonnet 4.6</td>
      <td align="center">88.47%</td>
      <td align="center"><strong>94.33%</strong></td>
      <td align="center">10.40% → <strong>4.73%</strong></td>
    </tr>
  </tbody>
</table>
</div>

Across both reporting settings, SeismicAgent consistently outperforms the direct MLLM baselines, improving report accuracy by **5.68–10.02 percentage points** for single-image cases and **5.86–6.70 percentage points** for multi-image cases. Hallucination rates decrease in every comparison, indicating that grounding report generation in image-linked specialist evidence helps suppress unsupported statements.

<p align="center">
  <img src="assets/report_level_comparison.svg" width="100%" alt="Report-level comparison between the proposed framework and direct MLLM baseline">
</p>

<p align="center"><em>Representative report-level comparison between SeismicAgent and a direct MLLM baseline. Image-linked specialist evidence improves assessment accuracy and reduces unsupported statements.</em></p>

Across four tested language-model backbones, single-image report accuracy remains between **88.83% and 92.87%**, while multi-image report accuracy remains between **92.37% and 95.10%**. These results indicate that the benefits of scene-adaptive orchestration and evidence grounding are not limited to one LLM/MLLM backbone.

## Source Code

> [!NOTE]
> **The source code is being prepared and will be uploaded to this repository.**

This repository serves as the project page for the proposed framework, methodology, representative outputs, and key experimental findings. For requests during the review period, please contact the corresponding author.

## Dataset

The study is evaluated on the **PEER Hub ImageNet (Φ-Net)** benchmark. The dataset is available from the [PEER Φ-Net website](https://apps.peer.berkeley.edu/phi-net/).
