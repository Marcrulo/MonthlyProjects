---
title: 9. Grocery Swiper (Part 2/3) - Create an app
description: Now that we have data, let's create an app!
published: true
image: 'grocery_swiper/wizard_4.png'
---


## [](#scraping)Scraping - fix
Since last time, the automation pipeline has failed on me, due to a strong assumption I made about the sales flyer website's scraping prevention. As it turns out, the ID used to identify the json data file on their server actually *does* change dynamically, meaning that I need a way to catch that as well. Currently, I depend on the data path to the json files to only depend on the item-id displayed directly on the webpage. To fix this, I used the `playwright` package which simulates a browser without a GUI (aka *headless*), in order to catch the data packages (and thereby the ID for the json files) that are retrieved on the client side, but not accesible through scraping of the static website.

## [](#attempt-1)Attempt 1 - Android Studio and Kotlin
I tried to go through Google's official [Android Studio + Kotlin (programming language) tutorial](https://developer.android.com/get-started/overview), in order to create my own Android app. After tedious amounts of tutorial-ing, I was overwhelmed of the idea of building this entire thing in Kotlin. I also needed a backend, and smart access to a database service such as Firebase. I do believe project building is the best way to learn new tools and skills; but this was just too big of a mouthful, and I had to try something else.

## [](#attempt-2)Attempt 2 - Online Software Creation using AI
There has been a lot of hype around tools such as [replit](https://replit.com/) that promise to create fully working software using only prompting. I was a bit conflicted with using it, as it would mean surrendering more of my coding to the AI assistants. But I realize that AI is the future, and it's better to harness its strengths rather than being a slave of my own ego - preventing me from adapting to the demands of the future. So here's what I did:

I used both [replit](https://replit.com/) and [Firebase Studio](https://firebase.studio/), which essentially do the same thing. I prompted something like:

> *Create a Tinder-like mobile app, but let each profile be a grocery item*

With this very simple prompt it got ...


## [](#attempt-3)Attempt 3 - Website Imitating a Mobile App





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
     