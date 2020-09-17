---
layout: post
title:  "How to Kemeny"
date:   2020-09-17 15:00:00 +0200
---
Currently, I am working on the revision of [StationRank], a paper we wrote with [Vahid Moosavi]. In the words of one of the academic referees: *"The paper presents a Markov Chain (MC) framework to analyse daily aggregated itineraries of the swiss railway systems. They use their MC framework to asses the congestion, resilience and fragility of the railway network"*. In the preprint we also discuss the relevance of a somewhat obscure notion: the Kemeny's constant.

# Preliminaries and assumptions
For this brief analysis, I assume an ergodic MC of discrete time, with a row-stochastic transition probability matrix <img src="https://render.githubusercontent.com/render/math?math=\mathbb{P}">. By solving <img src="https://render.githubusercontent.com/render/math?math=\pi^T\mathbb{P}=\pi^T">, we obtain the stationary distribution vector <img src="https://render.githubusercontent.com/render/math?math=\pi^T"> and its respective eigenvalues <img src="https://render.githubusercontent.com/render/math?math=\lambda_1 = 1,\lambda_2,\dots,\lambda_n">.
Then it is straightforward to calculate the Kemeny constant <img src="https://render.githubusercontent.com/render/math?math=K=\displaystyle\sum_{j=2}^{n} \frac{1}{1-\lambda_j}">. This formulation is simple, but difficult to interprete as it just emphasizes the correlation to the eigenvalues. Another formulation in terms of mean first passage time <img src="https://render.githubusercontent.com/render/math?math=m_i{}_j"> comes to the rescue<sup>[1](#Kirkland)</sup>.

#### Density distribution histograms of <img src="https://render.githubusercontent.com/render/math?math=m_j{}_i"> for every swiss station
![hist](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/remotehist.png?raw=true)

<div style="page-break-after: always;"></div>

# Mean First Passage Time
Mean first passage time is the average expected number of steps from the origin state <img src="https://render.githubusercontent.com/render/math?math=S_i">
to a destination state <img src="https://render.githubusercontent.com/render/math?math=S_j">,
randomly selected according to the stationary distribution. It has been shown that the Kemeny constant is independant from the choice of the initial state.   

{% highlight python %}
import math
import time
import numpy as np
from scipy import linalg as la
{% endhighlight %}

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
    #Calculate U^-1 only once
    U_inv = la.inv(U)
    V = U_inv @ A @ U
    Z = np.zeros((n, n))
    if k1 != 0:
        Z[:k1, :k1] = la.inv(V[:k1, :k1])        
    return U @ Z @ U_inv
{% endhighlight %}

{% highlight python %}
# Calculate the Drazin inverse
I = np.identity(P.shape[0])
Q = drazin_inverse((I - P), tol=1e-4)
{% endhighlight %}
{% highlight python %}
# Calculate Mean First Passage Time
n = len(indices[k])
MFPT = np.zeros(shape=(n, n))
for i, row in (enumerate(Q)):
    for j, _ in enumerate(row):
        if j != i:
	    m = (Q[j][j] - Q[i][j]) / APs[k][0][j]
	    MFPT[i][j] = m
{% endhighlight %}

#### Accesibility analysis (in terms of remoteness) of the swiss network
![remote](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/remoteH.png?raw=true)

<a name="Kirkland">1</a>: E. Crisostomi, S. Kirkland & R. Shorten (2011) A Google-like model of road network dynamics and its application to regulation and control, International Journal of Control, 84:3, 633-651, DOI: [10.1080/00207179.2011.568005]

[10.1080/00207179.2011.568005]: https://www.tandfonline.com/doi/abs/10.1080/00207179.2011.568005
[StationRank]: https://arxiv.org/abs/2006.02781
[Vahid Moosavi]: https://www.vahidmoosavi.me/
[OS]: https://www.ordnancesurvey.co.uk/
