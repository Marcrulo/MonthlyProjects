---
title: 10. Grocery Swiper (Part 3/3) - Recommender system 🤖
description: Now we just need to notify the user of the weekly recommendations
published: true
image: 'grocery_swiper/genie.png'
---

Github repo: [Grocery Swiper](https://github.com/Marcrulo/grocery_swiper)

## [](#model)Preference-Model
The app prototype works wonders, and my girlfriend has enjoyed swiping on groceries - huge win! (but we are not done). The groceries shown are randomly picked, and the preferences are not being used for anything yet. The database has been populated with approximately 500 preferences, which will constitute the training and validation sets. 

For this system we need 2 models; <br>
**a)** A model that defines the **best grocery to display to the user** <br>
**b)** A model that defines the **best recommendations in order** (ranking model)

These might even be the same model, but their goals are quite different. However, we should first think of a proper way to do feature engineering that fits this task.

### [](#preprocessing)Feature Engineering

#### Embedding models
I was stoked to do feature engineering, because I had thought of a *bullet-proof* plan of embedding the groceries in a large latent space. My intuition was that we didn't really care about the exact word of a grocery, but more the idea behind it. Such as if a person has a liking for lettuce, they might also like cabbage. Considering this assumption, it could actually be quite hard to define very specific preferences, without being bombarded with false positives of only somewhat similar groceries. 

Learning from my past mistakes regarding embedding models on Danish text, I had found myself a light-weight Danish-to-English translation LLM, in order to work on English text instead. In part 1, I show how this is then used to create the "bio" for the groceries - which had sadly shown subpar results. The dull quality of the bios created could surely be attributed to the light-weight text generator, right? But as it seems, the translations themselves, upon further inspection, are absolutely abysmal! My prime example is the translation of:

> "*Æblemost Hyldeblomst & Citron*"

Which would usually be translated to:

> "*Apple-juice Elderflower & Lemon*"

But for this model translates to:

> "*Emblem Cheese Shelf Blossom & Lemon*"

![mom_meme]({{ site.baseurl }}/assets/images/grocery_swiper/mom_meme.png "mom_meme")


**If you know Danish, I challenge you to inspect the sentence, and consider how it might have gone wrong**. It is really bad, as many translations destroy the meaning of the original phrase, and I had to try something else. 

Had I had the memory available within the automation pipeline (Github actions), I could simply use more powerful models, but this was not an option. In other words, using text-embeddings was simply not viable for this project.

#### A Classic (Boring) Approach
The groceries provide very few numerical features (only `price` is relevant), and is outshined by the string features `name`, `category` (such as "vegetable"), and `brand`. The `category` and `brand` features are categorical by nature, but I choose to now treat the name as a category as well, as embedding the name did not work as planned. 

The `price` feature was **standardized**, and the remaining features were encoded as vectors. `category` and `brand` were **one-hot encoded**, whereas `name` was encoded as a binary bag-of-words vector, consisting of all non-stopwords present in the groceries' names. The number of features in the final processed dataset will increase as new categories and words enter the dataset (after each new swipe). The larger the dataset we gather, the better our model will be. 



### [](#active_learning)Active (Machine) Learning
I'd like to cover the interesting topic of **active (machine) learning**, which is a machine learning paradigm that is concerned with updating its knowledge "on the fly" (*online*), as the user interacts with the system, instead of only learning *offline* in a dedicated training step prior to deployment.

In our case, we display groceries to the user, and they have to label them with either a "like" or "pass" (and a "super-like", but we ignore that for now). It does not make sense to expose the user to an item which matches other liked groceries completely, as we are confident this grocery will be labeled as "liked". Instead, we want to expose the user to groceries, where the model struggles the most. Figuring out the best candidate to display is found using **uncertainty sampling**. 

For binary classification, we simply consider samples with highest uncertainty (closest to 50% confidence) - aka. the "**least confidence** method". For multiple classes we can use **margin sampling** that selects candidates where the top 2 classes have similar confidence. Or we can use **maximum entropy** that finds candidates with a largest spread of confidence across classes (including the top class).

As the user swipes on a candidate grocery, the model will update its belief, which will impact the new candidates. Doing so without retraining the entire model requires a model that can be trained really fast, but also has an intrinsic probability metric associated with samples. A common choice for small datasets (in terms of number of total swiped items) is a **Gaussian Process** model (even with its cubic training time), which essentially is a regression curve with associated confidences at each point. 

A more simple choice is the **K-nearest-neighbors (KNN)** model, that considers the label of the K closest neighbors. When neighbors agree more, the confidence increases. This is a **non-parametric** model, meaning that it doesn't require training. Instead, during inference the model looks at neighboring points in the training. I found out that for this small dataset, training time is trivially small, hence using a more advanced model such as **XGBoost** yielded the best results.

In practice, we load the 20 most uncertain items into cache and showcase those. Then retrain the model after the user swipes the first 10 items, while displaying the remaining 10 items in the meantime. And that repeats itself.


### [](#ranker)Ranking Model
With preferences eventually collected, we now need to rank the new, unseen groceries from the latest sales flyer from most preferable, to least preferable. The knee jerk reaction would be to use a ***learning-to-rank*** model, that orders items in a sophisticated way. But that is a supervised learning problem where we expect ordering during training to be known. That would require too much work, but probably yield the best results. Instead we will assign probabilities/confidences to each entry, and sort by those. 

The choice for such a model could be as simple as a logistic regression model, which I would have chosen, had I not already access to a perfectly fine XGBoost model. To incorporate a super-like in such a model, we can simply duplicate the super-liked entry 10 times, in order to essentially "weight" the contribution of that entry more.

## [](#email)Email Notification System

With a working ranking model, we can easily setup an email notification system by extending the automation pipeline. To make things even better, the database actually stores the device/user -id of the swipes. If multiple people interacted with the system, they could each get a recommendation sent, based on their own swipes. 

To format a pretty email, we can use simple HTML, and the results will look like this:

![notification]({{ site.baseurl }}/assets/images/grocery_swiper/notification.png "notification")

(*Technically this email was generated "manually" which is why the active sales are a week behind*)

A vast majority of the items displayed (not all are shown) have been approved by my girlfriend, meaning that the model succesfully recommends valid groceries. It currently displays 50 items, instead of 400 that are usually in a sales flyer. I'd say that is a huge improvement!

## [](#conclusion)Conclusion
This project has surely been a mouthful, having to stretch it across 3 months, which is the majority of my semester. However, this is exactly why I chose to do these monthly projects. All my previous attempts at making a larger project have failed because my ambitions had exceeded my long-term structuring capabilities. 

Originally the project was officially finished with several things not working properly, which I had just accepted. But testing the app with my girlfriend wanted me to improve her user-experience, which essentially required me to finish up those last things. 

I think this project really encapsulates many aspects of me as a person and future engineer. 
* Building **for others**
* Get to **be creative and have fun**
* Use my skills, while also **stretching my capabilities** by challenging myself
* Having things work properly, while **not wasting time trying to perfect it**

<br>
These next 6 months I will be writing my Master's thesis as well as writing a curriculum for the UNF summer camp, where I will teach introductory deep learning. But I am certain that there will still be time for these monthly projects, while still performing well in the other things. This will be very exciting for sure!