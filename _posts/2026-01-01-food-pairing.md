---
title: 11. Food Pairing
description: Elevate any dish using this machine learning approach for gastronomy
published: true
image: 'food_pairing/remy.png'
---



Github repo: [Elevate My Dish](https://github.com/Marcrulo/ElevateMyDish)

## [](#prologue)Prologue
I recently started watching ***Master Chef*** and became fascinated by how effortlessly contestants seem to assemble cohesive dishes, even under intense pressure. This made me wonder whether such skill is primarily the result of experience, or whether it draws on *gastronomic theory*. Although cooking is often viewed as an art, gastronomy has increasingly developed into a science, providing structured principles that explain why certain flavors and ingredients complement each other.

We have seen chemistry been used extensively in this field - most notably *Molecular Gastronomy*, which utilizes chemical properties of foods to create fascinating dishes, but is also used to learn about the molecular components of food, which allows us to understand food more in depth. And as it turns out, having a much richer representation of food gives rise to yet another sub-field: ***Computational Gastronomy***, which takes a data-driven approach to gastronomy, meaning that we can apply machine learning (Hooray!) .

As this is a rather new field, there is not much to work with out there. Luckily, I did fall upon an open-source project that blew my mind: [FlavorGraph](https://github.com/lamypark/FlavorGraph/tree/master) - a graph structure of ingredients and molecules (nodes) that pair well together in a dish, the closer they are in this network. The project also comes with a 300-dimensional vector embedding of all foods, that has somewhat similar properties to the graph is terms of proximity. 

I even found an [awesome website](https://epicure.kaikaku.ai/) that use this FlavorGraph to create (free) food suggestions. They have done some processing on it which makes it way more usable than the vanilla FlavorGraph, which I was heavily inspired by, as they do the exact thing I want to do!

![epicure]({{ site.baseurl }}/assets/images/food_pairing/epicure.png "epicure")



## [](#flavorgraph)FlavorGraph
The FlavorGraph project was made by the *Data Mining and Information Systems Lab* at Korea University, and was made for the purpose of improving food pairing suggestions. 

To create such a graph, you would need some data. One part consist of a huge database of recipe, that tell you about ingredient co-occurance (top-down approach), while the other tells us about the molecule composition of those ingredients (bottom-up approach). So the data is essentially a mix of practical and theoretical food pairings. 

The graph is *heterogeneous*, as the nodes can either be ingredients *or* molecules, meaning that the edges correspond to ingredient-ingredient, ingredient-molecule, and molecule-molecule connections. Ingredients and/or molecules that occur together in the dataset are more likely to be connected in the graph - where the authors have a method for determining when to add edges or not.

Among the 6,653 food nodes, only 416 of those have known molecule data, and are referred to as *hubs*. These are also more common foods, wheres the niche foods just don't have that information (*non-hubs*). To create the embeddings from the graph, the authors utilize an altered version of the ***metapath2vec*** algorithm, which performs random walks in the graph many times to learn wihch nodes to together, switching between molecules-, hub-ingredient- and non-hub-ingredient- nodes, in order to capture rich information about nodes that don't have molecular data. From these walks you can learn dense vector embeddings of each node.

![embeddings]({{ site.baseurl }}/assets/images/food_pairing/embeddings.png "embeddings")



## [](#graph)Graph

* Explore structure of graph
* Visualize

## [](#embeddings)Embeddings
* Explore properties of embeddings
* Visualize 





## [](#conclusion)Conclusion
* How to recommend foods

