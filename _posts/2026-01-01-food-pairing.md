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

I even found an [awesome website](https://epicure.kaikaku.ai/), which I was heavily inspired by, that use this FlavorGraph to create (free) food suggestions. They have done some processing on it which makes it way more usable than the vanilla FlavorGraph.

![epicure]({{ site.baseurl }}/assets/images/food_pairing/epicure.png "epicure")



## [](#flavorgraph)FlavorGraph
The FlavorGraph project was made by the *Data Mining and Information Systems Lab* at Korea University, and was made for the purpose of improving food pairing suggestions. 

To create such a graph, you would need some data. One part consist of a huge database of recipes, that tell you about ingredient co-occurance (top-down approach), while the another source tells us about the molecule composition of those ingredients (bottom-up approach). So the data is essentially a mix of practical and theoretical food pairings. 

The graph is *heterogeneous*, as the nodes can either be ingredients *or* molecules, meaning that the edges correspond to ingredient-ingredient, ingredient-molecule, and molecule-molecule connections. Ingredients and/or molecules that occur together in the dataset are more likely to be connected in the graph - where the authors have a method for determining when to add edges or not.

Among the 6,653 food nodes, only 416 of those have known molecule data, and are referred to as *hubs*. These are also more common foods, where as the niche foods just don't have that information (*non-hubs*). To create the embeddings from the graph, the authors utilize an altered version of the ***metapath2vec*** algorithm, which performs random walks in the graph many times to learn which nodes go together, switching between molecules-, hub-ingredient- and non-hub-ingredient- nodes, in order to capture rich information about nodes that don't have molecular data. From these walks you can learn dense vector embeddings of each node.

![embeddings]({{ site.baseurl }}/assets/images/food_pairing/embeddings.png "embeddings")

Using the FlavorGraph and associated embeddings, I'd like to see how well I am able to create ingredient suggestions, based on a few starting ingredients. In my examples, I use: `tomato`, `pasta`, and `chicken`.


## [](#graph)Graph



![steiner]({{ site.baseurl }}/assets/images/food_pairing/steiner.png "steiner")

![stock_cube]({{ site.baseurl }}/assets/images/food_pairing/stock_cube.png "stock_cube")




## [](#embeddings)Embeddings

<iframe src="{{ site.baseurl }}/assets/images/food_pairing/tsne.html" width="100%" height="600px" frameborder="0"></iframe>


![vector_addition]({{ site.baseurl }}/assets/images/food_pairing/vector_addition.png "vector_addition")


### ALL

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 3790 | Light balsamic vinaigrette salad dressing | 0.6783 |
| 3554 | Kashmiri chili powder | 0.6747 |
| 2103 | Dry chili pepper | 0.6657 |
| 6690 | Vegan burger | 0.6630 |
| 7023 | Wishbone italian dressing | 0.6629 |
| 2997 | Green chutney | 0.6616 |
| 66 | Ahi | 0.6612 |

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

### Tomato-Pasta

| Ingredient ID | Ingredient Name | Similarity Score |
|---------------|-----------------|------------------|
| 6500 | Tuna in brine | 0.7458 |
| 2532 | fresh_mozzarella_ball | 0.6875 |
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





## [](#conclusion)Final Words
* Epicure did some very needed processing, combining ingredients and probably removing some as well
* Reliability (of vanilla graph)
  * maybe don't include Kraft Cheese and all that
* Weird labelling here and there. Kefir can technicaly contain alcohol, but is better categories in Dairy, rather than Alcoholic Beverage
* Potential (it's early stage and all that)

