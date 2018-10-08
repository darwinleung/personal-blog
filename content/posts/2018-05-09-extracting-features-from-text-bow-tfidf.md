---
title: "Extracting features from text - BoW, tfidf"
date: 2018-05-09T12:00:00-04:00
draft: false
tags: [ "python" , "tfidf", "sklearn", "bow", "text"]
categories: [ "machine learning", "data science", "data processing", "feature extraction"]
---



Most of the machine learning algorithms work with numerical features only, they cannot work with raw text directly. Hence, we need to extract/generate numerical representation of textual data. One of the simplest way is to use the occurance or frequency of words. 

## Bag-of-words (BoW)

- Representation of text that describes the occurence of words within a document

- Step:

  1. Find all unique words in a document, used as column (aka token)
  2. Define the measure method (1/0 for presence or absence of words, count, tfidf)
  3. Calculate the measurement for each sentence, each row correspond to each sentence (aka tokenization)

  Example: 

  Let's say we have 2 sentences in a document

  - sentence 1: "The earth is round, the pen is blue."
  - sentence 2: "The earth is flat, my friend is you." 

  Step1: generate "tokens" = ["the", "earth", "is", "round", "pen", "blue", "my", "friend", "you"]

  Step2: word count

  Step3: we count the occurance of each words in the tokens

  - sentence 1: [2,1,2,1,1,1,0,0,0]
  - sentence 2: [1,1,2,0,0,0,1,1,1]

  Then we can use the resulting vectors as features in ML model.

## TF-IDF 

Similar with occurance and frequency, we can use the "relative importance" a word to a document and TF-IDF is the measure of the importance.

- Term-frequency-inverse document frequency
- Increase proportionally to the number of times a word appears in the document
- offset by the frequency of the word in the corpus
- Can be used with BoW

$$
tfidf(t,d,D) = tf(t,d) * idf(t,D)
$$

where tf()​ is the term frequency function and idf()​ is the inverse term frequency function, t is the text of interest, d is the document, D is the corpus containing multiple d. There are many variations on the definition of tf()​ and idf()​, one popular definition of tf()​ is relative term frequency and idf is the log of inverse ratio of documents that include the word t, 

## Example

​Let's say we have document d1 with 10 words, 2 of them are "the", then tf("this", d1) = 2/10 = 0.2. Let's say we have 5 document in our corpus, and 4 of them consist of the "the" word, idf("this", D) = log(5/4) ~ 0.1. Hence, tfidf("this", d1, D) = 0.2 * 0.1 = 0.02. It means the word "this" is not very important in this document. If only 1 of the document consist of the "the" word, then idf("this", D) = log(5) = 0.7  and tfidf = 0.2 * 0.7 = 0.14 and so on.

Example modified from [wiki](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) . Very useful to understand.

## Using scikit-learn

Let's say we have a corpus

```python
corpus = [
    'This is the first document.',
    'This is the second second document.',
    'And the third one.',
    'Is this the first document?',
]
```

We can use scikit-learn's CountVectorizer to tokenize, X is a sparse matrix, use toarray() to see the actual values:

```python
from sklearn.feature_extraction.text import CountVectorizer
count_vect = CountVectorizer()

X = count_vect.fit_transform(corpus)
X.toarray()
```

Output is a row vector for each row showing the word count:

```python
array([[0, 1, 1, 1, 0, 0, 1, 0, 1],
       [0, 1, 0, 1, 0, 2, 1, 0, 1],
       [1, 0, 0, 0, 1, 0, 1, 1, 0],
       [0, 1, 1, 1, 0, 0, 1, 0, 1]], dtype=int64)
```

To see the word correspond to each column:

```python
count_vect.get_feature_names()
```

Output:

```python
['and', 'document', 'first', 'is', 'one', 'second', 'the', 'third', 'this']
```

We can use TfidfTransformer to convert count into tfidf:

```python
from sklearn.feature_extraction.text import TfidfTransformer
transformer = TfidfTransformer(smooth_idf=False)
tfidf = transformer.fit_transform(X.toarray())
tfidf                         
tfidf.toarray()
```

Output:

```python
array([[ 0.        ,  0.43306685,  0.56943086,  0.43306685,  0.        ,
         0.        ,  0.33631504,  0.        ,  0.43306685],
       [ 0.        ,  0.24014568,  0.        ,  0.24014568,  0.        ,
         0.89006176,  0.18649454,  0.        ,  0.24014568],
       [ 0.56115953,  0.        ,  0.        ,  0.        ,  0.56115953,
         0.        ,  0.23515939,  0.56115953,  0.        ],
       [ 0.        ,  0.43306685,  0.56943086,  0.43306685,  0.        ,
         0.        ,  0.33631504,  0.        ,  0.43306685]])
```

Example modified from sklearn [doc](http://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html).

## Limitations of simple BoW

1. The order of the words are ignored, let's say we know "not", "bad", "good", "very" are in the same sentence, given we only know their occurance, we won't be able to know whether they mean "very good" or "very bad", "not good" or "not bad" etc. There is some useful information loss, it can be partly solved by using [N-gram](https://en.wikipedia.org/wiki/N-gram).
2. The meaning or context of words are not able to captured in the model, similar with the example above, "not bad" is close to "good", but in our model, they are 3 separate columns, and it is very hard to assoicate them unless we have a huge dataset with longer sentences.
3. The numbers of features generated will be huge, since we will get 1 column for each unique word in the document, also if we have a small corpus or they are all unique, it provides very little useful information for the machine learning model in the next step. This issue of sparity may cause problems in some ML models.