---
title: "Victorian-Era Trade Optimiser Algorithm"
date: 2025-03-24
---

This is a follow-up explaining how my trade optimiser algorithm written in my [first hackathon]({/posts/first-hackathon/) works... roughly.

> Given some amount of countries with a certain amount of resources (positive is a surplus, negative is a defecit), what is the optimal trade that each country should be doing (e.g: trading 1.5 wood and 0.1 steel for 0.9 coal) for each other country to minimise their defecit using their surplus.

**This is currently a placeholder**.

<!--
Essentially, rather than trying to solve for an optimal strategy outright immediately like I was attempting to with my game theory approach, I instead generate $N$ traders for every country that the source country is trading to, all with really stupid initial trades.

A 'trade' is just an array representing the percentage amount of each resource that it will try to buy from the seller, $(s_1, s_2, \cdots, s_n)$. Each trader is assumed to be an 'average' trader of the country, so their 'inventory' is equal to their origin countries resources $(r_1, r_2, \cdots, r_n)$.

First we then need a way to calculate the 'percieved value' of a resource. In my model, I base this off of the defecit to surplus ratio of that resource. That is, if trader $A$ is trading with country $B$, $B$ places a low value on items that have a high surplus (if $B$ has 1000 iron, but only 10 wood, it would value wood over iron, naturally).

If we just directly base it off of defecits however, we run into a problem if a country has no resource in defecit, which would apparently mean it wouldn't trade at all since the ratio would never be negative. To fix this, I 'normalize' all resources of a country by subtracting their mean (we are assuming a zero-sum game here), so if a country is inputted by the user to have resources $(r_1, r_2, \cdots, r_n)$, then the internal resources used to calculate percieved value in the algorithm are actually,

$$
\bar{r} = \frac{r_1 + r_2 + \cdots + r_n}{n}, \ \left( r_1 - \bar{r}, r_2 - \bar{r}, \cdots, r_n - \bar{r} \right)
$$

This ensures that there is *at least one* resource that is in an 'internal defecit'. What this means in practice is that even countries with visibly no defecit will still try to use the resources they have more of to stabilise the resources that they have less of. Economics!

Moving on, since either side views the trade as being of equal value, we can derive the following for a trader $A$ making a deal with country $B$,

$$
\sum_{k=1}^{n}{P_B(k) \cdot t_k} = \sum_{k=1}^{n}{P_A(k) \cdot s_k}
$$

Where $P_X(k)$ is the percieved value function of resource $k$ by $X$, and $t_k$ is the percentage of $A$'s goods it's going to sell to $B$ to get back what it wants (similar to $s_k$ but in reverse),

We can then solve for $t_k$,

$$
t_k = \frac{P_A(k)}{P_B(k)} \cdot s_k
$$

So the discrete amount of goods that the trader is going to lose, $T_k$, and gain, $S_k$, are,

$$
\begin{cases}
T_k &= t_k r_k = s_k r_k \cdot \frac{P_A(k)}{P_B(k)} \\
S_k &= s_k r_k
\end{cases}
$$

Remember that there is an artificial limit that $T_k$ cannot be greater than $r_k$, since the trader can't trade more than he currently has, so we have to clamp it to be equal to or less than $r_k$.

So the amount of goods the trader is going to have by the end of the transaction, $r_k'$ is,

$$
r_k' = T_k - S_k = s_k r_k \cdot \frac{P_A(k)}{P_B(k)} - s_k r_k = \left[ s_k \left( \frac{P_A(k)}{P_B(k)} - 1 \right) \right] \cdot r_k
$$

For instance, one trader could have a randomly generated trade of 100 $r_1$, and nothing else. Then, they do this trade with their destination country and get back 1200 $r_2$. The country decides, like explained above, that 1 of the traders $r_1$ is worth 12 of its $r_2$, but the trader doesn't know this until the trade happens.

The traders 'output inventory' is then calculated - their inventory after the trade happens, using the equation shown above. From this output inventory, we then calculate it's percieved worth, $W$, to the country the trader came from based on the percieved value of each resource before the trade occurred. We do this for all of the traders. I'm going to use $P'(k)$ to denote the new percieved value of a resource $k$ and $P(k)$ to denote the old percieved value of resource $k$,

$$
W = \sum_1^n{r_k' \cdot P(k)}
$$

Out of those traders, we pick the one that had the most percieved worth. From this one, we generate $N$ children that all have the same trade percentages as them, but with small *randomly generated offsets*.

Then, we go around, sending $N$ the new improved traders out to each destination country again. And again, and again, and again. Each time finding the trader that generates the most profit possible. Eventually, we're going to settle on some maximum profit 'perfect trader' from which we can't get any better.

I believe that this does have the downside that we could land on a local maximum, rather than a global maximum - i.e: the traders could get stuck in a trade that is still really good, but not the potentially best (highest profit) trade ever.

As far as I know, this algorithm doesn't exist anywhere else, but I might be wrong. Let me know if something similar does exist.

It's been a lot of fun figuring this all out, and while I might not have got to complete it on time before we moved onto something else for the hackathon, I'm happy I figured it out later on regardless :).
-->
