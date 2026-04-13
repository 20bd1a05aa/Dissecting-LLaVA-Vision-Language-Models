Assignment-4 Dissecting LLaVA Vision-Language Models

Name: Tarani Chilamkoti UCID: tc533 Course: DS681

Overview

In this report, I explain how LLaVA integrates vision and language using a simple but effective architecture. The focus is on how images are converted into token embeddings, how the model is trained, and what trade-offs arise from this design.

Part 1 — Architecture Understanding

Task 1.1: Forward Pass Explanation

Data Flow Diagram

Image
  ↓
CLIP Vision Encoder
  ↓
Visual Features
  ↓
Projection Layer
  ↓
Image Tokens (LLM space)
  ↓
[Image Tokens + Text Prompt]
  ↓
Language Model (LLM)
  ↓
Generated Text
Step-by-Step Explanation

1. Image Input

The model takes a raw image and preprocesses it (resize, normalization).

2. Vision Encoder (CLIP)

𝑓vision(𝑥) 

CLIP encodes the image into a feature vector capturing semantic information such as objects and actions.

CLIP is trained using contrastive learning on image-text pairs, which aligns visual and textual representations into a shared semantic space.

3. Projection Layer

𝑧=𝑊⋅𝑓vision(𝑥) 

This layer maps CLIP features into the LLM embedding space.

Typically a linear layer or small MLP
Converts visual features into token-compatible embeddings
This works because CLIP already aligns images with text at a coarse level, so the projection only needs to perform a lightweight transformation.

4. Image Tokens

The projected output is converted into a sequence:

[IMG_1, IMG_2, ..., IMG_k]
These act as pseudo-token embeddings.

5. Language Model Input

Example:

[IMG TOKENS] + "What is happening in the image?"
Image tokens are prepended to the text prompt.

6. Language Model (LLM)

A standard transformer (e.g., LLaMA):

Processes both image and text tokens
Uses self-attention across the entire sequence
7. Text Generation

𝑃(𝑦𝑡∣𝑥image,𝑦<𝑡) 

The model generates text autoregressively conditioned on both image and previous tokens.

Example

Image: A person riding a bike

Input:

[IMG TOKENS] + "What is the person doing?"
Output:

"The person is riding a bicycle."
Key Insight

Images are converted into token embeddings so the language model can process them using its native sequence modeling mechanism.

Task 1.2: Projection Layer Intuition

Why a Simple Projection Works

CLIP embeddings already encode semantic meaning
LLM embeddings represent linguistic meaning
Both are structured high-dimensional spaces
A linear transformation can align them by preserving relative structure.

Assumption

There exists an approximate linear relationship between:

Vision embedding space
Language embedding space
So semantically similar concepts remain close after mapping.

What Could Go Wrong?

If alignment is poor:

The model ignores visual input
Outputs rely only on text priors
Hallucinations increase
Example Failure

Image: a dog Incorrect Output: “a cat”

This happens when visual features are misaligned with language space.

Task 1.3: Key Design Choice

Why No Cross-Attention?

Unlike models such as Flamingo, LLaVA avoids cross-attention and directly injects image tokens into the LLM.

Advantages

Simplicity

No architectural modification to the LLM
Efficiency

Lower computational and memory cost
Compatibility

Works with pretrained LLMs
Limitations

Weak Multimodal Fusion

No explicit interaction between modalities
Information Bottleneck

High-dimensional images compressed into limited tokens
Limited Reasoning

Struggles with spatial and compositional reasoning
Summary Table

Design	Pros	Cons
Token Injection	Simple, efficient	Weak multimodal fusion
Cross-Attention	Strong fusion	Higher cost
Part 2 — Training Pipeline

Task 2.1: Two-Stage Training

Training Objective

𝐿=−∑𝑡log𝑃(𝑦𝑡∣𝑥image,𝑦<𝑡) 

The model learns to predict the next token conditioned on the image and previous text.

Stage 1 — Feature Alignment

CLIP and LLM are frozen
Only the projection layer is trained
Goal: Align visual features with the LLM embedding space

What is learned: Mapping image features → meaningful token embeddings

Stage 2 — Visual Instruction Tuning

Train projection layer + LLM
Data format:

(Image + Instruction) → Response
Example

Input:

Image + "Describe the image"
Output:

"A dog is running in a park."
What the Model Learns

Stage	Learning
Stage 1	Representation alignment
Stage 2	Instruction following and multimodal behavior
Why Both Stages Are Needed

Stage 1 ensures the model understands visual input
Stage 2 teaches how to use that information in tasks
Without Stage 1 → no grounding Without Stage 2 → no useful responses

Task 2.2: Synthetic Data

Why Synthetic Data?

Human annotation is expensive and slow.

GPT-generated data provides:

Scalability
Speed
Low cost
Biases Introduced

Uniform writing style (LLM-like)
Possible incorrect reasoning
Limited diversity of scenarios
Effect on Generalization

Performs well on common tasks
Struggles with edge cases and real-world variability
Part 3 — Reflection

1. Is LLaVA Truly Multimodal?

LLaVA is not fully multimodal.

It is better described as:

A language model conditioned on visual features

This is because it does not learn a joint representation of vision and language, but instead relies on externally encoded visual embeddings.

2. Where Does Alignment Happen?

Alignment occurs at two levels:

Projection Layer (Structural Alignment)

Aligns embedding spaces
Instruction Tuning (Functional Alignment)

Teaches how to use visual information
Conclusion

Both stages are necessary:

Projection enables understanding
Instruction tuning enables correct behavior
3. Biggest Limitation

Weak Multimodal Fusion

Effects

Poor spatial reasoning
Hallucination
Limited compositional understanding
Root Cause

No cross-attention mechanism
Compressed visual representation
No iterative interaction between vision and language during inference
Final Summary

LLaVA connects vision and language using a projection layer
Images are converted into tokens and processed by an LLM
Training consists of alignment and instruction tuning
The architecture prioritizes simplicity and efficiency over deep multimodal reasoning
One-Line Takeaway

LLaVA works by aligning vision features with language embeddings rather than deeply integrating the two modalities.

