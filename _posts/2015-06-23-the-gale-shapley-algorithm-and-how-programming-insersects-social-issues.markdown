---
layout: single
title: "The Gale-Shapley Algorithm and How Programming Intersects with Social Issues"
date: 2015-06-23 14:12:03 -0400
blurb: "A Discussion on The Intersection between Computer and Social Sciences"
---
In college, I took a Computer Science course on Algorithms, where we explored ways that programmers and computer scientists approach complex problem solving. We looked at greedy algorithms, dynamic programming, recursion, network flow algorithms, and many more. One of the first algorithms we studied in the beginning of the course was the Gale-Shapley algorithm to solve the Stable Marriage problem.

##### The Stable Marriage Problem

Suppose you have `n` men and `n` women and you need to create `n` couples to marry them all off so that each man and woman has exactly one opposite sex spouse. However, these marriages need to be 'stable' - that is, there is no man and woman who are *not* in a marriage that would rather be with each other than their current spouses (these pairs of men and women are called "rogue couples").

To measure each person's preference for each potential partner, let's assume that each man and woman has a ranked preference list of all people of the opposite gender.

Let's look at some notation which might make this setup a little clearer:

~~~ruby
men = {
  1 => [1,2,3,4],
  2 => [3,2,1,4],
  3 => [4,3,2,1],
  4 => [2,1,4,3]
}

women = {
  1 => [2,1,3,4],
  2 => [4,2,1,3],
  3 => [1,4,3,2],
  4 => [3,2,4,1]
}
~~~

Okay, so each man and woman has an ordered list of the women and men they prefer, respectively. Example: Woman 2 prefers Man 4, then Man 2, then Man 1, then Man 3. Cool. So how can we decide on which couples should pair up and guarantee no rogue couples?

While this specific instance of the problem can easily be solved through trial and error, as the number of partipants gets larger, it becomes very difficult to obtain a stable matching. In 1962, David Gale and Lloyd Shapley devised an algorithm to programatically solve this problem. Their algorithm follows the following steps, which I've implemented in pseudocode (for the purpose of illustration, because an actual implementation wouldn't be as readable).


~~~
while any man is unmarried

  m = an unmarried man

  while m is unmarried

    w = most preferred woman of m whom he has not yet proposed

    if w is unmarried

      w and m get married

    else

      if w prefers m to her current husband

        w breaks up with her current husband and marries m
        #(as a result, the old husband is now unmarried)

      end
    end
  end
end
~~~

Seems straighforward enough right?

A very basic proof for this algorithm is as follows: If there is an unmarried man, then there must be an unmarried woman to whom he has not yet proposed since there are both `n` men and women. Additionally, once the algorithm is complete, each man has married his most preferred woman who was never proposed to by a man she prefers more, so there cannot be a rogue couple. (This proof was too simplified to warrant a QED, but belive me, this algorithm works.)

It's interesting to note that this is only one algorithm for finding stable marriages. There are more, but let's delve into how the Gale-Shapley algorithm for finding stable marriages affects the men and women separately. If we look at this algorithm, the men start with their most preferred woman, and then continue down their lists until they are no longer rejected. Seems harsh right? Poor guys. But wait... what about the women? They just marry whoever comes along and only can "upgrade" to a more preferred husband if that man is rejected enough to reach her in his preference list.

In this way, the Gale-Shapley algorithm has been proven to be *male-optimal*; that is, men are married to their *most*-preferred woman who keeps the marriages stable, and the women are married to their *least*-preferred man that still keeps the marriages stable.

Woah! Wait, what? Did Gale and Shapley create a *gender-biased* algorithm? Since when can algorithms have bias?!

![GRACE HOPPER](http://i.imgur.com/kxnvsD4.jpg)

(Grace is not pleased with this.)

It's interesting to note that if we simply switched the algorithm based on who proposes and who acceps proposals, the algorithm is immediately and clearly *female-optimal*. That's crazy!

While this is a sort of contrived and antiquated example, it's important as a programmer (and a human) to understand the impact of our work and try to see how the applications we create may have adverse affects on our world. When I first learned about the stable matching problem and the Gale-Shapley algorithm, I was astonished to realize how the way we implement programs have these adverse effects in the real world. While yeah, two old white straight men wrote this algorithm 50 years ago, these types of biases still exist in our modern world and it's important to actively combat against these  flaws in our work and programs. (Yet another reason why diversity in tech is so important!)

Facebook's News Feed is an interesting ethical example of how algorithms affect us in the real world - [scientists have already experimented with how the implementation of the news feed algorithm affect's users' happiness](http://www.slate.com/articles/health_and_science/science/2014/06/facebook_unethical_experiment_it_made_news_feeds_happier_or_sadder_to_manipulate.html). Moving forward here at Flatiron, it's important to keep these sorts of things in mind while thinking about developing applications and putting our skills to use.
