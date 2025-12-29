---
title: 10. Grocery Swiper (Part 3/3) - Recommender system
description: Now we just need to notify the user of the weekly recommendations
published: true
image: 'grocery_swiper/genie.png'
---

## [](#prologue)Prologue






* Oh well, they do change the URL dynamically. There's no free lunch :/ . Let's create a way to check for the correct URL! 
   * `playwright` package
   * Simulate browser (headless) and listen for network packages
* First attempt (Android studio)
   * Android studio + Kotlin tutorial (got overwhelmed)
   * Hard to implement AI assistant
* Second attempt (online AI tools)
   * Firebase studio
   * Replit
   * Retrieving code (large codebase)
   * Not even a mobile app - just website (I can do that myself then)
* Third attempt (website app)
   * Simple JS/HTML/CSS + Flask + SQLite
   * ...
* Raspberry PI server hosting
   * Port-forwarding for 100% uptime (?)
   * Tunneling using ngrok
   * Server + DB hosting on same port (flask app)
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
     