---
title: "St. Petersburg Paradox Simulations"
tags: [st petersburg paradox, probability, statistics, rstats, code, math, riddles]
---

A friend recently posed the [St. Petersburg Paradox](http://en.wikipedia.org/wiki/St._Petersburg_paradox) to me in the form of a riddle. I found it interesting, and wrote some R code to run some simulations. If you're familiar with the paradox, skip to the R code below. Otherwise, have a moment to read through the problem and solution in terms of expected value (I also enjoyed this [Stanford Encyclopedia of Philosophy write up](http://plato.stanford.edu/entries/paradox-stpetersburg/)).

The key point is that the expected value is infinite:

{% include figure.html src="stp1.png" class="noborder" %}
{% include figure.html src="stp2.png" class="noborder" %}
{% include figure.html src="stp3.png" class="noborder" %}

So, in theory, you should be willing to pay a large amount of money to play this game (in particular, anything less than infinity). Clearly that is not a good idea - hence a paradox.

But what if you could play the game many times? By the [law of large numbers](http://en.wikipedia.org/wiki/Law_of_large_numbers), the more times you play, the closer the average of the results should be to the expected value (which in our case is infinite). 

I was curious about what this would look like in reality, and wrote some R code to simulate playing the game many times. After running the code, I was initially surprised to see that even if you play the game a very large number of times, the average of the results was not converging to any number, let alone the expected value of infinity. 

The Stanford Encyclopedia of Philosophy includes a useful graph (reproduced below) that helps explains the interesting way that simulations of the St. Petersburg approach infinity. 

Essentially, every once in a while, you'll win big. This will cause a big jump in the average of the results thus far. The average will then slowly decrease (50% of the time you'll win 0), until another big jump. Because the average of the results increases at each jump, it becomes more difficult to increase your average as time goes on. So the big jumps become more and more rare.

I ran this for 20,000 simulations:

{% include figure.html src="st-petersburg-simulation.png" %}

Full code via [this GitHub Gist](https://gist.github.com/grano/fc854a2ffb2d6af8793b):

{% gist grano/fc854a2ffb2d6af8793b %}