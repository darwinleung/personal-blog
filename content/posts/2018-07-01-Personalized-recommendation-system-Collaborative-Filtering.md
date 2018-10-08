---
title: "Personalized recommendation system for articles - Collaborative Filtering"
date: 2018-07-01T12:00:00-04:00
draft: false
tags: [ "python" , "sklearn", "knn", "recommendation", "collaborative filtering"]
categories: [ "machine learning", "data science", "recommedation system", "feature extraction"]
---

# Personalized recommendation system for articles - Collaborative Filtering

From Andrew Ng's course, I learned there are 2 popular ways to built recommendation system: Collaborative Filtering and Content-based. Recently, I have a chance to research and implement them at work. Here is a Proof-of-Concept example. 



Recommendation system is about recommending items to users. Let's say we only have:

- Items: 6 articles: ['dog1', 'dog2', 'cat', 'food1', 'food2', 'food3']. (Note: we label them just for easier explanation, it is not necessary to have any context about the items, they can be random ids)
- Users: 5 visitors ['user_0','user_1','user_2', 'user_3', 'user_4', 'user_5']



## Collaborative Filtering

> [Collaborative filtering](https://en.wikipedia.org/wiki/Collaborative_filtering) approaches build a model from a user's past behaviour (items previously purchased or selected and/or numerical ratings given to those items) as well as similar decisions made by other users. This model is then used to predict items (or ratings for items) that the user may have an interest in.  - [Wikipedia](https://en.wikipedia.org/wiki/Recommender_system)

Collaborative filtering only use previous user behavior data, in this example, we convert pageview event into binary representation, i.e. whether a user viewed a page or not.

```python
import numpy as np
import pandas as pd

is_click = np.array([[1,1,1,0,0,0],[1,1,0,0,0,0],[0,0,1,0,0,0],[0,0,0,1,1,0],[1,0,0,0,1,1]])
df = pd.DataFrame(is_click,columns = ['dog1', 'dog2', 'cat', 'food1', 'food2','food3'], \
                  index = ['user_0','user_1','user_2','user_3','user_4'] )
df
```

|        | dog1 | dog2 | cat  | food1 | food2 | food3 |
| ------ | ---- | ---- | ---- | ----- | ----- | ----- |
| user_0 | 1    | 1    | 1    | 0     | 0     | 0     |
| user_1 | 1    | 1    | 0    | 0     | 0     | 0     |
| user_2 | 0    | 0    | 1    | 0     | 0     | 0     |
| user_3 | 0    | 0    | 0    | 1     | 1     | 0     |
| user_4 | 1    | 0    | 0    | 0     | 1     | 1     |

In this example, we can see user_0 viewed 3 articles about animal and none about food, user_1 viewed all dog-related articles, user_2 only viewed cat's, user_3 loves food articles and user_4 read 2 food articles and 1 dog article. 

Alternatively, if we have more data about user behavior like their time on page, scroll depth etc. we can combine these data to assign a numeric score of engagement between a user and article. It should give a better recommendation.

## Item-based CF (non-personalized)

First, based on the above representation, we can show related articles given an article. E.g. Given a user viewing dog2, what article should we recommend to view next? I think of the user behavior data as a vector representation for an article, i.e. [1,1,0,0,1] for dog1, [1,1,0,0,0] as dog2. 

We can apply KNN to solve this problem. KNN stands for K-Nearest-Neighbors. What it does is very straight forward: Take a single vector as input and calculate the distance/ similarity with all the vectors in the space, return the K nearest neighbor. 

![KNN](http://www.harrisgeospatial.com/docs/html/images/FXSupervisedClassification/KNN_2d_graph_distances.gif)

I used sklearn's neighbors package, and I can define similarity in different ways, cosine distance is one of them, alternatives can be Euclidean/ Manhattan, etc.

![img](https://image.slidesharecdn.com/networkbiologylent2010fmlecture2-12676050637629-phpapp02/95/network-biology-lent-2010-lecture-1-22-728.jpg?cb=1267583822)

```python
from sklearn.neighbors import NearestNeighbors

df_transpose = df.transpose()

model_knn = NearestNeighbors(metric = 'cosine', algorithm = 'brute', n_neighbors = 6)
model_knn.fit(df_transpose.values)
```



After fitting KNN, we can input i (index of article and get a list of similar articles sort by distance):

```python
i = 1
distances, indices = model_knn.kneighbors(df_transpose.values[i].reshape(1,-1),n_neighbors = 6)

for j in range(0, len(distances.flatten())):
    if j == 0:
        print 'Recommendations for {0}:\n'.format(df_transpose.index[i])
    else:
        print '{0}: {1}, with distance of {2}'.format(j, df_transpose.index[indices.flatten()[j]], distances.flatten()[j])
```



```
> Recommendations for dog2:

1: dog1, with distance of 0.183503419072
2: cat, with distance of 0.5
3: food1, with distance of 1.0
4: food2, with distance of 1.0
5: food3, with distance of 1.0
```



As expected, the most "similar" articles with dog2 is dog1, followed by cat and then other food articles. This can be used for non-personalized recommendation, i.e. show the same recommendation for anyone browser the same article. 

## User-based CF

We can use Item-based CF for non-personalized recommendation. How about personalized recommendation? We can start with user-based CF.

Next, we can use similar approach to find similar user. The only difference is now we use the row vector as a representation of a user (where column vector fis the representation of an article) i.e. [1,1,1,0,0,0] is the user behavior of user_0.

Here is our DataFrame again:

|        | dog1 | dog2 | cat  | food1 | food2 | food3 |
| ------ | ---- | ---- | ---- | ----- | ----- | ----- |
| user_0 | 1    | 1    | 1    | 0     | 0     | 0     |
| user_1 | 1    | 1    | 0    | 0     | 0     | 0     |
| user_2 | 0    | 0    | 1    | 0     | 0     | 0     |
| user_3 | 0    | 0    | 0    | 1     | 1     | 0     |
| user_4 | 1    | 0    | 0    | 0     | 1     | 1     |

```
model_knn = NearestNeighbors(metric = 'cosine', algorithm = 'brute')
model_knn.fit(df.values)
```



Again, we fit the data into KNN space, and take an input user and find other users with similar view history.

```python
i = 2
distances, indices = model_knn.kneighbors(df.values[i].reshape(1,-1))

for j in range(0, len(distances.flatten())):
    if j == 0:
        print 'Recommendations for {0}:\n'.format(df.index[i])
    else:
        print '{0}: {1}, with distance of {2}:'.format(j, df.index[indices.flatten()[j]], distances.flatten()[j])
```



```
> Recommendations for user_0:

1: user_1, with distance of 0.183503419072
2: user_2, with distance of 0.42264973081
3: user_4, with distance of 0.666666666667
4: user_3, with distance of 1.0
```



User_0 read all the animal articles and no article, user_1 read 2 dog article and user_2 read 1 cat article, hence, the closest user is user_1, then user_2, and so on.

Okay, so now we can find similar user based on their view history, it isn't really the goal since we are not Facebook, we would not which user see similar articles to you. We only want to show you what other articles How can we leverage this information to do personalized recommendation?

### Personalized recommendation based on User-based CF

Instead of showing them other similar user, we show them the articles those similar user read. To generate a list of articles, I applied a inverse weighted sum model by assigning a heavier weight to articles read by users with a closer distance. I do it for all articles and sort the sum. Hence, the larger the sum is, that means there are more other user view those articles, or those articles are viewed by those user with a similar taste with our target user. (note: we are still using user_0 for this example)

```python
recommend_lst = df.loc[df.index[indices.flatten()[1]]]
for j in range(1, len(distances.flatten())):
    recommend_lst += df.loc[df.index[indices.flatten()[j]]]*(1-distances.flatten()[j])
recommend_lst
```



```
> 
dog1     2.149830
dog2     1.816497
cat      0.577350
food1    0.000000
food2    0.333333
food3    0.333333
```

We can see dog1, dog2 and cat articles in the top ranks, however, they are already read by user_0, hence, we will show user_0 the food2 and food3 next. 

Hence, this is an toy example of how we can use Collaborative filtering and KNN to build a personalized recommendation system. One reason why it is called collaborative filtering is because we need data from a bunch of user collaboratively, the more user the better the recommendation.























