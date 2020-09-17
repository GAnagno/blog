---
layout: post
title:  "Self Organizing Maps"
date:   2020-09-16 16:00:00 +0200
---
Here I would like to share my experience in a real production pipeline with a very interesting data reduction algorithm: [Self Organizing Maps (SOM)][SOM] introduced by Professor Teuvo Kohonen. It is a simple but powerful example of unsupervised machine learning. There are several implementations available, but the most prominent is [SOMPY].   

{% highlight python %}
import sompylib.sompy as SOM
{% endhighlight %}

#### 14 million UK building geometries indexed with SOM
![som](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/som2.png?raw=true)

# Motivation
A major design requirement was to provide a common geographic reference for record linkage of different data sets related to building portfolios. You might want to think this as a data-driven postcode. Real postcodes greatly vary in terms of population density and aren't well suited for scalable data matching pipelines. Why not just use a nearest neighbor approach? The advantage of a many-to-many indexed comparison, performed on the level of artificial postcodes, is that it provides qualitative confidence measures, as in [Python Record Linkage Toolkit]. In contrast, a simple haversine distance is difficult to interpret because an absolute tolerance of 10 meters might be "good enough" in rural areas, but "poor" in densely populated urban areas like Soho.

# Data source
The data comes from [Ordinance Survey (OS)][OS] and contains all building geometries in the UK.
{% highlight python %}
X = selected_df[["Latitude", "Longitude"]].values
{% endhighlight %}

# Training the model
SOM is organized in a 2-D grid (more dimensions are theoretically possible) of fully connected neurons called Best Matching Units. After initialization and training, similar data points get assigned to the same BMUs and similar BMUs have topological proximity which is different to other approaches like [KNN]. When the data is in the range of millions, a sample of <img src="https://render.githubusercontent.com/render/math?math=1e5"> would suffice for training.

{% highlight python %}
# Randomly choose 100000 coordinates for fast training
size = int(1e5)
training = X[np.random.choice(X.shape[0], size, replace=False)]
{% endhighlight %}

# How big?
An important decision is the size of the model, or number of neurons <img src="https://render.githubusercontent.com/render/math?math=M">. Our goal is to achieve maximum utilization of the provided gird. Manual tuning of the hyperparameters can be time-consuming and lead to poor results or slow training. For this kind of simple model there exist some not widely known [heuristics] that satisfactorily answer the question. The answer is <img src="https://render.githubusercontent.com/render/math?math=M \approx 5 sqrt(N)">, where <img src="https://render.githubusercontent.com/render/math?math=N"> is the total size of the projected data. The final grid should then be <img src="https://render.githubusercontent.com/render/math?math=sqrt(M) \times sqrt(M)">.

{% highlight python %}
if os.path.exists(som_path):
    # Load SOM from disk
    som2 = SOM.load(filename=som_path)
else:
    # Train SOM from scratch
    msz11 = 136
    msz10 = 136

    som2 = SOM.SOM("",
                   training,
                   mapsize=[msz10, msz11],
                   norm_method="var",
                   initmethod="pca")
    som2.init_map()
    som2.train(n_job=8, shared_memory="no", verbose="final")
    som2.compname = [selected_df.columns[1:3]]
    # Save SOM to disk
    SOM.dump(som2, filename=som_path)
{% endhighlight %}

# Result
So finally we have a trained SOM that effectively acts as a density-aware smart grid of artificial postcodes. When projecting coordinates from different sources, best matching units are assigned and full indexing, with certain confidence, is performed locally in a scalable fashion.

{% highlight python %}
print("Calculating best matching units...")
bmusX = som2.project_data(X)
selected_df["bmu"] = bmusX
print("Done!")
{% endhighlight %}


#### Hitmap of neural activations of the best matching units (BMU)

![hitmap](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/hitmap.png?raw=true)

# A word of advice
Caution! SOM is extremely sensitive to duplicates, a problem commonly referred to as imbalanced data. Especially when the dimensionality is low, this can be very detrimental to the model, so make sure to drop any duplicates before training.

[SOM]: https://en.wikipedia.org/wiki/Self-organizing_map
[OS]: https://www.ordnancesurvey.co.uk/
[SOMPY]: https://github.com/sevamoo/SOMPY
[Python Record Linkage Toolkit]: https://recordlinkage.readthedocs.io/en/latest/ref-index.html
[KNN]: https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm
[heuristics]: https://www.researchgate.net/post/How_many_nodes_for_self-organizing_maps
