---
layout: post
title:  "Self Organizing Maps"
date:   2020-09-16 16:00:00 +0200
---
Here I would like to share my experience in a real production pipeline with a very interesting data reduction algorithm: [Self Organizing Maps (SOM)][SOM] introduced by Professor Teuvo Kohonen. It is a simple but powerful example of unsupervised machine learning. There are several implementations available, but the most prominent is [SOMPY].   

{% highlight python %}
import sompylib.sompy as SOM
{% endhighlight %}

#### 14 million UK residential buildings indexed with SOM
![som](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/som2.png?raw=true)

# The Data
The data comes from [Ordinance Survey (OS)][OS] and contains all building geometries in the UK.

#### Hitmap of neural activations of the best matching units (BMU)

![hitmap](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/hitmap.png?raw=true)


[SOM]: https://en.wikipedia.org/wiki/Self-organizing_map
[OS]: https://www.ordnancesurvey.co.uk/
[SOMPY]: https://github.com/sevamoo/SOMPY
