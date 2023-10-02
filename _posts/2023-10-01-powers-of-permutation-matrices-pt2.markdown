---
layout: post
title:  "Powers of a permutation matrix - Part 2"
author: Arun
tags: [mathematics, linearalgebra, permutations]
---

In [Part 1](https://arunvijayshankar.github.io/archive/powers-of-permutation-matrices-pt1/), we saw how every permutation has at least one cycle, and that the size of the cycle is the number of elements in the cycle. If we now consider a permutation that is one big cycle of all the elements in our set, then we can easily see that if we apply the same permutation *n* times, *n* being the number of elements in the set, all elements return to the positions with which they started out. So *p*<sup>*n*</sup> = *e* for such a permutation, where once again, *e* is the identity. 

But what about permutations that are not just one cycle of all elements? What if the permutation consists of a combinations of cycles of different cycles and/or fixed points (elements that remain fixed in their positions upon application of a permutation are called fixed points). To explore this further, let us consider a couple of specific examples.

<ol>
	<li> The permutation is one in which two elements in the set switch places, the rest are fixed points - In this case, there is one cycle of size two, and the rest are fixed points, so they are all in cycles of size one. So if we apply the permutation for a second time, both elements that switched places return to their original position, and the rest will have remained in the same position with which they started out. So if <em>p</em> denotes this particular permutation, then <em>p</em><sup>2</sup> = 2 </li>
	<li>The permutation consists of two cycles of size <em>k<sub>1</sub></em> and <em>k<sub>2</sub></em> respectively - Here, we immediately see that the first cycle with return to its starting arrangement after <em>k<sub>1</sub></em> applications of the permutation and the second cycle will return to its starting arrangement after <em>k<sub>2</sub></em> applications. To calculate how many applications of the permutation are needed for both cycles to return to the starting configuration, notice that either cycle will return to the start when the permutation is applied <em>mk</em> times, where <em>m</em> is a positive integer. So this means that if the permutation is applied <em>k<sub>1</sub>k<sub>2</sub></em> times, both cycles will have returned to their original positions (notice that if <em>k<sub>2</sub></em> = <em>qk<sub>1</sub></em> then both cycles will return to the starting positions after just <em>k<sub>2</sub></em> applications of the permutation). That is, <em>p<sup>k<sub>1</sub>k<sub>2</sub></sup></em> = <em>e</em></li> 
</ol>

Let's stop here for now, and in Part 3 let's explore permutations that are a little more complex, and finally try and nail down the answer for any generic permutation.  
