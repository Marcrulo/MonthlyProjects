---
title: 10. Grocery Swiper (Part 3/3) - Recommender system
description: Now we just need to notify the user of the weekly recommendations
published: true
image: 'grocery_swiper/genie.png'
---

## [](#prologue)Prologue






* Preference model
   * KNN (5-NN)
   * Feature engineering
      * One-hot
      * Scaling
      * Embeddings using LLMs?
   * Super-like = Generate 3 samples
   * Majority vote
* Idea/theory: Active (machine) learning
   * "What to show the user to learn the most"
   * Fast probability updates after each swipe
      * Non-parametric models (KNN/FAISS)
      * Low-complexity models (Logistic regression, Naïve Bayes)
   * Uncertainty sampling - Binary classification: Least confidence
* Email notification
   * Separate github workflow
   * Top K sales entries (or C% confidence threshold entries)
      * Ranking model 
* On AI usage
   * Transparency
   * Learning
   * Efficiency
     