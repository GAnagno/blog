---
layout: post
title:  "Self Organizing Maps"
date:   2020-09-16 16:00:00 +0200
---
Here I would like to share my practical experience with a very interesting algorithm: Self Organizing Maps (SOM).  

#### Basel SBB
![Basel SBB](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/som2.png?raw=true)

{% highlight python %}
%matplotlib inline
import os
import numpy as np
import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt
from matplotlib import cm
{% endhighlight %}

# Import Data
The original dataset contains information about the yearly total number of trains that pass through each route section of Switzerland. The list contains entries for 2016, 2017 and 2018. Distinction is also made between passenger transport and cargo. For this post, I will focus only on passenger transport in 2018. Comparative studies along the aforementioned two axes are reserved for the near future.

{% highlight python %}
df = pd.read_csv('data/zugzahlen.csv')
{% endhighlight %}

| PID          | Anzahl_Zuege |
| ------------ | ------------ |
| SBB_GESE_CHY | 126307       |
| SBB_GIBU_ROL | 97943        |
| SBB_MIES_TAN | 126101       |
| SBB_MOR_STJ  | 121566       |
| SBB_PER_ALL  | 98005        |

# Preprocessing
As is often the case with transportation networks, there are origin and destination points. Caution :skull:! These two sets of points do not have to be congruent and in this case they aren't. Let's extract the respective longitudes and latitudes and have a look.

#### Longitude of origin

{% highlight python %}
df['lon_von'] = df.geopos_von.str.split('\,').str[0]
df['lon_von'] = df['lon_von'].map(float)
{% endhighlight %}

#### Latitude of origin

{% highlight python %}
df['lat_von'] = df.geopos_von.str.split('\,').str[1]
df['lat_von'] = df['lat_von'].map(float)
{% endhighlight %}

#### Longitude of destination

{% highlight python %}
df['lon_bis'] = df.geopos_bis.str.split('\,').str[0]
df['lon_bis'] = df['lon_bis'].map(float)
{% endhighlight %}

#### Latitude of destination

{% highlight python %}
df['lat_bis'] = df.geopos_bis.str.split('\,').str[1]
df['lat_bis'] = df['lat_bis'].map(float)
{% endhighlight %}

#### Map of origins and destinations

Spot the difference :trollface:

![Map](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/hitmap.png?raw=true)

#### Coordinates of origins

{% highlight python %}
pos_von = {}
for i in range(0, len(df)):
    pos_von[df.BP_Von_Abschnitt[i]] = (df.lon_von[i], df.lat_von[i])
{% endhighlight %}

#### Coordinates of destinations

{% highlight python %}
pos_bis = {}
for i in range(0, len(df)):
    pos_bis[df.BP_Bis_Abschnitt[i]] = (df.lon_bis[i], df.lat_bis[i])
{% endhighlight %}

#### Final list of nodes

{% highlight python %}
# Magic merge of dictionaries
pos = {**pos_bis, **pos_von}
{% endhighlight %}

# Building the graph

The graph is directed, so I will initialize it as a `DiGraph` with weighted edges and proceed with a bidirectional copy.

{% highlight python %}
df['weight'] = df['Anzahl_Zuege'].map(float)
df['weight'] /= np.max(df['weight'])
{% endhighlight %}

{% highlight python %}
D = nx.from_pandas_edgelist(df, source = 'BP_Von_Abschnitt',
                            target = 'BP_Bis_Abschnitt',
                            edge_attr = 'weight',
                            create_using = nx.DiGraph())
{% endhighlight %}

{% highlight python %}
G = nx.Graph(D)
{% endhighlight %}

# Graph as a Map

Now the graph is fully operational and we can very easily do all kinds of fun calculations.

{% highlight python %}
bond = np.array(list(nx.get_edge_attributes(G,'weight').values()))
{% endhighlight %}

{% highlight python %}
# Calculate degree centrality,
eigenvector_centrality = nx.eigenvector_centrality(G)

# Set degree centrality metrics on each node,
nx.set_node_attributes(G, eigenvector_centrality, 'ec')

# Use eigenvector centrality for visualization.
ec = np.array(list(nx.get_node_attributes(G,'ec').values()))
{% endhighlight %}

{% highlight python %}
plt.figure(figsize = (12,9), dpi=150)
nx.draw(G, pos=pos, edge_color=bond, node_color=ec, with_labels=False,
        node_size=ec*1000, width=bond*20,
        edge_cmap=plt.cm.Spectral, cmap=plt.cm.Blues_r)
plt.savefig('SBB/map.jpg')
{% endhighlight %}

![SBB Map](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/SBBmap.png?raw=true)

# Graph as a Network
New positions based solely on network dynamics abstracting away geography.

{% highlight python %}
_pos = nx.spring_layout(G, seed=0)
{% endhighlight %}

{% highlight python %}
plt.figure(figsize = (12,12), dpi=150)
nx.draw(G, pos=_pos, edge_color=bond, node_color=ec, with_labels=True,
        font_size=5, weight='weight', node_size=ec*2500, width=bond*20,
        edge_cmap=plt.cm.Spectral, cmap=plt.cm.Blues_r, alpha=0.8)
plt.savefig('SBB/atom.jpg')
{% endhighlight %}

![SBB Net](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/SBBnet.png?raw=true)

Check out the [Jupyter notebook][notebook]!

P.S. The thick purple line in the core is `Langstrasse` with 336586 passenger trains in 2018.

[notebook]: https://github.com/GAnagno/Social-Web/blob/master/SBB.ipynb
[previous]: https://ganagno.github.io/myblog/2019/08/09/room-graph.html
