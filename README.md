
Assignment-4 Dissecting LLaVA Vision-Language Models

Name: Tarani Chilamkoti UCID: tc533 Course: DS681

Overview

In this report, I explain how LLaVA integrates vision and language using a simple but effective design. The focus is on how images are converted into tokens, how the model is trained, and what trade-offs this architecture makes.

Part 1 — Architecture Understanding

Task 1.1: Forward Pass Explanation

Diagram of Data Flow

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

The model takes a raw image and preprocesses it (resize, normalize).

2. Vision Encoder (CLIP)

[ f_{\text{vision}}(x) ]

CLIP encodes the image into a feature vector.

It captures semantic information (objects, actions).
Because CLIP is trained on image-text pairs, its embeddings already relate to language.
3. Projection Layer

[ z = W \cdot f_{\text{vision}}(x) ]

This layer maps CLIP features into the LLM embedding space.

Usually a linear layer or small MLP.
Output becomes compatible with text token embeddings.
4. Image Tokens

The projected vector is turned into a sequence:

[IMG_1, IMG_2, ..., IMG_k]
These are treated like tokens.

5. Language Model Input

Example:

[IMG TOKENS] + "What is happening in the image?"
The image tokens are prepended to the text.

6. Language Model

Standard transformer (e.g., LLaMA).
Uses self-attention over both image and text tokens.
7. Text Generation

[ P(y_t \mid x_{\text{image}}, y_{<t}) ]

The model generates text autoregressively.

Example

Image: A person riding a bike

Input:

[IMG TOKENS] + "What is the person doing?"
Output:

"The person is riding a bicycle."
Key Idea

The image is converted into token embeddings so the LLM can process it like text.

Task 1.2: Projection Layer Intuition

Why a Simple Projection Works

CLIP embeddings already encode semantic meaning.
LLM embeddings also represent semantic meaning (but in language form).
Because both spaces are structured, a linear transformation is often enough to align them.

Assumption

There is an approximate linear relationship between:

Vision embeddings
Language embeddings
So similar concepts stay close after mapping.

What Could Go Wrong?

If alignment is poor:

The model ignores the image
Outputs rely only on text
Hallucinations increase
Example Failure

Image: a dog Bad alignment → model says: “a cat”

Task 1.3: Key Design Choice

Why No Cross-Attention?

LLaVA does not use cross-attention (like Flamingo). Instead, it inserts image tokens directly into the LLM.

Advantages

Simplicity

No changes to LLM architecture
Efficiency

Fewer parameters and computations
Compatibility

Works with pretrained LLMs
Limitations

Weak fusion

No explicit interaction between vision and language
Information bottleneck

Image compressed into few tokens
Limited reasoning

Harder to understand spatial details
Summary Table

Design	Pros	Cons
Token Injection	Simple, fast	Weak multimodal fusion
Cross-Attention	Strong fusion	Expensive
Part 2 — Training Pipeline

Task 2.1: Two-Stage Training

Training Objective

[ L = - \sum_t \log P(y_t \mid x_{\text{image}}, y_{<t}) ]

The model learns to predict the next token given the image and previous text.

Stage 1 — Feature Alignment

CLIP and LLM are frozen
Only projection layer is trained
Goal: Align visual features with LLM embeddings

What is learned: How to convert image features into meaningful tokens

Stage 2 — Visual Instruction Tuning

Train projection + LLM
Data format:

(Image + Instruction) → Response
Example

Input:

Image + "Describe the image"
Output:

"A dog is running in a park."
What the Model Learns

Stage	Learning
Stage 1	Alignment
Stage 2	Instruction following
Why Both Stages Are Needed

Stage 1: makes image understandable
Stage 2: teaches how to respond
Without both, the model fails.

Task 2.2: Synthetic Data

Why Synthetic Data?

Human annotation is expensive

GPT-generated data is:

Scalable
Fast
Cheap
Biases Introduced

Similar writing style (LLM-like)
Possible incorrect answers
Limited diversity
Effect on Generalization

Works well on common tasks
Struggles with unusual or real-world cases
Part 3 — Reflection

1. Is LLaVA Truly Multimodal?

Not fully.

It is better described as:

A language model conditioned on visual features

Because:

Vision is converted into tokens
No deep multimodal fusion
2. Where Does Alignment Happen?

Two places:

Projection layer

Aligns embeddings
Instruction tuning

Teaches usage
Conclusion

Both are necessary:

Projection = understanding
Instruction tuning = behavior
3. Biggest Limitation

Weak Multimodal Fusion

Effects

Poor spatial reasoning
Hallucination
Limited compositional understanding
Cause

No cross-attention
Image compressed into tokens
Final Summary

LLaVA connects vision and language using a projection layer.
Images are turned into tokens and fed into an LLM.
Training uses alignment + instruction tuning.
The model is simple and efficient but has weak multimodal reasoning.
One Line Takeaway

LLaVA works by aligning vision features with language tokens, not by deeply integrating the two modalities.
