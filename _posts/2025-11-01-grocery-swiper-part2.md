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
There has been a lot of hype around tools such as [replit](https://replit.com/) that promise to create fully working software using only prompting. I was a bit conflicted with using it, as it would mean surrendering more of my coding to the AI assistants. But I realize that AI is the future, and it's better to harness its strengths rather than being a slave of my own ego - preventing me from adapting to the demands of the future. So I continued on.

I used both [replit](https://replit.com/) and [Firebase Studio](https://firebase.studio/), which essentially do the same thing. I prompted something like:

> *Create a Tinder-like mobile app, but let each profile be a grocery item*

With this very simple prompt and some small tweaks, I got to this point: <br>
(Left: *Replit*. Right: *Fireebase Studio*)

![both]({{ site.baseurl }}/assets/images/grocery_swiper/both.png "both")

The obvious upside is that this method of working is very efficient and satisfying, as you are simply managing the AI to create your vision. I find that the code assistant found in i.e. VSCode also feels quite similar. As the free credits ran out, I was wondering how I would even deploy it. I did not want to pay anything to get it deployed, so I tried downloading the code, and running it locally. 

The first thing I noticed was that the codebase was very large (300 MB), and was not even in a mobile app format. I had tried asking for a Kotlin-based, Android compatible app, but it kept using React/Typescript - probably in order to display the prototype in the browser I was working on, but I am not sure. I don't think they want users to use their code outside of the online environments.

I had given up on these type of AI tools, as they would limit my freedom in the long run. Instead, I gladly took their idea of simply simulating an app through the web browser - something that I am way more comfortable working with. So, plans changed. The mobile app will instead be a website, not an Android app, which I had first anticipated.

## [](#attempt-3)Attempt 3 - Website Imitating a Mobile App
I hosted a server locally using **Flask**, with the app itself being simple HTML/CSS/JS, with an SQLite database for storing swipe data for all users. 

![app]({{ site.baseurl }}/assets/images/grocery_swiper/app.png "app")


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
     