---
title: 10. Grocery Swiper (Part 3/3) - Recommender system
description: Now we just need to notify the user of the weekly recommendations
published: true
image: 'grocery_swiper/genie.png'
---

## [](#model)Preference-Model
The app prototype works wonders, and my girlfriend has enjoyed swiping on groceries - huge win! (but we are not done). The groceries shown are randomly picked, and the preferences are not being used for anything yet. The database has been populated with approximately 500 preferences, which will constitute the training and validation sets. 

For this system we need 2 models; <br>
**a)** A model that defines the **best grocery to display to the user** <br>
**b)** A model that defines the **best recommendations in order** (ranking model)

These might even be the same model, but their goals are quite different. However, we should first think of a proper way to do feature engineering that fits this task.

### [](#preprocessing)Feature Enginerring

#### Embedding models
I was stoked to do feature engineering, because I had thought of a *bullet-proof* plan of embedding the groceries in a large latent space. My intuition was that we didn't really care about the exact word of a grocery, but more the idea behind it. Such as if a person has a liking for lettuce, they might also like cabbage. Considering this assumption, it could actually be quite hard to define very specific preferences, without being bombarded with false positives of only somewhat similar groceries. Or maybe not.

![kanye]({{ site.baseurl }}/assets/images/grocery_swiper/kanye.gif "kanye")

(*I won't apologize for using that meme incorrectly*)

Learning from my past mistakes regarding embedding models on Danish text, I had a found myself a light-weight Danish-to-English translation LLM, in order to work on English text instead. In part 1, I show how this is then used to create the "bio" for the groceries - which had sadly shown subpar results. The dull quality of the bios created could surely be attributed to the light-weight text generator, right? But as it seems, the translations themselves, upon further inspection, are absolutely abysmal! My prime example is the translation of:

> "*Æblemost Hyldeblomst & Citron*"

Which would usually be translated to:

> "*Apple-juice Elderflower & Lemon*"

But for this model translates to:

> "*Emblem Cheese Shelf Blossom & Lemon*"

![mom_meme]({{ site.baseurl }}/assets/images/grocery_swiper/mom_meme.png "mom_meme")


**If you know Danish, I challenge you to inspect the sentence, and consider how it might have gone wrong**. It is really bad, as many translation destroy the meaning of the original phrase, and I had to try something else. 

Had I had the memory available within the automation pipeline (Github actions), I could simply use more powerful models, but this was not an option.

#### A Classic (Boring) Approach
The groceries provide very few numerical features (only `price` is relevant), and is outshined by the string features `name`, `category` (such as "vegatable"), and `brand`. The `category` and `brand` features are categorical by nature, but I choose to now treat the name as a category as well, as embedding the name did not work as planned. 

The `price` feature was **standardized**, and the remaining features were encoded as vectors. `category` and `brand` were **one-hot encoded**, whereas `name` was encoded as a binary bag-of-words vector, consisting of all non-stopwords present in the groceries' names. The number of features in the final processed dataset will increase as new categories and words enter the dataset (after each new swipe). The larger the dataset we gather, the better our model will be. 



### [](#active_learning)Active (Machine) Learning
I'd like to cover the interesting topic of **active (machine) learning**, which is a machine learning paradigm that is concerned with updating it's knowledge "on the fly" (*online*), as the user interacts with the system, instead of only learning *offline* in a dedicated training step prior to deployment.

In our case, we display groceries to the user, and they have to label them with either a "like" or "pass" (and a "super-like", but we ignore that for now). It does not make sense to expose the user to an item which matches other liked groceries completely, as we are confident this grocery will be labeled as "liked". Instead, we want to expose the user to groceries, where the model struggles the most. Figuring out the best candidate to display is found using **uncertainty sampling**. 

For binary classification, we simply consider samples with highest uncertainty (closest to 50% confidence) - aka. "The **least confidence** method". For multiple classes we can use **margin sampling** that select candidates where the top 2 classes have similar confidence. Or we can use **maximum entropy** can finds candidates with a largest spread of confidence across classes.

As the user swipes on a candidate grocery, the model should update its belief, which will impact the new candidates. Doing so without retraining the entire model requires a model that can be trained really fast, but also has an intrinsic probability metric associated with samples. A common choice for small datasets is a **Gaussian Process** model (even with its cubic training time), which essentially is a regression curve with associated confidences at each point. 

A more simple choice is the K-nearest-neighbors (KNN) model, that considers the label of the K closest neighbors. When neighbors agree more, the confidence increases. This is a **non-parametric** model, meaning that it doesn't require training. Instead, during inference the model looks at neighboring points in the training (in my case K=5). This K has to be high enough such that inference is robust, but also not so large that it will consider neighbors that are too far away, and pull the prediction in a completely wrong direction.

Even if this active learning approach did not make it to the final product, the system was designed with active learning in mind. Instead the user is just exposed to random unseen groceries.  


### [](#ranker)Ranking Model
With preferences now collected, we now need to model to rank the new, unseen groceries from the latest sales flyer from most preferable, to least preferable. The knee jerk reaction would be to use a ***learning-to-rank*** model, that orders items in a sophisticated way. This is supervised learning problem where we expect ordering during training to be known. That would require too much work, but probably yield the best results. Instead we will assign probabilities/confidences to each entry, and sort by those. 

The choice for such a model could be as simple as a logistic regression model, which I would have chosen, had I not already access to a perfectly fine KNN model. To incorporate a super-like in a KNN, we can simply duplicate the super-liked entry 2 times, in order to essentially "weight" the contribution of that entry more.

With K=5, the probabilities would only be in the set {0.6,0.8,1}, which definitely limits the expressiveness of prediction. I believe it's quite alright, since it's a somewhat hard problem to model anyway, as the dataset is very sparse. To get more varying probabilities, we should have a more dense dataset anyway. Or maybe just use a *Naïve Bayes* model that is very well sutied for datasets with binary features. Again, it's just a suspcision, but... 

![kanye]({{ site.baseurl }}/assets/images/grocery_swiper/kanye.gif "kanye")


## [](#email)Email Notification System

With a working ranking model, we can easily setup an email notification system by extending the automation pipeline. To make things even better, the database actually stores the device/user -id of the swipes. If multiple people interacted with the system, they could each get a recommendation sent, based on their own swipes. 

To format a pretty email, we can use simple HTML, and the results will look like this:

![notification]({{ site.baseurl }}/assets/images/grocery_swiper/notification.png "notification")

(*Technically this email was generated "manually" which is why the active sales are a week behind*)

A vast majority of the items displayed (not all are shown) have been approved by my girlfriend, meaning that the model succesfully recommends valid groceries. It currently displays 50 items, instead of 400 that are usually in a sales flyer. I'd say that is a huge improvement!

## [](#conclusion)Conclusion
This project has surely been a mouthful, having to stretch it across 3 months, which is the majority of my semester. However, this is exactly why I chose to do these monthly projects. All my previous attempts at making a larger project have failed because my ambitions had exceeded my long-term structuring capabilities. 

There is plenty of room for improvement in the final product, which I must simply ignore, as life has to move on. I like to cover the theory of things that I did not get to implement; mostly because I want to convince myself that I actually understand the material, and haven't simply given up. 

I think this project really encapsulates many aspects of me as a person and future engineer. 
* I build **for others**
* I get to **be creative and have fun**
* I get to use my skills, while also **stretching my capabilities** by challenging myself
* Things should work properly, but **time is not wasted trying to perfect it**

## [](#ai)Epilogue: Dependece on AI 

These past few years, vibe-coding has become an essential for programmers everywhere. The obvious reason is that you can produce code so much faster, with much less effort. So why does it still feel so wrong to use AI? Has outsourcing of our thinking gone too far, and why does this matter? Whenever I ask this question, a sense of ego and pride flares within me. 

I personally have a lot of conflict answering this question. On one hand, I find pride in the independence that comes with building my own code, but how is it then that I also feel pride in a project that has been created with AI-assistance? Is my sense of ownership of the project justified, or is it an illusion that covers over a lack of patience, skills, and perseverance? The architecture and idea for my latest project is certainly novel, but will that be all that matters in the end? The conflict will probably be there for a long time before I come to peace with it. 

I believe that the solution is not to surrender oneself to the AI, but instead to tame it. If vibe-coding is all you will amount to, you will probably be replaced quickly. Instead, become a person that can utilize AI well, and learn other skills, that are not so easily replaced my AI. This is just one of many challenges associated with this new exciting technology.

To finish up, I'd like to share some skills I look forward to learn in the future, that are hopefully not completely replacable by an AI agent:
* Robotics and mechanics (probably just using LEGOs, but still)
* Becoming more proficient in Linux
* Networking (the internet kind of networking, not the social kind)
* Playing piano at a higher level
* Creating/editing videos
* Better at cooking
* Public speaking

Whereas skills that are definitely aided by AI, that I also look forward to work with would be:
* Visual AI systems
* More web-scraping projects
* Reinforcement learning of games I like
* Reinforcement learning on bouldering simulation and bouldering route creation