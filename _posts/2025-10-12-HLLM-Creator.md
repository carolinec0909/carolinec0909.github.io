---
title: "HLLM-Creator: Hierarchical LLM Framework for Personalized Creative Generation"
date: 2025-10-12
---

**GitHub Repository:** [HLLM-Creator](https://github.com/bytedance/HLLM/tree/main)

## Objective
To efficiently generate personalized and factually consistent creative content, such as ad titles, tailored to individual user interests at a massive scale for use in industrial ads recommendation systems.

## Background
![Personalized Creative Generation](/assets/images/personalized-content-creation.png)
Generating personalized content is a significant challenge in online advertising and content creation.  
A single product can have multiple selling points, and different users will be interested in different aspects.  
Manually creating tailored content for diverse user groups is not scalable.  

Existing AI-based solutions often struggle with:
- Accurately modeling user interests  
- Avoiding factual "hallucinations" in the generated content  
- Scaling to millions of users and items  

**HLLM-Creator** addresses these challenges by proposing a hierarchical LLM-based framework for personalized creative generation, with a focus on practical application in real-world ad systems.

## Model Architecture
![Model Architecture](/assets/images/hllm-architecture.png)

HLLM-Creator uses a three-part architecture:

1. **Item LLM**  
   Encodes item features from ad titles and descriptions into item embeddings.

2. **User LLM**  
   Aggregates the item embeddings of a user's clicked ads to create a user embedding that represents their interests.

3. **Creative LLM**  
   Takes the user embedding and ad constraints (original title, selling points) to generate a personalized ad title.

## CoT-driven Personalized Title Dataset Construction
![CoT Driven Dataset Construction](/assets/images/cot-dataset-construction.png)

A key innovation is the **Chain-of-Thought (CoT)-driven Personalized Title Dataset Construction**.  
Since personalized data is scarce, HLLM-Creator uses a CoT approach to generate a high-quality synthetic dataset for training.

This process involves:

1. **User Interest Profiling**  
   An LLM extracts user interests from their behavior sequence.

2. **Interest-Driven Title Generation**  
   The LLM then uses these interests to generate personalized ad titles, focusing on the selling points a user is most likely to care about.

3. **Hallucination-Free Title Filtering**  
   A final filtering step removes any generated titles that contain fabricated information.


## Training Objectives

The model is trained with a combination of losses to ensure the generated titles are both personalized and relevant:

- **Recommendation Objective Loss (cls loss)**  
  A binary classification loss that predicts whether a user will click on an item.

- **Semantic Alignment Loss (align loss)**  
  A contrastive loss (InfoNCE) that aligns the user embedding with the user's interests.

- **Reconstruction Loss (recon loss)**  
  A loss that ensures the user embedding contains compressed user interest information.

## Inference Workflow for Industrial Applications
![Inference Workflow](/assets/images/inference-workflow.png)

To be practical for industrial use, HLLM-Creator uses a multi-step inference process:

1. **User Embedding Extraction**  
   Use the HLLM (Item LLM + User LLM) pipeline to extract embeddings for a large user base (performed offline).

2. **User Clustering**  
   Apply *k*-means clustering to group users into *K* clusters based on their embeddings.

3. **Cluster-Level Generation**  
   Use each cluster center embedding as a proxy for users in that cluster, feeding it into the Creative LLM (along with the ad content) to generate a personalized title.

4. **Matching Score Computation**  
   Compute the matching score between each ad and cluster center using the trained **Item-User Predictor**.

5. **Top N Cluster Generation**  
   Generate personalized titles only for the top-N matching scoring user clusters to balance personalization and scalability.

## Experiments
![Industry Deployment Framework](/assets/images/deployment.png)
**Dataset**  
2.6 million samples from online user click logs (click sequence, search query, original ad title, and selling points).

**Training Data**  
650K samples for model training, with personalized titles generated using the CoT-driven method.

**Evaluation**  
500 samples for offline evaluation.

**Models Used**  
- - TinyLlama-1.1B for the Item and User LLMs  
Qwen3-8B for the Creative LLM  

**Online Evaluation Metrics**  
The model showed significant improvements in:
- Adss (Advertiser Score)  
- Advv (Advertiser Value)  
- RankAdvv (Rank Advertiser Value)

**Offline Evaluation Metrics**  
Good-Same-Bad (GSB) preferences were used to evaluate the quality of the generated titles.

## Potentials for Ads Personalization

HLLM-Creator has significant potential for both personalized advertising and organic content creation.  
The ability to generate content tailored to user interests at scale can lead to more engaging and effective ads.

Beyond advertising, this framework could be used to generate:
- Personalized video titles  
- Descriptions  
- Short scripts for content creators  

These applications can help creators and advertisers better connect with their audience.  
The paper demonstrates the **practical value and scalability** of this approach in a real-world industrial setting.
