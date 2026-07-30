---
title: 16. Finally a way to beat my mom in Wordfeud 🔠
description: I created a bot to finally beat my mom in Wordfeud
published: true
image: 'wordfeud/board.png'
---

## [](#prologue)Prologue
Wordfeud is the mobile phone equivalent of the classic game "Scrabble". The goal of this game is to form words on a board each round using the existing letters, and up to 7 letters from your hand, where you score more points depending on the rarity of individual letters. Despite being fond of grammar and writing, I still suck at Wordfeud. My mom, however, is very good at it, and seems to always beat her friends and myself in the game. But one day I thought to myself; "how hard can it possibly be to create a bot in this game to beat my mom?". Knowing about fancy models like the chess bot *DeepBlue*, I was confident I could make something similar. 

## [](#natural-stupidity)Natural Stupidity
Wordfeud has some similarities with chess, in that it's turn-based and has a discrete number of possible actions to take each turn. DeepBlue uses the **alpha-beta** algorithm that does a tree search of possible outcomes, which prunes branches in order to expand the search even more, with the goal of finding the move that gives the largest advantage, assuming that the opponent plays perfectly. This method assumes perfect information, which Wordfeud unfortunately doesn't have, due to the hidden letters of the other player. Instead, the **expecti-mini-max** algorithm takes uncertainty into account. I thought this was the obvious way to solve the Wordfeud game, but realized that this might be a bit overkill to just win a game against my mom. I realized that the best solution wasn't advanced AI, but rather a very simple tree search. More on that later - now it's time to build the actual game "engine".

## [](#board)Board and simulation
It would be a drag to manually type in the current state of the board, hence I started off by making a script to extract the board state from a screenshot (that has been quite blurred due to *messenger*'s jpeg compression). 

![screenshot]({{ site.baseurl }}/assets/images/wordfeud/read_screenshot.png "read_screenshot")

By splitting the board into a 15x15 grid and reading the top left pixel value in each cell, it's possible to assign a class to each cell purely based on its color. These are the multipliers, blanks, and letters. It was actually quite hard reading the letters, using open-source **OCR** models, as there was often confusion between especially 'A' and 'Å'. In the end, I opted for a template matching approach, since all letters always look the same. 

### Dictionary
We can read the board now, but should also be able to suggest valid words. For this, I managed to find a dataset of all words, including all word inflections, from the [ordnet.dk](https://ordnet.dk/) online Danish dictionary. It even includes parts of speech ("ordklasser"), that I could use for filtering. I could remove fancy abbreviations, proper nouns, etc., but also remove words of length 1, and words with more than 15 characters.

## [](#bot)The Bot
So, a very intelligent bot would have access to a value function (neural net) that estimates the value of a board position, and combine that with a search tree algorithm. On the other end of the spectrum, we could greedily pick the word that yielded most points each round. What I found to work well is to pick the best word that also minimized the expected number of points for my opponents. The algorithm is:

1. Calculate all legal moves
2. Pick top K of these legal moves based on points
3. For each K moves, sample N possible sets of letters on my opponent's hand (based on which letters that are left), and calculate their best move
4. Pick one of the K moves that maximized the difference between that word, and the opponent's expected best move.

In other words, I pick the move where I expect to get the largest point difference after my opponent's turn. 

The following shows the best 10 moves and their scores. We see that the #1 move is "KEHRAUS" (43 points), which doesn't score nearly as much as #9 "HAUSA" for 63 points. This is because it sets up 2 triple word multipliers for the opponent. I can't tell if Wordfeud accepts "HAUSA" as a word, but it is definitely in the dictionary.

![suggestions]({{ site.baseurl }}/assets/images/wordfeud/suggestions.png "suggestions")



## [](#game)The game against my mom

The following are screenshots from the game that show how the board and the points progress throughout the game. The word "INDLÆS" gives me a strong lead.

![game-part1]({{ site.baseurl }}/assets/images/wordfeud/game-part1.png "game-part1")

The bot finds many strong plays such as "KEHRAUS" (whatever that means), "APROPOS" and "BØFFEL". It's very good at utilizing the multipliers, without sacrificing too many possibilities for the opponent to use the multipliers. 

![game-part2]({{ site.baseurl }}/assets/images/wordfeud/game-part2.png "game-part2")

The late-game wasn't very exciting, but it kept playing solidly.
![game-part3]({{ site.baseurl }}/assets/images/wordfeud/game-part3.png "game-part3")

At this point, I had an almost 200 point advantage, but then the bot kept suggesting words that didn't exist, but also had trouble differentiating between 'A' and 'Å' again - something I thought to be fixed. I ended up playing a bit myself, and skipping turns as to not give my opponent too many opportunities while the gap slowly decreased. But guess what? 



## [](#victory)Victory
I won! With a 113 point difference. I could never have done that myself against my mom in a million years.

![final_board]({{ site.baseurl }}/assets/images/wordfeud/final_board.png "final_board")

This was SO fun to make and play with. There are still a few bugs that needed fixing - mainly board reading capabilities. And maybe a way to play the game automatically, but that is quite difficult without an official Wordfeud API for taking turns. I definitely see a future where this project is more autonomous, and is much more complex, utilizing something like expecti-mini-max search and neural net based value functions.