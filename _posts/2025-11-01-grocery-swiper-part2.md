---
title: 9. Grocery Swiper (Part 2/3) - Create an Android App
description: Now that we have data, let's create an app!
published: true
image: 'grocery_swiper/wizard_4.png'
---

## [](#prologue)Prologue



Content
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
   * Port-forwarding for 100% uptime
* Active (machine) learning
   * "What to show the user to learn the most"
   * Fast probability updates after each swipe
      * Non-parametric models (KNN/FAISS)
      * Low-complexity models (Logistic regression, Naïve Bayes)
   * Uncertainty sampling - Binary classification: Least confidence
   * Super-like = Generate 5 samples (?) or somehow do weighting of these entries
* Email notification
   * Separate github workflow
   * Top K sales entries (or C% confidence threshold entries)
      * Ranking model 
     