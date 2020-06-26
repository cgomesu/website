---
title: "The math of a bruised souleater"
date: 2019-07-19
tags: mtg math
header:
  overlay_image: "/assets/posts/2019-07-19-Math-bruised-souleater/header.jpg"
---

This is a repost of my [original reddit post](https://www.reddit.com/r/magicTCG/comments/7phqk1/the_math_of_a_bruised_souleater/). I've made a few changes to the original version to improve readability.
{: .notice .notice--info}

This is a post about an interesting interaction that came up yesterday, when I was making changes to an EDH deck. I thought someone else might find this analysis interesting, so I decided to share it here. If you spot an error, feel free to point it out. The interaction is about two cards, namely **Immolating Souleater** and **Bruse Tarl, Boorish Herder**.

[![Immolating souleater](/assets/posts/2019-07-19-Math-bruised-souleater/immolating-souleater.jpg){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/immolating-souleater.jpg)

[![Bruse tarl](/assets/posts/2019-07-19-Math-bruised-souleater/bruse-tarl.jpg){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/bruse-tarl.jpg)

Suppose you have Immolating Souleater on the battlefield and Bruse Tarl, Boorish Herder's triggered ability give it double-strike and lifelink. Souleater is a 1/1 creature that costs 2 colorless and let's you pay one phyrexian red mana to give it +1/+0 until eot.  Let's assume you cannot pay red mana and if you want to activate Souleater's ability, you need to pay life. The souleater is not under summoning sickness, so it can attack an opponent. For now, let's also assume you have 40 life and your opponents also have 40 life. You see an opportunity to kill an opponent, declare attack, the souleater is not blocked, you pay 38 life to make it a 20/1 creature with double-strike and lifelink, and after all damage is done, your opponent dies and you gain 40 life back (for a +2 net life, as you end up with 42 in the end).

This is cool and all but what happens when you cannot pay 38 life before the first hit? What if your opponent has more life than you? Life-wise, does it matter how much life you pay before the first hit? How much life can you pay at any given point to either maximize damage or minimize net life loss? Let's think about it.

Souleater's activated ability is a simple linear function of the amount of life you pay. Let x be the amount of life paid, so that souleater's power is given by

[![Eq 01](/assets/posts/2019-07-19-Math-bruised-souleater/eq1.png){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/eq1.png)

(To make it simple, we ignore the fact that phyrexian mana is always 2 life and let x be any non-negative integer.) As in the previous example, if you pay 38 life, f(38) = 20. The tricky part is how this simple linear function interacts with Bruse's ability, that is, the ability to give a creature double-strike and lifelink ("bruised").

The intuition with double-strike is to multiply f(x) by 2, which would be correct if we only paid life before the first hit (x1) and nothing else before the second hit (x2 = 0) but that does not need to be the case. Say we pay 20 life before the first hit (x1 = 20) and an additional 10 life afterwards (x2 = 10). How much damage will we deal to the opponent? We know that f(20) = 11 and f(10) = 6 but the first hit should carry over to the second becasue the +1/+0 lasts until eot. At this point, let's make an equation for the first hit,

[![Eq 02](/assets/posts/2019-07-19-Math-bruised-souleater/eq2.png){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/eq2.png)

and another for the second hit,

[![Eq 03](/assets/posts/2019-07-19-Math-bruised-souleater/eq3.png){: .PostImage .PostImage--large}](/assets/posts/2019-07-19-Math-bruised-souleater/eq3.png)

Now, we can sum both functions to get the total amount of damage dealt (d) by a Souleater with double-strike:

[![Eq 04](/assets/posts/2019-07-19-Math-bruised-souleater/eq4.png){: .PostImage  .PostImage--large}](/assets/posts/2019-07-19-Math-bruised-souleater/eq4.png)

(It's easy to see how one could generalize this to triple-strike, quadruple-strike, ..., but let’s not delve into it now.) Therefore, if x1=20 and x2=10, d(20,10) = f1(20)+f2(20,10), which is 27 damage for 30 life paid.

So, now we have an equation (d) that we can play with. For example, if an opponent has 60 life, how much life do you need to pay? Because x1 is multiplied by two, if the goal is to deal as much damage as possible, you should always pay as much life as possible before the first hit. The easiest way here is to pay 58 life right away (x1 = 58 and x2 = 0) but what if you only have 40 life. Our dmg equation shows that if you have 40 life, you can pay 38 before the first hit (x1=38) but in order to kill your opp, you’d have to pay 40 before the second hit (x2=40), which we can already tell that won’t be possible with the other parameters we set for this scenario. The question is then how high can x2 be in order to deal the highest amount of damage without killing ourselves? Now we need to figure out how to add lifelink into the equation.

Lifelink will affect your life total (k) and because a souleater with double-strike hits twice, let’s define your starting life total (k1), our life total after the first hit (k2), and our life total after the second hit (k3). As before, let’s say that we pay 20 life before the first hit (x1 = 20) and an additional 10 life afterwards (x2 = 10). Because our life total matters here, let’s also assume that our starting life total is k1 = 40. So, if x1 = 20, our remaining life before the first hit is the difference k1 – x1 = 20 and immediately after the first hit, the lifelink effect makes it go up by f1(20) = 11 points, which means that we can define

[![Eq 05](/assets/posts/2019-07-19-Math-bruised-souleater/eq5.png){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/eq5.png)

and similarly,

[![Eq 06](/assets/posts/2019-07-19-Math-bruised-souleater/eq6.png){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/eq6.png)

Now we have a simple way of defining net-life gain/loss (nl) as the difference between k3 and k1, namely

[![Eq 07](/assets/posts/2019-07-19-Math-bruised-souleater/eq7.png){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/eq7.png)

which funny enough doesn’t include x1 into the equation. The equation for nl tells us three main things: (a) that we will never net more than 2 life, (b) we’ll net life loss proportional to the amount of life we pay before the second hit (x2), and (c) net-wise, it doesn’t really matter how much life you pay before the first hit (x1). (x1 doesn’t matter because in our scenario, it always nets 0 life. Think about it. You are paying 2 life to increase 1 dmg but with double-strike, souleater hits twice and in the end, you get your 2 life back with lifelink. The only thing that carries over is the base power, which is why there’s a constant 2 in the equation for k3.)
 
Good. Now, what can we do with that? Well, we can figure out how to maximize dmg in terms of our life total, so that you deal as much damage as possible without dying for any given amount of starting life total (the math equivalent of “just pay as much life as you can before each hit”). For k1 > 0, let i1 and i2 be the amount of life you want to have before the first hit and second hit, respectively, then

[![Eq 08](/assets/posts/2019-07-19-Math-bruised-souleater/eq8.png){: .PostImage .PostImage--large}](/assets/posts/2019-07-19-Math-bruised-souleater/eq8.png)

(There’s an additional complication here because of how we use phyrexian mana but we don’t need to worry too much about it as long as we can figure out the appropriate i1 and i2 for each value of k1. For example, if you have 40 life, max(d) = 2.5 + 1.25(40) - .75(2) - .5(2) = 50. Because k1 is multiplied by a value greater than one, this equation tells us that the amount of damage done above your starting life increases with our starting life. For example, if you have 40 life, you can deal 10 damage above your starting life to an opponent (e.g., kill an opponent with 50 life points) but if you have 20 life, you can only deal 5 damage above your starting life (e.g., kill an opponent with 25 life points). It’s easier to see this relationship when we plot max(d) as a function of the starting life (k1) using the minimum appropriate values of i1 and i2:

[![Regression](/assets/posts/2019-07-19-Math-bruised-souleater/regression.jpg){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/regression.jpg)

If we fit a linear equation to it, we get a solution that does not require knowing the exact values of i1 and i2, namely max(d) ≈ .65 + 1.25k1.  However, because each point is equally likely, as /u/darth_aardvark pointed out, the appropriate intercept should be the mean of all possible combinations of min(i1) and min(i2), which can only take on values {1, 2}.  There is a total of 2^2 combinations of  min(i1) and min(i2), namely

[![Eq 09](/assets/posts/2019-07-19-Math-bruised-souleater/eq9.png){: .PostImage .PostImage--large}](/assets/posts/2019-07-19-Math-bruised-souleater/eq9.png)

Those equations give us a mean intercept = .625 and a revised and final formula for the maximum amount of damage that a "bruised" souleater can do, specifically 

[![Eq 10](/assets/posts/2019-07-19-Math-bruised-souleater/eq10.png){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/eq10.png)

which is equivalent to the more memorable formula

[![Eq 11](/assets/posts/2019-07-19-Math-bruised-souleater/eq11.png){: .PostImage}](/assets/posts/2019-07-19-Math-bruised-souleater/eq11.png)

in which k1 is simply your life total before the attack.  The 5/4 coefficient is quite instructive because it connects this approach to /u/darth_aardvark's.  Specifically, the first maximized damage of a bruised souleater will hit for approximately half our life total (k1/2), while the second hit will hit for half our life total plus half of the life gained from the second hit (k1/2 + k1/4), which gives us a total of 5/4(k1).

Well, this has been fun. I’m sure there are other things we could play with here but I’ve had enough for now. I think there are two main conclusions about a “bruised” Souleater. The first and least obvious one imo is that the more life you have, the more damage you can deal above your own life total. This is useful because it gives an intuition about whether you can kill an opponent or not at any given point (low life = cannot deal much above my own life; high life = can deal a bit more than my own life), and whether you should attempt to compute exacties. In most cases, however, you won’t be able to deal a whole lot more than your own life. The other conclusion is that the amount of life you pay before the first hit and second hit is net neutral and net negative, respectively, but I feel most of us would be able to figure this one out without much effort (lifelink gets only half the amount of life paid immediately before the second hit, while it gets it all back for the amount of life paid immediately before the first hit).

Again, if you spot an error, please point it out. If you found a better way to look at this interaction or thought about a different scenario, feel free to explain it. 

[top](#){: .btn .btn--small .btn--light-outline}
