---
title: 11. Food Pairing
description: Elevate any dish using this machine learning approach for gastronomy
published: true
image: 'food_pairing/remy.png'
---


## [](#prologue)Prologue
I recently started watching ***Master Chef*** and became fascinated by how effortlessly contestants seem to assemble cohesive dishes, even under intense pressure. This made me wonder whether such skill is primarily the result of experience, or whether it draws on *gastronomic theory*. Although cooking is often viewed as an art, gastronomy has increasingly developed into a science, providing structured principles that explain why certain flavors and ingredients complement each other.

We have seen chemistry been used extensively in this field - most notably *Molecular Gastronomy*, which utilizes chemical properties of foods to create fascinating dishes, but is also used to learn about the molecular components of food, which allows us to understand food more in depth. And as it turns out, having a much richer representation of food gives rise to yet another sub-field: ***Computational Gastronomy***, which takes a data-driven approach to gastronomy, meaning that we can apply machine learning (Hooray!) .

As this is a rather new field, there is not much to work with out there. Luckily, I did fall upon an open-source project that blew my mind: [FlavorGraph](https://github.com/lamypark/FlavorGraph/tree/master) - a graph structure of ingredients and molecules (nodes) that pair well together in a dish, the closer they are in this network. The project also comes with a 300-dimensional vector embedding of all foods, that has somewhat similar properties to the graph is terms of proximity. 

I even found an awesome application ([Epicure](https://epicure.kaikaku.ai/)), which I was heavily inspired by, that use this FlavorGraph to create (free) food suggestions. They have done some processing on it which makes it way more usable than the vanilla FlavorGraph.

![epicure]({{ site.baseurl }}/assets/images/food_pairing/epicure.png "epicure")



## [](#flavorgraph)FlavorGraph
The FlavorGraph project was made by the *Data Mining and Information Systems Lab* at Korea University, and was made for the purpose of improving food pairing suggestions. 

To create such a graph, you would need some data. One part consist of a huge database of recipes, that tell you about ingredient co-occurance (top-down approach), while the another source tells us about the molecule composition of those ingredients (bottom-up approach). So the data is essentially a mix of practical and theoretical food pairings. 

The graph is *heterogeneous*, as the nodes can either be ingredients *or* molecules, meaning that the edges correspond to ingredient-ingredient, ingredient-molecule, and molecule-molecule connections. Ingredients and/or molecules that occur together in the dataset are more likely to be connected in the graph - where the authors have a method for determining when to add edges or not.

Among the 6,653 food nodes, only 416 of those have known molecule data, and are referred to as *hubs*. These are also more common foods, where as the niche foods just don't have that information (*non-hubs*). To create the embeddings from the graph, the authors utilize an altered version of the ***metapath2vec*** algorithm, which performs random walks in the graph many times to learn which nodes go together, switching between molecules-, hub-ingredient- and non-hub-ingredient- nodes, in order to capture rich information about nodes that don't have molecular data. From these walks you can learn dense vector embeddings of each node.

![embeddings]({{ site.baseurl }}/assets/images/food_pairing/embeddings.png "embeddings")

Using the FlavorGraph and associated embeddings, I'd like to see how well I am able to create ingredient suggestions, based on a few starting ingredients. In my examples, I use: `tomato`, `pasta`, and `chicken`.


## [](#graph)Graph
We have an undirected graph with weights corresponding to similarity, and to keep things simple, I only work with ingredient nodes (hubs and non-hubs). By itself, it is not a "recipe generator" of any kind. Instead, it shows how all ingredients are interconnected. For instance, `coffee` is closely related with `ladyfingers`, because of their common occurance in *Tiramisu*, but `coffee` is also related to certain `sirups`, as they are often combined in coffee-beverages. And yet, it's hard to say what the global impact on the graph is, based on the presence of `coffee`. There is supposedly a lage complex structure, but we can only clearly observe and try to understand the local structure. 

With this in mind, we can start creating a strategy, or *heuristic*, for how we will recommend ingredients for dishes, where we already have some ingredients. First of all, let's consider how we might think of the connectedness between two non-neighboring ingredients, `pasta` and `tomato`. If we consider the un-weighted scenario, these ingredients lie within a 2-hop neighborhood of each other (only 1 ingredient between them). You can connect them through either `canned tuna` or `sun-dried tomato pesto`, which makes sense. But if we consider the weighted graph (recall that weights refer to similarity), then the shortest (cheapest) path is 6 traversals long:

`pasta` → `sun-dried tomato pesto` → `garlic` → `canned chicken` → `lemon juice` → `nonfat cottage cheese` → `tomato`

We begin to understand that naively using the similarity between *2* ingredients leads us astray. Now consider the case when adding a *third* ingredient, `chicken`, as well:

![steiner]({{ site.baseurl }}/assets/images/food_pairing/steiner.png "steiner")

The red path is found by solving the ***Steiner Tree Problem*** that creates the ***Minimum Spanning Tree (MST)*** that connects K terminal nodes (in undirected and weighted graph). What if we consider the un-weighted graph? Then we connect all 3 ingredients with the `stock cube`:

![stock_cube]({{ site.baseurl }}/assets/images/food_pairing/stock_cube.png "stock_cube")

This problem is solved equivalently to the previous one, but all weights are set to be equal. This results in a much simpler solution. Even when adding unorthodox ingredients, it seems to find a link through only 1 other ingredient. In the case of adding `kefir`:

![cucumber]({{ site.baseurl }}/assets/images/food_pairing/cucumber.png "cucumber")

It most likely suggests cucumber because it could occur in tzatziki or salad dressing, both of which might occur in a dish with tomatoes. We can even try and add `beer` as an ingredient, which creates the link `bermuda onion`:

![bermuda]({{ site.baseurl }}/assets/images/food_pairing/bermuda.png "bermuda")

This type of onion does not seem to have any relation with beer as such, but they might share certain molecules that make them tasty together. The `bermuda` onion even seems to be related to the cucumber, which might aid in the cohesion of the dish. 

I think the best graph-based solution lies somewhere between the weighted and un-weighted graph algorithms. Just because ingredients are *slightly* connected doesn't mean that they necessarily make sense to have together in a dish. This is where utilizing the similarity weights could come in handy, as they would work as an indicator for when a connection between 2 ingredients is simply too bizarre. Also, this naive approach doesn't seem to take the interplay between all ingredients into account. I find it hard to create a model that does that properly, but it might be possible?


## [](#embeddings)Embeddings

With the graph we had very clear indicators of how well ingredients were connected, but with ingredients now instead existing in a 300-dimensional vector space, we only have position/distances to consider. This is not necessarily a bad thing, as we now have an intrinsic similarity metric between any 2 ingredients - the cosine similarity!

The authors explain that the properties of the embeddings are such that when adding 2 ingredients together, the resulting point/vector now represents a new entity that is both ingredients in one. Maybe you are wondering why we don't take the average of the vector instead? In all honesty, there might be cases where that is the best option, but the rule of thumb is that we can suffice with knowing the direction of a point. Taking the average between two points, *A* and *B* is just the same as scaling the A+B=C vector by the number of input points/vectors:

![vector_addition]({{ site.baseurl }}/assets/images/food_pairing/vector_addition.png "vector_addition")

Before diving into methods of suggesting ingredients, I'd like to provide an overview of how all embeddings are positioned wrt. each other. The following ***interactive plot*** is a projection of the 300-dim embeddings into 2D space, using T-SNE. This is a non-linear transformation that aims at preserving local structures, so it's easier to interpret how points are clustered in higher dimensions. 

Feel free to interact with the figure (*click it before trying to zoom in*). You might quickly find some obvious clusters such as dairy, which I imagine is closely clustered because they all share the main ingredient of *milk*.

<iframe src="{{ site.baseurl }}/assets/images/food_pairing/tsne.html" width="100%" height="600px" frameborder="0"></iframe>


If we return to the example of having 3 ingredients, `tomato`, `pasta`, and `chicken`, we should be able to generate a vector that encompasses all of these, simply by summing their vectors. The following table shows the 7 most similar foods:

### All 3 ingredients

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 3790 | Light balsamic vinaigrette salad dressing | 0.6783 |
| 3554 | Kashmiri chili powder | 0.6747 |
| 2103 | Dry chili pepper | 0.6657 |
| 6690 | Vegan burger | 0.6630 |
| 7023 | Wishbone italian dressing | 0.6629 |
| 2997 | Green chutney | 0.6616 |
| 66 | Ahi | 0.6612 |

None of these ingredients are even remotely related to the ones we saw using the graph methods. The `(...)vinaigrette` is the most similar ingredient, which most likely comes from the tomato that is commonly eaten alongside balsamic vinegar. I can see how most of these suggestions are connected to either of the 3 ingredients, but I don't think the suggestions make sense when considering the 3 ingredients together.

For the tomato, we see some suggestions that make a lot of sense, even though `vegan burger` is one of the weirdest ingredients I have ever witnessed (it was also present before).


### Tomato

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 1737 | Crisp salad green | 0.9028 |
| 2103 | Dry chili pepper | 0.8935 |
| 3790 | Light balsamic vinaigrette salad dressing | 0.8917 |
| 6802 | Western salad dressing | 0.8915 |
| 6381 | Tex mex cheese | 0.8913 |
| 2997 | Green chutney | 0.8910 |
| 6690 | Vegan burger | 0.8908 |

Something seems a bit off with some of the "ingredients". I can imagine that this is a product of web-scraping gone wrong. For instance, look at the suggestions for chicken; 2 different variations of `kraft cheese` that are very specifically the one variant called "with a touch of Philadelphia". 

### Chicken

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 5979 | Sourdough roll | 0.8345 |
| 4019 | Macaroni shells and cheese | 0.8332 |
| 3628 | Kraft shredded triple cheddar cheese with a touch of philadelphia | 0.8226 |
| 3219 | Heads of garlic | 0.8117 |
| 4300 | Montreal chicken seasoning | 0.7922 |
| 3627 | Kraft shredded three cheese with a touch of philadelphia | 0.7836 |
| 5392 | Reduced sodium cream of chicken soup | 0.7748 |

And yes, it does hurt my soul that 3/7 of the suggestions for `chicken` is (cheap) cheese. 

I can definitely see most of following suggestions make sense. I see `wine`, `parmesan cheese`, `italian plum tomato`, `red pepper` and `basil pesto`. But even `tune in brine` makes sense for pasta salads.

### Pasta

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 712 | Broccoli floret | 0.7370 |
| 2119 | Dry marsala wine | 0.7336 |
| 6698 | Vegan parmesan cheese | 0.7267 |
| 3462 | Italian plum tomato | 0.7233 |
| 6500 | Tuna in brine | 0.7038 |
| 6269 | Sweet red pepper | 0.6927 |
| 333 | Basil pesto | 0.6846 |

Feel free to look at the remaining food pairs. I think it's a good thing that after adding two ingredients, some of the original suggestions are kept. This would indicate that the suggestions are still valid, even when adding more ingredients. But really, the results don't make too much sense. Or maybe they actually do? I don't know, I'm not a professional chef ¯\\\_(ツ)\_/¯

### Tomato-Pasta

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 6500 | Tuna in brine | 0.7458 |
| 2532 | Fresh mozzarella ball | 0.6875 |
| 2997 | Green chutney | 0.6769 |
| 2103 | Dry chili pepper | 0.6744 |
| 3790 | Light balsamic vinaigrette salad dressing | 0.6729 |
| 6690 | Vegan burger | 0.6712 |
| 4569 | Oven roasted turkey breast | 0.6678 |

### Tomato-Chicken

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 3554 | Kashmiri chili powder | 0.8263 |
| 3790 | Light balsamic vinaigrette salad dressing | 0.7620 |
| 7023 | Wishbone italian dressing | 0.7558 |
| 1737 | Crisp salad green | 0.7521 |
| 6381 | Tex mex cheese | 0.7520 |
| 66 | Ahi | 0.7494 |
| 7005 | Whole wheat toast | 0.7486 |

### Pasta-Chicken

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 2741 | Frozen vegetable | 0.6877 |
| 3627 | Kraft shredded three cheese with a touch of philadelphia | 0.6151 |
| 4019 | Macaroni shells and cheese | 0.6095 |
| 3219 | Heads of garlic | 0.6001 |
| 3628 | Kraft shredded triple cheddar cheese with a touch of philadelphia | 0.5992 |
| 5979 | Sourdough roll | 0.5972 |
| 6122 | Stock cube | 0.5783 |

But hey, the `stock cube` finally showed up!



## [](#conclusion)Final Words
As always, this was very fun to work with! While I didn't achieve results anywhere near *Epicure's*, it was still fascinating to work with. It's obvious that  the Epicure version was way more curated, as they have removed and/or combined ingredients together (for instance you can't pick or get suggested `kraft cheese` on the website). The original data also labels Kefir as an alcoholic beverage, not dairy. Kefir *can* contain alcohol, but it's obviously not an alcoholic beverage.

I look forward to more open-source projects on computational gastronomy, especially if they get to improve FlavorGraph, or maybe create something entirely new!