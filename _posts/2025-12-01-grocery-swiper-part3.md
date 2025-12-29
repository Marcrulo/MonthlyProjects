---
title: 10. Grocery Swiper (Part 3/3) - Recommender system
description: Now we just need to notify the user of the weekly recommendations
published: true
image: 'grocery_swiper/genie.png'
---

## [](#model)Preference Model
The app prototype works wonders, and my girlfriend has enjoyed swiping on groceries - huge win! (but we are not done). The groceries shown are randomly picked, and the preferences are not being used for anything yet. The database has been populated with approximately 500 preferences, which will constitute the training and validation sets. 

For this system we need 2 models; <br>
**a)** A model that defines the **best recommendations in order** (ranking model) <br>
**b)** A model that defines the **best grocery to display to the user**

These might even be the same model, but their goals are quite different. However, we should first think of a proper way to do feature engineering that fits this task.

### [](#preprocessing)Feature Enginerring

#### Embedding models
I was stoked to do feature engineering, because I had thought of a *bullet-proof* plan of embedding the groceries in a large latent space. My intuition was that we didn't really care about the exact word of a grocery, but more the idea behind it. Such as if a person has a liking for lettuce, they might also like cabbage. Considering this assumption, it could actually be quite hard to define very specific preferences, without being bombarded with false positives, of somewhat similar groceries. 

![kanye]({{ site.baseurl }}/assets/images/grocery_swiper/kanye.png "kanye")

(*I apologize for using that meme wrongly*)

Learning from my past mistakes regarding embedding models on Danish text, I had a found myself a light-weight Danish-to-English translation LLM, in order to work on English text instead. In part 1, I show how this is then used to create the "bio" for the groceries - which had sadly shown subpar results. The dull quality of the bios created could surely be attributed to the light-weight text generator, right? But as it seems, the translations themselves, upon further inspection, are absolutely abysmal! My prime example is the translation of:

> "*Æblemost Hyldeblomst & Citron*"

Which would usually be translated to:

> "*Apple-juice Elderflower & Lemon*"

But for this model translates to:

> "*Emblem Cheese Shelf Blossom & Lemon*"

**If you know Danish, I challenge you inspect the sentence, and see how it might have gone wrong**. It is really bad, as many translation destroy the meaning of the original phrase, and I had to try something else. 

Had I had the memory available within the automation pipeline (Github actions), I could simply use more powerful models, but this was not an option.






### [](#ranker)Ranking Model


### [](#active_learning)Active (Machine) Learning

## [](#email)Email Notification System

## [](#conclusion)Conclusion

## [](#ai)Epilogue: AI Usage and Learning






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
* On AI, efficiency, and learning
   * Transparency
   * Learning
   * Efficiency
     