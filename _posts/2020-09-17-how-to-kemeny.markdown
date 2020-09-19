---
layout: post
title:  "How to Kemeny"
date:   2020-09-17 15:00:00 +0200
---
Currently, I am working on the revision of [StationRank], a paper we wrote with [Vahid Moosavi]. In the words of one of the academic referees: *"The paper presents a Markov Chain (MC) framework to analyse daily aggregated itineraries of the swiss railway systems. They use their MC framework to asses the congestion, resilience and fragility of the railway network"*. In the preprint we also discuss the relevance of a somewhat obscure notion: the Kemeny's constant.

# Preliminaries and assumptions
{% highlight python %}
import math
import time
import numpy as np
from scipy import linalg as la
{% endhighlight %}
For this brief analysis, I assume an ergodic MC of discrete time in first order, with a row-stochastic transition probability matrix <img src="https://render.githubusercontent.com/render/math?math=\mathbb{P}">. By solving <img src="https://render.githubusercontent.com/render/math?math=\pi^T\mathbb{P}=\pi^T">, we obtain the stationary distribution vector <img src="https://render.githubusercontent.com/render/math?math=\pi^T"> and the eigenvalues <img src="https://render.githubusercontent.com/render/math?math=\lambda_1 = 1,\lambda_2,\dots,\lambda_n">.
Having the eigenvalues, we directly obtain the Kemeny constant <img src="https://render.githubusercontent.com/render/math?math=K=\displaystyle\sum_{j=2}^{n} \frac{1}{1-\lambda_j}"><sup>[1](#Kirkland)</sup>. This is very straightforward to calculate, but difficult to interpret. I would like to focus on another formulation in terms of mean first passage time <img src="https://render.githubusercontent.com/render/math?math=m_i{}_j"> which grants us more insight to the inner workings of <img src="https://render.githubusercontent.com/render/math?math=K">.

# Mean First Passage Time
Mean first passage time is the expected number of steps from an origin state <img src="https://render.githubusercontent.com/render/math?math=S_i"> 
to a destination state <img src="https://render.githubusercontent.com/render/math?math=S_j">. The  equation
<img src="https://render.githubusercontent.com/render/math?math=K=\displaystyle\sum_{j=1}^{n} m_i{}_j \pi_j"> is yet another, more intuitive way to calculate the Kemeny constant as the average mean first passage time from any origin state to any other destination state chosen according to the probability <img src="https://render.githubusercontent.com/render/math?math=\pi_j"> and shows that <img src="https://render.githubusercontent.com/render/math?math=K"> is independent from the choice of the initial state
<img src="https://render.githubusercontent.com/render/math?math=S_i">.    

{% highlight python %}
# Calculate the Drazin inverse
I = np.identity(P.shape[0])
Q = drazin_inverse((I - P), tol=1e-4)
{% endhighlight %}

Obviously now, in addition to <img src="https://render.githubusercontent.com/render/math?math=\pi^T">, we also need to calculate mean first passage time, which in turn requires the calculation of the group (special case of Drazin) inverse of <img src="https://render.githubusercontent.com/render/math?math=\Q=(\I-\mathbb{P})">.

<div style="page-break-after: always;"></div>

A matrix <img src="https://render.githubusercontent.com/render/math?math=\G">is the group inverse of <img src="https://render.githubusercontent.com/render/math?math=\Q,">if and only if:
- <img src="https://render.githubusercontent.com/render/math?math=\Q \times \G \times \Q= \Q">
- <img src="https://render.githubusercontent.com/render/math?math=\G \times \Q \times \G= \G">
- <img src="https://render.githubusercontent.com/render/math?math=\Q \times \G = \G \times \Q">

Interpretability often comes at a cost. 
{% highlight python %}
def drazin_inverse(A, tol=1e-4, verbose='on'):
    """Compute the Drazin inverse of A.

    Parameters:
        A ((n,n) ndarray): An nxn matrix.

    Returns:
       ((n,n) ndarray) The Drazin inverse of A.
    """
    e1 = time.time()
    n = A.shape[0]
    f = lambda x: abs(x) > tol
    g = lambda x: abs(x) <= tol
    Q1, S, k1 = la.schur(A, sort=f)
    Q2, T, k2 = la.schur(A, sort=g)
    U = np.hstack((S[:, :k1], T[:, :n - k1]))
    U_inv = la.inv(U)
    V = U_inv @ A @ U
    Z = np.zeros((n, n))
    if k1 != 0:
        Z[:k1, :k1] = la.inv(V[:k1, :k1])        
    return U @ Z @ U_inv
{% endhighlight %}

The mean first passage time can be calculated as follows:
<img src="https://render.githubusercontent.com/render/math?math=\m_i{}_j = \frac{g_j{}_j - g_i{}_j}{\pi_j}">.

{% highlight python %}
# Calculate Mean First Passage Time
MFPT = np.zeros(shape=(n, n))
for i, row in (enumerate(G)):
    for j, _ in enumerate(row):
        if j != i:
	    m = (G[j][j] - G[i][j]) / pi[j]
	    MFPT[i][j] = m
{% endhighlight %}

<div style="page-break-after: always;"></div>

# Takeaway
If instead of <img src="https://render.githubusercontent.com/render/math?math=\m_i{}_j">, which by definition is expected to have very similar distribution for different initial stations, we have a look at the mean 
<img src="https://render.githubusercontent.com/render/math?math=\m_j{}_i">, then we come up with a kind of accessibility analysis in terms of "remoteness" of train stations. This can be easily verified on a map. 

![hist](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/remoteness_h.png?raw=true)
![remote](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/remoteness.png?raw=true)
*Geographic distribution of mean <img src="https://render.githubusercontent.com/render/math?math=m_j{}_i">, a measure of "remoteness"*

<a name="Kirkland">1</a>: E. Crisostomi, S. Kirkland & R. Shorten (2011) A Google-like model of road network dynamics and its application to regulation and control, International Journal of Control, 84:3, 633-651, DOI: [10.1080/00207179.2011.568005]
<div style="page-break-after: always;"></div>
[10.1080/00207179.2011.568005]: https://www.tandfonline.com/doi/abs/10.1080/00207179.2011.568005
[StationRank]: https://arxiv.org/abs/2006.02781
[Vahid Moosavi]: https://www.vahidmoosavi.me/
[OS]: https://www.ordnancesurvey.co.uk/
