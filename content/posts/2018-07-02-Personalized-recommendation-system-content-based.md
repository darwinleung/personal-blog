---
title: "Personalized recommendation system for articles - Content-based"
date: 2018-07-02T12:00:00-04:00
draft: false
tags: [ "python" , "sklearn", "knn", "recommendation", "content-based"]
categories: [ "machine learning", "data science", "recommedation system", "feature extraction"]
---

# Personalized recommendation system for articles - Content-based

We use the same example from CF post, let's say we have the same 6 articles and 5 visitors and their view history data. Also, we have data about the context of each articles, like title, author, categories, length, no. of picture, and the words from the articles. We might also have data from the user side like their demographic, gender, ages, country etc. We can make use of all these data to build our recommendation. Compare with CF, we only need user behavioral data to make a recommendation.



There are 3 main steps in content-based:

1. Item representation: Generate feature representing article (content itself, frequency of words, length, author, category etc.)
2. User representation: view or not/ session duration/ scroll depth etc.
3. Distance/ Similarity measure

## Find similar articles using content-based

First, let assume we already pre-process the content-related data for articles and reduce the feature space to 3 abstract dimensions about context and let's say we label them "dog-ness", "cat-ness", "food-ness". Of course in reality we might not be able to label them as they are abstract dimensions. In this case, maybe we just count how many times the word "dog"/ "cat"/ "food" appear in the article and use that to define our features. 

Dummy data:

```python
article_features = np.array([[1,0.8,0],[0.99,0.2,0],[0.5,1,0.1],[0,0.1,1],[0.1,0,0.99],[0,0,0.9]])
df2 = pd.DataFrame(article_features,columns = ['dogness','catness','foodness'], index = ['dog1', 'dog2', 'cat', 'food1', 'food2','food3'])
df2.transpose()
```



|          | dog1 | dog2 | cat  | food1 | food2 | food3 |
| -------- | ---- | ---- | ---- | ----- | ----- | ----- |
| dogness  | 1.0  | 0.99 | 0.5  | 0.0   | 0.10  | 0.0   |
| catness  | 0.8  | 0.20 | 1.0  | 0.1   | 0.00  | 0.0   |
| foodness | 0.0  | 0.00 | 0.1  | 1.0   | 0.99  | 0.9   |

In this example, dog1 article talks a lot about dog (1.0), and also cat (0.8) while dog2 article also talks about dog (0.99) a lot but just a bit about cats (0.2) and so on. Hence, we can represent an article by a vector like [1, 0.8, 0] and it represents what the article is about. We can apply KNN to find similar articles.

```python
model_knn = NearestNeighbors(metric = 'cosine', algorithm = 'brute')
model_knn.fit(df2.values)
```



```python
i = 4
distances, indices = model_knn.kneighbors(df2.values[i].reshape(1,-1),n_neighbors = 6)

for j in range(0, len(distances.flatten())):
    if j == 0:
        print 'Recommendations for {0}:\n'.format(df2.index[i])
    else:
        print '{0}: {1}, with distance of {2}'.format(j, df2.index[indices.flatten()[j]], distances.flatten()[j])
```



```
> Recommendations for food2:

1: food3, with distance of 0.0050628109775:
2: food1, with distance of 0.0100004949996:
3: cat, with distance of 0.866598268802:
4: dog2, with distance of 0.901491367424:
5: dog1, with distance of 0.921523695125:
```



As expected, based on the category representation, we found the closest articles with food2 is food3, food1, cat, etc.

## Personal preference prediction with Matrix factorization

Next, we can generate user representation based on their behavioral data, in this example, we simply use the same binary representation to indicate whether they view an article or not. For user_4:

```python
df.loc[['user_4']]
```



|        | dog1 | dog2 | cat  | food1 | food2 | food3 |
| ------ | ---- | ---- | ---- | ----- | ----- | ----- |
| user_4 | 1    | 0    | 0    | 0     | 1     | 1     |

Hence, we know this user viewed 2 food articles and 1 dog article. How can we combine this data with the item categorical representation above (i.e. dogness)? One simple method is to use the proportion of articles viewed, assuming user_4 see the title of all articles but only clicked into 3 of them. Hence we can get a vector for user preference like [1/2 for dogness, 0 for catness, 2/3 for foodness ]. 

If we want to find out the likelihood given an article features vector, we can multiply them together, for example, we want to know the likelihood of user_4 on food1:
$$
\begin{bmatrix}
    0.5 & 0 & 0.66 \\
    \end{bmatrix}
    \cdot
    \begin{bmatrix}
    0\\
    0.1\\
    1\\
    \end{bmatrix} = 0.66
$$
user_4 on dog2:
$$
\begin{bmatrix}
    0.5 & 0 & 0.66 \\
    \end{bmatrix}
    \cdot
    \begin{bmatrix}
    0.99\\
    0.2\\
    0\\
    \end{bmatrix} = 0.495
$$
user_4 on cat:
$$
\begin{bmatrix}
    0.5 & 0 & 0.66 \\
    \end{bmatrix}
    \cdot
    \begin{bmatrix}
    0.5\\
    1\\
    0.1\\
    \end{bmatrix} = 0.316
$$
We can do this for any new articles then we can sort them by the likelihood, show them the highest N articles in the list. As Andrew Ng noted, we are basically fitting a linear regression to each item to predict their preference.

Of course, there are a lot of simplification and assumption made in this toy example. This is the basic idea of using content-based data for personalized recommendation.















