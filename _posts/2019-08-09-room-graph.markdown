---
layout: post
title:  "Room Graph"
date:   2019-08-09 22:00:00 +0200
---
During the past few years, I have been responsible for the `room-list` and the `door-list` in the planning of a large biotechnology facility. It suddenly occurred to me that these two things are actually one and can be jointly represented with a `graph`.

For obvious confidentiality reasons and also for the sake of clarity, I will not publish that. Although free access to structured architectural data, aka BIM, is very restricted, there are some exceptions. BIMobject®, a digital platform for the construction industry based in Malmö, Denmark, is one of them. They actually published the model of their own headquarters designed by SHL Architects as a demonstration of best practice.

# Studio Malmö by SHL
![SHL Architects](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/Architects.jpg?raw=true)

{% highlight python %}
# https://www.bimobject.com/de-ch/bimobjectmodels/product/bimobject_hq
img = mpimg.imread('data/layout.jpg')
fig = plt.figure(figsize = (30,10))
ax = fig.add_subplot(1, 1, 1)
ax.imshow(img, interpolation='bilinear')
ax.axis('off')
ax.set_title('BIMobject HQ', fontsize=16);
{% endhighlight %}

![BIMobject HQ](https://github.com/GAnagno/myblog/blob/gh-pages/assets/images/HQ.png?raw=true)

# Networked Rooms
The idea, which is frankly not new but also not common practice in the industry, is to represent rooms and doors as nodes and edges of a `network`. Room properties such as area and perimeter become `node_attributes` and door properties such as width and height become `edge_weights`.

# Get structured data

{% highlight python %}
# Load rooms 
rooms = pd.read_csv('data/room_schedule.csv', skiprows=[2], keep_default_na=False)
rooms.columns = rooms.iloc[0]
rooms.drop([0], inplace=True)
rooms['Area'] = rooms['Area'].map(lambda x: x.rstrip(' m²'))
{% endhighlight %}

| Number | Name               | Area  | Perimeter |
| ------ | ------------------ | ----- | --------- |
| 1      | CTO Office         | 9.52  |	12794     |
| 2      | Legal Eagle Office | 9.51  |	12786     |
| 3      | PA Office          | 9.51  |	12786     |
| 4      | CEO Office         | 21.62 |	12587     |
| 5      | HR Office          | 9.30  |	12565     |

{% highlight python %}
# Load doors 
doors = pd.read_csv('data/door_schedule.csv', skiprows=[2], keep_default_na=False)
doors.columns = doors.iloc[0]
doors.drop([0], inplace=True)
doors.drop(doors.tail(1).index, inplace=True)
{% endhighlight %}

| Mark   | From Room          | To Room | Width | Height |
| ------ | ------------------ | ------- | ----- | ------ |
| 632296 | CTO Office         | Open    | 1015  | 2102   |
| 642739 | Legal Eagle Office | Open    | 1015  | 2102   |
| 650548 | PA Office          | Open    | 1015  | 2102   |
| 196335 | CEO Office         | Open    | 1015  | 2102   |
| 654502 | CFO Office         | Open    | 1015  | 2102   |

Check out the [Jupyter notebook][notebook] for the full code.

[notebook]: https://github.com/GAnagno/Social-Web/blob/master/Room%20Graph.ipynb
